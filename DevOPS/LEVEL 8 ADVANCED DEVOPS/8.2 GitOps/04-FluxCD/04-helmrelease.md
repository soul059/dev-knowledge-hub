# Chapter 4.4: HelmRelease

[← Previous: GitRepository and Kustomization](03-gitrepository-and-kustomization.md) | [Back to README](../README.md) | [Next → Image Automation](05-image-automation.md)

---

## Overview

**HelmRelease** is a Flux CRD that manages the lifecycle of Helm chart releases — install, upgrade, test, rollback, and uninstall. Unlike ArgoCD (which uses `helm template`), Flux's Helm Controller uses **native Helm SDK** operations, meaning releases are tracked by Helm.

---

## HelmRelease Lifecycle

```
┌──────────────────────────────────────────────────────────┐
│             HELMRELEASE LIFECYCLE                        │
│                                                          │
│  ┌─────────────────┐                                     │
│  │ HelmRepository / │                                    │
│  │ GitRepository    │  (source of chart)                 │
│  └────────┬────────┘                                     │
│           │                                              │
│  ┌────────▼────────┐                                     │
│  │  Helm Controller │                                    │
│  │                  │                                    │
│  │  1. Fetch chart  │                                    │
│  │  2. Merge values │                                    │
│  │  3. helm install │ (or helm upgrade)                  │
│  │  4. Run tests    │ (if enabled)                       │
│  │  5. Health check │                                    │
│  │  6. Auto-rollback│ (on failure)                       │
│  └─────────────────┘                                     │
│                                                          │
│  Key difference from ArgoCD:                             │
│  Flux uses real `helm install/upgrade`                   │
│  Helm releases ARE tracked (helm list shows them)        │
└──────────────────────────────────────────────────────────┘
```

---

## HelmRepository Source

```yaml
# Define a Helm chart repository
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  interval: 30m
  url: https://charts.bitnami.com/bitnami
---
# OCI-based Helm repository
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: podinfo-oci
  namespace: flux-system
spec:
  interval: 30m
  type: oci
  url: oci://ghcr.io/stefanprodan/charts
```

---

## HelmRelease Examples

### Basic HelmRelease from Helm Repository

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: nginx
  namespace: flux-system
spec:
  interval: 10m
  chart:
    spec:
      chart: nginx                        # Chart name
      version: "15.x"                     # Semver range
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
      interval: 5m                        # How often to check for new chart versions
  targetNamespace: web                    # Install to this namespace
  install:
    createNamespace: true
  values:
    replicaCount: 2
    service:
      type: ClusterIP
```

### HelmRelease from Git Repository

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 10m
  chart:
    spec:
      chart: ./charts/my-app             # Path to chart in Git repo
      sourceRef:
        kind: GitRepository
        name: my-app-repo
        namespace: flux-system
  values:
    image:
      tag: v2.0.0
```

### HelmRelease with Values from ConfigMap/Secret

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 10m
  chart:
    spec:
      chart: my-app
      version: "1.x"
      sourceRef:
        kind: HelmRepository
        name: my-charts
  valuesFrom:
  - kind: ConfigMap
    name: my-app-values
    valuesKey: values.yaml           # Key in the ConfigMap
  - kind: Secret
    name: my-app-secrets
    valuesKey: secret-values.yaml
    targetPath: database.password    # Mount at specific path
  values:
    # Inline values (lowest priority)
    replicaCount: 3
```

**Values precedence** (highest to lowest):
1. `valuesFrom` (in order listed)
2. `values` (inline)

---

## Upgrade and Rollback Configuration

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 10m
  chart:
    spec:
      chart: my-app
      version: "1.x"
      sourceRef:
        kind: HelmRepository
        name: my-charts
  
  # Install configuration
  install:
    createNamespace: true
    remediation:
      retries: 3                      # Retry install 3 times

  # Upgrade configuration
  upgrade:
    cleanupOnFail: true               # Delete new resources on failed upgrade
    remediation:
      retries: 3                      # Retry upgrade 3 times
      remediateLastFailure: true      # Rollback on last retry failure
      strategy: rollback              # rollback or uninstall

  # Rollback configuration
  rollback:
    timeout: 5m
    cleanupOnFail: true

  # Uninstall configuration
  uninstall:
    keepHistory: false
    timeout: 5m

  # Test configuration
  test:
    enable: true                      # Run helm test after install/upgrade
    timeout: 5m
    ignoreFailures: false
```

### Rollback Flow

```
┌──────────────────────────────────────────────────────────┐
│             AUTOMATIC ROLLBACK FLOW                      │
│                                                          │
│  1. New chart version detected                           │
│     │                                                    │
│     ▼                                                    │
│  2. helm upgrade --install                               │
│     │                                                    │
│     ├── Success → Done (reconciled)                      │
│     │                                                    │
│     └── Failure → Retry (up to retries count)            │
│         │                                                │
│         ├── Retry success → Done                         │
│         │                                                │
│         └── All retries failed →                         │
│             │                                            │
│             ▼                                            │
│         strategy: rollback                               │
│             → helm rollback to previous release          │
│                                                          │
│         strategy: uninstall                              │
│             → helm uninstall                             │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Dependencies Between HelmReleasese

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: cert-manager
  namespace: flux-system
spec:
  interval: 10m
  chart:
    spec:
      chart: cert-manager
      sourceRef:
        kind: HelmRepository
        name: jetstack
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 10m
  dependsOn:
  - name: cert-manager          # Install cert-manager first
  chart:
    spec:
      chart: my-app
      sourceRef:
        kind: HelmRepository
        name: my-charts
```

---

## CLI Commands

```bash
# Create a HelmRepository
flux create source helm bitnami \
  --url=https://charts.bitnami.com/bitnami \
  --interval=30m

# Create a HelmRelease
flux create helmrelease nginx \
  --source=HelmRepository/bitnami \
  --chart=nginx \
  --chart-version="15.x" \
  --target-namespace=web \
  --create-target-namespace

# Check HelmRelease status
flux get helmreleases -A

# Trigger immediate reconciliation
flux reconcile helmrelease nginx

# Suspend / resume
flux suspend helmrelease nginx
flux resume helmrelease nginx

# View Helm releases (standard Helm CLI works too)
helm list -A
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| HelmRepository | Source CRD pointing to a Helm chart index or OCI registry |
| HelmRelease | Manages install, upgrade, test, rollback of a chart |
| Native Helm | Flux uses `helm install/upgrade` (not template) |
| Values precedence | `valuesFrom` > `values` (inline) |
| Auto-rollback | On upgrade failure, rolls back to previous release |
| Remediation | Configurable retries and failure strategies |
| dependsOn | Order HelmRelease installations |
| Helm tracking | Releases visible in `helm list` |

---

## Quick Revision Questions

1. **How does Flux's Helm handling differ from ArgoCD's?**
   > Flux uses native `helm install/upgrade` via the Helm SDK, so releases are tracked. ArgoCD uses `helm template` and applies plain YAML — no Helm release tracking.

2. **What CRD defines where Helm charts come from?**
   > `HelmRepository` — it points to a Helm chart index URL or OCI registry.

3. **How do you configure automatic rollback on upgrade failure?**
   > Set `upgrade.remediation.strategy: rollback` and `upgrade.remediation.remediateLastFailure: true`.

4. **What is the values precedence in a HelmRelease?**
   > `valuesFrom` entries (in order) take precedence over inline `values`.

5. **How do you ensure a HelmRelease installs after another?**
   > Use `dependsOn` to list the HelmRelease resources that must be ready first.

---

[← Previous: GitRepository and Kustomization](03-gitrepository-and-kustomization.md) | [Back to README](../README.md) | [Next → Image Automation](05-image-automation.md)
