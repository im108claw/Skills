---
name: booknotion-kindle-sync
description: "Use when operating ClipsNest/BookNotion Kindle→Notion sync."
version: 1.1.0
author: toya-claw
license: MIT
metadata:
  hermes:
    tags: [clipsnest, booknotion, kindle, notion, toya-claw]
---

# ClipsNest（旧 BookNotion / ClipNest）Kindle → Notion sync

## When to Use

- Kindle notebook → Notion Books/Highlights の取得・同期・障害切り分け
- ハイライト**メモ／ノート**を Comment へ転記・検証するとき
- auth 切れ、diff 重複、cron `no_agent` 経路の確認
- ClipsNest スキーマ（Name / Comment / SourceKey / ASIN）や **Git 整形・デバイス移行**
- リポジトリ名 **ClipsNest**（private）。製品呼称は ClipsNest（旧: 独自 BookNotion / ClipNest）

自前 Books/Highlights 同期（Chrome 拡張なし）。LLM なし・script-first。

## Paths

| 項目 | パス |
|------|------|
| **実行正本（Git worktree）** | `/opt/data/work/ClipsNest/` |
| **GitHub** | private `im108claw/ClipsNest`（`main`） |
| **cron 入口** | `/opt/data/scripts/clipsnest/hourly_sync.sh` → worktree |
| **旧 scripts 入口** | `/opt/data/scripts/booknotion/{run,hourly_sync}.sh` → **forward only**（本体なし） |
| 状態 | `/opt/data/state/booknotion/`（env: `CLIPNEST_STATE` / 互換 `BOOKNOTION_STATE`） |
| Chromium profile（正本） | `…/chromium-profile/`（`CLIPNEST_PROFILE`） |
| auth snapshot | `…/auth_state.json`（成功時のみ） |
| diff index | `…/hl_index.json` |
| runner | worktree の `./run.sh` / `hourly_sync.sh` |
| cron job | `clipsnest-kindle-hourly-diff-sync`（id `6a24fb6c8746`）`no_agent` `0 * * * *` |

**2026-09-02 cutover:** live 実行は Git worktree。hourly は **先に `git pull --ff-only`** してから sync（別端末 push も次 tick で反映）。  
詳細: `references/clipsnest-repo.md`

## Notion IDs

- Books DB `0dbdfef3-2584-4349-a3c2-65024fa6da1b` / DS `e62e005f-1d81-4b2d-870f-ee72881be7f2`
- Highlights DB `f26b65db-bff7-4356-a16e-4c092e252123` / DS `8f759407-d5d3-4a4c-b90c-e2d4f3a983ec`
- **旧 BookNotion Z DB と同時書き込み禁止**

## Schema rules（必須）

| 対象 | ルール |
|------|--------|
| HL `Name` | ハイライト本文のみ |
| HL `Comment` | Kindle notebook のハイライトメモ（ノート） |
| HL `SourceKey` | `kid:{highlight-…}` 優先。fallback `txt:{ASIN}:{sha1[:16]}` |
| Books `ASIN` | notebook 由来。ISBN enrich なし |

### メモ／Comment

1. scrape は `Highlight.note` を **本文と分離**して持つ（`〔メモ〕` を Name に埋め込まない）
2. create/update で `Comment` に書く
3. **空の Kindle メモで既存 Comment を消さない**（Notion 手入力コメント保護）
4. **diff でも既知 HL の note は sync**（index skip しても Comment 更新）
5. レガシー Name 内 `〔メモ〕…` は sync 時に Comment へ migrate し Name を本文だけにする

## Commands

```bash
# 運用（Git worktree = 正本）
cd /opt/data/work/ClipsNest
export CLIPNEST_STATE=/opt/data/state/booknotion   # optional; auto-detects on Pi
./run.sh status
./run.sh login                 # 初回 / ログイン切れ
./run.sh sync --diff           # cron と同じ
./run.sh sync --diff --max-books 2
./run.sh sync --from-json /opt/data/state/booknotion/last_fetch.json --diff
./run.sh sync --full
./run.sh reindex
./run.sh dedupe --dry-run
```

Cron: `clipsnest/hourly_sync.sh` → worktree `hourly_sync.sh` → **`git pull --ff-only`** → `sync --diff`、**`no_agent: true`**（AI 不使用）。  
pull 認証: `GITHUB_TOKEN` / `GH_TOKEN` または `/opt/data/secrets/github-personal-access-token`（remote URL に token を残さない）。失敗時 exit 3 で cron 通知。

## Auth durability

- login と scrape は **同一** `launch_persistent_context(chromium-profile)` + 同一 UA/viewport/locale
- `auth_state.json` は notebook ライブラリ確認後の snapshot のみ。sign-in 画面では上書きしない
- 切れ時: `./run.sh login` または `start_login_handoff.sh`（noVNC）

## Verification checklist（メモ機能）

1. `last_fetch.json` で `note` が分離され、`text` に `〔メモ〕` が無い
2. Notion HL: `Comment` = メモ、`Name` = 本文のみ
3. Comment を空にして再 `sync --diff` → `hl_notes_updated≥1` で復元
4. 手入力 Comment（辞書メモ等）が空メモ同期で消えていない

## Pitfalls

- Name にメモを混ぜると SourceKey/text-key と Title マッチが壊れる
- diff short-circuit だけで note 更新を省略すると Comment が古いまま
- probe HTML の `note-` + `aok-hidden` は空プレースホルダ（本文扱いしない）
- Playwright DOM 変更時は `notebook.py` の evaluate セレクタを直す
- **repo 名は `ClipsNest`（ユーザー訂正）。`ClipLess` / 単独 `ClipNest` で repo を作らない**
- **secrets を git に入れない**: `auth_state.json` / `chromium-profile/` / Notion token / `*.pem` / `.venv`
- GitHub push: PAT は `/opt/data/secrets/github-personal-access-token`（1Password 由来）。`gh` 未導入でも API+HTTPS 可。remote URL に token を残さない
- cutover 後も **旧 py を scripts/booknotion に直書きしない**（forward only）。必ず worktree を編集して push
- `.venv` は worktree から live 旧 venv へ symlink 可。壊したら worktree で `python3 -m venv .venv && pip i -r requirements.txt && playwright install chromium`

## Related

- Git 正本 README: `/opt/data/work/ClipsNest/README.md`
- live（暫定）: `/opt/data/scripts/booknotion/`
- skill `hermes-cron-operations`（no_agent / script-first）
- skill `notion`（ntn / data_sources）
- skill `github-repo-management` / `github-auth`
- references: `references/comment-memo-sync.md`, `references/clipsnest-repo.md`
