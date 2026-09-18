---
name: fit-lecture-reports
description: "Use when writing FIT lecture reports (AI-DS etc.)."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, fit, coursework, reports]
---

# FIT 講義レポート（提出文）

大学の講義レポート（とくに **AIデータサイエンス基礎** の回ごと短文）を書くときの手順。  
教材探索の入口は skill `fit-coursework`（Notion正本）。**文体**は skill `toya-writing-voice` のレポート節。

## いつ使う

- 「AIデータサイエンス基礎」「第N回レポート」など FIT 提出文
- 課題文に **200字以上**・**必須語句**・字数上限がある短文レポート
- Drive 上の講義スクショ／PDF から設問を起こして下書きするとき

## 手順

1. **課題文を正本にする**  
   - ユーザーが課題文を貼ったらそれを最優先。推測で回を決め切らない。  
   - 未提示なら Drive から探す（下記）。見つからなければ **根拠を明示して確認**してから書く。

2. **教材を実測で探す（AI-DS）**  
   - `003_FIT/02_2026_1年前期/` に **科目フォルダが無い**ことがある。名前検索だけだと空振りする。  
   - Drive raw query の本命:
     - `fullText contains 'AIデータサイエンス基礎' and trashed=false`
     - ヒットの多くは **講義動画のスクショ PNG**（課題文が画面に載っている）
   - 親フォルダ例（2026-08 時点）: `1whEl7_8QlF3_SYFAMLUpMWEspsoFtQo_`  
   - 詳細マップ: `references/ai-ds-source-map.md`

3. **スクショから課題文を取る**  
   - `tesseract <png> stdout -l jpn+eng --psm 6`  
   - OCR は崩れる。必須語句・字数条件は複数枚で突き合わせる。読めない回はユーザーに課題文を求める。

4. **下書きの型（レポート声）**  
   - `toya-writing-voice` レポート節: 当事者口・だ/である混在・「思う／といえる／と考える」  
   - 導入 → 本論（意義/特徴）→ ただし限界 or 注意 → だから何を明らかにしたいか  
   - 講義口・「肝要／に資する」連発・整いすぎ三段禁止  
   - **必須語句は本文に必ず入れる**（課題の ※ 条件）

5. **字数は改行除きで Python 計測してから渡す**

```python
compact = text.replace("\n", "")
print(len(compact))
assert lo <= len(compact) <= hi  # 例: 200–300
```

6. **Discord では完成稿中心**  
   - 途中の探索ログを長く出さない。  
   - 先に **どこを見て判断したか** を短く1ブロック、次に **提出用本文**、最後にチェック表。

## ピットフォール

- 科目フォルダ名検索だけで「教材なし」と決めない（AI-DS はスクショ散在）  
- 第N回を Daily ログの ACT 名だけで推定して書き始めない（課題文不一致が起きる）  
- 字数を目視で済ませない（200字前後は特に数え漏れやすい）  
- 必須語句を入れ忘れる（平均値/中央値/最頻値、フレーム問題 など）  
- `fit-coursework` は Notion ポインタ。手順の増分は **この skill** か Notion Skills 側へ

## 関連

- skill `fit-coursework` — 教材・OCR方針の Notion 正本入口  
- skill `toya-writing-voice` — レポート声  
- skill `google-workspace` / `gws-hermes-ops` — Drive 取得  
- skill `ocr-and-documents` — PDF 抽出  
- `references/ai-ds-source-map.md` — AI-DS 探索レシピと既知の回  
- `references/fit-drive-source-map.md` — 003_FIT 一般マップ（流用）
