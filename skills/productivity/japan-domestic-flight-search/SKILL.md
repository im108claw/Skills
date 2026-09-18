---
name: japan-domestic-flight-search
description: "Use when finding cheapest Japan domestic flights (LCC/U25)."
---

# Japan domestic flight search (とーや)

至急の国内線「最安／始発／年齢割」探し。比較サイトはCAPTCHAが多いので **curl + Google Flights HTML** と **エアトリ日次カレンダー** を主軸にする。

## 先に確認する条件

1. **空港固定か**（成田NRT絶対 / 羽田HND可）
2. **行きの制約**（始発 / 最安 / 到着期限）
3. **帰りの制約**（最安 / 最終便前 / 到着期限）
4. **年齢割**（U18・U22・U25・学生）→ LCCには基本効かない
5. 人数・片道/往復・日付（年は会話年を仮定し明示）

条件が矛盾するときは **最優先条件を1つ** 決めてから検索（例: 始発必須なら最安夜便は参考扱い）。

## 路線の前提（よく使う）

| 路線 | 直行キャリア | メモ |
|------|-------------|------|
| **FUK–NRT** | Peach (MM) + Jetstar (GK) のみ | ANA/JAL直行なし |
| FUK–HND | ANA/JAL/LCC等 | 本数多・年齢割はこちら向き |

## 年齢割（U18/U22 等）

- **Peach / Jetstar: U18・U22 常設割引なし**（スカイスキャナー等の解説どおり）。最安は通常LCC運賃比較。
- **JAL スカイメイト / ANA ユース・スマートU25**: 主に **羽田路線**・当日0時以降など制約大。成田固定の最安ルートには乗りにくい。
- **FDA U22ハッピー割**: FDA路線のみ。FUK–NRT非対象。
- ユーザーが「U18/U22込み最安」と言ったら **成田固定なら効かない旨を先に言い**、羽田に条件変更できるか聞く。

## 検索手順（実効順）

### 1. 時刻表（始発・運休日）

- Jetstar 国内線PDF: `https://files.jetstar.com/api/public/content/gk_timetable_26nsdom`（年度でURL差し替え）
  - `curl -sL -o /tmp/gk_tt.pdf URL && pdftotext -layout /tmp/gk_tt.pdf -`
  - FUK↔NRT 節で **運休日** を読む（8月の火水木運休など）
- Peach 国内線PDF / NAVITIME / さくらトラベルの当日表で便名突合
- FUK→NRT の典型始発: **Peach MM348 08:00→10:05**（時期により要確認）
- NRT→FUK の早朝: Jetstar **GK523 06:15** 帯が始発候補（運航日注意）

### 2. 料金（比較サイトが死んだとき）

**A. Google Flights（往復・片側の相対価格）**

```bash
# 往復 FUK-NRT 例（日付を tfs 内の YYYY-MM-DD に合わせる）
URL='https://www.google.com/travel/flights/search?tfs=...&hl=ja&curr=JPY'
curl -sL -A 'Mozilla/5.0 ...' "$URL" -o /tmp/gflights.html
```

Pythonで `aria-label` を抽出:

- `往復の合計金額 N 円～。 {Peach|ジェットスター} ... 出発/到着時刻`
- itinerary リンク: `FUK-NRT-MM-348-YYYYMMDD` / `FUK-NRT-GK-524-...` → **便名確定**
- 表示が「往復の合計」でも片側指定URLだと片側料金に近い値になることがある。**往復URLとエアトリ片道を併用**して解釈する。

**B. エアトリ日次カレンダー（片道の日別最安）**

- 往路: `https://www.airtrip.jp/flight/FUK/NRT/`
- 復路: `https://www.airtrip.jp/flight/NRT/FUK/`
- HTML内 `data-date="YYYY/MM/DD"` + 近傍の `¥N,NNN` をパース
- **その日の最安便**であり始発固定価格ではない。始発は Google 側の便別RT or 公式で確認。

**C. 公式**

- Peach: https://booking.flypeach.com/jp  
- Jetstar: https://www.jetstar.com/jp/ja/home  
- 支払手数料・PFC・手荷物は別途と明記

### 3. 回答フォーマット（Discord向け）

1. **結論表**: 行き（条件付き）／帰り最安候補／合計目安
2. **便名 + 時刻 + キャリア**
3. **料金の出典と注意**（変動・手荷物別・LCC年齢割なし）
4. **予約リンク**（Google往復・公式・比較）
5. 空港到着目安（始発08:00ならFUK 6:30前後）
6. 年齢割が効く代替（羽田）は条件分岐で短く

長い手順ログは出さない。至急依頼は **先に結論**。

## Pitfalls

- Skyscanner / Expedia / HIS は bot・CAPTCHA で止まりやすい → 最初から Google+エアトリ+PDF
- NAVITIME等 CloudFront 403 → UA付きでも落ちる。公式PDF優先
- Googleの「往復合計」を片道と誤読しない。始発固定RTと夜便最安RTを分けて書く
- お盆前後は価格急騰。カレンダーの隣接日も一言添える
- JALコードシェア表示（JJP）と実運航 GK を混同しない
- ブラウザ操作でカレンダーUIを粘るより **curl抽出の方が速い** ことが多い

## References

- `references/fuk-nrt-lcc.md` — FUK–NRT 便・年齢割・抽出パターン
- `references/google-flights-scrape.md` — aria-label / itinerary 抽出メモ
