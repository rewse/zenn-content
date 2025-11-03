---
inclusion: always
---

# Zennプロジェクト コミットメッセージ標準

## 基本方針
Conventional Commitsに準拠し、Zennコンテンツ制作に特化したコミットメッセージ規約を適用する。

## 型（type）とスコープ（scope）

### 標準的な型
- `feat`: 新機能追加、新しいコンテンツ作成
- `fix`: バグ修正、誤字脱字、リンク切れなどの修正
- `docs`: ドキュメント更新（README、ガイド等）
- `style`: フォーマット、表記統一（内容に影響しない変更）
- `refactor`: 内容の再構成、可読性向上（機能変更なし）
- `chore`: 依存関係更新、設定ファイル変更、メタデータ更新

### Zenn特有のスコープ
- `article`: 記事関連の変更
- `book`: 本関連の変更
- `content`: その他コンテンツファイルの変更

## スコープの詳細指定

### 記事関連
- `article`: 記事全般への変更
- `article:slug名`: 特定記事への変更

### 本関連  
- `book`: 本全般への変更
- `book:slug名`: 特定の本への変更

## コミットメッセージ例

### 新機能・コンテンツ作成
```
feat(article): add synology nas btrfs troubleshooting guide
feat(article:docker-guide): add new section on volumes
feat(book): create new kubernetes deployment guide
feat(book:k8s-guide): add chapter on service mesh
```

### 修正・改善
```
fix(article): correct broken internal links in multiple articles
fix(article:synology-nas): fix incorrect command in step 3
fix(article): update outdated package versions in examples
```

### その他の変更
```
refactor(article:docker-guide): reorganize sections for better flow
style(article): standardize code block formatting across articles
chore: update zenn-cli to latest version
docs: add contribution guidelines
```

## 本文記述ルール

### 具体性の重視
- ❌ 抽象的: "improve readability"
- ✅ 具体的: "add filename to code blocks"
- ✅ 具体的: "expand step 3 explanation with details"

### 型の判断基準

#### featを使う場合
- 新しい記事・本の作成
- 記事内容の追加・更新（新しいセクション、情報追加）
- 記事の公開（下書きから公開状態への変更）
- 新機能やツールの追加

#### fixを使う場合  
- 明らかな間違いの修正（誤字、コマンドエラー、リンク切れ）
- 技術的な不正確さの修正
- フォーマットエラーの修正

### 日本語単語の扱い
```
fix: correct technical terminology

Change デプロイメント to デプロイ for consistency
Update レスポンシブ対応 to レスポンシブデザイン
```

### 変更理由の明記
```
feat(article:docker-guide): add troubleshooting section

Add solutions for common permission errors and port conflicts
based on frequent reader questions. Include actual error examples.
```

## 公開制御

### 記事の公開・非公開
```
chore(article:docker-guide): publish containerization guide

chore(article:k8s-intro): set to draft status
```

### 公開予約・設定変更
```
chore(article:docker-guide): schedule publication for 2024-12-01

chore(article): update topics and emoji for multiple articles
```

## 破壊的変更

### 記事構造の大幅変更
```
feat(article)!: restructure kubernetes series organization

BREAKING CHANGE: Split article into 3-part series.
Existing bookmark URLs may be affected.
```

### 本の価格・構成変更
```
feat(book:k8s-guide)!: change pricing model to free

BREAKING CHANGE: Convert from paid to free book.
Gradual implementation considering existing purchasers.
```
