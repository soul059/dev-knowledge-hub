# Chapter 4.3: GitRepository and Kustomization

[← Previous: Flux Installation](02-flux-installation.md) | [Back to README](../README.md) | [Next → HelmRelease](04-helmrelease.md)

---

## Overview

**GitRepository** and **Kustomization** are the two most fundamental CRDs in Flux. GitRepository fetches source code from a Git repo, and Kustomization builds and applies the manifests. Together, they form the core reconciliation pipeline.

---

## GitRepository CRD

### Basic GitRepository

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 1m              # How often to poll for changes
  url: https://github.com/myorg/my-app-config.git
  ref:
    branch: main             # Branch to track
  secretRef:
    name: git-credentials    # Auth secret (if private repo)
```

### GitRepository with Tag/Semver

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app-release
  namespace: flux-system
spec:
  interval: 5m
  url: https://github.com/myorg/my-app-config.git
  ref:
    # Track by tag
    tag: v1.2.3

    # Or by semver range (latest matching tag)
    # semver: ">=1.0.0 <2.0.0"

    # Or by commit SHA
    # commit: abc123def456
```

### GitRepository with SSH Authentication

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: private-repo
  namespace: flux-system
spec:
  interval: 1m
  url: ssh://git@github.com/myorg/private-config.git
  ref:
    branch: main
  secretRef:
    name: ssh-credentials
---
apiVersion: v1
kind: Secret
metadata:
  name: ssh-credentials
  namespace: flux-system
type: Opaque
stringData:
  identity: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    <your-ssh-private-key>
    -----END OPENSSH PRIVATE KEY-----
  known_hosts: |
    github.com ssh-ed25519 AAAA...
```

### GitRepository with Ignore Paths

```yaml
spec:
  ignore: |
    # Ignore everything except the deploy directory
    /*
    !/deploy
```

---

## How Source Controller Processes GitRepository

```
┌────────────────────────────────────────────────────────┐
│         GitRepository Reconciliation                   │
│                                                        │
│  1. Source Controller polls at interval (e.g., 1m)     │
│     │                                                  │
│     ▼                                                  │
│  2. git clone / git pull (shallow clone)               │
│     │                                                  │
│     ▼                                                  │
│  3. Compare HEAD commit with last seen                 │
│     │                                                  │
│     ├── Same commit → no action                        │
│     │                                                  │
│     └── New commit → produce new artifact              │
│         │                                              │
│         ▼                                              │
│  4. Store artifact (tarball of repo contents)           │
│     │                                                  │
│     ▼                                                  │
│  5. Update GitRepository .status.artifact              │
│     │  with URL, revision, and checksum                │
│     │                                                  │
│     ▼                                                  │
│  6. Downstream controllers (Kustomization/HelmRelease) │
│     are notified and reconcile                         │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Kustomization CRD

> **Note**: Flux's `Kustomization` CRD (`kustomize.toolkit.fluxcd.io`) is different from the Kustomize `kustomization.yaml` file.

### Basic Kustomization

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 10m             # Reconciliation interval
  sourceRef:
    kind: GitRepository
    name: my-app             # Reference to GitRepository
  path: ./deploy/production  # Path within the repo
  prune: true                # Delete resources removed from Git
  targetNamespace: my-app    # Override namespace for all resources
  timeout: 5m                # Timeout for apply operations
```

### Kustomization with Health Checks

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: my-app
  path: ./deploy
  prune: true
  wait: true                  # Wait for all resources to be ready
  healthChecks:
  - apiVersion: apps/v1
    kind: Deployment
    name: my-app
    namespace: my-app
  - apiVersion: v1
    kind: Service
    name: my-app-svc
    namespace: my-app
  timeout: 5m
```

### Kustomization with Patches

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app-prod
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: my-app
  path: ./deploy/base
  prune: true
  patches:
  - patch: |
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: my-app
      spec:
        replicas: 5
    target:
      kind: Deployment
      name: my-app
  - patch: |
      - op: add
        path: /metadata/labels/env
        value: production
    target:
      kind: Deployment
```

### Kustomization with Variable Substitution

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: my-app
  path: ./deploy
  prune: true
  postBuild:
    substitute:
      ENVIRONMENT: production
      CLUSTER_NAME: prod-us-east
    substituteFrom:
    - kind: ConfigMap
      name: cluster-vars
    - kind: Secret
      name: cluster-secrets
```

In your manifests use `${VARIABLE_NAME}`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  env: ${ENVIRONMENT}
  cluster: ${CLUSTER_NAME}
```

---

## Dependencies Between Kustomizations

```yaml
# Infrastructure must be applied before applications
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure
  prune: true
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 10m
  dependsOn:
  - name: infrastructure         # Wait for infrastructure first
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./apps
  prune: true
```

### Dependency Flow

```
┌──────────────────────────────────────────┐
│         KUSTOMIZATION DEPENDENCIES       │
│                                          │
│  ┌──────────────┐                        │
│  │ GitRepository│                        │
│  │ (source)     │                        │
│  └──────┬───────┘                        │
│         │                                │
│  ┌──────▼──────────┐                     │
│  │ infrastructure  │ ← applied first     │
│  │ (CRDs, operators│                     │
│  │  namespaces)    │                     │
│  └──────┬──────────┘                     │
│         │ dependsOn                      │
│  ┌──────▼──────────┐                     │
│  │    apps         │ ← applied second    │
│  │ (deployments,   │                     │
│  │  services)      │                     │
│  └─────────────────┘                     │
│                                          │
└──────────────────────────────────────────┘
```

---

## CLI Commands

```bash
# Create a GitRepository
flux create source git my-app \
  --url=https://github.com/myorg/my-app.git \
  --branch=main \
  --interval=1m

# Create a Kustomization
flux create kustomization my-app \
  --source=GitRepository/my-app \
  --path=./deploy \
  --prune=true \
  --interval=10m

# Check source status
flux get sources git

# Check kustomization status
flux get kustomizations

# Trigger immediate reconciliation
flux reconcile source git my-app
flux reconcile kustomization my-app

# Suspend / resume reconciliation
flux suspend kustomization my-app
flux resume kustomization my-app

# Export as YAML
flux export source git my-app > gitrepo.yaml
flux export kustomization my-app > ks.yaml
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| GitRepository | Fetches source from Git at a configured interval |
| Artifact | Tarball produced by Source Controller for downstream use |
| Kustomization | Builds and applies manifests from a source path |
| prune: true | Deletes resources removed from Git (garbage collection) |
| dependsOn | Ensures ordering between Kustomizations |
| postBuild substitute | Variable replacement in manifests |
| Health checks | Wait for Deployments/Services to become ready |
| Reconciliation trigger | `flux reconcile` for immediate sync |

---

## Quick Revision Questions

1. **What does a GitRepository resource do?**
   > It tells the Source Controller to poll a Git repository at a specified interval, produce an artifact when changes are detected, and make it available to downstream controllers.

2. **How does Flux detect changes in a Git repository?**
   > The Source Controller does a shallow clone/pull and compares the HEAD commit SHA with the previously observed one.

3. **What is the difference between Flux Kustomization CRD and kustomization.yaml?**
   > Flux Kustomization (`kustomize.toolkit.fluxcd.io`) is a CRD that tells the Kustomize Controller what source to use and what path to build. A `kustomization.yaml` file is the native Kustomize config that defines overlays/patches.

4. **How do you ensure infrastructure is applied before applications?**
   > Use `dependsOn` in the apps Kustomization to reference the infrastructure Kustomization.

5. **What does `prune: true` do?**
   > It enables garbage collection — if a resource is removed from Git, Flux deletes it from the cluster.

6. **How do you perform variable substitution in Flux?**
   > Use `postBuild.substitute` or `postBuild.substituteFrom` in the Kustomization, and reference variables as `${VAR_NAME}` in your manifests.

---

[← Previous: Flux Installation](02-flux-installation.md) | [Back to README](../README.md) | [Next → HelmRelease](04-helmrelease.md)
