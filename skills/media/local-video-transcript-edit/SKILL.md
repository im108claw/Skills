---
name: local-video-transcript-edit
description: "Use when Drive/local video needs SRT/STL + edit notes."
version: 1.0.0
---

# Local video transcript + edit notes

Drive / ローカル動画の **文字起こしファイル作成** と、解説動画向けの **編集ポイント整理**。
YouTube URL だけなら `youtube-content`。Gmail/Calendar 本体は `google-workspace`。

## Triggers

- Google Drive の動画リンク（`/file/d/FILE_ID/`）を渡された
- SRT / STL / VTT / タイムコード付き文字起こしが欲しい
- ストレッチ・筋トレ・動作解説など「素材動画 → 解説動画」の編集観点が欲しい

## Workflow

1. **メタ取得 → ダウンロード**
   - FILE_ID を URL から抜く
   - Hermes venv 経由で Drive API（システム python は `googleapiclient` 欠落しがち）:
     ```bash
     HERMES_PY="$HOME/.hermes/hermes-agent/venv/bin/python"
     GAPI="$HERMES_PY $HOME/.hermes/skills/productivity/google-workspace/scripts/google_api.py"
     $GAPI drive get FILE_ID
     $GAPI drive download FILE_ID --output /tmp/vid/source
     ```
2. **ffprobe → 音声抽出**
   - duration / 解像度 / fps / 音声トラック数を記録
   - STT 用: mono 16k wav（＋必要なら mp3）
     ```bash
     ffmpeg -y -i source -vn -ac 1 -ar 16000 -c:a pcm_s16le audio_16k.wav
     ```
   - 4K60 の再エンコードは重い。編集判断には **1fps サムネ** で足りることが多い:
     ```bash
     ffmpeg -y -i source -vf "fps=1,scale=1280:-2" -q:v 3 frames/f_%02d.jpg
     ```
3. **文字起こし（ローカル優先）**
   - クラウド STT キーが空でも進める: `faster-whisper` on Hermes venv
     ```bash
     $HERMES_PY -c "import faster_whisper"  # missing → uv/pip install into hermes venv
     ```
   - 既定: `small` + `language="ja"` + `word_timestamps=True` + `vad_filter=True`
   - 出力は JSON（segments/words）を残してから字幕に落とす
4. **補正（必須チェック）**
   - カウント系（3,4,5…）は冒頭が **「13」「4」誤認**しやすい → 系列と logprob で直す
   - 音声ピーク（RMS）で表示開始を微調整可
   - 低信頼トークンは md に「補正した」と明記
5. **字幕ファイルを書く**
   - **SRT**（編集・YouTube 最優先・互換最高）
   - **VTT**（Web）
   - **STL** = **Spruce plain-text** 字幕（`$FontName` + `HH:MM:SS:FF , HH:MM:SS:FF , text`）。**3Dメッシュ STL ではない**
   - Drive アップロード時、`.stl` が `model/stl` 扱いになることがある → **SRT を必ず併送**
6. **編集ポイント（解説動画化）**
   - 素材が「カウントだけ／片側だけ」なら、そのままでは解説にならないと先に言う
   - 最低構成: 種目名 → 効く部位 → 開始姿勢 → 動作+カウント → よくあるミス → 反対側 → 回数
   - 現場音カウントは Bロール扱い。本編ナレ or テロップ設計を分ける
   - 撮影メモ: 真横全身 / 斜め45° / 足元。1カメなら 4K クロップで関節を補う
   - 公開用は 1080p30 プロキシ前提（元が HEVC 4K60 でも）
7. **成果物の置き場**
   - ローカル作業コピー + 可能なら **元動画と同じ Drive 親** に upload
   - Discord なら `MEDIA:` で SRT/STL を添付

## Pitfalls

- `google_api.py` を `/usr/bin/python` で叩く → ModuleNotFoundError。**必ず Hermes venv**
- 「STL」曖昧語: ユーザーが字幕と言っているなら Spruce。3D と言い出したら確認
- medium モデルは Pi で遅い。まず small。精度不足時だけ上げる
- ビジョン API が落ちても **ffprobe + フレーム + 音声** で編集メモは出せる。ブロック理由を書いて先に進む
- OpenAI/Groq STT キーが `.env` コメントアウトだけのことがある → ローカル whisper にフォールバック

## Deliverable shape

1. 字幕ファイル（STL + SRT 最低）
2. 全文とタイムコード表
3. 補正の有無
4. 編集で押さえるポイント（構成 / テロップ / カット / 再撮）

## References

- `references/spruce-stl-and-count-asr.md` — STL 書式・カウント ASR 補正・Drive MIME
