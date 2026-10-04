---
name: lever-explorable
title: Lever explorable
description: One-page explorable with two numeric levers and a result that updates as the reader moves them. Fill the copy, keep the HITL settings hook, then hatch the HTML.
---

# Lever explorable

Use this when the reader should change two assumptions and see one number move. It is a starting shape, not a finished argument.

## Hatch

Call `hatch` once:

- `tier`: `"free"`
- `html`: the filled document below

Show the returned `url`. Remember `hatchId`. Do not hatch again for this page. Later edits use `upload`.

If a person must sign off, call `await_decision` after the hatch. The review attaches the lever values because the page sets `window.__VR_HITL_GET_SETTINGS__`. On `changes_requested`, regenerate using `settings` and `comment`, then `continue_decision`.

## HTML

Replace every `[bracketed]` string. Keep the inputs, the result update, and the settings hook.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>[Title]</title>
  <style>
    :root { color-scheme: light dark; }
    body { font: 16px/1.5 system-ui, sans-serif; max-width: 40rem; margin: 2rem auto; padding: 0 1.25rem; }
    output { display: block; font-size: 1.75rem; font-weight: 650; margin: 1rem 0; }
    label { display: block; margin: 1rem 0; }
  </style>
</head>
<body>
  <h1>[Claim the reader can test]</h1>
  <p>[One sentence of context. Say what the two levers change.]</p>
  <label>[Lever A label] <input id="a" type="range" min="0" max="100" value="40"></label>
  <label>[Lever B label] <input id="b" type="range" min="0" max="100" value="60"></label>
  <output id="result"></output>
  <p id="reading"></p>
  <script>
    const a = document.getElementById("a");
    const b = document.getElementById("b");
    const result = document.getElementById("result");
    const reading = document.getElementById("reading");
    const defaults = { a: 40, b: 60 };
    function value() {
      const av = Number(a.value);
      const bv = Number(b.value);
      return { a: av, b: bv, score: Math.round((av + bv) / 2) };
    }
    function render() {
      const v = value();
      result.textContent = String(v.score);
      reading.textContent = "[Sentence that uses the score.]";
    }
    a.addEventListener("input", render);
    b.addEventListener("input", render);
    render();
    window.__VR_HITL_GET_SETTINGS__ = () => ({
      state: value(),
      defaults
    });
  </script>
</body>
</html>
```
