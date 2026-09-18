---
name: xqueue-xposted-posting
description: "Use when posting/operating short-X queue via コンテンツ管理."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-810c-85c6-e94c6da83286"
    notion_url: "https://app.notion.com/p/X-3adfdf11aa03810c85c6e94c6da83286"
    canonical: notion-db-law-skills
---

# 短文Xキュー投稿

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-810c-85c6-e94c6da83286` |
| URL | https://app.notion.com/p/X-3adfdf11aa03810c85c6e94c6da83286 |
| カテゴリ | 発信 |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-810c-85c6-e94c6da83286
# or
ntn api v1/pages/3adfdf11-aa03-810c-85c6-e94c6da83286/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- DB_Law Skills カタログ
