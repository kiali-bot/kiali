CONSOLE_LOCAL_DIR ?= "../kiali-ui"

BACKEND_VERSION ?= $(shell sed -rn 's/^VERSION \?= v(.*)/\1/p' Makefile)
BACKEND_BUMPED_VERSION ?= $(shell semver bump minor $(BACKEND_VERSION))

BACKEND_GITHUB_URI ?= "git@github.com:kiali/kiali.git"
BACKEND_FORK_URI ?= $(shell git config --get remote.origin.url)
BACKEND_PULL_URI ?= "https://api.github.com/repos/kiali/kiali/pulls"

BUMP_BRANCH_ID ?= $(BUILD_TAG)

.PHONY: backend-build-release backend-test backend-push-docker
.PHONY: backend-push-version-tag backend-create-pr-next-version
.PHONY: all release

# Swagger not yet ready
# backend-swagger:
#	$(MAKE) make swagger-install"

all:
	$(error You must explicitly specify a target)

backend-build-release:
	sed -i -r 's/^VERSION \?= (.*)-SNAPSHOT/VERSION \?= \1/' Makefile
	$(MAKE) clean build

backend-test:
	$(MAKE) test-race

backend-push-docker:
ifdef DOCKER_USER
ifdef DOCKER_PASSWORD
	docker login -u "$(DOCKER_USER)" -p "$(DOCKER_PASSWORD)"
endif
endif
	DOCKER_VERSION="v$(BACKEND_VERSION)" \
	  CONSOLE_VERSION="local" \
	  CONSOLE_LOCAL_DIR="$(CONSOLE_LOCAL_DIR)" \
	  make docker-build docker-push

backend-push-version-tag:
	sed -i "s/^CONSOLE_VERSION ?=.*/CONSOLE_VERSION ?= $$(cat _output/docker/console/version.txt)/" Makefile
	git add Makefile
	git commit -m "Release $(BACKEND_VERSION)"
	git push $(BACKEND_GITHUB_URI) $$(git rev-parse HEAD):refs/tags/v$(BACKEND_VERSION)

backend-create-pr-next-version:
	sed -i "s/^CONSOLE_VERSION ?=.*/CONSOLE_VERSION ?= latest/" Makefile
	sed -i -r "s/^VERSION \?= (.*)/VERSION \?= v$(BACKEND_BUMPED_VERSION)-SNAPSHOT/" Makefile
	git add Makefile
	git commit -m "Prepare for next version"
	git push $(BACKEND_FORK_URI) $$(git rev-parse HEAD):refs/heads/$(BUMP_BRANCH_ID)
ifdef GH_TOKEN
	curl -H "Authorization: token $(GH_TOKEN)" \
	  -H "Content-Type: application/json" \
	  -d '{"title": "Prepare for next version", "head": "kiali-bot:$(BUMP_BRANCH_ID)", "base": "master"}' \
	  -X POST $(BACKEND_PULL_URI)
endif

release: backend-build-release backend-test backend-push-docker \
  backend-push-version-tag backend-create-pr-next-version
