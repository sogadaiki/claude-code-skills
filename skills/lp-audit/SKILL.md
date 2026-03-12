---
name: lp-audit
description: 求人LPの離脱診断・改善提案。Playwrightでスマホ/PCのスクショ取得、ユーザーフロー自動トレース、離脱ポイント特定、競合LP比較を実施。
argument-hint: [URL]
---

# LP Audit: $ARGUMENTS

LPの総合監査（離脱診断 + 改善提案）を実施する。

## 参照ナレッジ
- `~/.claude/knowledge/seo-best-practices.md` — SEO基準
- `~/.claude/knowledge/document-design.md` — デザイン評価基準

## フロー

### Step 1: スクリーンショット取得（Playwright）

`/webapp-testing` スキルのPlaywright機能を使い、以下を取得:
- スマホ表示（375px）のフルページスクショ
- PC表示（1440px）のフルページスクショ
- 各CTAボタンのクリック後遷移先

### Step 2: 技術SEO監査（seo-specialist）

seo-specialistエージェントに委譲:
- Core Web Vitals パターン検出
- メタタグ・OGP確認
- 構造化データ確認
- モバイルフレンドリーチェック
- ページ速度影響要因

### Step 3: UX/デザイン評価（yui-ux-designer）

yui-ux-designerエージェントに委譲（スクショを渡す）:
- ファーストビューの訴求力
- CTA配置・視認性
- 情報階層の適切さ
- モバイルでの操作性
- フォーム入力のハードル

### Step 4: コピー/CTA分析（kenji-marketing-strategist）

kenji-marketing-strategistエージェントに委譲:
- ヘッドラインの訴求力
- CTA文言の効果
- 社会的証明の有無
- 緊急性・限定性の演出
- ペルソナとのメッセージ適合性

### Step 5: クロス分析（統合分析エージェント）

Step 2-4 の3分野の分析結果を横断的に再評価する。各分野を個別に見るだけでは見逃す「分野間の相互影響」を特定する。

#### 入力
Step 2-4 の各エージェント出力（JSON中間出力）

#### 分析観点
- **ファネル全体でのボトルネック再評価**: 技術SEOが優秀でもコピーが弱ければ流入は来ても離脱する。逆にコピーが強くても表示速度が遅ければ到達しない。全体CVRの視点で各分野スコアを再重み付けする
- **矛盾・相殺の検出**: UXが「シンプルさ」を評価しているのにコピーが「情報量不足」を指摘している場合など、分野間で矛盾する評価を検出して統合判断を下す
- **相乗効果の特定**: ある改善が複数分野に波及するケース（例: ページ速度改善 → UXスコア向上 + SEOスコア向上）を特定する
- **クロススコアリング**: 各分野の個別スコアを全体CVR影響度で再スコアリング（0-100）

#### JSON中間出力フォーマット
```json
{
  "cross_analysis": {
    "funnel_bottleneck": {
      "primary_bottleneck": "string — ファネル上の最大ボトルネック段階",
      "flow_impact_chain": ["string — 影響の連鎖を記述"],
      "estimated_cvr_loss_pct": "number — この段階での推定CVR損失率"
    },
    "contradictions": [
      {
        "field_a": "string — 分野名",
        "field_b": "string — 分野名",
        "issue": "string — 矛盾の内容",
        "resolution": "string — 統合判断"
      }
    ],
    "synergies": [
      {
        "improvement": "string — 改善内容",
        "affected_fields": ["string — 波及する分野名"],
        "combined_impact_score": "number (0-100)"
      }
    ],
    "cross_scores": {
      "technical_seo": { "raw_score": "number", "cvr_weighted_score": "number" },
      "ux_design": { "raw_score": "number", "cvr_weighted_score": "number" },
      "copy_cta": { "raw_score": "number", "cvr_weighted_score": "number" },
      "overall_cvr_score": "number (0-100)"
    }
  }
}
```

### Step 6: 改善シナリオ生成（kenji-marketing-strategist）

Step 5 のクロス分析結果をもとに、具体的な改善シナリオを複数生成する。kenji-marketing-strategistエージェントに委譲。

#### 入力
Step 5 のクロス分析JSON + Step 2-4 の個別分析結果

#### 生成内容
- **CVR改善予測**: Critical Issues を改善した場合の CVR 改善率を推定（ベースライン比 +X%）。推定根拠（業界ベンチマーク・類似事例・ヒューリスティクス）を明記
- **Quick Wins 特定**: 実装コスト（工数）が低く効果が高い施策をランキング（例: H1修正で見出し認識度+15%推定）
- **A/Bテスト提案**: 効果が不確実な改善についてテスト設計を提案（バリアント・KPI・最小サンプルサイズ）
- **シナリオ比較**: 最低3つの改善シナリオを生成し、コスト・期待効果・リスクで比較

#### JSON中間出力フォーマット
```json
{
  "improvement_scenarios": {
    "baseline": {
      "current_estimated_cvr_pct": "number",
      "estimation_basis": "string — 推定根拠"
    },
    "scenarios": [
      {
        "id": "string — scenario_1, scenario_2, ...",
        "name": "string — シナリオ名",
        "description": "string — 概要",
        "changes": [
          {
            "target": "string — 改善対象",
            "action": "string — 具体的な変更内容",
            "field": "string — technical_seo | ux_design | copy_cta",
            "effort_hours": "number — 推定工数（時間）",
            "expected_cvr_delta_pct": "number — CVR改善幅（%）",
            "confidence": "high | medium | low"
          }
        ],
        "total_effort_hours": "number",
        "expected_cvr_after_pct": "number",
        "risk_notes": "string"
      }
    ],
    "quick_wins": [
      {
        "action": "string",
        "effort_hours": "number",
        "expected_impact": "string",
        "priority_rank": "number"
      }
    ],
    "ab_test_proposals": [
      {
        "hypothesis": "string",
        "variant_a": "string — コントロール",
        "variant_b": "string — テストバリアント",
        "primary_kpi": "string",
        "min_sample_size": "number",
        "estimated_duration_days": "number"
      }
    ]
  }
}
```

### Step 7: 統合レポート生成

Step 2-6 の全分析結果を統合し、HTMLレポートを生成:

```
<YOUR_PROJECT_ROOT>/clientwork/{project}/
└── lp-audit-YYYY-MM-DD.html
```

レポート構成:
1. 総合スコア（0-100）— Step 5 のクロススコアを使用
2. Critical Issues（即座に修正すべき問題）
3. 離脱ポイント分析（ファネル各段階）— Step 5 のファネルボトルネック分析を反映
4. 競合比較（同業種LPとの比較）
5. 改善シナリオ比較表（Step 6 のシナリオ3案以上を表形式で表示）
6. Quick Wins ランキング（Step 6 の優先度順）
7. A/Bテスト提案（Step 6 のテスト設計）
8. 分野間矛盾・相乗効果の注記（Step 5 の検出結果）

### Step 8: ユーザーへの報告

- 総合スコア（CVR加重）とTop 3改善項目を提示
- 推奨シナリオ（コスト対効果が最も高いもの）を提案
- Quick Wins リスト（すぐ着手できる施策）を提示
- レポートファイルパスを共有
