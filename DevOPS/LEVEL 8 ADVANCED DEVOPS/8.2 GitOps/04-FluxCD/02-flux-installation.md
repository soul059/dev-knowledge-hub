# Chapter 4.2: Flux Installation

[← Previous: Flux Architecture](01-flux-architecture.md) | [Back to README](../README.md) | [Next → GitRepository and Kustomization](03-gitrepository-and-kustomization.md)

---

## Overview

Flux is installed using the **Flux CLI** (`flux bootstrap`), which sets up all controllers in the cluster and creates a Git repository structure that manages Flux itself (self-managing GitOps). This chapter covers installation methods, prerequisites, and post-install verification.

---

## Prerequisites

```
┌──────────────────────────────────────────────────────────┐
│                 PRE-REQUISITES                           │
│                                                          │
│  ✔ Kubernetes cluster (1.26+)                           │
│  ✔ kubectl configured                                   │
│  ✔ Git repository (GitHub, GitLab, Bitbucket)           │
│  ✔ Personal Access Token with repo permissions          │
│  ✔ Flux CLI installed                                   │
└──────────────────────────────────────────────────────────┘
```

---

## Installing the Flux CLI

```bash
# macOS (Homebrew)
brew install fluxcd/tap/flux

# Linux
curl -s https://fluxcd.io/install.sh | sudo bash

# Windows (Chocolatey)
choco install flux

# Windows (Scoop)
scoop install flux

# Verify installation
flux --version

# Check cluster prerequisites
flux check --pre
```

---

## Bootstrap Methods

### Method 1: GitHub Bootstrap (Recommended)

```bash
# Set environment variables
export GITHUB_TOKEN=<your-personal-access-token>
export GITHUB_USER=<your-github-username>

# Bootstrap Flux on the cluster
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/my-cluster \
  --personal

# What this does:
# 1. Creates the GitHub repo (if it doesn't exist)
# 2. Installs Flux controllers in flux-system namespace
# 3. Commits Flux manifests to the repo
# 4. Configures Flux to manage itself from the repo
```

### Method 2: GitLab Bootstrap

```bash
export GITLAB_TOKEN=<your-gitlab-token>

flux bootstrap gitlab \
  --owner=my-group \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/my-cluster \
  --token-auth
```

### Method 3: Generic Git Bootstrap

```bash
flux bootstrap git \
  --url=ssh://git@git.example.com/myorg/fleet-infra.git \
  --branch=main \
  --path=clusters/my-cluster \
  --private-key-file=/path/to/ssh-key
```

### Method 4: Manual Install (without bootstrap)

```bash
# Install controllers only (no Git management)
flux install

# Or using kubectl
kubectl apply -f https://github.com/fluxcd/flux2/releases/latest/download/install.yaml
```

---

## Bootstrap Flow

```
┌──────────────────────────────────────────────────────────┐
│              FLUX BOOTSTRAP FLOW                         │
│                                                          │
│  1. flux bootstrap github ...                            │
│     │                                                    │
│     ▼                                                    │
│  2. Creates/connects to Git repository                   │
│     │                                                    │
│     ▼                                                    │
│  3. Generates Flux component manifests                   │
│     │  (controllers, CRDs, RBAC)                         │
│     │                                                    │
│     ▼                                                    │
│  4. Commits manifests to repo at:                        │
│     │  clusters/my-cluster/flux-system/                  │
│     │  ├── gotk-components.yaml                          │
│     │  ├── gotk-sync.yaml                                │
│     │  └── kustomization.yaml                            │
│     │                                                    │
│     ▼                                                    │
│  5. Applies manifests to cluster                         │
│     │  (installs controllers)                            │
│     │                                                    │
│     ▼                                                    │
│  6. Creates GitRepository + Kustomization                │
│     │  pointing to the repo itself                       │
│     │                                                    │
│     ▼                                                    │
│  7. Flux now manages itself from Git!                    │
│     (Self-managing GitOps)                               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Repository Structure After Bootstrap

```
fleet-infra/
├── clusters/
│   └── my-cluster/
│       └── flux-system/
│           ├── gotk-components.yaml    # All Flux controllers
│           ├── gotk-sync.yaml          # GitRepository + Kustomization
│           └── kustomization.yaml      # Kustomize entry point
```

### gotk-sync.yaml (generated)

```yaml
# GitRepository pointing to itself
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 1m0s
  ref:
    branch: main
  url: https://github.com/myuser/fleet-infra.git
  secretRef:
    name: flux-system
---
# Kustomization that applies everything under the cluster path
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 10m0s
  path: ./clusters/my-cluster
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
```

---

## Post-Install Verification

```bash
# Check all Flux controllers are running
flux check

# Expected output:
# ► checking prerequisites
# ✔ Kubernetes 1.28.0 >=1.26.0-0
# ► checking controllers
# ✔ source-controller is healthy
# ✔ kustomize-controller is healthy
# ✔ helm-controller is healthy
# ✔ notification-controller is healthy
# ✔ all checks passed

# View installed components
kubectl get pods -n flux-system

# NAME                                        READY   STATUS
# source-controller-xxxx                      1/1     Running
# kustomize-controller-xxxx                   1/1     Running
# helm-controller-xxxx                        1/1     Running
# notification-controller-xxxx                1/1     Running

# Check Flux version and components
flux version

# Check GitRepository status
flux get sources git -A

# Check Kustomization status
flux get kustomizations -A
```

---

## Upgrading Flux

```bash
# Upgrade Flux CLI first
brew upgrade fluxcd/tap/flux   # macOS

# Then re-run bootstrap (idempotent)
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/my-cluster \
  --personal

# This updates gotk-components.yaml with the new version
# and commits it to Git. Flux then upgrades itself.
```

---

## Uninstalling Flux

```bash
# Uninstall Flux and remove all CRDs
flux uninstall

# This removes:
# - All Flux controllers
# - All Flux CRDs (and all Flux custom resources!)
# - The flux-system namespace
# WARNING: Workloads deployed by Flux remain untouched
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| CLI install | `brew install fluxcd/tap/flux` or curl script |
| Bootstrap | `flux bootstrap github` creates repo + installs controllers |
| Self-managing | Flux manages its own installation via Git |
| Generated files | `gotk-components.yaml` (controllers) + `gotk-sync.yaml` (self-reference) |
| Pre-check | `flux check --pre` validates cluster prerequisites |
| Post-check | `flux check` verifies all controllers are healthy |
| Upgrade | Re-run `flux bootstrap` (idempotent) |
| Namespace | Installs in `flux-system` by default |

---

## Quick Revision Questions

1. **What does `flux bootstrap` do?**
   > It creates a Git repository (if needed), installs Flux controllers in the cluster, commits Flux manifests to the repo, and configures Flux to manage itself.

2. **Why is Flux considered self-managing after bootstrap?**
   > The bootstrap creates a GitRepository + Kustomization that points to the repo containing Flux's own manifests. Changes to Flux are applied through Git.

3. **What files are generated during bootstrap?**
   > `gotk-components.yaml` (controller manifests), `gotk-sync.yaml` (GitRepository + Kustomization), and `kustomization.yaml` (Kustomize entry point).

4. **How do you upgrade Flux?**
   > Upgrade the CLI, then re-run `flux bootstrap`. It regenerates `gotk-components.yaml` with new versions and commits to Git.

5. **What command verifies Flux is healthy?**
   > `flux check` — it validates all controllers are running and compatible.

---

[← Previous: Flux Architecture](01-flux-architecture.md) | [Back to README](../README.md) | [Next → GitRepository and Kustomization](03-gitrepository-and-kustomization.md)
