# Current version
# VERSION ?= 1.0.4

.DEFAULT_GOAL:=help

PATH     := $(PATH):$(PWD)/bin
OS       := $(shell uname -s | tr '[:upper:]' '[:lower:]' | sed 's/darwin/apple-darwin/' | sed 's/linux/linux-gnu/')
ARCH     := $(shell uname -m)

.PHONY: help all hello

help:  ## Display this help
		$(info Make: $@)
		$(info )
		@awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m\033[0m\n\nTargets:\n"} /^[a-zA-Z_-]+:.*?##/ { printf "  \033[36m%-10s\033[0m %s\n", $$1, $$2 }' $(MAKEFILE_LIST)

all: hello

hello: ## Hello World
		$(info Make: $@)
		$(info )
		@echo "Hello, World"
