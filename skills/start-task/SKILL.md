---
name: start-task
description: Issue番号を指定してブランチを作成し、安全に作業を開始する。並列作業の衝突を防止。
argument-hint: [Issue番号]
---

# Start Task: $ARGUMENTS

**1 Issue = 1 ブランチ = 1 ウィンドウ の原則を自動適用。**

## 手順

### 1. 現在のブランチ状態を確認

```bash
git status
git stash list
```

- 未コミットの変更がある場合: stash するか確認
- 既に作業中のブランチがある場合: 警告を出す

### 2. Issue詳細を取得

```bash
gh issue view $ARGUMENTS
```

- Issue番号、タイトル、ラベルを確認
- 既にアサインされている場合は警告(他のウィンドウで作業中の可能性)

### 3. Issueを自分にアサイン

```bash
gh issue edit $ARGUMENTS --add-assignee @me
```

### 4. ブランチ命名規則に従いブランチを作成

ラベルからブランチプレフィックスを決定:
| ラベル | プレフィックス | 例 |
|--------|---------------|-----|
| `dev` | `feat/` | `feat/42-add-auth` |
| `bug` | `fix/` | `fix/43-login-error` |
| `marketing` | `marketing/` | `marketing/44-sns-campaign` |
| `finance` | `finance/` | `finance/45-cf-report` |
| `ops` | `ops/` | `ops/46-setup-ci` |
| `design` | `design/` | `design/47-lp-redesign` |
| (ラベルなし) | `task/` | `task/48-misc-cleanup` |

```bash
git checkout main
git pull origin main
git checkout -b {prefix}/{issue-number}-{slug}
```

slug = Issueタイトルから英語キーワードを抽出(最大3語、ハイフン区切り)

### 5. 作業開始の確認

以下を表示して完了:
- 現在のブランチ名
- Issue概要
- 「作業完了後は `/finish-task` で PR を作成してください」

## 安全ルール

- **同じIssue番号のブランチが既にリモートに存在する場合**: そのブランチをチェックアウト(新規作成しない)
- **mainブランチで直接作業しない**: 必ずブランチを切る
- **ブランチ名にIssue番号を含める**: 追跡可能性を確保
