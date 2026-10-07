---
name: slide-mode-explorable
title: Slide-mode explorable
description: One scrolling page that doubles as a deck. A presentation toggle collapses it to one section at a time with Previous/Next and arrow keys, and two sliders raise text contrast and text size for a room. Fill the sections, keep the controls and the HITL settings hook, then hatch the HTML.
---

# Slide-mode explorable

Use this when the same page has two jobs: something a reader scrolls on their own, and something the author presents live. Reading mode is an ordinary page. Presentation mode turns each section into a row that opens one at a time, so the audience sees only what is being talked about.

The two sliders exist because projectors and conference-room TVs wash out light text. The presenter raises contrast and size in the room instead of asking people to squint.

It is a starting shape, not a finished argument. Three sections are stubbed; add or remove them and the controls keep up.

## Hatch

Call `hatch` once:

- `tier`: `"free"`
- `html`: the filled document below

Show the returned `url`. Remember `hatchId`. Do not hatch again for this page. Later edits use `upload`.

If a person must sign off, call `await_decision` after the hatch. The review attaches the presenter's current state because the page sets `window.__VR_HITL_GET_SETTINGS__`. On `changes_requested`, regenerate using `settings` and `comment`, then `continue_decision`.

## Notes

- **Sections drive everything.** Each `<section class="slide" id="…">` with an `<h2>` becomes one step. The script reads them at load, so the step count, the Previous/Next buttons and the keyboard follow whatever is in the markup. In presentation mode the `<h2>` is hidden and the section's own row carries the heading, so the title is never shown twice.
- **Reading mode is the default.** The page is a normal scrolling document until someone turns presentation mode on, so a first-time reader never meets a collapsed page.
- **State is per viewer.** Mode, slider values and the open section are kept in `localStorage` inside `try/catch`, so a presenter can set up before a meeting and reopen ready to go. A viewer with storage blocked still gets a working page.
- **Text size uses CSS `zoom`**, which scales diagrams and tables along with the type instead of letting text overflow its boxes. Chrome, Edge, Safari and current Firefox support it.
- **Keep `[hidden] { display: none !important }`.** Section bodies are collapsed with the `hidden` attribute, and a `display` rule elsewhere would otherwise beat it.

## HTML

Replace every `[bracketed]` string. Keep the control bar, the section script, and the settings hook.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
  <title>[Title]</title>
  <style>
    :root {
      --paper: #ffffff; --ink: #1a2b3c; --muted: #6c7882; --line: #d8e1e8;
      --accent: #0075a2; --accent-soft: #e7f1f7; --rule: #f47920; --on-accent: #ffffff;
      --sans: "Segoe UI", system-ui, -apple-system, "Helvetica Neue", Arial, sans-serif;
    }
    * { box-sizing: border-box; }
    [hidden] { display: none !important; }
    body { margin: 0; background: var(--paper); color: var(--ink); font: 400 16px/1.55 var(--sans); }
    .wrap { max-width: 68rem; margin: 0 auto; padding-inline: 1.5rem; }
    header.top { padding-block: 2.5rem 1.25rem; }
    h1 { font: 600 32px/1.2 var(--sans); margin: 0; max-width: 24ch; }
    .lede { color: var(--muted); margin: 0.75rem 0 0; max-width: 60ch; }

    .bar { position: sticky; top: 0; z-index: 5; background: var(--paper); border-block: 1px solid var(--line); }
    .bar .wrap { display: flex; flex-wrap: wrap; align-items: center; gap: 0.75rem 1.5rem; padding-block: 0.6rem; }
    .mode { display: inline-flex; align-items: center; gap: 0.5rem; font: 600 14px/1 var(--sans);
            color: var(--accent); background: var(--paper); border: 1px solid var(--accent); border-radius: 999px;
            padding: 0.55rem 0.95rem; cursor: pointer; }
    .mode[aria-pressed="true"] { background: var(--accent); color: var(--on-accent); }
    .ctrl { display: flex; align-items: center; gap: 0.55rem; font: 600 11px/1 var(--sans);
            letter-spacing: 0.1em; text-transform: uppercase; color: var(--muted); }
    .ctrl input { width: 7rem; accent-color: var(--accent); cursor: pointer; }
    .ctrl output { color: var(--ink); min-width: 3.2em; text-align: right; font-variant-numeric: tabular-nums; }
    .steps { display: flex; align-items: center; gap: 0.4rem; margin-left: auto; }
    .steps .pos { font: 600 12px/1 var(--sans); color: var(--muted); margin-right: 0.3rem; font-variant-numeric: tabular-nums; }
    .steps button { font: 600 14px/1 var(--sans); color: var(--ink); background: var(--paper);
                    border: 1px solid var(--line); border-radius: 0.4rem; padding: 0.5rem 0.7rem; cursor: pointer; }
    .steps button:disabled { opacity: 0.4; cursor: default; }
    button:focus-visible, input:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }

    .slide { padding-block: 2.5rem 0.5rem; scroll-margin-top: 4.5rem; }
    .slide + .slide { border-top: 1px solid var(--line); }
    .slide h2 { font: 600 24px/1.25 var(--sans); color: var(--accent); margin: 0 0 0.25rem; }
    .slide .body { padding-bottom: 1.5rem; }
    .slide .body > :first-child { margin-top: 0.75rem; }
    .toggle { display: none; align-items: center; gap: 0.7rem; width: 100%; text-align: left;
              font: 600 18px/1.3 var(--sans); color: var(--ink); background: none; border: 0;
              padding: 0.85rem 0; margin: 0; cursor: pointer; }
    .toggle .chev { width: 1.1rem; height: 1.1rem; border: 1px solid var(--accent); border-radius: 50%; position: relative; }
    .toggle .chev::before { content: ""; position: absolute; inset: 0.3rem 0.35rem auto; width: 0.3rem; height: 0.3rem;
                            border-right: 1.5px solid var(--accent); border-bottom: 1.5px solid var(--accent); transform: rotate(45deg); }
    .toggle .state { margin-left: auto; font: 600 10.5px/1 var(--sans); letter-spacing: 0.12em;
                     text-transform: uppercase; color: var(--muted); }
    body.present .toggle { display: flex; }
    body.present .slide { padding-block: 0; border-top: none; }
    body.present .slide h2 { display: none; }
    body.present .slide.open .toggle { color: var(--accent); font-size: 20px; border-bottom: 2px solid var(--rule); }
    body.present .slide.shut { border-bottom: 1px solid var(--line); }
    body.present .slide.shut .sub { display: none; }
    body.present .slide.open .sub { margin-top: 0.9rem; }
    @media (prefers-reduced-motion: reduce) { * { transition: none !important; } }
  </style>
</head>
<body>
  <header class="top wrap">
    <h1>[Title the audience sees first]</h1>
    <p class="lede">[One or two sentences. Say what the page shows and what the reader can do with it.]</p>
  </header>

  <div class="bar">
    <div class="wrap">
      <button type="button" class="mode" id="mode" aria-pressed="false">Presentation mode</button>
      <label class="ctrl">Text darkness
        <input type="range" id="dark" min="0" max="100" step="5" value="0" aria-label="Text darkness">
        <output id="dark-out">0%</output>
      </label>
      <label class="ctrl">Text size
        <input type="range" id="scale" min="100" max="180" step="5" value="100" aria-label="Text size">
        <output id="scale-out">100%</output>
      </label>
      <div class="steps" id="steps" hidden>
        <span class="pos" id="pos"></span>
        <button type="button" id="prev">&lsaquo; Previous</button>
        <button type="button" id="next">Next &rsaquo;</button>
      </div>
    </div>
  </div>

  <main class="wrap">
    <section class="slide" id="[section-one]">
      <h2>[First section heading]</h2>
      <p class="sub">[Optional one-line summary of this section.]</p>
      <div class="body">
        <p>[The content. A chart, a table, a diagram or plain prose — anything can live here.]</p>
      </div>
    </section>

    <section class="slide" id="[section-two]">
      <h2>[Second section heading]</h2>
      <p class="sub">[Optional one-line summary.]</p>
      <div class="body">
        <p>[Content.]</p>
      </div>
    </section>

    <section class="slide" id="[section-three]">
      <h2>[Third section heading]</h2>
      <p class="sub">[Optional one-line summary.]</p>
      <div class="body">
        <p>[Content.]</p>
      </div>
    </section>
  </main>

  <script>
    const root = document.documentElement;
    const body = document.body;
    const mode = document.getElementById("mode");
    const steps = document.getElementById("steps");
    const pos = document.getElementById("pos");
    const prev = document.getElementById("prev");
    const next = document.getElementById("next");
    const dark = document.getElementById("dark");
    const darkOut = document.getElementById("dark-out");
    const scale = document.getElementById("scale");
    const scaleOut = document.getElementById("scale-out");
    const defaults = { present: false, dark: 0, scale: 100 };

    const slides = [...document.querySelectorAll(".slide")].map(sec => {
      const content = sec.querySelector(".body");
      const name = sec.querySelector("h2").textContent.trim();
      const btn = document.createElement("button");
      btn.type = "button";
      btn.className = "toggle";
      btn.innerHTML = '<span class="chev" aria-hidden="true"></span><span class="label"></span><span class="state"></span>';
      btn.setAttribute("aria-expanded", "true");
      btn.querySelector(".label").textContent = name;
      btn.addEventListener("click", () => (content.hidden ? open(sec.id, true) : shut(sec)));
      sec.insertBefore(btn, sec.firstChild);
      return { id: sec.id, sec, content, btn, name };
    });
    const byId = Object.fromEntries(slides.map(s => [s.id, s]));
    let present = false;
    let current = slides[0] ? slides[0].id : null;

    function label(s) {
      const closed = s.content.hidden;
      s.btn.querySelector(".state").textContent = closed ? "Show" : "Hide";
      s.btn.setAttribute("aria-expanded", String(!closed));
      s.sec.classList.toggle("shut", closed);
      s.sec.classList.toggle("open", !closed);
    }
    function shut(sec) { const s = byId[sec.id]; s.content.hidden = true; label(s); save(); }
    function open(id, scroll) {
      current = id;
      slides.forEach(s => { s.content.hidden = present && s.id !== id; label(s); });
      const i = slides.findIndex(s => s.id === current);
      pos.textContent = (i + 1) + " / " + slides.length;
      prev.disabled = i <= 0;
      next.disabled = i >= slides.length - 1;
      if (scroll) byId[id].sec.scrollIntoView();
      save();
    }
    function step(d) {
      const i = slides.findIndex(s => s.id === current) + d;
      if (i >= 0 && i < slides.length) open(slides[i].id, true);
    }
    function setPresent(on, scroll) {
      present = on;
      body.classList.toggle("present", on);
      mode.setAttribute("aria-pressed", String(on));
      steps.hidden = !on;
      if (on) open(current, scroll !== false);
      else { slides.forEach(s => { s.content.hidden = false; label(s); }); save(); }
    }
    mode.addEventListener("click", () => setPresent(mode.getAttribute("aria-pressed") !== "true"));
    prev.addEventListener("click", () => step(-1));
    next.addEventListener("click", () => step(1));
    addEventListener("keydown", e => {
      if (!present || e.metaKey || e.ctrlKey || e.altKey) return;
      const t = e.target;
      if (t && (t.tagName === "INPUT" || t.tagName === "TEXTAREA" || t.isContentEditable)) return;
      if (e.key === "ArrowRight" || e.key === "PageDown") { e.preventDefault(); step(1); }
      if (e.key === "ArrowLeft" || e.key === "PageUp") { e.preventDefault(); step(-1); }
    });

    function applyDark() {
      const d = Number(dark.value);
      darkOut.textContent = d + "%";
      root.style.setProperty("--ink", "color-mix(in srgb, #1a2b3c, #000 " + d + "%)");
      root.style.setProperty("--muted", "color-mix(in srgb, #6c7882, #000 " + d + "%)");
      root.style.setProperty("--line", "color-mix(in srgb, #d8e1e8, #000 " + Math.round(d * 0.5) + "%)");
      save();
    }
    function applyScale() {
      const s = Number(scale.value);
      scaleOut.textContent = s + "%";
      body.style.zoom = s / 100;
      save();
    }
    dark.addEventListener("input", applyDark);
    scale.addEventListener("input", applyScale);

    const KEY = "[slug]-stage";
    function save() {
      try {
        localStorage.setItem(KEY, JSON.stringify({ present, dark: dark.value, scale: scale.value, current }));
      } catch (e) {}
    }
    try {
      const saved = JSON.parse(localStorage.getItem(KEY) || "{}");
      if (saved.dark != null) dark.value = saved.dark;
      if (saved.scale != null) scale.value = saved.scale;
      if (saved.current && byId[saved.current]) current = saved.current;
      applyDark();
      applyScale();
      if (saved.present) setPresent(true, false);
    } catch (e) { applyDark(); applyScale(); }
    slides.forEach(label);
    open(current, false);

    window.__VR_HITL_GET_SETTINGS__ = () => ({
      state: { present, section: current, dark: Number(dark.value), scale: Number(scale.value) },
      defaults
    });
  </script>
</body>
</html>
```
