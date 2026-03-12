---
name: map-project
description: プロジェクトの全体像を調査・文書化。ファイル構造、API、データモデル、外部連携、認証、データフロー、歴史を網羅的に整理し、memory/system-architecture.md に保存。プロジェクトの認識ズレを防ぐための Single Source of Truth を構築する。定期的に実行してナレッジを最新化。ソフトウェアプロジェクト・法務案件・事業計画・文書管理プロジェクトに対応。
argument-hint: [対象ディレクトリ（省略時はカレント）]
---

# Map Project: $ARGUMENTS

プロジェクトの全体像を**3名のExploreエージェント並行調査**で把握し、永続的なシステム設計書を生成する。

> **注意**: agent-teams.mdの「議論フロー」とは異なり、**並行調査モデル**。
> メンバー間の議論は不要。各自が独立に調査し、オーケストレータが統合する。

---

## Phase 1: 準備（オーケストレータが実行）

1. **対象ディレクトリ**: `$ARGUMENTS` 指定時はそのディレクトリ、省略時はcwd
2. **種別推定**: ファイル存在で自動判定（詳細: `references/project-type-detection.md`）
3. **既存ナレッジ収集**: system-architecture.md / MEMORY.md / CLAUDE.md / restart.md / git log（存在時のみ）
4. **ゴール確認**: 既存有→引き継ぎ、無→推定+`(要確認)`、推定不可→AskUserQuestion
5. **メモリディレクトリ**: `~/.claude/projects/` から特定、なければ `memory/` 作成

---

## Phase 2: 並行調査（Exploreエージェント3名）

オーケストレータが3名のExploreエージェントを **Agentツールで同時起動**（`run_in_background=true`）。各メンバーは独立に調査し、結果をオーケストレータに返す。メンバー間の相互通信・議論は行わない。

### 2.1 エージェント指示の読み込みとスポーン

種別に応じた指示ファイルを読み込み、各エージェントのプロンプトとして使用する:

| 種別 | 指示ファイル | エージェント名 |
|------|------------|--------------|
| ソフトウェア | `agents/software.md` | Architect, DataModeler, IntegrationAnalyst |
| legal / documents | `agents/legal.md` | DocumentStructureAnalyst, CaseAnalyst, RelationshipAnalyst |
| business-plan | `agents/business.md` | BusinessModelAnalyst, MarketAnalyst, StakeholderAnalyst |

**指示ファイルのパス**: `~/.claude/skills/map-project/agents/{type}.md`

各エージェントのプロンプト冒頭に `{対象ディレクトリ}` と `{判定結果}` を埋め込む。

### 2.2 結果収集

3名の報告が揃うのを待つ。

---

## Phase 2.5: コンテキストインタビュー（オーケストレータが実行）

> **目的**: ファイル探索では得られない「人・関係性・動機・地雷」をユーザーから引き出し、設計書のセクション0に保存する。
> **これが最重要フェーズ**。セクション0の質がセッション間の文脈維持を決定する。

### 2.5.1 インタビュー実施判定

- セクション0が**ない** → インタビュー実施（必須）
- セクション0が**ある** → スキップし内容を引き継ぐ（`(要確認)` タグ項目は再質問）

### 2.5.2 質問の生成（仮説確認型）

> **CRITICAL**: オープンクエスチョン禁止。探索結果から仮説を立て、「○○と推測しましたが合っていますか？」の形式で聞く。

**必須質問（3問、全プロジェクト共通）:**

1. **関係者と関係性（仮説付き）**: 探索で人名・組織名を収集し仮説を立てる
2. **地雷・教訓（仮説付き）**: エラーログ・修正履歴・MEMORY.mdの注意事項から仮説を立てる
3. **ストーリーと成功（仮説付き）**: プロジェクトの背景・動機・成功の定義を推測

**条件付き質問（0〜2問追加）**: 提案書・複数サービス・金額・外部パートナーを検出した場合

### 2.5.3 質問の実施

- 全質問をまとめて1回のメッセージで提示（1問ずつ聞かない）
- **フォールバック**: 仮説不可の場合のみオープンクエスチョン許可（前置き必須）

### 2.5.4 回答の記録形式

```markdown
## 0. コンテキスト（人・関係性・トーン）

### 一言ストーリー
### 関係者マップ
| 名前 | 役割 | 人となり・関係性 | 呼び方の注意 |
### トーン・アプローチ
### 地雷・過去の教訓
### 成功の定義
```

---

## Phase 3: 統合 & ドキュメント生成（オーケストレータが実行）

### 3.1 報告の統合

矛盾判定の優先順位（種別ごと）:
- **ソフトウェア**: IntegrationAnalyst > DataModeler > Architect
- **法務・文書**: CaseAnalyst > RelationshipAnalyst > DocumentStructureAnalyst
- **事業計画**: BusinessModelAnalyst > MarketAnalyst > StakeholderAnalyst

### 3.2 system-architecture.md 生成

出力先: `{メモリディレクトリ}/system-architecture.md`。種別に応じたテンプレートを使用:

**テンプレートのパス**: `~/.claude/skills/map-project/templates/{type}.md`（software / legal / business）

### 3.3 記述ルール

信頼度タグ（タグなし/書面参照/推定/要確認）、ソース参照義務、値の非記載ルール等。

> 詳細: `references/documentation-standards.md`
> 法務案件の追加ルール: `references/legal-documentation-rules.md`

---

## Phase 4: MEMORY.md 更新（オーケストレータが実行）

MEMORY.md に `system-architecture.md` への参照を追加（未登録時）、重複情報を削除（DRY）、200行以内に整理。

---

## Phase 4.5: エージェントメモリ伝播（オーケストレータが実行）

調査結果から関連する専門エージェントを特定し、各エージェントのメモリに5行以内で要点を追記する。

> 詳細: `references/agent-memory-propagation-matrix.md`

---

## Phase 5: 完了報告

出力パス・セクション数・検出種別・エージェントメモリ伝播先を表示。

---

## 使い方

`/map-project` (cwd調査) / `/map-project /path/to/project` (指定ディレクトリ調査)

推奨タイミング: 初回セッション / 大きな機能追加後 / 外部連携変更時 / 法務: 期日前・新文書提出後

既存 `system-architecture.md` がある場合は**差分更新**（変更セクションのみ更新、廃止セクション削除）。
