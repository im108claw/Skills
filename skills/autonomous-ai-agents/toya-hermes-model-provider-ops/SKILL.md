---
name: toya-hermes-model-provider-ops
description: "Use when switching Hermes models or API keys on Pi."
version: 1.0.1
author: とーやクロー
license: MIT
metadata:
  hermes:
    tags: [toya-claw, hermes, model, provider, gemini, auth, 1password]
    related_skills: [hermes-agent]
---

# Hermes model / provider ops（とーや Pi）

Switch or diagnose the **default LLM** on the Pi Hermes home. Load `hermes-agent` for generic docs; this skill is the **local** procedure.

## When to Use

- 「モデル切替」「Grok→Gemini」「provider 変更」「APIキー登録」
- `hermes model` / `hermes config set model.*` / credential missing for a provider
- Wiring a cloud LLM key from 1Password into Hermes on this Pi

## Local facts (Pi)

| Item | Value |
|------|-------|
| `HERMES_HOME` | `/opt/data` |
| CLI | `/opt/hermes/.venv/bin/hermes` (often not on bare PATH) |
| Settings | `/opt/data/config.yaml` via **`hermes config set` only** (never hand-edit) |
| Secrets store | `/opt/data/auth.json` credential pool preferred |
| `.env` | **char device (~/dev/null)** — do **not** rely on `save_env_value` / writing `.env` |
| 1Password CLI | `op-with-sa` (`/opt/data/.local/bin/op-with-sa`) |
| Vault | `とーやクロー` |

## Steps — switch default model

1. `export PATH="/opt/hermes/.venv/bin:/opt/data/.local/bin:$PATH"; export HERMES_HOME=/opt/data`
2. Confirm current: `hermes config get model --json` and `hermes auth list`
3. Ensure provider credential (see below)
4. Set default (example Gemini free-tier-friendly Flash):

```bash
hermes config set model.provider gemini
hermes config set model.default gemini-3.6-flash
hermes config set model.aliases.gemini gemini/gemini-3.6-flash
# optional fallback chain (inline JSON/YAML list):
hermes config set fallback_providers '[{"provider":"gemini","model":"gemini-3.6-flash"},{"provider":"xai-oauth","model":"grok-4.5"}]'
```

5. Verify: `hermes config get model --json` + live API probe (below)
6. Tell user: **config applies to new sessions**. Current Discord/CLI session stays on the old model until `/model <alias>` or a new thread/session.

## Steps — API key from 1Password → auth pool

`.env` cannot hold keys here. Put keys in the **credential pool**:

1. List: `op-with-sa item list --format=json` (SA needs vault context on get)
2. Get item with **`--vault とーやクロー`** (service account requires `--vault` on `item get`)
3. Read concealed field (`credential` / 認証情報) into a temp file under `/opt/data/tmp/`, never print full secret
4. Register without echoing the key:

```bash
hermes auth add <provider> --type api-key --api-key "$KEY" --label 1password-<name>
```

5. Wipe temp files immediately
6. Confirm: `hermes auth list` shows the entry

Known item: **Gemini API Key** in vault `とーやクロー` (field id `credential`).

`hermes status` 「API Keys」 rows only inspect **env vars**. Pool credentials can work even when that row is ✗ — trust `hermes auth list` + a live call.

## Gemini specifics

- Provider id: `gemini` (aliases: `google`, `google-ai-studio`)
- Env names if ever available: `GOOGLE_API_KEY` / `GEMINI_API_KEY`
- Curated defaults include `gemini-3.6-flash`, `gemini-3.1-pro-preview`, …
- User preference when free-tier / cost-sensitive: prefer **Flash** (`gemini-3.6-flash`) over Pro
- Thinking models spend tokens on `thoughtsTokenCount`; probes need **enough `maxOutputTokens`** (e.g. 256+) or finishReason=`MAX_TOKENS` with empty text

Live check pattern (no key in logs):

```python
from agent.credential_pool import load_pool
key = load_pool("gemini").peek().access_token.strip()
# generateContent on models/gemini-3.6-flash with maxOutputTokens>=256
```

## Pitfalls

- **Do not** `hermes gateway restart` / stop / uninstall from inside the gateway agent session — terminal blocks it (SIGTERM would kill the job).
- **Do not** hand-edit `config.yaml`.
- **Do not** pipe secrets into shell history-visible `echo`; use Python `subprocess` with argv or env and redact stdout.
- Interactive `hermes model` picker needs a TTY — prefer non-interactive `hermes config set` + `hermes auth add` from Discord agents.
- xAI OAuth remains valid as fallback; clearing it is separate from Gemini switch.
- Gateway-inside terminal may false-positive-block commands that only *mention* restart; split probes and avoid those phrases.

## Related

- skill `hermes-agent` (generic providers/config docs via its references)
- Docs: https://hermes-agent.nousresearch.com/docs/user-guide/configuring-models
- Local checklist: `references/gemini-switch-checklist.md`
