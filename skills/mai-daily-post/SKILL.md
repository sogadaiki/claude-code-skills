<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
---
name: mai-daily-post
description: "Automation-only: AI persona X (Twitter) auto-posting pipeline. Run via orchestrator with `claude -p`. Do NOT trigger for interactive sessions. BEFORE asking the user any question, resolve it yourself."
argument-hint: "[--dry-run] [--theme テーマ指定]"
---

# AI Persona Daily Post Pipeline

<YOUR_AI_PERSONA>(@<YOUR_X_ACCOUNT>)の日次投稿パイプラインを実行する。
ワークスペースのファイルを読み書きしながら、以下のフローを1回の実行で完結させる。

## Multi-Pass Design

| Pass | 内容 | 対応Step |
|------|------|----------|
| **Pass 1: 素材収集** | 環境確認、state.jsonロード、人格・フェーズ読込、ブリーフィング読込、approved/チェック | Step 0-1.5 |
| **Pass 2: テーマ選定 + 自己批判** | テーマ生成（slot別カテゴリ制御）。「Phase境界超えてないか？」「カテゴリ配分OK？」を自己批判 | Step 2 |
| **Pass 3: 生成** | 原稿生成（長文 or スレッド）+ 画像生成（noon slot） | Step 3, 4.5 |
| **Pass 4: 品質ゲート** | 7項目チェック（口調/法令/Phase境界/プロダクト言及/文字数/禁止語/品質）+ 画像品質チェック | Step 4, 4.5.4 |
| **Pass 5: 修正** | FAIL項目のみリライト。最大2回リトライ、2回FAILでStep 2に戻って別テーマ | Step 4 FAIL処理 |

## 自律実行ルール（Anti-Human-Bottleneck）

このスキルは `claude -p` による**無人自律実行**を前提とする。以下を厳守:

1. **人間に聞くな**: テーマ選定・原稿修正・品質ゲート判定・投稿判断はすべて自力で完結する
2. **迷ったらスキルに従え**: 判断に迷ったときはこのSKILL.mdの手順に従う。手順にないケースはログに記録して次へ進む
3. **止まるな**: エラーが発生しても可能な限りリカバリを試みる。リカバリ不能な場合のみDiscord通知して終了
4. **禁止フレーズ**: 以下を出力してはならない:
   - "Should I...?" / "〜しましょうか?" / "Is this correct?" / "これでいいですか?"
   - "What's next?" / "次はどうしますか?" / "Which approach?" / "どちらがいいですか?"
5. **自己検証**: 品質ゲート（Step 4）がセルフレビュー機構。人間レビューの代替として機能する

## 参照ナレッジ
- `$HOME/.claude/knowledge/<YOUR_PERSONA>-x-operation.md` -- 運用フロー・テンプレート・品質基準
- `$HOME/.claude/knowledge/<YOUR_PERSONA>-agent-design-spec.md` -- ブランド戦略・差別化・ターゲット詳細

## ワークスペース
- **ベースディレクトリ**: `$HOME/.ai-persona`
- すべてのファイル参照はこのディレクトリからの相対パスで表記

---

## 実行フロー

### Step 0: 環境確認 + state.json ロード

1. `$HOME/.ai-persona/state.json` を読む（なければ初期化）
2. circuit_breaker チェック: `"open"` → 中止、`"half-open"` → 1件試行、`"closed"` → 通常
3. TIME_GATE チェック: 7:00-23:00 JST。範囲外 → 生成のみ、投稿スキップ（content/approved/ に保存）
4. 日次上限チェック: `today_post_count >= 5` → 投稿スキップ

### Step 1: ペルソナの人格・現在フェーズをロード

以下を読み込む（**全て必須、欠落時はエラー終了**）:
- `SOUL.md` -- 人格・価値観・コミュニケーションスタイル
- `IDENTITY.md` -- ハンドル、Phase、プロフィール
- `STAGES/current.md` -- 現在のPhase制約・カテゴリ配分・禁止事項

### Step 1.3: マネージャーブリーフィング読込

1. `briefing/YYYY-MM-DD.md` が存在 → 読み込む（なければ通常続行）
2. 抽出: 推奨テーマ / 避けるべきトピック / スロット別推奨 / 注力カテゴリ / 特記事項
3. 週次ブリーフィング `briefing/YYYY-MM-DD-weekly.md` も存在すれば追加で読み込む

### Step 1.5: approved/コンテンツ優先チェック

1. `content/approved/` にファイルがあるか確認
2. ある場合: 現在の `--slot` に合うコンテンツを選択 → **Step 5に直行**
3. ない場合 → Step 2へ

> Consult `references/slot-control-matrix.md` for: スロット別マッチングルール、在庫消化ルール

### Step 2: テーマ生成

**STAGES/current.md のカテゴリ配分に従ってテーマを選定する。**
**ブリーフィングに `(強制: ...)` 指定がある場合、そのカテゴリは変更不可。**

Phase 0 カテゴリバランス: A/B/C/Dが満遍なく出現。C(憧れ)とD(行動促進)が不足しやすい。
C投稿で「確実に稼げる」「今だけ特別」等の扇動表現は絶対NG。

1. `activity/` の直近ログで重複回避
2. カテゴリ選定（slot別の優先ルールあり）
3. テーマ要件: 1投稿=1メッセージ、Before/After定義、禁止事項非抵触、ターゲット共感
4. フック文生成（280文字以内）
5. 投稿形式判定（スレッド vs 長文）
6. 3パス品質最大化: 初期案 → 批判的レビュー(5批判) → 最終修正

出力を `content/drafts/YYYY-MM-DD-topic.md` に保存。

> Consult `references/slot-control-matrix.md` for: slot別カテゴリ選定ルール、テンプレート番号対応

### Step 3: 原稿生成

テーマの投稿形式に応じて原稿を生成する。
出力を `content/drafts/YYYY-MM-DD-{slug}-draft.md` に保存。

> Consult `references/writing-rules.md` for: 長文/スレッド構成、口調ルール、キャラクター言及ルール、禁止表現

### Step 4: 品質ゲート（最大2回リトライ）

以下の7チェックを**全て**実行する。**1つでもFAILなら全体FAIL。**

| # | チェック項目 | FAIL基準 |
|---|------------|---------|
| 1 | TONE_CHECK | ペルソナの口調から逸脱（敬語混在、一人称不統一） |
| 2 | LEGAL_CHECK | 法令違反表現（`/確実に稼げ/`, `/月収\d+万円保証/`, `/絶対に?儲かる/`, `/リスクゼロ/`等） |
| 3 | BOUNDARY_CHECK | Phase境界超過 + 禁止キーワード: `<YOUR_FORBIDDEN_KEYWORDS>` |
| 4 | PRODUCT_CHECK | Phase 0での特定プロダクト言及 |
| 5 | LENGTH_CHECK | 長文: 1,000-25,000字 / スレッド各ツイート: 280字以内 |
| 6 | FORBIDDEN_CHECK | `<YOUR_FORBIDDEN_PHRASES>` |
| 7 | CONTENT_QUALITY_CHECK | 5項目: ペルソナの個性あるか / 1メッセージか / 中学生可読か / AI臭なしか / Before/After設計か |

**FAIL時**: retry < 2 → Step 3に戻り再生成 / retry >= 2 → content/rejected/ に保存 → Discord通知 → 終了
**PASS時**: content/approved/YYYY-MM-DD-{slug}.md に保存。drafts/の中間生成物は削除。

### Step 4.5: 画像生成（noon slotで1日1回）

1. `state.json` の `today_image_count >= 1` → スキップ
2. `--slot noon` → 実行 / 他slot → スキップ / `--dry-run` → 生成のみ実行
3. シーン選定 → Gemini API画像生成 → 品質チェック（6項目、最大3リトライ）
4. **画像なしでもパイプラインは止まらない**

> Consult `references/image-generation-pipeline.md` for: シーン選定ロジック、Gemini API設定、参照画像ルール、品質チェック6項目、bashコマンド

### Step 5: X投稿（`--dry-run` の場合はスキップ）

```bash
# テキストのみ
node $HOME/.ai-persona/scripts/x-post.mjs post "${text}"
# スレッド
node $HOME/.ai-persona/scripts/x-post.mjs thread "${tweet1}" "${tweet2}" ... "${tweetN}"
```

画像付き投稿は `--media content/images/YYYY-MM-DD.png` を追加。

**投稿前チェック**: TIME_GATE(7-23 JST) + circuit_breaker + 日次上限
**失敗時**: 最大3回リトライ(30秒間隔) → 3回失敗でDiscord通知 → approved/に残す
**成功時**: approved/ → posted/ に移動、メタデータ追記(posted_at, tweet_id, tweet_url)

### Step 6: Discord通知

通知先: **<YOUR_DISCORD_WEBHOOK>**（チームch）。`discord-notify.mjs team` で形式/カテゴリ/文字数/URL（画像付きならシーン/服装も）を通知。

### Step 7: 記録更新

1. `activity/YYYY/MM/YYYYMMDD.md` にログ追記
2. `state.json` 更新: `today_post_count += 1`, `last_post_at = now`, `consecutive_failures` リセット、画像付きなら `today_image_count += 1` + `last_image_scene/outfit` 更新、`slots_completed.{slot} = now`、失敗3連続→circuit_breaker "open"

### Step 8: 完了サマリー出力

```
[daily-post] 完了
テーマ: {テーマタイトル}
形式: {長文 or スレッド(N連投)}
品質ゲート: {PASS / FAIL(N回目でPASS) / ESCALATED}
投稿: {posted: URL / skipped: 理由 / failed: 理由}
```
