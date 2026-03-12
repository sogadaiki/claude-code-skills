# Step 4.5: エージェントメモリ健全性（司令塔モード限定）

`~/.claude/agent-memory/` の各エージェントディレクトリを走査し、メモリの蓄積状況を表示する。

```bash
for agent_dir in ~/.claude/agent-memory/*/; do
  name=$(basename "$agent_dir")
  count=$(find "$agent_dir" -name "*.md" -not -name "MEMORY.md" | wc -l | tr -d ' ')
  has_memory=$([ -f "$agent_dir/MEMORY.md" ] && echo "YES" || echo "NO")
  last=$(stat -f "%Sm" -t "%m/%d" "$agent_dir" 2>/dev/null || echo "不明")
  echo "$name | $count件 | MEMORY: $has_memory | 最終: $last"
done
```

## 表示フォーマット

```
## エージェント育成状況
| エージェント | 分析レポート | MEMORY.md | 最終活動 | 状態 |
|-------------|-------------|-----------|---------|------|
| mei-business-strategist | 20件 | YES | 3/9 | GREEN 順調 |
| kaito-data-analyst | 0件 | NO | -- | RED 要育成 |
...
```

## 判定基準
- GREEN **順調**: レポート3件以上 + MEMORY.md存在
- YELLOW **発展途上**: レポート1-2件 or MEMORY.mdのみ
- RED **要育成**: レポート0件（使われていない）

## 要育成エージェントへの提案

REDのエージェントがいる場合、**今日の作業候補との紐付け**を提案する:
- 例: kaitoが要育成 + データ分析タスクがある → 「GA4分析をkaitoに任せて育成しませんか？」
- 例: rikuが要育成 + 広告運用クライアントがある → 「広告レポートをrikuに分析させましょう」
