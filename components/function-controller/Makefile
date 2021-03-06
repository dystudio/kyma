PKG = github.com/kyma-project/kyma/components/function-controller

# Image URL to use all building/pushing image targets
APP_NAME = function-controller
APP_PATH = components/function-controller
BUILDPACK = eu.gcr.io/kyma-project/test-infra/buildpack-golang:go1.12
SCRIPTS_DIR = ../../common/makefiles

# COMPONENT_DIR is a local path to component
include $(SCRIPTS_DIR)/generic-make-go.mk

ENTRYPOINT := ./cmd/manager/main.go

all: test manager

# Run tests
# Default is 20s - available since controller-runtime 0.1.5
test-local: export KUBEBUILDER_CONTROLPLANE_START_TIMEOUT = 2m
# Default is 20s - available since controller-runtime 0.1.5
test-local: export KUBEBUILDER_CONTROLPLANE_STOP_TIMEOUT = 2m
test-local: generate-local fmt-local vet-local
	go test -race -v ./pkg/... ./cmd/... -coverprofile cover.out
	@echo "Total test coverage: $$(go tool cover -func=cover.out | grep total | awk '{print $$3}')"

.PHONY: test-docker test
test:
	@echo make test
	@docker run $(DOCKER_INTERACTIVE) \
		-v $(COMPONENT_DIR):$(WORKSPACE_COMPONENT_DIR):delegated \
		$(DOCKER_CREATE_OPTS) make test-docker

test-docker: export KUBEBUILDER_CONTROLPLANE_START_TIMEOUT = 2m
test-docker: export KUBEBUILDER_CONTROLPLANE_STOP_TIMEOUT = 2m
test-docker:
	curl -L https://github.com/kubernetes-sigs/kubebuilder/releases/download/v1.0.8/kubebuilder_1.0.8_linux_amd64.tar.gz -o /tmp/kubebuilder.tar.gz
	tar -C /tmp -xzf /tmp/kubebuilder.tar.gz
	mv /tmp/kubebuilder_1.0.8_linux_amd64 /usr/local/kubebuilder
	go test -race -v ./pkg/... ./cmd/... -coverprofile cover.out
	@echo "Total test coverage: $$(go tool cover -func=cover.out | grep total | awk '{print $$3}')"


# Generate code
generate-local:
	go generate ./pkg/... ./cmd/...

qt:
	go test -v ./pkg/... ./cmd/... -coverprofile cover.out
	@echo "Total test coverage: $$(go tool cover -func=cover.out | grep total | awk '{print $$3}')"

# Build manager binary
manager: generate-local fmt-local vet-local
	go build -o bin/manager $(PKG)/cmd/manager

# NOTE: does not work anymore until https://github.com/kubernetes-sigs/kubebuilder/issues/400 is implemented
# Run against the configured Kubernetes cluster in ~/.kube/config
run: generate-local fmt-local vet-local
	go run ./cmd/manager/main.go

# Install CRDs into a cluster
install:
	kubectl apply -f config/crds/

# Uninstall CRDs from cluster
uninstall:
	kubectl delete -f config/crds/

# Generate manifests e.g. CRD, RBAC etc.
manifests:
	go run vendor/sigs.k8s.io/controller-tools/cmd/controller-gen/main.go all

# Deploy controller in the configured Kubernetes cluster in ~/.kube/config

sedbackupflag = -i
ifeq ($(shell uname -s),Darwin)
	sedbackupflag +=
endif

deploy: manifests install bin/kustomize ko
	bin/kustomize build config/default | ko apply -f -

removewebhook:
	-kubectl delete mutatingwebhookconfigurations.admissionregistration.k8s.io function-webhook.serverless.kyma-project.io

undeploy: uninstall removewebhook bin/kustomize ko
	bin/kustomize build config/default | ko delete  -f -

# Build the docker image
.PHONY: docker-build
docker-build: resolve pull-licenses manifests test
	docker build . -t $(IMG)

# Push the docker image
.PHONY: docker-push
docker-push:
ifdef DOCKER_PUSH_REPOSITORY
	docker tag $(IMG) $(IMG-CI)
	docker push $(IMG-CI)
else
	docker push $(IMG)
endif

.PHONY: path-to-referenced-charts
path-to-referenced-charts:
	@echo "resources/function-controller"

# used by 'go install'
GOPATH ?= $(shell go env GOPATH)
export GOBIN := $(GOPATH)/src/$(PKG)/bin

.PHONY: bin/kustomize
bin/kustomize:
	@go install ./vendor/sigs.k8s.io/kustomize

.PHONY: ko
ko:
	@go get -u github.com/google/ko/cmd/ko



