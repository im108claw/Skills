---
name: toya-chat-reply
description: "Use when replying to とーや in chat/Discord. Keep short."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical, communication]
    notion_page_id: "3ccfdf11-aa03-8193-a831-e1be237d1e6c"
    notion_url: "https://app.notion.com/p/3ccfdf11aa038193a831e1be237d1e6c"
    canonical: notion-db-law-skills
---

# チャット返信（短く）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3ccfdf11-aa03-8193-a831-e1be237d1e6c` |
| URL | https://app.notion.com/p/3ccfdf11aa038193a831e1be237d1e6c |
| 親法 | Communication Act §7 (`3adfdf11-aa03-8135-9a4a-f895c0756dd4`) |
| カテゴリ | 身分とOS |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3ccfdf11-aa03-8193-a831-e1be237d1e6c
# or
ntn api v1/pages/3ccfdf11-aa03-8193-a831-e1be237d1e6c/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。
5. **最終返答**: 結論→最小根拠→次の一手1つ。やったこと最大3点。長文は Notion/Drive リンクのみ。memory に方針を貼らない。

## Related

- skill `toya-claw-os`
- Communication Act
- skill `skill-notion-source`
