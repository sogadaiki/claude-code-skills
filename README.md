# Claude Code Skills Collection

Claude Code (Anthropic公式CLI) で使えるカスタムスキル集。
日常の開発ワークフロー、コンテンツ生成、分析、自動化パイプラインを効率化するスキルを40個収録。

> **注意**: パイプライン系スキル（`mai-*`, `mei-*`, `sales-*` 等）はまだ仕上がっていない部分もあります。
> パターンや設計思想を参考にしつつ、一緒に磨き上げていきましょう！

## クイックスタート

```bash
# 1. リポをクローン
git clone https://github.com/sogadaiki/claude-code-skills.git

# 2. 使いたいスキルを ~/.claude/skills/ にコピー
cp -r claude-code-skills/skills/review ~/.claude/skills/

# 3. Claude Code で使う
# /review src/components/Auth.tsx --strict
```

## スキルカタログ

### 開発ワークフロー (12)

| スキル | 説明 | 使い方 |
|--------|------|--------|
| **[morning](skills/morning/)** | 朝の作業開始ルーティン。Issue横断・関係温度マップ・作業提案 | `/morning` |
| **[resume](skills/resume/)** | セッション間の軽量再開。restart.md + git status で即復帰 | `/resume` |
| **[plan](skills/plan/)** | 8割計画・2割実装の原則。Devil's Advocate検証付き | `/plan ユーザー認証機能の追加` |
| **[start-task](skills/start-task/)** | Issue番号からブランチ作成、安全に作業開始 | `/start-task 42` |
| **[finish-task](skills/finish-task/)** | 変更をコミット・push、Issueクローズ、restart.md更新 | `/finish-task` |
| **[finish-session](skills/finish-session/)** | セッション中断。進捗記録して次回 `/resume` で再開可能に | `/finish-session` |
| **[review](skills/review/)** | 5軸コードレビュー。`--strict` でシニアエンジニア視点 | `/review src/lib/api.ts --strict` |
| **[quick-fix](skills/quick-fix/)** | Issue取得→修正→型チェック→コミット→PR を一気通貫 | `/quick-fix 123` |
| **[investigate](skills/investigate/)** | サブエージェントでコードベース探索。メインコンテキスト温存 | `/investigate 認証フローの全体像` |
| **[learn-mistake](skills/learn-mistake/)** | ミスからルール策定→CLAUDE.md自動更新。再発防止 | `/learn-mistake Stripe金額をセント単位で送った` |
| **[knowledge-save](skills/knowledge-save/)** | 業務知見をメモリに蓄積。未来のセッションで活用 | `/knowledge-save 本番DBは読み取りレプリカ経由` |
| **[audit-skill](skills/audit-skill/)** | スキルのセキュリティ監査。プロンプトインジェクション・機密漏洩チェック | `/audit-skill write-blog --fix` |

### コンテンツ生成 (7)

| スキル | 説明 | 使い方 |
|--------|------|--------|
| **[write-blog](skills/write-blog/)** | SEO最適化ブログ記事。4グループ並列レビュー（信頼性/AI臭/ファクト/SEO） | `/write-blog AI研修の選び方` |
| **[write-sales-copy](skills/write-sales-copy/)** | ストーリー駆動のセールスページ。変革ナラティブで説得力を最大化 | `/write-sales-copy LP用セールスコピー` |
| **[revealjs](skills/revealjs/)** | reveal.jsプレゼン生成。テーマ・マルチカラム・アニメーション対応 | `/revealjs AI活用セミナー資料` |
| **[youtube-script](skills/youtube-script/)** | YouTube台本。感情曲線・オープンループ・パターン割り込み設計 | `/youtube-script FC加盟の失敗パターン` |
| **[generate-thumbnail](skills/generate-thumbnail/)** | AI画像生成+HTMLテキストオーバーレイでサムネイル作成 | `/generate-thumbnail 記事タイトル` |
| **[benchmark-lp](skills/benchmark-lp/)** | 参考サイト2-3個からオリジナルLP生成。5フェーズ設計 | `/benchmark-lp 求人LP` |
| **[edit-video](skills/edit-video/)** | Whisper文字起こし→テロップ自動焼き込み（Remotion） | `/edit-video recording.mp4` |

### 分析・監査 (6)

| スキル | 説明 | 使い方 |
|--------|------|--------|
| **[seo](skills/seo/)** | SEO分析8コマンド（audit/page/content/schema/technical/geo/plan/images） | `/seo audit https://example.com` |
| **[lp-audit](skills/lp-audit/)** | LP離脱診断。Playwrightでスクショ取得→技術/UX/コピー分析 | `/lp-audit https://example.com/lp` |
| **[deep-research](skills/deep-research/)** | エンタープライズ級リサーチ。10+ソース・引用追跡・検証付き | `/deep-research Next.js vs Remix 2026` |
| **[business-review](skills/business-review/)** | 事業計画書の多角レビュー（戦略/財務/マーケ/法務並列分析） | `/business-review 事業計画書.pdf` |
| **[weekly-review](skills/weekly-review/)** | 週次ビジネスレビュー。KPI確認+来週アクション策定 | `/weekly-review` |
| **[client-report](skills/client-report/)** | クライアント月次レポート。KPI・LINE・SNS・広告の数値まとめ | `/client-report padawan 2026-03` |

### プロジェクト管理 (5)

| スキル | 説明 | 使い方 |
|--------|------|--------|
| **[map-project](skills/map-project/)** | プロジェクト全体を3エージェント並列調査→architecture.md生成 | `/map-project` |
| **[tech-update](skills/tech-update/)** | 技術スタックのベストプラクティス更新。最新ドキュメント確認 | `/tech-update supabase` |
| **[deploy-report](skills/deploy-report/)** | レポートHTMLをCloudflare Pagesにデプロイ | `/deploy-report` |
| **[ingest-references](skills/ingest-references/)** | リファレンスファイル一括処理。トリアージ→ナレッジ抽出→Issue作成 | `/ingest-references` |
| **[delegate](skills/delegate/)** | 任意スキルをバックグラウンド委任する汎用メタスキル | `/delegate write-blog "AI研修記事"` |

### 自動化パイプライン (10)

> これらは `claude -p` でスケジューラーから自動実行されるスキルです。
> 設計パターン（マルチパス品質ゲート、事前生成アーキテクチャ等）の参考としてご活用ください。

| スキル | 説明 | 主要パターン |
|--------|------|-------------|
| **[mai-daily-post](skills/mai-daily-post/)** | AI ペルソナX自動投稿 | 5パスパイプライン、7項目品質ゲート、スロット制御 |
| **[mai-evening-session](skills/mai-evening-session/)** | 翌日コンテンツ事前生成 | Manager+Creator対話型生成、3スロット事前生成 |
| **[mai-alpha-post](skills/mai-alpha-post/)** | マイクロ投稿（調査/進捗/気づき） | 5タイプ分類、簡易品質ゲート |
| **[mai-secretary-task](skills/mai-secretary-task/)** | Discord経由の秘書タスク処理 | 6タスクタイプ、MCP統合、読み取り専用セキュリティ |
| **[sales-email-draft](skills/sales-email-draft/)** | Sheets→Gmail営業メール自動下書き | テンプレート個人化、opt-outチェック、日次上限 |
| **[mei-coo-session](skills/mei-coo-session/)** | COO日次プランニング | manifest.yaml動的生成、安全性ティア(auto/review/block) |
| **[mei-ceo-briefing](skills/mei-ceo-briefing/)** | CEO進捗フォロー+催促 | 朝/夕ブリーフィング、日数カウント圧力 |
| **[youtube-script-pipeline](skills/youtube-script-pipeline/)** | YouTube台本自動生成 | 3グループ並列レビュー、HTMLデザインシステム |
| **[soga-x-draft](skills/soga-x-draft/)** | 個人ブランドX短文ドラフト | 5パス設計、AI臭検出、引用チェック |
| **[x-long-article](skills/x-long-article/)** | X長文記事自動生成 | 4グループ品質ゲート、スライドカード生成 |

## スキルの仕組み

Claude Code のスキルは `~/.claude/skills/{skill-name}/SKILL.md` に配置するMarkdownファイルです。

```
~/.claude/skills/
├── review/
│   └── SKILL.md          # メインプロンプト
├── morning/
│   ├── SKILL.md           # メインプロンプト
│   └── references/        # 参照ファイル（大きなスキルの分割用）
│       ├── heat-map.md
│       └── health-check.md
└── deep-research/
    ├── SKILL.md
    ├── references/
    ├── templates/
    └── scripts/
```

### フロントマター

```yaml
---
name: skill-name
description: スキルの説明（Claude Code のスキル一覧に表示される）
argument-hint: [引数の説明]
---
```

### 主要な設計パターン

このコレクションで使われている設計パターン:

1. **マルチパスパイプライン** — 素材収集→設計→生成→品質ゲート→修正の5段階
2. **並列エージェントレビュー** — 複数の専門視点で同時レビュー（例: SEO + ファクトチェック + AI臭検出）
3. **Devil's Advocate** — 設計完了後に全否定検証を挟んで品質担保
4. **事前生成アーキテクチャ** — 夜間に翌日コンテンツを生成、朝は軽量publishのみ
5. **品質ゲート** — 通過条件を明示して自動的にリトライ（最大2回）
6. **関係温度マップ** — 最終接触日から対人フォロー優先度を自動判定
7. **メモリ連携** — 過去のセッション知見を次回に引き継ぐ永続化設計

## カスタマイズ

多くのスキルにはプレースホルダーが含まれています:

| プレースホルダー | 置換先 |
|---|---|
| `<YOUR_GITHUB_USER>` | あなたのGitHubユーザー名 |
| `<YOUR_OPS_REPO>` | メインの運用リポジトリ名 |
| `<YOUR_CF_PROJECT>` | Cloudflare Pagesのプロジェクト名 |
| `<YOUR_DISCORD_WEBHOOK>` | DiscordのWebhook URL |
| `<YOUR_X_ACCOUNT>` | X(Twitter)のアカウント名 |
| `<YOUR_API_ENDPOINT>` | 外部APIのエンドポイント |
| `~/.claude/knowledge/` | ナレッジファイルのパス（通常そのままでOK） |

## コントリビュート

スキルの改善提案・新規スキルの追加を歓迎します！

1. Fork → ブランチ作成 → PR
2. 新規スキルは `skills/{skill-name}/SKILL.md` に配置
3. **公開前に `/audit-skill` でセキュリティチェック** を推奨
4. PR本文にスキルの目的・使い方を記載

詳細は [CONTRIBUTING.md](CONTRIBUTING.md) を参照。

## ライセンス

MIT License - 自由に使ってください。
