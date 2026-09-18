---
name: subscription-ops-recommend
description: "Use when managing personal subscriptions recommend-only."
version: 1.0.0
---

# Subscription ops (recommend-only)

とーやがサブスクを「丸投げ」するときの **class-level** 手順。  
**丸投げ = 検知・台帳・推奨**。契約/解約ボタンはとーや。

Law: Billing-Media-Act（従量嫌い・棚卸し）。解約代行・パスワード入力・承認なきクラウド browser はしない。

## Role split

| Who | Does | Does not |
|-----|------|----------|
| Agent | Gmail (etc.) crawl, ledger update, keep/cancel/check recs | Auto-cancel, card change, secret paste |
| User | Approve + perform cancel/subscribe/card fix | Hunt every receipt alone |

## When to load

- 「サブスク管理を丸投げ」「課金洗い出し」「解約候補」「定期課金の監視」
- Stripe / Link / Apple / Google Play / Netflix / Workspace の棚卸し
- 月次 cron の subscription tick
- 運用器だけ先に立てる（PRJ + hub + cron）

## Ops surfaces (Pi, re-search if drift)

| Layer | Where (2026-08) |
|-------|-----------------|
| Ledger canonical | Notion **PRJ-5** body table — `3b5fdf11-aa03-81a3-aa3b-d45ff0739787` (**archived 2026-08-25**) |
| Audit narrative | DB_Logs Session `yyyy-mm-dd Session: …棚卸しした` |
| Human tasks | DB_Action ↔ PRJ-5 (related Actions already アーカイブ) |
| Agent queue | Kanban `prj-5-subscriptions` (**board archived**) |
| Thin hub | Discord `#prj-5-subscriptions` (`1535164020909412403`, under `🗑️｜Archived`) |
| Monthly cron | removed with archive (`5bdfdfa7bf47` was monthly 1st 10:00 JST) |
| Mail skill detail | `gmail-subscription-audit` (queries/throttle; **user-owned** — adopt before patching; still OK for ad-hoc) |

Prior one-shot evidence (read-only): vault `Log/Session/workspace/2026-07-15_gmail-subscription-audit.md`.

## Modes

### A. Ops vessel only

1. Clarify scope with user if needed (vessel / full audit / both).
2. `project-launch-set`: DB_Project (完了条件・期間・role split in body) → read `PRJ-N` → Discord hub → DB_Action human cards → `@everyone` intro.
3. Kanban board `prj-{n}-subscriptions`; monthly cron self-contained.
4. Ledger table skeleton in Project body (empty rows OK).
5. Session log the launch topic.
6. Skip full Gmail crawl unless asked.

### B. Full / monthly audit

1. GWS:  
   `GAPI="$HOME/.hermes/hermes-agent/venv/bin/python $HOME/.hermes/skills/productivity/google-workspace/scripts/google_api.py"`  
   same venv `setup.py --check` → AUTHENTICATED.
2. Two-pass search (always both) — details in `gmail-subscription-audit` / its `references/gmail-query-recipes.md` if present:
   - `category:purchases newer_than:…`
   - fail/cancel + `from:(stripe.com OR … apple … payments-noreply@google.com OR paypal…)`
3. Throttle `gmail get` (~0.5s). Classify: 契約中 / 決済危機 / 終了寄り / 不明（盲点）.
4. Update **PRJ ledger** (diff). No new Notion schema properties.
5. User report (JP): crises first → changes → **推奨 max 7** (残す/切る/要確認 + 1-line evidence) → next human steps.
6. DB_Logs Session with method window + open actions. Not vault Log full dump (Pi).

## Multi-mailbox

| Source | Access |
|--------|--------|
| Gmail | Primary (OAuth) |
| iCloud Apple receipts | Often **blind** until forward / IMAP / monthly manual drop |
| Portal-only (some Nous/xAI) | Mark 不明; do not over-claim |

Apple path options (user chooses; no secrets in chat): (1) iCloud→Gmail forward (2) himalaya IMAP (3) monthly screenshot (4) cloud browser only with explicit OK.

## Project/Action pitfalls (launch)

- After N Actions, GET Project and require `Action.relation.length == N`. Dual may attach **only the first** child — re-PATCH every Action's `Project` if short.
- Live Action status prop is **`Status`** (English), not `ステータス`. Project uses `ステータス`.
- Create-time `markdown` on `ntn api v1/pages` is flaky → `ntn pages update --content` after create.
- Personal DB_Project db `3adfdf11-aa03-803d-b847-d2d69abc7985` / Action db `3adfdf11-aa03-80f6-8445-f59ff2a0017d` (re-search).

## Recommend heuristics

- Unused 30d + substitute exists → 切る候補
- Life/school/work critical → 残す
- Free trial end soon → decide-before-renew
- Payment fail loop → card fix **or** explicit end

## Related

- `gmail-subscription-audit` — Gmail query/throttle detail (adopt if needs patches)
- `google-workspace` / `gws-hermes-ops`
- `project-launch-set` / `notion-workspace-ops` / `session-log-notion`
- `hermes-deferred-handoffs` — Kanban/cron
- Billing-Media-Act §5

## References

- `references/ops-home.md` — living IDs for PRJ-5
