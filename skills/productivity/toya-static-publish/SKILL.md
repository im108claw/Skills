---
name: toya-static-publish
description: "Use when publishing static HTML/manuals via Drive."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, publish, static-html, drive, lab]
    related_skills: [drive-file-exchange, google-workspace, claude-design, toya-slide-decks]
---

# とーや静的公開（Kラボ相当・Drive版）

KEITO 動画の「デプロイスキル / Kラボ」相当。Vercel 専用ラボは持たない。  
**正本ファイルは Google Drive**、チャットには **webViewLink** を返す（`drive-file-exchange` 方針）。

## When to Use

- HTML マニュアル / 1枚ツール / LP 下書き / 図解説ページを「URLで渡したい」
- 「公開して」「ラボに上げて」「ブラウザで見れる形に」
- とーやに成果物を渡すとき、Pi ローカル path だけで返したくない

**Don't use for:** 本番本番サイト運用、有料ドメイン DNS、秘密を含むページ、Notion 正本の文書（文書は Notion）。

## Policy

1. 作業は `/tmp` または `/opt/data/work/static-publish/<slug>/` の一時場。
2. 完了後は **Drive へ upload**（既定: `exchange/to-toya`）。必要なら専用 lab 親を指定。
3. 返却は Drive の `webViewLink`（HTML は Drive 上でプレビュー、またはダウンロード後ブラウザ）。
4. ファイル名は ASCII。日本語説明はチャット側。
5. パスワード保護が要る公開は別途確認。勝手に全世界公開リンクにしない（Drive 既定共有のまま）。

## Prerequisites

- `drive-file-exchange` / `google-workspace` の Drive API が動くこと
- 静的 HTML を書けること（`claude-design` 可）
- 日本語フォントが要る埋め込み画像なら IPA Gothic 系がホストにある

## Quick Reference

```bash
# 作業場
ROOT=/opt/data/work/static-publish/<slug>
mkdir -p "$ROOT"

# 単体 HTML を Drive へ
GAPI_PY="${HERMES_HOME:-/opt/data}/skills/productivity/google-workspace/scripts/google_api.py"
# Prefer Hermes-managed python that has google clients if available; else system may fail.
python3 "$GAPI_PY" drive upload "$ROOT/index.html" --name "<slug>-index.html" --parent 1Dw0I1wYna4xb7jWvTDmvNOVy8lqiISAL

# 複数ファイル（index + assets）はフォルダごと
python3 "$GAPI_PY" drive mkdir "<slug>" --parent 1Dw0I1wYna4xb7jWvTDmvNOVy8lqiISAL
# 返った folder id を PARENT にして各 file upload
```

補助スクリプト（カテゴリ分類つき簡易ラボ index）:

```bash
python3 ${HERMES_HOME:-/opt/data}/skills/productivity/toya-static-publish/scripts/build_lab_index.py \
  --title "とーやラボ" \
  --out /opt/data/work/static-publish/lab/index.html \
  --entry manuals:rich-guide:/path/to/manual.html \
  --entry tools:demo-tool:/path/to/tool.html
```

## Procedure

1. **slug を決める**（`^[a-z0-9][a-z0-9-]{1,40}$`）。完了条件: 作業ディレクトリ作成済み。
2. **成果物を静的化する**  
   - 1ページなら `index.html`（インライン CSS 推奨）  
   - 複数なら `index.html` + `assets/`  
   - Brand が要るなら Design A（`#393939` / `#F1EB34` / `#FFFDF9`）  
   完了条件: ローカルで HTML が開ける。
3. **ジャンルを付ける**（任意）: `manuals` / `tools` / `slides-html` / `lp` / `games`  
   ラボ index を更新するなら `build_lab_index.py`。
4. **Drive へ納品**（`drive-file-exchange`）  
   - 単体: `exchange/to-toya`  
   - セット: 先に mkdir → 子を upload  
   完了条件: upload レスポンスの `webViewLink` を取得。
5. **チャット返却**  
   - 結論1行 → Drive URL → 中身の種類 → 次の一手  
   - Pi 絶対パスだけ返さない。

## Pitfalls

- Drive の HTML は「サイトホスティング」ではない。プレビュー制限あり。必要なら zip 同梱か、とーや側でダウンロード。
- Vercel/Cloudflare 直デプロイは **ユーザー明示時のみ**。既定は Drive。
- 秘密・APIキーを HTML に埋め込まない。
- `google_api.py` は google client 入り Python が必要なことがある。失敗したら `gws-hermes-ops` / setup check。

## Verification

- [ ] ローカル HTML が空でない
- [ ] Drive `webViewLink` がチャットに出ている
- [ ] ファイル名 ASCII
- [ ] 一時作業場を掃除したか方針を一文で述べた
