# コミットメッセージ規約

## 基本原則
[Conventional Commits v1.0.0](https://www.conventionalcommits.org/ja/v1.0.0/) 規約に従ってコミットメッセージを作成する。

## 概要
Conventional Commits の仕様はコミットメッセージのための軽量の規約です。
明示的なコミット履歴を作成するための簡単なルールを提供し、自動化ツールの導入を簡単にします。

## コミットメッセージの構造

```
<型>[任意 スコープ]: <タイトル>

[任意 本文]

[任意 フッター]
```

## 必須の型（Type）

### 基本の型
- **feat**: 新しい機能を追加するコミット（Semantic Versioningの`MINOR`に相当）
- **fix**: バグにパッチを当てるコミット（Semantic Versioningの`PATCH`に相当）

### その他の推奨される型
[@commitlint/config-conventional](https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional)（Angularの規約が基）で推奨される型：

- **build**: ビルドシステムや外部依存関係に影響する変更
- **chore**: その他の変更
- **ci**: CI設定ファイルやスクリプトの変更
- **docs**: ドキュメントのみの変更
- **style**: コードの意味に影響しない変更（空白、フォーマットなど）
- **refactor**: バグ修正でも機能追加でもないコード変更
- **perf**: パフォーマンスを向上させるコード変更
- **test**: テストの追加や既存テストの修正

## スコープ（Scope）
型の後ろに記述できる任意の項目。コードベースのセクションを記述する括弧で囲まれた名詞：
- **articles**: 記事関連の変更
- **books**: 本関連の変更
- **config**: 設定ファイルの変更
- **deps**: 依存関係の変更
- **zenn**: Zenn CLI関連の変更

例: `feat(parser):`, `fix(articles):`

## タイトル（Description）
型/スコープの後ろのコロンとスペースの直後に続く、コード変更の短い要約：
- **英語で記述する**
- コード変更の短い要約
- 現在形の命令法で記述（"added" ではなく "add"）
- 最初の文字は小文字
- 末尾にピリオド（.）は付けない

## 本文（Body）
- 任意項目
- タイトルの下の1行の空行から始める
- コード変更に関する追加の情報を提供
- 自由な形式で、改行で区切られた複数の段落で構成可能

## フッター（Footer）
- 任意項目
- 本文の下の1行の空行に続けて記述
- 単語トークン + `:<space>` または `<space>#` + 文字列の値で構成
- フッターのトークンは空白の代わりに `-` を使用（例: `Acked-by`）
- 例外として `BREAKING CHANGE` はトークンとして使用可能

## 破壊的変更（BREAKING CHANGE）
API の破壊的変更を導入する場合（Semantic Versioningの`MAJOR`に相当）：

### 方法1: 型/スコープの接頭辞に `!` を追加
```
feat!: APIの認証方式を変更
chore!: drop support for Node 6
feat(api)!: send an email to the customer when a product is shipped
```

### 方法2: フッターに `BREAKING CHANGE:` を記述
```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

### 両方を使用することも可能
```
chore!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

## 実例

### タイトルのみのコミット
```
docs: correct spelling of CHANGELOG
feat: add new article creation feature
fix(articles): fix markdown syntax error
```

### スコープ付きのコミット
```
feat(lang): add polish language
feat(articles): add synology nas article
fix(config): fix zenn cli configuration issue
```

### 破壊的変更を目立たせるための `!` 付きコミット
```
feat!: send an email to the customer when a product is shipped
feat(api)!: send an email to the customer when a product is shipped
```

### 破壊的変更のフッター付きコミット
```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

### 複数段落の本文と複数のフッターを持つコミット
```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: #123
```

### プロジェクト固有の記述例
```
feat(articles): add new tech article template

Introduce standardized template structure to improve consistency
when creating articles.

- Standardize front matter format
- Unify section structure
- Add code block guidelines

Closes #123
```

## 仕様の要点

1. コミットは型から始まり、その後ろにスコープ（任意）と `!`（任意）、コロンとスペースが続く
2. 新機能追加時は `feat` を使用
3. バグ修正時は `fix` を使用
4. スコープは括弧で囲まれた名詞
5. タイトルは型/スコープの後のコロンとスペースの直後に記述
6. 本文はタイトルの下の1行の空行から始める
7. フッターは本文の下の1行の空行に続けて記述
8. 破壊的変更は型/スコープの接頭辞またはフッターで明示
9. `BREAKING CHANGE` 以外は大文字と小文字を区別しない

## 英語での記述ガイドライン

### 必須ルール
- **タイトル部分は必ず英語で記述する**
- **本文は英語で記述する**
- **フッターは英語で記述する**
- 型は英語を使用（`feat`, `fix`, `docs` など）
- スコープは英語を使用（`articles`, `config` など）

### 英語記述の注意点
- 簡潔で明確な英語で記述する
- 現在形の命令法を使用する（"add", "fix", "update" など）
- 一般的な開発用語を使用する
- 冠詞（a, an, the）は省略可能

### 推奨される英語表現
- **動詞**: add, remove, fix, change, update, improve, refactor
- **名詞**: feature, bug, error, test, config, dependency, performance
- **形容詞**: new, existing, broken, missing, deprecated

### よく使われるパターン
- `add [feature/file/function]`
- `fix [bug/issue/error]`
- `update [dependency/config/documentation]`
- `remove [deprecated/unused/obsolete] [item]`
- `improve [performance/readability/structure]`

### 文章の単語レベル変更に関するルール
文章の修正や改善を行う場合の記述方法：

#### タイトル（英語）
- 文字数制限を考慮して抽象的表現を許容する
- 例: `improve readability`, `fix typos`, `update wording`

#### 本文（英語）
- 修正点を具体的に記述し、抽象的表現は避ける
- 日本語の修正内容は英語に変換せず、日本語のまま記述する
- 例:
  ```
  docs: improve article readability
  
  Fix specific wording issues:
  - 「設定を行う」→「設定する」
  - 「実行を行います」→「実行します」
  - Remove redundant expressions in technical explanations
  ```

## Git コマンドの注意点

### git diff の使用時
`git diff` を使用してステージされた変更を確認する場合は、必ず `cat` コマンドをパイプして使用する：

```bash
# 推奨: catをパイプして使用
git diff | cat

# 非推奨: 直接実行
git diff
```

この方法により、差分の内容が適切に表示され、コミットメッセージ作成時の参考情報として活用できる。
