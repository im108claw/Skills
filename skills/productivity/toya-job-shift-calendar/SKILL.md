---
name: toya-job-shift-calendar
description: "Use when external job shifts need Calendar update or draft."
version: 1.0.1
author: とーやクロー
license: MIT
metadata:
  hermes:
    tags: [toya-claw, calendar, job, shift, gmail, google-workspace]
---

# 外部確定シフト → Job カレンダー反映

とーや 2026-09-15: **自分で変えられない外部確定の予定変更**（バイトのシフト確定・休み・着時間変更など）は確認待ちせず **Job カレンダー等へ即反映**する。

関連の正本（方針の長い表）:
- Notion skill `toya-schedule-coordination` page `3b8fdf11-aa03-8128-96c0-cc092ec87b34`（節「外部確定シフトのカレンダー反映」）
- `/opt/data/HEARTBEAT.md` P1（Gmail 薄読みで拾って即反映）

ローカル pointer skills（`toya-schedule-coordination` / `toya-claw-heartbeat` / `gws-hermes-ops`）は user-owned。本文を直すなら foreground で `hermes curator adopt <name>` 後に patch。

## When to Use

- 店・雇用側メールで出勤／休み／スタート時刻が **確定**した
- Heartbeat が Gmail で店都合シフトを見つけた
- 「カレンダーが変わった／勝手に直して」系の依頼
- シフト確定に伴う **返信下書き**（送信は別ゲート）

## Do（即実行してよい）

1. 一次ソースを読む（Gmail 本文）。推測で書かない。
2. 関連カレンダーを広く見る: primary / Job / Own Work / Travel / FIT 等。
3. **Job** カレンダー上の該当枠を更新:
   - 出勤確定 → `opaque`（busy）、要約例 `博多良品｜出勤確定`
   - 休み確定 → `transparent`（free）、要約例 `博多良品｜休み（店都合・確定）`。busy のまま残さない
   - 時間変更 → start/end を店指定に合わせる
4. description に根拠: 日時・gmail msg id / thread・要約1行・更新時刻
5. 返信が必要なら **Gmail 下書きのみ**（`toya-outbound-send-gate`）。送信しない
6. とーやへ短報: 何を変えたか / 下書き有無 / 送信待ちか

## Don't

- 本人都合の希望シフト提出・候補日の無断確定
- 面談・私的予定の無断作成削除
- 根拠あいまいな変更
- 「下書きして」「カレンダー直して」＝送信許可と解釈する

## Runtime（Pi）

```bash
# system python の google_api.py は依存不足で落ちやすい → gvenv 直呼び
/opt/data/gvenv/bin/python
# token
/opt/data/google_token.json
```

Job calendar id（2026-09 時点）:

`68bf5687c456dd4b94d8ed4a8286b634787b620262acc1770514599649c4312c@group.calendar.google.com`

詳細・例: `references/hakata-ryohin-shift-ops.md`

## Related

- `toya-schedule-coordination`（Notion 正本・候補出し）
- `toya-outbound-send-gate`（外へ届く送信のみ）
- `toya-claw-heartbeat` / `HEARTBEAT.md` P1
- `google-workspace` / `gws-hermes-ops`
