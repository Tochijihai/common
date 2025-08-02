# User Microservices IaC

このディレクトリには、User Microservices（user-chat-ai + user-backend）を統合する API Gateway の IaC（Infrastructure as Code）が含まれています。

## アーキテクチャ

```
API Gateway (UserServicesAPI)
├── /user-chat/* → User Chat AI Lambda (Python)
└── /user/*      → User Backend Lambda (Go)
```

## 構成要素

- **User Chat AI Lambda**: Python FastAPI アプリケーション（Bedrock Claude 3 Haiku 使用）
- **User Backend Lambda**: Go アプリケーション
- **API Gateway**: 両方の Lambda 関数を統合する HTTP API

## 使用方法

### 1. 初回デプロイ

```bash
cd common/iac/user
make deploy-guided
```

### 2. 通常のデプロイ

```bash
cd common/iac/user
make deploy
```

### 3. 環境別デプロイ

```bash
# 開発環境
make deploy-dev

# ステージング環境
make deploy-staging

# 本番環境
make deploy-prod
```

### 4. ローカルテスト

```bash
make local
```

### 5. API テスト

```bash
make test-api
```

### 6. スタック削除

```bash
make delete
```

## エンドポイント

デプロイ後、以下のエンドポイントが利用可能になります：

### User Chat AI

- `GET /user-chat/` - ルートエンドポイント
- `GET /user-chat/health` - ヘルスチェック
- `POST /user-chat/chat` - チャット機能

### User Backend

- `GET /user/health` - ヘルスチェック（未実装）
- `GET /user/profile` - ユーザープロフィール（未実装）
- `POST /user/test` - テストエンドポイント

## 環境変数

Makefile で以下の環境変数を設定できます：

- `ENVIRONMENT`: 環境名 (dev/staging/prod)
- `GITHUB_REPO`: GitHub リポジトリ URL
- `AWS_REGION`: AWS リージョン
- `AWS_ACCESS_KEY_ID`: AWS 認証情報
- `AWS_SECRET_ACCESS_KEY`: AWS 認証情報

## 前提条件

### User Chat AI

- Python 3.11
- Poetry
- AWS Bedrock Claude 3 Haiku へのアクセス権限

### User Backend

- Go 1.x
- Docker（ビルド用）

### 共通

- AWS SAM CLI
- AWS CLI
- 適切な AWS 認証情報

## トラブルシューティング

### 既存スタックとの競合

既存の`user-chat-ai-api`スタックがある場合は、以下のコマンドで削除してください：

```bash
make cleanup-old-stack
```

### Bedrock アクセス権限エラー

AWS Bedrock でモデルへのアクセス権限を確認してください：

1. AWS Console → Bedrock → Model access
2. Claude 3 Haiku モデルへのアクセスを有効化

### Go Lambda ビルドエラー

user-backend ディレクトリで以下を実行してください：

```bash
cd ../../user-backend
make build
```

## ヘルプ

利用可能なコマンドの一覧を表示：

```bash
make help
```
