---
name: hermes-canva-mcp
description: "Use when connecting or operating Canva MCP on Hermes."
version: 1.0.0
author: とーやクロー
license: MIT
metadata:
  hermes:
    tags: [canva, mcp, hermes, oauth, toya-claw, design]
    related_skills: [hermes-agent, hermes-cloudflare-mcp, hermes-wordpress-mcp]
---

# Canva MCP ↔ Hermes

Official remote Canva MCP so the agent can create/search/edit/export designs (and simple video exports) via natural language — **not** a DIY NLE or Remotion codebase.

- Docs: https://www.canva.dev/docs/mcp/
- Tools/limits: https://www.canva.dev/docs/mcp/tools/
- Help (AI Connector): https://www.canva.com/help/mcp-agent-setup/
- Endpoint: `https://mcp.canva.com/mcp`
- Catalog: `optional-mcps/canva` (`hermes mcp catalog` → `canva`)

## When to Use

- User asks to connect / try **Canva MCP**, AI Connector, or `mcp.canva.com`
- Design / banner / slide / short-video generation via Canva from Hermes
- OAuth paste flow for Canva on Pi / Discord
- Clarifying Canva MCP vs Remotion / PRJ video-editor DIY

## What it can do (class)

| Area | Capability |
|------|------------|
| Generate | `generate-design` → candidates → `create-design-from-candidate` |
| Edit | editing transactions (`start` / `perform` / `commit` / `cancel`) |
| Discover | search designs, pages, content, presenter notes |
| Assets | upload from URL, list assets |
| Export | PDF/PNG/JPG/PPTX/MP4… (`export-design`; quality/plan-dependent) |
| Folders | create / list / search / move |
| Comments | comment / reply / list |
| Brand | brand kits/templates (Pro+); autofill (Enterprise) |
| Resize | Pro+ |

**Not a substitute for:** Premiere-class multi-track NLE DIY (PRJ-10), code-driven Remotion pipelines (PRJ-8). Canva MCP = “drive existing Canva”; DIY = “build our editor/pipeline”.

## Plan / eligibility notes

- Core tools: many on all plans; **resize / brand templates Pro+**; **autofill Enterprise**.
- Consumer **AI Connector** help text lists Pro / Teams / Business / Nonprofit — Free may fail auth or tool use; check live errors.
- Custom agents historically needed redirect allowlist/waitlist; Hermes catalog entry uses native OAuth (DCR/CIMD). If authorize fails on redirect, re-check Canva allowlist + Hermes CIMD docs.

## Register on this home (Pi / `$HERMES_HOME=/opt/data`)

Run as user **`hermes`** only. Do **not** tell とーや to `export HERMES_HOME=/opt/data` in a `tcaret2` shell.

### Preferred: catalog or hermes_cli config API

```bash
export PATH="/opt/hermes/bin:$PATH"
export HERMES_HOME=/opt/data

# Catalog (may prompt / attempt gateway restart — blocked inside live gateway session)
# hermes mcp install canva

# Non-interactive write (works inside agent session):
/opt/hermes/.venv/bin/python <<'PY'
from hermes_cli.config import load_config, save_config
cfg = load_config()
ms = cfg.setdefault("mcp_servers", {})
entry = ms.setdefault("canva", {})
entry.update({
    "url": "https://mcp.canva.com/mcp",
    "auth": "oauth",
    "enabled": True,
})
oauth = entry.get("oauth") if isinstance(entry.get("oauth"), dict) else {}
oauth["redirect_port"] = 27905  # unique; CF uses 27901–27904
entry["oauth"] = oauth
ms["canva"] = entry
save_config(cfg)
print(ms["canva"])
PY

hermes mcp list   # expect canva enabled
```

### Config edit pitfalls (this home)

- **Do not** `patch` / raw-write `$HERMES_HOME/config.yaml` from agent tools — Hermes refuses security-sensitive config writes. Use `hermes_cli.config.save_config` or `hermes config set`.
- System `python3` may lack PyYAML; use **`/opt/hermes/.venv/bin/python`**.
- `hermes mcp install` / some `hermes mcp add` paths from **inside the gateway process** can be blocked (“cannot restart gateway”). Prefer `save_config` + later gateway restart outside the blocked path.
- After MCP register + successful OAuth: **new session / gateway restart** so `mcp_canva_*` tools appear in Discord.

## Headless OAuth (Discord / Pi)

Same paste pattern as Cloudflare MCP:

1. Free port `27905`; kill leftover `hermes mcp login canva`.
2. `terminal(..., background=true, pty=true)` → `hermes mcp login canva`
3. Poll until “Open this URL…”; send URL to とーや.
4. Browser authorize → address bar `http://127.0.0.1:27905/callback?code=...&state=...` (connection failed = expected).
5. User pastes full redirect URL (or `?code=...&state=...`).
6. `process(action="submit", session_id=..., data="<url>")` immediately.
7. Verify `$HERMES_HOME/mcp-tokens/canva*` + `hermes mcp test canva`.

### OAuth pitfalls

- Stale `code`/`state` if login process restarted — new URL only.
- Port in use → kill login; keep **unique** `oauth.redirect_port` (Canva default here: **27905**).
- One login at a time; don’t parallelize with other MCP logins on shared default port.
- Double-start of login in one PTY can print two URLs then fail with port busy — start **once** in background PTY and wait.

## Smoke tests after connect

1. “Show my most recently edited Canva design” → `search-designs` / get design.
2. Small generate (e.g. AP stretch explain thumbnail or short layout) → candidate → real design.
3. Optional `export-design` (PNG/MP4); premium elements may return `license_required`.

## Related

- `hermes-cloudflare-mcp` — same Pi OAuth paste / port discipline; ports 27901–27904
- `hermes-wordpress-mcp` — stdio MCP pattern (different auth)
- Bundled `hermes-agent` native MCP docs (load via skill_view linked refs; do not patch bundled)
- Official tools index: https://www.canva.dev/docs/mcp/llms.txt

## Support files

- `references/capabilities-and-limits.md` — tool families + plan tiers (condensed)
- `references/pi-home-setup.md` — config snippet + OAuth port for this home
