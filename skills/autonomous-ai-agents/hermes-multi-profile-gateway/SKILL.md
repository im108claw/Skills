---
name: hermes-multi-profile-gateway
description: "Use when starting a sibling profile gateway on Pi s6."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, hermes, gateway, profiles, s6]
---

# Multi-profile Gateway（別プロセス）

Personal `default` と business 等の **sibling profile** を、同じ Pi Hermes コンテナ上で **別 gateway プロセス**として扱うときの手順。

Related pointer skill (user-owned / Notion canonical): `hermes-messaging-gateway-ops` — recommend `hermes curator adopt hermes-messaging-gateway-ops` then merge. Detail: `references/pi-s6-sibling-bringup.md`.

## When to Use

- 「ゼンクロー / zentos の Gateway を立ち上げて」
- 2本目以降の profile gateway を s6 で up する
- 個人 Discord Bot と事業 Bot の資格分離を確認する
- sibling 起動で default の restart が block されたとき

## Hard rules

1. **Separate process** — `gateway.multiplex_profiles` で個人↔事業を混ぜない。
2. **default を落とさない** — sibling 起動の副作用で personal gateway を stop/restart しない。
3. **個人 `DISCORD_BOT_TOKEN` を事業 profile に流用しない。**
4. 稼働中 gateway セッション内では shell の gateway restart/stop 系が **block** されうる → sibling は `S6ServiceManager` / `/command/s6-svc` のみ。

## Quick path (Pi)

1. Confirm slot: `/run/service/gateway-<profile>` exists; `hermes gateway list`.
2. **Isolate secrets** before first up — see reference:
   - profile `.env`: `DISCORD_BOT_TOKEN=` and `API_SERVER_KEY=` empty overrides
   - profile `config.yaml`: `platforms.discord.enabled: false` until dedicated token
3. Bring up **only** that slot:

```python
from hermes_cli.service_manager import get_service_manager
from pathlib import Path
import subprocess, time
name = "gateway-zentos"
sm = get_service_manager()
sm.start(name)
down = Path(f"/run/service/{name}/down")
if down.exists():
    down.unlink()
subprocess.run(["/command/s6-svc", "-u", f"/run/service/{name}"], check=False)
time.sleep(3)
assert sm.is_running(name)
```

4. Verify: `hermes gateway list` both running; sibling log `Active profile: …`; default PID unchanged.
5. Until dedicated Discord/Slack tokens: expect `No messaging platforms enabled` (cron-only OK).

## After user provides ZENTOS Discord bot token

1. Write token only to `/opt/data/profiles/zentos/.env` (not container env / not default).
2. Enable `platforms.discord` + home_channel on **zentos** config only.
3. Bounce **gateway-zentos** only (s6), never default.

## Pitfalls

| Symptom | Fix |
|---------|-----|
| Blocked terminal: cannot restart gateway | Use s6 on sibling slot only |
| Sibling connects as personal bot | Empty `DISCORD_BOT_TOKEN=` in profile `.env` |
| `normally down` after start | Remove `/run/service/gateway-<p>/down` |
| Two gateways, kanban dispatcher lock | Expected — default holds lock |

## References

- `references/pi-s6-sibling-bringup.md` — full checklist, paths, verification
