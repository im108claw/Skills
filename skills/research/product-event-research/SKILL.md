---
name: product-event-research
description: "Use when researching product launch or keynote facts."
version: 1.1.0
author: とーやクロー
license: MIT
metadata:
  hermes:
    tags: [research, news, product-launch, fact-check, toya-claw]
    category: research
    related_skills: [web-research, grounded-citations, blocked-page-recovery, notion-workspace-ops, toya-chat-reply]
---

# Product event / keynote research

記事執筆向けに **製品発表・速報イベント** の事実を最大収集するクラス手順。
一般調査の正本は skill `web-research`（Notion）。本 skill は発表当日〜直後の **収集・切り分け・正本化** に特化。

## When to Use

- 「今日のイベントの情報を集めて」「発表内容まとめて」「キーノート調査」
- hardware launch / 製品イベント（事前予想と発表済みが検索に混在）

Don't use for: 静的ファクトのみ、本文の文体執筆。

## Procedure

1. **時刻固定** — `date`（JST）。開始前=予想ラベル必須、開始後=公式 PR 優先。
2. **workdir** — `$HERMES_HOME/work/research-<slug>-YYYYMMDD/` with `raw/` `parsed/` `draft.md` `final.md`.
3. **公式ハブ** — Newsroom / Events / トップ。検索より **archive HTML の href 一覧**:
   ```bash
   curl -sL 'https://www.example.com/newsroom/archive/' \
     | rg -o 'href="/newsroom/YYYY/MM/[^"]+"' | sort -u
   ```
4. **各 PR を curl 保存 → ローカル text 抽出** — `web_extract` が 429/403/ナビ汚染のとき:
   ```bash
   UA='Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15'
   curl -sL -A "$UA" --max-time 45 "$URL" -o raw/page.html
   # stdlib HTMLParser で script/style 除去 → parsed/*.txt
   # "PRESS RELEASE" / "Pricing and Availability" 窓を切る
   ```
   ledger には **origin URL** を `sources.py add`（一時ファイルパスではない）。
5. **製品 marketing URL** で画面サイズ・予約文言を再確認。
6. **地域価格は Buy ページ**（marketing に円が無いことが多い）— 例 `/jp/shop/buy-…`。
7. **草稿** — 事実節と「解釈・未確認」節を分離。マーケ文言は自己評価と注記。
8. **FC** — leaf に **claims 表 + `parsed/pr_*.txt` パス**。再スクレイプ禁止。詰まったら steer で verdict 表のみ。メインが一次 URL を spot-check。
9. **正本** — DB_AI Research（title プロパティは **`名前`**）。作成/本文は skill `notion-workspace-ops`（page-body-write の DB_AI Research 節）。Discord はリンク＋超要約（`toya-chat-reply`）。

## Pitfalls

| Pitfall | Fix |
|---------|-----|
| 地域 Newsroom が EN より遅延 / 同 slug 404 | EN PR を一次。地域は shop/product で価格・発売 |
| 検索が予想記事だらけ | 「発表」＋公式 URL 優先。予想は NOT_FACT 節 |
| 二段リリース（機種で予約日が違う） | 表で予約/発売を製品行ごとに分ける |
| eSIM 専用国とバッテリ公称の混同 | モデル条件を脚注どおり |
| FC 子が複雑な heredoc で gateway 失敗 | 既抽出数値を steer。追加ツール禁止 |
| `web_extract` が nav だけ返す | curl+local extract。extract 成功扱いしない |

## Deliverable shape

```markdown
# <Event>（調査）
- 更新 / FC実施
## 一言
## 発表一覧表（価格・予約・発売）
## 製品ごと詳細（。[n]）
## ファクトチェック結果
## ファクトではない／未確認
## Sources
```

## Related

- skill `web-research` — 可能なら `hermes curator adopt web-research` して本内容を吸収
- skill `grounded-citations` / `blocked-page-recovery` / `notion-workspace-ops` / `toya-chat-reply`
- skill `toya-longform-style` / `toya-writing-voice` — 記事本文の声
- `references/keynote-screenshots.md` — VOD 取り違え防止・yt-dlp フレーム・Notion single_part 添付
