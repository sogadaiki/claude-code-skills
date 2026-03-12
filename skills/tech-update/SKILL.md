---
name: tech-update
description: 技術スタックのベストプラクティス更新。Context7で最新ドキュメントを確認し、開発ナレッジに反映
argument-hint: [技術名: supabase|cloudflare|nextjs|stripe|all]
---

# Tech Update: $ARGUMENTS

技術スタックの最新ベストプラクティスを取得し、開発ナレッジを更新する。

## 参照ナレッジ
- `~/.claude/knowledge/architecture.md` — 現在のアーキテクチャ
- `~/.claude/knowledge/development.md` — 開発規約
- `~/.claude/CLAUDE.md` — グローバルルール（技術スタック固有ルール）

## 対象技術スタック

| 引数 | 技術 | 確認対象 |
|------|------|---------|
| supabase | Supabase | RLS, Edge Functions, Auth, Realtime, CLI |
| cloudflare | Cloudflare Workers/Pages | OpenNext, wrangler, bindings |
| nextjs | Next.js | App Router, Server Actions, Middleware |
| stripe | Stripe | API変更, Webhook, ゼロ小数点通貨 |
| all | 上記全て | 順番に巡回 |

## フロー

### Step 1: 現在のナレッジ確認

対象技術に関連するナレッジファイルとCLAUDE.mdの該当セクションを読み込む。

### Step 2: 最新ドキュメント取得

以下の方法で最新情報を取得:

1. **WebSearch** で「{技術名} breaking changes 2026」「{技術名} migration guide latest」を検索
2. **WebFetch** で公式ドキュメントの変更履歴ページを確認
3. 重要な変更点をリストアップ

### Step 3: 差分分析

現在のナレッジ/CLAUDE.md と最新情報を比較:

```
| 項目 | 現在の記載 | 最新の推奨 | 要更新？ |
|------|-----------|-----------|---------|
| ... | ... | ... | Yes/No |
```

### Step 4: ユーザー承認

更新が必要な箇所を提示し、AskUserQuestionで承認を得る。

### Step 5: 反映

承認された項目のみ、該当ファイルを更新する:
- CLAUDE.md の技術スタック固有ルール
- knowledge/ 配下の関連ファイル

### Step 6: 更新ログ

更新内容を以下に記録:
- プロジェクトメモリ（MEMORY.md）に「{日付} tech-update {技術名}: {変更概要}」

## 推奨実行頻度

月1回、`/morning` の月初チェックで提案される。
