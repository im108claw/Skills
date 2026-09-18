---
name: x-note-content-flywheel
description: "Use when running the X×note content flywheel from activity."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-81f7-b66c-c0ec1669227f"
    notion_url: "https://app.notion.com/p/X-note-3adfdf11aa0381f7b66cc0ec1669227f"
    canonical: notion-db-law-skills
---

# X×note コンテンツ輪

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-81f7-b66c-c0ec1669227f` |
| URL | https://app.notion.com/p/X-note-3adfdf11aa0381f7b66cc0ec1669227f |
| カテゴリ | 発信 |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-81f7-b66c-c0ec1669227f
# or
ntn api v1/pages/3adfdf11-aa03-81f7-b66c-c0ec1669227f/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
