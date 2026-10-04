# Templates

HTML shapes and explorable archetypes. Agents call `list_templates` on `https://mcp.theroost.dev/mcp`. That tool reads this directory on `main` and returns each template's name, title, and description. The HTML stays in the file. Open the file, or the `url` the tool returns, for the details.

A new file on `main` shows up in `list_templates` within about five minutes. No worker deploy is required for another template.

## Add one

1. Add `templates/<kebab-name>.md`.
2. Start with this frontmatter. Keep each value on one line.

```markdown
---
name: kebab-name
title: Short title
description: One or two sentences an agent can use to decide whether this shape fits.
---
```

3. Below the frontmatter, write when to use it and the HTML the agent should fill in.
4. Open a pull request. `name` should match the filename.

A file with no `description` is left off the list. This README is not a template.

## First template

- [Lever explorable](./lever-explorable.md) — one claim, two numeric levers, a result that moves, and the HITL settings hook.
