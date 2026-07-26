# API_NOTES.md — 使用API・制限・参照URL

## 1. 使用API

**Gemini API**（Google AI for Developers）を使用。

- 文字起こし：音声ファイルをマルチモーダル入力としてそのまま送信し、テキストを生成
- 要約：文字起こし結果（テキスト）を入力として、要約と要点をJSON形式で生成

## 2. 使用モデル・エンドポイント

| 項目 | 内容 |
|---|---|
| モデル | `gemini-3.5-flash` |
| エンドポイント | `POST https://generativelanguage.googleapis.com/v1beta/models/gemini-3.5-flash:generateContent` |
| 認証 | HTTPヘッダー `x-goog-api-key: <APIキー>`（URLクエリではなくヘッダーで送信） |
| 音声入力形式 | リクエストの `contents[].parts[]` に `inline_data: { mime_type, data(base64) }` として音声を格納 |
| 要約の出力形式 | `generationConfig.responseMimeType: "application/json"` と `responseSchema` を指定し、`{ summary, key_points[] }` のJSONを直接取得 |

### モデル選定の理由・変更履歴
- 旧モデル `gemini-2.0-flash` は2026年6月1日にシャットダウン済みのため使用不可
- 当初 `gemini-2.5-flash` を採用したが、本来の廃止予定日（2026年10月16日）より前の2026年7月上旬から、新規利用者に対して404エラー（`This model models/gemini-2.5-flash is no longer available`）が発生することを実機で確認（Google公式フォーラムでも同様の報告あり）
- 公式の音声理解ドキュメント（2026年5月18日更新）で案内されている現行モデル `gemini-3.5-flash` に切り替えて解決。音声入力・構造化出力（JSON）ともに対応していることをドキュメントで確認済み
- 参照：https://ai.google.dev/gemini-api/docs/audio （コード例が `gemini-3.5-flash` を使用）

## 3. 公式ドキュメントURL

- Gemini API 音声理解（Audio understanding）: https://ai.google.dev/gemini-api/docs/audio
- Gemini API レート制限（Rate limits）: https://ai.google.dev/gemini-api/docs/rate-limits
- Gemini API キー発行（Google AI Studio）: https://aistudio.google.com/apikey
- Firebase AI Logic 対応モデル一覧（モデル廃止スケジュールの確認用）: https://firebase.google.com/docs/ai-logic/models

## 4. 制限事項（2026年7月時点で確認できた内容）

- **レート制限**：プロジェクト単位でRPM（1分あたりリクエスト数）／RPD（1日あたりリクエスト数）／TPM（1分あたりトークン数）が設定される。無料枠の具体的な数値はGoogle AI Studioのコンソール上でプロジェクトごとに確認する必要があり、ドキュメント上の固定値は保証されない（Google公式ドキュメントより）
- **参考値（第三者情報、要現地確認）**：`gemini-2.5-flash` の無料枠はおおむね 1,500 RPD 前後とされる情報が複数の非公式サイトで報告されているが、変動する可能性があるため、実際の値は必ず自分のAI Studioコンソールで確認すること
- **音声インライン送信の上限**：リクエスト全体（base64エンコード後）でおよそ20MB程度が目安。長い音声はGoogle側の File API（アップロード方式）を使う必要があるが、本アプリでは短い音声（30秒〜2分）を想定しているためインライン送信方式のみを採用
- **課金**：無料枠を超えると従量課金（`gemini-2.5-flash` は入力・出力トークン数に応じた課金体系）。個人の検証用途であれば無料枠内に収まる想定

## 5. APIキーの取得方法

1. https://aistudio.google.com/apikey にアクセス（Googleアカウントでログイン）
2. 「Create API key」からキーを発行
3. 発行したキーをアプリ初回起動時のモーダルに貼り付け（アプリ内の `localStorage` にのみ保存され、外部には送信されない）

## 6. 注意事項

- 本アプリはAPIキーをクライアント側（ブラウザ）で保持し、ブラウザから直接Gemini APIを呼び出す構成のため、**そのAPIキーは（DevToolsのNetworkタブなどで）閲覧者本人には見える状態になる**。第三者と共有するURLにデプロイする場合は、自分専用のAPIキーを使い、他人に知られないよう注意する
- 無料枠のデータはGoogle側でモデル改善に利用される場合がある（Google公式の利用規約に準拠）。機密性の高い音声は入力しないこと（`SPEC.md` の取り扱い注意にも記載）
