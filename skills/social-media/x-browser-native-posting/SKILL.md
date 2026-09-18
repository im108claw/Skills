---
name: x-browser-native-posting
description: "Use when posting/scheduling X via browser (no paid API)."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, x, browser, prj-7, schedule]
---

# X ブラウザ投稿・予約（無課金経路）

有料 X API / `xurl` を使わず、**ログイン済み Chromium + CDP** で投稿・予約するときの手順。
PRJ-7 専用アカ（とーやクロー）と、本人アカのブラウザ経路の両方に使う。

## いつ使う

- 「ブラウザ経由で X に投稿／予約して」
- 有料 API 未承認、または明示的にブラウザ経路
- Hermes cron での時刻投稿ではなく、**X 本体の予約投稿**を求められたとき

## ユーザー拘束（必須）

1. **単一デフォルトプロファイルのみ**
   - `user-data-dir=/home/tcaret2/.config/chromium`
   - `profile-directory=Default`
   - CDP: `http://127.0.0.1:9222`
   - 専用 `~/.hermes/browser-profiles/...` は **使わない**（とーや指示 2026-08-20）
2. **「予約投稿」= X 本体 UI**。Hermes cron で時刻投稿にしない（cron は別用途）。
3. パスワード / 2FA / 人間確認ゲートは **人がやる**。エージェントは秘密を打たない。
4. 投稿文の勝手な改変・タグ追加・別アカ投稿禁止。

## 起動

```bash
export DISPLAY=:0
export WAYLAND_DISPLAY=wayland-0
# 既存デフォルト Chromium がデバッグ無しで掴んでいる場合は一旦終了してから:
chromium \
  --user-data-dir=/home/tcaret2/.config/chromium \
  --profile-directory=Default \
  --remote-debugging-port=9222 \
  --remote-allow-origins=* \
  --no-first-run \
  --no-default-browser-check \
  --window-size=1280,900 \
  https://x.com/home
```

設定:

```bash
hermes config set browser.cdp_url "http://127.0.0.1:9222"
hermes config set browser.allow_private_urls true
```

※ Gateway が古い CDP 設定のままなら再読込が要る場合あり。スクリプト直叩き CDP でも可。

## 文字数（無料枠）

- 上限 **加重 280**（twitter-text 系: 基本 CJK=2、英数=1）
- 下書き前に Python で加重カウント。オーバーしたら削る
- 絵文字は用途ごとに **1種1回**（例: 🙌 の二重使用を避ける）

## 予約投稿フロー（CDP）

1. ログイン確認: home で表示名 / `@handle`（PRJ-7 は `@im108claw`）
2. compose: `SideNav_NewTweet_Button` または `https://x.com/compose/post`
3. 本文: focus `tweetTextarea_0` → `Input.insertText`（空なら行ごと + Enter）
4. 予約: `data-testid="scheduleOption"`（aria「ポストを予約」）
5. ダイアログ: select 月/日/年/時/分 + `input[type=date|time]` を React 向けに value setter + change
6. `scheduledConfirmationPrimaryAction`（確認する）
7. チップ表示を確認（例: `2026年8月20日(木)の午後5:00に送信されます`）
8. `tweetButton` 文言 **予約設定** を押す（即時「ポストする」と取り違えない）
9. **read-back 必須**: `https://x.com/compose/post/unsent/scheduled` に本文断片があること
10. 成功証拠: スクショ + `~/.hermes/state/.../result.json`

参考スクリプト（PRJ-7）: `~/.hermes/state/prj-7/x_native_schedule.py`  
詳細メモ: `references/prj-7-browser-schedule.md`

## 詰まりどころ

| 症状 | 対処 |
|------|------|
| 専用プロファイルで起動してしまった | 止めてデフォルトのみでやり直す |
| 9222 が UNIPA/headless_shell 等の別 user-data-dir | その CDP で X 予約しない。Default ログイン Chromium を用意するまで停止 |
| home が空 / ログイン壁 / X privacy-extension エラー | **未ログイン扱い**。人に Default で `@im108claw` ログイン+CDP 起動を依頼。ポートや prj-7-x を渡り歩いて成功扱いにしない |
| ページ WS がハング / 白画面タブ | browser ターゲットから **新しい Target** を作り直す。壊れたタブは閉じる |
| `Input.insertText` が空 | クリックで focus → 1文字起こし → 再 insert。それでもダメなら改行分割 |
| 「予約設定」後に人間確認モーダル | **投稿未確定**。人に確認させてから再セット。予約一覧で本文が見えないなら成功扱いにしない |
| 予約一覧がホーム TL に見える | URL 遷移が SPA で効いてない。compose unsent を開き直し、「予約済み」タブを明示クリック |
| AT-SPI / cua capture が空 | Wayland では CDP を主経路にする（cua は補助） |
| 時刻指定の作業 | とーや方針: 即実行せず予約。X 予約なら本体 UI、それ以外の開始は cron |
| `import websocket` 失敗 | hermes venv か `state/prj-7/.venv-x` + websocket-client。system python3 は不可 |

## PRJ-7 専用アカの声（短く）

- 主体: とーやクロー（エージェント実況）。`@tcaret2jp` の代筆ではない
- 型: Do → 結果 / 詰まり / 次。講義口調にしない
- テーマ北極星: Notion×AI×推し活（本丸 IG、X は有用な Do/Fact のみ）
- 投稿方針: **量より質は維持**＋**週2〜3ターゲット / 直近7日≥1本フロア**（Meetup一次根拠）
- 日次0は可。週次0と7日空白は不可。薄い日は捏造せず **在庫の予約分**で埋める
- 在庫: Ready（X待ち）**≥2**。P1~80% / P2~20%（検証済み再構成・引用ブースト）
- 公開ガード: 当面レビュー、秘密・未確認なし。ゲート不合格は出さない

## 関連

- X-Publishing Act（価値ゲート＋第2.5編ケイデンス。専用アカは主体・声を PJ 内で読み替え）
- Meetup: [SNSの運用戦略](https://app.notion.com/p/3bffdf11aa0380239ad9dfb669a52119)（頻度・在庫は Meetup 優先）
- skill `xurl`（API。承認前は書込に使わない）
- skill `xqueue-xposted-posting`（本人アカ短文キュー / コンテンツ管理）
- Billing-Media-Act（有料 API 無断禁止）
