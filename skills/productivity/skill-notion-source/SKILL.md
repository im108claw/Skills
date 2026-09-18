---
name: skill-notion-source
description: "Use when adding/updating とーや skills; Notion DB_Skills is canonical."
version: 3.0.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical, agent-skills]
    notion_db_id: "45387f60b7414141b6e7ace36caf062e"
    data_source_id: "2ff24399-28d6-4038-8878-c9a8f6756a71"
    notion_url: "https://app.notion.com/p/45387f60b7414141b6e7ace36caf062e"
    canonical: notion-db-skills
---

# Skills正本ルール（Notion DB_Skills 統一運用）

**スキルの唯一の正本（SSOT）は Notion（DB_Skills）**。
`DB_Law` とは完全に分離し、公式 Agent Skills 仕様に準拠して一元管理します。

| 項目 | 値 |
|------|-----|
| Notion DB_Skills ID | `45387f60b7414141b6e7ace36caf062e` |
| Data Source ID | `2ff24399-28d6-4038-8878-c9a8f6756a71` |
| URL | https://app.notion.com/p/45387f60b7414141b6e7ace36caf062e |
| 仕様 | Notion Agent Skills 規格準拠 (Name, Description, Category, Status, Target Agents, Allowed Tools, Version) |

## Standard Operations (とーやクロー)

1. `export NOTION_KEYRING=0`
2. スキル参照・実行時は Notion `DB_Skills` の該当ページから最新のテキストを取得：

```bash
# Data Source クエリで該当スキルを検索
ntn api v1/data_sources/2ff24399-28d6-4038-8878-c9a8f6756a71/query
# または ページIDから取得
ntn pages get <page_id>
```

3. ローカル側には冗長な `.local-full.bak` などを保持せず、Notion DB_Skills を直接の正本として扱います。

## Related

- skill `skills-creator`（DB_Skills への追加・更新手順）
- DB_Skills カタログ
