# Basic Hugo Theme

シンプルでモノクロなHugoテーマです。

## 特徴

- 🎨 **モノクロデザイン**: 白・黒・グレーのシンプルな配色
- 📱 **レスポンシブ**: モバイル・タブレット・デスクトップ対応
- ⚡ **高速**: 最低限のCSS/JSで軽量
- 🎯 **読みやすさ重視**: タイポグラフィと余白にこだわったデザイン
- 🌍 **多言語対応**: 日本語・英語対応

## 使用方法

1. テーマをダウンロード
2. `themes/Basic/` に配置
3. `hugo.toml` で設定

```toml
theme = 'Basic'
```

## サポート機能

- 記事一覧・詳細ページ
- カテゴリー・サブカテゴリー・タグ
- ページネーション
- 記事ナビゲーション
- レスポンシブデザイン

## 設定例

```toml
baseURL = 'https://example.com/'
languageCode = 'ja-jp'
title = 'My Blog'
theme = 'Basic'

[params]
  description = 'シンプルなブログです'

[menu]
  [[menu.main]]
    name = "ホーム"
    url = "/"
    weight = 1
  [[menu.main]]
    name = "ブログ"
    url = "/posts/"
    weight = 2
```

## ライセンス

MIT License 