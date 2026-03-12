---
name: quick-fix
description: GitHub Issueを素早く修正。Issue取得→修正→型チェック→コミット→PR。
argument-hint: [Issue番号]
---

# Quick Fix: $ARGUMENTS

## 手順

1. **Issue取得**: `gh issue view $ARGUMENTS` で詳細確認
2. **関連ファイル特定**: Issue内容からサブエージェントで関連コードを探索
3. **修正実装**: 最小限の変更で修正
4. **型チェック**: `npm run type-check` で問題なしを確認
5. **コミット作成**: Issue番号を参照したコミットメッセージ
6. **PR作成またはpush**: ブランチ戦略に応じて対応

## ルール

- 修正は最小限に。関係ないリファクタリングをしない
- 型チェックが通らなければコミットしない
- Issueの要件を満たしているか最終確認

### 使い方

```
/quick-fix 42
/quick-fix 123
```
