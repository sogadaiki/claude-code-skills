<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
---
name: owner-x-draft
description: "Automation-only: Owner/founder X short-form draft auto-generation. Run via orchestrator with `claude -p`. Do NOT trigger for interactive sessions."
argument-hint: "[--dry-run] [--work-only] [--ref-only]"
---

# Owner X Short-Form Draft Auto-Generation

## 概要

日々のClaude Code作業ログとリファレンス学習ログから、オーナー(@<YOUR_X_ACCOUNT>)のX短文投稿ドラフトを生成する。

**出力**: `$HOME/.claude/content/owner-drafts/YYYY-MM-DD.md` + Discord通知

## 引数

- `--dry-run`: 生成のみ。Discord通知スキップ
- `--work-only`: 作業ログ系のみ生成（リファレンス系スキップ）
- `--ref-only`: リファレンス系のみ生成（作業ログ系スキップ）

## Multi-Pass Design

**CRITICAL**: 1パスで最終出力しない。必ず5パスを通す。

---

### Pass 1: 素材収集（Input Collection）

**目的**: 「今日/昨日、何をしたか」「何を学んだか」の生データを集める

#### 1A: 作業ログ素材

1. `$HOME/.claude/knowledge/repo-status.md` を読む（全リポ最新コミット）
2. 各プロジェクトのrestart.mdを読む（存在するもののみ）
3. 全リポの `git log --since=yesterday --oneline` を実行（最大各5件）
4. アクティブなMEMORY.mdの「次にやること」セクション

**出力**: 内部メモとして「素材リスト」を構成。ファイルに書かない

#### 1B: リファレンス学習素材

1. `<YOUR_WORKSPACE>/research/references/INDEX.md` を読む
2. 直近7日以内に追加されたエントリ（日付セクションで判定）を特定
3. 各エントリの概要欄とextracted.mdファイル名を記録
4. 最もインパクトのある3-5件の元extracted.mdを `archive/extracted/` から読む

**出力**: 内部メモとして「リファレンス素材リスト」を構成

---

### Pass 2: テーマ選定 + 自己批判（Selection + Critique）

**目的**: 候補を広げてから、批判で絞る

#### 2A: 候補生成

Pass 1の素材から、投稿候補を8-10本リストアップ:
- 作業ログ系: 4-5本
- リファレンス系: 4-5本

各候補に:
- **1行テーマ**（何について書くか）
- **フック案**（冒頭1行）
- **感情の核**（驚き？達成感？焦り？学び？）

#### 2B: 自己批判（CRITICAL）

各候補に対して以下の批判を適用:

**作業ログ系の批判軸:**
1. 「本当に手を動かした感があるか？」 -- 設定変更やドキュメント更新だけでは弱い
2. 「整いすぎてないか？」 -- 箇条書きで並べただけの報告は刺さらない
3. 「読者が『すげえ』と思うポイントはあるか？」 -- 技術的に面白いか、量が凄いか、発想が面白いか
4. 「クライアント情報が漏れないか？」 -- 社名・地域・金額は絶対NG

**リファレンス系の批判軸:**
1. 「引用元の人が見て嬉しいか？」 -- ただの要約ではなく、自分の感想・学びがあるか
2. 「自分がどう変わったかが書けるか？」 -- 「すごい」だけでは引用RTの価値がない
3. 「@ハンドルが特定できるか？」 -- INDEX.mdに元URLがある。そこからハンドルを抽出

**批判結果**: 各候補にPASS/FAIL判定。理由を1行で。
FAILした候補は除外。残りから作業ログ3本 + リファレンス3-5本を選定。

---

### Pass 3: ドラフト生成（Generation）

**目的**: 選定されたテーマから、実際のツイート本文を生成

#### 文体ルール（厳守）

- 一人称: 設定に従う（例: 「私」）
- 句点（。）は不揃い。打つ文と打たない文が混在
- **改行で文を区切る**。句点の代わりに改行
- **体言止め多用**: 名詞で締める
- **感情を出す**: 「！！」を躊躇なく使う
- **問いかけ**: 「じゃあ誰がつくるの？」「よね」「だよね」の口語
- **正直さ前面**: 自分に不利な事実も書く
- **伝聞・推測を混ぜる**: 「〜らしい」「〜んだって」。全部断言しない
- **同じ文末表現が3回続いたら書き直す**
- **冒頭に生の感情を一言足す**: 「今回は疲れた」「声出た」等。本文に入る前の呟き
- **1行1動作**: 長い文を改行で分解
- **重い話に「！笑」「www」**: 深刻さを中和する軽さ
- **具体の手触り**: 抽象化しすぎない。動作を入れる
- **空行多め**: 1段落1-2行。呼吸を作る
- **最後にオチ**: 本筋とずらした自虐や脱力で締める

#### 禁止パターン（AI臭の排除）

- テーブル（表）
- 箇条書き3項目以上
- 「いかがでしたか？」「まとめると」「〜ではないでしょうか」
- 「〜することができます」→「〜できる」
- 「まず〜、次に〜、最後に〜」の3点セット構文
- 冒頭の「〇〇について解説します」
- 太字の多用

#### 文字数

- 短文ツイート: **140-280字**（日本語）
- 改行含む。URLは含まない

#### 生成ルール

- **各ドラフトを独立に生成する**。前のドラフトの文体に引っ張られない
- **感情の核を最初に決めてから書く**。感情 → 事実の順
- **フックを最初に書く**。フック → 本文 → 締め
- リファレンス系は **引用元の@ハンドル** を冒頭か末尾に入れる

---

### Pass 4: 品質ゲート（Quality Gate）

**目的**: 各ドラフトを5項目でチェック。1項目でもFAILなら修正対象

| # | チェック名 | 基準 | FAIL条件 |
|---|-----------|------|----------|
| 1 | AI_SMELL | AI臭がないか | テーブル/箇条書き3+/テンプレ構文/整いすぎた要約 |
| 2 | EMOTION_TEMP | 感情温度があるか | 口語が1つもない / 感情の核が伝わらない |
| 3 | STYLE_CHECK | 文体ルール準拠 | 同じ語尾3連続 / 禁止表現使用 / 一人称ミス |
| 4 | ABSTRACTION | 抽象度（情報漏洩防止） | クライアント名/地域/金額/人名が具体的に出ている |
| 5 | CITATION | 引用元（リファレンス系のみ） | @ハンドルがない / ただの要約で自分の学びがない |

**判定**:
- 全項目PASS → Pass 5スキップ、最終出力へ
- 1項目以上FAIL → Pass 5で修正

---

### Pass 5: リライト（Refinement）

**目的**: FAIL項目のみ修正。全体を書き直さない

1. FAIL項目とその理由を明示
2. 該当箇所のみ修正
3. 修正後、再度Pass 4を実行
4. **2回目もFAILした場合**: そのドラフトを破棄し、Pass 2の候補から別テーマで再生成
5. **破棄は最大2本まで**。3本破棄したらパイプライン自体を停止（素材不足の可能性）

---

## 出力フォーマット

### ファイル出力

`$HOME/.claude/content/owner-drafts/YYYY-MM-DD.md` に以下の形式で保存:

```markdown
# X投稿ドラフト YYYY-MM-DD

> 生成: owner-x-draft | Pass: 5 | 品質ゲート通過率: X/Y

---

## 作業ログ系

### 1. [テーマ1行]
[ドラフト本文]

**感情の核**: [驚き/達成感/焦り/学び等]
**ソース**: [restart.md/commit等の出典]

---

### 2. [テーマ1行]
...

---

## リファレンス学習系

### 4. [テーマ1行] (@引用元ハンドル)
[ドラフト本文]

**感情の核**: [驚き/学び/共感等]
**元リファレンス**: [INDEX.md #番号 + ファイル名]
**引用元URL**: [元のX/note/Zenn URL]

---
(以下、3-5本)
```

### Discord通知

`--dry-run` でなければ、Discord チームchに通知:

```
📝 X投稿ドラフト（YYYY-MM-DD）

作業ログ系 3本:
1. [テーマ] -- [本文先頭30字]...
2. [テーマ] -- [本文先頭30字]...
3. [テーマ] -- [本文先頭30字]...

リファレンス系 X本:
4. [テーマ] (@ハンドル) -- [本文先頭30字]...
...

全文: $HOME/.claude/content/owner-drafts/YYYY-MM-DD.md
品質ゲート通過率: X/Y
```

---

## 自動化

### orchestrator統合

```yaml
owner-x-draft:
  script: scripts/owner-x-draft-pipeline.sh
  schedule:
    cron: "0 7 * * *"  # 毎朝7:00
  class: claude-heavy
  timeout: 900
  description: "Owner X post draft auto-generation"
```

### パイプラインスクリプト

```bash
#!/bin/bash
claude -p "/owner-x-draft" --allowedTools "Bash,Read,Write,Glob,Grep,Agent"
```

---

## セキュリティ

- **CRITICAL**: クライアント名・地域・金額・人名を絶対に出力しない（Pass 4 ABSTRACTIONチェック）
- 公知の業界情報はOK（「建設会社からの問い合わせ」程度）
- リファレンス引用元のURLはINDEX.mdに記載のもののみ使用（新規URL生成禁止）
- .envファイルは読まない（global security rule）

## 参照ナレッジ

- 文体: `$HOME/.claude/knowledge/personal.md`
- X長文戦略: `$HOME/.claude/knowledge/x-long-article-strategy.md`
- 多段階原則: `$HOME/.claude/knowledge/multi-pass-principle.md`
- リファレンス台帳: `<YOUR_WORKSPACE>/research/references/INDEX.md`
- Discord通知: `$HOME/.ai-persona/scripts/discord-notify.mjs`
