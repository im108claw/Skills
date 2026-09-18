---
name: toya-claw-meta-harness
description: "Use when recurring failures need harness outer-loop updates."
version: 2.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical, meta-harness]
    notion_page_id: "3c7fdf11-aa03-813e-aec1-d2ea72b80f6e"
    notion_url: "https://app.notion.com/p/Meta-Harness-3c7fdf11aa03813eaec1d2ea72b80f6e"
    canonical: notion-db-law-skills
---

# Meta-Harness（二重ループ）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。
方針の正本は **Procedure Code 第1編のter**。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3c7fdf11-aa03-813e-aec1-d2ea72b80f6e` |
| URL | https://app.notion.com/p/Meta-Harness-3c7fdf11aa03813eaec1d2ea72b80f6e |
| カテゴリ | 運用メタ |
| 出典 | arXiv:2608.13560 AutoDesign |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 方針: vault/Notion の **Procedure Code 第1編のter** を読む
3. 作業前に正本を読む:

```bash
ntn pages get 3c7fdf11-aa03-813e-aec1-d2ea72b80f6e
# or
ntn api v1/pages/3c7fdf11-aa03-813e-aec1-d2ea72b80f6e/markdown
```

4. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
5. optimization record は `HERMES_HOME/logs/meta-harness/`（現行 `/opt/data/logs/meta-harness`）。テンプレは本 skill の `templates/optimization-record.md`。
6. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- Procedure Code 第1編のter / 第1編の半
- skill `skill-notion-source`
- skill `toya-claw-os`
- skill `hermes-cron-operations`
- arXiv:2608.13560 AutoDesign
