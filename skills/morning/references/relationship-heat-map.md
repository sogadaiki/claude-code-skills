# Step 2.7: 関係温度マップ + 地雷リマインダー（司令塔モード限定）

`memory/*-architecture.md` のセクション0から **人間関係の文脈** を抽出し、朝一で表示する。

## 手順

1. `_registry.yaml` の `status: active` または `status: proposal` のプロジェクトを対象にする
2. 各プロジェクトの **architecture.md のセクション0** を読む（セクション0のみ。残りは読まない）
   - ファイルパス: `~/.claude/projects/<project-memory-path>/memory/{slug}-architecture.md`
   - slug一覧: 自分のアクティブクライアントのslugを列挙
   - `system-architecture.md` は対象外（社内システムなので人間文脈なし）
3. 各プロジェクトの `clients/{path}/` への **最終コミット日** を取得:

```bash
# slug:path のマッピングを自分の環境に合わせて変更
for slug_path in client-a:client-a client-b:client-b client-c:client-c; do
  slug="${slug_path%%:*}"
  path="${slug_path##*:}"
  last=$(git -C <ops-repo-path> log -1 --format="%ad" --date=short -- "clients/$path/" 2>/dev/null || echo "なし")
  echo "$slug|$last"
done
```

4. **温度判定**（最終コミット日と今日の差分）:
   - GREEN 7日以内
   - YELLOW 8-14日
   - RED 15日超 or コミットなし

5. 以下のフォーマットで表示:

```
## 関係温度マップ

| クライアント | キーパーソン | 最終接触 | 温度 | 推奨アクション |
|-------------|------------|---------|------|--------------|
| client-a | 田中さん | 3/8 | GREEN | -- |
| client-b | 山田さん | 3/1 | YELLOW | 進捗共有を検討 |
| client-c | 佐藤さん | 2/20 | RED | フォロー連絡を |
```

- **キーパーソン**: セクション0の人物テーブルから代表者（意思決定者 or 最初の人物）
- **推奨アクション**: REDの場合、セクション0の `next_actions` や文脈から具体的なアクションを提案。GREEN/YELLOWは「--」でOK
- **path: null のプロジェクト**（内部ツール等）はスキップ（対人クライアントではないため）

## 地雷リマインダー

セクション0の「地雷」が**空でない**かつ**「特記なし」でない**クライアントを抽出し、注意喚起する。

```
## 地雷リマインダー
- **client-b**: （セクション0の地雷内容をそのまま表示）
```

地雷が全て「特記なし」の場合、このセクション自体を非表示にする。
