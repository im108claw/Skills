---
name: toya-claw-heartbeat
description: "Use for とーやクロー Heartbeat secretary follow-up cron ticks."
version: 1.5.0
metadata:
  hermes:
    tags: [toya-claw, heartbeat, cron, reflection, secretary]
---

# とーやクロー Heartbeat（秘書フォローアップ型）

OpenClaw Heartbeat 相当を Hermes cron で回す。  
**既定スロットは使わない。** 各 tick は判断ベースの one-shot を **必ず1本**（チェーン維持）。時刻はフォロー価値で選ぶ。  
**監視だけで終わらない。** 観測 → 判断 → 低リスクなら1テーマ実行 → 報告。

## When

- cron job `toya-claw-heartbeat-next`
- 「自律で動いて」「Heartbeat 回して」「秘書としてフォローして」と言われたとき
- Task-Assignment Act §5 のリフレクションループ
- PRJ-7 予約投稿後の確認など、確定イベント直後のフォロー

## 正本

1. `~/.hermes/HEARTBEAT.md`（=`/opt/data/HEARTBEAT.md`）— **観測・実行プレイブック全文**
2. Task-Assignment Act §5
3. `/home/tcaret2/.hermes/scripts/toya_claw_heartbeat_prompt.txt`
4. 概念: https://docs.openclaw.ai/ja-JP/gateway/heartbeat

## 配信契約（紐づきで先を決める）

| 種類 | 先 |
|------|-----|
| Heartbeat の延長線（横断巡回・次起こし・全体健全性） | `#⏰cronjob` `1532907160441454753` |
| プロジェクト紐づき | **その PRJ チャンネル**（PRJ-7=`1538800749880811550`） |
| セッション／依頼スレ紐づき | **その ch / スレッド（origin）** |
| 詳細 | DB_Logs **Type=`Heartbeat`**（など） |

- 主目的で `deliver` を選ぶ。PRJ 予約着地 → PRJ ch。横断朝チェック → cronjob。
- 1 tick に両方あるときは **最終応答は主目的1先**。他方は短い別報告のみ。
- Home 禁止。要対応なしでも横断短報は cronjob へ。秘密禁止。同じ本文の二重配信禁止。

## 秘書としての本体

とーや要望（2026-08-25）:

- バックエンドで **フォローアップを実行**する（「何も実行していない短報」を減らす）
- 例: **約7日非アクティブなプロジェクトを追う** / **Gmail で Discord 言及の顛末を確認して報告**
- PRJ-7 作業は Heartbeat 範疇でも **報告先は PRJ-7 ch**（予約着地・Ready・reflection）
- 横断の秘書巡回まとめだけ `#⏰cronjob`

詳細手順・レーン定義は **HEARTBEAT.md の観測/実行プレイブック**に従う（ここに全文複製しない）。

## 次 tick の決め方（必須ルール・2026-08-28）

**禁止:** 10:00 / 14:00 / 18:00 など毎日同じグリッド。every-Nh forever。every 30m の秘書本体。**次なしで tick 終了**。

**やること:**

1. プレイブックで軽く見る（Calendar / DB_Action / Kanban / Gmail薄 / 他cron / PRJ-7予約 / 会話の残り）
2. 「とーやがフォローを欲しそうな最早の一点」を選ぶ。なければ薄巡回。**なぜその時刻かを報告に書く**
3. 未来の `toya-claw-heartbeat-next` は **必ず最大1本**（update or create --repeat 1）。**チェーン維持必須**
4. 時刻目安: 障害直後 +15–45m / 確定イベント +5–20m / フォロー中 +1–3h / 薄巡回 +3–6h（営業 08–23 JST。静音 0–7 はイベント以外避ける）
5. 材料が増える確定イベント（**予約投稿時刻を含む**）がある → その直後を最優先
6. drift_skip 対策: 可能なら CLI `hermes cron create --model grok --provider xai-oauth`。fleet は `cron.model` / `cron.model_provider`。バックストップは `scripts/heartbeat_chain_guard.py`（no_agent・数時間おき）

## Tick 手順

1. `read_file /home/tcaret2/.hermes/HEARTBEAT.md`
2. JST 実測
3. 観測プレイブック（P0→P5。高コスト全走査しない）
4. 判断 1〜3 行（秘書として優先した一点）
5. 低リスクなら 1 テーマだけ前進（Kanban 起票→完了）。優先: 障害修復 → イベント直後確認 → 秘書実行 → 在庫
6. 次 one-shot を **必ず1本**（時刻は判断。次なし禁止）
7. DB_Logs Heartbeat（ページ id は create 応答トップレベル `id`）
8. 最終応答 = 短報（観測・行動・とーやへ・次の job_id/時刻と **理由**）

## 推奨 skills（cron に載せる）

常時: `toya-claw-heartbeat`, `hermes-cron-operations`, `hermes-deferred-handoffs`, `notion-session-logging`, `session-log-notion`, `toya-claw-os`, `google-workspace`, `gws-hermes-ops`, `notion-task-management`, `notion-project-pages`, `notion-workspace-ops`

PRJ-7 予約確認 tick のみ追加可: `x-short-post-ops`, `xqueue-xposted-posting`, `x-browser-native-posting`, `notion-project-pages`

文体 brushup 実行時: `notion-workspace-ops` + 必要なら `toya-short-x-writing` / `note-longform-drafting` / `tcaret2jp-longform-content`

## DB_Logs

```bash
export NOTION_KEYRING=0
OUT=$(ntn pages create --parent database:3adfdf11-aa03-8003-8ec2-ced34d3c67fb --json < body.md)
# id = JSON top-level "id" または url 内 32hex。created_by.id を使うな
ntn api v1/pages/<PAGE_ID> -X PATCH -d '{
  "properties": {
    "名前": {"title": [{"type": "text", "text": {"content": "yyyy-mm-dd HH:MM Heartbeat: …"}}]},
    "Type": {"select": {"name": "Heartbeat"}}
  }
}'
```

DB: `3adfdf11-aa03-8003-8ec2-ced34d3c67fb` / ds: `3adfdf11-aa03-8039-b763-000b5cee1d69`

## 短報テンプレ（読みやすさ必須・2026-09-02）

とーや向けは **日本語の普通の文**。英語略語・内部IDの羅列禁止。

- 出してよい: 時刻、日本語の状況、サービス名（最小限）、詳細URL
- 短報に出さない: job_id / Kanban id / `CDP` `IR` `P4b` `f=` `pin` `exit=0` / model名 / パス
- 言い換え: CDP→予約用ブラウザ、In Review→レビュー待ち、Ready→投稿準備の在庫、reflection→振り返り、chain→次の巡回
- 次の行は `次: 9/3 8:40（朝の薄い巡回）` 形式。job名は書かない（詳細Notionへ）

```text
Heartbeat HH:MM

考えたこと: …
観測: …
行動: …
とーやへ: … / なし
次: M/D H:MM（理由を日本語で）
詳細: <URL>
```
## ガード

- 固定スロット・惰性同時刻埋め禁止
- cron 乱立禁止（未来 HB は最大1本）/ **次なし禁止（チェーン維持）**
- **実行中の自分の job を remove しない**（2026-09-02: 手動runで自己削除→短報完了なのに FAILED・自動配信欠落）。重複は「未来の別id」だけ整理し、今動いている tick の job は触らない
- 課金・公開・Gateway 無断・秘密禁止
- 即時フル自動投稿禁止
- cron で `execute_code` 不可
- 過去チャット掘り返しスパム禁止
- needs_input / In Review を毎日突かない
- **drift_skip（#44585）**: unpinned one-shot は model 切替で消費される。fleet `cron.model`/`cron.model_provider` + 可能なら CLI pin。バックストップ `scripts/heartbeat_chain_guard.py`（job `toya-claw-heartbeat-chain-guard`, no_agent, every 3h）
- **スクリプト優先（2026-08-25）**: 観測の決定的部分（state 読取・差分・閾値・cron list 整形）は `~/.hermes/scripts/` に切り出して保存。判断・短報文面だけ agent。新規 collector を作ったら本 skill と HEARTBEAT.md にパスを追記
- **文体学習（2026-08-29）**: `⏰｜予約投稿済み` → `scripts/content_style_brushup_candidates.py` → AI稿(DB_AI Writing) vs 人本文の差分で PF「書き方」（DB_Reference）を更新。Heartbeat と同タイミング（P4b）
- **公開確認（2026-09-12）**: 差分学習の**直後**に CM `公開URL` を実GET。公開済みなら Complete 月へ更新。学習済みで予約のまま残も検証のみ可。月 option 無ければ **Complete の `YYYY.MM` を全 option 再送で追加**（部分 PATCH 禁止）。正本: `content-ai-writing-pipeline.md` / `HEARTBEAT.md` §P4b

## 関連

- `hermes-cron-operations`（Script-first recurring jobs） / `hermes-deferred-handoffs` / `session-log-notion` / `toya-claw-os` / `google-workspace` / `notion-task-management`
