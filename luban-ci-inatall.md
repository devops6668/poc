# Install Luban CI
# Prerequisite
## Change General Settings to match your environment
### Update path: luban-ci/Makefile.env
```
# Image Registry for Luban-ci
REGISTRY_SERVER := quay.io
REGISTRY_NAMESPACE := luban-ci
REGISTRY_EMAIL := devops@gmail.com
TAG_PREFIX := al2023

# Image Repository for Dagster/code 
QUAY_USERNAME := rke2+build
QUAY_PASSWORD := 9XLLQ9H1OAC770HCWBWIH2B4YI0K5E9VUD64T6NS5EOQCS7TZB4NW518SG1GQM3Y
HARBOR_SERVER := harbor.luban.paulhome.local
HARBOR_USERNAME := robot$luban-ci
HARBOR_PASSWORD := LPv85vJJuDL00YXcC39U6rr0vamwLvCq
HARBOR_RO_USERNAME := robot$deployer
HARBOR_RO_PASSWORD := SV56NR5AiPMp5FS8QyWHXCuO3jgOfd3G

# Kubenertes Setting
K8S_NAMESPACE := luban-ci
ARGOCD_NAMESPACE := argocd

# Git Repository Setting
AZURE_ORGANIZATION := data-service
AZURE_DEVOPS_TOKEN := BS2fz3TtPlMeZJh3G8WCtQGF4Np.................
GITHUB_ORGANIZATION := data-service
GITHUB_TOKEN := ghp_qHkS7aCV.........................
```
---


## Prepare secret
Create Azure PAT, ssh secret.
path: luban-ci/secrets
```
mkdir secrets
```
### Create Azure PAT
```
vi luban-ci/secrests/azure-creds.env

AZURE_DEVOPS_TOKEN=BS2fz3TtPlMeZJh3G8WCtQGF4NpHJ............................
AZURE_ORGANIZATION=deovps
```

### Azure SSH Keys (Required for kpack builds on Azure)**:
1. Generate an SSH key pair: `ssh-keygen -t rsa -b 4096 -f secrets/azure_id_rsa`
2. Add the public key (`secrets/azure_id_rsa.pub`) to your Azure DevOps user settings (SSH Public Keys).
3. Add Azure's host key to `secrets/known_hosts`:
```
ssh-keyscan -t rsa ssh.dev.azure.com > secrets/known_hosts
```
---
# 1. Create Secrets - please exclude github from the MakeFile if you don't have github
Apply all credentials to the Kubernetes cluster:
```
make secrets
```

---
# 2. Prepare Luban kpack image "require internet access, you may need to buld the "luban-base" outside of company"
Image:
- luban-base
- luban-run
- luban-build

path: luban-ci/
## Run the following cli to build images
```
make stack-build
```
## Run the follwing cli to push image to "Image Registry for Luban-ci", eg: quay.io
```
make stack-push
```
## Inside company
### Downlod the following image and add custome CA certificate
Image:
- luban-run
- luban-build
This 2 image is used by the kpack CRD
- ClusterStack.buildImage: rancher.harbor.local/luban-ci/luban-build@sha256:46d424ef824f19e628548c789ea191afb12a14c1b49cd1bc42bb63e6e913e951
- ClusterStack.runImage: rancher.harbor.local/luban-ci/luban-run@sha256:f7c725b92384c44f7c4a50967d222a3cc2d85ccf194d7d0232f6a5d38cc21c6c
## Add custom CA certification to "luban-run", "luban-build" and "luban-buildinit" image
```
# Dockerfile.builder
FROM quay.io/luban-ci/luban-xxx:al2023

USER root

# Add your internal CA (PEM)
COPY cacert.crt /usr/local/share/ca-certificates/cacert.crt

RUN set -eux; \
    if command -v update-ca-certificates >/dev/null 2>&1; then \
      update-ca-certificates; \
    elif command -v update-ca-trust >/dev/null 2>&1; then \
      mkdir -p /etc/pki/ca-trust/source/anchors; \
      cp /usr/local/share/ca-certificates/cacert.crt /etc/pki/ca-trust/source/anchors/cacert.crt; \
      update-ca-trust extract; \
    else \
      echo "No known CA update tool found (need update-ca-certificates or update-ca-trust)" >&2; \
      exit 1; \
    fi

USER 1000

```
---
# 3. build the kpack builder
path: luban-ci/builder
## edit builder.toml, update the section "build" and "run"
eg:
```
description = "Luban CI Builder"

[[buildpacks]]
  uri = "../buildpacks/python-uv"
  version = "0.0.32"

[lifecycle]
  uri = "cache/lifecycle.tgz"

[[order]]
  [[order.group]]
    id = "luban-ci/python-uv"
    version = "0.0.32"

[[targets]]
  os = "linux"
  arch = "amd64"

[build]
  image = "quay.io/luban-ci/luban-build:al2023"

[run]
  [[run.images]]
    image = "quay.io/luban-ci/luban-run:al2023"

```
## Create and Push Builder and push it to Quay.io
```
make builder-build
make builder-push
```
## Inside company
### Downlod the following image and add custome CA certificate
Image:
- luban-builder

This image is used by the kpack CRD
- ClusterBuilder.spec.tag: rancher.harbor.local/luban-ci/luban-builder:al2023

## Add custom CA certification to "luban-run" and "luban-build" image
```
# Dockerfile.builder
FROM quay.io/luban-ci/luban-xxx:al2023

USER root

# Add your internal CA (PEM)
COPY cacert.crt /usr/local/share/ca-certificates/cacert.crt

RUN set -eux; \
    if command -v update-ca-certificates >/dev/null 2>&1; then \
      update-ca-certificates; \
    elif command -v update-ca-trust >/dev/null 2>&1; then \
      mkdir -p /etc/pki/ca-trust/source/anchors; \
      cp /usr/local/share/ca-certificates/cacert.crt /etc/pki/ca-trust/source/anchors/cacert.crt; \
      update-ca-trust extract; \
    else \
      echo "No known CA update tool found (need update-ca-certificates or update-ca-trust)" >&2; \
      exit 1; \
    fi

USER 1000
```
---

# 4. Deploy Argo workflows template and Set up ServiceAccounts and RBAC for Argo Workflows

```
make pipeline-deploy
```
```
make events-deploy events-webhook-secret
```



---
---
# Tools
- gitops-utils : luban-ci-dispatch, luban-ci-kpack-template
- luban-provisioner : git related operation

## gitops-utils
path: luban-ci/tools/gitops-utils
```
make build
make push
```

## luban-provisioner
path: luban-ci/tools/luban-provisioner
```
make build
make push
```

## Inside Company
Download this 2 images and add custom ca certificate.


---
---
# Image need to ship
- luban-builder
- luban-run
- luban-build
- gitops-utils
- luban-provisioner

## Kpack image used when image build 
- analyze : harbor.luban.paulhome.local/luban-ci/luban-builder:al2023@sha256:8fac8b0af16e88a9f66a188a31f06e827ca6f295b5d47038addcb60f54bd817c
- build: harbor.luban.paulhome.local/luban-ci/luban-builder:al2023@sha256:8fac8b0af16e88a9f66a188a31f06e827ca6f295b5d47038addcb60f54bd817c
- completion: harbor.luban.paulhome.local/luban-ci/luban-kpack-buildcompletion:al2023
- detect: harbor.luban.paulhome.local/luban-ci/luban-builder:al2023@sha256:8fac8b0af16e88a9f66a188a31f06e827ca6f295b5d47038addcb60f54bd817c
- export: harbor.luban.paulhome.local/luban-ci/luban-builder:al2023@sha256:8fac8b0af16e88a9f66a188a31f06e827ca6f295b5d47038addcb60f54bd817c
- prepare: harbor.luban.paulhome.local/luban-ci/luban-kpack-buildinit:al2023.1
- restore: harbor.luban.paulhome.local/luban-ci/luban-builder:al2023@sha256:8fac8b0af16e88a9f66a188a31f06e827ca6f295b5d47038addcb60f54bd817c
- 

# Other
## Project developer RBAC
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: project-developer-rb
  namespace: snd-data-service
subjects:
  - kind: ServiceAccount
    name: project-developer
    namespace: argo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: edit

---

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: argo-workflows-developer
rules:
  - apiGroups: ["argoproj.io"]
    resources:
      - workflows
      - workflowtemplates
      - cronworkflows
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: argo-workflows-developer-binding
subjects:
  - kind: ServiceAccount
    name: project-developer
    namespace: argo
roleRef:
  kind: ClusterRole
  name: argo-workflows-developer
  apiGroup: rbac.authorization.k8s.io



```
