---
name: toya-cloudflare-ops
description: "Use when operating Cloudflare via MCP and Wrangler."
version: 1.0.0
author: とーやクロー
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [toya-claw, cloudflare, wrangler, mcp, notion-canonical]
    notion_page_id: "3ccfdf11-aa03-8175-9dd5-e24d10ca9888"
    notion_url: "https://app.notion.com/p/Cloudflare-MCP-Wrangler-3ccfdf11aa0381759dd5e24d10ca9888"
    canonical: notion-db-law-skills
    related_skills: [hermes-cloudflare-mcp, wrangler, cloudflare, hermes-agent, toya-claw-project-ops]
---

# Cloudflare クイック操作（MCP × Wrangler）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3ccfdf11-aa03-8175-9dd5-e24d10ca9888` |
| URL | https://app.notion.com/p/Cloudflare-MCP-Wrangler-3ccfdf11aa0381759dd5e24d10ca9888 |
| カテゴリ | Hermes運用 |

## When to Use

- Cloudflare 上の Workers / KV / R2 / D1 / ビルド / ログを素早く確認・操作するとき
- MCP と Wrangler の使い分けが必要なとき
- 「アカウント俯瞰」「Worker 調査」「デプロイ」など運用タスク

Don't use for: 初回 Skills/MCP インストールだけ（→ `hermes-cloudflare-mcp`）。

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3ccfdf11-aa03-8175-9dd5-e24d10ca9888
# or
ntn api v1/pages/3ccfdf11-aa03-8175-9dd5-e24d10ca9888/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、`SKILL.md.local-full.bak` と `references/quick-ops.md` を一時フォールバック。読めなかったことをユーザーに言う。
5. セットアップ不足なら skill `hermes-cloudflare-mcp`。

## Related

- skill `hermes-cloudflare-mcp`
- skill `wrangler` / `cloudflare`
- skill `hermes-agent`
- skill `toya-claw-project-ops`
