---
name: generate-thumbnail
description: 記事・コンテンツのサムネイル画像を生成。記事の目的・ターゲット・メッセージを読み取り、Nano Banana 2 (Gemini 3.1 Flash Image) で背景画像を生成→HTMLテキストオーバーレイ→Playwrightでスクリーンショット。
argument-hint: [記事ファイルパス or テーマ説明]
---

# Generate Thumbnail: $ARGUMENTS

## 参照ナレッジ
- `~/.claude/knowledge/design-standards.md` — **統合デザイン基準（最上位ルール。Section 3 NG集、Section 7 画像生成ルール）**

## Multi-Pass Design

既存ワークフロー（Phase 0-7）を多段階原則にマッピング。

| Pass | 内容 | 対応Phase |
|------|------|-----------|
| **Pass 1: 素材収集** | コンテキスト分析（目的/ターゲット/コアメッセージ/感情/差別化の5項目言語化） | Phase 1 |
| **Pass 2: 設計 + 自己批判** | 視覚コンセプト設計→Nano Banana 2プロンプト構築。「コアメッセージが1つに絞れているか？」「テキスト配置の空間を確保しているか？」「プロンプト品質チェックリスト全項目OK?」を自己批判 | Phase 2-3 |
| **Pass 3: 生成** | Nano Banana 2 API呼び出し→HTMLテキストオーバーレイ→Playwrightスクリーンショット | Phase 4-6 |
| **Pass 4: 品質ゲート** | 以下のチェック項目で最終画像を検証 | Phase 6後 |
| **Pass 5: 修正** | FAIL項目のみ修正（プロンプト調整 or HTMLテキスト調整）→再スクショ。2回FAILでPhase 1の分析が間違っている→再分析 | Phase 7前 |

### Pass 4 品質ゲート項目（サムネイル固有）

| # | チェック項目 | FAIL基準 |
|---|------------|---------|
| 1 | テキスト可読性 | 背景と文字のコントラスト不足で読めない |
| 2 | コアメッセージ伝達 | サムネを3秒見て「何の記事か」分からない |
| 3 | ビジュアルメタファー | 背景画像がテーマと無関係 / 抽象的すぎる |
| 4 | テキスト量 | メインコピー3行以上（2行以内が目標） |
| 5 | 解像度 | 画像がぼやけている / 圧縮アーティファクト |

## 概要

記事やコンテンツのサムネイル画像を生成する。
最重要なのは「何の記事か」「誰に届けるか」「何を伝える画像か」を深く理解し、それを1枚の画像に翻訳すること。

## Phase 0: モード選択（最初に1回だけ聞く）

「確認なしで一気に生成しますか？」
- **自動モード**: 分析→プロンプト→生成→HTML→スクショまで一切確認なしで完走。最終結果だけ見せる
- **確認モード**: 各Phaseでユーザー確認を挟む。初めて使う場合やこだわりたい場合向け

以降、自動モードなら全Phaseをノンストップで実行する。

## 手順

### Phase 1: コンテキスト分析（最重要）

記事ファイルまたはテーマ説明から以下を抽出する。ここの精度が画像の質を決める。

1. **記事を読み込む**（ファイルパスが指定された場合は全文読む）
2. **以下の5項目を言語化する**:

```
[目的] この記事は何を売る/伝えるのか（1文）
[ターゲット] 誰がこのサムネを見てクリックするのか（具体的なペルソナ）
[コアメッセージ] サムネで伝えるべきたった1つのメッセージ
[感情] 見た人にどんな感情を起こしたいか（驚き/憧れ/共感/焦り等）
[差別化] 同ジャンルの他記事サムネと何が違うべきか
```

**確認モードの場合のみ** ユーザー確認を取る。自動モードなら分析結果をそのまま次Phaseに渡す。

### Phase 2: 視覚コンセプト設計

Phase 1の分析結果を「目に見える絵」に翻訳する。

考えるべきこと:
- コアメッセージを**1つのビジュアルメタファー**に変換する
  - 例: 「AI社員9人のチーム」→「9体のキャラが各々の専門道具を持つ集合絵」
  - 例: 「法的リスクの見える化」→「赤い警告が浮かぶダッシュボード画面」
  - 例: 「偉人が現代に蘇る」→「和服の偉人がスマホを持つギャップ絵」
- ターゲットの文化圏に合うスタイルを選ぶ
  - 経営者向け → 洗練、プロフェッショナル、chibi可（親しみ）
  - エンジニア向け → テック感、回路、ターミナル風
  - 一般消費者向け → 明るい、カジュアル、写真寄り
- **テキストを後から載せる前提で構図を考える**
  - 左側1/3はテキスト領域として空けるか暗くする
  - メインビジュアルは中央〜右寄り

視覚コンセプトを1〜2文で言語化する。
**確認モードの場合のみ** ユーザーに提示。自動モードならそのままPhase 3へ。

### Phase 3: Nano Banana 2 プロンプト生成

#### プロンプト構造（この順序で組む）

```
[スタイル指定], [メイン被写体の詳細], [構図・配置], [背景・環境], [照明・色彩], [技術仕様]
```

#### プロンプト品質チェックリスト

生成前に以下を全て満たしているか確認:

- [ ] スタイルが明示されている（chibi, photorealistic, flat illustration, etc.）
- [ ] メイン被写体が具体的（「people」ではなく「9 chibi characters each holding different professional tools」）
- [ ] 各要素の位置関係が指定されている（centered, left-aligned, arranged in two rows）
- [ ] 背景が具体的（「dark background」ではなく「deep navy to purple gradient background with subtle glow effects」）
- [ ] 色パレットが明示されている（dominant: navy #0f172a, accent: cyan #06b6d4, warm: amber #f59e0b）
- [ ] テキスト配置を考慮した空間がある（left third darker for text overlay）
- [ ] 画像内に日本語テキストを入れる場合、正確に文字列を指定している（Nano Banana 2は日本語テキストレンダリングが高精度）
- [ ] ネガティブ要素がない（「no text」等は書かない。描いてほしいものだけ書く）

#### 成功パターンの参考

**Playbook記事（AI社員9人チーム）で採用されたv8の推定プロンプト構造:**
```
Chibi anime style digital illustration, 1792x1024,
9 unique chibi characters standing in two rows representing different professional roles:
- armored knight with glowing sword (security)
- girl with purple hair holding a book (knowledge)
- boy in lab coat with laptop (developer)
- girl with pink hair and headphones (designer)
- boy in black hoodie with briefcase (hacker/engineer)
- white-haired central figure with glowing hands (leader/AI core)
- boy with glasses and laptop (analyst)
- boy with blue hair and goggles (tech specialist)
- boy in casual jacket with cap (project manager),
arranged against a deep navy to purple gradient background,
vibrant neon blue and purple lighting from below,
clean composition with slight spacing between characters,
game art style, high detail on accessories and expressions,
dark atmospheric background with color glow effects
```

このレベルの具体性が必要。抽象的なプロンプトは抽象的な結果しか返さない。

### Phase 4: Nano Banana 2 API呼び出し

Gemini APIキーは macOS Keychain から取得する。

#### 画像生成 + ダウンロード（1コマンドで実行）

```bash
GEMINI_KEY=$(security find-generic-password -s "GEMINI_API_KEY" -a "claude-ops" -w) && \
RESPONSE=$(curl -s "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image-preview:generateContent?key=${GEMINI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "ここにPhase 3で組んだプロンプト"}]}],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"],
      "imageConfig": {
        "aspectRatio": "16:9",
        "imageSize": "2K"
      }
    }
  }') && \
echo "$RESPONSE" | python3 -c "
import sys, json, base64
d = json.load(sys.stdin)
if 'candidates' not in d:
    print('ERROR:', d.get('error', {}).get('message', json.dumps(d))); sys.exit(1)
for part in d['candidates'][0]['content']['parts']:
    if 'text' in part:
        print('Model text:', part['text'])
    if 'inlineData' in part:
        img = base64.b64decode(part['inlineData']['data'])
        with open('$OUTPUT_PATH', 'wb') as f:
            f.write(img)
        print('Saved: $OUTPUT_PATH (' + str(len(img)) + ' bytes)')
"
```

- アスペクト比: `16:9`（横長、サムネに最適。他: `1:1`, `3:2`, `4:3`, `9:16` 等14種類対応）
- imageSize: `2K`（高解像度。`512px`, `1K`, `4K` も選択可）
- **responseModalities に必ず `["TEXT", "IMAGE"]` を指定**（IMAGE単独は非対応）
- レスポンスは base64 エンコードされた画像データが直接返る（URLダウンロード不要）
- **Nano Banana 2 は日本語テキストを高精度で画像内に描画できる**。サムネタイトルを画像に直接焼き込む選択肢もある

#### 背景画像の圧縮（必須ステップ）

生成画像が大きい場合、HTMLに使う用にJPEG圧縮版を作る:

```bash
sips -Z 1792 "$OUTPUT_PATH" --out "${OUTPUT_DIR}/thumbnail-bg.jpg" -s format jpeg -s formatOptions 85
```

HTMLの `<img>` には圧縮版(.jpg)を指定する。元のPNGは高解像度バックアップとして保持。

**確認モード**: 生成した画像をユーザーに見せて確認。修正が必要ならプロンプト調整して再生成（最大2回）。
**自動モード**: 画像を保存してそのままPhase 5へ進む。最終結果でまとめて見せる。

### Phase 5: HTMLテキストオーバーレイ

背景画像が確定したら、テキストを載せたHTMLを作成する。

#### フォント指定（重要）

**外部フォント（Google Fonts等）は使わない。** Playwrightのスクリーンショットでタイムアウトの原因になる。
macOSのシステムフォントを使う:

```css
font-family: 'Hiragino Sans', 'Hiragino Kaku Gothic ProN', sans-serif;
```

#### HTMLテンプレート

```html
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 1280px;
    height: 670px;
    overflow: hidden;
    position: relative;
    background: #000;
  }
  .bg-image {
    position: absolute;
    top: 0; left: 0;
    width: 100%; height: 100%;
    object-fit: cover;
    opacity: 0.7;
  }
  .overlay {
    position: absolute;
    top: 0; left: 0;
    width: 100%; height: 100%;
    background: linear-gradient(90deg,
      rgba(0,0,0,0.85) 0%,
      rgba(0,0,0,0.6) 45%,
      rgba(0,0,0,0.1) 100%);
  }
  .content {
    position: absolute;
    top: 0; left: 0;
    width: 100%; height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding-left: 60px;
    font-family: 'Hiragino Sans', 'Hiragino Kaku Gothic ProN', sans-serif;
  }
  /* テキスト要素は記事に合わせてカスタマイズ */
</style>
</head>
<body>
  <img class="bg-image" src="thumbnail-bg.jpg" />
  <div class="overlay"></div>
  <div class="content">
    <!-- 記事に応じたテキスト要素をここに配置 -->
  </div>
</body>
</html>
```

#### テキスト要素の設計指針
- **数字を大きく**: 数字は訴求力が高い（「9」「3つ」「0円」）
- **メインコピーは2行以内**: それ以上は読まれない
- **サブタイトルは英語も可**: 英語はデザイン的に映える
- **タグ/バッジ**: 具体的な数値（9 Agents, 17 Skills等）は信頼感を出す
- **色**: アクセントカラーは1〜2色まで。背景画像の色相と合わせる

**確認モード**: HTMLをPlaywrightで表示してユーザーに見せて確認。
**自動モード**: そのままPhase 6へ。

### Phase 6: Playwrightでスクリーンショット

#### 手順（簡易HTTPサーバー経由）

Playwright MCPは `file://` プロトコルをブロックする。ローカルHTTPサーバーを経由する:

```bash
# 1. HTMLと画像があるディレクトリでHTTPサーバーを起動
cd "$OUTPUT_DIR" && python3 -m http.server 8765 &
SERVER_PID=$!
sleep 1

# 2. Playwright CLIでスクリーンショット
npx playwright screenshot \
  --viewport-size "1280,670" \
  --wait-for-timeout 3000 \
  "http://localhost:8765/thumbnail-overlay.html" \
  "$OUTPUT_DIR/thumbnail-final.png"

# 3. サーバー停止
kill $SERVER_PID
```

**Playwright MCP (`browser_take_screenshot`) ではなく、Playwright CLI (`npx playwright screenshot`) を使う。**
理由: MCP版はタイムアウトが5秒固定で、画像を含むHTMLでは頻繁にタイムアウトする。
CLI版は `--wait-for-timeout` でレンダリング待機時間を自由に設定できる。

#### Playwright CLIが使えない場合のフォールバック

```bash
# npx playwright screenshot が失敗した場合:
npx playwright install chromium  # ブラウザ未インストールの可能性
npx playwright screenshot ...     # 再実行
```

保存先: 背景画像と同じディレクトリに `thumbnail-final.png`

### Phase 7: 完了報告

以下を報告:
- 最終画像パス（Read toolで表示する）
- 使用したNano Banana 2プロンプト（再利用のため記録）
- Model text（API側のテキスト応答）
- HTMLファイルパス（テキスト変更時に再利用可能）

## 制約

- Phase 1のコンテキスト分析を省略しない（ここを飛ばすと何枚生成しても当たらない）
- 再生成は最大2回。3回目が必要ならPhase 1の分析が間違っている
- APIキーは `~/.claude/secrets.env` の `GEMINI_API_KEY`。gitリポジトリ外なので安全
- HTMLではシステムフォント使用（Google Fontsはタイムアウトの原因）
- 生成画像が大きい場合はJPEG圧縮してからHTMLに使う
- スクリーンショットは `npx playwright screenshot`（MCP版はタイムアウトしやすい）
- `file://` は使えない。必ずローカルHTTPサーバー経由
- Nano Banana 2は日本語テキストを高精度で画像内に描画できるため、シンプルなタイトルのみのサムネイルなら画像生成だけで完結も可（Phase 5, 6スキップ可）
- 全生成画像に SynthID ウォーターマークが自動埋込される（Googleのポリシー）
