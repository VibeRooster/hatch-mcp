# Hatch — the Vibe Rooster MCP server

**Publish a live website from your AI assistant in one tool call.**

Hatch is [Vibe Rooster](https://viberooster.com)'s official MCP connector. Ask your assistant to build something, and it goes live at a real HTTPS URL — no account, no API key, no Docker, no cloud console.

```
https://mcp.theroost.dev/mcp
```

Remote server, Streamable HTTP, anonymous. 19 tools, 1 prompt.

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
| `hatch` | Create a **new** site and return `{ hatchId, slug, url, apex }`. Four modes: empty (instant placeholder), `manifest` (presigned PUT URLs — preferred for images/fonts/CSS), `site` (inline files, small text-only sites), `script` (server-side ES module). |
| `upload` | Add or replace files on an existing roost. Returns one presigned PUT URL per file; bytes never pass through the tool call. |
| `lookup` | Resolve a roost by `slug` or `hatchId` — recovers state when context is lost. |
| `list` | List every hatch in a workspace (paired workspace session). Call before hatching again so you do not duplicate a site. |
| `convert` | Atomically rename a roost, toggle gallery listing, or bind a custom domain after checkout. Subscription Pins may `convert(newTier: forever)` while slots remain; paid Pins use `checkout` grant `publish`. |
| `catalog` | List payable features (Pin, Pack, Roost, Roost Audit, custom domain) and Stripe Price ids. |
| `checkout` | Create a Stripe Checkout Session. **Show `checkoutUrl`.** Then `poll_checkout`. |
| `poll_checkout` | Wait until the user pays; the webhook applies the grant. |
| `deploy` | Advanced: replace a roost's server-side code with a full ES module (1.5 MiB max). |

### Human-in-the-loop

Hatch renders the review UI **on the live artifact itself** — reviewers see a "Review required" chip that opens a modal. Mobile also has a Decisions inbox.

**Workspace policies:** `required` (fork sibling `{slug}-vN` when review is outstanding/approved) or `good_effort` (overwrite in place; supersede open HITL). Paired workspaces auto-open HITL on `hatch` / `upload` / `deploy` and return `decisionId`.

| Tool | What it does |
|---|---|
| `await_decision` | Open a review on a live roost. Default options Approve / Request changes / Reject; supports custom options, `maxIterations`, `timeoutSeconds`, `contextUrl`, `webhookUrl`. |
| `poll_decision` | Long-poll (~20s) until the decision resolves: `approved`, `rejected`, `changes_requested`, `timeout_exceeded`, `max_iterations_exceeded`, `superseded`. Returns comment, conversation and iteration count. |
| `continue_decision` | After `changes_requested`, regenerate and reopen the same decision for the next human round. |
| `share` | Signed, expiring guest view URL (`?vt=…`) for any tier — private run reports without password auth. |

### Access & identity

| Tool | What it does |
|---|---|
| `auth` | Put a sign-in screen in front of a `forever` roost (shared site password). |
| `whoami` | Caller identity and hatch state; returns a pairing path when unidentified rather than erroring. |
| `get_pairing_code` | Issue a pairing code + URL (10 min TTL) for claiming a hatch — render as a QR for the Vibe Rooster app. |
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

- **Hatch** — free, live instantly, 48h default TTL (configurable 1h–7d via `ttlSeconds`). Try HITL. No account, no card.
- **Pin** — $4.99/yr keeps one hatch for a year.
- **Pack** — $4.99/yr per extra GB.
- **Roost** — $259/mo team workspace (10 subscription Pins, durable HITL, unlimited members).
- **Roost Audit** — $459/mo: Roost plus exportable audit trail and 1-year retention.
- **Custom domain** — +$29/mo on Roost / Roost Audit.

This connector is anonymous and creates **Hatch** sites only. To keep a site a year, ask the agent to collect Pin payment (`checkout` grant `publish`) in the same conversation.

## What Hatch does not do

- It does not run your Python/Node on a schedule. Roosts are static (or a server-side script you supply). Refreshing a generated dashboard stays on your side — cron, CI, or asking your agent again and calling `upload`.
- It does not store third-party API keys. Keep credentials in your env or CI secrets; never embed them in HTML or in `deploy` script source.

---

## Links

[Vibe Rooster](https://viberooster.com) · [Install](https://viberooster.com/install.html) · [Connect to Claude](https://viberooster.com/connect.html) · [Explorables](https://viberooster.com/explorables.html) · [Privacy](https://viberooster.com/privacy.html) · [hello@viberooster.com](mailto:hello@viberooster.com)

© 2026 Vibe Rooster, Inc.
