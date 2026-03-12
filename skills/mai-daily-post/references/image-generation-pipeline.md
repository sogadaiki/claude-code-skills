<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
# 画像生成パイプライン詳細

## 4.5.2 シーン選定

1. `$HOME/.ai-persona/visual-identity.md` を読み込む（なければスキップ）
2. シーンライブラリからランダム候補を3つ選出
3. 除外フィルタ:
   - `state.json` の `posting.last_image_scene` と同一 → 除外
   - `activity/` の直近7日ログに `image_scene:` タグがあれば使用済みシーンを除外
4. 投稿テーマとのマッチング:
   - AI活用/作業系テーマ → `cafe-laptop`, `office-desk` を優先
   - 共感/日常テーマ → `home-relax`, `morning-coffee` を優先
   - 外出/アクティブテーマ → `outdoor-walk`, `bookstore` を優先
5. 服装は季節に合わせて選定（3月=春の服装バリエーション優先）

## 4.5.3 画像生成（image-to-image、参照画像2-3枚）

**参照画像選定ルール**（1枚だと無視される率が高いため2-3枚必須）:
- **必須1**: メイン顔リファレンス（顔正面、最高画質）
- **必須2**: シーンに近いリファレンス（ビジネス→プレゼン系、カジュアル→日常系等）
- **推奨3**: 全身リファレンス（全身比率・服装の参照）

参照画像は送信前に1024px以下にリサイズ（`sips -Z 1024`）。

```bash
# 1. リファレンス画像2-3枚をbase64読み込み
REF_DIR="<YOUR_REFERENCE_IMAGE_DIR>"

# 必須: 顔リファレンス
sips -Z 1024 "${REF_DIR}/face-reference.png" --out /tmp/ref-face.png 2>/dev/null
REF_FACE=$(base64 < /tmp/ref-face.png)

# 必須: シーン別リファレンス（変数 $SCENE_REF_FILE はStep 4.5.2で決定）
sips -Z 1024 "${REF_DIR}/${SCENE_REF_FILE}" --out /tmp/ref-scene.png 2>/dev/null
REF_SCENE=$(base64 < /tmp/ref-scene.png)

# 推奨: 全身リファレンス（全身ポーズが必要な場合のみ）
sips -Z 1024 "${REF_DIR}/full-body-reference.png" --out /tmp/ref-body.png 2>/dev/null
REF_BODY=$(base64 < /tmp/ref-body.png)

# 2. プロンプト組み立て（BASE_CHARACTER + シーン + 服装）
#    visual-identity.md の BASE_CHARACTER を必ず含める（参照画像任せにしない）
#    冒頭に「Generate a photo of this same person...」を付与
#    動作確認済みテンプレートは一文字も変えない

# 3. Gemini API送信（参照画像2-3枚 + テキストプロンプト）
GEMINI_API_KEY=$(security find-generic-password -s "GEMINI_API_KEY" -a "claude-ops" -w)

PAYLOAD=$(jq -n \
  --arg face "$REF_FACE" \
  --arg scene "$REF_SCENE" \
  --arg prompt "$PROMPT" \
  '{
    "contents": [{"parts": [
      {"inline_data": {"mime_type": "image/png", "data": $face}},
      {"inline_data": {"mime_type": "image/png", "data": $scene}},
      {"text": $prompt}
    ]}],
    "generationConfig": {"responseModalities": ["IMAGE", "TEXT"]}
  }')

# 全身リファレンスを追加する場合は parts 配列に3つ目のinline_dataを挿入

RESPONSE=$(curl -s "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp-image-generation:generateContent?key=${GEMINI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD" \
  --max-time 120)

# Extract image
echo "$RESPONSE" | jq -r '.candidates[0].content.parts[] | select(.inlineData) | .inlineData.data' | base64 -d > content/images/YYYY-MM-DD.png
```

**リファレンス**: `<YOUR_REFERENCE_IMAGE_DIR>` から2-3枚選択
**出力先**: `content/images/YYYY-MM-DD.png`

## 4.5.4 画像品質チェック（サブエージェント自動検証）

生成画像をReadツールで読み込み、以下6項目を自動判定:

| 項目 | 検証内容 | NG例 |
|------|---------|------|
| 性別 | 設定通りであること | 性別が異なる |
| 髪型 | 設定通りであること | 設定と異なる髪型 |
| 特徴 | 顔の雰囲気がリファレンスと一致 | 別人に見える |
| リアル感 | 不自然な描画がないか | 手指の崩れ、uncanny valley |
| 不要テキスト | 画像内に意図しないテキストがないか | ロゴ、文字が混入 |
| 総合 | 全体的な品質 | ぼやけ、低解像度 |

**判定ルール**:
1. 全項目PASS → 採用
2. 1つでもFAIL → 再生成（最大3回リトライ）
3. 3回リトライ後もFAIL → テキストのみ投稿に進む
4. **画像なしでもパイプラインは止まらない**

追加チェック:
- ファイルサイズ: 5MB以下（X API media upload制限）
- Gemini APIエラー（セーフティフィルター含む）→ テキストのみ投稿に進む

**ログ出力**: `[Step 4.5] 画像生成成功: content/images/YYYY-MM-DD.png ({size}KB, scene={scene}, refs={N}枚, retry={N})` or `[Step 4.5] 画像生成失敗: {reason}。テキストのみ投稿へ`

## X投稿時の画像添付コマンド

**画像付き単一ツイート:**
```bash
node $HOME/.ai-persona/scripts/x-post.mjs post "${text}" --media content/images/YYYY-MM-DD.png
```

**画像付きスレッド（1ツイート目に画像添付）:**
```bash
node $HOME/.ai-persona/scripts/x-post.mjs thread "${tweet1}" "${tweet2}" ... "${tweetN}" --media content/images/YYYY-MM-DD.png
```
