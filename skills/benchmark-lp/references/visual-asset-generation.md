# Visual Asset Generation (Phase 3 詳細)

**最重要フェーズ**。ここがAI生成っぽさを消す核心。

## Nano Banana 2 画像生成

以下のケースで Nano Banana 2 (Gemini 3.1 Flash Image) を使う:

| 用途 | プロンプト方針 | アスペクト比 | imageSize |
|------|-------------|------------|-----------|
| **ヒーロー背景** | 業界テーマ + 抽象的 + ブランドカラー | 16:9 | 2K |
| **セクション背景** | テクスチャ系 / パターン系 | 16:9 | 1K |
| **特徴アイコン** | ミニマル + ブランドカラー + 3D風 | 1:1 | 1K |
| **イラスト** | ビジネスシーン + ブランドトーン | 4:3 | 2K |

**生成手順:**
```bash
GEMINI_KEY=$(security find-generic-password -s "GEMINI_API_KEY" -a "claude-ops" -w) && \
curl -s "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image-preview:generateContent?key=${GEMINI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "[詳細なプロンプト]"}]}],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"],
      "imageConfig": {
        "aspectRatio": "16:9",
        "imageSize": "2K"
      }
    }
  }' | python3 -c "
import sys, json, base64
d = json.load(sys.stdin)
for part in d['candidates'][0]['content']['parts']:
    if 'inlineData' in part:
        img = base64.b64decode(part['inlineData']['data'])
        with open('public/images/hero-bg.png', 'wb') as f:
            f.write(img)
        print('Saved: public/images/hero-bg.png')
"
```

**プロンプトの原則:**
- 「抽象的」「グラフィック」「ミニマル」方向が安全
- ブランドカラーをプロンプトに含める
- Nano Banana 2は日本語テキストを高精度で画像内に描画できるため、ヒーローセクションのキャッチコピーを画像に直接焼き込む選択肢もある
- `responseModalities` には必ず `["TEXT", "IMAGE"]` を含める（IMAGE単独は非対応）

## 背景動画（オプション）

クライアントが動画素材を持っている場合、または無料素材（Pexels, Pixabay）を使う場合:

```tsx
// 背景動画コンポーネントパターン（BizBench参考）
<div className="relative overflow-hidden">
  <video
    autoPlay
    muted
    loop
    playsInline
    className="absolute inset-0 w-full h-full object-cover"
  >
    <source src="/videos/hero-bg.mp4" type="video/mp4" />
  </video>
  {/* オーバーレイ */}
  <div className="absolute inset-0 bg-black/50" />
  {/* コンテンツ */}
  <div className="relative z-10">...</div>
</div>
```

動画がない場合は **CSSグラデーション + ぼかしオブジェクト** で代替:

```tsx
{/* フォールバック: 動的背景 */}
<div className="absolute inset-0 bg-gradient-to-br from-[primary] via-[dark] to-[secondary]" />
<div className="absolute top-1/4 -left-20 w-96 h-96 bg-[primary]/20 rounded-full blur-[120px] animate-pulse-slow" />
<div className="absolute bottom-1/4 -right-20 w-96 h-96 bg-[secondary]/20 rounded-full blur-[120px] animate-pulse-slow" />
```

## アニメーション定義

全LPに共通で入れるカスタムアニメーション:

```css
@keyframes fade-in-up {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}
@keyframes pulse-slow {
  0%, 100% { opacity: 0.2; transform: scale(1); }
  50% { opacity: 0.3; transform: scale(1.05); }
}
@keyframes slide-in-left {
  from { opacity: 0; transform: translateX(-30px); }
  to { opacity: 1; transform: translateX(0); }
}
@keyframes slide-in-right {
  from { opacity: 0; transform: translateX(30px); }
  to { opacity: 1; transform: translateX(0); }
}
```

**必須ルール**: 要素ごとに `animationDelay` をずらす（100ms刻み）。全要素同時出現は禁止。
