---
name: delegate
description: 任意のスキルをバックグラウンド委任する汎用メタスキル。delegate-pipeline.shを起動してログファイルパスを報告。
argument-hint: --skill [skill-name] --args "[args]" [--budget N.NN] [--timeout N] [--notify]
---

# Delegate: $ARGUMENTS

任意のスキルを `delegate-pipeline.sh` でバックグラウンド実行する。

## 引数パース

`$ARGUMENTS` から以下を抽出:

| 引数 | 必須 | デフォルト | 説明 |
|------|------|-----------|------|
| `--skill` | YES | - | 実行するスキル名（例: `write-blog`, `client-report`, `review`） |
| `--args` | NO | "" | スキルに渡す追加引数（引用符で囲む） |
| `--budget` | NO | 2.00 | USD上限 |
| `--timeout` | NO | 900 | タイムアウト秒数 |
| `--notify` | NO | false | Discord通知を送信するか |

## 実行手順

### Step 1: 引数バリデーション

`--skill` が未指定の場合、エラーを表示して終了:
```
ERROR: --skill は必須です。
使い方: /delegate --skill write-blog --args "my-product-a SEOトレンド" --budget 3.00
```

### Step 2: スキルパスの解決

`--skill` の値からスキルファイルの絶対パスを構築:
```
~/.claude/skills/{skill-name}/SKILL.md
```

スキルファイルが存在するか確認（Bash `test -f`）。存在しない場合:
```
ERROR: スキル '{skill-name}' が見つかりません。
パス: ~/.claude/skills/{skill-name}/SKILL.md
```

### Step 3: ラベルの生成

`--args` の先頭単語とスキル名を組み合わせてラベルを生成:
```
{skill-name}-{first-arg-word}
```
例: `--skill write-blog --args "my-product-a SEOトレンド"` → ラベル `write-blog-my-product-a`

`--args` が空の場合はスキル名のみ: `write-blog`

### Step 4: delegate-pipeline.sh をバックグラウンド起動

Bash toolで以下を **バックグラウンド実行** する:

```bash
<YOUR_PROJECT_ROOT>/scripts/delegate-pipeline.sh \
  --skill "~/.claude/skills/{skill-name}/SKILL.md" \
  --label "{label}" \
  --budget {budget} \
  --timeout {timeout} \
  --prompt "{args}" \
  --workspace "<YOUR_WORKSPACE>/<YOUR_OPS_REPO>" \
  {--notify if specified}
```

**重要**: Bash toolの `run_in_background: true` を使用する。

### Step 5: ユーザーへの報告

起動成功後、以下を報告:

```
[delegate] バックグラウンドで起動しました

- スキル: {skill-name}
- ラベル: {label}
- 予算: ${budget}
- タイムアウト: {timeout}秒
- 通知: {on/off}
- ログ: <YOUR_AUTOMATION_DIR>/logs/delegate-{label}-{YYYY-MM-DD}.log

ログを確認するには:
  tail -f <YOUR_AUTOMATION_DIR>/logs/delegate-{label}-{YYYY-MM-DD}.log
```

## 使用例

```
/delegate --skill review --args "scripts/notify.sh" --budget 0.50 --timeout 120 --notify
/delegate --skill write-blog --args "my-product-a SEOトレンド" --budget 3.00 --timeout 1800
/delegate --skill client-report --args "client-a --monthly" --notify
```
