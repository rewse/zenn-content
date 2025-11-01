---
inclusion: always
---

# 技術スタック

## 主要な依存関係
- **zenn-cli**: Zennプラットフォーム公式CLIツール (v0.2.9)
- **Node.js**: JavaScriptランタイム (CommonJSモジュール)
- **npm**: パッケージマネージャー

## ビルドシステム
- 標準的なnpmパッケージ構造
- CommonJSモジュールシステム (`"type": "commonjs"`)

## Zenn CLI コマンド

### 記事管理
```bash
# 新しい記事を作成
npx zenn new:article

# オプション付きで記事を作成
npx zenn new:article --slug 記事のスラッグ --title タイトル --type idea --emoji ✨

# 記事をローカルでプレビュー
npx zenn preview
```

### 本管理
```bash
# 新しい本を作成
npx zenn new:book

# スラッグ指定で本を作成
npx zenn new:book --slug ここにスラッグ

# 本をプレビュー（記事と同じコマンド）
npx zenn preview
```

### その他
```bash
# 依存関係のインストール
npm install
```

## ファイル配置ルール

### 記事（Articles）
- `articles/` ディレクトリ内に配置
- ファイル名: `◯◯.md` 形式
- スラッグ（slug）は記事のユニークID

### 本（Books）
```
books/
└── 本のスラッグ/
    ├── config.yaml      # 本の設定
    ├── cover.png        # カバー画像（JPEGまたはPNG）
    └── チャプター.md     # 各チャプター
```

## コンテンツ形式

### 記事のFrontmatter
```yaml
---
title: "記事のタイトル"
emoji: "😸"              # アイキャッチ絵文字（1文字）
type: "tech"             # "tech" または "idea"
topics: ["markdown", "rust", "aws"]  # タグ配列
published: true          # 公開設定（falseで下書き）
published_at: 2050-06-12 09:03  # 公開予約（オプション）
---
```

### 本の設定（config.yaml）
```yaml
title: "本のタイトル"
summary: "本の紹介文"
topics: ["markdown", "zenn", "react"]  # 最大5つ
published: true          # 公開設定
price: 0                 # 価格（200-5000円、100円単位）
toc_depth: 2            # 目次の深さ（0-3）
chapters:
  - chapter1             # チャプター順序
  - chapter2
  - chapter3
```

### チャプターのFrontmatter
```yaml
---
title: "チャプターのタイトル"
free: true              # 有料本での無料公開（オプション）
---
```

## 画像挿入方法
1. [Zenn画像アップロードページ](https://zenn.dev/dashboard/uploader)を使用
2. GitHubリポジトリ内に画像を配置
3. Gyazoなど外部サービスのURL使用

## 公開・更新・削除

### 公開
- `published: true` に設定
- GitHubリポジトリにプッシュ
- コミットメッセージに `[ci skip]` または `[skip ci]` でデプロイスキップ

### 更新
- ファイル編集後、GitHubにプッシュ
- 同一slugを維持すること

### 削除
- [ダッシュボード](https://zenn.dev/dashboard)から実行
- ファイル削除だけでは削除されない

## 制限事項
- 本のチャプター数: 最大100個
- 本の価格: 200-5000円（100円単位）
- トピック数: 最大5個
- カバー画像推奨サイズ: 500px × 700px
