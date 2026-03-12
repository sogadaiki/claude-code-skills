<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
# 学習更新・ゴール追跡 詳細手順

> SKILL.md Step 4.1-4.5 の詳細。学習ファイル更新パス・計算式・エスカレーション判定。

## 4-1. memory/howto/ 更新

`memory/howto/x-writing.md` に「何が効いたか」パターンを**最低1エントリ**追記:
- 今日の投稿で得られた知見
- フォーマット: `- [YYYY-MM-DD] {カテゴリ}: {パターン名} -- {詳細}`

`memory/howto/hook-patterns.md` のパフォーマンスデータを更新(データがある場合)

## 4-2. memory/process/quality-evolution.md 更新

品質トレンドを追記:
- 本日のQGパス率、リトライ回数
- フォーマット: `| YYYY-MM-DD | N本 | パス率 | 主な指摘 |`

## 4-3. goals.json 更新 + フェーズ整合性チェック

`$HOME/.ai-persona/goals.json` を読み込み、以下を更新:

**実データ計算（v3）**: `content/posted/` の実ファイルからカテゴリ配分を正確に再計算する。
1. 直近7日間のposted/ファイルを列挙（ファイル名の日付で判定）
2. 各ファイルの「カテゴリ:」行を読み取り、A/B/C/D別にカウント
3. 計算結果をgoals.jsonに反映:

- `weekly_targets.posts_per_week.actual`: 直近7日間のposted/ファイル数
- `weekly_targets.category_balance.B_actual_pct`: B本数 / 全体本数 * 100
- `weekly_targets.category_balance.C_actual_pct`: C本数 / 全体本数 * 100
- `weekly_targets.category_balance.D_actual_pct`: D本数 / 全体本数 * 100
- `last_updated`: 今日の日付

**フェーズ整合性クロスバリデーション**:
1. `goals.json.phase` と `state.json.current_phase` を比較
2. 不一致の場合:
   - Discord警告を送信:
     ```bash
     node $HOME/.ai-persona/scripts/discord-notify.mjs send \
       --persona manager --preset warning \
       --title "[evening-session] フェーズ不一致検出" \
       --content "goals.json: {goals.phase} vs state.json: {state.current_phase}。state.jsonを正として修正します"
     ```
   - `goals.json.phase` を `state.json.current_phase` に合わせて更新
3. `MEMORY.md` のフェーズ・投稿数も `state.json` の値に合わせて同期更新
   - フォロワー数: 手動更新のみ（X API Free tierでは自動取得不可）
   - 投稿数: `state.json.stats.total_posts` の値を反映

**エスカレーション判定**:
- `posts_per_week.actual < 14` → Discord通知（オーナーアラート）
- `state.json.errors.consecutive_failures >= 5` → Discord通知（オーナーアラート）

## 4-4. strategy/active-experiments.md 更新

- 実行中の実験の進捗データを追記
- 検証期間終了の実験 → `concluded`に変更、結果と学びを記入
- 新しい仮説があれば新実験を追加

## 4-5. manager-log 出力

`manager-log/YYYY-MM-DD.md` (今日の日付)に以下を記録:
```markdown
# Manager Log: YYYY-MM-DD

## Summary
- Posts today: N
- Approved for tomorrow: N (morning/noon/evening)
- Quality Gate: X/X passed
- Goals: follower {current}/{target}

## Analysis
{Step 1の分析要約}

## Knowledge Updates
- {更新ファイルリスト}

## Tomorrow
- morning: {テーマ} ({カテゴリ})
- noon: {テーマ} ({カテゴリ}) {画像: あり/なし}
- evening: {テーマ} ({カテゴリ})
```
