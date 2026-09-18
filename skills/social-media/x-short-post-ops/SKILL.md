---
name: x-short-post-ops
description: "Use when scheduling short X posts with a fixed body."
version: 1.0.0
---

# 短文X運用（指定文面・予約）

とーや / @im108claw 向けの短文X。Notion 正本スキル `xqueue-xposted-posting`（DB_Law）と X-Publishing Act の下で動く **運用レイヤ**。

## いつ使う

- 「この文面で予約して」「今日HH:MMにポスト」
- コンテンツ管理の `Platform=X` 短文を投稿・予約する
- ユーザーが本文を確定したあとの投稿作業

## 絶対ルール（ユーザー訂正 2026-08-21）

1. **指定本文は verbatim**  
   とーやが本文を渡したら一字一句そのまま。禁止: 読みやすさ改善・有益性盛り・AI感除去のための再執筆・字数のための言い換え圧縮。  
   許可する正規化のみ:  
   - Discord メンション `<@id>` / 壊れた `@…jp` → 正しい `@handle`  
   - 既知固有名の明らかな誤変換のみ（例 `Gmmnii`→`Gemini`）  
   正規化したら報告する。
2. **X 本体 UI の予約投稿**（即時ポストは明示指示があるときだけ）。
3. **時刻は 5 分刻み**（`:00` 優先）。言われた時刻を勝手にずらさない。
4. コンテンツ管理の当該行本文も **同じ指定文面**に合わせる。

## 手順（指定文面あり）

1. 文面をファイルに固定（改変しない）。
2. weighted 長を測る（CJK/全角系≈2、その他≈1）。**280超でも勝手に短くしない** — 超過を伝えて判断を仰ぐ。
3. CDP: Default Chromium only、`9222`、`--remote-allow-origins=*`。
4. スクリプト: `~/.hermes/state/prj-7/x_native_schedule_generic.py`  
   `--text-file … --when YYYY-MM-DDTHH:MM --tag … --needle …`
5. 成功 = 予約済み一覧に **本文断片 + 時刻**（例 `午後10:00`）。絵文字は list DOM で欠けることがある → compose の text_match と list を分けて報告。
6. Notion: 当該 X 行を更新（本文=指定、メモに予約時刻）。予約が入ったらステータス `⏰｜予約投稿済み`（Complete 月は公開後のみ）。学習用 AI 稿があるなら DB_AI Writing とリレーション済みであること。

## 手順（指定文面なし・下書き）

- X-Publishing Act + 文体プロファイル。作業ログの「〜した。〜した。」連打や講義調は避ける。
- ユーザーが文面を確定したら **以降は verbatim ルール**に切り替え。

## Pitfalls

- 「直してあげた方がいい」判断で指定文を捨てる → 最大の失敗。
- 即時投稿してから消す/差し替える。
- 有料 X API を使う（禁止。Billing-Media / 既存方針）。
- Chromium を `chromium-browser` 名で起動（Pi は `/usr/bin/chromium`）。CDP は `--remote-allow-origins=*` がないと WS 403。
- **CDP が生きていても identity 未確認で compose に進む** → 9222 が UNIPA 等だと空 home / no textarea。先に `@im108claw` を確認（詳細: `x-browser-native-posting` checklist）。失敗時は文面ファイルと時刻を残し、人のログイン復旧を次にする。

## Related

- PRJ-7 PDCA 正本: `~/.hermes/state/prj-7/PDCA.md`（設計→作成→**人レビュー後予約**→分析。即時フル自動禁止）
- Notion skill pointer: `xqueue-xposted-posting`（正本は DB_Law；user-owned のときは `hermes curator adopt xqueue-xposted-posting` 推奨）
- `x-browser-native-posting`
- Law: X-Publishing Act / Billing-Media Act
- local detail: `references/verbatim-and-schedule.md`
- `toya-content-management`（`⏰｜予約投稿済み`・DB_AI Writing relation）
- `notion-workspace-ops/references/content-ai-writing-pipeline.md`
