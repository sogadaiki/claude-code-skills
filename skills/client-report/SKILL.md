---
name: client-report
description: クライアント向け月次/週次レポートを生成。広告KPI、LINE登録数、SNSパフォーマンス、Googleマップ広告の数値をまとめ、次月アクション提案を含むレポートを作成。
argument-hint: [client-name: client-a|client-b|client-c] [--weekly|--monthly]
---

# Client Report: $ARGUMENTS

クライアント向けの定期レポートを生成する。

## 参照ナレッジ

引数のクライアント名に応じて読み込み:
- `client-a` → `~/.claude/knowledge/client-client-a.md`
- `client-b` → `~/.claude/knowledge/client-client-b.md`
- `client-c` → `~/.claude/knowledge/client-client-c.md`

共通参照:
- `~/.claude/knowledge/google-ads-automation.md` -- 広告データ取得方法

---

## Multi-Pass Design

既存ワークフロー（Phase 1-3）を多段階原則にマッピング。

| Pass | 内容 | 対応Phase |
|------|------|-----------|
| **Pass 1: 素材収集** | クライアント情報読込、データ収集並列（riku:広告 / kaito:GA4・LINE・マップ） | Phase 1 |
| **Pass 2: 分析 + 自己批判** | KPI集計テーブル作成→異常値検出→原因推定。「数字の根拠あるか？」「季節変動を考慮しているか？」「原因推定が断定になっていないか？」を自己批判 | Phase 2 |
| **Pass 3: 生成** | kenjiにマーケティング提案委譲→レポートHTML生成→Drive アップロード | Phase 3 |
| **Pass 4: 品質ゲート** | 以下のチェック項目で検証 | Phase 3後 |
| **Pass 5: 修正** | FAIL項目のみ修正→再検証。2回FAILで該当セクションのデータ再取得から | — |

### Pass 4 品質ゲート項目（レポート固有）

| # | チェック項目 | FAIL基準 |
|---|------------|---------|
| 1 | KPI正確性 | 数値がPhase 1の生データと不一致 |
| 2 | 前月比計算 | 計算ミス（四捨五入含む） |
| 3 | 異常値の原因推定 | 推定なし / 断定口調で書かれている |
| 4 | 次月アクション | 抽象的（「改善する」レベル）/ 実行不可能 |
| 5 | グラフ・テーブル整合 | グラフの数値とテーブルの数値が不一致 |
| 6 | 機密情報 | 内部KPI・コスト構造がレポートに混入 |

---

## Phase 1: データ収集・検証（サブエージェント並列）

### Step 1.1: クライアント情報読み込み

クライアントナレッジファイルから以下を把握:
- 契約内容・KPI目標
- 広告アカウント情報
- 過去のレポート実績

### Step 1.2: データ収集（並列）

**riku-ad-analyst** に委譲:
- Google Ads パフォーマンスデータ
- CPA / CTR / CVR の推移
- 予算消化状況

**kaito-data-analyst** に委譲:
- GA4データ（セッション、CV、流入経路）
- LINE登録数の推移
- Googleマップ広告の表示回数・クリック数

### Phase 1 出力フォーマット（各エージェント）

```json
{
  "agent": "riku|kaito",
  "period": "YYYY-MM",
  "raw_data": {
    "metrics": [
      {
        "name": "CPA|CTR|CVR|sessions|conversions|...",
        "current": 0,
        "previous": 0,
        "target": 0
      }
    ]
  },
  "data_quality_notes": ["欠損データや異常値に関する注記"]
}
```

---

## Phase 2: KPI分析・解釈（新規段階）

Phase 1 の生データを受け取り、以下の分析を行う。この段階はメインエージェントが実行する。

### Step 2.1: KPI集計テーブル

| KPI | 前月 | 今月 | 目標 | 達成率 | 前月比 |
|-----|------|------|------|--------|--------|
| ... | ... | ... | ... | ...% | +/-...% |

前月比較・目標対比を算出。

### Step 2.2: 異常値検出

以下の閾値で異常値を自動検出する:

| 指標 | 警告閾値（前月比） | 危険閾値（前月比） |
|------|--------------------|--------------------|
| CPA | +20%以上 | +50%以上 |
| CTR | -15%以上低下 | -30%以上低下 |
| CVR | -15%以上低下 | -30%以上低下 |
| セッション数 | -20%以上低下 | -40%以上低下 |
| LINE登録数 | -30%以上低下 | -50%以上低下 |
| 予算消化率 | 120%超過 or 70%未満 | 150%超過 or 50%未満 |

閾値は目安であり、季節変動やキャンペーン施策を考慮して判断すること。

### Step 2.3: 原因推定

検出された異常値に対し、以下の観点で原因を推定する:

1. **外部要因**: 季節性、市場トレンド、競合の動き
2. **内部要因**: 広告設定変更、LP変更、予算変更
3. **データ要因**: 計測タグの不備、GA4設定の問題

原因推定は「可能性」として提示し、断定しない。

### Phase 2 出力フォーマット

```json
{
  "kpi_summary": [
    {
      "name": "CPA",
      "current": 3500,
      "previous": 2800,
      "target": 3000,
      "achievement_rate": "85.7%",
      "mom_change": "+25.0%",
      "status": "warning|danger|normal"
    }
  ],
  "anomalies": [
    {
      "metric": "CPA",
      "severity": "warning|danger",
      "change": "+25.0%",
      "possible_causes": [
        "競合の入札強化による平均CPC上昇",
        "新規キーワード追加による学習期間"
      ]
    }
  ],
  "period_context": "キャンペーン施策や季節要因など、数値に影響する背景情報"
}
```

---

## Phase 3: レポート生成・提案（拡張）

Phase 2 のKPI分析結果を入力とし、レポートHTML生成と次月アクション提案を行う。

### Step 3.1: マーケティング提案（サブエージェント）

**kenji-marketing-strategist** に委譲:

入力: Phase 2 の出力JSON（KPIサマリー + 異常値 + 原因推定）

kenji の担当:
- KPI解釈を受け取り「次月アクション Top 3」を優先度付きで提案
- チャネル別施策提案（広告/SEO/SNS/LINE/マップ）
- 異常値に対する具体的な改善アクション

kenji 出力フォーマット:
```json
{
  "top3_actions": [
    {
      "priority": 1,
      "action": "具体的なアクション内容",
      "expected_impact": "期待される効果",
      "effort": "high|medium|low"
    }
  ],
  "channel_recommendations": {
    "ads": "広告に関する提案",
    "seo": "SEOに関する提案",
    "sns": "SNSに関する提案",
    "line": "LINEに関する提案",
    "map": "マップに関する提案"
  }
}
```

### Step 3.2: レポートHTML生成

Phase 2 のKPI分析 + Phase 3.1 のマーケティング提案を統合し、HTMLレポートを生成する。

レポートを以下のパスに保存:
```
<YOUR_PROJECT_ROOT>/clientwork/{client-name}/reports/
├── YYYY-MM-monthly-report.html  (月次)
└── YYYY-WXX-weekly-report.html  (週次)
```

レポート構成:
1. **サマリー**: KPIハイライト + 異常値アラート（赤/黄で視覚表示）
2. **KPI詳細テーブル**: 前月比・目標対比・達成率
3. **異常値分析**: 検出された異常値と原因推定
4. **広告パフォーマンス詳細**
5. **ウェブサイト分析**
6. **SNS/LINE/マップ実績**
7. **次月アクション提案 Top 3**（kenjiの提案を反映）
8. **チャネル別施策提案**

### Step 3.3: Google Driveアップロード

Google Workspace MCPでクライアントフォルダにアップロード:
- クライアントA: フォルダID `<YOUR_DRIVE_FOLDER_ID>`
- クライアントB: フォルダID `<YOUR_DRIVE_FOLDER_ID>`
- client-c: フォルダID `<YOUR_DRIVE_FOLDER_ID>`

### Step 3.4: 完了報告

- レポートファイルパス
- アップロード先URL
- 異常値サマリー（検出された場合）
- 次月の重点アクション Top 3
