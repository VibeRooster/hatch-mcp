---
name: guided-walkthrough
title: Guided walkthrough
description: One example traced through a system, a step at a time. Back, Play and Next light the trail behind the reader while a panel narrates the current step. Use it for "follow one request through the pipeline", an onboarding flow, an incident timeline or any explanation whose point is the order things happen.
---

# Guided walkthrough

Use this when the reader needs to see sequence: what happens first, what each stage adds, where a thing can stop or branch. A list of stages tells them the parts. This shows one real example moving through those parts, which is what they will remember.

Pick a concrete example before writing the steps. "A rodent alert at 2 a.m." beats "an event," because every step then has something specific to say.

## When it fits

- A request, order, ticket or signal moving through a pipeline.
- An onboarding or approval flow with a handoff in the middle.
- An incident reconstructed after the fact.

If the parts relate to each other but have no order, use the diagram explorable instead.

## Hatch

Call `hatch` once:

- `tier`: `"free"`
- `html`: the filled document below

Show the returned `url`. Remember `hatchId`. Later edits use `upload`.

`await_decision` works here: `window.__VR_HITL_GET_SETTINGS__` reports the step the reader stopped on, so a reviewer's comment lands against a specific stage.

## Notes

- **The trail is the point.** Steps already visited stay lit and the current one is filled, so the reader sees how far along the example is without a progress bar.
- **One claim per step.** Two or three sentences in the panel. If a step needs more, it is probably two steps.
- **Mark the stages that differ.** Set `kind: "aside"` on a stage that is optional, automated or a branch; it renders in the accent color and is called out in the panel.
- **Play is a convenience, not the default.** The page opens on step one, paused. Autoplay without a click is for a kiosk, not a reader.
- **Keep `[hidden] { display: none !important }`** so the controls can hide cleanly.

## HTML

Replace every `[bracketed]` string and the `STEPS` entries. Four to twelve steps works; the row wraps onto a second line when it needs to.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>[Title]</title>
  <style>
    :root {
      --paper: #ffffff; --ink: #1a2b3c; --muted: #6c7882; --line: #d8e1e8; --edge: #95a5b0;
      --accent: #0075a2; --accent-soft: #e7f1f7; --mark: #f47920; --mark-soft: #fdeee2; --on-accent: #ffffff;
      --sans: system-ui, -apple-system, "Segoe UI", sans-serif;
    }
    * { box-sizing: border-box; }
    [hidden] { display: none !important; }
    body { margin: 0; background: var(--paper); color: var(--ink); font: 400 16px/1.55 var(--sans); }
    .wrap { max-width: 76rem; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; }
    h1 { font: 600 32px/1.2 var(--sans); margin: 0 0 0.5rem; }
    .lede { color: var(--muted); max-width: 60ch; margin: 0 0 1.75rem; }
    .cols { display: grid; grid-template-columns: minmax(0, 1fr) 21rem; gap: 1.5rem; align-items: start; }
    @media (max-width: 62rem) { .cols { grid-template-columns: minmax(0, 1fr); } }

    .track { display: flex; flex-wrap: wrap; gap: 0.6rem; border: 1px solid var(--line);
             border-radius: 0.4rem; padding: 1rem; background: #fff; }
    .stage { display: flex; align-items: center; gap: 0.6rem; }
    .stage button { display: grid; gap: 0.15rem; min-width: 9.5rem; text-align: left; font: inherit; color: inherit;
                    background: #fff; border: 1px solid var(--line); border-radius: 0.4rem;
                    padding: 0.6rem 0.75rem; cursor: pointer; opacity: 0.45; transition: opacity 0.2s; }
    .stage button .n { font: 600 10.5px/1 var(--sans); letter-spacing: 0.1em; color: var(--muted); }
    .stage button .t { font: 600 14.5px/1.25 var(--sans); }
    .stage.lit button { opacity: 1; background: var(--accent-soft); border-color: var(--accent); }
    .stage.now button { background: var(--accent); border-color: var(--accent); color: var(--on-accent); }
    .stage.now button .n { color: var(--on-accent); }
    .stage.aside.lit button { background: var(--mark-soft); border-color: var(--mark); }
    .stage.aside.now button { background: var(--mark); border-color: var(--mark); color: var(--on-accent); }
    .stage .arrow { color: var(--edge); font-size: 1.1rem; }
    .stage:last-child .arrow { display: none; }
    button:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }

    aside { border: 1px solid var(--line); border-radius: 0.4rem; padding: 1.1rem; position: sticky; top: 1rem; }
    .bar { display: flex; flex-wrap: wrap; align-items: center; justify-content: space-between;
           gap: 0.5rem; margin-bottom: 0.9rem; }
    .pos { font: 600 12px/1 var(--sans); color: var(--muted); font-variant-numeric: tabular-nums; }
    .btns { display: flex; gap: 0.35rem; }
    .btns button { font: 600 13.5px/1 var(--sans); color: var(--ink); background: #fff;
                   border: 1px solid var(--line); border-radius: 0.35rem; padding: 0.5rem 0.7rem; cursor: pointer; }
    .btns button.primary { background: var(--accent); border-color: var(--accent); color: var(--on-accent); }
    .btns button:disabled { opacity: 0.4; cursor: default; }
    .meter { display: flex; gap: 0.2rem; margin-bottom: 1rem; }
    .meter i { flex: 1; height: 0.25rem; border-radius: 0.15rem; background: var(--line); }
    .meter i.done { background: var(--accent-soft); }
    .meter i.now { background: var(--accent); }
    .meter i.aside.now { background: var(--mark); }
    aside h2 { font: 600 24px/1.25 var(--sans); color: var(--accent); margin: 0 0 0.5rem; }
    aside.aside h2 { color: var(--mark); }
    .tag { display: inline-block; font: 600 10.5px/1 var(--sans); letter-spacing: 0.1em; text-transform: uppercase;
           color: var(--mark); margin-bottom: 0.5rem; }
    aside p { margin: 0 0 0.75rem; }
    .example { font-size: 13px; color: var(--muted); margin: 0; padding-top: 0.8rem; border-top: 1px solid var(--line); }
  </style>
</head>
<body>
  <main class="wrap">
    <h1>[Title]</h1>
    <p class="lede">[One or two sentences. Name the example being traced and invite the reader to step through it.]</p>
    <div class="cols">
      <div class="track" id="track"></div>
      <aside id="panel" aria-live="polite"></aside>
    </div>
  </main>

  <script>
    // kind: "aside" marks an optional, automated or branching stage.
    const STEPS = [
      { id: "[step-one]", title: "[Stage one]", text: "[What happens here, in two or three sentences, using the example.]" },
      { id: "[step-two]", title: "[Stage two]", text: "[What this stage adds.]" },
      { id: "[step-three]", title: "[Stage three]", kind: "aside", text: "[What makes this stage different — optional, automated, or a branch.]" },
      { id: "[step-four]", title: "[Stage four]", text: "[What the reader should take away by now.]" },
      { id: "[step-five]", title: "[Stage five]", text: "[How the example ends.]" }
    ];
    const EXAMPLE_NOTE = "[One line naming the example, e.g. “Illustrative: a single order placed at 9:04 a.m.”]";

    const track = document.getElementById("track");
    const panel = document.getElementById("panel");
    let at = 0;
    let timer = null;

    track.innerHTML = STEPS.map((s, i) => `
      <span class="stage${s.kind === "aside" ? " aside" : ""}" data-i="${i}">
        <button type="button"><span class="n">${String(i + 1).padStart(2, "0")}</span><span class="t">${s.title}</span></button>
        <span class="arrow" aria-hidden="true">&rarr;</span>
      </span>`).join("");
    track.querySelectorAll(".stage").forEach(el => {
      el.querySelector("button").addEventListener("click", () => { stop(); go(Number(el.dataset.i)); });
    });

    function render() {
      const s = STEPS[at];
      track.querySelectorAll(".stage").forEach((el, i) => {
        el.classList.toggle("lit", i <= at);
        el.classList.toggle("now", i === at);
      });
      panel.classList.toggle("aside", s.kind === "aside");
      panel.innerHTML = `
        <div class="bar">
          <span class="pos">Step ${at + 1} of ${STEPS.length}</span>
          <div class="btns">
            <button type="button" id="back" ${at === 0 ? "disabled" : ""}>&lsaquo; Back</button>
            <button type="button" id="play" class="primary">${timer ? "Pause" : at === STEPS.length - 1 ? "Replay" : "Play"}</button>
            <button type="button" id="next" ${at === STEPS.length - 1 ? "disabled" : ""}>Next &rsaquo;</button>
          </div>
        </div>
        <div class="meter" aria-hidden="true">${STEPS.map((x, i) =>
          `<i class="${i < at ? "done" : ""} ${i === at ? "now" : ""} ${x.kind === "aside" ? "aside" : ""}"></i>`).join("")}</div>
        ${s.kind === "aside" ? '<span class="tag">[Label for these stages]</span>' : ""}
        <h2>${s.title}</h2>
        <p>${s.text}</p>
        <p class="example">${EXAMPLE_NOTE}</p>`;
      panel.querySelector("#back").onclick = () => { stop(); go(at - 1); };
      panel.querySelector("#next").onclick = () => { stop(); go(at + 1); };
      panel.querySelector("#play").onclick = () => (timer ? stop(true) : play());
    }
    function go(i) {
      at = Math.max(0, Math.min(STEPS.length - 1, i));
      render();
    }
    function stop(redraw) {
      if (timer) { clearInterval(timer); timer = null; if (redraw) render(); }
    }
    function play() {
      if (at === STEPS.length - 1) at = -1;
      timer = setInterval(() => {
        if (at >= STEPS.length - 1) { stop(true); return; }
        go(at + 1);
      }, 2600);
      go(at + 1);
    }
    addEventListener("keydown", e => {
      const t = e.target;
      if (t && (t.tagName === "INPUT" || t.tagName === "TEXTAREA" || t.isContentEditable)) return;
      if (e.key === "ArrowRight") { e.preventDefault(); stop(); go(at + 1); }
      if (e.key === "ArrowLeft") { e.preventDefault(); stop(); go(at - 1); }
    });

    render();

    window.__VR_HITL_GET_SETTINGS__ = () => ({
      state: { step: at + 1, stepId: STEPS[at].id, of: STEPS.length },
      defaults: { step: 1, stepId: STEPS[0].id, of: STEPS.length }
    });
  </script>
</body>
</html>
```
