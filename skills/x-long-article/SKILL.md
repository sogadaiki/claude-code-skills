<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
---
name: x-long-article
description: "Automation-only: X long-form article (Note) auto-generation pipeline. Run via orchestrator with `claude -p`. Do NOT trigger for interactive sessions."
argument-hint: "[--dry-run]"
---

# X Long-Form Article Pipeline

オーナー(@<YOUR_X_ACCOUNT>)のX長文投稿（Note）を自動生成する。
開発ログ3日分を読み、抽象化して記事を生成し、HTML保存・Cloudflare Pagesデプロイ・ダッシュボード記録まで行う。
**X投稿は自動化しない**（トークンなし）。生成した記事はダッシュボードに記録し、ユーザーが手動で抽出・投稿する。

## Multi-Pass Design

既存ワークフロー（Step 0-9）を多段階原則にマッピング。

| Pass | 内容 | 対応Step |
|------|------|----------|
| **Pass 1: 素材収集** | 環境確認、開発ログ3日分読込、テーマ選定 | Step 0-1 |
| **Pass 2: 初稿 + 自己批判** | 初稿生成（Pass 1）→ 簡易セルフレビュー（Pass 2）。「1メッセージに絞れてるか？」「vulnerabilityが出てるか？」を自己批判 | Step 2 |
| **Pass 3: 品質ゲート** | 4グループ並列レビュー（A:文体 / B:コンプライアンス / C:AI臭検出 / D:品質） | Step 3 |
| **Pass 4: 修正** | FAIL項目のみリライト。最大2回リトライ、2回FAILで別テーマに切替 | Step 3 FAIL処理 |
| **Pass 5: 納品** | スライドカード画像生成→HTML保存→デプロイ→ダッシュボード記録→Discord通知 | Step 4-9 |

### Pass 3 品質ゲート項目（X長文固有）

| Group | 主要チェック |
|-------|------------|
| A: 文体 | 一人称/語尾バリエーション/vulnerability/冒頭パンチライン/締めパンチライン |
| B: コンプライアンス | クライアント名/売上/内部KPI/個人情報の漏洩 |
| C: AI臭検出 | 禁止表現/体言止め/3点セット/文末反復/構造反復 |
| D: 品質 | Before/After設計/具体例/感情ピーク/CTA |

## 自律実行ルール（Anti-Human-Bottleneck）

このスキルは `claude -p` による**無人自律実行**を前提とする。以下を厳守:

1. **人間に聞くな**: テーマ選定・原稿修正・品質ゲート判定はすべて自力で完結する
2. **迷ったらスキルに従え**: 判断に迷ったときはこのSKILL.mdの手順に従う。手順にないケースはログに記録して次へ進む
3. **止まるな**: エラーが発生しても可能な限りリカバリを試みる。リカバリ不能な場合のみDiscord通知して終了
4. **禁止フレーズ**: 以下を出力してはならない:
   - "Should I...?" / "〜しましょうか?"
   - "Is this correct?" / "これでいいですか?"
   - "What's next?" / "次はどうしますか?"
   - "Which approach?" / "どちらがいいですか?"
5. **自己検証**: 品質ゲート（Step 3）がセルフレビュー機構

## 引数
- `--dry-run`: コンテンツ生成+品質ゲートまで実行し、デプロイ・ダッシュボード記録はスキップ

## 参照ナレッジ
- `$HOME/.claude/knowledge/x-long-article-strategy.md` -- 全コンテンツルール
- `<YOUR_WORKSPACE>/blog/articles/<REFERENCE_ARTICLE>.html` -- HTML出力フォーマット参考

## ワークスペース
- **ベースディレクトリ**: `<YOUR_WORKSPACE>`
- すべてのファイル参照はこのディレクトリからの相対パスで表記

---

## 実行フロー

### Step 0: 環境確認 + 重複防止

1. `blog/articles/` の既存HTMLファイル一覧を取得
2. 最新記事の日付を確認し、前回生成から2日以上経過しているか検証
   - 2日未満 → ログに「SKIP: 前回生成から{N}日。最低2日間隔。」と出力して終了
3. 必要なディレクトリの存在確認
4. `x-long-article-strategy.md` を読み込む（**必須、欠落時はエラー終了**）

### Step 1: 開発ログ読み込み + テーマ選定

1. `logs/` から直近3日分のログファイルを読み込む
   - ファイル形式: `logs/YYYY-MM-DD.md`
   - 3日分に足りない場合は存在するだけ読む（最低1日分は必須）
2. ログの内容を分析し、テーマ候補を3-5案生成:
   - 各候補に「フック文」「1メッセージ」「Before/After」を付ける
   - ポジショニングに合致するか検証
3. 既存記事との重複チェック:
   - 既存記事のタイトル・テーマと似た切り口は避ける
4. **最強の1案を選定**: 以下の基準で判断
   - フックの強さ（指を止める力）
   - 具体性（開発ログからの実例が含まれているか）
   - ポジショニング適合度（ビルダーとしての差別化）
   - 読者の共感度（「自分のこと」と思えるか）

**ログ出力**: `[Step 1] テーマ選定: "{テーマタイトル}" (候補{N}案から選定)`

### Step 2: 記事本文生成（3パス品質プロセス）

**x-long-article-strategy.md の文体ルール・構造テンプレートに厳密に従う。**

#### Pass 1: 初稿生成

構造テンプレート:
```
1. フック（冒頭1行で指を止める）
2. 自分の立場の正直な開示
3. 実際にやっていること（開発ログから抽象化した具体例）
4. 1つの知見・主張（ワンメッセージ）
5. CTA（控えめに。「DMください」程度）
```

文体ルール:
- 一人称: 設定に従う
- 句点（。）は不揃いに使う。打つ文と打たない文が混在する状態が自然
- 改行で文を区切る。句点の代わりに改行
- 体言止め多用
- 感情を出す: 「!!」を躊躇なく使う
- 問いかけ: 口語で
- 正直さ前面: 自分に不利な事実も書く
- 見出し: **太字**で区切る（X NoteではMarkdown不可）
- テーブル（表）禁止
- 「整った要約」にしない: 印象に残った場面だけ厚く書き、他は雑に流す
- 同じ文末表現が3回続いたら書き直す

文字数: **1500-2500字**

#### Pass 2: 簡易セルフレビュー

以下5つの批判を自分で行い、明らかな問題があれば即修正する:
1. AI臭がないか
2. ポジショニングがブレていないか（コンサルではなくビルダー）
3. 具体性は十分か
4. 読者は「自分のこと」と思えるか
5. フックは本当に指を止めるか

**ログ出力**: `[Step 2] 記事生成完了: "{タイトル}" ({文字数}字, 3パス完了)`

### Step 3: 品質ゲート（4グループ並列、最大2回リトライ）

6チェックを4グループに分け、**並列サブエージェント**で同時実行する。

#### Review Group A: 文体

| # | チェック項目 | 基準 |
|---|-------------|------|
| A1 | 文体統一 | 一人称統一、句点不揃い、体言止めあり、同語尾3連続なし |
| A2 | 禁止パターン | AI定型表現が含まれていないこと |

#### Review Group B: コンプライアンス

| # | チェック項目 | 基準 |
|---|-------------|------|
| B1 | 情報漏洩チェック | 禁止キーワード・クライアント名・金額・地域名が含まれていないこと |
| B2 | 文字数 | 1500-2500字の範囲内 |

#### Review Group C: AI臭検出

| # | チェック項目 | 基準 |
|---|-------------|------|
| C1 | テーブル禁止 | テーブルが含まれていないこと |
| C2 | 箇条書き制限 | 4項目以上連続する箇条書きがないこと |
| C3 | 太字制限 | 1セクション内で太字が3箇所以上使われていないこと |
| C4 | 構文パターン | 「3つのポイント」型の構文がないこと |
| C5 | 文章の偏り | 均等に整っていないこと。印象に残った場面が厚く書かれていること |

#### Review Group D: 品質

| # | チェック項目 | 基準 |
|---|-------------|------|
| D1 | メッセージ明確性 | 1記事1メッセージに絞れているか |
| D2 | Before/After | 読者の認識変化が設計されているか |
| D3 | 具体例 | 開発ログからの具体例が1つ以上含まれているか |

#### 結果集約
4グループの結果をマージし、`review.json` として記録する。

#### FAIL時の処理
- FAILしたグループの修正指示**のみ**を反映して記事を修正する
- PASSしたグループの該当箇所には触らない（最小限修正の原則）
- retry_count < 2 → 修正して再チェック
- retry_count >= 2（3回失敗）→ Discord通知(escalation) → 終了

**ログ出力**: `[Step 3] 品質ゲート: {PASS / FAIL(Group {X}: {理由})}`

### Step 4: Playwrightスライドカード画像2枚生成

**記事に添付するスライドカード画像を2枚生成する。1200x675px (16:9)。**

1. 記事のタイトルとワンメッセージからスライドカード用テキストを抽出:
   - **画像1**: タイトル or フック文をカード化
   - **画像2**: ワンメッセージ or 結論をカード化
2. 各カードのHTMLを生成:
   ```html
   <div style="width:1200px;height:675px;background:#1a1a2e;color:#fff;display:flex;align-items:center;justify-content:center;padding:60px;font-family:'Hiragino Kaku Gothic ProN',sans-serif;">
     <div style="text-align:center;">
       <p style="font-size:48px;font-weight:900;line-height:1.5;margin-bottom:24px;">{テキスト}</p>
       <p style="font-size:20px;color:#94a3b8;">@<YOUR_X_ACCOUNT></p>
     </div>
   </div>
   ```
3. PlaywrightでHTMLをスクリーンショット:
   ```bash
   npx playwright screenshot --viewport-size="1200,675" {html_file} {output_png}
   ```
4. 保存先:
   - `clients/public/x-articles/{slug}-1.jpg`
   - `clients/public/x-articles/{slug}-2.jpg`

**画像生成失敗時**: テキストのみで続行（パイプラインは止まらない）

### Step 5: HTML保存

1. 参考HTMLテンプレートのフォーマットに従ってHTMLを生成:
   - `<p class="meta">` に日付・著者・用途
   - `<h1>` にタイトル
   - 本文は `<p>` タグ、見出しは `<p class="bold-heading">`
   - 画像は `<img>` タグで公開URLを参照
   - CTA部分は `<div class="cta">`
   - フッターに `<div class="footer-meta">`
2. 保存先: `blog/articles/YYYY-MM-{slug}.html`
3. `clients/public/` にコピー

### Step 5.5: Google Docs保存

**生成した記事をGoogle Docsに保存し、ユーザーがスマホから即確認・編集できるようにする。**

1. `docs_create_document` MCPツールでドキュメントを作成:
   - タイトル: `[X長文] {記事タイトル} (YYYY-MM-DD)`
2. `docs_append_text` MCPツールで記事本文をプレーンテキストとして追記
3. 返却された `document_link` を変数に保存（Discord通知で使用）

**Google Docs保存失敗時**: ログに記録して続行（パイプラインは止まらない）

### Step 6: Cloudflare Pagesデプロイ（`--dry-run` の場合はスキップ）

```bash
cd <YOUR_WORKSPACE>
npx wrangler pages deploy clients/public --project-name=<YOUR_CF_PROJECT>
```

**デプロイ失敗時**: Discord通知して続行（記事は既にローカル保存済み）

### Step 7: ダッシュボード記録（`--dry-run` の場合はスキップ）

`dashboard/src/lib/clients.ts` のdeliverablesに新しいエントリを追加。

### Step 8: Discord通知

通知先: **<YOUR_DISCORD_WEBHOOK>**（チームch）

```
[X長文投稿] 記事生成完了
タイトル: {タイトル}
文字数: {文字数}字
確認: {Google Docs URL}
公開: {公開URL}
画像: {N}枚
```

### Step 9: ログ記録

1. `logs/YYYY-MM-DD.md` に追記
2. 完了サマリーを標準出力:
   ```
   [x-long-article] 完了
   テーマ: {テーマタイトル}
   文字数: {文字数}字
   品質ゲート: {PASS / FAIL(N回目でPASS) / ESCALATED}
   URL: {URL / dry-run}
   ```
