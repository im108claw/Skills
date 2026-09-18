---
name: web-research
description: "Use when asked to research/investigate web facts."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical, research, fact-check]
    notion_page_id: "3ccfdf11-aa03-812f-b994-e90105fad4ab"
    notion_url: "https://app.notion.com/p/Web-Research-Web-3ccfdf11aa03812fb994e90105fad4ab"
    canonical: notion-db-law-skills
---

# Web Research（Web調査・ファクトチェック）

**手順の正本は Notion（DB_Law → Skills → 調査）**。このファイルは発火条件と読み先だけ持つ。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3ccfdf11-aa03-812f-b994-e90105fad4ab` |
| URL | https://app.notion.com/p/Web-Research-Web-3ccfdf11aa03812fb994e90105fad4ab |
| カテゴリ | 調査 |
| 成果物 | **DB_AI Research** `3cbfdf11-aa03-8034-b3e2-f8e337436889` |

## When to Use

- 「〜〜を調べて。」「調査して。」および類似（リサーチ／裏取り／ファクトチェック）
- 外部の固有名・サービス・市場・技術・ニュースなど **Web 上の事実** 確認

Don't use for: コードデバッグ、Law/Skills 手順確認、コンテンツ本文執筆そのもの。

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3ccfdf11-aa03-812f-b994-e90105fad4ab
# or
ntn api v1/pages/3ccfdf11-aa03-812f-b994-e90105fad4ab/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. 必須: 成果は DB_AI Research / サブエージェント FC / 文末 `。[n]` 引用 / 非ファクトは理由付き明示。
5. 併読: skill `grounded-citations`、`notion-workspace-ops`。
6. Notion が読めないときだけ、同ディレクトリの `SKILL.md.local-full.bak`（あれば）を一時フォールバック。読めなかったことをユーザーに言う。

## Related

- skill `grounded-citations`
- skill `notion-workspace-ops` / `notion-obsidian-content-homes`
- skill `blocked-page-recovery`
- skill `skill-notion-source`
- DB_AI Research
