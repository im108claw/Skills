---
name: im108claw-public-site-design
description: "Use when designing im108claw public intro/products pages."
version: 1.0.0
author: とーや (tcaret2), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [im108claw, design, landing, products, catalog, brand-a, toya-claw]
    category: creative
    related_skills: [claude-design, toya-slide-decks, design-md, toya-static-publish, remotion-editor-ops]
    brand: design-a
    live_refs:
      home: "https://im108claw.tcaret2.jp/"
      products: "https://im108claw.tcaret2.jp/products/"
      structure_ref: "https://tsubuan-design.pages.dev/apps"
---

# im108claw 公開サイト構成（紹介 + Products）

とーやクロー公開面（`im108claw.tcaret2.jp`）の **構成・トーン・公開境界** を固定するスキル。  
**軸 = 参考 apps カタログ構成**（[tsubuan-design `/apps`](https://tsubuan-design.pages.dev/apps)）。  
**付加 = 紹介フォーム（Decide/Learn LP）** をルートに載せる。

見た目トークンは **Design A 固定**（炭 `#393939` · 看板黄 `#F1EB34` · 紙 `#FFFDF9`）。  
参考サイトの色・フォント・blob 背景は **コピーしない**。レイアウト骨格だけ借りる。

## When to use

- `im108claw.tcaret2.jp` の `/` または `/products/` を作る・直す・増やす
- 「AIサービスっぽい紹介ページ」「Products 一覧」を同じトーンで足す
- 公開HTMLに何を出してよいか迷ったとき（内部情報の遮断）
- path-proxy 静的面（`homepage/`）の情報設計

**Don't use for:** Editor UI 本体（`/products/video-editor/`）、スライド（`toya-slide-decks`）、汎用ワンオフデザイン（`claude-design` のみで足りる場合）

## Surface map（必須）

| 面 | URL | 役割 | 表面タイプ |
|----|-----|------|------------|
| Home | `/` | 紹介・世界観・導線 | **Decide / Learn** |
| Products | `/products/` | アプリカタログ | **Explore**（一覧） |
| App | `/products/<slug>/` | 実プロダクト | Operate（既存アプリ） |

- ルートを **Editor 作業フォーム** にしない（「動画を開く／他を見る」専用品にしない）
- Products は **ハッシュ `#products` ではなく実パス** `/products/`
- `/products` → `/products/` は 308（トレイリングスラッシュ正規化）

## Axis A — 参考カタログ構成（tsubuan `/apps`）

Products ページの **骨格の正**。次を必ず含める。

1. **← Home** 戻るリンク（下線アニメ可。色は ink、アクセント線は yellow）
2. **page-head**
   - kicker（小・大文字・字間広め）例: `PRODUCTS`
   - 大見出し `Products`（clamp 大・weight 800）
   - 1〜2行の lead（「カードを開くとそのアプリへ」）
3. **catalog grid**
   - `repeat(auto-fill, minmax(280px, 1fr))`
   - 各カード:
     - **thumb**（16:10 前後 · 下枠線）
     - **cat-tag**（黄地・ink 字・大文字）
     - **title**
     - **desc**（短文）
     - **開く ↗**（hover で矢印が少し動く）
4. **カード hover**
   - 参考: `translateY` + **ink の硬い drop shadow**（`6px 8px 0 ink` 系）
   - 枠は **2px solid ink**（参考の「はっきり枠」を Design A に翻訳）
5. **Coming soon**
   - 実リンクなし · dashed 枠 · CTA は「準備中」
6. **footer** 最小（名前 + Home リンク）。内部ホスト名・Brand Kit・色コード列は出さない

参考から **借りないもの:** 灰背景 `#ededed`、Montserrat、黄色い morph blob 全面、他社サムネ・文言。

## Axis B — 紹介フォーム付加（Home）

ルートに載せる **付加レイヤ**。カタログの前段。

1. **top bar:** brand mark（icon+`im108claw`）| nav `Home` / `Products`（active ピル）
2. **hero（2カラム→SPは縦）**
   - eyebrow
   - H1（1メッセージ。強調は黄マーカー下線で `em`）
   - lead（プロダクトホームであることの説明）
   - **主CTA = Products を見る**（Editor 直行を主にしない）
   - 副CTA = ページ内「どんな場所？」
   - 任意: 小さな stat 行（数は嘘をつかない。公開数が 1 なら 1）
   - hero-art: **公開してよい icon のみ**（Drive Brand Kit の `im108claw.png` 系）。Brand Kit 名やパスは UI に出さない
3. **About（3点）** — ショーケース / すぐ開く / エージェントと育つ、など短く
4. **Pick up** — 1枚は現役プロダクト、1枚は catalog への導線
5. **footer** 最小

## Design A トークン（実装用・画面に印刷しない）

| token | hex | 用途 |
|-------|-----|------|
| ink | `#393939` | 文字・枠 LOCK |
| ink-deep | `#2A2A2A` | 強い面 |
| yellow / primary | `#F1EB34` | CTA・tag・信号 |
| yellow-deep | `#F1AC34` | hover |
| paper | `#FFFDF9` | 地 |
| paper-muted | `#FFF1C0` | 注釈帯 |
| border | `#E8E4D8` | 薄い境界 |
| on-yellow | `#393939` | 黄地の文字（**白禁止**） |

紙地 + 薄い黄 radial は可。ガラス盛りの全面 blur・紫グラデ SaaS は禁止。  
詳細: `/opt/data/work/brand-color/brand-design-a.json` · vault `Reference/brand/DESIGN.md`

## 公開境界（MUST）

**出してよい:** プロダクト名、短い紹介、公開 URL パス、icon、日本語 UI コピー  
**出してはいけない:**

- 内部ホスト / Cloudflare / path-proxy / ポート
- Brand Kit・Drive パス・「Design A」ラベル・色コード一覧
- 作業用メモ、PRJ 番号、デプロイ手順
- 未公開のブランド資産・中間ファイル名

CSS コメントにも内部フレーズを残さない（`/* public site styles */` 程度）。

## 実装配置（PRJ-9）

```
apps/remotion-editor/
  homepage/
    index.html                 # /
    products/index.html        # /products/
    assets/                    # shared CSS, public icon, favicon
  server/path-proxy.mjs        # static / & /products/ & /assets/*
                               # /products/video-editor/* → UI/API
```

- 静的面を足すとき: repo の `homepage/` にファイル → path-proxy の静的パス許可を更新
- 新プロダクトカード: `homepage/products/index.html` の catalog に1枚（thumb・tag・title・desc・href）
- 作業コピーは Pi 一時可。**公開正本は repo `homepage/`**
- 構成の詳細メモ: この skill の `references/structure.md`

## Procedure

1. **面を決める** Home / Products / 両方
2. **Axis A を Products に適用**（戻る・head・grid・tag・開く↗・硬い hover）
3. **Axis B を Home に付加**（紹介 hero・主CTA=Products・Pick up）
4. **Design A のみ**で彩色。参考の色を持ち込まない
5. **公開境界チェック**（上記 NG リストを grep 相当で確認）
6. path-proxy 経由で `curl` + ブラウザ確認:
   - `/` `/products/` `/products`（308） `/assets/*` `/products/video-editor/`
7. コミットは `homepage/` + `server/path-proxy.mjs`。内部メモを README に書くなら deploy 側のみ

## Copy tone

- 短文・断言・「〜できます」の連発を避ける
- 主メッセージ例: 「つくると動かすを、ひとつの場所で。」
- Products lead 例: 「気になったカードを開くと、そのアプリへ進みます。」
- 英語 nav ラベル（Home / Products）は可。本文は日本語優先

## Pitfalls

- ルートを Editor 専用 CTA に戻してしまう → 主CTA は常に catalog
- `#products` インページリンクで済ませる → 必ず `/products/`
- 参考の blob 背景や灰地を「それっぽく」入れる → Design A 紙地を崩す
- カードをただの薄い border カードにする → Axis A の **2px ink + 硬い影** を忘れない
- Coming soon を実リンク付きで出す
- Notion Skills 行が未作成でも、この SKILL を構成の正として使ってよい（DB_Law 追記は別途許可時）

## Verification

- [ ] `/` が紹介（Decide）で、主ボタンが Products
- [ ] `/products/` がカタログ（戻る・大見出し・カード・tag・開く）
- [ ] 公開 HTML/CSS に host / Brand Kit / Design A 字面 / 色コード表が無い
- [ ] Editor が `/products/video-editor/` で 200
- [ ] 見た目が炭×黄×紙のまま（紫・灰 blob 無し）

## Live reference (2026-08-27)

- Home: https://im108claw.tcaret2.jp/
- Products: https://im108claw.tcaret2.jp/products/
- Structure ref only: https://tsubuan-design.pages.dev/apps
- Repo snapshot: `apps/remotion-editor` commit 系 `feat(site): intro home + /products catalog`
