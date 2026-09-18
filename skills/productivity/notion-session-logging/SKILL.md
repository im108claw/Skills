---
name: notion-session-logging
description: "Use when writing/auditing DB_Logs Session (〜した)."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, notion, session-log, db-logs]
---

# DB_Logs Session（作成・欠落調査）

とーやクロー向け。**粒度・本文見出しの正本**は Notion skill `session-log-notion`（DB_Law）を先に読む。  
この skill は **ntn 実装・欠落監査・後追い復元** の実行手順。

## When

- トピック完了後に Session を書く
- ユーザーが「Session 残ってる？」「セットアップ時のログは？」と聞く
- 導入・設定変更をしたのに Session が無いことに気づいた

## 正本との関係

1. `export NOTION_KEYRING=0`
2. `ntn pages get 3adfdf11-aa03-81be-9450-cf70e98df764`（session-log-notion）
3. 粒度: **1 トピック = 1 行**、タイトルは `yyyy-mm-dd Session: <動詞完了形>`
4. プロパティは **名前 + Type=Session のみ**（スキーマ増やさない）

DB: database `3adfdf11-aa03-8003-8ec2-ced34d3c67fb` / data_source `3adfdf11-aa03-8039-b763-000b5cee1d69`

## 作成手順（ntn）

### 推奨: JSON で properties を明示

`ntn pages create` の frontmatter `title:` だけだと、DB 行の **名前が空のまま**になることがある。必ず PATCH で確定する。

```bash
export NOTION_KEYRING=0
# 1) 本文ファイル（frontmatter なしでも可）
# 2) create
ntn pages create --parent database:3adfdf11-aa03-8003-8ec2-ced34d3c67fb --json < body.md
# 3) 返った id で Type + 名前を確定
ntn api v1/pages/<id> -X PATCH -d '{
  "properties": {
    "名前": {"title": [{"type": "text", "text": {"content": "2026-07-30 Session: Fooを導入した"}}]},
    "Type": {"select": {"name": "Session"}}
  }
}'
# 4) 検証
ntn pages get <id> | head
```

本文セクション（推奨）: `## トピック` / `## やったこと` / `## 成果物` / `## Hermes`（`@session:default/...`） / `## 次` / `## 注意`（秘密なし）

### 忘れ防止

実質的な成果物・設定変更・方針決定の **返答前** に Session を書く。  
同一スレで複数トピックなら **複数行**（例: レポート作成と note 下書きは分ける）。

## 欠落監査（「残ってる？」への答え方）

1. **DB_Logs を正**にする。data_source query で全 Session タイトルを列挙し、キーワード検索。
2. Hermes `session_search` / `state.db` は **補助**。対話が無い導入もある。
3. 実機痕跡を必ず見る（導入の有無と日付）:
   - `~/.bash_history`
   - Docker (`docker inspect` の Created)
   - 設定ディレクトリ mtime（compose / settings.yml）
   - `~/.hermes/config.yaml` / `.env` の **キー名のみ**
4. 結論は三値で言う: **Session あり / 作業痕跡のみ / どちらも無し**

詳細: `references/missing-session-audit.md`

## 後追い Session（backfill）

- タイトル日付は **作業日**（mtime / Created / history）、作成操作の今日ではない
- 推測で盛らない。痕跡に無い動機・会話は「不明」か省略
- 秘密・パスワード・token 本文は載せない
- ユーザーが明示拒否しなければ、欠落が分かった時点で補完してよいか聞く（または短い確認後に作成）

## やらないこと

- Daily に Session を混ぜる / cron で Session 量産
- 旧ログの大量移植
- 雑談・接続確認のみの Session
- Obsidian への Session 書き込み

## Related

- skill `session-log-notion`（Notion 正本・粒度）
- skill `obsidian-ai-daily-report`（Daily は別）
- skill `notion-workspace-ops` / `notion-cli`
- skill `note-longform-drafting`（Session を note 材料にするとき）
