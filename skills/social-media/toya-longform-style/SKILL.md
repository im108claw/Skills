---
name: toya-longform-style
description: "Use when writing とーや blog long-form in his real-post voice."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical, writing, longform]
    notion_page_id: "3b2fdf11-aa03-812c-bf80-d7df937192d2"
    notion_url: "https://app.notion.com/p/3b2fdf11aa03812cbf80d7df937192d2"
    canonical: notion-db-law-skills
---

# とーや 長文スタイル（実記事解剖）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3b2fdf11-aa03-812c-bf80-d7df937192d2` |
| URL | https://app.notion.com/p/3b2fdf11aa03812cbf80d7df937192d2 |
| カテゴリ | 発信 |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3b2fdf11-aa03-812c-bf80-d7df937192d2
# or
ntn api v1/pages/3b2fdf11-aa03-812c-bf80-d7df937192d2/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。
6. 声の土台は `toya-writing-voice`、媒体の仕事は `tcaret2jp-longform-content`、構成は `note-longform-drafting` を重ねる。
7. 下書きはコンテンツ管理 DB（`Platform=note` / `とやログ`、status=`🤖｜AI Writing`）。

## Related

- skill `toya-writing-voice`
- skill `tcaret2jp-longform-content`
- skill `note-longform-drafting`
- skill `skill-notion-source`
- skill `humanizer`（一般脱AI。とーや声は voice + この skill 優先）
