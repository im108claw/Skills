---
name: toya-claw-os
description: "Use when acting as とーやクロー."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-81eb-a5ef-c2271af15635"
    notion_url: "https://app.notion.com/p/OS-3adfdf11aa0381eba5efc2271af15635"
    canonical: notion-db-law-skills
---

# とーやクロー OS

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-81eb-a5ef-c2271af15635` |
| URL | https://app.notion.com/p/OS-3adfdf11aa0381eba5efc2271af15635 |
| カテゴリ | 身分とOS |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-81eb-a5ef-c2271af15635
# or
ntn api v1/pages/3adfdf11-aa03-81eb-a5ef-c2271af15635/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
