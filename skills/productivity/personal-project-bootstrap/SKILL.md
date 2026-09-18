---
name: personal-project-bootstrap
description: "Use when bootstrapping a new personal project home."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-8136-96dc-fecac0f4b613"
    notion_url: "https://app.notion.com/p/bootstrap-3adfdf11aa03813696dcfecac0f4b613"
    canonical: notion-db-law-skills
---

# 個人プロジェクト bootstrap

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-8136-96dc-fecac0f4b613` |
| URL | https://app.notion.com/p/bootstrap-3adfdf11aa03813696dcfecac0f4b613 |
| カテゴリ | Notion |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-8136-96dc-fecac0f4b613
# or
ntn api v1/pages/3adfdf11-aa03-8136-96dc-fecac0f4b613/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
