<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
# noon画像生成手順

> SKILL.md Step 3-2 の画像生成詳細。noonスロットのコンテンツ用に画像を1枚生成する。

## 手順

1. `$HOME/.ai-persona/visual-identity.md` を読み込み
2. **参照画像2-3枚必須**（1枚だと無視される率が高い）:
   - 必須1: メイン顔リファレンス（顔正面、最高画質）
   - 必須2: シーンに近いリファレンス（ビジネス→プレゼン系、カジュアル→日常系等）
   - 推奨3: 全身リファレンス（全身比率・服装の参照）
3. **プロンプトにBASE_CHARACTERを必ず含める**（参照画像任せにしない。二重指定で精度向上）
4. 参照画像は送信前に `sips -Z 1024` でリサイズ（8MB超はpayload過大）
5. Gemini API (gemini-2.0-flash-exp-image-generation) でimage-to-image生成
6. 出力: `content/images/YYYY-MM-DD.png` (明日の日付)
7. APIキー: `$GEMINI_API_KEY`  # macOS: `security find-generic-password -s "GEMINI_API_KEY" -a "<YOUR_ACCOUNT>" -w`

## 品質チェック

- 生成画像をReadツールで読み込み、6項目自動判定（性別/髪型/特徴/リアル感/不要テキスト/総合）
- FAILなら最大3回リトライ
- 3回リトライ後もFAIL or 生成失敗時: テキストのみ投稿にフォールバック（パイプライン停止しない）
