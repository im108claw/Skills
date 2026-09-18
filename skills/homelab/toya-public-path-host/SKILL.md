---
name: toya-public-path-host
description: "Use when Pi HTTP publishes on tcaret2.jp without Tailnet, or MC Java ops/console on that host."
version: 1.0.0
author: とーやクロー
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [toya-claw, cloudflare, path-proxy, homelab, no-tailnet]
    related_skills: [toya-cloudflare-ops, hermes-cloudflare-mcp, toya-claw-project-ops]
---

# Pi 公開パスホスト（IP非公開・Tailnetなし）

とーや向け: Pi 上の HTTP を `*.tcaret2.jp/...` で出す。**既定は Cloudflare**。未知の TCP SaaS を勝手に足さない。

## When to Use

- 「IPは公開せず」「Tailnetなし」でドメイン/パス公開
- `games.tcaret2.jp/mc-java` のような **パス付き**エントリ
- path-proxy（:8787）や im108claw と同じ型の追加ホスト
- MC Java on this host: start/restart, **op 付与**, console/RCON, Fabric light mods

## Don't

- playit / ngrok 等を**確認なしで**導入（とーや未指定ツール禁止）
- Minecraft 等の **TCP 参加先**を HTTP パスだと偽る
- tunnel `origin_ip` や宅内グローバル IP をチャットに書く

## Steps

1. Also load `toya-cloudflare-ops` if CF auth/deploy details needed.
2. Local origin first (path-proxy or dedicated port). Prove `127.0.0.1` health.
3. Prefer extend **existing** `raspberrypi` tunnel + path-proxy over new tunnels.
4. Tunnel ingress PUT if needed; if DNS 403 → **Worker route** on already-proxied hostname.
5. Verify with browser UA; short report: public URL + real protocol users need.
6. **Long-running** (MC / path-proxy / tunnel client): always `terminal(..., background=true)`. Foreground 180s timeout kills the server and looks “done” then 502.
7. If とーや says slow / 「並列」: parallel tool calls or `delegate_task` for verify/setup streams — don’t serialize health checks.

## Live IDs (2026-09)

| Item | Value |
|------|--------|
| Account | `7a73f666dace0bb8d2f5eb9fd00d185e` |
| Zone | `tcaret2.jp` |
| Tunnel | `raspberrypi` `69b903e3-983f-4e0f-a48b-08502cf0a337` |
| HTTP origin | `http://127.0.0.1:8787` path-proxy |
| Example host | `im108claw.tcaret2.jp` |
| games path | Worker `games-tcaret2-proxy` route `games.tcaret2.jp/*` |

Detail: `references/path-publish-recipe.md`.  
MC list ping: `scripts/mc-list-ping.py HOST PORT`.

## Minecraft / game servers

- Web entry path = OK via Worker/path-proxy (`/mc-java` landing + `status.json`).
- In-game join = **TCP `host:port` only**. CF Free Worker/path ≠ multiplayer connect string. Never claim path-join.
- MC **26.2** needs **Java 25** (class file 69). Temurin user-local: `/opt/data/.local/java/current` (JRE 21 fails with `UnsupportedClassVersionError`).
- Workdir: `/opt/data/work/mc-java` — `start.sh` launches **`fabric-server-launch.jar`** (not plain `server.jar`); `server.properties` `server-ip=127.0.0.1`, `online-mode=true` (正規垢). Vanilla jar kept as `server.jar` for Fabric.
- Connect string file: `/opt/data/work/mc-java/connect.txt` (status.json reads it).
- **「今すぐマルチプレイ」** and CF Free/Spectrum unavailable: after **one-line** what/why (IP非公開のTCP中継), `bore local 25565 --to bore.pub` (aarch64 musl binary under `/opt/data/bin/bore`). Verify with `scripts/mc-list-ping.py` **before** telling とーや to join. Do **not** default playit (unknown brand → 反感).
- path-proxy: if live status helper `:8790` is down, fall back to static `homepage/mc-java/status.json` (avoid public 502).

### Lightweight / “OptiFine” requests

- **OptiFine = client-only.** Never install OptiFine (or Sodium) on the dedicated server. Say so in one line, then do server-side pack or client guide.
- **26.2 client FPS:** Fabric client + **Sodium** (+ Iris if shaders). OptiFine 26.2 is often missing/unstable.
- **Server perf (vanilla clients still join):** Fabric Loader + release mods only — Fabric API, Lithium, FerriteCore, Krypton, ServerCore, Spark. Skip alpha C2ME/VMP unless とーや asks max risk.
- Convert recipe + pin versions: `references/mc-java-fabric-light-mods.md`.
- Before restart: note online players; SIGTERM **exact** `java …fabric-server-launch.jar` PID only (avoid `pgrep` self-match / multi-kill); backup under `backups/`; keep **bore** + `games-http` running across MC restarts.

### OP / commands（「op付与」「コマンド許可」）

- **OP = in-game commands.** Level 4 + `op-permission-level=4` covers 「コマンド使用も許可」. Command blocks: `enable-command-block=true`.
- UUIDs from `usercache.json` (or log `UUID of player …`). Write `ops.json` then apply live via **RCON** `op <name>` (or restart so file loads).
- **Console access:** process stdin is often `/dev/null` (gateway-spawned). Prefer workdir helpers:
  - `./start.sh` — named FIFO `console.in` + `./console-cmd.sh "say hi"`
  - Local RCON `127.0.0.1:25575` + `python3 rcon-cmd.py "…"` (password only in `server.properties` mode 600; **never** Discord)
- **Pitfall:** do **not** run `op list` — that **ops a player named `list`**, it does not list operators. Confirm with `ops.json` / `op <known>` (“already is an operator”).
- Detail + JSON shape: `references/mc-java-ops-console.md`.

## Wrangler OAuth limits (this home)

| OK | Often 403 |
|----|-----------|
| Workers deploy/routes | DNS records CRUD |
| Tunnel configurations PUT | `cloudflared tunnel route dns` without cert.pem |

Device login: **one** `wrangler login --device` at a time; only share the live code.

## Related

- `toya-cloudflare-ops`（user-owned Notion pointer — recommend `hermes curator adopt toya-cloudflare-ops` if curator should extend it）
- `hermes-cloudflare-mcp`
