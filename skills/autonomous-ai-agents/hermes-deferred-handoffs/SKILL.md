---
name: hermes-deferred-handoffs
description: "Use for agent Kanban/cron deferred work (not human DB_Action)."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-8152-bc87-c73b213ea32c"
    notion_url: "https://app.notion.com/p/Kanban-cron-handoff-3adfdf11aa038152bc87c73b213ea32c"
    canonical: notion-db-law-skills
---

# 延期・Kanban・cron handoff

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-8152-bc87-c73b213ea32c` |
| URL | https://app.notion.com/p/Kanban-cron-handoff-3adfdf11aa038152bc87c73b213ea32c |
| カテゴリ | Hermes運用 |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-8152-bc87-c73b213ea32c
# or
ntn api v1/pages/3adfdf11-aa03-8152-bc87-c73b213ea32c/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
