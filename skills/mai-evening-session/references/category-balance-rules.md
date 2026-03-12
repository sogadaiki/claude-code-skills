<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
# カテゴリ配分 v3 強制ロジック

> SKILL.md Step 3-0 の詳細。ブリーフィング生成時のカテゴリ配分チェック。

## ブリーフィング生成手順

1. `briefing/YYYY-MM-DD.md` (明日の日付)を生成
2. **カテゴリ配分チェック（v3強制ロジック）**:
   - `content/posted/` から直近7日間のファイルを読み込み、メタ情報の「カテゴリ」行を集計
   - 以下の強制ルールを適用:

   | 条件 | 強制アクション |
   |------|-------------|
   | B > 30%（直近7日） | 明日のmorning/noonをA or Cに強制 |
   | C = 0本（直近5日） | 明日のmorningをCに強制 |
   | D = 0本（直近5日） | 明日のeveningをDに強制 |

   - ブリーフィングに `(強制: B過多のためC指定)` vs `(推奨)` を明記

## テーマ選定ロジック

- 直近3日間で使っていないカテゴリを優先
- 週間テーマテーブルの曜日テーマに合致
- 木曜の場合、木夜ゴールデンタイム用ベストコンテンツ案を含める
- **強制指定されたカテゴリは変更不可**
- 3テーマをスロット別に割り当て

## 週末の場合

**翌日が平日（月曜）の場合**: 通常通り3投稿(morning/noon/evening)を生成する。
**翌日も休日（土曜eveningで日曜分を生成する場合）**: 2投稿(evening + weekend)を生成する。

判定ロジック:
- 日曜evening → 翌日は月曜(平日) → 3投稿生成（morning/noon/evening）
- 土曜evening → 翌日は日曜(休日) → 2投稿生成（evening + weekend）
