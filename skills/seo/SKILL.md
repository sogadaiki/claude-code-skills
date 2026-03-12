---
name: seo
description: SEO分析・最適化。サイト監査、ページ分析、コンテンツ品質、スキーマ生成、AI検索最適化。
argument-hint: [audit|page|content|schema|technical|geo|plan|images] <URL or ファイルパス>
---

# SEO Analysis: $ARGUMENTS

## 参照ナレッジ
まず `~/.claude/knowledge/seo-best-practices.md` を読み込む（E-E-A-T基準、Core Web Vitals閾値、非推奨スキーマ一覧、GEO最適化等）
- `~/.claude/knowledge/aeo-strategy.md` — AEO/GEO分析時に読み込み

## サブコマンド

引数の最初のワードでモードを判定:

### `/seo audit <URL>`
サイト全体のSEO監査。seo-specialistエージェントに委譲して並列分析:
1. WebFetch でHTMLを取得
2. 技術的SEO（8カテゴリ）を分析
3. Schema検出・検証
4. コンテンツ品質チェック
5. 画像最適化チェック
6. SEOヘルススコア（0-100）を算出
7. 優先度別アクションプラン出力

### `/seo page <URL>`
単一ページの詳細分析:
- title（50-60文字）、meta description（150-160文字）
- H1が1つだけか、見出し階層スキップなし
- キーワード密度（1-3%自然分布）
- 内部/外部リンク品質
- OGP・Twitterカード
- 画像（alt、サイズ、フォーマット、lazy loading）
- Schema検出・推奨
- Core Web Vitalsパターン検出

### `/seo content [URL or ファイルパス]`
E-E-A-T品質評価:
- Experience / Expertise / Authoritativeness / Trustworthiness スコアリング
- 語数チェック（ページ種別ごとの基準）
- 可読性メトリクス
- AI生成コンテンツの兆候検出
- 著者情報・信頼シグナル
- AI引用適性（GEO）評価

### `/seo schema <URL or ファイルパス>`
構造化データ分析・生成:
1. 既存スキーマ検出（JSON-LD / Microdata / RDFa）
2. バリデーション（7点チェックリスト）
3. 非推奨スキーマ警告（HowTo、FAQ制限等）
4. 不足スキーマの特定
5. 実装用JSON-LDコード生成

### `/seo technical <URL>`
テクニカルSEO監査（8カテゴリ）:
1. クロール可能性
2. インデックス可能性
3. セキュリティヘッダー
4. URL構造
5. モバイル最適化
6. Core Web Vitals（LCP/INP/CLS）
7. 構造化データ
8. JSレンダリング

### `/seo geo <URL or ファイルパス>`
AI検索最適化（GEO）分析:
- 引用適性スコア（134-167語ブロック、最初の40-60語）
- 構造的可読性（見出し、リスト、表）
- マルチモーダルコンテンツ有無
- 権威シグナル（著者、公開日、出典）
- AIクローラーアクセシビリティ（robots.txt、llms.txt）
- AI Overview / ChatGPT / Perplexity での表示最適化提案

### `/seo plan`
SEO戦略プランニング:
1. Discovery（ビジネスコンテキスト、ターゲット、KPI）
2. 競合分析（上位サイトのコンテンツ・技術・権威性）
3. サイト構造設計（URL、コンテンツピラー、情報階層）
4. コンテンツ戦略（コンテンツギャップ、E-E-A-T考慮）
5. 技術基盤（ホスティング、スキーマ、パフォーマンス）
6. 12ヶ月ロードマップ（4フェーズ）

### `/seo images <URL or ディレクトリ>`
画像最適化分析:
- alt属性（10-125文字、説明的）
- ファイルサイズ（サムネ<50KB、コンテンツ<100KB、ヒーロー<200KB）
- フォーマット（WebP/AVIF推奨）
- レスポンシブ（srcset/sizes）
- 遅延読み込み（ATF=eager、BTF=lazy）
- 寸法属性（CLS防止）
- LCP画像の優先読み込み

## 実行方法

- URLが指定された場合: WebFetch + Playwright（webapp-testing）で取得・分析
- ファイルパスが指定された場合: ローカルファイルを直接読み込み
- 大規模な監査（audit, technical）: seo-specialistエージェントをサブエージェントとして起動

## 出力フォーマット

全サブコマンド共通:
1. **スコア**: カテゴリ別スコアとSEOヘルススコア（0-100）
2. **Critical**: インデックスブロッカー、セキュリティ問題
3. **High**: ランキングに影響する問題
4. **Medium**: 最適化の機会
5. **Low**: 改善提案
6. **Quick Wins**: 30分以内で対応可能な高インパクト施策
7. **コードスニペット**: 実装用JSON-LD、metaタグ等

## 禁止事項
- 非推奨スキーマを推奨しない（HowTo, SpecialAnnouncement等）
- FIDに言及しない（INPに置換済み）
- 語数をランキング要因として扱わない（カバレッジの下限）
