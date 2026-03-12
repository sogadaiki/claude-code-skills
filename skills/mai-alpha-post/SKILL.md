<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
---
name: alpha-post
description: "Automation-only: AI persona autonomous micro-post generation. Run via orchestrator with `claude -p`. Do NOT trigger for interactive sessions."
argument-hint: "--type {survey-announce|survey-progress|micro-insight|manager-report|weekly-report}"
---

# Alpha Post -- 自走型α投稿パイプライン

外部トリガー（依頼・反応）なしで自律的に生成する超短文コンテンツ。
Phase 0の「誰にも届かない」状況を「頑張ってる感」に変換する。

## Multi-Pass Design

| Pass | 内容 |
|------|------|
| **Pass 1: 素材収集** | 人格ロード、直近投稿確認、今日のメイン投稿テーマ把握 |
| **Pass 2: テーマ選定 + 自己批判** | alpha-typeに基づくテーマ生成。「これ投稿して意味あるか?」を自問 |
| **Pass 3: 生成** | 超短文原稿生成（60-140字） |
| **Pass 4: 品質ゲート** | 簡易3チェック（口調/禁止語/Phase境界） |
| **Pass 5: 修正** | FAILならリライト。2回FAILで別テーマ |

## 自律実行ルール（Anti-Human-Bottleneck）

このスキルは `claude -p` による**無人自律実行**を前提とする。以下を厳守:

1. **人間に聞くな**: テーマ選定・原稿修正・品質判定はすべて自力で完結する
2. **迷ったらスキルに従え**: 手順にないケースはログに記録して次へ進む
3. **止まるな**: エラーが発生しても可能な限りリカバリを試みる
4. **禁止フレーズ**: `"Should I...?"`, `"〜しましょうか?"`, `"これでいいですか?"` を出力しない

## 引数

- `--type [alpha-type]`: 必須。以下のいずれか:
  - `survey-announce`: 今日の調査宣言（朝9:00、80-120字）
  - `survey-progress`: 調査中のつぶやき（午後14:00、100-140字）
  - `micro-insight`: マイクロ気づき（夜21:00、60-100字）
  - `manager-report`: マネージャーブリーフィング報告（夜21:00、80-120字）
  - `weekly-report`: 週次振り返り（土曜朝、180-200字）

## 参照ナレッジ
- `$HOME/.claude/knowledge/<YOUR_PERSONA>-x-operation.md` -- 運用フロー
- `$HOME/.claude/knowledge/<YOUR_PERSONA>-agent-design-spec.md` -- ブランド戦略

## ワークスペース
- **ベースディレクトリ**: `$HOME/.ai-persona`

---

## Step 0: 環境確認

1. `$HOME/.ai-persona/state.json` を読む
2. circuit_breaker チェック: `"open"` → Discord通知して終了
3. `--type` 引数を確認（未指定→エラー終了）

## Step 1: 素材収集

以下を読み込む:
- `SOUL.md` -- 人格・口調ルール
- `STAGES/current.md` -- Phase制約
- `content/approved/` -- 今日のメイン投稿のテーマを把握（重複回避）
- `content/posted/` -- 直近3日の投稿内容（テーマ重複回避）

## Step 2: α投稿生成

`--type` に基づいて投稿を1本生成する。

### survey-announce（今日の調査宣言）

「今日はこれを調べてみる」の宣言。外部依頼なしで自走する疑似受注。

テンプレート:
1. 「今日は{テーマ}について調べてみる。{動機}。昼ごろまでにまとめる」
2. 「今日の調査テーマは{テーマ}にした。{予想}。時間かかるかも」
3. 「{テーマ}について自分でも整理できてないから、今日ちゃんと調べることにした」

テーマ選定:
- 今日のnoon投稿（approved/にあれば）のテーマと関連づける
- なければ、STAGES/current.mdの投稿カテゴリから選ぶ

字数: 80-120字
カテゴリ: A（価値提供の予告）

### survey-progress（調査中のつぶやき）

進捗報告。完成前の「過程」を見せる最重要投稿。

テンプレート:
1. 「{テーマ}調べてたら、思ってたよりずっと{発見}。まだ途中だけど面白い」
2. 「今日の調査、難航してる。{困難}。でもちゃんとやらないと意味ないから続ける」
3. 「{テーマ}調べてたら全然関係ない面白い話が出てきた。後でまとめる価値ありそう」

字数: 100-140字
カテゴリ: A or B（過程の共有）

### micro-insight（マイクロ気づき）

「ふと思ったんだけど」系。短いほど良い。

テンプレート:
1. 「{普遍的な観察}。{自分の視点を一言}」
2. 「{常識だと思ってたこと}、調べてみると{意外な事実}」
3. 「{二項対立}。どっちが正解かわからないけど、考えること自体が面白い」

字数: 60-100字
カテゴリ: A or C（気づき・価値観）

### manager-report（マネージャーブリーフィング報告）

マネージャーキャラを通じた数字の公開。ペルソナが孤独じゃないことも伝わる。

テンプレート:
1. 「<MANAGER_NAME>から昨日の数字が来た。{数字の要約}。{感想}」
2. 「<MANAGER_NAME>のブリーフィングに『{指摘}』って書いてあった。{反応}」
3. 「<MANAGER_NAME>から今週の集計。投稿{N}本、{主要KPI}。{淡々とした感想}」

データソース:
- `goals.json` の actual 値
- `content/posted/` の直近7日のファイル数
- `manager-log/` の直近ログ

字数: 80-120字
カテゴリ: A or B（数字公開 + 人格演出）

### weekly-report（週次振り返り）

土曜定番の「今週の報告」。

テンプレート:
```
今週の報告

投稿: {N}本
フォロワー: {N}人（{増減}）
調査テーマ: {テーマ1}、{テーマ2}、{テーマ3}

{1-2行の感想。淡々と。}
来週も続ける
```

字数: 150-200字
カテゴリ: A（数字公開）

---

## Step 3: 簡易品質ゲート（3チェック）

α投稿は超短文のため、簡易版の3チェックのみ:

| # | チェック | FAIL基準 |
|---|---------|---------|
| 1 | 口調 | ペルソナの口調から逸脱（敬語混在等） |
| 2 | 禁止語/Phase境界 | STAGES/current.mdの禁止事項、プロダクト名言及 |
| 3 | 文字数 | type別の上限を超過 |

FAIL → 1回リライト。2回FAILなら別テーマで再生成。3回目FAILでスキップ（Discord通知のみ）。

## Step 4: 保存 + X投稿

1. `content/approved/YYYY-MM-DD-alpha-{type}-{slug}.md` に保存

コンテンツファイルフォーマット:
```markdown
# Xショートポスト: {タイトル}

## メタ情報
- 形式: 短文ツイート
- カテゴリ: {A/B/C/D} ({カテゴリ名})
- テンプレート: alpha-{type}
- slot: alpha
- 文字数: 約{N}字
- 生成日: YYYY-MM-DD
- 品質ゲート: PASS

## 本文

{投稿テキスト}
```

2. X投稿を実行:
```bash
node $HOME/.ai-persona/scripts/x-post.mjs post "${text}"
```

3. 投稿成功 → `content/posted/` に移動（メタデータ追記）
4. 投稿失敗 → `content/approved/` に残す（次のpublishで消化）

## Step 5: Discord通知 + 記録更新

1. Discord通知:
```bash
node $HOME/.ai-persona/scripts/discord-notify.mjs send \
  --persona <YOUR_PERSONA> --preset info \
  --title "[alpha/{type}] Posted" \
  --content "{投稿テキスト冒頭100字}"
```

2. `state.json` 更新:
   - `posting.today_post_count += 1`
   - `posting.last_post_at = now`
   - `stats.total_posts += 1`

3. `activity/YYYY/MM/YYYYMMDD.md` にログ追記

## Step 6: 完了サマリー

```
[alpha-post] Complete
Type: {alpha-type}
Category: {category}
Length: {N}chars
Posted: {yes/no} {tweet_url}
```
