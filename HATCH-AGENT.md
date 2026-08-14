# HATCH-AGENT.md

Full agent reference for the Hatch MCP server (`https://mcp.theroost.dev/mcp`). This is the document the server's `instructions` block points to.

## Core loop

1. **`hatch`** once to create the site. Returns `{ tenantId, slug, url, apex, uploads? }`. Show `url` to the user; remember `tenantId`.
2. **`upload`** to add or replace files afterwards.
3. **`convert`** to rename or change tier. **`lookup`** to recover a lost `tenantId`.

## Anti-patterns

1. **Never call `hatch` twice for the same site.** Re-hatching creates a new site with a new id and orphans the old one. Rename or change tier with `convert`; add or replace files with `upload`; recover a lost id with `lookup`.
2. **Never base64-encode images, fonts, or binaries into a tool argument.** Use `manifest` mode and PUT the bytes to the presigned URLs. That is what the system is designed for.
3. **Never invent apex domains.** Use only the list below. If unsure between a specialty apex and the default, prefer the specialty that matches intent; otherwise omit for `theroost.dev`.
4. **Never claim Vibe Rooster will poll the user's APIs** or run a platform cron. It does not.

## Choosing an apex

Pass `apex` on `hatch`. Pick from user intent — always prefer a specialty apex when one fits.

| Apex | Intent |
|---|---|
| `theroost.dev` | Default; generic roost, app, dashboard, explorable |
| `theroost.homes` | One residential home for sale |
| `theroost.estate` | Commercial RE / luxury property marketing |
| `theroost.land` | Land, lots, development parcels |
| `theroost.wedding` | Wedding info / RSVP |
| `theroost.events` | Conference, meetup, one-off event |
| `theroost.agency` | Freelancer / agency pitch or portfolio |
| `theroost.site` | Short-lived site with no named vertical |
| `theroost.rentals` | **RESERVED — do not hatch here** |

## `hatch` modes

| Mode | When |
|---|---|
| *(omit all)* | Instant placeholder page. Best zero-token first turn. |
| `manifest` | **Preferred** for anything with images, fonts, or CSS. Returns presigned PUT URLs in `uploads[]`. Upload each file's bytes: `curl -T <local> -H 'Content-Type: <mime>' "$url"`. Bytes never travel through the tool call. |
| `site` | Inline file map. Only for small text-only sites (a few HTML/CSS files). |
| `script` | Advanced. Full server-side code as one ES module, text only, 1.5 MiB max. |

Required: `tier`. Also accepts `kind`, `ttlSeconds` (1h–7d), `apex`, `tenantId`, `preferredSlug`, `scriptMetadata`, `tosAcceptedAt`.

This connector is anonymous, so it can only create `free` sites. If the user wants permanence: hatch `free`, show the URL, then point to https://viberooster.com/connect.html#upgrade

## Human-in-the-loop

The review surface renders **on the live artifact**. Reviewers see a *Review required* chip that opens a modal — there is no separate approval inbox to check.

```
await_decision(tenantId, options?, title?, agentOutput?,
               maxIterations?, timeoutSeconds?, contextUrl?, webhookUrl?)
  → decisionId

poll_decision(decisionId, tenantId)     # long-polls ~20s
  → approved | rejected | changes_requested
  | timeout_exceeded | max_iterations_exceeded | pending_review

# changes_requested is NON-terminal:
continue_decision(decisionId, tenantId, agentOutput?, title?, timeoutSeconds?)
  → reopens the same decision for the next human round
```

Default options are Approve / Request changes / Reject. `poll_decision` returns the reviewer's comment, the settings, the conversation, and the iteration count.

Use `share(tenantId, expiresSeconds?)` for a signed, expiring guest URL (`?vt=…`) when a reviewer needs access to a private report without forever-tier password auth.

## Identity & pairing

Unauthenticated use works for everything above. Pairing binds a roost to a Vibe Rooster account.

```
whoami(tenantId?)              # never errors; returns a pairing path if unidentified
get_pairing_code(tenantId)     # code + pairingUrl, 10 min TTL — render as QR
poll_pairing(code, tenantId)   # waits ~20s for phone approval
                               # → sessionToken, refreshToken, grantId
refresh_session(refreshToken, tenantId)   # renews ~1h token; grant lasts 7 days
poll_approval(approvalId, tenantId)       # Tier-2 phone approval result
```

Do not call `get_pairing_code` again until `poll_pairing` returns `expired` or `completed` — repeated calls invalidate the outstanding code. Each refresh rotates the `refreshToken`; store the new one. `refresh_session` requires the same connector session (`MCP-Session-Id`) used at pairing.

## Keeping dashboards fresh

Hatch hosts static (or script) sites. It does **not** run user Python/Node on a schedule, and roosts do not store third-party API keys.

If a user builds a dashboard that regenerates HTML from backends:

1. Hatch/upload the current HTML so they have a live URL now.
2. Say plainly that freshness stays on their side — cron, launchd, CI, or asking you again to re-run the generator and call `upload`.
3. Keep credentials in their env or CI secrets. Never embed keys in HTML or in `deploy` script source.

Suggested line after the first hatch:

> Your roost is live at {url}. To refresh it, re-run the generator (locally or in CI) and I'll upload again — or set a cron that regenerates and uploads.

## `run-report` prompt

`prompts/get` with name `run-report` scaffolds an agent observability report — what I did / inferred / about to do / confidence / alternatives — then hatch it with `kind: "run-report"`.

Arguments: `title` (short run title), `ttlHours` (1–168, default 48).

Pair with `share` to hand a reviewer a private link, or with `await_decision` to require sign-off on what the agent did.
