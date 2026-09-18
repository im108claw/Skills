---
name: hermes-wordpress-mcp
description: "Use when connecting WordPress MCP or changing tcaret2.jp (monologue) theme/layout."
version: 1.1.0
metadata:
  hermes:
    tags: [wordpress, mcp, tcaret2, hermes, monologue, risolnet]
    related_skills: [hermes-agent, tcaret2jp-longform-content, hermes-messaging-gateway-ops, drive-file-exchange]
---

# WordPress MCP ↔ Hermes (+ monologue site ops)

Connect self-hosted WordPress to Hermes as MCP, and operate **とーや's blog** when theme/layout work is requested.

**Primary site (とーや):** `https://tcaret2.jp/` (brand: **monologue**)  
**WP user slug (public REST):** `toya071213`  
**Active theme (2026-09):** `risolnet` 1.0.4 (wp.org classic theme)

## When to load

- User mentions WordPress MCP, `wordpress-mcp`, `mcp-adapter`, `composer install` in a WP plugin dir
- Connecting Hermes / Claude / Cursor to the blog
- JWT vs application password / “半恒久” auth for MCP
- Error: `Please run composer install in the plugin directory: .../wordpress-mcp...`
- Theme / homepage layout on tcaret2.jp (Future / Articles / Risolnet / monologue)

## Plugin choice

| Plugin | Status | Notes |
|--------|--------|--------|
| `Automattic/wordpress-mcp` | Works now; **deprecating** | Endpoint bare-domain → `/wp-json/wp/v2/wpmcp`. Release ZIP includes `vendor/` + built assets. |
| `WordPress/mcp-adapter` | Canonical going forward | Default server URL pattern: `/wp-json/mcp/mcp-adapter-default-server`. Prefer for new installs. |

### Install: never use raw trunk without build

- **Do:** download **GitHub Releases** `wordpress-mcp.zip` (e.g. v0.2.5) → upload/activate. No Composer on the host.
- **Don’t:** clone/upload `wordpress-mcp-trunk` without `composer install --no-dev` (+ often `npm run build`). That triggers the composer-missing admin error.
- Host path on shared hosting often looks like `/home/<account>/.../public_html/wp-content/plugins/...` — **not on the Pi**. Run Composer/SSH on the **web host**, or skip Composer via release ZIP.

## Auth (prefer semi-permanent)

| Method | Lifetime | Hermes fit |
|--------|----------|------------|
| **Application Password** + `mcp-wordpress-remote` (stdio) | Until revoked | **Default for Hermes on Pi** |
| JWT (plugin UI) | Often presented as 1–24h | Annoying to re-issue |
| JWT via REST `expires_in` | Code default max **30 days** (`wpmcp_jwt_max_expiration_time` filter) | Still expires |
| OAuth (mcp-adapter + remote) | Refreshable | Better long-term; browser flow awkward on headless Pi |

### Application password flow (recommended)

1. WP admin: **Users → Profile → Application Passwords** → name e.g. `hermes-mcp`.
2. Hermes stdio MCP (not HTTP streamable — streamable is JWT-only on wordpress-mcp):

```yaml
mcp_servers:
  wordpress:
    command: "npx"
    args: ["-y", "@automattic/mcp-wordpress-remote@latest"]
    env:
      WP_API_URL: "${WP_API_URL}"          # bare origin OK for legacy wordpress-mcp
      WP_API_USERNAME: "${WP_API_USERNAME}"
      WP_API_PASSWORD: "${WP_API_PASSWORD}"
      OAUTH_ENABLED: "false"
    timeout: 120
    connect_timeout: 90
```

3. Secrets in `~/.hermes/.env` only (not `config.yaml` literals). Hermes interpolates `${VAR}` / `${env:VAR}` in MCP config.
4. Register via Hermes MCP helpers (`hermes mcp add` or `_save_mcp_server`), then `hermes mcp test wordpress`.
5. Restart gateway after MCP add so Discord/CLI sessions see tools (`mcp_wordpress_*`).

### Sanity checks

```bash
# Plugin alive (401 without auth is OK)
curl -sI "https://tcaret2.jp/wp-json/wp/v2/wpmcp" | head -5

# Public author slug (not always login name — verify if auth fails)
curl -sL "https://SITE/wp-json/wp/v2/users?per_page=10"
```

WP admin: **Settings → WordPress MCP** → enable MCP; optionally Create/Update tools; leave Delete off unless needed.

## Credential intake (Discord / chat)

**Never ask the user to paste application passwords or JWT into Discord/chat.** User already rejected that.

Safe pattern used on Pi:

1. Drop file `~/.hermes/wordpress-mcp.app-password` (mode `600`) — password on first non-comment line.
2. Apply script: `~/.hermes/bin/wordpress-mcp-apply-password` → writes `WP_API_PASSWORD` into `.env`, scrubs drop file.
3. Or user SSH: edit `.env` / run apply script themselves.
4. Optional: one-line file **attachment** only (not message body); agent imports then deletes local copy; remind user to delete Discord attachment.

Confirm with “入れた” / attachment — never echo the secret back.

## Client URL notes (`@automattic/mcp-wordpress-remote`)

- **Legacy wordpress-mcp:** `WP_API_URL=https://tcaret2.jp/` (origin) still resolves to `/wp-json/wp/v2/wpmcp`.
- **mcp-adapter:** set full server URL, e.g. `https://site/wp-json/mcp/mcp-adapter-default-server`.

## monologue / Risolnet theme ops (tcaret2.jp)

Public theme CSS: `https://tcaret2.jp/wp-content/themes/risolnet/style.css`  
Source when PHP is needed: **wp.org theme ZIP** (`https://downloads.wordpress.org/theme/risolnet.<ver>.zip`) — do **not** curl PHP templates from the live site (they execute and return fatals/empty).

| UI label | Mechanism |
|----------|-----------|
| **Future** (and Section 2/3 titles) | Customizer → **Homepage Sections** → `risolnet_home_section_N_title` / `_enabled` / `_count` / `_source` |
| Main latest-posts grid on home | `index.php` loop under `.content-area` — **no** built-in section title |
| Section layout | §1 full-width primary; §2–3 secondary compact cards (see `template-parts/home-sections.php`) |

**Articles above the main posts grid** (same visual as Future): not a Homepage Section control. Options:

1. **Additional CSS** (fastest if agent cannot write theme files) — see `references/monologue-risolnet-theme.md`
2. Child theme overriding `index.php` wrapping the home posts-grid in `<section class="home-section">` + `h2.home-section__title`
3. Direct parent-theme edit on host (prefer child theme)

**Auth gate before claiming “I will apply on production”:**

- Check `WP_API_PASSWORD` is non-empty (s6 env / MCP child env). Empty → do **not** promise remote apply.
- Intake: `/opt/data/secrets/wordpress-mcp.app-password` (or Hermes-home drop file) + `bin/wordpress-mcp-apply-password` — chat body still forbidden.
- `wp-admin` from agent browser often hits **CloudSecure** (`/cloudsecurewp_404`). Prefer REST/MCP with app password, or hand the user Additional CSS / ZIP via Drive.

**Discord attachments:** gateway `discord` tool may strip media; if user says「添付画像」and attachments are empty, fetch raw Discord REST with bot token from gateway process env and download `attachments[].url`.

## Pitfalls

- Trunk/source zip without `vendor/` → composer error; fix with **release ZIP**, not random host Composer if unavailable.
- HTTP/`streamable` transport requires JWT on wordpress-mcp; app passwords need **stdio proxy**.
- `hermes mcp add` may prompt interactively; non-interactive: save config via `_save_mcp_server` + env refs, test after secrets exist.
- MCP env subprocess is filtered; put credentials in server `env:` or secret-scoped `.env` vars that interpolation can resolve — don’t assume full shell env is passed.
- After config change, **restart gateway** or MCP tools won’t appear in messaging sessions.
- Plugin deprecation: don’t invest heavily in wordpress-mcp-only customizations; plan mcp-adapter migration.
- On this Pi host, `.env` may be a null/char device and s6 `WP_API_PASSWORD` can stay empty even when MCP is “configured” — verify length before theme/API work.
- Enabling Homepage **Section 2** titled Articles is **not** the same as labeling the main posts loop (different layout: secondary compact cards).

## References

- `references/tcaret2jp-setup-state.md` — site-specific paths, env keys, apply script, pending steps
- `references/monologue-risolnet-theme.md` — Risolnet structure, Future/Articles, Additional CSS recipe
- Upstream: https://github.com/Automattic/wordpress-mcp · https://github.com/WordPress/mcp-adapter · npm `@automattic/mcp-wordpress-remote`
- Theme: https://wordpress.org/themes/risolnet/ · SVN/ZIP on downloads.wordpress.org