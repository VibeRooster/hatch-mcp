---
name: diagram-explorable
title: Diagram explorable
description: A clickable diagram beside a detail panel. Selecting a box dims the rest, highlights what it connects to and fills the panel with that part's description; selecting a connection chip walks the reader across the system. Use it to explain an architecture, an org, a supply chain or any structure whose parts relate to each other.
---

# Diagram explorable

Use this when the point is a structure rather than a number: which parts exist, what each one does, and what talks to what. A static picture forces the reader to hold every label in their head. This one answers a part at a time while keeping the whole visible.

The diagram is drawn from a data array, not hand-placed SVG. Edit `NODES` and `EDGES` and the picture, the highlighting and the panel all follow.

## When it fits

- An architecture with four to a dozen named parts.
- A flow of responsibility across teams or vendors.
- Any "how does X reach Y" question a reader asks more than once.

If the reader's real question is "what happens if I change this number," use the lever explorable instead. If it is "what happens first, then next," use the guided walkthrough.

## Hatch

Call `hatch` once:

- `tier`: `"free"`
- `html`: the filled document below

Show the returned `url`. Remember `hatchId`. Later edits use `upload`.

`await_decision` works here: `window.__VR_HITL_GET_SETTINGS__` reports which part the reader had selected, which is usually the thing under discussion.

## Notes

- **Lay out in columns.** Each node gives a `col` and a `row`. The script turns those into coordinates, so moving a box is a number change, not a path edit.
- **Label the edges.** An unlabeled arrow says "related somehow." `writes`, `polls every 30s`, `escalates to` is information. The label appears in the panel as well as on the line.
- **Keep the copy short.** A node needs a one-line summary and two to four bullets. Anything longer belongs in prose under the figure.
- **One selection at a time.** Everything else dims rather than disappearing, so the reader keeps the shape of the whole.

## HTML

Replace every `[bracketed]` string. Keep the render loop, the select function and the settings hook.

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
      --accent: #0075a2; --accent-soft: #e7f1f7; --on-accent: #ffffff;
      --sans: system-ui, -apple-system, "Segoe UI", sans-serif;
    }
    * { box-sizing: border-box; }
    body { margin: 0; background: var(--paper); color: var(--ink); font: 400 16px/1.55 var(--sans); }
    .wrap { max-width: 76rem; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; }
    h1 { font: 600 32px/1.2 var(--sans); margin: 0 0 0.5rem; }
    .lede { color: var(--muted); max-width: 60ch; margin: 0 0 1.75rem; }
    .cols { display: grid; grid-template-columns: minmax(0, 1fr) 20rem; gap: 1.5rem; align-items: start; }
    @media (max-width: 62rem) { .cols { grid-template-columns: minmax(0, 1fr); } }
    figure { margin: 0; border: 1px solid var(--line); border-radius: 0.4rem; background: #fff; }
    .scroll { overflow-x: auto; padding: 0.9rem; }
    svg { display: block; width: 100%; height: auto; min-width: 42rem; }
    figcaption { font-size: 13px; color: var(--muted); padding: 0.75rem 1rem; border-top: 1px solid var(--line); }

    .node { cursor: pointer; transition: opacity 0.2s; }
    .node rect { fill: var(--accent-soft); stroke: var(--accent); stroke-width: 1.2; }
    .node .t { fill: var(--ink); font: 600 14px var(--sans); }
    .node .s { fill: var(--muted); font: 400 11px var(--sans); }
    .node:focus { outline: none; }
    .node:focus-visible rect { stroke-width: 3; }
    .node.sel rect { fill: var(--accent); stroke: var(--accent); }
    .node.sel .t, .node.sel .s { fill: var(--on-accent); }
    .node.rel rect { stroke-width: 2.5; }
    svg.focused .node:not(.sel):not(.rel) { opacity: 0.3; }
    svg.focused .edge:not(.on) { opacity: 0.12; }
    .edge { fill: none; stroke: var(--edge); stroke-width: 1.4; marker-end: url(#arrow); }
    .edge.on { stroke: var(--accent); stroke-width: 2.2; }
    .edge-label { fill: var(--muted); font: 500 11px var(--sans); }

    aside { border: 1px solid var(--line); border-radius: 0.4rem; padding: 1.1rem; position: sticky; top: 1rem; }
    aside h2 { font: 600 24px/1.25 var(--sans); color: var(--accent); margin: 0 0 0.4rem; }
    aside .sum { margin: 0 0 0.9rem; }
    aside ul { margin: 0; padding-left: 1.1rem; color: var(--muted); font-size: 14.5px; }
    aside h3 { font: 600 11px/1 var(--sans); letter-spacing: 0.1em; text-transform: uppercase;
               color: var(--muted); margin: 1.2rem 0 0.5rem; padding-top: 0.9rem; border-top: 1px solid var(--line); }
    .chips { display: flex; flex-wrap: wrap; gap: 0.35rem; }
    .chips button { font: 500 13px/1 var(--sans); color: var(--ink); background: #fff; border: 1px solid var(--line);
                    border-radius: 999px; padding: 0.45rem 0.7rem; cursor: pointer; }
    .chips button:hover { border-color: var(--accent); color: var(--accent); }
    .chips .via { color: var(--muted); }
  </style>
</head>
<body>
  <main class="wrap">
    <h1>[Title]</h1>
    <p class="lede">[One or two sentences. Say what the diagram shows and that clicking a box explains it.]</p>
    <div class="cols">
      <figure>
        <div class="scroll"><svg id="dg" role="img" aria-label="[Describe the diagram in one sentence for screen readers]"></svg></div>
        <figcaption>[What the arrows mean. One sentence.]</figcaption>
      </figure>
      <aside id="panel" aria-live="polite"></aside>
    </div>
  </main>

  <script>
    // col = column from the left (0, 1, 2 …), row = position within that column.
    const NODES = [
      { id: "[part-a]", title: "[Part A]", sub: "[two or three words]", col: 0, row: 0,
        summary: "[What this part is for, in one line.]",
        points: ["[Point.]", "[Point.]"] },
      { id: "[part-b]", title: "[Part B]", sub: "[two or three words]", col: 1, row: 0,
        summary: "[What this part is for.]",
        points: ["[Point.]", "[Point.]"] },
      { id: "[part-c]", title: "[Part C]", sub: "[two or three words]", col: 1, row: 1,
        summary: "[What this part is for.]",
        points: ["[Point.]"] },
      { id: "[part-d]", title: "[Part D]", sub: "[two or three words]", col: 2, row: 0,
        summary: "[What this part is for.]",
        points: ["[Point.]", "[Point.]"] }
    ];
    const EDGES = [
      { from: "[part-a]", to: "[part-b]", label: "[verb]" },
      { from: "[part-a]", to: "[part-c]", label: "[verb]" },
      { from: "[part-b]", to: "[part-d]", label: "[verb]" },
      { from: "[part-c]", to: "[part-d]", label: "[verb]" }
    ];

    const NS = "http://www.w3.org/2000/svg";
    const W = 190, H = 68, GAP_X = 110, GAP_Y = 34, PAD = 16;
    const byId = Object.fromEntries(NODES.map(n => [n.id, n]));
    const cols = Math.max(...NODES.map(n => n.col)) + 1;
    const rows = Math.max(...NODES.map(n => n.row)) + 1;
    const width = PAD * 2 + cols * W + (cols - 1) * GAP_X;
    const height = PAD * 2 + rows * H + (rows - 1) * GAP_Y;
    const svg = document.getElementById("dg");
    svg.setAttribute("viewBox", `0 0 ${width} ${height}`);

    function el(tag, attrs, parent, text) {
      const n = document.createElementNS(NS, tag);
      for (const k in attrs) n.setAttribute(k, attrs[k]);
      if (text != null) n.textContent = text;
      (parent || svg).appendChild(n);
      return n;
    }
    function box(n) {
      return { x: PAD + n.col * (W + GAP_X), y: PAD + n.row * (H + GAP_Y), w: W, h: H };
    }

    const defs = el("defs", {});
    const marker = el("marker", { id: "arrow", viewBox: "0 0 10 10", refX: "9", refY: "5",
      markerWidth: "8", markerHeight: "8", markerUnits: "userSpaceOnUse", orient: "auto-start-reverse" }, defs);
    el("path", { d: "M0,0 L10,5 L0,10 z", fill: "var(--edge)" }, marker);

    EDGES.forEach((e, i) => {
      const a = box(byId[e.from]), b = box(byId[e.to]);
      const x1 = a.x + a.w, y1 = a.y + a.h / 2, x2 = b.x - 5, y2 = b.y + b.h / 2;
      const mid = (x1 + x2) / 2;
      e.path = el("path", { d: `M${x1},${y1} C${mid},${y1} ${mid},${y2} ${x2},${y2}`,
        class: "edge", "data-i": i });
      if (e.label) e.text = el("text", { x: mid, y: (y1 + y2) / 2 - 6, "text-anchor": "middle", class: "edge-label" }, svg, e.label);
    });

    NODES.forEach(n => {
      const b = box(n);
      const g = el("g", { class: "node", "data-id": n.id, tabindex: "0", role: "button", "aria-label": n.title });
      el("rect", { x: b.x, y: b.y, width: b.w, height: b.h, rx: 7 }, g);
      el("text", { x: b.x + b.w / 2, y: b.y + b.h / 2 - 2, "text-anchor": "middle", class: "t" }, g, n.title);
      if (n.sub) el("text", { x: b.x + b.w / 2, y: b.y + b.h / 2 + 16, "text-anchor": "middle", class: "s" }, g, n.sub);
      const pick = () => select(n.id);
      g.addEventListener("click", pick);
      g.addEventListener("keydown", ev => {
        if (ev.key === "Enter" || ev.key === " ") { ev.preventDefault(); pick(); }
      });
      n.g = g;
    });

    const panel = document.getElementById("panel");
    let current = NODES[0].id;

    function select(id) {
      current = id;
      const n = byId[id];
      const links = [];
      svg.classList.add("focused");
      EDGES.forEach(e => {
        const on = e.from === id || e.to === id;
        e.path.classList.toggle("on", on);
        if (on) links.push({ other: e.from === id ? e.to : e.from, label: e.label, out: e.from === id });
      });
      const related = new Set(links.map(l => l.other));
      NODES.forEach(o => {
        o.g.classList.toggle("sel", o.id === id);
        o.g.classList.toggle("rel", related.has(o.id));
      });
      panel.innerHTML = `
        <h2>${n.title}</h2>
        <p class="sum">${n.summary}</p>
        <ul>${n.points.map(p => `<li>${p}</li>`).join("")}</ul>
        ${links.length ? `<h3>Connects to</h3><div class="chips">${links.map(l =>
          `<button type="button" data-go="${l.other}">${byId[l.other].title}${l.label ? ` <span class="via">${l.out ? "" : "← "}${l.label}</span>` : ""}</button>`
        ).join("")}</div>` : ""}`;
      panel.querySelectorAll("[data-go]").forEach(b => b.addEventListener("click", () => select(b.dataset.go)));
    }

    select(current);

    window.__VR_HITL_GET_SETTINGS__ = () => ({
      state: { selected: current },
      defaults: { selected: NODES[0].id }
    });
  </script>
</body>
</html>
```
