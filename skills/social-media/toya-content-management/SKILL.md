---
name: toya-content-management
description: "Use when running とーや content CM/AI-Writing/PF guides."
version: 1.2.0
created_by: agent
metadata:
  hermes:
    tags: [toya-claw, content, notion, publishing]
---

# コンテンツ管理パイプライン（とーや）

公開系コンテンツの **置き場・執筆分離・PF別書き方・予約後学習** のクラス技能。  
媒体ごとの声の細部は各 writing skill / DB_Reference「書き方」。ntn の細手順は `notion-workspace-ops`。

## When

- コンテンツ管理 / DB_AI Writing / 「AI Writing の意味」に触れる
- note・とやログ・X・IG・Threads・Room の下書き〜予約〜完了
- 「書き方を育てる」「差分学習」「予約投稿済み」「公開確認」「Complete月」
- 旧「共有コンテンツ管理」表記に出会った（現行名は **コンテンツ管理**）
- とーや: 月 option は足せるか → **Complete の `YYYY.MM` は足してよい**（workflow は増やさない）

## 一枚絵

```
コンテンツ管理（Platform / ステータス / 関係）
  status=🤖｜AI Writing → クローが書く番
       ↓ 必読: Platform ごとの DB_Reference「書き方」
  本文 = DB_AI Writing（人と混ぜない）
       ↓ relation dual
  👨‍💻｜In Review → 人直し
       ↓
  ⏰｜予約投稿済み → Heartbeat が AI稿 vs 人本文で書き方 brushup
       ↓ 差分学習の直後（必須）
  公開URL を実GET検証 → 公開済みなら Complete月へ更新
       ↓
  Complete月（公開URLは維持）
```

**理由（とーや 2026-08-29）:** クロー文と本人文の差が消えないようにする。

## 正本の読み順

1. この skill（パイプライン）
2. `notion-workspace-ops` → `references/content-ai-writing-pipeline.md`（IDs・create・brushup・pitfalls）
3. `notion-workspace-ops` → `references/toya-claw-db-map.md` Publish 節
4. Platform の書き方ページ（`/opt/data/state/content-pf-writing-guides.json`）
5. 媒体 skill（note → `note-longform-drafting` / X → `toya-short-x-writing` + `x-short-post-ops` 等）

## ステータス（値のみ・増やすな）

| 値 | 意味 |
|----|------|
| `未着手` | ネタ捕捉 |
| `🪴｜アイデアを育てる` | 種 |
| `🤖｜AI Writing` | クロー執筆（本文は DB_AI Writing） |
| `👨‍💻｜In Review` | Ready / 人チェック |
| `⏰｜予約投稿済み` | 予約済 + **文体学習トリガ** |
| Complete 月（例 `2026.08` / `2026.09`） | **URL検証で公開確認できたものだけ** |
| `アーカイブ` | 出さない |

## 予約投稿済み → 差分学習 → 公開確認（必須・2026-09-12）

`⏰｜予約投稿済み` で **差分学習（brushup）を終えたら、続けて**:

1. CM の **`公開URL`** プロパティを読む（空ならステータスは触らない。短報1行）
2. その URL を **実GETで検証**（リダイレクト追随。推測禁止）
3. **公開済み**（概ね HTTP 200 系・本文が下書き/ログイン壁/404でない）なら  
   ステータスを **Complete 月**へ更新  
   - 月の決め方: `Publish` 開始日の `YYYY.MM`（無ければ検証日の JST 月）
   - その月 option が DS に無い → **`YYYY.MM` はエージェントが足してよい**（下の手順）。workflow 系（未着手/AI Writing 等）は増やさない
4. 未公開・失敗・空URL → 予約のまま。次回 Heartbeat で再検証可（再学習はしない）

### Complete 月 option の足し方（2026-09-12 実測）

`PATCH v1/data_sources/<CM_DS>` で status options を更新できる。**部分リストは他 option を消す**ので **全 option + `group` を毎回送る**。

**推奨（手打ち ntn より先）:** skill 同梱スクリプト

```bash
PROMOTE=/opt/data/skills/social-media/toya-content-management/scripts/content_publish_status_promote.py
# 1カード昇格（URL検証→月option確保→Complete）
python3 "$PROMOTE" --cm-id <CM_PAGE_ID>
# 予約のまま残を一括（学習済だけなら --prefer-brushed）
python3 "$PROMOTE" --all-scheduled --prefer-brushed
# 月 option だけ
python3 "$PROMOTE" --ensure-month 2026.10
# dry-run
python3 "$PROMOTE" --cm-id <ID> --dry-run --verbose
```

手で ntn する場合だけ: 全 option + `group`（To-do / In progress / Complete）。新月は `color=green, group=Complete`。

許可: Complete の `YYYY.MM` 追加のみ。禁止: 新規 workflow ステータス・グループ改変・既存名のリネーム・**部分 options PATCH**。

詳細: `notion-workspace-ops` → `references/content-ai-writing-pipeline.md`  
Heartbeat: `/opt/data/HEARTBEAT.md` §P4b

## 絶対（本人方針）

1. **CM にクロー全文を直書きしない**（差が消える）
2. 書く前に **その PF の書き方** を読む（複数 Platform なら全部）
3. 書き上がり後 **必ず** CM ↔ DB_AI Writing リレーション
4. スキーマの **workflow option 追加禁止**（未着手/AI Writing 等）。**Complete 月 `YYYY.MM` だけ**は全 option 再送 + `group=Complete` で追加可（部分 PATCH 禁止＝他が消える）
5. 即時フル自動公開禁止。指定短文Xは verbatim
6. vault Law は readonly のことがある → 運用正本は skills + Notion 書き方 + db-map
7. **差分学習後に公開URL未検証のまま放置しない**（予約のまま永久化を防ぐ）

## PF 書き方（DB_Reference・領域=発信）

| Platform | ハブ |
|----------|------|
| note / とやログ | 同一ページ（声同じ・仕事だけ違う） |
| X | X の書き方（+ X文体プロファイル） |
| Instagram / Threads / Room | 各「書き方」（骨格→差分で育成） |

## Heartbeat 連携

- 候補: `~/.hermes/scripts/content_style_brushup_candidates.py`
- 処理済み: `/opt/data/state/content-style-brushup-done.json`
- **公開昇格:** `scripts/content_publish_status_promote.py`（brushup 直後 or 予約残り）
- skill `toya-claw-heartbeat` の P4b と同タイミング（本体 skill が user-owned のときは HEARTBEAT.md を正）
- 学びは **具体ルール** を書き方へ（抽象スローガンだけ増やさない）
- **P4b 後半:** brushup 後（または学習済みなのにまだ予約）→ promote スクリプト（`公開URL` 検証 → Complete 月）

## 関連

- `notion-workspace-ops` / `notion-obsidian-content-homes`（後者は adopt 推奨でローカル追記可）
- `note-longform-drafting` / `tcaret2jp-longform-content` / `toya-longform-style`
- `toya-short-x-writing` / `x-short-post-ops` / `xqueue-xposted-posting`
- `toya-claw-heartbeat`（adopt 推奨）
- `rakuten-room-ops`
