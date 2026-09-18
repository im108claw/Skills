---
name: coursework-quiz-drills
description: "Use when turning quiz screenshots into one-by-one drills."
version: 1.0.0
---

# Coursework quiz drills

Build **1問ずつ** drills from course quiz sources without changing problem form or wording.

## Triggers

- Drive folder of myFIT / LMS **テスト結果** screenshots
- User asks to quiz one question at a time, randomize order/choices, prioritize weak items
- User asks for a Notion **HTML block** quiz app

## Hard rules (とーや)

1. Keep **stem, choices, single/multi type** faithful to source.
2. Shuffle **order of questions** and **order of choices** only when asked.
3. Weak-priority = items where source shows 学習者回答 ≠ 正解 (or later wrong answers in-session).
4. OCR/vision is lossy — prefer fields labeled **正解**; treat original images as final authority.

## Pipeline

1. **Acquire**
   - Drive folder: Hermes venv Python + `google-workspace` `google_api.py`
   ```bash
   PY="$HOME/.hermes/hermes-agent/venv/bin/python"
   GAPI="$HOME/.hermes/skills/productivity/google-workspace/scripts/google_api.py"
   "$PY" "$GAPI" drive search "'<FOLDER_ID>' in parents and trashed=false" --raw-query --max 50
   "$PY" "$GAPI" drive download <FILE_ID> --output ~/tmp/.../name.JPG
   ```
2. **Read text**
   - Prefer `vision_analyze` when available.
   - Fallback: `tesseract img -l jpn --psm 6` (optional preprocess + `jpn+eng` / PSM 4).
3. **Normalize bank JSON**
   - Fields: `id, round, type(single|multi), points, stem, choices[], answer[], weak?`
4. **Deliver**
   - Chat: one question → grade → next.
   - Or self-contained HTML app (inline CSS/JS).
5. **Notion HTML block**
   - Follow skill `notion-workspace-ops` → `references/html-block-embed.md`
   - Scratch parent = Inbox original synced block `3b1fdf11-aa03-8034-8d2a-df7e37ae4ada`（page `3adfdf11-aa03-80ad-8b82-e4acfc605220` 直下 NG）

## Pitfalls

- System `python3` may lack `googleapiclient` — use Hermes agent venv.
- Do not invent missing choices; re-OCR or open the source image.
- free Notion: avoid relying on `ntn files create < file.html` multipart; use external_url import path in html-block-embed.md.

## Related

- `notion-workspace-ops` / `references/html-block-embed.md`
- `google-workspace` Drive download
- FIT coursework policy lives in Notion skill `fit-coursework` (user-owned; needs `hermes curator adopt fit-coursework` before agent edits)
