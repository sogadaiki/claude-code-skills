# Benchmark Analysis Checklist (Phase 1 詳細)

## 1.0 PageGrab分析データの読み込み（あれば自動化）

PageGrabのLP Analysis JSONが提供されている場合、Phase 1.2を自動化できる:

```
手順:
1. ~/Downloads/pagegrab/analysis/ から該当JSONを読み込む
2. 各ベンチマークの配色・フォント・画像データを以下の表に自動転記
3. ダウンロード済み画像（~/Downloads/pagegrab/images/{slug}/）をビジュアルリファレンスとして参照
```

**JSONがある場合のPhase 1.2出力例**:
```markdown
### Benchmark 1: [サイト名]（PageGrab自動抽出）
- **配色**: #333333（テキスト）, #00BFCB（アクセント）, #F94CBE（CTA）
- **フォント**: Zen Maru Gothic 500/700（Google Fonts）, Helvetica Neue 400/700（システム）
- **画像**: 63枚（imgタグ）+ 15枚（CSS背景）= 78枚。タイトル類は画像テキスト
- **特記**: セクションタイトルがPNG画像。HTMLフォントではなくデザインツール制作
```

PageGrabデータがない場合はスキップし、手動分析（1.1以降）を実施する。

## 1.1 セクション構成マッピング

各サイトのセクション順序を表にする:

```
| # | Benchmark 1 | Benchmark 2 | Benchmark 3 |
|---|-------------|-------------|-------------|
| 1 | Hero + CTA  | Video Hero  | Hero + Form |
| 2 | 実績数値    | Pain Points | Before/After|
| 3 | Features    | Solution    | Features    |
| ...                                         |
```

## 1.2 デザインパターン抽出

各サイトから以下を抽出:

| 要素 | 抽出内容 |
|------|---------|
| **配色** | プライマリ/セカンダリ/背景のHEXコード |
| **タイポグラフィ** | フォント、サイズ比率、ウェイト |
| **レイアウト** | グリッド構成、余白比率、セクション高さ |
| **ビジュアル効果** | グラデーション、ブラー、シャドウ、アニメーション |
| **CTA配置** | 位置、頻度、デザイン |
| **画像の使い方** | ヒーロー画像、背景、アイコン、イラスト |

## 1.3 「強い要素」ピックアップ

各サイトから最も効果的な要素を2-3個選定:
```
Benchmark 1: ヒーローの動画背景が印象的、CTA色のコントラストが強い
Benchmark 2: 数値セクションのアニメーションが効果的、FAQ構成が良い
Benchmark 3: Before/Afterの見せ方がうまい、社会的証明の配置が良い
```

## PageGrab LP Analysis JSONの構造

```json
{
  "fonts": {
    "used": [{ "family": "...", "weight": "700", "size": "32px", "count": 8 }],
    "googleFontsUrls": ["https://fonts.googleapis.com/css2?family=..."],
    "fontFaceDeclarations": ["@font-face { ... }"]
  },
  "colors": {
    "palette": [{ "hex": "#FF6B35", "rgb": "rgb(255,107,53)", "property": "color, background-color", "count": 45 }]
  },
  "images": [{ "url": "...", "type": "img|background", "alt": "...", "width": 1920, "height": 1080, "localPath": "..." }]
}
```

**活用法**:
- `fonts.used` → Phase 2 タイポグラフィ選定の参考（ベンチマークと同じ or あえて差別化）
- `colors.palette` → Phase 2 デザインシステムのカラーパレット基盤
- `images` → ヒーロー画像サイズの参考、DALL-Eプロンプトの解像度指定
- `googleFontsUrls` → そのまま採用 or 類似フォント選定のヒント
- ダウンロード済み画像 → ビジュアルリファレンスとしてyui-ux-designerに渡す
