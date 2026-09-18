---
name: notion-project-pages
description: "Use when creating/updating DB_Project pages."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-8198-a1fe-e21473a22ab5"
    notion_url: "https://app.notion.com/p/DB_Project-3adfdf11aa038198a1fee21473a22ab5"
    canonical: notion-db-law-skills
---

# DB_Project ページ

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-8198-a1fe-e21473a22ab5` |
| URL | https://app.notion.com/p/DB_Project-3adfdf11aa038198a1fee21473a22ab5 |
| カテゴリ | Notion |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-8198-a1fe-e21473a22ab5
# or
ntn api v1/pages/3adfdf11-aa03-8198-a1fe-e21473a22ab5/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。
5. Live IDs/props は正本 + `notion-workspace-ops` → `references/toya-claw-db-map.md` / `project-action-pattern.md`。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `notion-task-management`（DB_Action）
- skill `notion-workspace-ops`
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
