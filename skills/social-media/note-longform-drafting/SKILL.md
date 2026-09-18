---
name: note-longform-drafting
description: "Use when drafting note long-form. Lead hooks, bottom meta. DB_AI Writing home."
version: 1.1.0
metadata:
  hermes:
    tags: [toya-claw, note, longform]
---

# note 長文下書き（A×B）

とーや向け note 記事の **構成・冒頭・メタ置き場**。媒体の正本ルール（声・note仕事）は Notion skill `tcaret2jp-longform-content` と Reference「note / とやログ の書き方」を先に読む。

## When

- note の下書きを書く・直す
- 体験・ガジェット・移行ものの長文
- ユーザーが「伸びる書き方」「構成」に触れたとき

## 媒体の前提（再掲・短い）

| 項目 | 値 |
|------|-----|
| note の仕事 | 一次情報「こんなことしてみました！」 |
| 声 | ぼく／使っている人 |
| 禁止 | 講義・5選主役・マウント・同一全文の二重投稿 |
| パイプライン | コンテンツ管理 `Platform=note` + status `🤖｜AI Writing` |
| クロー本文の置き場 | **DB_AI Writing**（人と混ぜない）。書き方正本を必読 |

## A×B

- **A（主観）:** 体験・本音・過程・早とちり・一次の数字
- **B（伸び骨格）:** 冒頭で読者代弁→「ぼくもそうだった」／Before→行動→After／「この記事で書くこと」

本文を How-to 講座にしない。冒頭と骨格だけ B を足す。

### 推奨の流れ

1. フック（悩み代弁＋ぼくも）
2. 結論の温度（何の話か）
3. この記事で書くこと
4. いまの状態
5. Before
6. 決断
7. 過程（失敗込み）
8. After（実測）
9. 買ったもの（商品リンクはここ）
10. 向き不向き → 所感
11. **この記事について**（最下部のみ）

詳細: `references/note-draft-ab-and-pitfalls.md`

## 落とし穴（本人訂正・必須）

**冒頭で記事メタを言わない。読者離れする。**

NG 例:
- 「この記事は一次メモです」
- 「末尾にアフィリエイトを貼ります」
- 所感に「写真はあとで足す」など下書き用の独り言

OK:
- 商品リンクは「買ったもの」節に自然に置く
- メモである説明・アフィ制度・公開前 ToDo は最下部見出しへ

```markdown
## この記事について
### アフィリエイト表記
### 公開前チェック（自分用・公開時は削除）
```

## 手順

1. Notion 正本を読む（`export NOTION_KEYRING=0`）:
   - [note / とやログ の書き方](https://app.notion.com/p/note-3adfdf11aa03811c96dcd673877c3e61)（**必須**）
   - skill `tcaret2jp-longform-content` / `toya-longform-style`
2. 一次情報を Session / 実機 / 本人回答から集める。足りない動機・買い物・アフィは逆質問
3. `references/note-draft-ab-and-pitfalls.md` を読む
4. コンテンツ管理カードが `Platform=note|とやログ` / `🤖｜AI Writing` であることを確認
5. **DB_AI Writing** に下書きを作成（CM にクロー全文を直書きしない）
6. CM ↔ AI Writing をリレーション
7. 冒頭メタが無いことを確認し、CM を `👨‍💻｜In Review` へ
8. 詳細パイプライン: skill `notion-workspace-ops` → `references/content-ai-writing-pipeline.md`

## 材料の切り分け（Inbox レポート vs note）

| 材料 | 置き場 | note にするとき |
|------|--------|-----------------|
| 仕組み・用語・一般論の整理 | **Inbox / メモ** | そのまま載せない。核にしない |
| 自分が立てた・繋いだ・測った事実 | **note** | 「こんなことしてみました」の A 層 |

**NG:** 公式 docs 要約だけの note。  
**OK:** 実機（compose / ポート / settings 差分 / エージェント設定 / 失敗ログ）を軸に、仕組みは短く噛み砕く。

Session が無い導入でも、`bash_history`・Docker・設定ファイル・`.env` キー名（値は出さない）から一次は取れる。詳細は `references/note-draft-ab-and-pitfalls.md`。

## Related

- skill `tcaret2jp-longform-content`（Notion 正本。ローカル改修は `hermes curator adopt tcaret2jp-longform-content` 後）
- skill `toya-writing-voice`（声）
- skill `toya-longform-style`（実記事解剖の型・モードA/B。僕っぽく書く）
- skill `x-note-content-flywheel`
- skill `notion-session-logging`（Session 欠落調査・後追い作成）
- skill `toya-content-management`（CM / DB_AI Writing / 予約後 brushup）
- Content-Homes / コンテンツ管理 DB / DB_AI Writing
- `notion-workspace-ops/references/content-ai-writing-pipeline.md`
