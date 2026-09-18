---
name: hermes-cron-operations
description: "Use when configuring or troubleshooting Hermes cron jobs."
version: 1.1.0
created_by: agent
metadata:
  hermes:
    tags: [hermes, cron, scheduling, delivery, operations]
---

# Hermes cron operations

Use this skill when creating, updating, removing, or diagnosing Hermes `cronjob` jobs, especially when the work affects Discord/channel delivery, job prompts, `context_from`, model/provider pins, or fallback behavior.

## Operating principles

1. **Read the existing job first.** Use `cronjob(action="list")` and, when changing prompt-sensitive jobs, inspect the persisted job record so you know the current `prompt`, `deliver`, `skills`, `schedule`, `model`, `provider`, `workdir`, `script`, `monitor_script`, `no_agent`, and `context_from` before changing anything.
2. **Preserve fields explicitly.** `cronjob(action="update")` can replace fields. If you pass an empty prompt or empty lists accidentally, you may clear job state. When updating one field, carry forward the existing prompt/skills/schedule/workdir/script unless the user explicitly wants them changed.
3. **Prefer changing the existing job over adding another job.** If the user's intent is to change where/how an existing cron reports, update that job's `deliver` and final-output instructions. Do not create a second notification job unless the user explicitly asks for a separate cadence or separate post-processing step.
4. **Separate channel design from implementation.** Before adding jobs, infer why channels are being separated. High-frequency daily jobs may belong in their own channel so intermittent or important cron alerts do not get buried.
5. **Remove accidental duplicate jobs.** If a notification job was created but the right design is a single creating/reporting job, remove the notification job and verify only the intended job remains.
6. **Verify with real state.** After edits, run `cronjob(action="list")` and inspect the persisted job data for job count, destination, next run, prompt length, and any expected marker text.
7. **Script-first for recurring work (とーや 2026-08-25).** Anything programmable in periodic/ops jobs must be saved as a program/script under the Hermes scripts dir and wired from the job. Goal: cut tokens and raise speed. See **Script-first recurring jobs** below and Procedure Code §「定期実行のスクリプト優先」。

## Script-first recurring jobs

とーや方針（2026-08-25）: 定期実行などでプログラム化できるものは **全て** スクリプト化して保存する。トークン削減・速度向上が目的。

### Choose execution mode

| Mode | When | Tokens |
|------|------|--------|
| `no_agent=true` + `script` | Deterministic check/collect/alert text | None (stdout delivered; empty = silent) |
| `monitor_script` / `monitor_url` | Only act on change; stable output required | Agent only on change |
| `script` + agent prompt | Collect/normalize in script; judge/write in agent | Lower input |
| agent-only | Browser judgment, prose drafting, secretary decisions not yet automated | Full |

Do **not** force `no_agent` onto jobs that need real judgment (Daily prose, UNIPA triage, Heartbeat secretary). Still extract collection/diff/state into scripts.

### Save location

- Primary: `~/.hermes/scripts/` (on this home often `/home/tcaret2/.hermes/scripts/` or `/opt/data/scripts` via cron relative `~/./scripts/`)
- Prompt templates that remain agent-driven: same dir as `*_prompt.txt`
- Name: `<domain>_<verb>.sh` or `.py` (e.g. `unipa_resume_outage.sh`, `gateway_health_check.sh`)
- Make executable; document purpose + stdout contract in header comments
- Point the job at the file via `script` / `monitor_script`; do not leave the logic only inside a long chat prompt

### Implementation checklist (every new/updated cron)

1. Split steps into **deterministic** vs **judgment**.
2. Write deterministic steps to `scripts/` and commit path into the job + skill.
3. Prefer silence on no-op (`no_agent` empty stdout, or monitor unchanged, or final `[SILENT]` only when the job contract allows).
4. Keep secrets out of scripts/stdout.
5. After create/update: `cronjob list` + one dry-run of the script if safe.

### Existing active jobs (scriptability snapshot 2026-08-25)

| Job | Mode now | Script path |
|-----|----------|-------------|
| `fit-unipa-board-monitor` | agent + browser | keep agent; extract state/diff helpers over time |
| `obsidian-ai-daily-report-…` | agent writing | collect evidence via script later; keep prose agent |
| `toya-claw-heartbeat-next` | agent secretary | prompt `toya_claw_heartbeat_prompt.txt`; **chain must always leave 1 future job**; prefer CLI `--model/--provider` pin against drift_skip |
| `toya-claw-heartbeat-chain-guard` | **no_agent** backstop | `heartbeat_chain_guard.py` — if zero future HB, alert + recreate (every ~3h, not 30m) |
| `prj7-x-autonomous-reflection-next` | agent PDCA | keep agent; reuse PRJ-7 native schedule scripts |
| outage pause/resume | **already** `no_agent` script | `unipa_pause_outage.sh` / `unipa_resume_outage.sh` |
| `clipsnest-kindle-hourly-diff-sync` (was booknotion-…) | **no_agent** **diff** Kindle→Notion | `clipsnest/hourly_sync.sh` → **`/opt/data/work/ClipsNest`** · `hl_index.json` · `kid:{highlight-id}` · `0 * * * *` · logs `/opt/data/state/booknotion/logs/` · auth=`chromium-profile` · cutover 2026-09-02 |

When touching any of the above, advance script coverage rather than growing the prompt only.

### Pitfall: BookNotion auth dies “every other hour” (2026-09-02)

**Root cause:** login saved `auth_state.json` with custom UA/viewport; hourly scrape opened a *bare* Playwright context from that JSON and rewrote cookies under a different fingerprint. Amazon invalidated the session → next tick hit sign-in (`max_auth_age`-ish).

**Fix (in `scripts/booknotion/notebook.py`):**
- login **and** scrape use the same `launch_persistent_context(user_data_dir=…/chromium-profile)` + identical UA/locale/viewport/args
- `auth_state.json` = success-only atomic snapshot (never overwrite on sign-in wall)
- Hard login wall = signin/mfa URL after settle — not a lone `input[name=email]` on any page
- Seed empty profile from existing `auth_state.json` via `add_cookies` (persistent context does **not** accept `storage_state=`)

**Ops:** login via `start_login_handoff.sh` (noVNC). Use **http** + `encrypt=0`; keep `scripts/booknotion/self.pem` for browsers that try wss. Detail: `references/booknotion-auth-durability.md`

### Mass pause until time T + auto-resume (2026-09-02)

When とーや says pause **all** crons until a clock time (e.g. noon after Amazon re-login):

1. `cronjob list` → collect every `enabled=true` job id (not completed).
2. Write inventory JSON under `/opt/data/cron/pause_until_YYYYMMDD_HHMM.json` (`jobs[]`, `resume_at`, `reason`).
3. `cronjob(action=pause)` each id.
4. Create **one** `no_agent` resume job: `script=cron_resume_after_booknotion_pause.sh` (or generic resume script), `schedule=ISO at T`, `repeat=1`, deliver cron channel.
5. Resume script: `HERMES_HOME=/opt/data` + `/opt/hermes/.venv/bin/hermes cron resume <id>` for each inventory id; mark `resumed_at`; empty/skip if already resumed.
6. If user corrects the date (“明朝と言ったが日付跨ぎ” → **today** noon): `hermes cron edit <resume_job_id> --schedule "YYYY-MM-DDTHH:MM:00+09:00"` — do not create a second resume job.
7. Note: one-shots whose `next_run_at` already passed while paused may fire on resume or need `--at` re-arm — call out in the user reply.

CLI path when tool lacks edit: `export HERMES_HOME=/opt/data; /opt/hermes/.venv/bin/hermes cron …`

### Pitfall: cron create blocked by script content scan (2026-09-01)

`cronjob(action=create)` may refuse with *gateway lifecycle / launchctl / systemd supervision* if the **script file tree** (or a `source`d path) contains strings like `systemctl`, `hermes gateway restart`, or similar — even when the job itself is harmless `no_agent` sync.

- Keep **install/systemd docs** out of the executable path the job runs (ok under `scripts/booknotion/systemd/`; do not `source` install scripts).
- Prefer thin wrappers: `hourly_sync.sh` → `run.sh` → python; avoid `source /opt/data/.env` in cron scripts if `.env` embeds gateway lifecycle notes (token load belongs in the Python/config layer).
- If create is blocked: strip the script, re-create; do not invent a second agent-only job.

Details + checklist: `references/script-first-recurring-jobs.md`  
Inventory helper: `/opt/data/scripts/cron_jobs_scriptability_report.py`（=`~/.hermes/scripts/` 同一）  
Law: `Law/codes/Procedure-Code.md` §定期実行のスクリプト優先

## Delivery routing pattern

For Discord cron routing:

- Use explicit `deliver="discord:<channel_id>"` when the destination matters.
- For a job that creates an artifact and should notify about it, keep creation and final notification in the same cron when a single cadence is enough.
- Shape the job's final response for the destination channel. For Daily Log-style output, the final response should be link + up to 3 day-level action principles (意識する行動指針), not task-list rehash, long logs, or raw creation traces.

## Prompt update checklist

Before updating a prompt-bearing cron:

1. Save or reconstruct the current prompt.
2. Modify only the intended sections.
3. Include the final-output contract in the prompt when the Discord delivery content matters.
4. Confirm the updated persisted prompt is non-empty and contains the expected marker phrase.

## Model/provider fallback notes

Hermes cron jobs use the global fallback chain when available:

- `fallback_providers` in `config.yaml` is passed into cron agent runs.
- Cron scheduler also tries fallback entries during primary provider auth/provider-resolution failure.
- Runtime API-call failover can switch to fallback providers/models for failover-classified errors such as rate limits, billing/quota problems, model-not-found, server/provider failures, or timeouts after retries.
- Fallback does not fix every failure: content policy refusals, context/payload size handling, cron hard timeouts, or `no_agent=True` script-only jobs follow their own paths.
- Fallback changes the model/provider used to execute; it does not change the job's `deliver` destination.

## User-specific routing lesson

When とーや asks to move Daily Log output, first consider channel-noise intent:

- Daily jobs run every day and can bury other cron output.
- Daily creation/reporting should usually be one cron job delivering to the Daily Log channel, with the final response shaped as `Notion link + concrete next actions`.
- Other cron jobs should use the general cronjob channel unless the user specifies otherwise.

See `references/daily-log-routing-correction.md` for the session correction that motivated this rule.

## Cron agent constraints (runtime)

- **`execute_code` is blocked in cron** (`approvals.cron_mode`). Use `terminal` / file tools / normal tool calls. Do not design scheduled jobs that depend on `execute_code`.
- Final response is auto-delivered. Do **not** call `send_message` to re-deliver. Use `[SILENT]` only when the job prompt allows silence and there is nothing to report.
- Keep prompts **self-contained** (no chat context). Attach skills with `--skill` (repeatable).
- **Do not `remove` the job id that is currently running.** Dedup only *future* sibling one-shots. Self-delete mid-run can mark the fire `FAILED` and drop Discord auto-delivery even when the final short report was written (2026-09-02 Heartbeat manual run).

### Manual `cronjob(action=run)` (とーや 2026-09-02)

1. Fire returns immediately (`executed: true`, background). If とーや asked only for setup ack, reply **できた/できなかった** and stop waiting.
2. On `status=error`, still open the cron session (`session_search` / `cron_<jobid>_*`) — final assistant text may be complete.
3. If the target channel has no message, redeliver the final text (Discord REST with gateway process `DISCORD_BOT_TOKEN`; never log the token). Detail: `references/manual-cron-run-delivery.md`.
4. After a consumed one-shot, verify exactly one future chain job remains.

## Project reflection one-shot (not forever)

For Task-Assignment-style loops (e.g. PRJ-2 ROOM):

1. Finish this unit of work in the current one-shot.
2. Schedule **exactly one** next job — `hermes cron create --repeat 1` (or once-at timestamp). **No forever heartbeat.**
3. Prefer `--deliver origin` when the next report should land on the parent job's thread/channel.
4. Name clearly (e.g. `prj2-room-reflection-next`) and embed the full self-contained prompt.
5. After create, `hermes cron list` and report the **new job id** + next run in the final response.
6. The *current* job may still show as running until this invocation ends; the new job should be the only future one-shot for that loop.
7. In-Review / human-wait: schedule **2–4 days** later for a light check, not daily spam.

```bash
hermes cron create \
  --name "prj2-room-reflection-next" \
  --deliver origin \
  --repeat 1 \
  --skill rakuten-room-ops \
  --skill notion-workspace-ops \
  --skill session-log-notion \
  --workdir /home/tcaret2 \
  "2026-08-10 20:00" \
  "<full self-contained prompt>"
```

Details: `references/project-reflection-oneshot.md`.

For **first-time project setup** that pairs DB_Project + Discord project channel + human Action + project Kanban board + first reflection one-shot, use `references/project-board-and-reflection-launch.md` (search past sessions/Notion first; report found/related/not-found; no forever jobs).

For human-review-gated loops where the prepared artifact is already `In Review`, use `references/human-review-reflection-loop.md`: check only the gate signal, avoid starting extra work while とーや is still the blocker, and return exactly `[SILENT]` when the prompt allows silence and nothing changed.

## Kanban while doing agent project work

```bash
hermes kanban boards switch <board-slug>   # e.g. prj-2-rakuten-room
hermes kanban create --created-by toya-claw --assignee default \
  --idempotency-key "<stable-key>" \
  --body "<done condition>" \
  "【PRJ-N】日本語タイトル"
hermes kanban comment <task_id> --author toya-claw "<outcome + URLs>"
hermes kanban complete <task_id> --result "<short>" --summary "<handoff>"
```

- Subcommand is **`create`**, not `add`.
- Prefer **`--board <slug>`** on each command when multiple boards exist; do not assume `boards current` stays put.
- A ready task may flip to `running` via dispatcher even if this session already owns the work — still complete/comment when done.
- Human research-fill stays out of DB_Action when the project says agent owns research (ROOM: とーや=投稿のみ).

### Human-gated agent work (needs_input)

When the next agent step is blocked on とーや's decision (名義・方針・課金承認など):

```bash
# if dispatcher already claimed it:
hermes kanban --board <slug> reclaim <task_id>
hermes kanban --board <slug> block <task_id> --kind needs_input "とーやの初期仕様決定待ち（…）"
# reason is a positional arg after task_id (NOT --reason)
hermes kanban --board <slug> comment <task_id> "blocked: <one-line why>"
```

- There is **no** `kanban update --status`. Use `block` / `unblock` / `schedule` / `request-review`.
- `block --kind needs_input` (or `capability`) = human gate. `dependency` waits on parent tasks, not people.
- Do **not** leave a human-gated task as `running`; reclaim first if needed, then block.
- Keep the human decision itself in **DB_Action**; Kanban only tracks the agent unit waiting on that decision.

### Project board + project reflection pair

When launching a multi-session agent project (after Notion PRJ + Discord channel exist):

1. `hermes kanban boards create <slug> --name 'PRJ-N …' --description '…'`
2. Create **one** blocked/ready design-or-next task on that board (1 theme).
3. Schedule **one** named reflection one-shot (`prjN-…-reflection-next`, `--repeat 1`) with `deliver=discord:<project-channel-id>`.
4. Put board slug, channel id, job id, and human Action URL on the Project page「起動時の配置」.

Details: `references/project-board-and-reflection-launch.md`.
## ntn property PATCH (cron-safe)

Method flag is **`-X` / `--method`** (capital X). **`-x` fails.**

```bash
export NOTION_KEYRING=0
ntn pages edit <page_id> < /tmp/body.md
ntn api v1/pages/<page_id> -X PATCH -d '{
  "properties": {
    "ステータス": {"status": {"name": "👨‍💻｜In Review"}}
  }
}'
ntn pages get <page_id> | head -20
```

Use exact existing option strings only. Never add schema/options on shared content DBs.
