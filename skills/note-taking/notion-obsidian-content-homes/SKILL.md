---
name: notion-obsidian-content-homes
description: "Use when choosing where to put notes/logs/content (Notion-first OS)."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-81b3-b08b-e77141ad307b"
    notion_url: "https://app.notion.com/p/Notion-3adfdf11aa0381b3b08be77141ad307b"
    canonical: notion-db-law-skills
---

# 置き場判定（Notion中心）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-81b3-b08b-e77141ad307b` |
| URL | https://app.notion.com/p/Notion-3adfdf11aa0381b3b08be77141ad307b |
| カテゴリ | Notion |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-81b3-b08b-e77141ad307b
# or
ntn api v1/pages/3adfdf11-aa03-81b3-b08b-e77141ad307b/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
