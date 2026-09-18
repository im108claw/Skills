---
name: tcaret2jp-longform-content
description: "Use when writing long-form note/とやログ/X articles."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-8155-bcb4-fdce83828603"
    notion_url: "https://app.notion.com/p/note-X-3adfdf11aa038155bcb4fdce83828603"
    canonical: notion-db-law-skills
---

# 長文（note / とやログ / X記事）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-8155-bcb4-fdce83828603` |
| URL | https://app.notion.com/p/note-X-3adfdf11aa038155bcb4fdce83828603 |
| カテゴリ | 発信 |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-8155-bcb4-fdce83828603
# or
ntn api v1/pages/3adfdf11-aa03-8155-bcb4-fdce83828603/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `toya-writing-voice`（声の土台）
- skill `toya-longform-style`（実記事解剖の型・リズム。コンテンツ管理で僕っぽく書く）
- skill `note-longform-drafting`（構成・冒頭・最下部メタ）
- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
