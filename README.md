# Hatch — the Vibe Rooster MCP server

**Publish a live website from your AI assistant in one tool call.**

Hatch is [Vibe Rooster](https://viberooster.com)'s official MCP connector. Ask your assistant to build something, and it goes live at a real HTTPS URL — no account, no API key, no Docker, no cloud console.

```
https://mcp.theroost.dev/mcp
```

Remote server, Streamable HTTP, anonymous. 15 tools, 1 prompt.

- **Registry:** [`com.viberooster/hatch`](https://registry.modelcontextprotocol.io/v0.1/servers?search=com.viberooster/hatch) in the official MCP Registry
- **Install guide:** https://viberooster.com/install.html
- **Tool reference & tiers:** https://viberooster.com/connect.html
- **Agent reference:** [HATCH-AGENT.md](./HATCH-AGENT.md)

---

## Install

**Claude.ai / Claude Desktop** — Settings → Connectors → Add custom connector, paste the URL, auth **None**.

**Claude Code**

```bash
claude mcp add --transport http hatch https://mcp.theroost.dev/mcp
```

**Cursor** (`~/.cursor/mcp.json`)

```json
{ "mcpServers": { "hatch": { "url": "https://mcp.theroost.dev/mcp" } } }
```

**Windsurf** (`~/.codeium/windsurf/mcp_config.json`) — note the field is `serverUrl`, not `url`:

```json
{ "mcpServers": { "hatch": { "serverUrl": "https://mcp.theroost.dev/mcp" } } }
```

**stdio-only clients** — bridge with [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) (Node 20+):

```json
{
  "mcpServers": {
    "hatch": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.theroost.dev/mcp", "--transport", "http-only"]
    }
  }
}
```

Full per-platform instructions: https://viberooster.com/install.html

---

## Try it

> "Hatch me a one-page landing site for my coffee shop and give me the live link."

> "Here's my portfolio — index.html, styles.css, and three photos. Put it online."

> "Build a status dashboard from this script and tell me how to keep it updating."

> "Use the run-report prompt, fill a compliance run report, and hatch it. Give me the live URL."

---

## Tools

### Publishing

| Tool | What it does |
|---|---|
| `hatch` | Create a **new** site and return `{ tenantId, slug, url, apex }`. Four modes: empty (instant placeholder), `manifest` (presigned PUT URLs — preferred for images/fonts/CSS), `site` (inline files, small text-only sites), `script` (server-side ES module). |
| `upload` | Add or replace files on an existing roost. Returns one presigned PUT URL per file; bytes never pass through the tool call. |
| `lookup` | Resolve a roost by `slug` or `tenantId` — recovers state when context is lost. |
| `convert` | Atomically rename a roost and/or change tier (`free` → `forever`), or toggle gallery listing. |
| `deploy` | Advanced: replace a roost's server-side code with a full ES module (1.5 MiB max). |

### Human-in-the-loop

Hatch renders the review UI **on the live artifact itself** — reviewers see a "Review required" chip that opens a modal. No separate approval inbox.

| Tool | What it does |
|---|---|
| `await_decision` | Open a review on a live roost. Default options Approve / Request changes / Reject; supports custom options, `maxIterations`, `timeoutSeconds`, `contextUrl`, `webhookUrl`. |
| `poll_decision` | Long-poll (~20s) until the decision resolves: `approved`, `rejected`, `changes_requested`, `timeout_exceeded`, `max_iterations_exceeded`. Returns comment, conversation and iteration count. |
| `continue_decision` | After `changes_requested`, regenerate and reopen the same decision for the next human round. |
| `share` | Signed, expiring guest view URL (`?vt=…`) for any tier — private run reports without password auth. |

### Access & identity

| Tool | What it does |
|---|---|
| `auth` | Put a sign-in screen in front of a `forever` roost (shared site password). |
| `whoami` | Caller identity and tenant state; returns a pairing path when unidentified rather than erroring. |
| `get_pairing_code` | Issue a pairing code + URL (10 min TTL) for claiming a tenant — render as a QR for the Vibe Rooster app. |
| `poll_pairing` | Device-grant style poll for phone approval. Returns `sessionToken`, `refreshToken`, `grantId`. |
| `refresh_session` | Renew a ~1h access token using the refresh token; the grant lasts up to 7 days. |
| `poll_approval` | Poll a pending Tier-2 phone approval. |

## Prompts

| Prompt | What it does |
|---|---|
| `run-report` | Scaffold an HTML agent run report — what I did / inferred / about to do / confidence / alternatives — and hatch it as a live URL. Args: `title`, `ttlHours` (1–168, default 48). |

---

## Where sites live

Each published site is a **roost**. The default namespace is `theroost.dev`; vertical namespaces are sized to a real-world lifecycle:

| Apex | For |
|---|---|
| `theroost.dev` | Default — apps, dashboards, explorables |
| `theroost.homes` | One residential listing, live till it sells |
| `theroost.estate` | Commercial / luxury property, live through closing |
| `theroost.land` | Land, lots, development parcels |
| `theroost.wedding` | Wedding info & RSVP, through the day and after |
| `theroost.events` | Conference, meetup, one-off event microsite |
| `theroost.agency` | Freelancer or agency pitch / portfolio |
| `theroost.site` | Generic short-lived site |

## Tiers

- **Free** — live instantly, 48h default TTL (configurable 1h–7d via `ttlSeconds`), 3 free Continues (8 live days total). No account, no card.
- **Continue** — $4.99 buys two more weeks.
- **Forever** — $24.99 one-time, permanent and always-on.

This connector is anonymous and creates **free** sites only. Upgrading to Forever requires an account: https://viberooster.com/connect.html#upgrade

## What Hatch does not do

- It does not run your Python/Node on a schedule. Roosts are static (or a server-side script you supply). Refreshing a generated dashboard stays on your side — cron, CI, or asking your agent again and calling `upload`.
- It does not store third-party API keys. Keep credentials in your env or CI secrets; never embed them in HTML or in `deploy` script source.

---

## Links

[Vibe Rooster](https://viberooster.com) · [Install](https://viberooster.com/install.html) · [Connect to Claude](https://viberooster.com/connect.html) · [Explorables](https://viberooster.com/explorables.html) · [Privacy](https://viberooster.com/privacy.html) · [hello@viberooster.com](mailto:hello@viberooster.com)

© 2026 Vibe Rooster, Inc.
