---
name: notion-workspace-ops
description: "Use when writing Notion DBs via ntn as an agent."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-81ff-94f6-dea6ed4cf38e"
    notion_url: "https://app.notion.com/p/Notion-workspace-ops-ntn-3adfdf11aa0381ff94f6dea6ed4cf38e"
    canonical: notion-db-law-skills
---

# Notion workspace ops（ntn）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-81ff-94f6-dea6ed4cf38e` |
| URL | https://app.notion.com/p/Notion-workspace-ops-ntn-3adfdf11aa0381ff94f6dea6ed4cf38e |
| カテゴリ | Notion |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-81ff-94f6-dea6ed4cf38e
# or
ntn api v1/pages/3adfdf11-aa03-81ff-94f6-dea6ed4cf38e/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。
5. **Inbox 指定のメモ**は page 直下禁止。synced block への書き方・API制約は `references/page-body-write.md`（2026-08-13: `parent.block_id` の pages create は不可 → blocks children append）。
6. **新規DBそのものを作る**場合（私的セクション / page配下、schema、初期行）は `references/db-create-via-ntn.md`。`v1/databases` で DB + data_source + 初期ビューを一括作成。UIビュー（ギャラリー等）はAPI変更不可 → 人が1手切替する形で返す。
7. 現行トークンで `object_not_found` のページは **旧WS一時読み**を試す（`references/cross-workspace-read.md`）。書き込みは現行WSのみ。
8. 旧ハンズオン内容の PRJ 取り込みは `references/handson-project-ingest.md`。
9. **レシピ写真 → DB_Recipe** は `references/db-recipe-photo-ingest.md`（DB IDs は `references/toya-claw-db-map.md` Recipe 節）。free plan の画像添付は `page-body-write.md` の **single_part** 手順。
10. `ntn` 実体は PATH に無いことがある → `/opt/data/.local/lib/node_modules/ntn/bin/ntn`。`export NOTION_KEYRING=0`。
11. **完了クローズ**（DoD達成・「完了してアーカイブするイメージ」）→ `references/project-complete-closeout-ntn-discord.md`。status=`完了` + Discord は Archived へ。打ち切りは従来どおり archive レシピ。
12. **旅行後しおり**（Gmail/Calendar/X 突合）→ `references/trip-shiori-from-evidence.md`。Discord チャンネル PRJ ≠ 旅行 PRJ のことがある（先に DB_Project 特定）。
13. **コンテンツ AI Writing**（2026-08-29〜）→ クラス skill `toya-content-management` + `references/content-ai-writing-pipeline.md`。`🤖｜AI Writing` の本文は **DB_AI Writing**。書き方は Platform ごとの DB_Reference。予約投稿済み差分学習は Heartbeat。**学習後は `公開URL` 検証→Complete月**（`toya-content-management/scripts/content_publish_status_promote.py`。月 option は全件再送のみ・部分 PATCH 禁止）。
14. **第三者Notion同期 / Kindle HL**（BookNotion 系）→ 先に live DS + 実ページ **blocks JSON** を実測。howto は **既存ドメインhubの子ページ**。**正本=自前 Books×Highlights**；取得は **拡張なし** Pi Playwright→`read.amazon.co.jp/notebook`（`/opt/data/scripts/booknotion/`）。**識別子は Books.`ASIN` のみ**（notebook DOM 由来）。**ISBN web enrich 禁止**（Amazon 著者紹介が別本 ISBN を載せる → 誤本。実例: `B09QKLDMMH`『こうやって、考える。』が『日本語の作法』ISBN を拾った）。Cover は ASIN→CDN 決定処理のみ。**同期は script / `no_agent`（LLMトークン不要）** — hourly `booknotion-kindle-hourly-full-sync` + `hourly_sync.sh`。Clippings/GAS は補助のみ（主経路にしない）。login は Xvfb+noVNC handoff または host Desktop。**setup/backup** Hub 子 `3cefdf11-aa03-8164-90ba-f11cf75920dd`。詳細: `references/booknotion-z-schema-and-gas.md`。Z DB と自前DBの同時書き込み禁止。

## Related

- skill `toya-content-management`（コンテンツパイプライン入口）
- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
- local ref: `references/page-body-write.md`（DB行作成・body・**画像 single_part・edit後に画像壊れる罠・after 挿入**・hub子 title 空対策）
- local ref: `references/db-recipe-photo-ingest.md`（写真レシピ → DB_Recipe）
- local ref: `references/content-management-x-card.md` / `references/content-ai-writing-pipeline.md`（pitfalls 含む）
- local ref: `references/toya-claw-db-map.md`（DB_Recipe IDs・コンテンツ管理・DB_AI Writing・PF書き方 含む）
- local ref: `references/cross-workspace-read.md`
- local ref: `references/handson-project-ingest.md`
- local ref: `references/db-action-live-schema.md`（DB_Action プロパティ再読込 2026-09-03 · Start/End Time · Routine）
- local ref: `references/project-action-pattern.md`（作成・完了 closeout・**アーカイブ**）
- local ref: `references/project-complete-closeout-ntn-discord.md`（**完了** closeout + Discord Archived + Session）
- local ref: `references/trip-shiori-from-evidence.md`（旅行/イベントしおり・根拠突合）
- local ref: `references/project-archive-ntn-discord.md`（PRJ アーカイブ一式: Notion/Action/カンバン/cron/Discord/Session）
- local ref: `references/db-create-via-ntn.md`（新規DB作成 + 初期行 + ビュー制約）
- local ref: `references/booknotion-z-schema-and-gas.md`（自前 Books×Highlights + Playwright + **ASINのみ・ISBN enrich禁止** + no_agent hourly + setup/backup）
