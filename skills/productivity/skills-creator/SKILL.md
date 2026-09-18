---
name: skills-creator
description: "Use when creating/editing skills. Notion DB_Skills is canonical."
version: 3.0.0
metadata:
  hermes:
    storage: notion-db-skills
    db_skills_id: "45387f60b7414141b6e7ace36caf062e"
    data_source_id: "2ff24399-28d6-4038-8878-c9a8f6756a71"
    notion_url: "https://app.notion.com/p/45387f60b7414141b6e7ace36caf062e"
---

# skills-creator (Notion Agent Skills)

**スキルの正本はすべて Notion の `DB_Skills` データベースで管理します。**

新規スキルの追加、および既存スキルの更新手順：

```bash
export NOTION_KEYRING=0

# DB_Skills へのスキル登録/更新 (Notion API v1/pages)
ntn api v1/pages -d '{
  "parent": {"data_source_id": "2ff24399-28d6-4038-8878-c9a8f6756a71"},
  "properties": {
    "Name": {"title": [{"text": {"content": "<skill-name>"}}]},
    "Description": {"rich_text": [{"text": {"content": "<description>"}}]},
    "Category": {"select": {"name": "<category>"}},
    "Status": {"status": {"name": "Active"}},
    "Target Agents": {"multi_select": [{"name": "Hermes"}, {"name": "Claude Code"}, {"name": "ChatGPT"}]},
    "Version": {"rich_text": [{"text": {"content": "1.0.0"}}]}
  }
}'
```

公式 Notion Agent Skills 規格に基いてプロパティとページ本文を維持します。
ローカルには二重バックアップを作成せず、DB_Skills のエントリーを主とします。
