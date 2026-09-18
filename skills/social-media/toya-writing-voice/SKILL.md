---
name: toya-writing-voice
description: "Use when writing とーや's Japanese prose (report/blog/general)."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical, writing, voice]
    notion_page_id: "3aefdf11-aa03-812d-83ed-e0819e4d6055"
    notion_url: "https://app.notion.com/p/3aefdf11aa03812d83ede0819e4d6055"
    canonical: notion-db-law-skills
---

# とーや 文章の声（全体）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3aefdf11-aa03-812d-83ed-e0819e4d6055` |
| URL | https://app.notion.com/p/3aefdf11aa03812d83ede0819e4d6055 |
| カテゴリ | 発信 |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3aefdf11-aa03-812d-83ed-e0819e4d6055
# or
ntn api v1/pages/3aefdf11-aa03-812d-83ed-e0819e4d6055/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。
6. note / とやログは続けて `tcaret2jp-longform-content` と `toya-longform-style`。就活は `toya-job-application`。

## Related

- skill `tcaret2jp-longform-content`
- skill `toya-longform-style`（実記事解剖の型。ブログ長文は必ず重ねる）
- skill `note-longform-drafting`
- skill `toya-job-application`
- skill `fit-coursework`
- skill `skill-notion-source`
- skill `humanizer`（一般脱AI。とーや声はこの skill 優先）
