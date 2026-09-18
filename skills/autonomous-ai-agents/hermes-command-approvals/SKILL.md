---
name: hermes-command-approvals
description: "Use when Hermes shell approvals or allowlists need change."
version: 1.0.0
metadata:
  hermes:
    tags: [hermes, approvals, security, allowlist, discord, gateway]
---

# Hermes command approvals

Shell **approval cards** only gate **terminal** commands (not file writes). Fix permission-hell and owner policy here — not by setting `approvals.mode: off` unless とーや explicitly wants yolo.

## Defaults (とーや 2026-08-27 → ABC router)

**Buckets A/B/C only — no absolute-deny D.** Former hardline is **C** (card; とーや can approve).

| Bucket | Examples | Policy |
|--------|----------|--------|
| **A** | `execute_code`, heredoc/`-c`, `ntn`, everyday allowlisted work | No card (permanent allowlist) |
| **B** | `chmod`/`chown`/ACL | Owner auto-session; others card + ask とーや; **never permanent** |
| **C** | delete/wipe/`git clean` + **former hardline** (`rm -rf /`, mkfs, dd disk, shutdown, fork bomb) | **Always card** (とーや can still approve) |
| Unknown key | anything not in classify table | **C for this session**; log → ask とーや **after the task** to classify A/B/C |

- Card **Reason must be Japanese** (`approval-abc-router`)
- YOLO / `approvals.mode: off` → do **not** use to escape approval hell
- Plugin: `$HERMES_HOME/plugins/approval-abc-router/` (+ keep `permission-owner-gate` as B belt)

Owner Discord user id: `1239720373600256118`  
(`platforms.discord.home_channel.user_id`)

## Why cards suddenly appear

Often **not** the surface tool the user named, and **not always delete**:

### A) Shell shape on `terminal`

- `python3 <<'PY' ...` → `script execution via heredoc`
- `python3 -c '...'` / `bash -c` → `-e/-c` / shell `-c` patterns
- Fix: allowlist those **pattern keys** (and `ntn *` if needed)

### B) Whole-script gate on `execute_code` (common false “delete?”)

Gateway/ask surfaces run `check_execute_code_guard` **before** the sandbox starts.

- Pattern key: **`execute_code`** (separate from terminal heredoc/`-c`)
- Card text looks like: `execute_code script execution. The script can spawn subprocesses...`
- User may say “削除でもないのに承認” — check the ⚠️ first line; if it says `execute_code`, this is the gate
- Terminal allowlist of `script execution via heredoc` does **not** cover this key
- とーや default (non-delete OK): put **`execute_code`** on permanent `command_allowlist`
- Verify: `is_approved(session_key, "execute_code")` and/or that key ∈ `load_permanent_allowlist()` — do **not** call `check_execute_code_guard` while unapproved in a live Discord turn (it opens another card and can hang the agent ~timeout)

Allowlist **pattern keys** (canonical descriptions) and/or legacy regex aliases. Verify with `detect_dangerous_command` + `check_all_command_guards` for terminal; for code tool use the `execute_code` key above.

## Permanent allowlist (global)

- Store: `config.yaml` → `command_allowlist` (via `save_permanent_allowlist` / `hermes config`)
- Scope: **all sessions / all users** — cannot express “only とーや”
- Prefer canonical **pattern keys** from `DANGEROUS_PATTERNS` descriptions; keep legacy aliases if already present
- After bulk edits: confirm delete keys are **absent**, desired keys present, empty `''` entries removed

## Per-user policy (permission changes)

`command_allowlist` cannot do per-user. Use:

1. Plugin: `$HERMES_HOME/plugins/permission-owner-gate/` (enabled in `plugins.enabled`)
2. Config:
   - `approvals.permission_policy.owner_user_ids`
   - `approvals.permission_policy.session_only: true`
   - `platforms.discord.require_admin_for_exec_approval: true`
   - `platforms.discord.allow_admin_from: [<owner id>]`  
     → only とーや can click exec-approval buttons
3. Identity at runtime: `HERMES_SESSION_USER_ID` via `gateway.session_context`

Behavior:

- Owner (or local CLI with `cli_is_owner`) → `approve_session` on permission pattern keys before guards
- Non-owner → normal card; agent must **ask とーや**
- とーや approves → **session allow only**; permanent blocked for permission keys (guard on `approve_permanent` / `save_permanent_allowlist` + `post_approval_response` cleanup)

Details: `references/permission-owner-gate.md`

## Safe change sequence

1. List dangerous hits: `detect_dangerous_command(cmd)` / scan `DANGEROUS_PATTERNS_COMPILED`
2. Decide grain: permanent glob vs pattern key vs session-only vs owner plugin
3. Edit allowlist **or** plugin/config — avoid yolo
4. Unit-check guards as owner vs other user id (session vars)
5. **Gateway restart required** for plugin + most approval config  
   - **Never** restart gateway from inside a live gateway agent turn (self-SIGTERM). Tell とーや to run `hermes gateway restart` (or s6) outside.

## Diagnose a mystery card (fast)

1. Read the card’s first ⚠️ line → that is the **pattern key / description**
2. Logs: `gateway.log` / `agent.log` for `Discord button resolved` + nearby `Auxiliary approval` / `required approval (...)`
3. Classify:
   - delete/wipe → keep gated
   - permission (`chmod`/`chown`/ACL) → owner plugin, not global allow
   - `execute_code` → permanent allowlist key `execute_code` (if とーや wants non-delete free)
   - heredoc/`-c` → those terminal pattern keys
4. Fix the **exact key**; do not yolo the whole approvals mode

## Pitfalls

- Putting permission keys on **global** allowlist = everyone auto-runs them (violates owner-only)
- `approvals.mode: off` also skips delete gates
- Allowlisting only `ntn *` does not stop heredoc/`-c` **or** `execute_code` cards
- Terminal script allowlist ≠ `execute_code` tool allowlist (different keys)
- Re-probing with `check_execute_code_guard` while unapproved in Discord = more cards + long waits
- `plugins.enabled` is opt-in for **user** plugins; bundled platforms/backends still load via their own paths — do not “enable everything” blindly
- Mid-gateway `hermes gateway restart` / `s6-svc -r` from the agent is blocked or suicidal; schedule external restart
- Smart approval may auto-approve low risk; owner button gate is separate (`require_admin_for_exec_approval`)
- Config already listing a key is not enough until runtime `load_permanent_allowlist()` / process memory has it (gateway restart or `save_permanent_allowlist` path)

## Related

- `hermes-agent` → `references/security-privacy.md` (modes, yolo, reset allowlist)
- `hermes-messaging-gateway-ops` (Discord gateway ops)
- Plugin path: `/opt/data/plugins/approval-abc-router/` (+ `permission-owner-gate`)
- `references/approval-abc-router.md` — A/B/C router, JP reasons, hardline→C, unknown review
- `references/permission-owner-gate.md` — owner-only permission policy (B belt)
- `references/execute-code-gate.md` — whole-script approval key
