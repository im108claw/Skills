---
name: crisp
description: "Use when operating Crisp chat via REST (inbox, messages, people)."
version: 1.0.0
metadata:
  hermes:
    tags: [Crisp, chat, support, REST, customer]
---

# Crisp REST API（とーやクロー）

Crisp ワークスペースを REST で読む・書く。GAS 不要。認証は **Website Token**（単一ワークスペース）を基本にする。

## 認証ファイル

| 項目 | 値 |
|------|-----|
| 保存先 | `$HERMES_HOME/crisp_credentials.json`（通常 `/opt/data/crisp_credentials.json`） |
| 権限 | `600` |
| 代替 env | `CRISP_IDENTIFIER` / `CRISP_KEY` / `CRISP_WEBSITE_ID` / `CRISP_TIER` |
| 備考 | このホストの `.env` は device node で書けないことがある → **JSON ファイルを正** |

JSON 例:

```json
{
  "identifier": "...",
  "key": "...",
  "website_id": "...",
  "tier": "website"
}
```

`tier` は Website Token なら `website`、Marketplace Plugin Token なら `plugin`。

## セットアップ（人間側）

Website Token（推奨・最短）:

1. https://app.crisp.chat/ を開く
2. **Settings → Workspace Settings → Advanced configuration**
3. **API Token → Generate Token**（identifier / key は一度だけ表示）
4. **Settings → Workspace Settings → Setup Instructions** で **Website ID** を控える
5. とーやクローに次を渡す（DM 推奨。公開チャンネルに生 key を貼らない）:
   - `identifier`
   - `key`
   - `website_id`

エージェント側:

```bash
python /opt/data/scripts/crisp_api.py setup \
  --identifier '...' \
  --key '...' \
  --website-id '...' \
  --tier website

python /opt/data/scripts/crisp_api.py check
# → AUTHENTICATED
```

漏えいしたら Crisp 側で即 Regenerate。

## CLI

```bash
CAPI="python /opt/data/scripts/crisp_api.py"

$CAPI check
$CAPI website get
$CAPI conversations list --page 1
$CAPI conversations list --page 1 --filter-unread 1
$CAPI conversation get SESSION_ID
$CAPI messages list SESSION_ID
$CAPI message send SESSION_ID --text '返信本文' --from operator
$CAPI people list --page 1
$CAPI raw GET '/website/{website_id}/conversations/1'
```

出力は常に JSON。`ok: false` なら `status` / `error` / `response` を読む。

## ルール

1. **初回は必ず `check`**。`NOT_AUTHENTICATED` なら setup を案内。
2. **メッセージ送信・会話操作は実行前にとーや確認**（宛先 session_id と本文を見せる）。
3. 秘密情報を Session ログや Notion に全文転記しない。`check` の website_name 程度に留める。
4. 日次クォータ目安: Website Token 約 10,000 req/day。連打しない。
5. 書き込み失敗時は body schema を REST Reference で確認（`invalid_data`）。

## よく使う Reference

- Auth: https://docs.crisp.chat/guides/rest-api/authentication/website-token/
- REST v1: https://docs.crisp.chat/references/rest-api/v1/
- Website ID: Crisp → Settings → Workspace Settings → Setup Instructions

## トラブル

| 症状 | 対処 |
|------|------|
| `NOT_AUTHENTICATED` | setup 未実施。3値を保存 |
| `invalid_session` / 401 | identifier/key/tier 不一致。Website token なのに `plugin` になっていないか |
| 403 | token 権限不足、または website_id 違い |
| 404 conversation | session_id 誤り / 別ワークスペース |

## 関連

- スクリプト正本: `/opt/data/scripts/crisp_api.py`
- Google 直叩きが向く作業は `google-workspace`（Crisp 経由にしない）
