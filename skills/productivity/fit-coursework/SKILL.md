---
name: fit-coursework
description: "Use for FIT coursework PDFs/audio/summaries."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-81a8-8e4c-fefff6050b3a"
    notion_url: "https://app.notion.com/p/FIT-3adfdf11aa0381a88e4cfefff6050b3a"
    canonical: notion-db-law-skills
---

# FIT 教材・課題

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-81a8-8e4c-fefff6050b3a` |
| URL | https://app.notion.com/p/FIT-3adfdf11aa0381a88e4cfefff6050b3a |
| カテゴリ | Google / FIT |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-81a8-8e4c-fefff6050b3a
# or
ntn api v1/pages/3adfdf11-aa03-81a8-8e4c-fefff6050b3a/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
