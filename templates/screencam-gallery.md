---
name: screencam-gallery
title: Screencam gallery
description: A grid of product recordings with play buttons that open a player on the page, built to work on hosts that do not answer HTTP Range requests. Covers preparing the files as well as the HTML. Use it for a product tour, a release demo reel or an explorable's "see it in action" section.
---

# Screencam gallery

Use this when the argument is better shown than described: a product tour, a release note with three short demos, or the closing section of a longer explorable. Each recording gets a card with a poster frame, a title, a sentence and its running time. Clicking a card opens the video in a dialog over the page, so the reader never loses their place.

## Prepare the files first

Raw screen recordings are rarely ready to publish. Fix these before the hatch, because every one of them will otherwise reach the viewer.

- **Format.** iPhone and recent Mac captures are HEVC (H.265), which Chrome and Firefox often refuse. Convert to H.264: `ffmpeg -i in.mp4 -c:v libx264 -crf 25 -profile:v high -movflags +faststart out.mp4`.
- **Dead air.** Recordings frequently open or close with long black stretches. Find them with `ffmpeg -i in.mp4 -vf blackdetect=d=0.5 -f null -` and trim with `-ss` and `-to`.
- **Letterboxing.** A desktop capture shot on a phone-shaped canvas puts the real content in a band. `ffmpeg -i in.mp4 -vf cropdetect -f null -` reports the box; crop to it so the detail is legible.
- **Audio.** Silent tracks are common. `-an` drops them and the file shrinks.
- **Posters.** Pull a frame from about a third of the way in: `ffmpeg -ss 12 -i out.mp4 -frames:v 1 poster.jpg`. Keep the poster small and embed it as a `data:` URI so the grid paints instantly.
- **Size.** Aim for a few megabytes per clip. See the next section for why.

## Why it downloads instead of streaming

A `<video src>` makes the browser ask for byte ranges. A host that answers those requests with a 404 leaves the player with nothing, and the video fails even though the file is fine. This template sidesteps that: it fetches the whole file once with `fetch`, which sends no `Range` header, shows a loading percentage, then plays it from a blob URL. Trimmed, cropped, silent clips of a few megabytes load in a second or two.

If the host does answer Range requests, replace `loadDemo` with a plain `video.src = clip.src` and keep everything else.

## Hatch

Upload the page and the media together, then show the returned `url`:

- `hatch` with `tier: "free"` and a `manifest` listing `index.html` plus every `media/*.mp4`. The manifest mode returns one presigned PUT URL per file; PUT the bytes yourself.
- Remember `hatchId`. Later edits use `upload`, not a second `hatch`.

`await_decision` works here: the page reports which clip was last opened through `window.__VR_HITL_GET_SETTINGS__`.

## HTML

Replace every `[bracketed]` string and the `CLIPS` entries. One entry per recording; `portrait: true` gives a phone-shaped recording a narrower dialog.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>[Title]</title>
  <style>
    :root {
      --paper: #ffffff; --ink: #1a2b3c; --muted: #6c7882; --line: #d8e1e8;
      --accent: #0075a2; --play: #f47920; --on-accent: #ffffff;
      --sans: system-ui, -apple-system, "Segoe UI", sans-serif;
    }
    * { box-sizing: border-box; }
    [hidden] { display: none !important; }
    body { margin: 0; background: var(--paper); color: var(--ink); font: 400 16px/1.55 var(--sans); }
    .wrap { max-width: 72rem; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; }
    h1 { font: 600 32px/1.2 var(--sans); margin: 0 0 0.5rem; }
    .lede { color: var(--muted); max-width: 60ch; margin: 0 0 2rem; }
    h2 { font: 600 24px/1.25 var(--sans); color: var(--accent); margin: 2rem 0 0.25rem; }
    .group > p { color: var(--muted); margin: 0 0 1rem; }
    .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(17rem, 1fr)); gap: 1.1rem; }
    .grid.tall { grid-template-columns: repeat(auto-fill, minmax(13rem, 1fr)); }
    .card { display: flex; flex-direction: column; text-align: left; font: inherit; color: inherit;
            background: var(--paper); border: 1px solid var(--line); border-radius: 0.4rem;
            padding: 0; overflow: hidden; cursor: pointer; }
    .card:hover { border-color: var(--accent); }
    .card:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
    .thumb { position: relative; height: 11rem; background: #fff; border-bottom: 1px solid var(--line); }
    .grid.tall .thumb { height: 18rem; }
    .thumb img { width: 100%; height: 100%; object-fit: cover; object-position: top center; display: block; }
    .grid.tall .thumb img { object-fit: contain; }
    .play { position: absolute; left: 50%; top: 50%; width: 3.6rem; height: 3.6rem; margin: -1.8rem 0 0 -1.8rem;
            border-radius: 50%; background: var(--play); display: grid; place-items: center; }
    .play::before { content: ""; margin-left: 0.25rem; border-left: 1rem solid #fff;
                    border-top: 0.6rem solid transparent; border-bottom: 0.6rem solid transparent; }
    .len { position: absolute; right: 0.5rem; bottom: 0.5rem; font: 600 11.5px/1 var(--sans);
           color: #fff; background: rgba(26, 43, 60, 0.82); padding: 0.3rem 0.45rem; border-radius: 0.25rem; }
    .meta { display: grid; gap: 0.25rem; padding: 0.8rem 0.9rem 0.9rem; }
    .ttl { font: 600 16px/1.3 var(--sans); color: var(--accent); }
    .dsc { font-size: 13.5px; line-height: 1.45; color: var(--muted); }

    dialog { width: min(70rem, calc(100vw - 2rem)); max-width: none; padding: 0; border: 1px solid var(--line);
             border-radius: 0.4rem; background: var(--paper); color: var(--ink); }
    dialog.portrait { width: min(27rem, calc(100vw - 2rem)); }
    dialog::backdrop { background: rgba(8, 14, 12, 0.72); }
    .head { display: flex; align-items: center; justify-content: space-between; gap: 0.75rem;
            padding: 0.7rem 0.7rem 0.7rem 1.1rem; border-bottom: 1px solid var(--line); }
    .head h3 { font: 600 18px/1.2 var(--sans); margin: 0; }
    .head button { font: 600 14px/1 var(--sans); border: 1px solid var(--line); background: var(--paper);
                   color: var(--ink); border-radius: 0.4rem; padding: 0.5rem 0.75rem; cursor: pointer; }
    .stage { position: relative; background: #000; }
    video { display: block; width: 100%; height: auto; max-height: 80vh; background: #000; }
    .status { position: absolute; left: 50%; top: 50%; transform: translate(-50%, -50%); margin: 0;
              font: 600 13px/1 var(--sans); color: #fff; background: rgba(0, 0, 0, 0.66);
              padding: 0.6rem 0.9rem; border-radius: 999px; white-space: nowrap; }
  </style>
</head>
<body>
  <main class="wrap">
    <h1>[Title]</h1>
    <p class="lede">[One or two sentences. Say what these recordings show.]</p>
    <div id="groups"></div>
  </main>

  <dialog id="player" aria-labelledby="player-title">
    <div class="head">
      <h3 id="player-title"></h3>
      <button type="button" id="close">Close</button>
    </div>
    <div class="stage">
      <video id="video" controls playsinline preload="none"></video>
      <p class="status" id="status" hidden>Loading video…</p>
    </div>
  </dialog>

  <script>
    // One entry per recording. `poster` is a data: URI or a path next to the page.
    const CLIPS = [
      { id: "[clip-one]", title: "[Clip one title]", desc: "[One sentence about what it shows.]",
        len: "[0:42]", group: "[Group name]", src: "media/[clip-one].mp4", poster: "media/[clip-one].jpg" },
      { id: "[clip-two]", title: "[Clip two title]", desc: "[One sentence.]",
        len: "[1:10]", group: "[Group name]", src: "media/[clip-two].mp4", poster: "media/[clip-two].jpg" },
      { id: "[clip-three]", title: "[Clip three title]", desc: "[One sentence.]",
        len: "[0:20]", group: "[Second group name]", portrait: true,
        src: "media/[clip-three].mp4", poster: "media/[clip-three].jpg" }
    ];

    const byId = Object.fromEntries(CLIPS.map(c => [c.id, c]));
    const dialog = document.getElementById("player");
    const video = document.getElementById("video");
    const status = document.getElementById("status");
    const title = document.getElementById("player-title");
    let last = null;

    const groups = [...new Set(CLIPS.map(c => c.group))];
    document.getElementById("groups").innerHTML = groups.map(g => {
      const tall = CLIPS.some(c => c.group === g && c.portrait);
      const cards = CLIPS.filter(c => c.group === g).map(c => `
        <button type="button" class="card" data-clip="${c.id}" aria-label="Play ${c.title}, ${c.len}">
          <span class="thumb">
            <img src="${c.poster}" alt="" loading="lazy">
            <span class="play" aria-hidden="true"></span>
            <span class="len">${c.len}</span>
          </span>
          <span class="meta"><span class="ttl">${c.title}</span><span class="dsc">${c.desc}</span></span>
        </button>`).join("");
      return `<section class="group"><h2>${g}</h2><div class="grid${tall ? " tall" : ""}">${cards}</div></section>`;
    }).join("");

    document.querySelectorAll("[data-clip]").forEach(b => {
      b.addEventListener("click", () => play(b.dataset.clip));
    });

    // Fetch sends no Range header, so this works on hosts that 404 range requests.
    const blobs = {};
    async function load(clip, onProgress) {
      if (blobs[clip.src]) return blobs[clip.src];
      const res = await fetch(clip.src);
      if (!res.ok) throw new Error("HTTP " + res.status);
      const total = Number(res.headers.get("content-length")) || 0;
      const reader = res.body.getReader();
      const chunks = [];
      let got = 0;
      for (;;) {
        const { done, value } = await reader.read();
        if (done) break;
        chunks.push(value);
        got += value.length;
        if (total) onProgress(got / total);
      }
      return (blobs[clip.src] = URL.createObjectURL(new Blob(chunks, { type: "video/mp4" })));
    }

    async function play(id) {
      const clip = byId[id];
      last = id;
      title.textContent = clip.title;
      dialog.classList.toggle("portrait", !!clip.portrait);
      if (video.dataset.clip !== id) { video.pause(); video.removeAttribute("src"); video.load(); }
      video.setAttribute("poster", clip.poster);
      dialog.showModal();
      if (video.dataset.clip === id) { video.play().catch(() => {}); return; }
      status.hidden = false;
      status.textContent = "Loading video…";
      try {
        const url = await load(clip, f => { status.textContent = "Loading video… " + Math.round(f * 100) + "%"; });
        if (title.textContent !== clip.title) return;
        video.src = url;
        video.dataset.clip = id;
        status.hidden = true;
        if (dialog.open) video.play().catch(() => {});
      } catch (e) {
        status.textContent = "The video could not load. Check your connection and try again.";
      }
    }

    document.getElementById("close").addEventListener("click", () => dialog.close());
    dialog.addEventListener("close", () => video.pause());
    dialog.addEventListener("click", e => { if (e.target === dialog) dialog.close(); });

    // Deep link: /?clip=[clip-one] opens that recording.
    try {
      const want = new URLSearchParams(location.search).get("clip");
      if (want && byId[want]) setTimeout(() => play(want), 200);
    } catch (e) {}

    window.__VR_HITL_GET_SETTINGS__ = () => ({
      state: { lastOpened: last, count: CLIPS.length },
      defaults: { lastOpened: null, count: CLIPS.length }
    });
  </script>
</body>
</html>
```
