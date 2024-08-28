.DEFAULT_GOAL:=help

PATH     := $(PATH):$(PWD)/bin
OS       := $(shell uname -s | tr '[:upper:]' '[:lower:]' | sed 's/darwin/apple-darwin/' | sed 's/linux/linux-gnu/')
ARCH     := $(shell uname -m)

.POSIX:

.PHONY: run-all
run-all: checkenv
	$(info Make: Run All)
	@for item in `ls -d core/??-* | sort -n`; do terraform -chdir="$$item" init -reconfigure -upgrade -backend-config=./../../envs/$(env)/backend.hcl ; terraform -chdir="$$item" apply -auto-approve -var-file=envs/$(env)/terraform.tfvars -var-file=./../../envs/$(env)/terraform.tfvars; done

.PHONY: destroy-all
destroy-all: checkenv
	$(info Make: Destroy All)
	@for item in `ls -d core/??-* | sort -nr`; do terraform -chdir="$$item" init -reconfigure -upgrade -backend-config=./../../envs/$(env)/backend.hcl ; terraform -chdir="$$item" destroy -auto-approve -var-file=envs/$(env)/terraform.tfvars -var-file=./../../envs/$(env)/terraform.tfvars; done

.PHONY: help
help:  ## Display this help
	@awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m\033[0m\n\nTargets:\n"} /^[a-zA-Z_-]+:.*?##/ { printf "  \033[36m%-10s\033[0m %s\n", $$1, $$2 }' $(MAKEFILE_LIST)

.PHONY: init
init: checkenv checkitem ## Initialize Terraform
	$(info Make: Init Item $(item))
	@terraform -chdir=core/$(item) init -reconfigure -upgrade -backend-config=./../../envs/$(env)/backend.hcl

.PHONY: validate
validate: checkitem ## Validate Terraform
	$(info Make: Validate Terraform Code and Format)
	@terraform -chdir=core/$(item) validate -no-color
	@terraform -chdir=core/$(item) fmt --check

.PHONY: plan
plan: checkenv checkitem init ## Plan Terraform
	$(info Make: Plan Item $(item))
	@terraform -chdir=core/$(item) plan -var-file=envs/$(env)/terraform.tfvars -var-file=./../../envs/$(env)/terraform.tfvars

.PHONY: apply
apply: checkenv checkitem init ## Apply Terraform
	$(info Make: Apply Item $(item))
	@terraform -chdir=core/$(item) apply -var-file=envs/$(env)/terraform.tfvars -var-file=./../../envs/$(env)/terraform.tfvars

.PHONY: destroy
destroy: checkenv checkitem init ## Destroy Terraform
	$(info Make: Destroy Item $(item))
	@terraform -chdir=core/$(item) destroy -var-file=envs/$(env)/terraform.tfvars -var-file=./../../envs/$(env)/terraform.tfvars

.PHONY: clean
clean: ## Cleanup the project folders with git clean
	$(info Make: Cleaning up things)
	@git clean -dfx

.PHONY: checkenv
checkenv:
	$(info Make: Check env variable)
	@[ "${env}" ] || ( echo "\033[36menv\033[0m is not set"; exit 1 )

.PHONY: checkitem
checkitem:
	$(info Make: Check item variable)
	@[ "${item}" ] || ( echo "\033[36mitem\033[0m is not set"; exit 1 )
