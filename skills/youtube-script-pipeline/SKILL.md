<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
---
name: youtube-script-pipeline
description: "Automation-only: YouTube script auto-generation pipeline. Run via orchestrator with `claude -p`. Do NOT trigger for interactive sessions."
argument-hint: "issue #N [--dry-run]"
---

# YouTube Script Pipeline: $ARGUMENTS

BEFORE asking the user any question, requesting confirmation, or stopping to wait for human input, resolve it yourself using available tools and the instructions in this skill. Only ask a human for physically impossible tasks (SMS codes, CAPTCHA, hardware access).

## Step 0: ナレッジ読込 + Issue情報取得

以下を全て並列で読み込む:

**必須ナレッジ（5ファイル）**:
- `$HOME/.claude/knowledge/youtube-scriptwriting.md`
- `$HOME/.claude/knowledge/scriptwriting-patterns.md`
- `$HOME/.claude/knowledge/youtube-script-production.md`
- `<YOUR_WORKSPACE>/marketing/youtube-strategy.md`
- `$HOME/.claude/skills/youtube-script/SKILL.md` (セルフレビューチェックリスト参照)

**Issue情報**:
- `$ARGUMENTS` からIssue番号を抽出
- `gh issue view <N>` でタイトル・本文を取得（存在しない場合はqueue.jsonから情報取得）
- `content/youtube/queue.json` から該当エントリの詳細を読む

**dry-runフラグ**:
- `$ARGUMENTS` に `--dry-run` が含まれる場合、Step 6のDiscord通知をスキップ

## Step 1: リサーチ

### 1a. 既存データ分析
- YouTube戦略ドキュメントの分析データから関連パターンを抽出
- テーマに関連する既存ナレッジ、ブログ記事、過去台本を検索
- `content/youtube/drafts/` に過去の台本があれば参照（トーン統一）

### 1b. Web検索（3-5回）
- 競合YouTube動画のタイトル・構成パターン
- テーマに関する最新データ・統計
- 視聴者が検索しそうなキーワード

### 1c. リサーチ保存
全ての調査結果を `content/youtube/drafts/{issue}/research.md` に保存:
```
# リサーチ: {テーマ}
## 競合分析
## キーワード
## データ・統計
## 既存ナレッジからの知見
```

## Step 2: 構成設計

queue.jsonの `format` フィールドに基づき構成を設計:

### 型選択
- `list`: リスト形式 -- ティップス、ランキング、共通点系
- `discovery`: 発見追体験形式（ミステリー型） -- データ分析、調査系
- `story`: ストーリーテリング形式 -- ケーススタディ、ブランドストーリー

### 設計項目
1. **感情カーブ**: 時間軸に沿った感情設計図（3分以上のフラットゾーン禁止）
2. **オープンループ配置**: 2-3分ごとのフック。セクション末尾の引きポイント
3. **パターンインタラプト配置**: 25-35秒ごとの変化ポイント
4. **チャプター構成**: 概要欄用タイムスタンプ
5. **図表リスト**: 必要なビジュアル素材

構成を `content/youtube/drafts/{issue}/structure.md` に保存。

## Step 3: 台本生成（3パス）

### Pass 1: 初稿
youtube-script-production.mdのHTMLデザインシステムに完全準拠した台本HTMLを生成。

必須セクション（全て含めること）:
1. ヘッダー（タイトル、想定尺、構成、データソース、最終更新日）
2. タイトル候補3案（推奨1+代替2）+ 設計意図
3. サムネイル方針（CSSモックアップ + 仕様表）
4. 制作メモ（図表リスト、テロップルール、編集リズム、BGM指定）
5. 免責事項（動画冒頭表示用 + 概要欄用）
6. 表現ガイドライン（禁止表現・推奨表現）
7. 感情カーブ（CSS棒グラフで可視化）
8. 台本本文（シーンブロック: 時間タグ+タイトル+演出指示+セリフ+テロップ指示）
9. 概要欄（コピペ用: チャプター、CTA、データソース、免責、ハッシュタグ）
10. ピン留めコメント（コピペ用）
11. チェックリスト（撮影前/撮影中/編集/公開の4段階）

HTMLデザインシステム:
```
色:
  --accent: #c0392b（赤、警告・強調）
  --blue: #2c5282（演出指示）
  --green: #276749（ポジティブ情報）
  --text: #1a1a1a / --text-light: #555

コンポーネント:
  .scene         -- シーンブロック（黒ヘッダー+赤タイムタグ）
  .line          -- 台詞テキスト（17px, line-height:2）
  .line strong   -- 強調ワード（赤）
  .line em       -- ハイライト（黄色マーカー）
  .direction     -- 演出指示（青背景+青左ボーダー）
  .beat          -- 間（ま）の指示（中央揃え、灰色）
  .box           -- 情報ボックス（red/blue/green/gray/orange）
  .copyable      -- コピペ用ブロック（灰色背景）
  .checklist     -- チェックリスト（CSSチェックボックス）
  .title-candidate -- タイトル候補（推奨=赤枠、代替=灰枠）
  .fig-spec      -- 図表仕様（番号バッジ+説明）
  .curve         -- 感情カーブ（flexbox棒グラフ）
  .thumbnail-mock -- サムネモックアップ（16:9 ダーク背景）
```

### Pass 2: セルフレビュー（3グループ並列）

初稿を3つの観点グループに分け、**並列サブエージェント**でチェックする。
各グループは独立して判定し、FAIL項目ごとに具体的な修正指示を返す。

#### 実行方法
3つのサブエージェント（general-purpose）を同時に起動する。各エージェントには台本HTML全文と担当グループのチェック項目を渡す。

#### Review Group A: 構成品質

台本の骨格が視聴維持率を最大化する構造になっているかを検証する。

| # | チェック項目 | 基準 |
|---|-------------|------|
| A1 | 最初の8秒 | 好奇心ギャップがあるか |
| A2 | OP/自己紹介 | 3秒以下 or 削除されているか |
| A3 | データの読み上げ | 結論ファーストで1文で伝えているか |
| A4 | オープンループ | 2-3分ごとに配置されているか |
| A5 | パターンインタラプト | 25-35秒ごとに変化ポイントがあるか |

#### Review Group B: 感情設計

| # | チェック項目 | 基準 |
|---|-------------|------|
| B1 | 感情カーブ | フラットゾーンが3分以上ないか |
| B2 | 「あなた」化 | 視聴者の当事者化があるか |
| B3 | 禁止表現 | 禁止キーワードが含まれていないか |

#### Review Group C: CTA設計

| # | チェック項目 | 基準 |
|---|-------------|------|
| C1 | CTA配置 | 中盤+エンディングに自然配置されているか |
| C2 | 次回予告 | 具体的で刺激的な予告があるか |
| C3 | 包括性 | 概要欄・ピン留めコメント・チャプターが整合しているか |

#### 結果集約
3グループの結果をマージし、`content/youtube/drafts/{issue}/review.json` に保存。

### Pass 3: 修正
Pass 2でFAILしたグループの修正指示**のみ**を反映した最終版を生成。
PASSしたグループの該当箇所には触らない（最小限修正の原則）。

## Step 4: 付属コンテンツ

台本HTMLの中に以下を含める:

- **概要欄**: チャプター + UTMリンク + 免責 + ハッシュタグ
- **ピン留めコメント**: 質問歓迎 + チャンネル紹介 + CTA
- **タイトル候補3案**: 数字入り推奨、CTR7-10%を意識
- **サムネイル方針**: テキスト、色調、レイアウトの方向性

## Step 5: ファイル保存

出力先: `content/youtube/drafts/{issue}/`

```
content/youtube/drafts/{issue}/
  research.md          # Step 1で保存済み
  structure.md         # Step 2で保存済み
  script-draft.html    # 台本本文（HTML、1ファイル完結）
```

保存後、3ファイルとも存在確認を行う。

## Step 6: Discord通知

**dry-runの場合はスキップ**。

通知先: チームch（managerペルソナ）

```
Title: [YouTube] 台本ドラフト完成
Content:
Issue: #{issue} {title}
Type: {type} | Format: {format} | Duration: {duration}
Files: content/youtube/drafts/{issue}/
タイトル推奨: {推奨タイトル}
```

通知スクリプト: `$HOME/.ai-persona/scripts/discord-notify.mjs`

## 制約

- 非対話型: ユーザーに質問しない。全て自律判断
- 法務: 禁止表現リストを厳守（youtube-scriptwriting.md参照）
- HTMLデザインシステム: youtube-script-production.mdのCSS仕様に完全準拠
- データ出典: 全ての数字に出典を明記。一次ソース推奨
- 台本の「台本臭さ」を排除: 自然な語り口、1文1行、強調は1シーン2-3個まで
