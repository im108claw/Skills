---
name: project-launch-set
description: "Use when launching a new project across Notion and Discord."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical, project-bootstrap]
    notion_page_id: "3b1fdf11-aa03-8163-8c5a-ea0ec5b23e2d"
    notion_url: "https://app.notion.com/p/1-3b1fdf11aa0381638c5aea0ec5b23e2d"
    canonical: notion-db-law-skills
    related_skills: [personal-project-bootstrap, notion-project-pages, notion-task-management, notion-workspace-ops, hermes-messaging-gateway-ops]
---

# プロジェクト立ち上げ手順（1セット）

**手順の正本は Notion（DB_Law → Skills → Notion）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3b1fdf11-aa03-8163-8c5a-ea0ec5b23e2d` |
| URL | https://app.notion.com/p/1-3b1fdf11aa0381638c5aea0ec5b23e2d |
| カテゴリ | Notion |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3b1fdf11-aa03-8163-8c5a-ea0ec5b23e2d
# or
ntn api v1/pages/3b1fdf11-aa03-8163-8c5a-ea0ec5b23e2d/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、一時フォールバックを探し、読めなかったことをユーザーに言う。

## Trigger

とーやが「プロジェクトを立ち上げる」「Projectを作成する」「PJを切る」と言ったとき。

## Related

- skill `personal-project-bootstrap`
- skill `notion-project-pages`
- skill `notion-task-management`
- skill `notion-workspace-ops`
- skill `hermes-messaging-gateway-ops`
