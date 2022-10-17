---
layout: "../../layouts/docs/Layout.astro"
title: "Development"
index: 1
---

Here you can find development notes intended for maintainers and guidance for new contributors.

## Build Kairos

Kairos uses [earthly](https://earthly.dev/) as a build system instead of Makefiles.

To build Kairos you need only Docker installed locally, and there is a convenience script in the root of the repository (`earthly.sh`) which wraps `earthly` inside Docker to avoid to install locally which can be used instead of `earthly` (e.g. `./earthly.sh +iso ...`).

To build a Kairos ISO, you need to specify the flavor. For example, to build Kairos Alpine:

```
earthly -P +iso --FLAVOR=alpine
```

This will build a container image from scratch and create an ISO which is ready to be booted.

Note earthly targets are prefixed with `+` while variables are passed as flags.

### Adding flavors

Every source image used as a flavor is inside the `images` folder in the top-level directory. Any Dockerfile have the extension corresponding to the flavor which can be used as an argument for earthly builds (you will find a `Dockerfile.alpine` that will be used by our `earthly -P +iso --FLAVOR=alpine` above).

To add a flavor is enough to create a Dockerfile corresponding to the flavor and check if any specific setting is required for it in the `+framework` target.

Generally to add a flavor the image needs to have installed:

- An init system (systemd or openRC are supported)
- Kernel
- GRUB
- rsync

If you are building a flavor without Earthly, be sure to consume the packages from our repository to convert it to a Kairos-based version.

## New controllers

Kairos-io adopts [operator-sdk](https://github.com/operator-framework/operator-sdk). 

To install `operator-sdk` locally you can use the `kairos` repositories:

1. Install Luet:
   `curl https://luet.io/install.sh | sudo sh`
2. Enable the Kairos repository locally:
   `luet repo add kairos --url quay.io/kairos/packages --type docker`
3. Install operator-sdk:
   `luet install -y utils/operator-sdk`

### Create the controller

Create a directory and let's init our new project it with the operator-sdk:

```bash

$ mkdir kairos-controller-foo
$ cd kairos-controller-foo
$ operator-sdk init --domain kairos.io --repo github.com/kairos-io/kairos-controller-foo

```

### Create a resource

To create a resource boilerplate:

```
$ operator-sdk create api --group <groupname> --version v1alpha1 --kind <resource> --resource --controller
```

### Convert to a Helm chart

operator-sdk does not have direct support to render Helm charts (see [issue](https://github.com/operator-framework/operator-sdk/issues/4930)), we use [kubesplit](https://github.com/spectrocloud/kubesplit) to render Helm templates by piping kustomize manifests to it. `kubesplit` will split every resource and add a minimal `helm` templating logic, that will guide you into creating the Helm chart.

If you have already enabled the `kairos` repository locally, you can install `kubesplit` with:

```
$ luet install -y utils/kubesplit
```

### Test with Kind

Operator-sdk will generate a Makefile for the project. You can add the following and edit as needed to add kind targets:

```
CLUSTER_NAME?="kairos-controller-e2e"

kind-setup:
	kind create cluster --name ${CLUSTER_NAME} || true
	$(MAKE) kind-setup-image

kind-setup-image: docker-build
	kind load docker-image --name $(CLUSTER_NAME) ${IMG}

.PHONY: test_deps
test_deps:
	go install -mod=mod github.com/onsi/ginkgo/v2/ginkgo
	go install github.com/onsi/gomega/...

.PHONY: unit-tests
unit-tests: test_deps
	$(GINKGO) -r -v  --covermode=atomic --coverprofile=coverage.out -p -r ./pkg/...

e2e-tests:
	GINKGO=$(GINKGO) KUBE_VERSION=${KUBE_VERSION} $(ROOT_DIR)/script/test.sh

kind-e2e-tests: ginkgo kind-setup install undeploy deploy e2e-tests
```