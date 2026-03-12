---
name: write-sales-copy
description: Story-driven sales page / Brain article writing. Creates persuasive content using transformation narratives, not just feature lists. References sales-writing knowledge base.
argument-hint: [product-name or topic] [--style paradigm-a | paradigm-b | hybrid]
---

# Write Sales Copy: $ARGUMENTS

## Overview
Write a story-driven persuasive sales page or Brain article. This skill differs from `/write-blog` (SEO informational content) by focusing on **transformation narratives** and **emotional purchase triggers**.

## Multi-Pass Design

既存ワークフロー（Step 1-7）を多段階原則にマッピング。

| Pass | 内容 | 対応Step |
|------|------|----------|
| **Pass 1: 素材収集** | プラットフォームリサーチ（売り場を見る）、スタイル決定、3並列Deep Research（Product/Competitor/Story） | Step 1-3 |
| **Pass 2: 構成設計 + 自己批判** | アウトライン構築（FREE/PAID構成、有料ライン配置）。「Anti-thesisが明確か？」「ストーリー/実用の比率は適切か？」「押し売り感がないか？」を自己批判 | Step 4 |
| **Pass 3: 生成** | 執筆（Voice & Tone / Story / Practical / Closing ルール適用） | Step 5 |
| **Pass 4: 品質ゲート** | 3グループ並列レビュー（A:Story Quality / B:Practical Section / C:Conversion Design） | Step 6 |
| **Pass 5: 修正** | FAIL項目のみ最小限修正→再チェック。2回FAILで該当セクション再設計 | Step 6後半 |

### Pass 4 品質ゲート項目（セールスコピー固有）

| Group | 主要チェック |
|-------|------------|
| A: Story Quality | オープニング突き刺さり/テンション持続/権威の間接構築/Voice&Tone/AI臭い表現 |
| B: Practical | deliverables具体性/Anti-thesis明確性/ストーリー比率/有料ライン配置/モバイル読みやすさ |
| C: Conversion | 価格アンカリング/変革フレーミング/CTA明確性/レビューインセンティブ |

## IMPORTANT: Read knowledge FIRST
Before writing anything, read:
- `~/.claude/knowledge/sales-writing.md` (sales writing principles)
- `~/.claude/knowledge/prompt-techniques-sales-copy.md` — セールスコピー技法
- Relevant product knowledge file if it exists

## Step 1: Platform Research (MUST NOT SKIP)

**いきなり書き始めない。まず売り場を見に行く。**

販売プラットフォーム（Brain, note, Udemy等）のトップページ・カテゴリページを実際に訪問し、「今何が売れているか」を把握する。これを飛ばすと市場感覚ゼロの記事になる。

### 手順

1. **トップページのスクショを撮る**
   - Playwright MCP の `browser_navigate` で販売プラットフォームにアクセス
   - `browser_take_screenshot` でフルページスクショ（fullPage: true）
   - トレンド・ランキング・ピックアップ記事を目視で分析

2. **同カテゴリの売れ筋を調査**
   - カテゴリページに移動
   - 上位10記事の以下を収集:
     - タイトルのパターン（数字訴求? 疑問形? ストーリー型?）
     - 価格帯（最安〜最高、中央値）
     - レビュー数（売れている指標）
     - サムネイルの傾向（色、フォント、写真 or テキストのみ）

3. **市場ギャップを特定**
   - 「売れている記事がカバーしていない領域」を見つける
   - 「全員がやっていること」を避ける（差別化の源泉）
   - 価格のスイートスポットを特定する

4. **リサーチ結果をユーザーに報告してから次に進む**

## Step 2: Determine Style

Parse $ARGUMENTS for `--style` flag:
- `paradigm-a`: Template/practical style (low-price, feature-focused)
- `paradigm-b`: Full story/transformation style (high-price, narrative-focused)
- `hybrid` (DEFAULT): Story opening + practical body + vision close

If no style specified, default to `hybrid`.

## Step 3: Deep Research Phase (use subagents in parallel)

### Subagent 1: Product Deep-Dive
- Read all relevant product knowledge files
- Identify: core value proposition, target user pain points, unique differentiators
- List concrete deliverables (what the buyer actually gets)

### Subagent 2: Competitor Deep-Dive (Step 1のリサーチ結果を活用)
- Step 1で見つけた売れ筋記事の中から、直接競合を2-3本ピックアップ
- 各記事の無料セクションを読み、構成・トーン・訴求ポイントを分析
- 価格・レビュー数・アフィリエイト率から市場ポジションを把握
- **Anti-thesis（逆張り）ポイントを必ず1つ以上見つける**

### Subagent 3: Story Material Gathering
- Interview the user (or use knowledge base) for:
  - The origin story: Why did you build this?
  - The breaking point: What was broken before this existed?
  - The transformation: What changed after building/using this?
  - The mission: Why share this publicly?

## Step 4: Outline Construction

### For Hybrid Style (default):
```
[FREE SECTION - approximately 40% of total]

1. STORY HOOK (15-20% of free section)
   - Opening line: Must stop the scroll
   - The problem/frustration (relatable pain)
   - The turning point (discovery/insight)
   - The result (concrete, multi-dimensional change)

2. PRACTICAL SHOWCASE (60% of free section)
   - Content overview with specific deliverables
   - Structure/table of contents preview
   - One sample of actual content
   - Concrete numbers: X agents, Y skills, Z files, etc.

3. SOCIAL PROOF + INVITATION (20% of free section)
   - Testimonials or usage results
   - Price anchoring (compare to alternatives)
   - Transformation framing: "not buying X, becoming Y"
   - Clear CTA

--- PAID LINE ---

4-N. FULL CONTENT

N+1. BONUS / REVIEW INCENTIVE
```

## Step 5: Writing Execution

### Voice & Tone Rules
- First person, conversational
- Short paragraphs (1-3 sentences each, mobile-first reading)
- Generous whitespace and line breaks
- Mix of casual and serious tone
- NO academic/formal language
- NO bullet-point walls in the story section
- Japanese: Use plain form (だ/である) mixed with polite (です/ます) for rhythm

### Story Section Rules
- Open with the LOWEST point, not achievements
- Show, don't tell: Specific scenes > abstract claims
- Include sensory details (what you saw, felt, thought)
- Build authority INDIRECTLY (results speak, third-party validation)
- No "I am an expert" statements

### Practical Section Rules
- Concrete deliverables with specific counts
- Visual structure (tables, numbered lists)
- One free sample of real content
- Anti-thesis: What makes this different from alternatives

### Closing Rules
- Frame purchase as transformation, not transaction
- Price anchor against a more expensive alternative
- End with invitation, not hard sell
- Include review template or review incentive

## Step 6: セルフレビュー（3グループ並列）

完成した原稿を3つの観点グループに分け、**並列サブエージェント**でチェックする。

#### Review Group A: Story Quality

| # | チェック項目 | 基準 |
|---|-------------|------|
| A1 | オープニング突き刺さり度 | スクロールを止める力があるか |
| A2 | ストーリーテンション | 緊張感が持続しているか |
| A3 | 権威の間接構築 | 自称がないか。結果で伝わるか |
| A4 | Voice & Tone適合 | 一人称・会話調・短パラグラフか |
| A5 | AI臭い表現の排除 | 「画期的」「革新的」等のAI定型表現がないか |

#### Review Group B: Practical Section

| # | チェック項目 | 基準 |
|---|-------------|------|
| B1 | 具体的deliverables | 数量が具体的にリスト化されているか |
| B2 | Anti-thesis明確性 | 競合との差別化が1つ以上明示されているか |
| B3 | ストーリー/実用バランス | 比率が適切か |
| B4 | 有料ライン配置 | 「もっと読みたい」が最大化される位置か |
| B5 | モバイル読みやすさ | 短パラグラフ・十分な余白があるか |

#### Review Group C: Conversion Design

| # | チェック項目 | 基準 |
|---|-------------|------|
| C1 | 価格アンカリング | 高額代替手段との比較があるか |
| C2 | 変革フレーミング | 「商品を買う」ではなく「自分が変わる」フレーミングがあるか |
| C3 | CTA明確性 | 自然で明確、招待のトーンになっているか |
| C4 | レビューインセンティブ | 特典が用意されているか |

## Step 7: Output

Deliver the complete article in markdown format with a clear `<!-- PAID LINE -->` marker.

Also provide:
1. Suggested platform settings (price, affiliate %)
2. Review incentive proposal (what bonus to offer reviewers)
3. 3 X/Twitter promotional post drafts
4. Thumbnail text suggestions (2-3 options)

## Rules
- ALWAYS read the sales-writing knowledge base before writing
- ALWAYS ask the user for story material if not available in knowledge base
- NEVER write generic "AI-generated" sounding copy
- NEVER use walls of bullet points in the story section
- NEVER self-proclaim expertise directly ("I am the best at X")
- Story must feel HONEST, not manufactured
- Practical section must deliver REAL value preview, not teasers
- The free section must be good enough to share even without buying
