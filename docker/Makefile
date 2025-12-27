USERNAME=RapidMark
REPO=_acc
TAG=dev
VERSION=2.58.02
PLATFORMS=linux/amd64,linux/arm64,linux/arm/v7
DATE := $(shell date -u +"%Y%m%dT%H%M%SZ")

build-server:
	docker buildx build -f Dockerfile -t ${USERNAME}/tdarr${REPO}:${TAG} --build-arg VERSION=$(VERSION) --build-arg MODULE=Tdarr_Server --build-arg BUILD_DATE=$(DATE) --platform ${PLATFORMS} --push .
build-node:
	docker buildx build -f Dockerfile -t ${USERNAME}/tdarr_node${REPO}:${TAG} --build-arg VERSION=$(VERSION) --build-arg MODULE=Tdarr_Node --build-arg BUILD_DATE=$(DATE) --platform ${PLATFORMS} --push .
