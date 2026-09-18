---
name: obsidian-ai-daily-report
description: "Use when writing the AI daily report / DB_Logs Daily."
version: 2.1.0
metadata:
  hermes:
    tags: [toya-claw, notion-canonical]
    notion_page_id: "3adfdf11-aa03-81ec-9caa-e1a8c348f9da"
    notion_url: "https://app.notion.com/p/Daily-3adfdf11aa0381ec9caae1a8c348f9da"
    canonical: notion-db-law-skills
---

# 日報（Daily）

**手順の正本は Notion（DB_Law → Skills）**。このファイルは発火条件と読み先だけ持つ。  
Notion ページが integration 未共有で読めないときは **ローカル references + `SKILL.md.local-full.bak`** を施行（黙って省略しない）。

| 項目 | 値 |
|------|-----|
| Notion page_id | `3adfdf11-aa03-81ec-9caa-e1a8c348f9da` |
| URL | https://app.notion.com/p/Daily-3adfdf11aa0381ec9caae1a8c348f9da |
| カテゴリ | 記録と日報 |
| **書き先** | Notion `DB_Logs` Type=`Daily`（フル本文） |
| Obsidian | **readonly 根拠のみ**（書かない） |

## Required (とーやクロー)

1. `export NOTION_KEYRING=0`
2. 作業前に正本を読む:

```bash
ntn pages get 3adfdf11-aa03-81ec-9caa-e1a8c348f9da
# or
ntn api v1/pages/3adfdf11-aa03-81ec-9caa-e1a8c348f9da/markdown
```

3. 手順・例外・表は **Notion 本文**に従う。ここに長い手順を複製しない。
4. **必ずローカルも読む（Journal / メンション / 構造 / Discord配信）:**
   - `references/journal-as-secondary-fact.md` — `DB_Journal` を二次事実として既存 H2 に折り込む（構造変更禁止）
   - `references/structure-from-2026-07-25.md` — H2 固定・Notion メンション
   - `references/evidence-gathering.md` — Journal query 含む根拠集め
   - `references/discord-delivery-contract.md` — `#📒｜daily-log` 最終応答。「今日/次にやること」= 行動指針1〜3（ToDo再掲禁止）
5. **vault `Log/Daily` に write しない。** 構造見本は readonly `Log/Daily/2026-07-25.md`。
6. Notion が読めないときだけ `SKILL.md.local-full.bak` をフル手順フォールバック（読めなかったことを言う）。それでも vault へ日報を書かない。
7. Discord 最終応答は日報本文の貼付ではなく **Notion URL + その日意識する行動指針（最大3）**。タスク管理の要約にしない。

## Journal → Daily（要約・詳細は references）

- 正本の感情文: `DB_Journal`（ds `379347f1-3f4a-8034-9cdc-000bf0e68210`）
- Daily では **二次事実のみ**（件数・タイトル・基調・Action Status）。全文転載しない
- 置き場は既存 H2 のみ: スケジュール / 内部事情 / 根拠メモ（新 H2 禁止）
- Notion ページ引用: `<mention-page url="…">タイトル</mention-page>`（Obsidian `[[wikilink]]` の代替）

## Related

- skill `skill-notion-source` / `toya-claw-os` / Content-Homes Act
- DB_Logs ds `3adfdf11-aa03-8039-b763-000b5cee1d69`（要 integration 共有）
- references: `journal-as-secondary-fact.md`, `structure-from-2026-07-25.md`, `evidence-gathering.md`, `discord-delivery-contract.md`, `cron-diagnosis.md`, `cron-one-shot.md`
