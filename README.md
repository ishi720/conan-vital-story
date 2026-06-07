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
| flags.black_org | boolean | 黒の組織関連回 |
| flags.new_character | boolean | 新キャラ登場回 |
