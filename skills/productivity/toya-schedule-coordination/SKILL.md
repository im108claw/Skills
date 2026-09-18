---
name: toya-schedule-coordination
description: "Use when drafting schedule replies; read Calendar first."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, schedule, calendar, google-workspace, notion-canonical]
    notion_page_id: "3b8fdf11-aa03-8128-96c0-cc092ec87b34"
    notion_url: "https://app.notion.com/p/3b8fdf11aa03812896c0cc092ec87b34"
    canonical: notion-db-law-skills
---

# 日程調整（候補出し）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3b8fdf11-aa03-8128-96c0-cc092ec87b34` |
| URL | https://app.notion.com/p/3b8fdf11aa03812896c0cc092ec87b34 |
| カテゴリ | Google / FIT |

## Required

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3b8fdf11-aa03-8128-96c0-cc092ec87b34
# or
ntn api v1/pages/3b8fdf11-aa03-8128-96c0-cc092ec87b34/markdown
```

3. 日程調整が必要な文面では、自発的にGoogle Calendarを読み、プレースホルダーを出さない。
4. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
5. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `google-workspace`
- skill `gws-hermes-ops`
- skill `skill-notion-source`
