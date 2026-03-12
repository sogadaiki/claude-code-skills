---
name: benchmark-lp
description: ベンチマーク模倣LPの高速納品。参考サイト2-3個からオリジナルLPを生成。Use when user says "LP作成", "ランディングページ", "ベンチマークLP", "模倣LP", "LP納品"
argument-hint: [クライアント名 or プロジェクト名]
---

# Benchmark LP: $ARGUMENTS

## 参照ナレッジ
- `~/.claude/knowledge/design-standards.md` — **統合デザイン基準（最上位ルール）**
- `~/.claude/knowledge/document-design.md` — デザイン原則
- `~/.claude/knowledge/benchmark-business-strategy.md` — LP事業のポジショニング・価格設計
- `~/.claude/knowledge/benchmark-tech-catalog.md` — LP制作技術スタック（CSS/SVG/GSAP Tier分類）

ベンチマークサイト2-3個の構造・デザインを分析し、オリジナリティの高いLPを高速生成する。

---

## Multi-Pass Design

| Pass | 内容 | 対応Phase | 詳細 |
|------|------|-----------|------|
| **Pass 1: 素材収集** | インプット確認 + ベンチマーク分析 | Phase 0-1 | [benchmark-analysis-checklist.md](references/benchmark-analysis-checklist.md) |
| **Pass 2: 設計 + 自己批判** | yui-ux-designer相談 → セクション構成・デザインシステム確定。「NG集抵触?」「丸コピー?」「CTA配置十分?」を自己批判。ユーザー承認必須 | Phase 2 | [design-system-guide.md](references/design-system-guide.md) |
| **Pass 3: 生成** | ビジュアルアセット生成 + コード実装（並列サブエージェント活用） | Phase 3-4 | [visual-asset-generation.md](references/visual-asset-generation.md) / [implementation-patterns.md](references/implementation-patterns.md) |
| **Pass 4: 品質ゲート** | Playwrightスクリーンショット（375px + 1440px）+ チェックリスト全項目 | Phase 5 | [quality-checklist.md](references/quality-checklist.md) |
| **Pass 5: 修正** | FAIL項目のみ修正 → 再スクリーンショット。2回FAILで Phase 2 に戻す | Phase 5後 | [quality-checklist.md](references/quality-checklist.md) |

### Pass 4 品質ゲート項目（LP固有）

| # | チェック項目 | FAIL基準 |
|---|------------|---------|
| 1 | モバイル表示 | 375pxで崩れ・はみ出し |
| 2 | PC表示 | 1440pxで崩れ・余白異常 |
| 3 | アニメーション | 段階的表示が未動作 / 全要素同時出現 |
| 4 | 画像 | 未ロード / WebP未最適化 / alt属性なし |
| 5 | CTA | 目立たない / クリック不能 / 配置不足（最低2箇所） |
| 6 | パフォーマンス | LCP > 2.5s |
| 7 | AI臭さ | NG集（Inter/Nunito/Lucide大量/均等3カラム/白背景影カード）に抵触 |
| 8 | ベンチマーク整合 | 構成が模倣できていない / 丸コピーになっている |

---

## Phase 0: インプット確認（最重要フェーズ）

> **絶対ルール**: 全質問の回答が揃うまで1行もコードを書かない。
> 「聞くコスト < やり直しコスト」。いい問いを投げてデータを集めることが最良の設計。
> **勝手に実装を始めることは禁止。** ユーザーが「作って」と言っても、データが足りなければ質問で返す。

### Step 0-1: ビジネス理解（必ず最初に聞く）

`AskUserQuestion` で以下を**1問ずつ丁寧に**確認する。まとめて投げない。

| # | 質問 | なぜ聞くか |
|---|------|----------|
| 1 | **誰に向けたLPですか？**（ペルソナ: 年齢、職業、課題、現在の行動） | ターゲット不明のまま作ると全方位的な薄いLPになる |
| 2 | **このLPを見た人に、最終的に何をしてほしいですか？**（CTA: 問い合わせ/購入/LINE登録/資料DL） | CTAが曖昧だとセクション構成が定まらない |
| 3 | **競合と比べて、この商品/サービスの「これだけは負けない」ポイントは？** | USP不明だとどのベンチマークを模倣すべきか判断できない |
| 4 | **価格帯は？**（具体額 or 「高単価」「低単価」レベルでもOK） | 高単価は信頼構築重視、低単価は即決導線重視でLP構造が変わる |
| 5 | **既存のブランドガイドライン（ロゴ、色、フォント）はありますか？** | なければデザインシステムをゼロから設計する必要がある |

### Step 0-2: ビジュアル素材収集（回答を待ってから聞く）

| # | 質問 | なぜ聞くか |
|---|------|----------|
| 6 | **ベンチマークサイト（「こういう雰囲気にしたい」URL）を2-3個ください** | 曖昧指示（"かっこよく"）ではAI臭いデザインになる。リファレンスが命 |

> ユーザーがURLを持っていない場合:
> 1. プロジェクト種別（LP / SaaS / 一般Web）を判断
> 2. `benchmark-design-workflow.md` Section 7 のカタログから適切なサイトを選定
> 3. WebFetch で 2-3 サイトを取得し、雰囲気・構造を要約
> 4. AskUserQuestion で候補を提示し、ユーザーに選んでもらう
> 5. 選ばれたサイトをベンチマークとして Phase 1 に進む

| 7 | **スクリーンショット / DivMagicコード / PageGrab分析データはありますか？**（なければURLだけでOK） | ビジュアル入力が多いほど精度が上がる。PageGrabがあれば配色・フォント・画像を自動抽出できる |
| 8 | **使いたい写真・動画・素材はありますか？**（なければDALL-E生成 or フリー素材で対応） | 実写素材があると一気にオリジナリティが出る |

### Step 0-3: 技術要件（最後に確認）

| # | 項目 | デフォルト |
|---|------|----------|
| 9 | **技術スタック** | Next.js + Tailwind |
| 10 | **デプロイ先** | Vercel |
| 11 | **フォーム連携**（問い合わせ先のメールアドレス等） | 必要に応じて確認 |

### Phase 0 完了チェック

以下が全て埋まるまでPhase 1に進まない:

```
[ ] ペルソナ（誰に）
[ ] CTA（何をさせたいか）
[ ] USP（競合との差別化）
[ ] 価格帯
[ ] ブランドガイドライン有無
[ ] ベンチマークURL 2-3個
[ ] ビジュアル素材の有無
[ ] 技術スタック・デプロイ先
```

### DivMagic / PageGrab 入力フォーマット

DivMagic: セクション単位コピー（JSX+Tailwind）/ ページ全体一括 / スクリーンショットのみ。
受け取ったコード/データは `reference/` ディレクトリに保存してから分析する。

PageGrab: Chrome拡張「LP Analysis」でベンチマーク分析。JSONの構造・活用法は → [benchmark-analysis-checklist.md](references/benchmark-analysis-checklist.md)

```
{project}/reference/
├── benchmark-1-hero.tsx           # DivMagicコード
├── benchmark-1-analysis.json      # PageGrab LP Analysis JSON
├── benchmark-2-hero.tsx
├── benchmark-2-analysis.json
└── screenshots/
    ├── benchmark-1.png
    └── benchmark-2.png
```

---

## Phase 1: ベンチマーク分析

各ベンチマークサイトをセクション構成・デザインパターン・「強い要素」の3軸で分解する。
PageGrab JSONがあれば配色・フォント・画像を自動抽出して効率化。

**詳細手順** → [benchmark-analysis-checklist.md](references/benchmark-analysis-checklist.md)

---

## Phase 2: 設計（yui-ux-designerに相談）

### 2.1 UXデザイン相談

`yui-ux-designer` エージェントにクライアントブリーフ + ベンチマーク分析結果 + 「強い要素」リストを渡し、セクション構成案 + 配色 + ビジュアル方針の提案を受ける。

### 2.2 セクション構成確定

yuiの提案をベースに、セクション構成・デザインシステム（配色・フォント・余白・ボーダー半径）・ビジュアル戦略を確定する。PageGrab分析データがあればデザインシステムの初期値として使う。

**デザインシステム定義の詳細** → [design-system-guide.md](references/design-system-guide.md)

### 2.3 ユーザー承認

構成案をユーザーに提示し、承認を得る。承認なしで実装に進まない。

---

## Phase 3: ビジュアルアセット生成

Nano Banana 2 (Gemini 3.1 Flash Image) で画像生成 + 背景動画 + アニメーション定義。
デザインシステムを先行構築し、NG集を厳守してAI生成っぽさを排除する。

**詳細手順** → [visual-asset-generation.md](references/visual-asset-generation.md)
**デザインシステム・NG集** → [design-system-guide.md](references/design-system-guide.md)

---

## Phase 4: 実装

Next.js+Tailwind or 静的HTML。Hero → 各セクション順に実装。モバイルファースト。
セクション数が多い場合はサブエージェントで並列実装。

**詳細手順** → [implementation-patterns.md](references/implementation-patterns.md)

---

## Phase 5: 検証

Playwrightで375px + 1440pxスクリーンショット撮影 → チェックリスト8項目 → ユーザーレビュー。
FAIL項目のみ修正、2回FAILでPhase 2に戻す。

**詳細手順** → [quality-checklist.md](references/quality-checklist.md)

---

## 制約・ルール

1. **Phase 0のインプットが揃うまで実装に入らない**
2. **Phase 2のユーザー承認なしで実装に進まない**
3. **ビジュアルNG集を厳守する**（→ [design-system-guide.md](references/design-system-guide.md)）
4. **DivMagicコードはそのまま使わない** — 構造を参考にしてオリジナルで書き直す
5. **全角スペースは半角に統一**
6. **画像にalt属性を必ず付与**
7. **`mobile-first-design` スキルの原則を遵守**
