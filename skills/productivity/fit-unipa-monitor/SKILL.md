---
name: fit-unipa-monitor
description: "Use when monitoring FIT myFIT/UNIPA board notices only."
version: 1.3.1
created_by: agent
metadata:
  hermes:
    tags: [toya-claw, fit, unipa, myfit, monitoring, cron, notion]
---

# FIT UNIPA / myFIT 監視

とーやの福岡工業大学ポータル（myFIT → UNIPA）を定期巡回し、**掲示板の必要そうな新着だけ**を:

1. Discord `#unipa` 親チャンネルの**新規スレ**へ要約
2. Notion **DB_Topic News** へ 1件=1行でスタック

**チャンネル設計の正本:** vault `Reference/fit/unipa-channel-design.md`  
**運用メモ:** vault `Reference/fit/unipa-monitor.md`

## いつ使う

- cron `fit-unipa-board-monitor` 実行時
- とーやが UNIPA / myFIT / 掲示監視を頼んだとき
- アプリ移行・次期 myFIT・システム停止の確認時
- 監視スレへの返信で「この件もう少し」と聞かれたとき（掲示板範囲内）

## 前提・制約

1. **秘密を出さない**: 学籍 ID/パスワードは vault `Reference/id-pass/my.fit.ac.jp.md` のみ。Discord・cron 最終文・Session・Notion に生パスワードを書かない。
2. **クラウド browser 無断禁止**（Criminal / Billing）。既定は Hermes のローカル `browser_*`。
3. **緊急の休講リレーは主務にしない**（友達づて優先）。
4. **惰性の超高頻度 heartbeat 禁止**（Task-Assignment §5）。既定は毎日 06:00 JST。
5. **差分のみ報告**。新規なしなら最終応答は厳密に `[SILENT]`。
6. パスワード入力はブラウザの password 欄にのみ。
7. **掲示板のみ**。クラスプロファイル / 学修情報・相談 / 成績 / レジュメDL / アンケート深掘りは定期監視の対象外。
8. **設定用の旧スレや Home に監視結果を返さない**。deliver は `#unipa` 親 + `attach_to_session`。
9. チャンネル設計と矛盾したら **channel-design を優先**し、skill を直す。
10. **DB_Topic News のプロパティを増やさない**（最小7列固定）。

## 入口

| 項目 | 値 |
|------|-----|
| ポータル | `https://my.fit.ac.jp` |
| SSO 後 UNIPA | `https://unipa.fit.ac.jp/...` |
| 資格情報 | vault `Reference/id-pass/my.fit.ac.jp.md` |
| state | `~/.hermes/state/unipa/seen.json` |
| 設計正本 | vault `Reference/fit/unipa-channel-design.md` |
| 配信先 Discord | `#unipa`（`1536937690950541393`） |
| Notion DB | `DB_Topic News` |
| database_id | `3bbfdf11-aa03-8080-b2b0-f2a576f621c7` |
| data_source_id | `3bbfdf11-aa03-8045-a27a-000b2c51553b` |
| 親ページ | FIT.ac.jp `3b5fdf11-aa03-8001-8e26-ed4c270b603b` |
| cron 本体 | `1e8791da73fc` / `0 6 * * *` |
| attach_to_session | `true` |
| pause / resume | `764a6f875bb8` / `619bdcb0a9de` |

## Notion DB_Topic News（最小スキーマ）

| プロパティ | 型 | 用途 |
|------------|----|------|
| 名前 | title | UNIPA 件名（そのまま） |
| Topic | select | 固定 `UNIPA`（他ソース拡張用） |
| ジャンル | select | システム/授業/試験/学務/奨学金/就職/その他 |
| 状態 | select | 新着 / 確認中 / 対応済 / 不要 |
| 日付 | date | 掲示日（分かれば）。無ければ拾った日 JST |
| リンク | url | **必ず https 完全URL**。UNIPA直URL優先。無ければ `https://my.fit.ac.jp/` |
| 要約 | rich_text | Discord と同じ短い要約（1行） |

**置かない:** 差出人・重要度・未読・Discord message id・本文全文・添付・学籍情報（煩雑化禁止。必要なら本文ブロックか将来の1列だけ）。

### スタック手順（新規ありのとき・Discord より先でも後でも可）

```bash
export NOTION_KEYRING=0
# 1) 同名が既にあればスキップ（冪等）
ntn api v1/data_sources/3bbfdf11-aa03-8045-a27a-000b2c51553b/query -X POST -d '{
  "filter": {"and": [
    {"property": "Topic", "select": {"equals": "UNIPA"}},
    {"property": "名前", "title": {"equals": "<件名>"}}
  ]},
  "page_size": 1
}'
# 2) 無ければ create
ntn api v1/pages -X POST -d '{
  "parent": {"database_id": "3bbfdf11-aa03-8080-b2b0-f2a576f621c7"},
  "properties": {
    "名前": {"title": [{"type": "text", "text": {"content": "<件名>"}}]},
    "Topic": {"select": {"name": "UNIPA"}},
    "ジャンル": {"select": {"name": "<ジャンル>"}},
    "状態": {"select": {"name": "新着"}},
    "日付": {"date": {"start": "YYYY-MM-DD"}},
    "リンク": {"url": "https://my.fit.ac.jp/"},
    "要約": {"rich_text": [{"type": "text", "text": {"content": "<要約>"}}]}
  }
}'
```

- cron が拾った新規は **状態=新着**
- create 成功時の **Notion page URL**（`https://app.notion.com/p/<id>` 形式の完全URL）を控える → Discord の `[n]` に使う
- とーやがスレで「対応した/不要」と言ったら 対応済 / 不要 に更新してよい
- Notion 書き込み失敗でも Discord 配信は行う。最終応答の末尾に載せない。失敗は state の `notion_last_error` に短く残す

## 手順（cron / 手動共通）

### 1. 停止期間

JST が **2026-08-19〜2026-08-23** ならログインせず:

```text
[SILENT]
```

### 2. state

```bash
cat ~/.hermes/state/unipa/seen.json
```

`known_titles` に無いものだけ候補。

### 3. ログイン

1. `browser_navigate` → `https://my.fit.ac.jp`
2. myFIT ID → ログイン
3. Microsoft password → Sign in
4. アンケート[Bsc005] 着地が多い → **掲示板へ即移動**
5. MFA が出たら止めてとーやへ

### 4. 巡回（掲示板のみ）

未読 → 新着 → 重要 → 授業

### 5. 拾う / 捨てる

**拾う:** 課題・提出・締切・レジュメ・資料・要確認・試験・履修・システム/アプリ/ポータル移行・奨学金/就職（期限近 or 重要）・履修科目名・教員名

**捨てる:** 一般イベント・市広報・ボランティア・既知タイトル

### 6. Notion スタック + state 更新

1. 報告する各新規を DB_Topic News に 1行（上記・冪等）
2. `known_titles` に件名追加し `updated_at` 保存

### 7. 最終応答（Discord）

#### 新規なし / 停止

```text
[SILENT]
```

#### 新規あり（これだけ）

```text
### ジャンル名
- 要約 [リンク文言](https://完全なURL)
```

ジャンル: `システム` / `授業` / `試験` / `学務` / `奨学金` / `就職` / `その他`

**`[n]` は必ず Markdown リンク + https 完全URL**（相対パス・`#` だけ・テキストのみ禁止）:

優先順:
1. UNIPA 件名の恒久URL（取れたときだけ・`https://...` 完全形）
2. **いま create した Notion 行の URL**（`https://app.notion.com/p/<page_id>`。スラッグ付きでも可だが必ず `https://app.notion.com/` から書く）
3. それでも無ければ `https://my.fit.ac.jp/`

Notion `リンク` プロパティも同じ優先順で **https 完全URLのみ**（portal フォールバック可）。

#### 障害

```text
【UNIPA監視エラー】短い理由。次に人がやること1つ。
```

## 既知の画面癖

- 入口 myFIT、本体 UNIPA JSF / PrimeFaces
- 件名は `href="#"` + AJAX モーダル
- 2026-08-19〜23: 次期 myFIT 移行停止
- 旧 UNIPA アプリは 2026-08-24 以降不可
- 更改はアプリだけではない（Web myFIT 移行あり）

## スレ返信時

とーやが監視スレに返信したら、**そのスレの文脈**で掲示板範囲の追加確認に答える。親 ch や Home に横流ししない。レジュメ全量・クラスプロファイルは別タスク。

## 関連

- vault `Reference/fit/unipa-channel-design.md`
- vault `Reference/fit/unipa-monitor.md`
- skill `notion-workspace-ops` / `notion-cli`
- skill `fit-coursework`
- Law: Criminal / Task-Assignment §5
