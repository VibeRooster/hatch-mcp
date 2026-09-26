# HATCH-AGENT.md

Full agent reference for the Hatch MCP server (`https://mcp.theroost.dev/mcp`). This is the document the server's `instructions` block points to.

## Core loop

1. **`hatch`** once to create the site. Returns `{ hatchId, slug, url, apex, uploads? }`. Show `url` to the user; remember `hatchId` (`tenantId` is a legacy alias).
2. **`upload`** to add or replace files afterwards.
3. **`convert`** to rename. **`lookup`** to recover a lost `hatchId`. **`list`** to see every hatch in a workspace.
4. **`catalog` → `checkout` → `poll_checkout`** to collect payment (Pin, Pack, Roost, Roost Audit). Show `checkoutUrl`.

## Anti-patterns

1. **Never call `hatch` twice for the same site.** Re-hatching creates a new site with a new id and orphans the old one. Rename with `convert`; add or replace files with `upload`; recover a lost id with `lookup`; see every hatch in a workspace with `list`; collect payment with `checkout`.
2. **Never base64-encode images, fonts, or binaries into a tool argument.** Use `manifest` mode and PUT the bytes to the presigned URLs. That is what the system is designed for.
3. **Never invent apex domains.** Use only the list below. If unsure between a specialty apex and the default, prefer the specialty that matches intent; otherwise omit for `theroost.dev`.
4. **Never claim Vibe Rooster will poll the user's APIs** or run a platform cron. It does not.
5. **Never convert `newTier: forever` to collect payment, and never use Stripe MCP for VibeRooster features.** Hatch `checkout` stamps the metadata the webhook needs. Subscription Pins (10 on Roost / Roost Audit) may still convert while slots remain.

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

Required: `tier`. Also accepts `kind`, `ttlSeconds` (1h–7d), `apex`, `hatchId`, `preferredSlug`, `scriptMetadata`, `tosAcceptedAt`.

This connector is anonymous, so it can only create `free` sites. If the user wants a year of keep: hatch `free`, then `checkout` with `grant: "publish"` (Pin, $4.99/yr), show `checkoutUrl`, `poll_checkout` until complete. Do not convert(newTier: forever) unless unused subscription Pins remain. Do not use Stripe MCP (`mcp.stripe.com`) for VibeRooster features.

## HITL policies (workspace default)

Paired workspaces default to **`good_effort`** HITL (admin can set **`required`**). When a policy applies, **`hatch` / `upload` / `deploy` auto-open HITL** and return `decisionId` + `policy`.

| Policy | Behavior |
|--------|----------|
| **`required`** | Agent blocks on `poll_decision` / webhook until resolved. Overwriting a roost with pending review or prior approval **forks a sibling** at `{slug}-v2`, `{slug}-v3`, … **Request changes** stays on the same URL (revise → `continue_decision`). |
| **`good_effort`** | Updates overwrite in place; open HITL becomes **`superseded`** (`decision.superseded` webhook) and a fresh review opens on new content. |

Override with optional `policy` on `hatch`. Unpaired MCP stays opt-in (`await_decision` only).

## Human-in-the-loop

The review surface renders **on the live artifact**. Reviewers see a *Review required* chip that opens a modal — mobile also has a Decisions inbox.

```
await_decision(hatchId, options?, title?, agentOutput?,
               maxIterations?, timeoutSeconds?, contextUrl?, webhookUrl?)
  → decisionId

poll_decision(decisionId, hatchId)     # long-polls ~20s
  → approved | rejected | changes_requested
  | timeout_exceeded | max_iterations_exceeded | superseded | pending_review

# changes_requested is NON-terminal:
continue_decision(decisionId, hatchId, agentOutput?, title?, timeoutSeconds?)
  → reopens the same decision for the next human round
```

Default options are Approve / Request changes / Reject. `poll_decision` returns the reviewer's comment, the settings, the conversation, and the iteration count.

### Webhooks for automation (n8n, Temporal, CI)

When the user wires HITL into an external workflow, **pass `webhookUrl` on `await_decision`** — do not assume a separate operator or queue must create the review.

**MCP opens, webhook closes:** `hatch`/`upload` → `await_decision({ hatchId, webhookUrl: "https://…" })` → store `decisionId` → human reviews on the roost → platform POSTs each transition to `webhookUrl` → partner continues. **`poll_decision` is optional** (fallback only). On `changes_requested`, regenerate and call `continue_decision` for the next round (another webhook follows).

Payload shape and Roost Audit HMAC: repo `PARTNER-WEBHOOKS.md` or partner docs.

Use `share(hatchId, expiresSeconds?)` for a signed, expiring guest URL (`?vt=…`) when a reviewer needs access to a private report without forever-tier password auth.

## Workspace hatches

When this connector is paired to a workspace:

```
list(workspaceId?)             # every hatch in the workspace
                               # omit workspaceId when the session is already scoped
hatch({ workspaceId, … })      # new hatch in that workspace (session can supply workspaceId)
```

Call `list` before `hatch` if you might already have a site. Use the returned `hatchId` with `upload` / `convert` / `await_decision`.

`lookup` and `list` include first-party traffic: `pageviews24h` (HTML navigations, not CSS/JS) and `visitors24h` (approximate unique browsers). `lookup` also returns `topPaths`, `topCountries`, and `topReferrers`. This is not Google Analytics — no page JS, no third-party tracker.

## Identity & pairing

Unauthenticated use works for everything above. Pairing binds a roost to a Vibe Rooster account (or a workspace, via `workspaceId`).

```
whoami(hatchId?)              # never errors; returns a pairing path if unidentified
get_pairing_code(hatchId)     # code + pairingUrl, 10 min TTL — render as QR
get_pairing_code(workspaceId)  # B2B: authorize this agent in a workspace
poll_pairing(code, hatchId)   # waits ~20s for phone approval
                               # → sessionToken, refreshToken, grantId
refresh_session(refreshToken, hatchId | workspaceId)
poll_approval(approvalId, hatchId)       # Tier-2 phone approval result
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

## Payments

Collect payment in-chat with Hatch — **not** Stripe MCP (`mcp.stripe.com`). Stripe MCP is for operators setting up Prices with `vr_grant` metadata; Hatch `checkout` charges VibeRooster's Stripe account and stamps `tenant_id` / `workspace_id` / `org_id` the webhook expects.

```
catalog({ grant? })
checkout({ grant: "publish", hatchId })   # SHOW checkoutUrl
poll_checkout({ sessionId })               # until complete
```

| grant | What | Pass |
|---|---|---|
| `publish` (**Pin** $4.99/yr) | Keep one hatch 1 year | `hatchId` |
| `record` / `credit_topup` | Snapshot / AI credits | `hatchId` |
| `workspace_coin_pack` (**Pack** $4.99/yr/GB) | Extra storage | `hatchId` |
| `workspace_subscription` (**Roost** $259/mo) | Team workspace | `workspaceId` |
| `agentic_pro` (**Roost Audit** $459/mo) | Roost + audit trail | `orgId` or `workspaceId` |
| `custom_domain` (+$29/mo) | Custom hostname | `workspaceId` |

Do not `convert(newTier: forever)` to collect payment. Subscription Pins may convert while 10 slots remain.

## Roost Audit (audited HITL system of record)

Organizations with **Roost Audit** (`vr_grant: agentic_pro` via Stripe) get:

- **Authenticated reviewers** via Cloudflare Access at `id.theroost.dev` (SSO session cookie on `.theroost.dev`)
- **Sealed Decision Records** — platform JWS + hash chain + RFC 3161 timestamp
- **Artifact snapshots** in the archive, independent of free-roost TTL
- **Evidence packs** — `GET https://id.theroost.dev/evidence/{decisionId}` (JSON) or `?format=html`
- **Signed webhooks** — `X-VR-Signature` + `X-VR-Timestamp`
- **Agent registry** — register agents with API keys; pass `agentId` + `agentApiKey` on `await_decision`

Free / unpaired HITL remains anonymous on the artifact chip.
