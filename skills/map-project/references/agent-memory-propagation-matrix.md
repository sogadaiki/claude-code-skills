# エージェントメモリ伝播（Phase 4.5 詳細）

調査結果のうち、専門エージェントの知識として有用な情報を各エージェントのメモリに伝播する。

## 4.5.1 関連エージェントの特定

生成した `system-architecture.md` の内容から、関連するエージェントを自動判定する:

| 検出した要素 | 関連エージェント | 伝播する情報 |
|-------------|-----------------|-------------|
| 広告運用（Google Ads, Meta Ads等） | riku-ad-analyst | 広告アカウント構成、KPI、予算 |
| GA4/Analytics連携 | kaito-data-analyst | 計測設定、主要KPI、データソース |
| 収支・売上データ | rina-finance-advisor | 料金体系、収支構造、KPI |
| SEO施策・コンテンツ | seo-specialist | サイト構造、対象キーワード、技術SEO設定 |
| SNS運用・マーケ施策 | kenji-marketing-strategist | チャネル、施策、ターゲット |
| 契約・法的要件 | hiroshi-legal-advisor | 契約形態、法的リスク、規制 |
| 認証・セキュリティ設計 | samurai-security-auditor | 認証方式、RLS、API保護 |
| UI/フロントエンド設計 | yui-ux-designer | 画面構成、デザインシステム |
| 技術スタック・アーキテクチャ | spec-tech-selection | 技術選定理由、依存関係 |
| 事業戦略・競合 | mei-business-strategist | ビジネスモデル、競合、市場 |

## 4.5.2 メモリ更新の実行

関連エージェントが特定されたら:

0. **ディレクトリの存在確認と作成**:
   ```bash
   mkdir -p ~/.claude/agent-memory/{agent}/
   ```
1. `~/.claude/agent-memory/{agent}/MEMORY.md` を読む（なければ作成）
2. プロジェクト名のセクションを探す（なければ追加）
3. 該当情報を **5行以内** で追記する（詳細はsystem-architecture.mdへリンク）
4. 形式:
   ```markdown
   ## {プロジェクト名} ({日付})
   - {要点1}
   - {要点2}
   - 詳細 → `{system-architecture.mdのパス}`
   ```

## 4.5.3 制約

- **値は書かない**: APIキー、売上金額の具体値、個人情報は伝播しない
- **5行以内**: 詳細はリンクで参照。メモリを肥大化させない
- **既存情報の上書き禁止**: 追記のみ。既存のエージェント知識を消さない
- **対象エージェントが0件の場合**: このフェーズをスキップ
