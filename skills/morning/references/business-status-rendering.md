# Step 2.5: 事業状況サマリー（司令塔モード限定）

`_registry.yaml` と各 `project.yaml` から **activeな事業の現況** を一覧表示する。

## 手順

1. `clients/_registry.yaml` を読み、`projects:` のうち `status: active` または `status: proposal` のものを抽出
2. 該当する各 `project.yaml` を読み、`status` + `next_actions`（先頭1件）を取得
3. 以下のフォーマットで表示:

```
## 事業状況（_registry.yaml + project.yaml）

### CS（Client Services）
| slug | service | status | 次のアクション |
|------|---------|--------|---------------|
| client-a | marketing-bpo | active | （project.yamlのnext_actions[0]） |
| client-b | ai-training | active | ... |
...

### PD（Products）
| slug | service | status | 次のアクション |
...

### IO（Internal Ops）
| slug | service | status | 次のアクション |
...
```

## 注意
- `path: null` のプロジェクト（内部ツール等）は `project.yaml` が存在しないので status列のみ表示
- `project.yaml` が未作成（マッピング未実施）のプロジェクトは「未マッピング」と表示
- archiveは表示しない
- マッピング成果物（`*-architecture.md`等）はここでは読まない。作業選択後のナレッジ活性化で読む
