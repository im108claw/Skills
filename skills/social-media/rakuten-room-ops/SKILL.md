---
name: rakuten-room-ops
description: "Use when researching or operating Rakuten ROOM."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical, rakuten-room]
    notion_page_id: "3affdf11-aa03-8138-81d7-d31e379e869a"
    notion_url: "https://app.notion.com/p/ROOM-3affdf11aa03813881d7d31e379e869a"
    canonical: notion-db-law-skills
    related_skills: [skill-notion-source, toya-claw-os, x-note-content-flywheel, xqueue-xposted-posting, toya-writing-voice]
---

# 楽天ROOM 研究・運用

**手順の正本は Notion（DB_Law → Skills → 発信）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3affdf11-aa03-8138-81d7-d31e379e869a` |
| URL | https://app.notion.com/p/ROOM-3affdf11aa03813881d7d31e379e869a |
| カテゴリ | 発信 |
| Project | PRJ-2 楽天ROOMの研究 |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3affdf11-aa03-8138-81d7-d31e379e869a
# or
ntn api v1/pages/3affdf11-aa03-8138-81d7-d31e379e869a/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. 商品リサーチはコンテンツ管理DBへ。`Platform=Room` + `ステータス=未着手`（値のみ、スキーマ変更禁止）。
5. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `skill-notion-source`（Skills の書き方）
- skill `toya-claw-os`（OS 全体）
- skill `x-note-content-flywheel`
- skill `xqueue-xposted-posting`
- DB_Law Skills カタログ
