---
name: drive-file-exchange
description: "Use when exchanging files with とーや via Google Drive."
version: 1.0.0
metadata:
  hermes:
    tags: [toya-claw, google-drive, file-exchange]
    related_skills: [google-workspace, gws-hermes-ops, toya-claw-os]
---

# Drive ファイル受け渡し（とーや ⟷ とーやクロー）

## When

- とーやと **ファイル**をやり取りする（成果物・入力・中間バイナリ）
- 「取り出しづらい」「Drive に置いて」「000_im108claw」系の話
- 生成物をチャット添付や Pi ローカルパスだけで返したくなったとき

## Policy (user preference)

1. **基本の受け渡しは Google Drive**。Pi ローカルに成果物を溜めない。
2. チャットには **Drive の webViewLink / folder URL** を返す。Pi 絶対パスだけ渡さない。
3. 公開コンテンツ本文は Notion「コンテンツ管理」。ここは **バイナリ・中間成果物・受け渡し**用。
4. 秘密（password / token / 鍵）は exchange に置かない。
5. ファイル名は ASCII（`A–Z a–z 0–9 . _ -`）。日本語は中身かチャット説明。
6. 正本の硬いルール: Notion **Vault Act §1b**（DB_Law）。食い違いなら Law を優先。

## Layout

| 役割 | Drive パス | folder_id |
|------|------------|-----------|
| ルート | `MyDrive/000_im108claw` | `1DaSuZ1yWfOGqW0J3S-pihl5o_YeMcpZ-` |
| exchange 親 | `000_im108claw/exchange` | `1yi-dKj6r-FhdRjyn2QZ0HEI6xDjfuEDX` ⚠️ may 404 |
| とーや → クロー | `exchange/from-toya` | `1VK06DOJVwC_4nmb0kkm-dWe-GFUdy0UY` ⚠️ may 404 |
| クロー → とーや | `exchange/to-toya` | `1Dw0I1wYna4xb7jWvTDmvNOVy8lqiISAL` ⚠️ may 404 |

- ルート URL: https://drive.google.com/drive/folders/1DaSuZ1yWfOGqW0J3S-pihl5o_YeMcpZ-
- 表記ゆれ: 歴史文書の `00_im108claw` は旧名。**実フォルダ名は `000_im108claw`**
- 詳細 ID・README 文言: `references/folder-ids.md`
- **2026-09-12:** historical `exchange/*` IDs returned Drive `notFound`. Live children under root included `Inbox`, `Backup`, `Remotion`, `Obsidian`, PRJ folders — **no `exchange`**. Until recreated, deliver to **root** (`ROOT`) or `Inbox` (`1T4veUi4GoySFZh862-aWxQBm5PNztvYz`), then tell user the link. Do not loop retries on dead exchange IDs.

## Pi reality

- Pi home に Drive **ライブマウントは無い**（`/mnt/shared/GoogleDrive/...` は使わない）
- vault 読み取り根拠は `/home/tcaret2/Documents/Obsidian`（エージェントは vault に新規書き込みしない）
- Drive は **API のみ**

## Commands (Pi)

`google_api.py` needs a Python that imports `googleapiclient` **and** `HERMES_HOME` pointing at the dir with `google_token.json` (on gateway-default often **`/opt/data`**, not only `~/.hermes`).

```bash
export HERMES_HOME="${HERMES_HOME:-/opt/data}"   # token: $HERMES_HOME/google_token.json
SCRIPT="${HERMES_HOME}/skills/productivity/google-workspace/scripts/google_api.py"
# fallback if skills live under /opt/data/skills while HERMES_HOME differs:
[ -f "$SCRIPT" ] || SCRIPT=/opt/data/skills/productivity/google-workspace/scripts/google_api.py

# Pick any python that can import the client (Hermes venv may be missing on this host):
GPY=""
for c in \
  "${HERMES_HOME}/hermes-agent/venv/bin/python" \
  "$HOME/.hermes/hermes-agent/venv/bin/python" \
  /opt/data/gvenv/bin/python \
  /opt/data/home/.cache/uv/archive-v0/*/bin/python
 do
  [ -x "$c" ] || continue
  "$c" -c "import googleapiclient,google.oauth2" 2>/dev/null && GPY=$c && break
done
GAPI="${GPY:?no python with googleapiclient} $SCRIPT"

ROOT=1DaSuZ1yWfOGqW0J3S-pihl5o_YeMcpZ-
FROM=1VK06DOJVwC_4nmb0kkm-dWe-GFUdy0UY
TO=1Dw0I1wYna4xb7jWvTDmvNOVy8lqiISAL

# list children
$GAPI drive search "'$TO' in parents and trashed=false" --raw-query --max 50

# deliver to とーや
$GAPI drive upload /path/to/artifact.ext --name "artifact.ext" --parent "$TO"

# ingest from とーや
$GAPI drive search "'$FROM' in parents and trashed=false" --raw-query --max 20
$GAPI drive download FILE_ID --output /tmp/work/file.ext

# PRJ docs (optional second parent): upload same file to PRJ-…/docs then also to $TO for chat pickup
```

Upload 成功レスポンスの `webViewLink` をチャットにそのまま返す。

## Workflow

### クロー → とーや（成果物）

1. 作業は `/tmp` 等の一時場所でよい
2. 完了したら `exchange/to-toya` へ `drive upload --parent $TO`（404 なら ROOT または Inbox — Layout 参照）
3. 返却 JSON の `webViewLink`（と必要なら `id` / `name`）をユーザーへ
4. ローカル一時ファイルは upload 後に掃除してよい（置きっぱ禁止）

### とーや → クロー（入力）

1. ユーザーに `exchange/from-toya` へ置くよう案内（または既置を search）
2. `drive search` / `drive download` で取得して作業
3. 結果は再び `to-toya` へ（ローカル完結にしない）

## Pitfalls

- **Pi パスだけ返す** → 取り出し不能。必ず Drive リンク
- **システム `python` / 固定 venv パスだけ試して諦める** → `import googleapiclient` できる interpreter を探して `GPY` にする（uv archive 可）。`HERMES_HOME` と token の組を合わせる
- **死んだ exchange ID にリトライし続ける** → upload が File not found なら即 ROOT か Inbox へフォールバックし、`references/folder-ids.md` を直す
- **ルート直下にバラ撒く（exchange があるとき）** → `exchange/{from,to}-toya` を使う。exchange 欠落時のみ root/Inbox 可
- **Obsidian / vault に成果物を書く** → Content-Homes / Vault Act 違反。readonly 根拠のみ
- **公開記事本文を exchange に正本置き** → Notion コンテンツ管理へ
- **folder_id を推測でハードコードし直す前に** → まず `drive search` で `000_im108claw` を実在確認。ドリフト時は `references/folder-ids.md` を更新
- **要件HTMLだけ Pi work に残す** → docs + to-toya に上げ、Notion からリンク

## Related

- skill `google-workspace`（汎用 API 手順・OAuth）
- skill `gws-hermes-ops`（Notion 正本ポインタ・ヘルス）
- Law: Vault Act §1b / Content-Homes Act
- skill `toya-claw-os`
