---
name: hermes-state-db-ops
description: "Use when Hermes state.db is corrupt/unwritable, or when estimating token/API cost from session history."
version: 1.0.0
created_by: agent
metadata:
  hermes:
    tags: [hermes, state-db, sqlite, recovery, gateway, session-storage]
---

# Hermes state.db / session storage ops

Use when Discord/CLI shows session-persistence failures, `hermes doctor` reports state.db FTS/schema corruption, Home broadcasts `Session database corruption detected`, **or** とーや asks how long a prepaid budget / model switch would last “in this environment” (session-history cost estimate).

## Symptom → meaning

| Surface error | Meaning |
|---|---|
| `session storage could not be written (the transcript would have been lost on restart)` | Turn aborted because session rows could not be persisted. Check state.db health first. |
| `database disk image is malformed` | SQLite page/FTS corruption (often starts as FTS write failure). |
| `malformed database schema ()` | Schema/catalog damage; auto-repair and many backups may already be unusable. |
| Home: `Session database corruption detected` + recovery options | Gateway detected store failure and warned home channels. |

Disk full is a *separate* path (message may say `disk full:`). On this Pi home, free space is usually ample — still check `df -h` once, then focus on SQLite.

## Pi home facts (とーや / raspberrypi)

- Gateway unit: **user** systemd `hermes-gateway.service`
  - Stop/start: `systemctl --user stop|start|status hermes-gateway.service`
  - **Not** `sudo systemctl stop hermes` / `hermes-agent` (those units are not loaded here).
- DB path: `~/.hermes/state.db` (+ `-wal`/`-shm`)
- Auto forensic copies often appear as:
  `state.db.malformed-backup-<YYYYMMDD_HHMMSS>`
  Manual retreat names used in recovery: `state.db.broken`, `state.db.old`
- CLI: `hermes doctor`, `hermes doctor --fix`
- **Do not** run gateway stop/start from a tool shell *inside* the live gateway process (child gets SIGTERM). Prefer instructing とーや / an external shell, or only diagnose while gateway stays up.

## Recovery ladder (safe order)

1. **Diagnose (read-only first)**
   ```bash
   hermes doctor
   hermes doctor --fix   # may attempt FTS/schema repair + backups
   ls -lh ~/.hermes/state.db*
   df -h /
   ```
2. **Stop writers** before file moves:
   ```bash
   systemctl --user stop hermes-gateway.service
   systemctl --user status hermes-gateway.service   # expect inactive/failed/dead
   # only if leftovers remain:
   pgrep -af hermes | head
   ```
3. **Prefer restore over reset**
   ```bash
   cd ~/.hermes
   mv state.db state.db.broken          # keep corpse
   # pick newest plausible backup, then:
   cp state.db.malformed-backup-<ts> state.db
   # integrity (install sqlite3 once if missing: sudo apt install sqlite3)
   sqlite3 state.db "PRAGMA integrity_check;"
   ```
   - `ok` → go to step 5.
   - still malformed → try an *older* backup the same way.
   - all backups fail integrity → step 4.
4. **Reset only when restore is hopeless**
   - This **wipes session history** (SOUL / skills / MEMORY.md style files usually kept).
   - On current Hermes CLI here: `hermes memory reset --confirm` is **invalid**. Use:
     ```bash
     hermes memory reset
     # or: hermes memory reset confirm
     ```
     Follow interactive confirm if prompted.
5. **Verify + start**
   ```bash
   hermes doctor          # state.db should be healthy; npm audit noise can remain
   systemctl --user start hermes-gateway.service
   ```
   Smoke-test Discord send/receive. Expect **empty session history** after reset; tell とーや explicitly.
6. **If Home still warns after reset**
   Full stop again (`stop` + ensure no stale hermes PIDs), re-check doctor, then start once. Concurrent writers / stale handles can re-flag corruption.

Optional salvage (when doctor suggests and restore failed but file still partially readable):

```bash
cd ~/.hermes
sqlite3 state.db ".recover" | sqlite3 state_recovered.db
mv state.db state.db.old
mv state_recovered.db state.db
sqlite3 state.db "PRAGMA integrity_check;"
```

## What reset does / does not drop

| Lost | Usually kept |
|---|---|
| Session transcripts in state.db (`session_search` empty/near-empty) | `SOUL.md`, skills, config, `.env` |
| In-DB routing/session rows | MEMORY.md / USER.md if present as files |
| Cron *execution* history tied to DB rows may thin out | `cron/jobs.json` definitions |

After reset: write a DB_Logs **Session** row documenting cause + steps (skill `notion-session-logging` / `session-log-notion`). Point at forensic files, not secrets.

## Pitfalls

- **Wrong unit names** from generic web advice (`hermes.service`, `hermes-agent.service`) — on this host always check:
  `systemctl --user list-unit-files --type=service | grep hermes`
- **`malformed-backup-*` is not guaranteed good** — always `PRAGMA integrity_check` before trusting it.
- **Doctor `--fix` can fail** while still leaving useful backups; failed_attempts live in `state.db.repair-attempts.json`.
- **FTS-only rebuild** may report success then still fail writes; treat repeated FTS errors as full-DB risk.
- **Do not delete** `state.db.broken` / large backups until とーや confirms stability (hundreds of MB; forensic value).
- npm vulnerabilities under web/ui-tui reported by doctor are **usually non-blocking** for gateway runtime; do not confuse them with state.db failure.
- Sakana `fugu` credit exhaustion is unrelated: gateway may still answer via **fallback** (e.g. xAI grok) while default model 429s.
- **Token cost estimate ≠ recovery.** Do not stop writers or move DB files for a budget question — read-only only (see below).

## Session token → budget estimate (read-only)

When the ask is “$N でこの環境どれくらい？” / “過去セッション的にどう？” / model switch cost:

1. **Read-only** open `$HERMES_HOME/state.db` (on this home: `/opt/data/state.db`). Prefer `execute_code` + `sqlite3.connect('file:…?mode=ro', uri=True)`. A bare `terminal` path that looks gateway-adjacent can be blocked inside the live gateway — do not escalate to restart.
2. Tables: `sessions` (per-session totals), `session_model_usage` (per-model breakdown). Useful columns: `input_tokens`, `output_tokens`, `cache_read_tokens`, `reasoning_tokens`, `api_call_count`, `source`, `started_at` / `last_activity_at` (**Unix epoch float**, not ISO).
3. **Cache accounting:** here `cache_read_tokens` often **exceeds** `input_tokens`. Treat them as **separate billable piles** (fresh input @ input rate + cache @ cache rate + output(+reasoning) @ output rate). Do **not** `min(cache, input)` unless you have proof input already includes cache.
4. Apply target model rates (e.g. Namazu: in $0.95 / out $4.00 / cache $0.15 per 1M; Fugu Ultra: in $5 / out $30 / cache $0.50). Report **range**: full Hermes pace (discord+cron+kanban) vs human Discord-only vs automated-only.
5. Quote medians + a few heavy sessions; avoid one global “requests left” from light-chat assumptions.

Detail + query sketch: `references/session-token-cost-estimate.md`.

## Related

- skill `hermes-messaging-gateway-ops` (gateway day-to-day)
- skill `hermes-backup-and-recovery` (Notion-canonical home migrate/import)
- skill `notion-session-logging` (log the incident after recovery)
- Incident detail: `references/state-db-corruption-2026-08.md`
- Cost estimate: `references/session-token-cost-estimate.md`
