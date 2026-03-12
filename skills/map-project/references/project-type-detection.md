# プロジェクト種別推定（Phase 1.2 詳細）

以下のファイルの存在で自動判定（Glob/Bash で確認）:

## ステップ1: コードファイルの存在確認

```
.ts / .tsx / .js / .jsx / .py / .rs / .go / .rb / .java / .php / .cs
のいずれかが1ファイルでも存在する → ソフトウェアプロジェクトとして以下で判定
```

コードファイルが存在する場合の検出テーブル:

| 検出ファイル | 種別タグ |
|------------|---------|
| `package.json` | node |
| `wrangler.toml` | cloudflare-workers |
| `next.config.*` | nextjs |
| `Cargo.toml` | rust |
| `pyproject.toml` / `requirements.txt` | python |
| `docker-compose.yml` | docker |
| Supabase URL in config | supabase |
| `gas/` or `*.gas.js` | gas |
| WordPress `style.css` with `Theme Name` | wordpress |
| `.github/workflows/` | github-actions |
| `Gemfile` | rails |
| `go.mod` | go |
| `pubspec.yaml` | flutter/dart |

複数該当OK（例: node + cloudflare-workers + supabase + gas）。

## ステップ2: コードファイルが存在しない場合（非コード系プロジェクト）

```
PDF / DOCX / XLSX ファイルの数を確認する:
  → ディレクトリ名に法務用語（訴状, 答弁書, 証拠, 準備書面, 裁判, 弁護士, 懲戒, 申立等）が含まれる
    → 種別タグ: legal
  → 法務用語なし、ビジネス用語（事業計画, 収支, 売上, KPI, 投資, クラウドファンディング, ピッチ等）が含まれる
    → 種別タグ: business-plan
  → 上記いずれにも該当しない文書主体
    → 種別タグ: documents
  → Markdown / テキストファイルが主体（PDF/DOCXが少ない）
    → 種別タグ: knowledge-base
```

非コード系は `legal` / `documents` / `knowledge-base` を最後のフォールバックとして適用する。
コードファイルが1ファイルでも存在すれば、この判定は行わない。
