<!-- This is an automation-only skill template. Customize the persona, accounts, and endpoints for your setup. -->
# コンテンツファイルフォーマット

> SKILL.md Step 3 のコンテンツファイル形式詳細。approved/ に保存する際のメタ情報フォーマット。

## 標準フォーマット

```markdown
# X{形式}ポスト: {タイトル}

## メタ情報
- 形式: {短文ツイート / 長文ツイート / スレッド(Nツイート)}
- カテゴリ: {A/B/C/D} ({カテゴリ名})
- テンプレート: {テンプレート名}
- slot: {morning/noon/evening/weekend}
- 文字数: 約{N}字
- 生成日: YYYY-MM-DD
- 品質ゲート: PASS
- Before: {読前の認識}
- After: {読後の認識変化}

## 本文

{投稿テキスト}
```

## スレッドの場合

```markdown
## 本文

### ツイート1
{1ツイート目テキスト}

### ツイート2
{2ツイート目テキスト}

...
```
