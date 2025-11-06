---
title: Kiroを使ってZennの執筆効率と品質を改善する
emoji: 👻
type: tech
topics:
  - kiro
  - zenn
  - ai
  - writing
  - productivity
published: true
publication_name: aws_japan
---

Zennで記事を書くときに、本文はそれなりに書けるとしても、良いタイトルを考えたり、適切なトピックを選ぶことが面倒だったりしませんか？ このようなアシスタント作業はAIに任せて、人間は本文に集中しましょう。

この記事では、Kiroを使ってZennの記事執筆を効率化し、品質を向上させる方法を紹介します。また、Kiroを一度も触ったことのない方が、どのようにKiroを始めれば良いのかを具体例で示した入門ガイドにもこの記事はなっています。

## 前提条件

- GitHubリポジトリとZennを連携させている
- Zenn CLI をインストール済み

上記の準備がまだの方は、以下のガイドに従って事前準備を行ってください。

- [アカウントにGitHubリポジトリを連携してZennのコンテンツを管理する](https://zenn.dev/zenn/articles/connect-to-github)
- [Zennで途中からGitHubリポジトリ連携をはじめるときの手順](https://zenn.dev/zenn/articles/setup-zenn-github-with-export)
- [Zenn CLIをインストールする](https://zenn.dev/zenn/articles/install-zenn-cli)

## Kiroを始めよう

### プロジェクトを開いてKiroと連携する

Kiroを https://kiro.dev/ からダウンロードして初期セットアップを終えたら `Open a project` からZennと連携しているGitHubリポジトリのローカルコピーを選びます。

![Kiroのプロジェクト選択ダイアログ](/images/kiro-zenn-writing-efficiency-improvement/kiro-open-project-dialog.png)

:::message
KiroのUIを日本語に変更したい場合は `View > Command Palette...` メニューを開き、`Configure Display Language` を検索して、日本語を選びます。
:::

### Kiroの機能を理解する

左パネルのKiroアイコンを選ぶと以下のような画面となります。それぞれのパネルについて簡単に説明します。

![Kiroパネルの概要画面](/images/kiro-zenn-writing-efficiency-improvement/kiro-panels-overview.png)

#### Specs: 今回は使用しない

Kiroには、チャットから始めて開発を行うVibeモードと、Kiroの大きな特徴の一つである、計画から始めて開発を行うSpecモードがあります。ただし、本記事ではVibeしか使わないため、このパネルは使用しません。

#### Agent Hooks: 今回は使用しない

ファイルの作成などをトリガーにKiroになにかをさせる機能が Agent Hook です。本記事では使用しません。

#### Agent Steering: 重要

Kiroに事前に知っておいてほしいREADMEのようなものを定義します。「Steering（操縦）」という名前のとおり、Kiroの回答や行動を適切な方向に導くための仕組みです。他のAI開発アシスタントで言う`AGENTS.md` / `CLAUDE.md` / `GEMINI.md` などと同様ですが、Kiroでは特定のファイルやフォルダーにのみ適用するなど、柔軟な設定が可能です。本記事ではこの設定を主に行います。

#### MCP Servers

MCP (Model Context Protocol) サーバーを定義します。本記事ではこちらも使用します。

:::message
MCPは、AIが外部のツールやデータソースにアクセスするための標準的な仕組みです。例えば、ウェブページを読み込んだり、データベースにアクセスしたりできるようになります。
:::

### 外部リソースにアクセスできるようにする: MCP設定

この後の設定でZennの公式ドキュメントなどの外部サイトを参照しますが、KiroはそのままだとURLを指定しても読みにいかないので、URL先を読みにいけるようになる`fetch`を使えるようにします。

MCP Servers パネルの右上の📝アイコンを押してエディターを開くと、以下のような設定が自動入力されるはずです。`disabled`を`false`に変え、`autoApprove`に`fetch`を追加して閉じます。Connecting... の表示の後、✅が表示されたら有効になっています。

```js
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"],
      "env": {},
      "disabled": false,
      "autoApprove": ["fetch"]
    }
  }
}
```

![](/images/kiro-zenn-writing-efficiency-improvement/mcp-server-fetch-config.png)

:::message
何らかのエラーで`fetch`が有効にならない場合は、ウィンドウ右下のチャットボックスからKiroに「fetch MCPサーバーが動かないけど原因はなに？」と聞いてみてください
:::

## KiroにZennの知識を教える: Agent Steering

### 基本的な知識ファイルを自動生成する

Agent Steering パネルにある `Generate Steering Docs` ボタンを押すとproduct / structure / techという3つのファイルが作成されます。

![](/images/kiro-zenn-writing-efficiency-improvement/generate-steering-docs-button.png)

:::message
右パネルのチャットの最後に `Credits used: 0.34` と表示されています。つまり、月間50クレジットの無料枠では、このくらいの作業が月間150回くらいできることになります。
:::

すべて英語で書かれているので、中身を読む前にKiroに日本語に翻訳してもらいましょう。ウィンドウ右下のチャットボックスに日本語で依頼します。

```
Steering内のファイルを日本語に翻訳
```

![](/images/kiro-zenn-writing-efficiency-improvement/translate-steering-files-chat.png)

#### プロジェクトの目的を教える: product.md

`product`には、このプロジェクトの目的 / コンセプト / ビジネス要件を記載します。上の画像のように、KiroはZennについてすでに多少知っているようですが、もっと詳しい前提知識を持ってもらうため、以下を右下のチャットボックスに入力します。

```
以下のページを参照して #product.md をリファイン
- https://zenn.dev/about
- https://zenn.dev/guideline
```

:::message
リファイン (Refine) とは「洗練させる」の意味です。AIに「良い感じにしてもらう」ときに使いやすい用語です。
:::

MCPサーバーが正しく動作すれば、チャットウィンドウ内に `Called MCP tool fetch` と表示されます。Kiroが`product`を以下のように変更し、Zennがどのようなプラットフォームなのか詳しく記述されました。

![](/images/kiro-zenn-writing-efficiency-improvement/product-md-after-refine.png)

一方で、`プロダクト概要`が Zenn CLI のデモプロジェクトと誤解されています。これは Zenn CLI しかインストールされていないリポジトリを使っていることと、私がリポジトリ名を`demo`にしてしまったからでしょう。そのため、ここを修正させます。

```
プロダクト概要が異なるので、そこだけ修正して。
このプロダクトはZennのコンテンツを管理するために連携しているGitHubリポジトリである。
```

:::message
`○○だけ修正して`は、問題なかった部分が変更されることを防ぐために良い方法です。
:::

![プロダクト概要を修正後の画面](/images/kiro-zenn-writing-efficiency-improvement/product-overview-corrected.png)

`プロダクト概要`が正しくなりました。このように Agent Steering は、人間が最初から最後まで書かなくても、参考文献を渡すだけで良い感じに書いてもらうことができます。

#### ファイル構造を教える: structure.md

`structure`には、このプロジェクトのファイル構造やアーキテクチャを記載します。すでに十分な品質なので、このまま使うことにします。

![](/images/kiro-zenn-writing-efficiency-improvement/structure-md-content.png)

#### 技術的な詳細を教える: tech.md

`tech`には、このプロジェクトの技術的な実装詳細 / ツール / デプロイ手順などを記載します。

![](/images/kiro-zenn-writing-efficiency-improvement/tech-md-initial-content.png)


Kiroは Zenn CLI については詳しいようですが、ファイル形式については不十分なため、参考文献を渡します。

```
以下のページを参照して #tech.md をリファイン
- https://zenn.dev/tech-or-idea
- https://zenn.dev/zenn/articles/what-is-slug
- https://zenn.dev/zenn/articles/zenn-cli-guide
```

![](/images/kiro-zenn-writing-efficiency-improvement/tech-md-after-refine.png)

`ファイル形式・構造`について詳しく書かれるようになりました。

### さらに詳しい設定を追加する

#### 日本語でやりとりするための設定

Kiroに日本語で質問すると日本語で答えてくれていますが、これを明示的に Agent Steering で定義しましょう。Agent Steering パネル右上の + ボタンを押してワークスペースレベルの新規 Agent Steering を`language`という名前で作成します。

![](/images/kiro-zenn-writing-efficiency-improvement/create-language-steering.png)

作成された`language.md`にルールを簡単に記述します。

```md
---
inclusion: always
---

- チャットのやりとりは日本語で行う
- Agent Steering は日本語で記述する
```

![](/images/kiro-zenn-writing-efficiency-improvement/language-md-initial-rules.png)

書き終わったら右上の`Refine`ボタンを押します。

![](/images/kiro-zenn-writing-efficiency-improvement/language-md-after-refine.png)

だいぶ良い感じにしてくれました。ただ、コミットメッセージは英語が良かったので、そこだけ手動で修正しました。

#### 適切なコミットメッセージを自動生成する設定

適切なコミットメッセージを書くことは退屈な割りに重要なので、Kiroに任せましょう。`commit-message-standards`という新規 Agent Steering を作成し、規約は [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) に従ってもらいましょう。

```md
---
inclusion: always
---

コミットメッセージは [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) に従う。
```

`Refine`ボタンを押すと以下のようになりました。

![](/images/kiro-zenn-writing-efficiency-improvement/commit-message-standards-initial.png)

悪くはないのですが、fetch MCPサーバーを使用せずに記憶で書いているのと、英語になってしまいました。この点をチャットで修正します。

```
https://www.conventionalcommits.org/en/v1.0.0/ を参照して、
その内容で #commit-message-standards.md を日本語でリファイン
```

![](/images/kiro-zenn-writing-efficiency-improvement/commit-message-standards-refined.png)

Zennの記事リポジトリであることを想定した良い Agent Steering ができました。

:::message
言語やコミットメッセージなどの Agent Steering は、全てのプロジェクトで共通利用したいかもしれません。これらのファイルをもっと汎用的な内容にして`~/.kiro/steering`に保存することで、全てのプロジェクトで適用される Global Agent Steering にすることもできます。
:::

#### ZennのMarkdown記法を教える設定

ZennのMarkdown記法は`tech`に書いてもよいのですが、この記法は`articles`と`books`フォルダー内のファイルにしか適用されず、Agent Steering には適用されないので、念のため別ファイルにして、`inclusion:`を使って適用範囲を狭めましょう。`zenn-markdown-standards`という新規 Agent Steering を作成し、[ZennのMarkdown記法一覧](https://zenn.dev/zenn/articles/markdown-guide) に従ってもらいましょう。

```md
---
inclusion: fileMatch
fileMatchPattern:
  - "articles/*.md"
  - "books/**/*.md"
---

Zennの記事と本のMarkdownは
[ZennのMarkdown記法一覧](https://zenn.dev/zenn/articles/markdown-guide) に従う。
```

`Refine`ボタンだとURLを参照しないので、直接チャットに以下を入力してリファインします。

```
https://zenn.dev/zenn/articles/markdown-guide を参照して、
その内容で #zenn-markdown-standards を日本語でリファイン
```

![](/images/kiro-zenn-writing-efficiency-improvement/zenn-markdown-standards-refined.png)

:::message
私が実際に使用している Agent Steering は以下から参照することができます。
- [zenn\-content/\.kiro/steering at main · rewse/zenn\-content](https://github.com/rewse/zenn-content/tree/main/.kiro/steering)
- [rewse/kiro\-global\-steering: Global Steering for Kiro](https://github.com/rewse/kiro-global-steering)
:::

## 実際に記事を書いてみよう

これで準備は整いました！ あとは記事を書いていきましょう。まったく違うことを始める場合はコンテキスト（今までのやりとりに基づく知識）をリセットするため、右側のチャットパネルを閉じて、新規チャットを始めるのが良いです。

### タイトルとファイルをKiroに作ってもらう

```
「Kiroを使ってZennの記事の品質を向上させる方法」について書きたい。
お薦めのタイトルは？
```

![](/images/kiro-zenn-writing-efficiency-improvement/article-title-suggestions.png)

```
「Kiroを使ってZennの記事の執筆効率と品質を改善する方法」にしよう。
適切なslugでファイルを作成して
```

![](/images/kiro-zenn-writing-efficiency-improvement/article-file-created.png)

適切なslugでファイルが作成され、Front Matter も良い感じです。本文まで書いてくれましたが、私はこれを参考にせず、全部消しています。

### 好みのエディターで本文を書く

このままKiroで本文を書いてもよいでしょうが、私は日常のメモをObsidianで書いていることもあって、Obsidianで書いています。Obsidianの `File > Open Vault... > Open folder as vault` からGitHubリポジトリを指定しても良いですし、すでに利用している Obsidian Vault があるならば、GitHubリポジトリをそのVault内に移動しても良いでしょう。

![](/images/kiro-zenn-writing-efficiency-improvement/obsidian-vault-setup.png)

## Kiroに記事をブラッシュアップしてもらう

本文を書き終えたら、最終調整をKiroに手伝ってもらうことで品質を向上させましょう。

### まずは誤字脱字をチェック

```
誤字脱字を修正して
```

![](/images/kiro-zenn-writing-efficiency-improvement/typo-correction-chat.png)

私はスペースを含む英語と日本語の間には半角スペースを入れているのですが、それが修正されてしまいました。今後のために、これは Agent Steering に含めたほうが良さそうです。

```
以下を #zenn-markdown-standards.md に追加
- スペースを含む英語と日本語の間には半角スペースを入れる
- スペースを含まない英単語と日本語の間には半角スペースを入れない
- 文末の！と？の後には半角スペースを入れる
```

![](/images/kiro-zenn-writing-efficiency-improvement/markdown-standards-updated.png)

このように、「そうじゃないんだよなー」というアシストが行われたときは、都度 Agent Steering に追記していき、Agent Steering を育てていくことが重要です。

:::message
より正確な Agent Steering を書くためには、[RFC 2119: Key words for use in RFCs to Indicate Requirement Levels](https://www.ietf.org/rfc/rfc2119.txt) と、[その日本語訳](https://www.nic.ad.jp/ja/tech/ipa/RFC2119JA.html)が参考になります。

例）
- スペースを含む英語と日本語の間には半角スペースを入れる必要がある (SHOULD)
- スペースを含まない英単語と日本語の間には半角スペースを入れないほうがよい (SHOULD NOT)
- 文末の！と？の後には半角スペースを入れる必要がある (SHOULD)
:::

### 読みやすさと構造を改善する

以下では、その他の校正のやりとりの例を挙げます。

```
あなたが読んで分かりづらかった点を指摘して
```

![](/images/kiro-zenn-writing-efficiency-improvement/readability-feedback.png)

どのように修正すべきかも聞いてみましょう。

```
1についてどうしたほうがよい？
```

![](/images/kiro-zenn-writing-efficiency-improvement/improvement-suggestion.png)

見出しや「まとめ」は本文の概要なので、AIが得意とする領域です。

```
目次となるように適切に見出しを入れて
```

![](/images/kiro-zenn-writing-efficiency-improvement/headings-added.png)

```
最後に「まとめ」を追加して
```

![](/images/kiro-zenn-writing-efficiency-improvement/summary-section-added.png)

いざ書いてみると当初の予定からは少しずれているときもあるので、タイトルとトピックを書き終わった後に再検討しても良いでしょう。

```
本文を読んでみて、より適切なタイトルとトピックがあれば提案して。
今のままで良ければ今のままにしておく
```

![](/images/kiro-zenn-writing-efficiency-improvement/title-topic-suggestions.png)

```
タイトルを「Kiro AI でZenn記事執筆を自動化する」に変更して、
それに合わせてファイル名のslugも変更
```

![](/images/kiro-zenn-writing-efficiency-improvement/title-slug-updated.png)

### 完成した記事をコミットする

文章が完成したらコミットしましょう。

```
新しい記事をコミット
```

![](/images/kiro-zenn-writing-efficiency-improvement/git-commit-with-message.png)

`commit-message-standards` Agent Steering で定義した規約に従ってコミットメッセージが作成されました。

## まとめ

この記事では、Kiroを使ってZennの記事執筆を効率化し、品質を向上させる方法を紹介しました。Agent Steering とMCPサーバーの設定によってKiroに適切な前提知識を与えることで、より精度の高いアシスタント機能を実現できます。また、誤字脱字チェックから構造改善まで、複数の観点で段階的に記事の品質を向上させることができます。このようにKiroに執筆の効率化と品質の改善を任せることで、人間は本文を書くことに集中できます。

## 補足: 無料枠でどのくらい使える？ クレジット使用量の実例

月間クレジット使用量はウィンドウ右下に表示されています。今回は47.39クレジットから始めて、すべての Agent Steering を作成し終えたときに53.99クレジットだったので、環境構築に6.6クレジット使用しました。コミットしたときには62.86クレジットだったので、一つの記事の校正に8.87クレジット使いました。そのため、校正内容や記事の規模にもよりますが、無料枠の毎月50クレジットで月に5本程度の校正をKiroで行えそうです。初回サインアップから30日間は500クレジットのボーナスがもらえるので、いろいろ試してみてください。無料枠で足りないほど記事を書いたり、Kiroを使いこなしてクレジット消費が多い場合は、月額$20払うと毎月1,000クレジットになります。

[![Kiroの料金プラン画面。KiroFREE（$0/月、50クレジット）、KiroPRO（$20/月、1,000クレジット）、KiroPRO+（$40/月、2,000クレジット）、KiroPOWER（$200/月、10,000クレジット）の4つのプラン](/images/kiro-zenn-writing-efficiency-improvement/kiro-credit-usage.png)](https://kiro.dev/pricing/)

## この記事の動作環境

- Kiro: 0.5.9
