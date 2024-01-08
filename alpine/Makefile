REPO=ghcr.io/wagoodman/test-ctr-images
NAME=alpine
FULL_REPO=$(REPO)/$(NAME)
IMAGE=$(FULL_REPO):3.13.12
VEX_DIR=vexdocs
VEX_DOC=vex.json

all: image

## Commands to build and push assets

.PHONY: digest-ref
digest-ref:
	@docker image inspect --format='{{range .RepoDigests}}{{.}}{{end}}' $(IMAGE)

.PHONY: purl-version
purl-version:
	@echo $(shell make digest-ref) | sed 's|.*@||'

.PHONY: purl
purl:
	@echo "pkg:oci/$(NAME)@$(shell make purl-version)?repository_url=$(REPO)"

.PHONY: image
image:
	docker pull $(IMAGE) || (docker build -t $(IMAGE) . && docker push $(IMAGE))

.PHONY: vex
vex:
	mkdir -p $(VEX_DIR)

	vexctl create \
		--product "pkg:apk/alpine/busybox@1.32.1-r9?arch=x86_64" \
		--vuln="CVE-2022-48174"  \
		--status="not_affected"  \
		--impact-statement="omitted for testing purposes"  \
		--justification="vulnerable_code_not_in_execute_path"  \
		--author="Alex Goodman" > $(VEX_DIR)/CVE-2022-48174.json

	vexctl create \
		--product "pkg:apk/alpine/zlib@1.2.12-r3?arch=x86_64" \
		--vuln="CVE-2023-45853"  \
		--status="not_affected"  \
		--impact-statement="omitted for testing purposes"  \
		--justification="inline_mitigations_already_exist"  \
		--author="Alex Goodman" > $(VEX_DIR)/CVE-2023-45853.json

	vexctl merge $(VEX_DIR)/*.json > $(VEX_DOC)

## General commands to help

.PHONY: vex-attest
vex-attest:
	vexctl attest --attach --sign $(VEX_DOC) $(shell make purl)

.PHONY: oci-assets
oci-assets:
	cosign tree $(shell make digest-ref)

.PHONY: oci-explore
oci-explore:
	open https://oci.dag.dev/?repo=$(shell echo $(FULL_REPO) | sed 's|/|%2F|g')

.PHONY: grype
grype:
	grype $(shell make digest-ref) -o table -o json=grype.json
