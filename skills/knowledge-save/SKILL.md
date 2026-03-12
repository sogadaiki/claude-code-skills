---
name: knowledge-save
description: 個人ナレッジの保存。業務で得た知見、出来事、意思決定の記録をメモリに蓄積。LLMにはない「自分固有の情報」を保存し、未来のセッションで活用可能にする。
argument-hint: [カテゴリ: business|finance|legal|marketing|personal|decision]
---

# ナレッジ保存

## カテゴリ: $ARGUMENTS

ユーザーから共有された情報を、以下のメモリファイルに適切に保存します。

## 保存先マッピング

| カテゴリ | 保存先 | 用途 |
|----------|--------|------|
| business | ~/.claude/knowledge/business.md | 事業に関する事実、KPI、取引先情報 |
| finance | ~/.claude/knowledge/finance.md | 財務データ、売上、コスト、資金調達情報 |
| legal | ~/.claude/knowledge/legal.md | 契約、規制、コンプライアンス情報 |
| marketing | ~/.claude/knowledge/marketing.md | マーケティングデータ、SNS実績、メディア実績 |
| personal | ~/.claude/knowledge/personal.md | 個人の経歴、ネットワーク、スキル |
| decision | ~/.claude/knowledge/decisions.md | 重要な意思決定の記録（日付、理由、結果） |

## 保存フォーマット

各エントリは以下の形式で記録:

```markdown
## [日付] タイトル

**コンテキスト**: なぜこの情報が重要か
**内容**: 具体的な情報
**ソース**: どこから得た情報か
**関連項目**: 他のナレッジとの関連
```

## 保存ルール

1. 事実と意見を明確に分離する
2. 数値データは必ず日付と出典を添える
3. 古い情報が更新された場合は上書きではなく追記（変遷を追えるように）
4. 機密性の高い情報（パスワード、API key等）は絶対に保存しない
5. 200行を超えたファイルはサマリーセクションを冒頭に追加

## 使い方

```
/knowledge-save business
> 今月のAI顧問契約が3社増えて合計8社になった。月額合計120万円。
```

ユーザーの入力を解析し、適切なファイルに構造化して保存します。

---

## 用途判定（保存後に必ず実行）

保存したナレッジについて以下を判断し、適切な接続を行う。

### a) 判定基準

| 判定 | 基準 | アクション |
|------|------|----------|
| スキル接続 | 特定スキル実行時に読み込むべき | そのスキルのSKILL.mdに `参照ナレッジ` として追記 |
| エージェント接続 | 特定エージェントの判断に必要 | エージェント設定の参照ナレッジとして記録 |
| 参照のみ | 必要時に手動で読めばよい | ナレッジマッピング表に記録のみ |
| 不要 | 保存したが活用場面がない | 保存を取り消し or `archive/` に移動 |

### b) 接続先マッピング表

保存したナレッジのカテゴリから、接続先を特定する:

| ナレッジカテゴリ | 接続先スキル | 接続先エージェント |
|----------------|-------------|-----------------|
| SEO/AEO | /seo | seo-specialist |
| 広告運用 | /client-report | ad-analyst |
| デザイン | /benchmark-lp, /revealjs | ux-designer |
| 事業戦略 | /business-review, /morning | business-strategist |
| 財務 | /financial-model | finance-advisor |
| 法務 | /business-review | legal-advisor |
| マーケ | /write-blog, /write-sales-copy | marketing-strategist |
| クライアント | /client-report | ad-analyst, data-analyst |
| 技術設計 | /plan | tech-selection |
| セキュリティ | /review | security-auditor |
| YouTube | /youtube-script | -- |
| SNS | /social-post | marketing-strategist |
| インフラ | -- | tech-selection |

### c) 実行手順

1. 保存完了後、上記マッピング表を参照して接続先を特定
2. **スキル接続の場合**: 該当スキルの SKILL.md に `参照ナレッジ` セクションがあれば追記、なければ新設
3. **エージェント接続の場合**: エージェント設定の該当エージェント欄に参照ナレッジとして記録
4. **参照のみの場合**: ナレッジマッピング表に記録
5. 「ナレッジを保存し、[接続先] に参照を追加しました」と報告

### d) 接続判断に迷った場合

- 「このナレッジが更新されたとき、どのスキル/エージェントの挙動が変わるべきか？」で考える
- 変わるべきスキル/エージェントがあれば → スキル接続 or エージェント接続
- 特にないなら → 参照のみ

### e) INDEX.md 更新

1. `~/.claude/knowledge/INDEX.md` を読む
2. 保存先ファイルがどのクラスタに属するか確認
3. 既存クラスタにあれば → 何もしない
4. 適切なクラスタはあるがファイルが未登録 → ファイルリストに追加
5. どのクラスタにも合わない → 新クラスタ作成 + 関連 `also:` を追加
