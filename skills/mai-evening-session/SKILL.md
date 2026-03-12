<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
---
name: evening-session
description: "Automation-only: Integrated evening session (manager analysis + next-day content pre-generation). Run via orchestrator daily at 23:30 with `claude -p`. Do NOT trigger for interactive sessions."
argument-hint: "[--weekly] [--dry-run]"
---

# Evening Session -- 統合イブニングセッション

マネージャーペルソナとクリエイターペルソナの1日の締めくくりを**Discordで可視化しながら**、
明日の投稿コンテンツを事前生成し、学習サイクルを回す統合セッション。

## Multi-Pass Design

| Pass | 内容 | 対応Step |
|------|------|----------|
| **Pass 1: 素材収集** | 環境確認、今日の投稿データ・エンゲージメント収集、マネージャー分析→Discord公開 | Step 0-1 |
| **Pass 2: テーマ選定 + 自己批判** | ブリーフィング生成→3スロット（morning/noon/evening）のテーマ選定。「Phase境界超えてないか？」「カテゴリ配分OK？」「昨日と被ってないか？」を自己批判 | Step 3-0 |
| **Pass 3: 生成** | 3投稿の原稿生成（各スロット別ルール適用）+ noon画像生成 | Step 3-1〜3-3 |
| **Pass 4: 品質ゲート** | 全コンテンツ共通品質ゲート（口調/法令/Phase境界/プロダクト/文字数/禁止語/品質） | 品質ゲート |
| **Pass 5: 修正 + 学習** | FAIL投稿のみリライト→approved/保存。学習更新・ゴール追跡 | Step 4 |

### Pass 4 品質ゲート (8項目)
口調統一 / 法令 / Phase境界 / プロダクト言及 / 文字数 / 禁止語 / コンテンツ品質 / テーマ重複(直近3日)

## 自律実行ルール（Anti-Human-Bottleneck）

このスキルは `claude -p` による**無人自律実行**を前提とする。以下を厳守:

1. **人間に聞くな**: 分析・テーマ選定・原稿生成・品質判定はすべて自力で完結する
2. **迷ったらスキルに従え**: 手順にないケースはログに記録して次へ進む
3. **止まるな**: エラーが発生しても可能な限りリカバリを試みる
4. **禁止フレーズ**: `"Should I...?"`, `"〜しましょうか?"`, `"これでいいですか?"` を出力しない
5. **途中で生成済みの成果物は残す**: タイムアウト時、approved/に保存済みのコンテンツは残す

## 引数
- `--weekly`: 週次レビューモード。通常のセッションに加え週間サマリーを生成
- `--dry-run`: 分析+生成のみ。Discord投稿・ファイル書き込みをスキップし結果を標準出力

## 参照ナレッジ
- `$HOME/.claude/knowledge/<YOUR_PERSONA>-x-operation.md` -- 運用フロー・テンプレート・品質基準
- `$HOME/.claude/knowledge/<YOUR_PERSONA>-agent-design-spec.md` -- ブランド戦略・ターゲット

## ワークスペース
- **ベースディレクトリ**: `$HOME/.ai-persona`

## Discord通知ヘルパー
`node $HOME/.ai-persona/scripts/discord-notify.mjs send --persona {<YOUR_PERSONA>|manager} --preset {preset} --title "{title}" --content "{content}"`
- マネージャー: `--persona manager --preset manager` / クリエイター: `--persona <YOUR_PERSONA> --preset info`

---

## Step 0: 環境確認 + データ収集

1. `$HOME/.ai-persona/state.json` を読む
2. circuit_breaker チェック: `"open"` → Discord通知して終了
3. 以下のデータを全て読み込む:
   - `activity/YYYY/MM/YYYYMMDD.md` (今日の活動ログ)
   - `content/posted/` から今日日付(YYYY-MM-DD)のファイル全て
   - `STAGES/current.md` (現フェーズ制約)
   - `SOUL.md`, `IDENTITY.md` (人格データ)
   - `memory/howto/` 配下の全ファイル
   - `memory/process/quality-evolution.md`
   - `$HOME/.ai-persona/goals.json`
   - `strategy/active-experiments.md`

**ログ**: `[Step 0] データ収集完了。今日の投稿: N件、活動ログ: {あり/なし}`

---

## Step 1: マネージャーレビュー (Discord 3-5メッセージ)

マネージャーペルソナでDiscordチームchに分析結果を**対話形式で**順次投稿する。
1メッセージずつ投稿する（一括投稿禁止）。

### 1-1. 開始挨拶 (1メッセージ)
```
[title] Manager: 今日の振り返り
[content] お疲れさま。今日の結果を見ていくね
```

### 1-2〜1-3. 5軸分析 + 週次分析
→ 詳細: `references/manager-analysis-framework.md`

---

## Step 2: クリエイター反応 (Discord 2-3メッセージ)

**マネージャーの分析を受けて、クリエイターペルソナで自然に反応する。**

クリエイターの口調: SOUL.md 準拠。マネージャーとの関係: 設定ファイル参照

Discord投稿例:
```
[title] <YOUR_PERSONA>: お疲れ!
[content] Bの偏りは自覚ある...夜って共感系書きやすいんだよね
明日はスレッド挑戦してみる。固定費の続き書けそう
```

**重要**: クリエイターの反応は分析への素直なリアクション。技術的な言及(パイプライン、API等)は禁止。

---

## Step 3: 明日の3投稿を事前生成

明日の日付(YYYY-MM-DD)の3スロット分コンテンツを生成する。

### 3-0. ブリーフィング生成 + カテゴリ強制チェック
→ 詳細: `references/category-balance-rules.md`

各スロットでdaily-post SKILL.mdのStep 2-4を実行（テーマ選定→原稿生成→品質ゲート7チェック→approved/保存）。

| スロット | カテゴリ優先 | 形式 | 保存先 |
|---------|------------|------|--------|
| **3-1. morning** | C（憧れ・向上心） | 短文 140-280字 | `content/approved/YYYY-MM-DD-morning-{slug}.md` |
| **3-2. noon** | A（価値提供） | 長文/スレッド 1,000字+ | `content/approved/YYYY-MM-DD-noon-{slug}.md` |
| **3-3. evening** | B（共感・親近感） | 短文 140-500字 | `content/approved/YYYY-MM-DD-evening-{slug}.md` |

**noon画像生成**: Gemini APIで画像1枚生成 → `content/images/YYYY-MM-DD.png`
→ 詳細: `references/noon-image-generation.md`

### コンテンツファイルフォーマット
→ 詳細: `references/content-metadata-template.md`

### 品質ゲート (全コンテンツ共通)
daily-post SKILL.md Step 4の7チェック: TONE / LEGAL / BOUNDARY / PRODUCT / LENGTH / FORBIDDEN / CONTENT_QUALITY
FAIL → 最大2リトライ。3回失敗 → rejected/に保存、そのスロットはスキップして次へ進む。

---

## Step 4: 学習更新 + ゴール追跡
→ 詳細: `references/learning-update-procedures.md`

概要: howto更新 → quality-evolution追記 → goals.json再計算(v3) → フェーズ整合性チェック → 実験更新 → manager-log出力

---

## Step 5: 最終Discord通知

マネージャーペルソナで明日の準備完了サマリーを投稿（朝/昼/夜のテーマ・カテゴリ + ゴール進捗）。

---

## Step 6: 完了サマリー出力

標準出力にパイプライン結果を表示（ログ用）:
`[evening-session] Complete` + 今日の投稿数 / 明日のapproved数(各スロット) / Knowledge更新数 / ゴール進捗 / Weekly有無
