BASE_DISTRO = "ubuntu"
BASE_VERSION = "14.04"
BASE_INIT = "/sbin/init"
ROLE_NAME = $(shell basename $${PWD%/*})
CONTAINER_NAME = "$(BASE_DISTRO)-$(BASE_VERSION)-$(ROLE_NAME)"
CONTAINER_ID_FILE = "$(BASE_DISTRO)-$(BASE_VERSION)-$(ROLE_NAME).cid"

# CircleCI uses Docker's lxc driver which doesn't support docker exec yet
# https://circleci.com/docs/docker/#docker-exec
ifndef CIRCLECI
	DOCKER_EXEC = docker exec --tty `cat $(CONTAINER_ID_FILE)` env TERM=xterm
else
	DOCKER_EXEC = sudo lxc-attach -n `cat $(CONTAINER_ID_FILE)` -- bash -c
	DOCKER_EXEC_PREFIX = """
	DOCKER_EXEC_SUFFIX = """
endif

.DEFAULT_GOAL := help
.PHONY: build clean help run shell test test-syntax test-execute test-idempotent test-serverspec

build: ## Build Docker image from repo Dockerfile
	@echo "Building docker container from repo Dockerfile..."
	@docker pull $(BASE_DISTRO):$(BASE_VERSION)
	@docker build --rm=true --file=Dockerfile.$(BASE_DISTRO)-$(BASE_VERSION) --tag=$(BASE_DISTRO)-$(BASE_VERSION):ansible .

clean: ## Remove Docker image and container id file
	@echo "Destroying container and cleaning up CID file..."
	@docker rm -f $(CONTAINER_NAME) || true
	@rm $(CONTAINER_ID_FILE) || true

run: clean ## Run fresh Docker container
	@echo "Starting a fresh container..."
	@docker run --detach \
		--name "$(CONTAINER_NAME)" \
		--volume="$${PWD%/*}":/etc/ansible/roles/role_under_test:ro \
		$(BASE_DISTRO)-$(BASE_VERSION):ansible \
		"$(BASE_INIT)" > $(CONTAINER_ID_FILE)

shell: ## Drop into container shell
	docker exec -it `cat $(CONTAINER_ID_FILE)` /bin/bash

test-syntax: ## Ansible syntax check on running container
	@echo "Running Ansible syntax check..."
	$(DOCKER_EXEC) $(DOCKER_EXEC_PREFIX)ansible-playbook /etc/ansible/roles/role_under_test/tests/test.yml --syntax-check$(DOCKER_EXEC_SUFFIX)

test: ## Execute all tests
	@make run
	@make test-syntax
	@make clean

help: ## Halp!
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
