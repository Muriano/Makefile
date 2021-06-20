.PHONY: all test

# CONFIG ---------------------------------------------------------------------------------------------------------------
ifneq (,$(findstring xterm,${TERM}))
    BLACK   := $(shell tput -Txterm setaf 0)
    RED     := $(shell tput -Txterm setaf 1)
    GREEN   := $(shell tput -Txterm setaf 2)
    YELLOW  := $(shell tput -Txterm setaf 3)
    BLUE    := $(shell tput -Txterm setaf 4)
    MAGENTA := $(shell tput -Txterm setaf 5)
    CYAN    := $(shell tput -Txterm setaf 6)
    WHITE   := $(shell tput -Txterm setaf 7)
    RESET   := $(shell tput -Txterm sgr0)
else
    BLACK   := ""
    RED     := ""
    GREEN   := ""
    YELLOW  := ""
    BLUE    := ""
    MAGENTA := ""
    CYAN    := ""
    WHITE   := ""
    RESET   := ""
endif

COMMAND_COLOR := $(GREEN)
HELP_COLOR := $(BLUE)
COMMAND_HELP_SEPARATOR_SIZE := 20

IMAGE_NAME=jamuriano/project-name-goes-here
IMAGE_TAG_BASE=base
IMAGE_TAG_DEV=dev

# DEFAULT COMMANDS -----------------------------------------------------------------------------------------------------
all: help

help: ## Shows a list with all available commands in this Makefile
	@echo "${CYAN}╔══════════════════════════════════════════════════════════════════════════════╗${RESET}"
	@echo "${CYAN}║                           .: AVAILABLE COMMANDS :.                           ║${RESET}"
	@echo "${CYAN}╚══════════════════════════════════════════════════════════════════════════════╝${RESET}"
	@echo ""
	@grep -E '^[a-zA-Z_0-9%-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "${COMMAND_COLOR}%-${COMMAND_HELP_SEPARATOR_SIZE}s${RESET} ${HELP_COLOR}%s${RESET}\n", $$1, $$2}'
	@echo ""

check-deps:
	docker -v

# BUILD COMMANDS -------------------------------------------------------------------------------------------------------

build: check-deps build-container-dev build-composer-install-dev ## Builds project dependencies

build-container-dev:
	docker build --target $(IMAGE_TAG_DEV) -t $(IMAGE_NAME):$(IMAGE_TAG_DEV) ./docker

build-composer-install-dev:
	docker run --rm -v ${PWD}:/app -w /app $(IMAGE_NAME):$(IMAGE_TAG_DEV) composer install --verbose

composer-install: build-composer-install-dev ## Installs project dependencies using composer

composer-update: ## Updates project dependencies using composer
	docker run --rm -v ${PWD}:/app -w /app $(IMAGE_NAME):$(IMAGE_TAG_DEV) composer update --verbose

composer-require: ## Adds new production project dependencies using composer
	docker run --rm -ti -v ${PWD}:/app -w /app $(IMAGE_NAME):$(IMAGE_TAG_DEV) composer require --verbose

composer-require-dev: ## Adds new development project dependencies using composer
	docker run --rm -ti -v ${PWD}:/app -w /app $(IMAGE_NAME):$(IMAGE_TAG_DEV) composer require --dev --verbose

# SYMFONY COMMANDS -----------------------------------------------------------------------------------------------------
symfony-console: ## Executes Symfony's console. To provide a command or flags use: make symfony console COMMAND=your:command --your=flag
	docker run --rm -ti -v ${PWD}:/app -w /app $(IMAGE_NAME):$(IMAGE_TAG_DEV) php /app/bin/console ${COMMAND}

# TESTING COMMANDS -----------------------------------------------------------------------------------------------------
test: ## Executes project's tests using phpunit
	docker run --rm -v ${PWD}:/app -w /app $(IMAGE_NAME):$(IMAGE_TAG_DEV) php ./vendor/bin/phpunit --no-coverage --color=always

test-coverage: ## Executes project's test and coverage generation using phpunit
	docker run --rm -v ${PWD}:/app -w /app -e "XDEBUG_MODE=coverage" $(IMAGE_NAME):$(IMAGE_TAG_DEV) php ./vendor/bin/phpunit --color=always

test-mutant: ## Executes project's mutant testing using Infection
	docker run --rm -v ${PWD}:/app -w /app -e "XDEBUG_MODE=coverage" $(IMAGE_NAME):$(IMAGE_TAG_DEV) ./vendor/bin/infection

test-verbose: ## Executes project's tests using phpunit in verbose mode
	docker run --rm -v ${PWD}:/app -w /app $(IMAGE_NAME):$(IMAGE_TAG_DEV) php ./vendor/bin/phpunit --no-coverage --color=always --verbose

test-group: ## Executes project's tests tagged with a given group. To provide a group use: make test-group GROUP=your-group
	docker run --rm -v ${PWD}:/app -w /app $(IMAGE_NAME):$(IMAGE_TAG_DEV) php ./vendor/bin/phpunit --no-coverage --color=always --group ${GROUP}

# OTHER COMMANDS & UTILS -----------------------------------------------------------------------------------------------
git-describe: ## Shows last version tag in project
	@git describe --abbrev=0 || echo "No version tagged yet"

git-prune-branches: ## Purges git branches that were merged with remote repository
	@git fetch --all -p
	@git branch --merged | grep -v main |grep -v develop | grep -v master>/tmp/merged-branches && vi /tmp/merged-branches && xargs git branch -d </tmp/merged-branches || echo "There are no branches to be deleted"
