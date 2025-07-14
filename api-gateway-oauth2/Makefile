include ../.env
include .env
$(VERBOSE).SILENT:

VERSION=0.1.0
STACK_NAME=MyApi
RESOURCE_SERVER=https://example.com

.PHONY: help
help:  ## Display this help
	$(info CloudFormation Examples - API Gateway with OAuth2)
	awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m<target>\033[0m\n"} /^[a-zA-Z0-9_-]+:.*?##/ { printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2 } /^##@/ { printf "\n\033[1m%s\033[0m\n", substr($$0, 5) } ' $(MAKEFILE_LIST)

.PHONY: deploy
deploy: ## Deploy the CloudFormation template
	$(info Deploy the CloudFormation template)
	aws cloudformation deploy \
		--region $(AWS_REGION) \
		--profile $(AWS_PROFILE) \
		--template-file template.yaml \
		--stack-name $(STACK_NAME) \
		--capabilities CAPABILITY_NAMED_IAM \
		--no-fail-on-empty-changeset \
		--parameter-overrides Version=$(VERSION)

.PHONY: read-token
read-token: ## Request the OAuth2 token (read access) to be used in the API calls
	$(eval ENDPOINT=$(shell aws cloudformation describe-stacks \
		--region $(AWS_REGION) \
		--profile $(AWS_PROFILE) \
		--stack-name $(STACK_NAME) \
		--query "Stacks[0].Outputs[?OutputKey=='CognitoEndpoint'].OutputValue" \
		--output text))
	curl -X POST --basic --user $(CLIENT_ID):$(CLIENT_SECRET) \
       -d "grant_type=client_credentials&scope=$(RESOURCE_SERVER)/read" \
       $(ENDPOINT)/oauth2/token | jq -r '.access_token' > access_token

.PHONY: read-write-token
read-write-token: ## Request the OAuth2 token (read/write access) to be used in the API calls
	$(eval ENDPOINT=$(shell aws cloudformation describe-stacks \
		--region $(AWS_REGION) \
		--profile $(AWS_PROFILE) \
		--stack-name $(STACK_NAME) \
		--query "Stacks[0].Outputs[?OutputKey=='CognitoEndpoint'].OutputValue" \
		--output text))
	curl --X POST --basic --user $(CLIENT_ID):$(CLIENT_SECRET) \
       -d "grant_type=client_credentials&scope=$(RESOURCE_SERVER)/read $(RESOURCE_SERVER)/write" \
       $(ENDPOINT)/oauth2/token | jq -r '.access_token' > access_token


.PHONY: hello
hello: ## Call the API on the /hello path
	$(eval ENDPOINT=$(shell aws cloudformation describe-stacks \
		--region $(AWS_REGION) \
		--profile $(AWS_PROFILE) \
		--stack-name $(STACK_NAME) \
		--query "Stacks[0].Outputs[?OutputKey=='ApiEndpoint'].OutputValue" \
		--output text))
	$(eval TOKEN=$(shell cat access_token))
	curl $(ENDPOINT)/hello \
     -X GET \
     -H 'content-type: application/json' \
     -H "Authorization: Bearer $(TOKEN)"

.PHONY: post-hello
post-hello: ## Call the API on the /hello path
	$(eval ENDPOINT=$(shell aws cloudformation describe-stacks \
		--region $(AWS_REGION) \
		--profile $(AWS_PROFILE) \
		--stack-name $(STACK_NAME) \
		--query "Stacks[0].Outputs[?OutputKey=='ApiEndpoint'].OutputValue" \
		--output text))
	$(eval TOKEN=$(shell cat access_token))
	curl $(ENDPOINT)/hello \
     -X POST \
     -H 'content-type: application/json' \
     -H "Authorization: Bearer $(TOKEN)" \
     -d '{"message":"message from my machine"}'