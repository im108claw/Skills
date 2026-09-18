---
name: toya-slide-decks
description: "Use when making Japanese PPTX/PDF decks for とーや."
version: 1.2.0
metadata:
  hermes:
    tags: [slides, pptx, powerpoint, deck, japanese, toya, brand]
    category: productivity
    related_skills: [powerpoint, claude-design, toya-writing-voice, design-md]
---

# とーや向けスライドデッキ

「スライドで教えて」「資料にして」「1枚にまとめて」系は、説明だけで終わらず **開いて使える PPTX（＋PDF/プレビュー）** まで出す。

**既定ブランド = Design A（2026-08-26 確定）:** 炭 `#393939` + 看板黄 `#F1EB34` + 紙 `#FFFDF9`。  
詳細トークン: vault `Reference/brand/DESIGN.md` / work `/opt/data/work/brand-color/`。

## When to use

- とーやがスライド作成能力の活かし方・使い方を聞く
- 授業PDF・プロジェクト状況・就活・発信の骨子をデッキ化
- 共有用1枚、発表用、理解用ミニデッキ、台本付き発表資料
- **ブランド見本・色見本・トークン説明**のデッキ

## Default deliverable

1. **PPTX**（編集可能・正本）
2. **PDF**（すぐ見る用。LibreOffice があるとき）
3. Discord なら主要スライドの **JPG プレビュー** も添付
4. 返答は短く：結論 → ファイル → 枚数/中身 → 次の一手

Discord 添付: 本文に `MEDIA:/absolute/path/to/file` を書く。

## Default palette — Design A（必須）

| トークン | Hex | 用途 |
|---------|-----|------|
| `ink` | `393939` | 骨格・本文・枠（**LOCK**） |
| `ink-deep` | `2A2A2A` | より強い面 |
| `primary` | `F1EB34` | 合図・CTA・章の信号 |
| `primary-deep` | `F1AC34` | 二次強調・ホバー |
| `primary-soft` | `FFFB79` | 薄いハイライト |
| `surface` | `FFFDF9` | 紙地（既定背景） |
| `surface-muted` | `FFF1C0` | 注釈帯 |
| `border` | `E8E4D8` | カード境界 |
| `on-primary` | `393939` | 黄地の文字（**白禁止**） |
| `on-ink` | `FFFFFF` | 墨地の文字 |
| `danger` | `D3312D` | 警告のみ |

pptxgenjs では `#` なし・6桁: `color: "393939"`。

機械可読: `references/brand-design-a.json` と vault `Reference/brand/palette-canonical.json`。

## Workflow

1. **Brief を先に固定**（足りなければ仮定を明示して進む）
   - 誰に見せる？（自分 / 先生 / 面談 / 公開）
   - 何を起こす？（理解 / 提出 / 説明 / 判断）
   - 素材は何？（PDF / メモ / URL / 雑な箇条書き）
2. **Decide/Learn surface** として設計（講義口の長文ではなく、削って並べる）
3. **構成型を選ぶ**（下の Structure recipes）
4. `pptxgenjs` で 16:9（`10" × 5.625"`）を生成。作業場例: `/opt/data/work/slides-deck/<topic>/`
   - 既存の `brand-color/slides/node_modules` があれば **symlink 再利用**（毎回 npm し直さない）
   - `validate.py` 用に topic 配下 `.venv` + `defusedxml`（なければ `uv venv` → `uv pip install defusedxml`）
5. **検証パイプライン（必須・段階）**
   ```bash
   node build_deck.js
   # 1) structure（必須）
   .venv/bin/python /opt/data/skills/productivity/powerpoint/scripts/office/validate.py out.pptx
   # 2) text smoke（必須・PDF 無しでも）— OOXML から全スライド本文を出す
   #    手順は references/contrast-and-qa.md
   # 3) PDF/JPG（あるとき）
   python .../soffice.py --headless --convert-to pdf out.pptx
   pdftoppm -jpeg -r 130 out.pdf slide
   ```
6. PDF が出せないときは **PPTX 正本 + 代表 JPG 近似プレビュー**で納品してよい（ギャップを一文）。  
   近似プレビューは `Pillow` + ホストの日本語フォント（例: IPA Gothic）。**正本は常に PPTX**。  
   手順: `references/contrast-and-qa.md` →「Preview without LibreOffice」
7. 画像を見て（可能なら `vision_analyze`）文字切れ・低コントラストを直して再生成。vision 不可なら validate + text smoke + 代表プレビューで止めて出荷
8. 最終返答は短く: 結論 → `MEDIA:` 添付 → 枚数/中身1行リスト → 次の一手

## Structure recipe — Brand Specimen（好きな型・既定のお手本）

2026-08-26 の `toya-brand-color-specimen.pptx` を **とーや好みの構成**として固定。新規デッキでも「見本・仕様・方針」系はこれを型にする。

| # | 役割 | レイアウトの型 | メモ |
|---|------|----------------|------|
| 1 | **Title / lock dunk** | 全面 `ink`。下帯だけ `primary`（厚め）。上に kicker、大タイトル白、副題薄白 | タイトル下の細いアクセント線は禁止。**帯は面** |
| 2 | **Lock / 定義** | 左に巨大スウォッチ＋HEX。右に事実カード（役割・由来・禁止） | 1メッセージ＝固定色 |
| 3 | **Overview / 分岐** | 3カラムカード。各カード: ink ヘッダ → primary ストリップ → 名前/タグ/一言 → 下にミニバー4色 | 比較・選択肢 |
| 4–N | **Specimen / 中身** | ink ヘッダ帯（必要なら「推奨」ピル）。説明1行。**2×4 または 2×3 の色/項目チップ**（上色面＋下白ラベル） | トークンや論点を並べる |
| N+1 | **Contrast / ルール実演** | 全幅の横バーを縦に4本。bg×fg と比率をバー内に書く | 読める組み合わせを体感 |
| N+2 | **Components** | Primary / Ink / Outline ボタン行 → 下に紙カード＋ink パネル | 部品の見本 |
| N+3 | **DO / DON'T** | 左右2ペイン。緑系地に DO、赤系地に DON'T。箇条は短く | 判断用 |
| last | **NEXT** | 全面 ink。上に primary の細い帯（面）。白タイトル＋次アクション | 締めは行動 |

### この型のリズム（再利用）

1. **暗 → 明 → 暗**（タイトル/締めは ink、中身は surface）
2. **番号付きセクション**（`01  /  固定色` 形式、グレー小ラベル）
3. **1スライド1メッセージ**（タイトルは短く、説明は1–2行）
4. **面で階層**（細い下線・サイドバー・左縁ストライプ禁止）
5. **黄は合図だけ**（面積 5–15%。全面黄は特別なタイトルのみ）
6. **セーフフォント**: 見出し Cambria / 本文 Calibri
7. **マージン** ≥ 0.5"。カード角 `rectRadius` 0.08–0.12

実装参考: `/opt/data/work/brand-color/slides/build_deck.js`  
詳細: `references/specimen-structure.md`

## Design rules（とーや向け）

- 日本語本体。タイトルは短く、1スライド1メッセージ
- **既定色は Design A**（トピック固有色が必要なときだけ例外を明示）
- 汎用 indigo SaaS・青紫テックグラデ禁止
- **暗い背景**: 半透明白カード＋白文字禁止 → 白カード＋炭字、または墨カード＋白字
- **黄地に白字禁止**（必ず `on-primary` = `#393939`）
- 表は本文 14pt 前後。セル詰めすぎたら分割
- AI臭: タイトル下のアクセント線、縦サイドバー、左縁ストライプ、アイコン付き3カード万能レイアウトを避ける
- とーやの声: 当事者口・具体・盛り禁止。就活系は `toya-job-application` を優先

## Content patterns that work

| 用途 | 形 | 返すもの |
|------|----|----------|
| 授業 | 長いPDFを流れに分解 | 理解用6–8枚＋想定質問/台本 |
| PRJ | 目的・現状・次の1枚 | Discord共有用1枚 |
| 就活 | 経験→強み→根拠 | 面接練習デッキ（盛り禁止） |
| 発信 | 読者導線の可視化 | note/X の骨子スライド |
| **ブランド/仕様** | **Brand Specimen 型** | 見本PPTX＋トークン表 |
| **製品/世代の進化** | **Brand Specimen 型を流用** | 定義→世代概観→数字チップ→比較表→伸び幅バー→DO/DON'T→NEXT |
| 使い方説明 | 実物で教える | 教えつつ成果物になるデッキ |

### 製品進化デッキ（Brand Specimen の載せ替え）

「M6 の進化をまとめて」系は新レイアウトを発明せず、Specimen の役割を差し替える。

| Specimen 役割 | 製品進化での中身 |
|---------------|------------------|
| Lock | 今回の核（チップ名 / プロセス / 価格帯） |
| 3-up overview | 世代フェーズ（例: M1 / M4 / M6） |
| Token chips | CPU・GPU・NE・帯域・AI倍率など数字 |
| Contrast bars | vs 前世代の伸び幅（用途依存と明記） |
| Components | 変わらない筐体 vs 接続・上位SKU注意 |
| DO / DON'T | 買い替え判断（盛り禁止・条件付き） |
| NEXT | 用途分岐のチェックリスト |

事実は公式一次（Newsroom）を優先。二次記事の価格改定ストーリー等は「読み方」スライドに分離。

## Pitfalls

- 口頭の「活かし方」だけで終わる → **必ずデッキを作る**
- 暗い背景で `fill('FFFFFF', 5)` のような半透明白＋白文字 → 読めない
- 黄の上に白字 → コントラスト不足（必ず炭字）
- `validate.py` が `defusedxml` 不足 → topic `.venv` に `uv pip install defusedxml`
- PDF 変換が無い/落ちる → **deb 展開で LibreOffice を無理に立て直さない**。validate + text smoke + Pillow 近似 JPG で出荷し、PDF ギャップを一言
- 近似プレビューを「実スライドのスクリーンショット」と偽らない（Discord では代表数枚、正本は PPTX）
- 製品比較で公式「最大N倍」を全面に当てはめない → DO/DON'T と伸び幅スライドで用途依存を書く
- `powerpoint` 一般ルールと衝突したら **とーや向け可読性（日本語・Design A・短文）を優先**

## Related

- `powerpoint` — pptxgenjs / OOXML / QA
- `claude-design` — surface-first
- `design-md` — DESIGN.md トークン
- `toya-writing-voice` — 日本語の声
- `toya-job-application` — 就活・盛り禁止

## References

- `references/contrast-and-qa.md` — コントラスト失敗例・検証・PDF無しプレビュー
- `references/specimen-structure.md` — Brand Specimen 構成の詳細（製品進化マッピング含む）
- `references/brand-design-a.json` — Design A 機械可読
- `references/post-deck-skill-loop.md` — 実戦後の Kanban 起票と skill 改善ループ

## Known-good builders

- Brand specimen: `/opt/data/work/brand-color/slides/build_deck.js`
- 製品進化の実例: `/opt/data/work/slides-deck/m6-mac-mini/build_deck.js`（Design A + Specimen 11枚）
- 近似プレビュー例: `/opt/data/work/slides-deck/m6-mac-mini/render_previews.py`（Pillow + IPA Gothic）

## 実戦後のスキル改善（ループ）

デッキを出したあと「スキルをブラッシュアップしたい」「明日やって」が出たら:

1. **今すぐ直せる小さな穴**（手順1行・pitfall・reference）→ この skill をその場で patch
2. **まとまった改善**（構成レシピ追加・検証本線・Notion 正本同期）→ エージェント Kanban に起票  
   - 人の DB_Action には載せない（`hermes-deferred-handoffs`）  
   - 実行日が明日なら `create` のあと `schedule` で今夜の dispatch を止める  
   - workspace はデッキ作業場 `dir:/opt/data/work/slides-deck/<slug>`  
   - 添付: 正本 PPTX + `build_deck.js`（idempotency-key を付ける）  
   - skills: `toya-slide-decks` + `skill-notion-source` + `powerpoint`  
3. 改善の完了条件に「Notion DB_Law Skills 正本へ反映」を必ず入れる（ローカルだけ更新して終わらない）

CLI 形のメモ: `references/post-deck-skill-loop.md`
