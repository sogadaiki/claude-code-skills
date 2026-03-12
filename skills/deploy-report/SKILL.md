---
name: deploy-report
description: プロジェクトのレポート/ダッシュボードHTMLをCloudflare Pages（<YOUR_CF_PROJECT>）にデプロイ。Use when user says "デプロイ", "レポートを公開", "ダッシュボード更新", "ops deploy", or after generating report HTML files.
argument-hint: [project-slug] (例: client-f, client-a, client-c)
---

# Deploy Report: $ARGUMENTS

任意のプロジェクトからレポート/ダッシュボードHTMLをCloudflare Pagesにデプロイする。

## デプロイ先情報
- **Cloudflare Pages プロジェクト**: `<YOUR_CF_PROJECT>`
- **公開URL**: `https://<YOUR_PROJECT>.pages.dev/{slug}/`
- **デプロイステージング**: `<YOUR_PROJECT_ROOT>/clients/public/`
- **デプロイコマンド**: <YOUR_OPS_REPO> ディレクトリから `npx wrangler pages deploy clients/public --project-name <YOUR_CF_PROJECT> --commit-dirty=true`

## 手順

### 1. slugの特定

優先順位:
1. 引数 `$ARGUMENTS` で明示指定された場合 → そのまま使用
2. 引数なしの場合 → 現在の作業ディレクトリ名から推定

```
<YOUR_PROJECT> → slug: client-f
<YOUR_WORKSPACE>/<YOUR_OPS_REPO> → slug: （引数必須）
```

### 2. HTMLソースファイルの自動検出

現在の作業ディレクトリから以下のパターンで検索（上から優先）:

```bash
PROJECT_DIR="$(pwd)"

# パターン1: 08_分析レポート/ フォルダ（client-f等の訴訟案件）
ls "$PROJECT_DIR"/08_分析レポート/*.html 2>/dev/null

# パターン2: reports/ フォルダ
ls "$PROJECT_DIR"/reports/*.html 2>/dev/null

# パターン3: report*/ フォルダ（report/, report-2026/ 等）
ls "$PROJECT_DIR"/report*/*.html 2>/dev/null

# パターン4: docs/ フォルダ
ls "$PROJECT_DIR"/docs/*.html 2>/dev/null

# パターン5: dist/ や public/ フォルダ（ビルド済みプロジェクト）
ls "$PROJECT_DIR"/dist/*.html 2>/dev/null
ls "$PROJECT_DIR"/public/*.html 2>/dev/null

# パターン6: プロジェクトルート直下
ls "$PROJECT_DIR"/*.html 2>/dev/null
```

**検出結果が0件の場合**: ユーザーに「デプロイ対象のHTMLファイルが見つかりません。パスを指定してください」と聞く。

### 3. ステージングディレクトリにコピー

```bash
DEPLOY_BASE="<YOUR_PROJECT_ROOT>/clients/public"
SLUG="{slug}"

mkdir -p "$DEPLOY_BASE/$SLUG"

# dashboard.html / index.html → index.html（メインページ）
for f in dashboard.html index.html; do
  if [ -f "{source}/$f" ]; then
    cp "{source}/$f" "$DEPLOY_BASE/$SLUG/index.html"
    break
  fi
done

# その他のHTML（index.html以外）をコピー
for f in {source}/*.html; do
  [ "$(basename "$f")" = "dashboard.html" ] && continue
  [ "$(basename "$f")" = "index.html" ] && continue
  cp "$f" "$DEPLOY_BASE/$SLUG/" 2>/dev/null || true
done

# CSS/JS/画像など関連アセットもコピー（存在する場合）
for d in css js img assets images; do
  [ -d "{source}/$d" ] && cp -r "{source}/$d" "$DEPLOY_BASE/$SLUG/" 2>/dev/null || true
done
```

### 4. 機密情報チェック（CRITICAL）

コピー先のHTMLを簡易スキャン:
```bash
grep -ril '080-\|090-\|070-\|@.*\.co\.jp\|丁目.*番.*号' "$DEPLOY_BASE/$SLUG/" 2>/dev/null
```
ヒットした場合、ユーザーに警告して続行確認を取る。

### 5. デプロイ実行

```bash
cd <YOUR_WORKSPACE>/<YOUR_OPS_REPO>
npx wrangler pages deploy clients/public --project-name <YOUR_CF_PROJECT> --commit-dirty=true
```

### 6. 検証・URL報告

```bash
# デプロイ後30秒待ってからcurlで検証
curl -s -o /dev/null -w "%{http_code}" "https://<YOUR_PROJECT>.pages.dev/$SLUG/"
```

デプロイしたファイル一覧と公開URLをユーザーに報告:
```
https://<YOUR_PROJECT>.pages.dev/{slug}/              ← メイン
https://<YOUR_PROJECT>.pages.dev/{slug}/report-*.html ← 各レポート
```

## 注意

- デプロイは**全置換（アトミック）** — `clients/public/` 配下の全ファイルが対象
- 他プロジェクトの既存ファイルも含めて全体がデプロイされる（既存は維持される）
- 元のプロジェクトディレクトリのHTMLは残す（コピーであり移動ではない）
- wranglerが古い場合は `npx wrangler@latest` を使用
- 環境変数 `CLOUDFLARE_API_TOKEN` はKeychainから取得: `security find-generic-password -s "CLOUDFLARE_API_TOKEN" -a "claude-ops" -w`

## 使い方

```
# 明示的にslug指定（どのディレクトリからでもOK）
/deploy-report client-f
/deploy-report client-a
/deploy-report client-c

# slug省略（カレントディレクトリ名を使用）
cd ~/Desktop/development/client-f && /deploy-report
```
