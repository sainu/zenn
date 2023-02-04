# zenn

zennの記事を管理するリポジトリ

# 使い方

## 記事の作成

```bash
npx zenn new:article --slug 記事のスラッグ --title タイトル --type idea --emoji ✨
```

* slug - 記事のURLになります。slug は`a-z0-9`、ハイフン`-`、アンダースコア`_`の 12〜50 字の組み合わせにする必要があります。
* title - 記事のタイトルです。
* type - 記事の種類です。tech か idea のどちらかを指定します。
* emoji - 記事のアイキャッチになる絵文字です。絵文字は GitHub の絵文字リストから選択します。

## 記事のプレビュー

```bash
npx zenn preview
```
