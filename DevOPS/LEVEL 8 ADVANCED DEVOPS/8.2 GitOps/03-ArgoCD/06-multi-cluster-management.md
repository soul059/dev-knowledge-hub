# Chapter 3.6: Multi-Cluster Management

[← Previous: Rollouts and Rollbacks](05-rollouts-and-rollbacks.md) | [Back to README](../README.md) | [Next → RBAC in ArgoCD](07-rbac-in-argocd.md)

---

## Overview

ArgoCD can manage applications across **multiple Kubernetes clusters** from a single control plane. This is essential for multi-environment, multi-region, or multi-cloud architectures. This chapter covers cluster registration, ApplicationSets for multi-cluster, and hub-spoke vs standalone patterns.

---

## Multi-Cluster Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│              MULTI-CLUSTER GITOPS                                │
│                                                                  │
│  ┌──────────────────────┐                                        │
│  │     Git Repo         │                                        │
│  │  (Single Source of   │                                        │
│  │   Truth)             │                                        │
│  └──────────┬───────────┘                                        │
│             │                                                    │
│  ┌──────────▼───────────┐                                        │
│  │  ArgoCD Hub Cluster  │                                        │
│  │  (Management Plane)  │                                        │
│  └──────────┬───────────┘                                        │
│             │                                                    │
│    ┌────────┼────────┬────────────────┐                          │
│    │        │        │                │                          │
│    ▼        ▼        ▼                ▼                          │
│  ┌──────┐ ┌──────┐ ┌──────┐    ┌──────────┐                    │
│  │ Dev  │ │Staging│ │Prod  │    │Prod      │                    │
│  │Cluster│ │Cluster│ │US    │    │EU        │                    │
│  │      │ │      │ │      │    │          │                    │
│  └──────┘ └──────┘ └──────┘    └──────────┘                    │
│                                                                  │
│  One ArgoCD instance manages all clusters                        │
│  Each cluster is registered as a destination                     │
└──────────────────────────────────────────────────────────────────┘
```

---

## Registering Clusters

### Using CLI

```bash
# List current clusters
argocd cluster list

# Add a new cluster (uses current kubeconfig context)
argocd cluster add my-staging-cluster

# Add with a specific context
argocd cluster add staging-context \
  --name staging-cluster \
  --kubeconfig ~/.kube/staging-config

# The CLI creates a ServiceAccount in the target cluster
# and configures ArgoCD to use it
```

### Using Declarative Secret

```yaml
# Register cluster via Secret (preferred for GitOps)
apiVersion: v1
kind: Secret
metadata:
  name: staging-cluster
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
type: Opaque
stringData:
  name: staging-cluster
  server: https://staging-api.example.com:6443
  config: |
    {
      "bearerToken": "<service-account-token>",
      "tlsClientConfig": {
        "insecure": false,
        "caData": "<base64-encoded-ca-cert>"
      }
    }
```

---

## ApplicationSet for Multi-Cluster

### Cluster Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp-all-clusters
  namespace: argocd
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          env: production       # Only production clusters
  template:
    metadata:
      name: "myapp-{{name}}"   # Cluster name from label
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/config.git
        targetRevision: main
        path: "overlays/production"
      destination:
        server: "{{server}}"    # Cluster API URL
        namespace: myapp
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### Matrix Generator (Cluster × App)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: multi-cluster-apps
  namespace: argocd
spec:
  generators:
  - matrix:
      generators:
      - clusters:
          selector:
            matchLabels:
              env: production
      - list:
          elements:
          - app: frontend
            namespace: frontend
          - app: backend
            namespace: backend
          - app: monitoring
            namespace: monitoring
  template:
    metadata:
      name: "{{app}}-{{name}}"
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/config.git
        targetRevision: main
        path: "apps/{{app}}/overlays/production"
      destination:
        server: "{{server}}"
        namespace: "{{namespace}}"
```

### What Matrix Generates

```
  Clusters:                  Apps:
  ├── prod-us (server1)      ├── frontend
  └── prod-eu (server2)      ├── backend
                              └── monitoring

  Generated Applications (cluster × app):
  ├── frontend-prod-us   (server1, frontend ns)
  ├── frontend-prod-eu   (server2, frontend ns)
  ├── backend-prod-us    (server1, backend ns)
  ├── backend-prod-eu    (server2, backend ns)
  ├── monitoring-prod-us (server1, monitoring ns)
  └── monitoring-prod-eu (server2, monitoring ns)
```

---

## Hub-Spoke vs Standalone

```
┌──────────────────────────────────────────────────────────────┐
│           HUB-SPOKE vs STANDALONE                            │
│                                                              │
│  HUB-SPOKE:                                                  │
│  ┌─────────────────┐                                         │
│  │   ArgoCD Hub    │──── manages ────┐                       │
│  │   (1 instance)  │                 │                       │
│  └─────────────────┘      ┌──────────┼──────┐               │
│                           ▼          ▼      ▼               │
│                        Cluster1  Cluster2  Cluster3          │
│                                                              │
│  STANDALONE:                                                 │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐                   │
│  │ ArgoCD  │   │ ArgoCD  │   │ ArgoCD  │                   │
│  │    +    │   │    +    │   │    +    │                   │
│  │Cluster1 │   │Cluster2 │   │Cluster3 │                   │
│  └─────────┘   └─────────┘   └─────────┘                   │
│  Each cluster has its own ArgoCD                            │
│  All point to the same Git repo                              │
└──────────────────────────────────────────────────────────────┘
```

| Aspect | Hub-Spoke | Standalone |
|--------|-----------|------------|
| ArgoCD instances | 1 (central) | 1 per cluster |
| Single pane of glass | ✓ Yes | ✗ No |
| Network requirement | Hub must reach all clusters | Only outbound to Git |
| Blast radius | Hub failure affects all | Isolated per cluster |
| Scalability | Limited by hub resources | Scales independently |
| Best for | < 20 clusters | > 20 clusters, strict isolation |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Cluster registration | CLI (`argocd cluster add`) or Secret |
| Cluster generator | ApplicationSet generates apps per cluster |
| Matrix generator | Cross-product of clusters × apps |
| Hub-spoke | Single ArgoCD manages all clusters |
| Standalone | ArgoCD per cluster, same Git repo |
| Best for < 20 clusters | Hub-spoke pattern |
| Best for > 20 clusters | Standalone with shared Git repo |

---

## Quick Revision Questions

1. **How do you register a remote cluster with ArgoCD?**
   > Using `argocd cluster add <context>` (CLI) or by creating a Secret with the label `argocd.argoproj.io/secret-type: cluster`.

2. **What is a Cluster Generator in ApplicationSet?**
   > It automatically generates one Application per registered cluster matching a label selector.

3. **What does a Matrix Generator produce?**
   > A cross-product — for every combination of clusters and apps, it generates a unique Application.

4. **When would you choose standalone over hub-spoke?**
   > When you have > 20 clusters, need strict isolation, or want to avoid a single point of failure.

5. **What happens if the ArgoCD hub cluster goes down in a hub-spoke model?**
   > Existing workloads continue running but new syncs, drift detection, and self-healing stop until the hub recovers.

6. **How do you ensure the same application is deployed to all production clusters?**
   > Use an ApplicationSet with a Cluster Generator that selects clusters labeled `env: production`.

---

[← Previous: Rollouts and Rollbacks](05-rollouts-and-rollbacks.md) | [Back to README](../README.md) | [Next → RBAC in ArgoCD](07-rbac-in-argocd.md)
