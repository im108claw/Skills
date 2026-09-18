---
name: hermes-messaging-gateway-ops
description: "Use when operating Discord/messaging Gateway."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-8110-8247-ebad17c5396d"
    notion_url: "https://app.notion.com/p/Messaging-Gateway-3adfdf11aa0381108247ebad17c5396d"
    canonical: notion-db-law-skills
---

# Messaging Gateway

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-8110-8247-ebad17c5396d` |
| URL | https://app.notion.com/p/Messaging-Gateway-3adfdf11aa0381108247ebad17c5396d |
| カテゴリ | Hermes運用 |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-8110-8247-ebad17c5396d
# or
ntn api v1/pages/3adfdf11-aa03-8110-8247-ebad17c5396d/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
