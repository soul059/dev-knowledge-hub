# Chapter 3.8: ArgoCD with Helm

[← Previous: RBAC in ArgoCD](07-rbac-in-argocd.md) | [Back to README](../README.md) | [Next → Flux Architecture](../04-FluxCD/01-flux-architecture.md)

---

## Overview

ArgoCD natively supports **Helm charts** as an application source. You can deploy charts from Git repos, Helm repositories, or OCI registries. ArgoCD renders the chart server-side and manages the resulting manifests declaratively. This chapter covers all patterns for using Helm with ArgoCD.

---

## How ArgoCD Renders Helm

```
┌──────────────────────────────────────────────────────────┐
│           ArgoCD Helm Rendering Pipeline                 │
│                                                          │
│  ┌──────────────┐    ┌──────────────┐                   │
│  │ Helm Chart   │───→│ Repo Server  │                   │
│  │  (source)    │    │ (renders     │                   │
│  └──────────────┘    │  templates)  │                   │
│                      └──────┬───────┘                   │
│                             │                            │
│                    helm template                         │
│                    (server-side)                          │
│                             │                            │
│                      ┌──────▼───────┐                   │
│                      │  Plain YAML  │                   │
│                      │  Manifests   │                   │
│                      └──────┬───────┘                   │
│                             │                            │
│                      ┌──────▼───────┐                   │
│                      │ App Controller│                   │
│                      │ (sync to     │                   │
│                      │  cluster)    │                   │
│                      └──────────────┘                   │
│                                                          │
│  Note: ArgoCD does NOT use `helm install`.               │
│  It uses `helm template` then applies with kubectl.      │
└──────────────────────────────────────────────────────────┘
```

> **Key insight**: ArgoCD uses `helm template` (not `helm install`). This means Helm releases are NOT tracked in the cluster. ArgoCD owns the lifecycle entirely.

---

## Pattern 1: Helm Chart from Git Repository

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/helm-charts.git
    targetRevision: main
    path: charts/myapp            # Path to chart in repo
    helm:
      valueFiles:
      - values.yaml               # Default values
      - values-production.yaml    # Environment overlay
      parameters:
      - name: image.tag
        value: "v2.1.0"
      - name: replicas
        value: "3"
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
```

---

## Pattern 2: Helm Chart from Helm Repository

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-ingress
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://kubernetes.github.io/ingress-nginx   # Helm repo URL
    chart: ingress-nginx                                    # Chart name
    targetRevision: 4.9.1                                   # Chart version
    helm:
      releaseName: nginx-ingress
      values: |
        controller:
          replicaCount: 2
          service:
            type: LoadBalancer
          metrics:
            enabled: true
  destination:
    server: https://kubernetes.default.svc
    namespace: ingress-nginx
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
```

---

## Pattern 3: OCI Helm Registry

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-oci
  namespace: argocd
spec:
  project: default
  source:
    repoURL: registry-1.docker.io/myorg       # OCI registry
    chart: myapp                                # Chart name in registry
    targetRevision: 1.5.0
    helm:
      values: |
        env: production
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
```

Register OCI credentials:

```bash
argocd repo add registry-1.docker.io/myorg \
  --type helm \
  --name myorg-oci \
  --enable-oci \
  --username myuser \
  --password mytoken
```

---

## Pattern 4: Multiple Sources (Chart + External Values)

```yaml
# ArgoCD 2.6+: Use multiple sources for chart + values from different repos
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-multi-source
  namespace: argocd
spec:
  project: default
  sources:                                         # Note: sources (plural)
  - repoURL: https://charts.bitnami.com/bitnami    # Helm chart repo
    chart: nginx
    targetRevision: 15.4.4
    helm:
      valueFiles:
      - $values/envs/production/nginx-values.yaml   # Reference to 2nd source
  - repoURL: https://github.com/myorg/config.git   # Values repo
    targetRevision: main
    ref: values                                     # Reference name ($values)
  destination:
    server: https://kubernetes.default.svc
    namespace: nginx
```

This pattern separates the **chart** (upstream) from the **values** (your config repo).

---

## Helm Parameters vs Values

```yaml
helm:
  # Inline values (like -f values.yaml)
  values: |
    replicaCount: 3
    image:
      repository: myapp
      tag: v2.0.0

  # Value files from the source repo
  valueFiles:
  - values.yaml
  - values-prod.yaml

  # Individual parameters (like --set)
  parameters:
  - name: image.tag
    value: "v2.1.0"
  - name: service.type
    value: NodePort

  # Force string parameters (like --set-string)
  - name: podAnnotations.timestamp
    value: "20240101"
    forceString: true

  # File parameters (like --set-file)
  fileParameters:
  - name: config
    path: files/config.json

  # Skip CRDs
  skipCrds: false

  # Pass all values through
  passCredentials: false

  # Release name (default: app name)
  releaseName: my-release

  # Helm version (v2 or v3, default: v3)
  version: v3
```

---

## Comparison of Helm Patterns

| Pattern | Source | Values Location | Best For |
|---------|--------|-----------------|----------|
| Chart from Git | Git repo | Same repo | Internal charts |
| Chart from Helm Repo | Helm repo URL | Inline or file | Community charts |
| Chart from OCI | OCI registry | Inline or file | Private registries |
| Multi-source | Separate repos | Separate config repo | Chart + env-specific values |

---

## Common Configurations

### Adding a Private Helm Repository

```bash
# HTTPS
argocd repo add https://charts.example.com \
  --type helm \
  --name my-charts \
  --username admin \
  --password secret

# Or declaratively
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-helm-repo
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
type: Opaque
stringData:
  type: helm
  name: my-charts
  url: https://charts.example.com
  username: admin
  password: secret
```

### Ignoring Helm Hook Differences

```yaml
syncPolicy:
  syncOptions:
  - RespectIgnoreDifferences=true
spec:
  ignoreDifferences:
  - group: ""
    kind: Secret
    jsonPointers:
    - /data
```

---

## Troubleshooting Helm in ArgoCD

| Issue | Cause | Solution |
|-------|-------|----------|
| App shows OutOfSync but no diff | Helm hook resources | Add annotation `argocd.argoproj.io/hook` |
| Values not applying | Wrong value file path | Verify path relative to chart root |
| CRD issues on first sync | CRDs not installed before operator | Use sync waves: CRDs in wave -1 |
| `helm template` fails | Chart requires dependencies | Ensure `Chart.lock` is committed |
| Diff shows random changes | Non-deterministic templates | Pin Helm version, avoid `randAlpha` |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Rendering | ArgoCD uses `helm template`, NOT `helm install` |
| Git source | Set `path` to chart directory in Git repo |
| Helm repo source | Set `chart` + `repoURL` (Helm repo URL) |
| OCI source | Set `chart` + OCI registry URL + `--enable-oci` |
| Multiple sources | Separate chart repo and values repo (ArgoCD 2.6+) |
| Values precedence | parameters > values > valueFiles |
| Release name | Defaults to Application name, overridable |
| Helm hooks | Not directly supported; use sync hooks instead |

---

## Quick Revision Questions

1. **Does ArgoCD use `helm install` or `helm template`?**
   > `helm template` — ArgoCD renders charts server-side and applies plain YAML via kubectl. Helm releases are NOT tracked in the cluster.

2. **How do you deploy a chart from a public Helm repository?**
   > Set `source.repoURL` to the Helm repo URL, `source.chart` to the chart name, and `source.targetRevision` to the chart version.

3. **What is the multi-source pattern and when would you use it?**
   > Using `sources` (plural) in ArgoCD 2.6+, you separate the chart source from the values file source. Useful when using upstream charts with environment-specific values in a config repo.

4. **What is the precedence of Helm values in ArgoCD?**
   > `parameters` override `values` (inline), which override `valueFiles`.

5. **Why might a Helm-based app show OutOfSync with no visible diff?**
   > Helm hooks create resources that ArgoCD doesn't expect. Use ArgoCD sync hooks or annotations to handle them.

---

[← Previous: RBAC in ArgoCD](07-rbac-in-argocd.md) | [Back to README](../README.md) | [Next → Flux Architecture](../04-FluxCD/01-flux-architecture.md)
