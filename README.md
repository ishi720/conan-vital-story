# 名探偵コナン 進捗DB

## セットアップ

```bash
npm install
npm run dev
```

## データ編集

`data/episodes.json` を直接編集してエピソードを追加・修正できます。

### フィールド説明

| フィールド | 型 | 説明 |
|---|---|---|
| id | number | 一意のID |
| episode | number | 話数 |
| title | string | タイトル |
| volume | number | 単行本巻数 |
| progress | boolean | 進捗あり／なし |
| categories | string[] | 進捗ありの場合のカテゴリ配列（`"black_org"`, `"new_character"` など）。空の場合は「その他」 |
