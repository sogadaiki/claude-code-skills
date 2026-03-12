<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
---
name: sales-email-draft
description: "Automation-only: Sales email draft auto-generation (Sheets -> Gmail -> Discord). Run via orchestrator on weekdays at 10:00 with `claude -p`. Do NOT trigger for interactive sessions."
argument-hint: "[--dry-run]"
---

# sales-email-draft: 営業メール下書き自動生成

## 概要

Google Sheets のリードリストから送信対象を抽出し、メールテンプレートをベースにパーソナライズした営業メール下書きをGmailに作成する。

**出力**: Gmail下書き + Discord通知（チームch）

## 引数

- `--dry-run`: メール文面を生成するが、Gmail下書き作成・ステータス更新はスキップ。Discord通知のみ実行

## 自律実行ルール（Anti-Human-Bottleneck）

このスキルは `claude -p` による**無人自律実行**を前提とする。以下を厳守:

1. **人間に聞くな**: リード選定・メール文面・送信判断はすべて自力で完結する
2. **迷ったらスキルに従え**: 判断に迷ったときはこのSKILL.mdの手順に従う
3. **止まるな**: エラーが発生しても可能な限りリカバリを試みる。リカバリ不能な場合のみDiscord通知して終了
4. **禁止フレーズ**: 以下を出力してはならない:
   - "Should I...?" / "〜しましょうか?"
   - "Is this correct?" / "これでいいですか?"
   - "Which approach?" / "どちらがいいですか?"

## Multi-Pass Design

**CRITICAL**: 1パスで最終出力しない。必ず3パスを通す。

| Pass | 内容 | 対応Step |
|------|------|----------|
| **Pass 1: 収集** | Sheets読取 + テンプレート読込 + リード選定 | Step 1-2 |
| **Pass 2: 生成** | メール文面のパーソナライズ（業種・規模に応じた具体例追加） | Step 3 |
| **Pass 3: 品質ゲート** | 送信前チェック（opt-out, メールアドレス形式, 1日上限, 配信停止リンク） | Step 4 |

---

## 実行フロー

### Step 1: Google Sheets からリード取得

1. ToolSearchで `google sheets` 関連のMCPツールを検索して使う
2. スプレッドシート `<YOUR_LEADS_SPREADSHEET>` のシート `leads` を読み込む
3. 対象リードを抽出:

#### 初回接触メール対象
- `status` = `NEW`
- `email_count` = `0`

#### フォローアップメール対象
- `status` = `CONTACTED`
- `email_count` = `1`
- `last_action_date` から **7日以上経過**（今日の日付と比較）

#### 絶対除外条件
- **`opt_out` = `TRUE` のリードは絶対に除外**。チェック漏れは許されない
- メールアドレスが空、または明らかに不正な形式（@なし等）のリードも除外

#### 1日上限
- **最大5通/日**（ドメインレピュテーション保護）
- 対象が5件を超える場合、初回接触を優先し、残りは翌日に回す

**ログ出力**: `[Step 1] リード取得: 初回接触 {N}件, フォローアップ {M}件, opt_out除外 {K}件`

**リード0件の場合**: 静かに終了（Discord通知不要）

---

### Step 2: メールテンプレート読込

1. `<YOUR_WORKSPACE>/marketing/sales-automation/email-templates.json` を読み込む
2. リードの種別に応じてテンプレートを選択:
   - 初回接触（`email_count` = 0）→ 初回接触用テンプレート
   - フォローアップ（`email_count` = 1）→ フォローアップ用テンプレート

---

### Step 3: メール生成（パーソナライズ）

各リードについて以下を実行:

#### 3A: テンプレート変数置換

テンプレート内の変数をリード情報で置換:
- `[会社名]` → リードの会社名
- `[業種]` → リードの業種
- `[担当者名]` → リードの担当者名（あれば）
- その他テンプレートで定義された変数

#### 3B: AIパーソナライズ

置換後のメール文面をさらにパーソナライズ:

1. **業種に合わせた具体例の追加**: リードの業種に関連するAI活用事例を1つ追加
2. **トーンの微調整**: 業種・規模に応じた言い回しの調整
3. **サービス情報の明記**:
   - サービスLP: `<YOUR_SERVICE_URL>`
   - 形式・定員・価格等をテンプレートで定義
   - **重要**: オンライン対応も可能であることを必ず明記（遠方リードへの訴求）
4. **配信停止リンク**: メール末尾に配信停止の案内を必ず含める

#### 3C: 生成ルール

- 件名は30文字以内。開封率を意識した具体的な件名
- 本文は400-600字程度。簡潔に要点を伝える
- from: `<YOUR_EMAIL>`
- 署名ブロックを含める

---

### Step 4: 品質ゲート

**CRITICAL**: 全リードのメールについて以下を全チェック。1つでもFAILなら該当メールを修正。

| # | チェック名 | 基準 | FAIL条件 |
|---|-----------|------|----------|
| 1 | OPT_OUT_RECHECK | opt_out再確認 | opt_out=TRUEのリードが含まれている |
| 2 | EMAIL_FORMAT | メールアドレス形式 | @がない、ドメイン部分が不正 |
| 3 | DAILY_LIMIT | 1日上限チェック | 5通を超えている |
| 4 | UNSUBSCRIBE_LINK | 配信停止案内 | メール本文に配信停止の案内がない |
| 5 | CONTENT_CHECK | 内容チェック | テンプレート変数が未置換（[会社名]等がそのまま残っている） |
| 6 | SERVICE_INFO | サービス情報 | サービスLPのURLが含まれていない |

**判定**:
- 全項目PASS → Step 5へ
- FAIL項目あり → 修正して再チェック（最大2回）
- 2回修正してもFAIL → 該当リードをスキップしてDiscord通知

---

### Step 5: Gmail下書き作成

**`--dry-run` の場合はこのStepをスキップ**

1. ToolSearchで `gmail` 関連のMCPツールを検索して使う
2. 各リードについて `create_gmail_draft` で下書きを作成:
   - **to**: リードのemailアドレス
   - **subject**: 生成済みの件名
   - **body**: 生成済みの本文
   - from: `<YOUR_EMAIL>`

**下書き作成失敗時**: 該当リードをスキップし、失敗リストに追加。パイプラインは止めない

**ログ出力**: `[Step 5] Gmail下書き作成: 成功 {N}件, 失敗 {M}件`

---

### Step 6: ステータス更新

**`--dry-run` の場合はこのStepをスキップ**

Google Sheetsのリードリストを更新:

| フィールド | 更新内容 |
|-----------|---------|
| `status` | `NEW` → `CONTACTED` |
| `email_count` | 現在値 + 1 |
| `last_action_date` | 今日の日付（YYYY-MM-DD） |

**更新失敗時**: Discord通知して終了（メール下書きは作成済みなので、手動で更新してもらう）

---

### Step 7: Discord通知

通知先: `<YOUR_DISCORD_WEBHOOK>` (チームch)

**通知内容（1メッセージにまとめる）**:

```
営業メール下書き {N}件作成しました。Gmailで確認してください

1. {会社名} ({業種}) -- {件名}
2. {会社名} ({業種}) -- {件名}
...

失敗: {M}件（{失敗理由}）
```

**通知条件**:
- 下書き1件以上作成 → 通知する
- リード0件 → 通知しない（静かに終了）
- 全件失敗 → エラー通知する
- `--dry-run` → 「[DRY-RUN] メール文面 {N}件生成しました（下書き未作成）」

---

### Step 8: 完了サマリー出力

パイプライン実行結果を標準出力に出す（orchestratorログ用）:

```
[sales-email-draft] 完了
対象リード: 初回接触 {N}件, フォローアップ {M}件
下書き作成: {K}件
ステータス更新: {L}件
失敗: {F}件
```

---

## エラーハンドリング

| エラー | 対応 |
|--------|------|
| Google Sheets接続失敗 | Discord通知して終了 |
| リード0件 | 静かに終了（通知不要） |
| Gmail下書き作成失敗（個別） | 該当リードをスキップ、Discord通知に含める |
| Gmail API全体障害 | Discord通知して終了 |
| ステータス更新失敗 | Discord通知して終了（下書きは作成済み） |
| **opt_outリードへの送信** | **絶対に行わない。Step 1 + Step 4で二重チェック** |

---

## セキュリティ

- **opt_out=TRUEのリードには絶対にメールを送らない**（Step 1除外 + Step 4再確認の二重チェック）
- 1日最大5通の上限を厳守（ドメインレピュテーション保護）
- メール本文に配信停止案内を必ず含める
- .envファイルは読まない（global security rule）
- シークレットはmacOS Keychainから取得

## 参照ナレッジ

- メールテンプレート: `<YOUR_WORKSPACE>/marketing/sales-automation/email-templates.json`
- Discord通知: `$HOME/.ai-persona/scripts/discord-notify.mjs`
- 多段階原則: `$HOME/.claude/knowledge/multi-pass-principle.md`
