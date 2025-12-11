---
title: Home Assistant Power for Kiro を作った経験から学ぶ Kiro Power の作り方
emoji: ⚡
type: tech
topics:
  - kiro
  - aws
  - mcp
  - skills
  - homeassistant
published: true
publication_name: aws_japan
---

私は自宅のスマートホーム化が趣味で、その中心として [Home Assistant](https://www.home-assistant.io/) というソフトを使っています。GitHubの年次レポートである [Octoverse 2025](https://github.blog/news-insights/octoverse/octoverse-a-new-developer-joins-github-every-second-as-ai-leads-typescript-to-1/) によると、Home Assistant は「今年急成長したリポジトリ」の第10位に入るほど人気のあるソフトです。

そんな Home Assistant のための強力な非公式 MCP Server、[ha-mcp](https://github.com/homeassistant-ai/ha-mcp)がリリースされました。[公式 MCP Server](https://www.home-assistant.io/integrations/mcp_server/) もあったのですが、これは音声アシストで動くエンティティしか制御できないという制限がある一方、ha-mcpは Home Assistant のAPIすべてを網羅する勢いで開発されています。

一方で、ツールが80以上あるという機能の豊富さから、ha-mcpを有効にすると、私の環境ではコンテキスト消費量が14%から33%に爆増してしまいました。ちょうどそんなとき、この問題を解決できそうな機能、Kiro Power がリリースされたので、ちょうど良いとばかりにha-mcpを使って [Home Assistant Power for Kiro](https://github.com/rewse/kiro-power-homeassistant) を作ってみました。

この記事では、自分で作成した経験をもとに Kiro Power の作り方を紹介します。Home Assistant の知識は不要です。Home Assistant は Kiro Power の具体例として登場するだけで、記事の内容は他のどんな MCP Server にも応用できます。

![Home Assisstant Power for Kiro](https://raw.githubusercontent.com/rewse/kiro-power-homeassistant/refs/heads/main/cover.jpg)

:::message
この記事はKiroやその他のAIコーディングツールを使ったことがある方、事前知識がある方を対象としています。基本的な使い方については[Kiro公式ドキュメント](https://kiro.dev/docs/)を参照してください。
:::

## Kiro Power とは

このコンテキスト消費の問題を解決するのが Kiro Power です。通常の MCP Server 設定では、使うかどうかに関わらず、Kiro起動時にすべてのツール定義がコンテキストに読み込まれます。ha-mcpのように80以上のツールを持つ MCP Server だと、それだけで19% (= 33 - 14) ものコンテキストを常時消費してしまいます。

Kiro Power は、この問題を動的読み込みで解決します。具体的には、`Home Assistant` のような特定キーワードがプロンプトに含まれたとき、初めて関連する MCP Server のツールと専門知識（ステアリング）が読み込まれます。つまり、Home Assistant と関係ない作業をしているときはコンテキストを消費せず、必要になったタイミングでKiroが Home Assistant の専門エージェントに変身するという仕組みです。

:::message
ステアリングは、Kiroの振る舞いをカスタマイズするためのルールや指示を記述したファイルです。コーディング規約 / コミットメッセージの形式 / プロジェクト固有のルールなどを定義することで、Kiroがそれらに従った提案をしてくれるようになります。
:::

さらに、MCP Server の設定とステアリングを一つのパッケージとして配布できるため、誰かが作った Kiro Power を簡単にインポートしてプラグインできます。

他のAIコーディングツールを使っている方向けに補足すると、Claude Code や OpenAI Codex にはSkillsというAIに専門知識をプラグインする仕組みがあります。Kiro Power はそれに MCP Server を組み合わせたもの、つまり Power = Skills + MCP として考えれば分かりやすいでしょう。もっと詳しく知りたい方は、公式ブログの [Introducing Kiro powers](https://kiro.dev/blog/introducing-powers/)（[日本語訳](https://aws.amazon.com/jp/blogs/news/introducing-powers/)）が参考になります。

## Kiro Power の構成要素

最小構成のPowerは`POWER.md`だけです。ツールなしで知識だけを動的ロードするPowerで、Claude Code などのSkillsに相当します。

```
power-tiny/
  └── POWER.md
```

MCP Server も含めたい場合は`mcp.json`を追加します。これが典型的なPowerの構成です。`mcp.json`には動的に読み込みたい MCP Server の設定を記述します。`POWER.md`にはステアリングとなる知識を書きます。

```
power-small/
  ├── POWER.md
  └── mcp.json
```

ただし、この構成だと`POWER.md`の内容すべてが一回の動的ロードで読まれます。`POWER.md`がけっこう大きいのに、一部の内容は限られた状況にしか関係ないという場合、コンテキスト消費が無駄になります。そのような場合は以下のように`steering`ディレクトリ内の別ファイルとすることで、その状況になったとき改めてそのファイルだけ動的ロードできるようになります。ファイル名は`.md`で終われば、あとは自由です。

```
power-medium/
  ├── POWER.md
  ├── mcp.json
  └── steering/
        └── rare-guide.md
```

このような「限られた状況でしか使われないステアリング」がたくさんある場合、以下のようにそれぞれ別ファイルにすることでコンテキスト消費をさらに抑えることができます。

```
power-large/
  ├── POWER.md
  ├── mcp.json
  └── steering/
        ├── rare-guide.md
        ├── epic-guide.md
        └── legendary-guide.md
```

なお、Powerのディレクトリには隠しファイルや実行ファイルなどを含むことができません。例えば、`.git`ディレクトリが含まれているとインポートでエラーになります。

![Unable to install power: Power validation failed: Power contains disallowed files: - .git - .github - .gitignore - .kiro - scripts/bump_version.sh Powers should only contain: POWER.md, mcp.json, and steering/*.md files. Binary executables, scripts, hidden files, credentials, and archives are not permitted in power packages.](/images/kiro-power-creation-guide/power-validation-failed-error.png)
*`.git`などが含まれているときに発生する Power validation failed エラー*

そのため、GitHubで公開する場合は、以下のようにPower本体をサブディレクトリに配置する構成をお薦めします。[公式マニュアル](https://kiro.dev/docs/powers/create/#sharing-your-power)ではリポジトリのルートに`POWER.md`などを置く例が紹介されていますが、その構成だと`.git`などが含まれてしまうためです。

```
kiro-power-medium-tool/          # GitHubリポジトリ
  ├── power-medium-tool/         # ← このディレクトリをインポート
  │   ├── POWER.md
  │   ├── mcp.json
  │   └── steering/
  │       └── rare-prompt.md
  ├── .git                       # Powerには含められない
  ├── .gitignore                 # Powerには含められない
  ├── LICENSE.md
  └── README.md
```

インポート時は、リポジトリ全体 (`kiro-power-medium-tool`) ではなく、Power本体のサブディレクトリ(`power-medium-tool`) を指定します。

## mcp.jsonの構造

`mcp.json`は、Kiroの `/.kiro/settings/mcp.json` と同じ形式です。`"mcpServers"`の中に動的に読み込みたい MCP Server の設定を記述します。

Home Assistant Power では以下のように、ha-mcpだけでなく、URL先を読みにいくためのfetchも入れています。

```json:mcp.json
{
  "mcpServers": {
    "Home Assistant": {
      "command": "uvx",
      "args": ["ha-mcp@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR",
        "HOMEASSISTANT_URL": "${HOMEASSISTANT_URL}",
        "HOMEASSISTANT_TOKEN": "${HOMEASSISTANT_TOKEN}"
      },
      "autoApprove": [
	  ...
      ]
    },
    "fetch": {
      "command": "uvx",
      "args": [
        "mcp-server-fetch",
        "--ignore-robots-txt"
      ],
      "autoApprove": [
        "fetch"
      ]
    }
  }
}
```

`"env"`内の`${HOMEASSISTANT_URL}`のような`${...}`形式は、シェルの環境変数を参照します。KiroがこのPowerをインポートしてアクティベートするときに、最適な方法で実装してくれます。

## POWER.mdの構造

### Front Matter とキーワード選定

`POWER.md`の先頭は以下のような Front Matter です。重要なのは`keywords`で、これらのキーワードのどれかがプロンプトに含まれていたときにPowerが読み込まれます。


```markdown:POWER.md
---
name: "homeassistant"
displayName: "Home Assistant"
description: "Control your smart home with natural language. 80+ tools for device control, automation creation, dashboard management, and more"
keywords: ["homeassistant", "home assistant", "hass", "ha-mcp", "lovelace"]
author: "Shibata, Tats"
---
```

キーワード選定のポイントは以下のとおりです。

- 固有名詞を優先する: Powerはユーザーレベルでインポートされるため、一般的な用語は避けましょう。例えば`HA`を含めてしまうと、Home Assistant とまったく関係ないプロジェクトで High Availability（高可用性）のつもりで「HA環境も追加して」と言ったときに Home Assistant Power が読み込まれてしまいます
- 表記揺れを網羅する: `homeassistant`（スペースなし）と `home assistant`（スペースあり）のように、ユーザーが使いそうな表記を複数登録しておきます
- 関連ツール名を含める: `ha-mcp`のように、このPowerで使う MCP Server の名前を含めておくと、ユーザーがそのツール名でも呼び出せます
- 特徴的な機能名を含める: `lovelace`（Home Assistant のダッシュボード機能）のように、そのドメイン特有の用語を含めておくと、詳しいユーザーがピンポイントで呼び出せます

### 本文の構成

Front Matter 以外は自由に記述できますが、[公式 Kiro Power Repository](https://github.com/kirodotdev/powers) 内のPowerに何が書いてあるかを参考にして、Home Assistant Power では以下のような構成にしています。

```markdown:POWER.md
# Overview
# Onboarding
## Step 1
## Step 2
[...]
# When to Load Steering Files
# Common Workflows
# Best Practices
# Troubleshooting
# Resources

---
**Package:** `ha-mcp`
**Source:** [Home Assistant AI Community](https://github.com/homeassistant-ai/ha-mcp)
**License:** MIT
```

`Overview`には、このPowerで何ができるかを記述しています。

`Onboarding`にインストール方法を記述しておくと、PowerをインポートしたタイミングでKiroがこのステップで処理を進めてくれます。たとえば「uvがあるかどうか確認」「なければuvをインストール」「環境変数の設定」「動作確認方法」などを書いておくと良いでしょう。

`steering`ディレクトリ内に複数のステアリングを入れている場合、どのようなユースケースでどのファイルを読むのかを `When to Load Steering Files` に記述します。これはKiroへの指示で、「こういう状況になったら、このステアリングファイルを追加で読み込んで」というルールを定義するものです。Home Assistant Power では以下のような感じにしています。

```markdown:POWER.md
WHEN writing YAML automations, scripts, triggers, conditions, or control flow (repeat, choose, if-then, parallel, wait, delays, response variables)
THEN load `homeassistant-dev-guide.md` and `homeassistant-scripts-guide.md`

WHEN debugging automation or script issues
THEN load `homeassistant-dev-guide.md`, `homeassistant-scripts-guide.md`, and `homeassistant-templating-guide.md`

WHEN working with Jinja2 templates (entity states, attributes, time/dates, filtering, aggregation, areas, devices, macros)
THEN load `homeassistant-templating-guide.md`

WHEN working with mobile notifications, location tracking, app sensors, widgets, Siri Shortcuts, Android Quick Seetings, or troubleshooting app issues
THEN load `homeassistant-companion-app-guide.md`

WHEN looking up specific MCP tool usage or parameters
THEN load `homeassistant-mcp-tools.md`
[...]

<!-- 日本語訳:
YAMLオートメーション、スクリプト、トリガー、条件、制御フローを書くとき
homeassistant-dev-guide.md と homeassistant-scripts-guide.md を読み込む

オートメーションやスクリプトの問題をデバッグするとき
homeassistant-dev-guide.md と homeassistant-scripts-guide.md、homeassistant-templating-guide.md を読み込む

Jinja2テンプレートを扱うとき
homeassistant-templating-guide.md を読み込む

モバイル通知、位置追跡、アプリセンサー、ウィジェット、Siriショートカット、Android Quick Settings、アプリの問題のトラブルシューティングを扱うとき
homeassistant-companion-app-guide.md を読み込む

特定のMCPツールの使い方やパラメータを調べるとき
homeassistant-mcp-tools.md を読み込む
-->
```

`Common Workflows` は `When to Load Steering Files` と似ていますが、より具体的なシナリオを記述します。人間からの質問例に対して、Kiroがどのようなステップで処理すべきかを示すものです。`When to Load Steering Files` が「いつ何を読むか」のルールなのに対し、`Common Workflows` は「こう聞かれたらこう動く」という具体例です。

```markdown:POWER.md
## Create Automation (MCP + Steering)

User: Create an automation that turns on the porch light at sunset
Agent: Loads homeassistant-dev-guide.md for YAML conventions
       Loads homeassistant-scripts-guide.md for action syntax
       Creates automation following best practices:
       - Descriptive alias: "Porch Light at Sunset"
       - Service action targets (not legacy entity_id)
       - Two-space indentation, proper quoting
       Uses ha_config_set_automation to deploy
       Confirms automation is created and enabled

## Debug Automation (MCP + Steering)

User: Why isn't my motion sensor automation working?
Agent: Uses ha_get_automation_traces to check execution history
       Loads homeassistant-dev-guide.md if YAML issues suspected
       Loads homeassistant-scripts-guide.md for action/condition syntax
       Loads homeassistant-templating-guide.md if template errors found
       Identifies trigger conditions, timing issues, or syntax problems
       Suggests fixes based on trace analysis and best practices
[...]

<!-- 日本語訳:
## オートメーションの作成（MCP + ステアリング）

ユーザー: 日没時に玄関の照明を点けるオートメーションを作って
エージェント: YAML規約のために homeassistant-dev-guide.md を読み込む
              アクション構文のために homeassistant-scripts-guide.md を読み込む
              ベストプラクティスに従ってオートメーションを作成
              - 説明的なエイリアス: 「日没時の玄関照明」
              - targetを使ったサービスアクション（古いentity_id直接指定ではなく）
              - 2スペースインデント、適切な引用符
              ha_config_set_automation を使ってデプロイ
              オートメーションが作成・有効化されたことを確認

## オートメーションのデバッグ（MCP + ステアリング）

ユーザー: モーションセンサーのオートメーションが動かないのはなぜ？
エージェント: ha_get_automation_traces で実行履歴を確認
              YAMLの問題が疑われる場合は homeassistant-dev-guide.md を読み込む
              アクション/条件構文のために homeassistant-scripts-guide.md を読み込む
              テンプレートエラーが見つかった場合は homeassistant-templating-guide.md を読み込む
              トリガー条件、タイミングの問題、構文の問題を特定
              トレース分析とベストプラクティスに基づいて修正を提案
-->
```

`Best Practices` にはトークンなどの機密情報の保存方法などを記述しています。`Troubleshooting` には接続エラーなどの、このPowerを使おうとしたときに起きそうなトラブルの解決方法を記述しています。`Resources` には参考文献をリストしています。フッターは仕様にないのですが、公式Powerには以下のように書いてあるものが多いので、それに合わせています。

### Kiroに書いてもらう方法

これらを一から書くとなったら大変ですよね。大丈夫です、これらもKiroに書いてもらいましょう。Kiroに公式ドキュメントや参考記事のURLを渡して、それらを読ませた上で`POWER.md`を生成してもらう方法です。

そのためには、KiroがURLの内容を読めるようにする必要があるので、fetch MCP Server を設定しましょう。MCP Servers パネルの右上の📝アイコンを押してエディターを開くと、以下のような設定が自動入力されるはずです。`"--ignore-robots-txt"` を追加して、`disabled`を`false`に変え、`autoApprove`に`fetch`を追加して保存します。Connecting...の表示の後、✓が表示されたら有効になっています。

```json
{
 "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch",  "--ignore-robots-txt"],  
      "env": {},
      "disabled": false,
      "autoApprove": ["fetch"]
    }
  }
}
```

![fetch MCP Server が有効になった状態](https://res.cloudinary.com/zenn/image/fetch/s--f7iZK2gA--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/9046a625e790523690cecc89.png%3Fsha%3D339b94dc510495a5c7085dd5baa9329408e8719b)

:::message
何らかのエラーでfetchが有効にならない場合は、ウィンドウ右下のチャットボックスからKiroに「fetch MCPサーバーが動かないけど原因はなに？」と聞いてみてください
:::

fetch MCP Server が使えるようになったら、Kiroに以下のように尋ねます。

```markdown
Home Assistant のための Kiro Power を作りたい。
以下のような見出しで POWER.md を英語で作成して。

# Overview
# Onboarding
## Step 1
## Step 2
[...]
# When to Load Steering Files
# Common Workflows
# Best Practices
# Troubleshooting
# Resources

書き始める前に以下を読んで、それらを参考にして。

- https://kiro.dev/docs/powers/create/
- https://kiro.dev/blog/introducing-powers/
- https://github.com/kirodotdev/powers 内の各ディレクトリ
- https://github.com/homeassistant-ai/ha-mcp/README.md
- https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/FAQ.md
- https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/Windows-uv-guide.md
- https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/macOS-uv-guide.md
```

これで95%くらいの品質でKiroが仕上げてくるので、追加でさらにKiroに指示したり、手動で残り5%を調整したりするだけで済みます。`Common Workflows` などをOpusできちんと書いておいたら、Haikuでもうまいことやってくれそうな気がします。

あなたが役に立った文書はKiroにもきっと役に立ちます。それらを参考文献として渡しましょう。公式ドキュメントだけでなく、誰かが書いたすばらしいブログ記事なんかも渡すと、あなたらしい Kiro Power になるでしょう。そう、例えば Home Assistant Power とか、この記事とかですね！

[Obsidian Web Clipper](https://obsidian.md/clipper) を使って役に立つ記事をクリップしている場合は、[MCP server for Obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) をKiroで使えるようにして、「Obsidianからも関連記事を読んで」とかやってみても良いかもしれません。

なお、英語であることは必須でないものの、英語のほうがトークン量を一般的には減らせます。長文の英語を読みたくない場合は、Kiroに「レビュー用に日本語訳を表示して」と尋ねれば、`POWER.md`は英語のまま日本語訳を表示してくれます。

## ステアリングの作成

ステアリングも人間が一から書くのは大変なので、Kiroに書いてもらっています。

```markdown
steeringディレクトリの下に新規ステアリングを英語で作成。
参考文献は以下のとおり

- https://www.home-assistant.io/docs/scripts/
- https://www.home-assistant.io/docs/scripts/perform-actions/
- https://www.home-assistant.io/docs/scripts/conditions/
```

これによって例えば `homeassistant-scripts-guide.md` というファイルが生成されたら、続いて以下のように尋ねます。

```markdown
- homeassistant-scripts-guide.md を使用するユースケースを
  POWER.md の When to Load Steering Files に追加。
- homeassistant-scripts-guide.md を使用するワークフローを
  POWER.md の Common Workflows に追加。

どちらも現在のファイルに単に追加するのではなく、既存のものを更新すべきか、
新たなものを作るかべきか、適切に判断して。
```

これで先ほど作成したステアリングが`POWER.md`に統合されました。

このような手順を繰り返して、Home Assistant Power は5個以上のステアリングを含んでいます。似たような Best Practices やTroubleshootingが複数のファイルでできてしまったりするので、「foo.mdとbar.mdの Best Practices をPOWER.mdのそれに統合」などと依頼して整理しました。

また、`When to Load Steering Files` と `Common Workflows` がどんどん大きくなっていくので、「多すぎるな」と思ったタイミングで整理しても良いと思います。

```markdown
POWER.md の When to Load Steering Files と Common Workflows の項目を減らしたい。

- 似たようなものを整理して
- あまり一般的でないものは steering/advanced-workflows.md に移動して
- 「ワークフローの例をもっと知りたい」たい時に、advanced-workflows.md を読む
  というユースケースを When to Load Steering Files に追加して
```

:::message
Home Assistant Power の最新のステアリングは [kiro\-power\-homeassistant/power\-homeassistant/steering at main · rewse/kiro\-power\-homeassistant](https://github.com/rewse/kiro-power-homeassistant/tree/main/power-homeassistant/steering) から参照できます。
:::

## Kiro Power のインポート

作成した Kiro Power をGitHubで公開すると、他のユーザーはそれを Custom Power として簡単にインポートしてプラグインできます。

Custom Power を使いたい場合、左パネルからPowerパネル選んで、`Add Custom Power` を選びます。`Import power from GitHub` を選んで`POWER.md`が含まれているディレクトリ（例えば [https://github.com/rewse/kiro-power-homeassistant/tree/main/power-homeassistant](https://github.com/rewse/kiro-power-homeassistant/tree/main/power-homeassistant)）を入力するか、GitHubから事前にダウンロードしておいて、`Import power from a folder` を選んで`POWER.md`が含まれているフォルダーを選びます。

![Add Custom Power メニュー](/images/kiro-power-creation-guide/add-custom-power-menu.png)

Powerのインポート後、`Try power` を押すと、Kiroが`POWER.md`に書かれた`Onboarding`に従ってセットアップを行います。

![Powerインポート完了画面](/images/kiro-power-creation-guide/power-import-complete.png)
*`#homeassistant`などのDescription下のものがPowerをトリガーするキーワード*

インポートが完了すると、全てのファイルが `/.kiro/powers/installed` にコピーされ、`/.kiro/settings/mcp.json` に以下のように追記されます。

```json
{
  "mcpServers": {
    ...
  },
  "powers": {
    "mcpServers": {
      "power-homeassistant-Home Assistant": {
        "command": "uvx",
       "args": ["ha-mcp@latest"],
        "env": {
          "FASTMCP_LOG_LEVEL": "ERROR",
          "HOMEASSISTANT_URL": "${HOMEASSISTANT_URL}",
          "HOMEASSISTANT_TOKEN": "${HOMEASSISTANT_TOKEN}"
        },
        "autoApprove": [
  	  ...
        ]
    },
    ...
  }
}
```

### Custom Powerの更新について

Custom Power を更新する場合の注意点として、Powerパネルの `Check for updates` ボタンは公式Powerのみが対象で、Custom Power には使えません。Custom Power を更新するには、以下のいずれかの方法を取ります。

1. 再度 `Add Custom Power` でインポートし直す
2. `~/.kiro/powers/installed/` 内のファイルを手動で上書きする

1の方法は簡単ですが、`~/.kiro/settings/mcp.json` の `"powers"` セクションも初期値で上書きされます。もしセットアップの過程で例えば `"${HOMEASSISTANT_URL}"` を実際のURLに直接書き換えているような場合は、その変更が消えてしまいます。そのような場合は2の方法で、`~/.kiro/powers/installed/` 内のファイルだけを新しいもので上書きするほうが楽かもしれません。

## まとめ

Kiro Power は、MCP Server と専門知識を組み合わせて、特定のキーワードで動的に読み込まれる仕組みです。これにより、常時コンテキストを消費することなく、必要なときだけKiroが専門的なエージェントに変身できます。

構成はシンプルで、最小構成なら `POWER.md` だけ、MCP Server を含める場合は `mcp.json` を追加し、複雑な知識は `steering/` ディレクトリに分割できます。注意点としては、これらのファイル以外を含めないことです。

`POWER.md` の作成は、Kiro自身に公式ドキュメントや参考記事を読ませて書いてもらうのが効率的です。fetch MCP Server 設定後にURLを渡すだけで関連情報を収集して、適切な構成で仕上げてくれます。ステアリングファイルも同様に、参考文献を渡してKiroに作成してもらい、`POWER.md` に統合する流れで進められます。

完成したPowerをGitHubで公開すれば、他のユーザーは簡単にインポートしてプラグインできます。

公式ドキュメントだけでなく、あなたが役に立った記事も参考文献として渡して、あなたらしい Kiro Power を作ってみてください。

## この記事の動作環境

- Kiro: 0.7.21
