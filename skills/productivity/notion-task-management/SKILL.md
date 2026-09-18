---
name: notion-task-management
description: "Use when managing とーや's human tasks in DB_Action (Second Brain CODE: Capture/Organize/Distill=Zettelkasten/Express, Timeboxing forecast & timeframe, DB_Logs review)."
version: 2.2.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-8105-97be-f9a4e8422fab"
    notion_url: "https://app.notion.com/p/DB_Action-3adfdf11aa03810597bef9a4e8422fab"
    canonical: notion-db-law-skills
---

# 人のタスク（DB_Action）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-8105-97be-f9a4e8422fab` |
| URL | https://app.notion.com/p/DB_Action-3adfdf11aa03810597bef9a4e8422fab |
| カテゴリ | Notion |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-8105-97be-f9a4e8422fab
# or
ntn api v1/pages/3adfdf11-aa03-8105-97be-f9a4e8422fab/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。
5. Live schema のローカル補助: skill `notion-workspace-ops` → `references/db-action-live-schema.md`（2026-09-03 再読込）。
6. **クロー対話で完了した人タスク**の実行時間 = 当該 Hermes セッション時間（`started_at`〜完了時刻）。正本の「セッション時間」節に従う。
7. **セカンドブレイン＆Timeboxing自動化**: 会話からのキャプチャ・20〜30分分解、Forecast/Timeframe自動提案、Daily/Heartbeat時のDB_Logs集計レポートを順守。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `notion-workspace-ops`（`db-action-live-schema` / `project-action-pattern`）
- skill `notion-project-pages`
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
