# Current version
# VERSION ?= 1.0.3

.DEFAULT_GOAL:=help

PATH  := $(PATH):$(PWD)/bin
OS    = $(shell uname -s | tr '[:upper:]' '[:lower:]')
ARCH  = $(shell uname -m | sed 's/x86_64/amd64/')
OSOPER   = $(shell uname -s | tr '[:upper:]' '[:lower:]' | sed 's/darwin/apple-darwin/' | sed 's/linux/linux-gnu/')
ARCHOPER = $(shell uname -m )

.PHONY: help requirements validate security lint providers apply

help:  ## Display this help

		## ----------------------------------------------------------------------
		#
		# Run in the following order:
		# make env=<env> dir=bootstrap apply
		# make env=<env> dir=network apply
		# make env=<env> dir=<iam>
		# make env=<env> dir=<component>
		#
		## ----------------------------------------------------------------------

		@awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m\033[0m\n\nTargets:\n"} /^[a-zA-Z_-]+:.*?##/ { printf "  \033[36m%-10s\033[0m %s\n", $$1, $$2 }' $(MAKEFILE_LIST)

all: validate security lint

.PHONY: requirements
requirements: ## Install Python requirements
		$(info Make: Python requirements)
		pip install --upgrade -r requirements.txt

.PHONY: validate
validate: ## Validate Code
		$(info Make: Validate Format and Code)
		terraform init -plugin-dir="plugins"
		terraform fmt --check
		terraform validate -no-color

.PHONY: security
security: ## Check Terraform Security
		$(info Make: Check Security with TFSec)
		$$PWD/bin/tfsec --format JUnit > ./reports/tfsec-report.xml
		$$PWD/bin/tfsec

.PHONY: lint
lint: ## Terraform Static Lint
		$(info Make: Static Lint Check)
		pre-commit run --all-files

.PHONY: init
init: check ## Initialize Terraform
		$(info Make: Terraform init)
ifeq ($(reg), eu-central-1)
		terraform -chdir=terraform/$(dir) init -upgrade \
		-backend-config="bucket=zih-zggl-terraform-$(env)-ec-s3" \
		-backend-config="dynamodb_table=zih-zggl-terraform-$(env)-ec-dyn" \
		-backend-config="region=$(reg)" \
		-plugin-dir="plugins"
else ifeq ($(reg), eu-west-1)
		terraform -chdir=terraform/$(dir) init -upgrade \
		-backend-config="bucket=zih-zggl-terraform-$(env)-ew-s3" \
		-backend-config="dynamodb_table=zih-zggl-terraform-$(env)-ew-dyn" \
		-backend-config="region=$(reg)" \
		-plugin-dir="plugins"
else
		$(info No region set or region not supported)
		$(info Supported regions are eu-central-1 and eu-west-1)
endif

.PHONY: plugins
plugins: modules ## Install plugins Locally
		$(info Make: Static Lint Check)
		terraform -chdir=terraform/$(dir) init -input=false -backend=false -upgrade
		mkdir -p terraform/$(dir)/plugins
		terraform -chdir=terraform/$(dir) providers mirror plugins

.PHONY: check
check: ## Check Variables
		$(info Make: Check Variables)
		@[ "${env}" ] || ( echo "\033[36menv\033[0m is not set"; exit 1 )
		@[ "${dir}" ] || ( echo "\033[36mdir\033[0m is not set"; exit 1 )
		@[ "${reg}" ] || ( echo "\033[36mreg\033[0m is not set"; exit 1 )

.PHONY: clean
clean: check ## Clean Stuff
		$(info Make: Clean Stuff)
		rm -rf terraform/$(dir)/.terraform*
		rm -rf terraform/$(dir)/main_override.tf

.PHONY: modules
modules: ## Define Module source
		$(info Make: Define modules)
ifeq ($(mod), local)
		@echo "Using local modules"
		cp terraform/$(dir)/override/*.tf terraform/$(dir)/
endif

.PHONY: run
apply: modules init ## Create Infrastructure - Run with 'make env=poc dir=network reg=eu-central-1 apply'
		$(info Make: Run)
		terraform -chdir=terraform/$(dir) apply -var-file="env/$(env).tfvars" -auto-approve
