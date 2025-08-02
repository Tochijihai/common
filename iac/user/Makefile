# User Microservices IaC管理用Makefile

# 環境変数
STACK_NAME ?= user-microservices
ENVIRONMENT ?= dev
GITHUB_REPO ?= https://github.com/your-username/tochijihai
AWS_REGION ?= ap-northeast-1

# AWS認証情報（実際の値に置き換えてください）
AWS_ACCESS_KEY_ID ?= AKXXXXXXXXXXXXX
AWS_SECRET_ACCESS_KEY ?= XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# SAMビルド
build:
	@echo "==> User Microservices SAM ビルド開始"
	sam build -t template.yaml

# SAMデプロイ（初回）
deploy-guided: build
	@echo "==> User Microservices SAM デプロイ（初回設定）"
	sam deploy --guided \
		--parameter-overrides \
			Environment=$(ENVIRONMENT) \
			GitHubRepo=$(GITHUB_REPO)

# SAMデプロイ（通常）
deploy: build
	@echo "==> User Microservices SAM デプロイ"
	AWS_ACCESS_KEY_ID="$(AWS_ACCESS_KEY_ID)" \
	AWS_SECRET_ACCESS_KEY="$(AWS_SECRET_ACCESS_KEY)" \
	sam deploy \
		--stack-name $(STACK_NAME)-$(ENVIRONMENT) \
		--parameter-overrides \
			Environment=$(ENVIRONMENT) \
			GitHubRepo=$(GITHUB_REPO) \
		--capabilities CAPABILITY_IAM \
		--region $(AWS_REGION)

# 開発環境デプロイ
deploy-dev:
	@$(MAKE) deploy ENVIRONMENT=dev

# ステージング環境デプロイ
deploy-staging:
	@$(MAKE) deploy ENVIRONMENT=staging

# 本番環境デプロイ
deploy-prod:
	@$(MAKE) deploy ENVIRONMENT=prod

# SAMローカルテスト
local:
	@echo "==> User Microservices ローカルテスト開始"
	sam local start-api -t template.yaml

# スタック削除
delete:
	@echo "==> User Microservices スタック削除: $(STACK_NAME)-$(ENVIRONMENT)"
	AWS_ACCESS_KEY_ID="$(AWS_ACCESS_KEY_ID)" \
	AWS_SECRET_ACCESS_KEY="$(AWS_SECRET_ACCESS_KEY)" \
	sam delete --stack-name $(STACK_NAME)-$(ENVIRONMENT) --region $(AWS_REGION)

# 開発環境削除
delete-dev:
	@$(MAKE) delete ENVIRONMENT=dev

# ステージング環境削除
delete-staging:
	@$(MAKE) delete ENVIRONMENT=staging

# 本番環境削除
delete-prod:
	@$(MAKE) delete ENVIRONMENT=prod

# スタック情報表示
info:
	@echo "==> User Microservices スタック情報: $(STACK_NAME)-$(ENVIRONMENT)"
	aws cloudformation describe-stacks \
		--stack-name $(STACK_NAME)-$(ENVIRONMENT) \
		--region $(AWS_REGION) \
		--query 'Stacks[0].Outputs'

# APIエンドポイントテスト
test-api:
	@echo "==> User Microservices API エンドポイントを取得中..."
	$(eval API_URL := $(shell aws cloudformation describe-stacks \
		--stack-name $(STACK_NAME)-$(ENVIRONMENT) \
		--region $(AWS_REGION) \
		--query 'Stacks[0].Outputs[?OutputKey==`UserServicesApiUrl`].OutputValue' \
		--output text))
	@echo "API URL: $(API_URL)"
	@echo "\n==> User Chat AI テスト"
	@echo "ルートエンドポイント:"
	curl $(API_URL)/user-chat/ || echo "Root endpoint test"
	@echo "\nヘルスチェック:"
	curl $(API_URL)/user-chat/health || echo "Health check test"
	@echo "\nチャット機能:"
	curl -X POST $(API_URL)/user-chat/chat \
		-H "Content-Type: application/json" \
		-d '{"messages": [{"role": "user", "content": "こんにちは"}]}' || echo "Chat test"
	@echo "\n==> User Backend テスト"
	@echo "ヘルスチェック:"
	curl $(API_URL)/user/health || echo "User Backend health check (未実装)"
	@echo "\nユーザー情報:"
	curl $(API_URL)/user/profile || echo "User profile endpoint (未実装)"

# 既存のuser-chat-ai単体スタックを削除
cleanup-old-stack:
	@echo "==> 既存のuser-chat-ai-api スタックを削除"
	AWS_ACCESS_KEY_ID="$(AWS_ACCESS_KEY_ID)" \
	AWS_SECRET_ACCESS_KEY="$(AWS_SECRET_ACCESS_KEY)" \
	sam delete --stack-name user-chat-ai-api --region $(AWS_REGION) || echo "Stack not found or already deleted"

# ヘルプ
help:
	@echo "User Microservices 利用可能なコマンド:"
	@echo "  build           - SAMビルド"
	@echo "  deploy-guided   - 初回デプロイ（設定付き）"
	@echo "  deploy          - 通常デプロイ"
	@echo "  deploy-dev      - 開発環境デプロイ"
	@echo "  deploy-staging  - ステージング環境デプロイ"
	@echo "  deploy-prod     - 本番環境デプロイ"
	@echo "  local           - ローカルテスト"
	@echo "  delete          - スタック削除"
	@echo "  delete-dev      - 開発環境削除"
	@echo "  delete-staging  - ステージング環境削除"
	@echo "  delete-prod     - 本番環境削除"
	@echo "  info            - スタック情報表示"
	@echo "  test-api        - APIエンドポイントテスト"
	@echo "  cleanup-old-stack - 既存のuser-chat-ai-api スタック削除"
	@echo "  help            - このヘルプを表示"
	@echo ""
	@echo "環境変数:"
	@echo "  ENVIRONMENT     - 環境名 (dev/staging/prod)"
	@echo "  GITHUB_REPO     - GitHubリポジトリURL"
	@echo "  AWS_REGION      - AWSリージョン"

.PHONY: build deploy-guided deploy deploy-dev deploy-staging deploy-prod local delete delete-dev delete-staging delete-prod info test-api cleanup-old-stack help
