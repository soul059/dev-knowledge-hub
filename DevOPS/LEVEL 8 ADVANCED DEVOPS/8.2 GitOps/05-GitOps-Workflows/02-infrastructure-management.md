# Chapter 5.2: Infrastructure Management

[← Previous: Application Deployment](01-application-deployment.md) | [Back to README](../README.md) | [Next → Environment Promotion](03-environment-promotion.md)

---

## Overview

GitOps isn't just for applications — it excels at managing **cluster infrastructure**: service meshes, ingress controllers, monitoring stacks, cert-manager, operators, and namespaces. Infrastructure-as-Code (IaC) in Git ensures every cluster component is versioned, auditable, and reproducible.

---

## Infrastructure GitOps Architecture

```
┌──────────────────────────────────────────────────────────────┐
│         INFRASTRUCTURE GITOPS LAYERS                         │
│                                                              │
│  ┌────────────────────────────────────────┐                  │
│  │ Layer 3: Applications                  │                  │
│  │ (my-app, frontend, backend)            │                  │
│  │ dependsOn: Layer 2                     │                  │
│  └───────────────────┬────────────────────┘                  │
│                      │                                       │
│  ┌───────────────────▼────────────────────┐                  │
│  │ Layer 2: Platform Services             │                  │
│  │ (ingress-nginx, cert-manager,          │                  │
│  │  monitoring, logging)                  │                  │
│  │ dependsOn: Layer 1                     │                  │
│  └───────────────────┬────────────────────┘                  │
│                      │                                       │
│  ┌───────────────────▼────────────────────┐                  │
│  │ Layer 1: Cluster Foundation            │                  │
│  │ (namespaces, RBAC, network policies,   │                  │
│  │  storage classes, CRDs)                │                  │
│  └────────────────────────────────────────┘                  │
│                                                              │
│  Each layer depends on the one below it                      │
│  GitOps operator manages all layers from Git                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Repository Structure for Infrastructure

```
fleet-infra/
├── clusters/
│   ├── dev/
│   │   ├── flux-system/
│   │   ├── infrastructure.yaml      # Kustomization for infra
│   │   └── apps.yaml                # Kustomization for apps
│   └── production/
│       ├── flux-system/
│       ├── infrastructure.yaml
│       └── apps.yaml
├── infrastructure/
│   ├── base/
│   │   ├── cert-manager/
│   │   │   ├── namespace.yaml
│   │   │   ├── helmrelease.yaml
│   │   │   └── kustomization.yaml
│   │   ├── ingress-nginx/
│   │   │   ├── namespace.yaml
│   │   │   ├── helmrelease.yaml
│   │   │   └── kustomization.yaml
│   │   ├── monitoring/
│   │   │   ├── namespace.yaml
│   │   │   ├── kube-prometheus-stack.yaml
│   │   │   └── kustomization.yaml
│   │   └── kustomization.yaml
│   └── overlays/
│       ├── dev/
│       │   └── kustomization.yaml
│       └── production/
│           └── kustomization.yaml
└── apps/
    ├── base/
    └── overlays/
```

---

## Managing Infrastructure Components

### Cert-Manager via HelmRelease

```yaml
# infrastructure/base/cert-manager/helmrelease.yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: cert-manager
  namespace: cert-manager
spec:
  interval: 30m
  chart:
    spec:
      chart: cert-manager
      version: "1.14.x"
      sourceRef:
        kind: HelmRepository
        name: jetstack
        namespace: flux-system
  install:
    crds: CreateReplace
    createNamespace: true
  upgrade:
    crds: CreateReplace
  values:
    installCRDs: true
    prometheus:
      enabled: true
```

### Ingress-Nginx via HelmRelease

```yaml
# infrastructure/base/ingress-nginx/helmrelease.yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
spec:
  interval: 30m
  chart:
    spec:
      chart: ingress-nginx
      version: "4.9.x"
      sourceRef:
        kind: HelmRepository
        name: ingress-nginx
        namespace: flux-system
  values:
    controller:
      replicaCount: 2
      metrics:
        enabled: true
      service:
        type: LoadBalancer
```

### Monitoring Stack

```yaml
# infrastructure/base/monitoring/kube-prometheus-stack.yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: kube-prometheus-stack
  namespace: monitoring
spec:
  interval: 30m
  chart:
    spec:
      chart: kube-prometheus-stack
      version: "56.x"
      sourceRef:
        kind: HelmRepository
        name: prometheus-community
        namespace: flux-system
  install:
    crds: CreateReplace
    createNamespace: true
  values:
    grafana:
      adminPassword: ${GRAFANA_ADMIN_PASSWORD}
    prometheus:
      prometheusSpec:
        retention: 30d
        storageSpec:
          volumeClaimTemplate:
            spec:
              accessModes: ["ReadWriteOnce"]
              resources:
                requests:
                  storage: 50Gi
```

---

## Infrastructure Dependencies (Flux)

```yaml
# clusters/production/infrastructure.yaml
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
  path: ./infrastructure/overlays/production
  prune: true
  wait: true
---
# clusters/production/apps.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 10m
  dependsOn:
  - name: infrastructure           # Apps wait for infra
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./apps/overlays/production
  prune: true
```

## Infrastructure Dependencies (ArgoCD)

```yaml
# App of Apps for infrastructure
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: infrastructure
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "-1"    # Sync before apps
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/fleet-infra.git
    targetRevision: main
    path: infrastructure/production
  destination:
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## Managing CRDs

```
┌──────────────────────────────────────────────────────────┐
│              CRD MANAGEMENT STRATEGIES                   │
│                                                          │
│  Option A: Install CRDs with Helm chart                  │
│  ├── Set installCRDs: true in chart values               │
│  └── Upgrade: crds: CreateReplace                        │
│                                                          │
│  Option B: Separate CRD Kustomization                    │
│  ├── Apply CRDs in a dedicated Kustomization             │
│  ├── Use dependsOn for operator Kustomization            │
│  └── Prevents CRD deletion on chart uninstall            │
│                                                          │
│  Option C: Server-side apply                             │
│  ├── syncOptions: ServerSideApply=true (ArgoCD)          │
│  └── Handles large CRDs that exceed annotation limits    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Separate CRD Management

```yaml
# Apply CRDs first
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: cert-manager-crds
  namespace: flux-system
spec:
  interval: 30m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/crds/cert-manager
  prune: false                  # Never prune CRDs!
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: cert-manager
  namespace: flux-system
spec:
  interval: 30m
  dependsOn:
  - name: cert-manager-crds     # CRDs must exist first
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/base/cert-manager
  prune: true
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Infrastructure layers | Foundation → Platform Services → Applications |
| Dependency ordering | Use `dependsOn` (Flux) or sync waves (ArgoCD) |
| Helm-based infra | HelmRelease for ingress, cert-manager, monitoring |
| CRD management | Install separately with `prune: false` or via Helm |
| Environment overlays | Base config + per-env patches (Kustomize) |
| Self-healing | Operator restores infra if manually changed |
| Server-side apply | Required for large CRDs exceeding annotation limits |

---

## Quick Revision Questions

1. **Why manage infrastructure through GitOps?**
   > Infrastructure components become versioned, auditable, reproducible, and self-healing — same benefits as GitOps for applications.

2. **What is the typical layering for infrastructure?**
   > Layer 1: Cluster foundation (namespaces, RBAC, CRDs), Layer 2: Platform services (ingress, cert-manager, monitoring), Layer 3: Applications.

3. **How do you ensure CRDs are applied before operators?**
   > Use a separate Kustomization for CRDs with `prune: false`, and set `dependsOn` in the operator Kustomization.

4. **Why set `prune: false` on CRD Kustomizations?**
   > To prevent accidental deletion of CRDs, which would cascade-delete all custom resources of that type.

5. **How do you handle environment differences for infrastructure?**
   > Use Kustomize overlays with a shared base and per-environment patches (e.g., different replica counts, storage sizes, feature flags).

---

[← Previous: Application Deployment](01-application-deployment.md) | [Back to README](../README.md) | [Next → Environment Promotion](03-environment-promotion.md)
