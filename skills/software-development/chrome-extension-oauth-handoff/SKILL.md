---
name: chrome-extension-oauth-handoff
description: "Use when Chrome ext needs OAuth with a secret backend."
version: 1.1.0
author: とーやクロー
license: MIT
metadata:
  hermes:
    tags: [chrome-extension, oauth, notion, cloudflare]
    related_skills: [toya-cloudflare-ops, drive-file-exchange, toya-claw-project-ops]
---

# Chrome extension OAuth handoff (confidential client)

## When to Use

- MV3 extension must call an API that issues tokens only with **client_secret** (Notion Public OAuth and similar)
- Fixed HTTPS redirect on own domain (not only `*.chromiumapp.org`)
- Alfred-style **center overlay** capture UI + destination picker

Don't use for: internal integration token pasted into options only (no OAuth).

## Hard rules

1. **Never** ship `client_secret` inside the extension bundle.
2. Redirect URI is an exact string match — lock it early with the user.
3. Token exchange happens on a **backend** (Cloudflare Worker preferred on this home).
4. Extension receives tokens via **one-time handoff**, not durable tokens in URL history.

## Canonical flow

```
Extension → provider authorize (client_id, redirect_uri, state)
  → redirect https://app…/…/oauth?code&state
  → Worker: code→token (Basic client_id:secret)
  → KV put handoff:<uuid> (TTL ~10m, one-time)
  → 302 /oauth/complete?handoff=<uuid>
  → content_script on host → background GET /api/handoff?id=
  → chrome.storage.local access_token; KV delete
```

### Extension pieces

| Piece | Role |
|-------|------|
| `background` SW (module) | OAuth start, handoff fetch, API calls |
| `commands` + `scripting` | Center overlay inject on active tab |
| `content_scripts` on redirect path | Bridge complete page → SW |
| `options_ui` | Store **client_id only** |
| fallback extension page | chrome:// cannot inject overlay |

### OAuth start — open the authorize tab (must)

Symptom fixed 2026-09 (Swift Notion): UI shows「OAuthを開いています…」then error / **no tab**.

**Do both** (see `references/oauth-start-tab-open.md`):

1. SW `START_OAUTH` returns `{ ok, authorizeUrl, tabOpened, tabError }` after optional `chrome.tabs.create`
2. Overlay click handler opens `authorizeUrl` under the **user gesture** (`<a target=_blank>.click()` then `window.open`)
3. Overlay `sendMessage` **must** surface `chrome.runtime.lastError` (never map undefined → silent "no response" only)
4. Seed public `client_id` on `onInstalled` if storage empty; never seed secret

If both open paths fail, show the full authorize URL for manual open.

### Notion capture defaults (Swift Notion)

- **Page** destination → append **to_do** (unchecked) via block children
- **Database** destination → create page row; detect title property from DB schema and cache
- Search shared pages/DBs via `POST /v1/search`; user pins a destination list

### Worker surface

- `GET …/oauth` — exchange
- `GET …/oauth/complete` — HTML bridge target
- `GET …/api/handoff?id=` — one-time JSON + CORS
- `GET …/health` — smoke

Path hosting details: `references/worker-path-route-over-origin.md`.

## User setup checklist

1. Public OAuth app; redirect URI exact.
2. Worker + KV + secrets for client id/secret.
3. Load unpacked (or Drive tar of `extension/`); options → client_id.
4. Shortcut → connect → add destinations → capture.

## Delivery

- Pi worktree temp only; ship package via **Drive** `exchange/to-toya` (`drive-file-exchange`).
- Reference tree (2026-09): `/opt/data/work/swift-notion/` — product **Swift Notion**, redirect `https://app.tcaret2.jp/extension/swift-notion/oauth`.

## Verify before claiming OAuth works

Ship/run a small checker (example: `scripts/verify-oauth.mjs` in product tree):

- Static: no raw `secret_*` in extension; `tabs` + host_permissions present; dual-open markers
- Live: Worker `/health` → `secrets_bound: true`; authorize URL HTTP 302 to notion.com; handoff without id fails; bare `/oauth` errors

## Pitfalls

- Provider secret exchange cannot be skipped with `chrome.identity` alone when secret is required.
- **SW-only `tabs.create` after async message** → no tab / opaque error — always return `authorizeUrl` and open from the click path.
- Complete page must match `content_scripts` + `host_permissions` or handoff never runs.
- After handoff, `tabs.sendMessage` overlays on other tabs to refresh connected state.
- Handoff API should require **state match** + one-time delete + short TTL.
- DB title keys vary — always detect, never hardcode only `Name`.
- Chat-only builds: skip DB_Project unless user asks PRJ launch (`toya-claw-project-ops`).
- After MVP: private GitHub under user org is normal; **never** commit client_secret; public `client_id` OK.
- Overlaps skill `notion-oauth-browser-extension` (Notion-specific); keep class Chrome handoff rules here.

## Related

- `notion-oauth-browser-extension` (Notion capture defaults + 1P→secrets)
- `toya-cloudflare-ops` (user-owned Notion-canonical pointer; path-route detail stays in this skill's refs unless adopted)
- `drive-file-exchange` / `toya-claw-project-ops`
- `references/worker-path-route-over-origin.md`
- `references/oauth-start-tab-open.md`
