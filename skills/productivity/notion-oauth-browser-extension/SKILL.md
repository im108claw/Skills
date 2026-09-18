---
name: notion-oauth-browser-extension
description: "Use when a browser extension needs Notion Public OAuth."
version: 1.1.0
author: とーやクロー
license: MIT
metadata:
  hermes:
    tags: [toya-claw, notion, chrome-extension, oauth, cloudflare]
    related_skills: [toya-cloudflare-ops, notion, drive-file-exchange, toya-chat-reply]
---

# Notion Public OAuth × browser extension

## When to Use

- Chrome/Edge MV3 extension needs **Notion user OAuth** (not Internal integration token paste alone).
- Capture UX like Fast Notion / Alfred: center overlay, destination picker, quick send.
- Fixed public redirect on the user's domain (e.g. `https://app…/…/oauth`).

Don't use for: Internal integration + static token only; pure Notion API scripts without a browser client.

## Hard rules

1. **Never put `client_secret` in the extension** (or git). Token exchange runs on a small backend (Cloudflare Worker preferred on this home).
2. **client_id** may ship in the extension (authorize URL); still prefer options override.
3. Do **not** paste secrets into Discord. Source from 1Password service account → Worker secrets.
4. Final chat replies stay short (`toya-chat-reply`); deliver binaries via Drive (`drive-file-exchange`).

## Architecture (default)

```
Extension (MV3)
  ├─ command / action → inject Alfred-style center overlay
  ├─ START_OAUTH → api.notion.com/v1/oauth/authorize
  │     client_id, response_type=code, owner=user, redirect_uri, state
  └─ content_script on redirect host → OAUTH_HANDOFF

Backend (Worker on redirect path)
  ├─ GET …/oauth          code(+state) → Basic(client_id:secret) → /v1/oauth/token
  ├─ KV one-time handoff id (short TTL, e.g. 5 min)
  ├─ 302 …/oauth/complete?handoff=
  └─ GET …/api/handoff?id=&state=  one-time + state match → JSON token

Extension storage
  └─ access_token local; destinations; last destination
```

### Capture modes (Swift Notion lock)

| Destination | Action |
|-------------|--------|
| Normal **page** | `PATCH /v1/blocks/{id}/children` → unchecked `to_do` |
| **database** | `POST /v1/pages` + `database_id`; title prop auto-detect |

Search: `POST /v1/search` (only resources shared with the connection).

## Redirect host routing (this home)

When the hostname already serves another origin (e.g. Vercel on `app.tcaret2.jp`):

1. DNS **proxied** (orange cloud), same origin target.
2. Zone Workers route: `host/path-prefix/*` → Worker.
3. Verify with DoH / `curl --resolve` to CF anycast — local DNS may still show origin IP.

Full CF/MCP deploy notes: `references/cf-worker-oauth-backend.md`.

## 1Password → Worker secrets

```bash
export PATH="/opt/data/bin:/opt/data/.local/bin:$PATH"
export HOME=/opt/data/home
export OP_SERVICE_ACCOUNT_TOKEN="$(cat /opt/data/home/.config/op/service-account-token)"
op item get <id> --vault 'とーやクロー' --format json   # write file; no chat dump
# API_CREDENTIAL often: username=client_id, credential=client_secret
# Put via Cloudflare MCP secrets API; wipe temps; never echo values in the user reply
```

If tool logs may have retained a secret, tell とーや to **rotate** the Notion client secret.

## Security checklist

- [ ] Extension generates `state`; backend stores it with handoff; handoff API rejects mismatch
- [ ] Handoff KV deleted after pickup (one-time)
- [ ] Short TTL on handoff keys
- [ ] Health may show `secrets_bound` only — never secret values
- [ ] Logout clears tokens

## Deliverable layout (chat-only OK)

```
work/<slug>/
  extension/   # Load unpacked
  worker/      # OAuth + handoff
  README.md
```

`tar -czf` extension → Drive `exchange/to-toya` (ASCII name). `zip` may be missing on Pi.

## Pitfalls

- chrome:// / Web Store cannot inject overlay → extension fallback page.
- **Authorize tab never opens:** SW-only `tabs.create` after async message is insufficient — return `authorizeUrl` and open under the click gesture; surface `lastError`. Class detail: skill `chrome-extension-oauth-handoff` → `references/oauth-start-tab-open.md`.
- Search empty until pages/DBs are shared with the integration.
- DB rows are `page` objects — destination **type** must come from search `object`.
- Re-upload Worker script: re-include KV binding; verify secret **names** still listed (`secrets_bound` on `/health`).
- MCP OAuth tokens ≠ raw `api.cloudflare.com` bearer (9106). Use MCP `execute` when wrangler is logged out.
- Tooling that greps gateway `.env` / full `config.yaml` for Discord/CF tokens may be **blocked inside gateway** — prefer documented s6 env files / MCP / 1Password paths.
- After MVP lock: private GitHub OK; keep secret out of git; public client_id in extension OK.

## Related

- skill `chrome-extension-oauth-handoff` (generic MV3 confidential-client handoff + tab-open fix)
- skill `toya-cloudflare-ops` (user-owned Notion-canonical ops; recommend `hermes curator adopt` if it should absorb CF recipes)
- skill `drive-file-exchange` / `toya-chat-reply` / `notion` / `toya-claw-project-ops`
- Example: `/opt/data/work/swift-notion` · GitHub `im108claw/swift-notion` (private) · PRJ-11
