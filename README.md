# Zenn Content Repository

[Zenn](https://zenn.dev/) に公開する記事と本を管理するリポジトリです。ZennのGitHub連携を使っているため、`main` ブランチにプッシュすると `published: true` の記事と本がZennに反映されます。

## セットアップ

Node.js 22 以上が必要です（zenn-cliの動作要件）。Zennアカウントとこのリポジトリを [GitHub連携](https://zenn.dev/zenn/articles/connect-to-github)しておきます。

```bash
npm install
```

## 使い方

```bash
# 記事を作成
npx zenn new:article --slug article-slug --title "タイトル" --type tech --emoji ✨

# 本を作成
npx zenn new:book --slug book-slug

# http://localhost:8000 でプレビュー
npx zenn preview

# 記事の文章を textlint でチェック
npm run lint
```

### 公開・更新・削除

記事の Front Matter で `published: true` にしてプッシュすると公開されます。同じスラッグのまま編集してプッシュすれば更新になり、ファイル名（スラッグ）を変えると別の記事として扱われます。コミットメッセージに `[skip ci]` を含めると、そのプッシュではデプロイされません。

ファイルを削除してもZenn上の記事は消えないので、削除は [ダッシュボード](https://zenn.dev/dashboard) から行います。

## ディレクトリ構成

```
.
├── .github/workflows/  # シークレットと脆弱性のスキャン
├── articles/           # 記事（<slug>.md）
├── books/              # 本（<slug>/config.yaml とチャプター）
├── images/             # 画像（<slug>/ ごとに配置）
├── .textlintrc.json    # textlint の設定
├── AGENTS.md           # AI エージェント向けの執筆・コミット規約
└── package.json
```

## 記事のFront Matter

```yaml
---
title: "記事のタイトル"
emoji: "😸"              # アイコン絵文字（1文字）
type: "tech"             # "tech" または "idea"
topics: ["docker", "aws", "devops"]  # タグ（最大5個）
published: true          # false で下書き
published_at: 2050-06-12 09:03  # オプション: 公開予約日時
---
```

`type` は、プログラミングやインフラなど技術の具体的な内容なら `tech`、キャリアやマネジメント、技術に関する考え方やまとめ記事なら `idea` にします（[選び方](https://zenn.dev/tech-or-idea)）。

## 画像

画像は `images/<slug>/` に置き、記事からは `/images/` から始まる絶対パスを指定します。相対パスを使うとZenn上の画像が表示されません。

```markdown
![代替テキスト](/images/article-slug/screenshot-1.png)
```

対応形式は `.png` `.jpg` `.jpeg` `.gif` `.webp` で、1ファイル3MBまでです。これを超えるとデプロイが失敗します。

## 執筆ガイドライン

Zennの [コミュニティガイドライン](https://zenn.dev/guideline) に従います。タイトルは内容と一致させ、冒頭で記事の概要と対象読者を示します。再現できるように環境やバージョンを書き、公式情報に自分の経験や考察を加えます。宣伝が主目的の投稿、誇張したタイトル、引用の範囲を超えた転載、正確さを確認していないAI生成の文章は避けます。

記法、コミットメッセージ、その他の細かい規約は [AGENTS.md](AGENTS.md) にまとめています。

## 参考リンク

- [Markdown記法](https://zenn.dev/zenn/articles/markdown-guide)
- [Zenn CLIの使い方](https://zenn.dev/zenn/articles/install-zenn-cli)
- [Zennについて](https://zenn.dev/about)
