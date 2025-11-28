# Zenn Content Repository

Zennプラットフォームで技術記事や本を公開するためのGitHub連携リポジトリです。ローカルでの執筆から自動公開まで、シームレスなワークフローを提供します。

## 概要

このリポジトリはZennプラットフォームと連携し、技術記事や本を管理・公開します。ローカル環境でMarkdown形式のコンテンツを執筆し、GitHubにプッシュすると自動的にZennへ公開されます。

### 主な機能

- GitHub連携によるプッシュ時の自動公開
- お気に入りのエディタでのローカル執筆
- Zenn CLI によるローカルプレビュー
- リポジトリ内での一元的な画像管理
- Gitによる変更履歴の追跡

## 技術スタック

- zenn-cli: Zennプラットフォーム公式CLIツール
- Node.js: CommonJSモジュール
- npmパッケージマネージャー

## セットアップ

### 前提条件

- Node.jsがインストールされていること
- GitHubアカウントがZennアカウントと連携されていること

### インストール

```bash
# 依存関係をインストール
npm install
```

## 使い方

### 新しい記事を作成

```bash
# 記事を作成
npx zenn new:article

# オプション付きで作成
npx zenn new:article --slug article-slug --title "タイトル" --type tech --emoji ✨
```

### 新しい本を作成

```bash
# 本を作成
npx zenn new:book

# スラッグ指定で作成
npx zenn new:book --slug book-slug
```

### ローカルプレビュー

```bash
# プレビューサーバーを起動
npx zenn preview
```

ブラウザで http://localhost:8000 にアクセスしてコンテンツをプレビューできます。

### 記事を公開

1. 記事の Front Matter で `published: true` を設定
2. GitHubリポジトリにコミット＆プッシュ
3. コンテンツが自動的にZennへ公開される

```bash
git add .
git commit -m "feat(article): add new article about docker"
git push origin main
```

## プロジェクト構造

```
.
├── .kiro/             # Kiro AI 設定
├── articles/          # 技術記事
├── books/             # 本のコンテンツ
├── images/            # 記事・本で使用する画像
│   └── article-slug/  # 記事ごとに整理
└── package.json       # プロジェクト設定
```

## 記事の Front Matter

```yaml
---
title: "記事のタイトル"
emoji: "😸"              # アイコン絵文字（1文字）
type: "tech"             # "tech" または "idea"
topics: ["docker", "aws", "devops"]  # タグ（最大5個）
published: true          # 公開状態
published_at: 2050-06-12 09:03  # オプション: 公開予約日時
---
```

### 記事タイプ

- tech: プログラミング、インフラ、ハードウェアなどの技術に関する具体的な内容
- idea: キャリア、マネジメント、抽象的な考え方、情報のまとめ記事

## 画像管理

### 画像の配置

```
images/
└── article-slug/
    ├── screenshot-1.png
    └── diagram.jpg
```

### 画像の参照

```markdown
![代替テキスト](/images/article-slug/screenshot-1.png)
```

### 制限事項

- ファイルサイズ: 最大3MB
- 対応フォーマット: `.png` `.jpg` `.jpeg` `.gif` `.webp`

## コンテンツガイドライン

### 推奨される記事の特徴

- タイトルと内容が一致している
- 冒頭に明確な概要がある
- 環境や再現条件が詳細に記述されている
- 対象読者が明確
- 個人の経験や考察が含まれている

### 避けるべき内容

- 広告や宣伝が主目的の投稿
- 誇張的なクリックベイトタイトル
- 著作権を侵害するコンテンツ
- 検証されていないAI生成コンテンツ

## 参考リンク

- [Zennについて](https://zenn.dev/about)
- [コミュニティガイドライン](https://zenn.dev/guideline)
- [GitHub連携ガイド](https://zenn.dev/zenn/articles/connect-to-github)
- [Markdownガイド](https://zenn.dev/zenn/articles/markdown-guide)
- [Zenn CLI](https://zenn.dev/zenn/articles/install-zenn-cli)
