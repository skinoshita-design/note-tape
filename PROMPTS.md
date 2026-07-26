# PROMPTS.md — AIに投げた重要指示（根拠URL含む）

このアプリの開発は Claude（Anthropic）とのチャットで行った。
このファイルには、(1) 開発中にClaudeへ投げた重要な指示、(2) アプリ自体がGemini APIへ投げているプロンプト、
(3) 技術的判断の根拠として参照したURL、をまとめる。

## 1. 開発時にClaudeへ投げた重要指示（要旨）

1. **課題要件の提示**：「AIアプリ開発の最初の一歩」の課題文（最短導線A〜E、見本サイトURL等）をそのまま提示し、要件を満たすアプリの実装を依頼
2. **API変更の指示**：「APIはすべてジェミニではなくクロードやクロードコードを使用するものとします。」
   → Claudeから「Claude APIは音声入力に対応していない」という技術的制約の説明を受け、対応方針（ブラウザ録音への変更 or Gemini継続）を確認された
3. **最終判断**：「なるほどではジェミニAPIで作成しましょう」→ 最初に実装したGemini API版を正式版として採用
4. **提出物一式の依頼**：`docs/SPEC.md` `docs/API_NOTES.md` `docs/TESTCASES.md` `docs/STATUS.md` `docs/PROMPTS.md` `README.md` `screenshots/` アプリコード一式（GitHub Pages公開形式）の作成を依頼

## 2. アプリ内でGemini APIに投げているプロンプト（実装内容）

### 2-1. 文字起こし用プロンプト（`index.html` 内 `transcriptPrompt`）

> これは「（講義名）」という講義・会話の音声です。この音声を一字一句忠実に日本語で文字起こししてください。フィラーや言い淀みも自然な範囲で構いません。文字起こし本文のみを出力し、前置きや見出しは付けないでください。

音声データは `inline_data`（base64、`mime_type` はアップロードされたファイルのMIMEタイプ）として同じリクエストに含めて送信している。

### 2-2. 要約用プロンプト（`index.html` 内 `summaryPrompt`）

> 以下は（講義名）の音声の文字起こしです。この内容を読み、150〜250字程度の日本語の要約（summary）と、重要な要点を3〜5個の箇条書き（key_points）で抽出してください。

出力は `generationConfig.responseSchema` で `{ summary: string, key_points: string[] }` のJSON形式を強制し、パースの安定性を確保している。

## 3. 技術判断の根拠として参照したURL

| 参照内容 | URL |
|---|---|
| Gemini API 音声理解（対応形式・使い方） | https://ai.google.dev/gemini-api/docs/audio |
| Gemini API レート制限の考え方 | https://ai.google.dev/gemini-api/docs/rate-limits |
| Gemini APIキーの発行元 | https://aistudio.google.com/apikey |
| `gemini-2.0-flash` シャットダウン情報（モデル選定の根拠） | https://firebase.google.com/docs/ai-logic/models |
| `gemini-2.5-flash` の早期404エラー報告（実機テストで遭遇したエラーの根拠） | https://discuss.ai.google.dev/t/gemini-2-5-flash-and-gemini-2-5-flash-lite-returning-404-no-longer-available-today-july-9-contradicts-oct-16-2026-shutdown-date/174267 |
| `gemini-3.5-flash` への切り替え根拠（音声理解ページのコード例） | https://ai.google.dev/gemini-api/docs/audio |
| Claude APIが音声ファイルを直接文字起こしできない旨の確認 | https://ai.google.dev/gemini-api/docs/audio（Geminiとの対比として） および一般的なAnthropic製品仕様の確認 |

## 4. 見本サイト（課題側から提示）

- https://lec-audio-ai-kshxl3jm.manus.space/ （最小機能の完成イメージとして課題文中で提示されたもの）
