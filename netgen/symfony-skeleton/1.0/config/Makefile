APP_ENV = dev
PHP_VERSION = php8.2
PHP_RUN = /usr/bin/env $(PHP_VERSION)
COMPOSER_PATH = /usr/local/bin/composer2
ifeq ("$(wildcard $(COMPOSER_PATH))","")
	COMPOSER_PATH = /usr/local/bin/composer
endif
COMPOSER_RUN = $(PHP_RUN) $(COMPOSER_PATH)
CACHE_POOL = cache.redis
STASH_HASH := $(shell git stash create)

.PHONY: help
help: ## List of all available commands
	@awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z_-]+:.*?## / {printf "\033[36m%-15s\033[0m %s\n", $$1, $$2}' $(MAKEFILE_LIST)

.DEFAULT_GOAL := help

.PHONY: php-version
php-version: ## See PHP version needed for this project
	@echo $(PHP_VERSION)

COMPOSER_INSTALL_PARAMETERS =
ifeq ($(APP_ENV), prod)
	COMPOSER_INSTALL_PARAMETERS = --no-dev -o
endif

.PHONY: vendor
vendor: ## Run composer install
	$(COMPOSER_RUN) install $(COMPOSER_INSTALL_PARAMETERS)

.PHONY: assets
.ONESHELL:
assets: ## Build frontend assets for DEV environment
	. ${NVM_DIR}/nvm.sh && nvm use || nvm install $(cat .nvmrc)
	yarn install
	yarn build:dev

.PHONY: assets-prod
.ONESHELL:
assets-prod: ## Build frontend assets for PROD environment
	. ${NVM_DIR}/nvm.sh && nvm use || nvm install $(cat .nvmrc)
	yarn install
	yarn build:prod

.PHONY: assets-watch
.ONESHELL:
assets-watch: ## Watch frontend assets (during development)
	. ${NVM_DIR}/nvm.sh && nvm use || nvm install $(cat .nvmrc)
	yarn install
	yarn watch

.PHONY: clear-cache
clear-cache: ## Clear caches for specified environment (default: APP_ENV=dev)
	$(PHP_RUN) bin/console cache:clear --env=$(APP_ENV)

.PHONY: clear-all-cache
clear-all-cache: ## Clear all caches for specified environment (including eg. Redis) (default: APP_ENV=dev)
	@$(MAKE) -s clear-cache
	$(PHP_RUN) bin/console cache:pool:clear $(CACHE_POOL) --env=$(APP_ENV)

.PHONY: migrations
migrations: ## Run Doctrine migrations for specified environment (default: APP_ENV=dev)
	$(PHP_RUN) bin/console doctrine:migration:migrate --allow-no-migration --env=$(APP_ENV)

.PHONY: build
build: ## Build the project (install vendor, migrations, build assets, clear cache) for specified environment (default: APP_ENV=dev)
	@$(MAKE) -s vendor
	@$(MAKE) -s migrations
ifeq ($(APP_ENV), prod)
	$(MAKE) -s assets-prod
else
	$(MAKE) -s assets
endif
	@$(MAKE) -s clear-cache

.PHONY: update-code
update-code: ## Pull the latest code from the repository (on the current branch)
ifeq ($(STASH_HASH),)
	git pull --rebase
else
	git stash && git pull --rebase && git stash pop
endif

.PHONY: refresh
refresh: ## Fetch latest changes and build the project for specified environment (default: APP_ENV=dev)
	@$(MAKE) -s update-code
	@$(MAKE) -s build
