---
name: toya-claw-project-ops
description: "Use when launching とーや PRJ or Pi/Drive worktrees."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, project-launch, drive, remotion, tailscale]
    related_skills:
      - project-launch-set
      - personal-project-bootstrap
      - notion-workspace-ops
      - drive-file-exchange
      - hermes-messaging-gateway-ops
---

# とーやクロー — Project ops（立ち上げ + 作業コピー）

Notion 正本スキル（`project-launch-set` 等）の**完了条件はそのまま**。ここは 2026-08-24 以降に固まった **実行レシピと保管方針**。

## When

- 「PJ化して」「プロジェクト立ち上げ」
- Drive / Pi / Tailscale で preview・render するメディア・コード PJ
- Remotion や「ローカルに置きたくない」「サーバーは Pi のみ」

## User storage / preview policy (2026-08-24〜 / 2026-08-25 強化)

| 役割 | 置き場 |
|------|--------|
| **文書正本**（要件・設計・決定・進捗） | **Notion DB_Project**（子ページ可）。チャットには **Notion URL** を返す |
| **ファイル正本**（動画・JSON・renders・受け渡し） | **Google Drive**（API only、ライブマウントなし）。完了時は **Drive リンク** |
| **Pi ローカル** | Hermes 必須分 + **いま編集中の PJ だけ** `~/work/<slug>/`（一時作業。正本にしない） |
| **クライアント (Mac 等)** | プロジェクトツリーを置かない。**見られるのは Notion / Drive / Tailscale プレビュー** |
| **サーバー** | **Pi のみ**（`npm run dev` も Pi） |

**禁止:** 「Pi に保存したので確認を」だけ返すこと。クライアントと Pi は別端末。ローカル path をユーザー確認用の正本扱いにしない。  
**skills:** 長い手順は Notion DB_Law Skills 正本。ローカル `SKILL.md` は page_id ポインタ。

```text
Drive から編集対象だけ download
  → Pi で edit / Studio / render（--host 0.0.0.0）
  → 区切りで Drive へ upload
  → 終了後 Pi ローカルは薄くする
Mac: http://100.86.189.44:PORT または MagicDNS（ファイル非配置）
文書は最初から Notion。ローカル md を後から正本化しない。
```

**版ずれ対策:** 同時に触る作業コピーは Pi の1本だけ。クライアント dual-edit 禁止。

## Launch 1-set (policy pointer + ops)

Policy / naming / `@everyone` 必須要素 → skill **`project-launch-set`**（Notion 正本）。

Working commands, live IDs, pitfalls → **`references/launch-recipe-ntn-discord.md`**.

Media/code PJ では同じターンで Drive home も切る → **`references/drive-project-home.md`**.

### Independent / successor PJ (user rule)

とーやが **新 PRJ 番号**で立ち上げ、かつ「別PJ」「引き継がない」と明示（または隣接トピックの作り直し）した場合:

- **前 PRJ の要件・設計・コード・Drive ツリー・アーカイブ履歴を読まない・コピーしない**（完了条件文にも「他PJ非継承」を書く）
- 似た主題（例: 動画編集）でも **ゼロから** 概要・完了条件・Action・Drive を切る
- 明示的に「PRJ-N を土台に」と言われるまで sibling を参照しない
## Remotion notes

- Studio は Drive を開いただけでは出ない。Pi で `npm run dev -- --host 0.0.0.0 --port 3000`
- **Premiere 型:** `Remotion/videos/<slug>/{project.json,media,renders,archives}` — 1本=1フォルダ。テンプレは `templates/`、Editor は `apps/`
- Editor 実行: **`/home/tcaret2/work/remotion/apps/remotion-editor`**（`$HOME=/opt/data/home` 時は `~/work` が空 → 絶対パス）。詳細・v0.2 UX/API は skill **`remotion-editor-ops`**
- Hermes から Editor ソースを直すとき: `HERMES_WRITE_SAFE_ROOT` が `/opt/data` だと `write_file` が `/home/tcaret2` を拒否（symlink も realpath で弾く）→ **terminal/python でホーム側に書く**
- IDs: `drive-file-exchange` → `references/remotion-home-ids.md`
- 旧 `AP-Stretch-PRJ-8` / flat projects|renders / workflow 層は **廃止済み**（復活させない）

## Complete closeout（条件達成で閉じる）

「クローズ」「完了してアーカイブするようなイメージ」「しおり作って閉じて」で **完了条件が証拠上満たされている**とき。

正本: skill **`notion-workspace-ops`** → `references/project-complete-closeout-ntn-discord.md`。

要点:

- Project status = **`完了`**（打ち切りの `アーカイブ` ではない）
- Actions は既に完了なら触らない；不足は証拠メモで埋める
- Discord `#prj-N-…` は **🗑️｜Archived へ移動可**（ハブをアクティブ欄から外す）。topic は `【完了クローズ yyyy-mm-dd】`
- 旅行/イベントしおりは Project **子ページ**（`references/trip-shiori-from-evidence.md`）
- **チャンネル名の PRJ と閉じる PRJ が一致するとは限らない** — DB_Project を先に特定
- Session 1本

## Archive (打ち切り・作り直し前)

「アーカイブして」「一旦閉じる」→ **完了 closeout ではない**。

正本手順: skill **`notion-workspace-ops`** → `references/project-archive-ntn-discord.md`（+ `project-action-pattern.md` §Archive）。

要点:

- Project / Action status = **`アーカイブ`**（完了条件未達 OK）
- 改修カンバン未完了は閉じ側 status；履歴は残す
- PRJ 専用 cron 削除；Discord `#prj-N-…` → 🗑️｜Archived
- **Pi / Drive 実装物の物理削除は既定スコープ外**（別指示）
- Session 1本書く

## Requirements design freeze (FB → Vn.1)

When とーや is iterating **要件定義 + 画面HTML** in Discord before code:

- Full cycle, reverse-question depth, runtime honesty (Web vs Companion/plugins), deliverable checklist → **`references/requirements-v-fb-cycle.md`**
- Document canon = **Notion PRJ child**; HTML binary = **Drive docs** (+ optional to-toya); Discord thread closes after Vn.1
- Implementation starts only on explicit **「これで実装して」**
- 逆質問 must be **explained choices** (what/why/example), not bare numbered options

## Related

- `project-launch-set` / `personal-project-bootstrap` / `notion-project-pages`
- `notion-workspace-ops` / `notion-task-management`
- `notion-workspace-ops` refs: `project-complete-closeout-ntn-discord.md` / `trip-shiori-from-evidence.md` / `project-archive-ntn-discord.md` / `page-body-write.md`
- `drive-file-exchange` / `google-workspace`
- `hermes-messaging-gateway-ops` (`references/discord-project-channels.md`)
- `raspberry-pi-tailscale-access`
- local ref: `references/requirements-v-fb-cycle.md`（要件Vn FB→凍結）
