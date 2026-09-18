---
name: gmail-subscription-audit
description: "Inventory personal subscriptions and payment failures from Gmail (and related billing mail). Use when the user wants to cancel unused subs, list active memberships, find services that lapsed after failed payments, or crawl category:purchases / receipts / invoices."
version: 1.0.0
related_skills:
  - google-workspace
  - obsidian
---

# Gmail subscription audit

Class-level workflow: discover **active**, **trial**, and **failed/cancelled** subscriptions from Gmail, then write a durable vault asset log. Does **not** cancel anything for the user (no password/browser billing logins unless they explicitly OK cloud browser).

## When to load

- 「サブスク解約」「契約してるだけ」「課金洗い出し」
- Gmail クロール / `category:purchases` / 決済失敗で落ちたサービス
- Inventory of Netflix, Workspace, Stripe receipts, Google Play, Apple, etc.

## Prerequisites

1. Load `google-workspace` patterns:  
   `GAPI="$HOME/.hermes/hermes-agent/venv/bin/python $HOME/.hermes/skills/productivity/google-workspace/scripts/google_api.py"`
2. `$GSETUP --check` → `AUTHENTICATED` (use hermes-backup-and-recovery / GWS recovery if not).
3. User vault asset logs:  
   `Notion DB_Logs Session + PRJ-5 台帳（vault Log へは書かない・Pi cutover）`  
   Pi: 正本は Notion PRJ-5 台帳。監査詳細は DB_Logs Session。

Daily AI reports usually **avoid Gmail** by policy; this skill is an **explicit** Gmail task exception.

## Workflow (do not skip)

### 1. Auth + scope

```bash
$HOME/.hermes/hermes-agent/venv/bin/python $HOME/.hermes/skills/productivity/google-workspace/scripts/setup.py --check
```

### 2. Two-pass search (always both)

**Pass A — Purchases category (high recall for receipts)**

```text
category:purchases newer_than:365d
category:purchases (subscription OR membership OR Premium OR Pro OR 定期 OR receipt OR invoice OR ご注文明細 OR 解約 OR payment)
```

**Pass B — Fail / cancel / billing keywords** (EN + JP; body search helps JP)

```text
subject:(payment failed OR unsuccessful OR canceled OR cancelled OR paused OR downgraded OR 解約 OR お支払い方法 OR 自動更新失敗)
from:(stripe.com OR googleplay-noreply@google.com OR noreply-purchases@youtube.com OR email.apple.com OR paddle.com OR payments-noreply@google.com)
```

Also run service clusters: X/Sakana Stripe, Anthropic, Notion billing, Workspace, Netflix members, Weverse, Discord Nitro, Bitfan, CAMPFIRE, Amazon Prime, Rakuten Mobile.

Cap: Hermes `gmail search --max` is often **50**. Paginate with tighter `newer_than:` / `older_than:` or `from:` slices. Deduplicate by message `id`.

### 3. Fetch bodies with throttle

- Prioritize: receipt, invoice, cancel, fail, trial ends, ご注文明細, 定期購入.
- Sleep **~0.5s** between `gmail get` calls. Avoid blasting 90+ gets in one tight loop (partial null bodies / misses).
- Strip HTML: regex drop tags, collapse whitespace; extract `¥/￥/$` and last-4 card digits.

### 4. Classify each service

| Bucket | Criteria |
|--------|----------|
| **Active-ish** | Recent successful receipt / renewal notice / members content still arriving |
| **Payment crisis** | Failures + still subscribed + deadline (e.g. Workspace stop date) |
| **Failed → cancelled** | Fail mail then cancel/paused/downgraded with no later success |
| **Trial / marketing only** | Trial ending, welcome without charge trail |
| **Unknown (Gmail blind)** | No mail (other address, app-only, SuperGrok/Nous sometimes) — say so |

Correlate **card last4** across services (e.g. same debit failing Claude + Manus + Workspace).

### 5. Deliverables

1. **User-facing table** (active / failed / cancel candidates) — concise JP/EN as user language.
2. **Vault asset log** (full):  
   DB_Logs Session `yyyy-mm-dd Session: Gmailサブスクを棚卸しした` + PRJ-5 台帳更新  
   Include method limits, query window, open actions.
3. Optional `/tmp/*.json` for session debug only — not the durable store.

### 6. Cancel guidance only

- Link manage portals (Google Play, Apple, admin.google.com, Stripe customer, service settings).
- Do **not** use pay-as-you-use cloud browser without explicit OK (user billing policy).

## Pitfalls

| Pitfall | What to do |
|---------|------------|
| JP `subject:(サブスク…)` returns empty | Search body tokens or `category:purchases`; many JP mails use EN subjects from Stripe |
| Stripe "Your receipt" body empty | Keep as existence proof; amount may be in PDF/HTML only — note "amount unknown" |
| Keyword-only miss (Speak, Workspace, Netflix) | Always run **category:purchases** pass |
| `category:purchases` polluted by 楽天ペイ one-offs | Separate "true sub" vs "one-shot payment / Suica charge" |
| Hermes/Nous/xAI silent | Check `hermes status` / portals; mark Gmail-blind |
| Rate limit / null gets | Throttle; re-fetch miss IDs; prefer script file over nested `python -c` + pipes |
| Shell f-string / quote hell | Write `/tmp/audit_*.py` with subprocess list args (no nested escaped f-strings) |
| Terminal UX | Prefer short commands / script files over giant one-liners (user preference) |

## Decision prompts for "just contracted for status"

For each active-ish service: used in 30d? breaks tomorrow if gone? duplicate of another sub?  
Recommend cancel if 2+ "yes" on unused/duplicate.

## References

- `references/gmail-query-recipes.md` — copy-paste query packs and sender map
- `google-workspace` → `references/gmail-search-syntax.md` for operators

## Related

- `google-workspace` — auth + `gmail search/get`
- `obsidian` — vault write path
- User rejects pay-as-you-use tools; prefer Gmail API over cloud browser for this class of task
