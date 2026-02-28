# Chapter 4.6: Multi-Tenancy

[← Previous: Image Automation](05-image-automation.md) | [Back to README](../README.md) | [Next → Application Deployment](../05-GitOps-Workflows/01-application-deployment.md)

---

## Overview

**Multi-tenancy** in Flux allows a single cluster to serve multiple teams or tenants, each with their own Git repositories, namespaces, and RBAC boundaries. Flux achieves this through **Kustomization scoping**, **ServiceAccount impersonation**, and **cross-namespace references**.

---

## Multi-Tenancy Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              FLUX MULTI-TENANCY MODEL                        │
│                                                              │
│  ┌─────────────────────────┐                                 │
│  │   Platform Team          │                                │
│  │   (flux-system namespace)│                                │
│  │                          │                                │
│  │   flux-system/           │                                │
│  │   ├── GitRepository      │ (fleet-infra repo)             │
│  │   ├── Kustomization      │ (applies tenant configs)       │
│  │   └── RBAC (admin)       │                                │
│  └──────────┬───────────────┘                                │
│             │ creates tenant namespaces + RBAC               │
│    ┌────────┼────────┐                                       │
│    ▼        ▼        ▼                                       │
│  ┌──────┐ ┌──────┐ ┌──────┐                                 │
│  │Team A│ │Team B│ │Team C│                                  │
│  │      │ │      │ │      │                                  │
│  │Git:  │ │Git:  │ │Git:  │                                  │
│  │team-a│ │team-b│ │team-c│                                  │
│  │-repo │ │-repo │ │-repo │                                  │
│  │      │ │      │ │      │                                  │
│  │NS:   │ │NS:   │ │NS:   │                                  │
│  │team-a│ │team-b│ │team-c│                                  │
│  └──────┘ └──────┘ └──────┘                                 │
│                                                              │
│  Each team can only deploy to their own namespace(s)         │
│  Platform team controls what each tenant can do              │
└──────────────────────────────────────────────────────────────┘
```

---

## Setting Up Multi-Tenancy

### Step 1: Tenant Namespace and RBAC

```yaml
# tenants/team-a/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: team-a
  labels:
    toolkit.fluxcd.io/tenant: team-a
---
# tenants/team-a/rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: team-a
  namespace: team-a
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-a-reconciler
  namespace: team-a
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin      # Or a custom restricted role
subjects:
- kind: ServiceAccount
  name: team-a
  namespace: team-a
```

### Step 2: Tenant GitRepository

```yaml
# tenants/team-a/source.yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: team-a
  namespace: team-a             # In tenant's namespace
spec:
  interval: 1m
  url: https://github.com/myorg/team-a-config.git
  ref:
    branch: main
  secretRef:
    name: team-a-git-auth
```

### Step 3: Tenant Kustomization with Service Account

```yaml
# tenants/team-a/kustomization.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: team-a-apps
  namespace: team-a             # In tenant's namespace
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: team-a
  path: ./apps
  prune: true
  targetNamespace: team-a       # Force all resources to this namespace
  serviceAccountName: team-a    # Impersonate tenant SA
  # This SA only has permissions in team-a namespace
```

---

## ServiceAccount Impersonation

```
┌──────────────────────────────────────────────────────────┐
│         SERVICE ACCOUNT IMPERSONATION                    │
│                                                          │
│  Without impersonation:                                  │
│  Kustomize Controller uses its own SA (cluster-admin)    │
│  → Tenant could deploy to ANY namespace                  │
│                                                          │
│  With impersonation:                                     │
│  Kustomize Controller impersonates tenant's SA           │
│  → Tenant can only deploy where SA has permissions       │
│                                                          │
│  ┌──────────────────┐     ┌──────────────────┐          │
│  │ Kustomization    │     │ K8s API Server   │          │
│  │ (team-a-apps)    │────→│                  │          │
│  │                  │     │ "Acting as"      │          │
│  │ serviceAccount:  │     │ team-a SA        │          │
│  │   team-a         │     │ (team-a ns only) │          │
│  └──────────────────┘     └──────────────────┘          │
│                                                          │
│  If team-a tries to deploy to team-b namespace →         │
│  FORBIDDEN (SA has no permissions there)                 │
└──────────────────────────────────────────────────────────┘
```

---

## Platform Team Repository Structure

```
fleet-infra/
├── clusters/
│   └── production/
│       ├── flux-system/              # Flux components
│       │   ├── gotk-components.yaml
│       │   └── gotk-sync.yaml
│       ├── infrastructure.yaml       # Infra Kustomization
│       └── tenants.yaml              # Tenants Kustomization
├── infrastructure/
│   ├── cert-manager/
│   ├── ingress-nginx/
│   └── monitoring/
└── tenants/
    ├── base/
    │   ├── namespace.yaml            # Template namespace
    │   ├── rbac.yaml                 # Template RBAC
    │   └── kustomization.yaml
    ├── team-a/
    │   ├── kustomization.yaml        # Kustomize overlay
    │   ├── source.yaml               # GitRepository
    │   └── sync.yaml                 # Kustomization
    ├── team-b/
    │   ├── kustomization.yaml
    │   ├── source.yaml
    │   └── sync.yaml
    └── team-c/
        ├── kustomization.yaml
        ├── source.yaml
        └── sync.yaml
```

### tenants.yaml (Platform Team's Entry Point)

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: tenants
  namespace: flux-system
spec:
  interval: 10m
  dependsOn:
  - name: infrastructure
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./tenants
  prune: true
```

---

## Restricting Tenant Permissions

### Custom ClusterRole for Tenants

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: tenant-reconciler
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets", "services", "serviceaccounts"]
  verbs: ["*"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["*"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["*"]
# Notably MISSING:
# - No Namespace creation
# - No ClusterRole/ClusterRoleBinding
# - No PersistentVolume (cluster-scoped)
# - No CustomResourceDefinition
```

### Network Policies for Tenant Isolation

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-cross-tenant
  namespace: team-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          toolkit.fluxcd.io/tenant: team-a
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          toolkit.fluxcd.io/tenant: team-a
  - to:   # Allow DNS
    - namespaceSelector: {}
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - port: 53
      protocol: UDP
```

---

## Cross-Namespace References

By default, Flux restricts cross-namespace source references. Enable selectively:

```yaml
# Allow team-a to use a HelmRepository from flux-system
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: shared-charts
  namespace: flux-system
spec:
  interval: 30m
  url: https://charts.example.com
  accessFrom:
    namespaceSelectors:
    - matchLabels:
        toolkit.fluxcd.io/tenant: team-a
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Tenant isolation | Separate namespace, ServiceAccount, GitRepository per tenant |
| SA impersonation | `serviceAccountName` in Kustomization limits permissions |
| Platform team | Manages tenants directory, infrastructure, Flux config |
| Tenant team | Manages their own Git repo with application configs |
| Cross-namespace refs | Controlled via `accessFrom` on source resources |
| Network isolation | NetworkPolicies prevent cross-tenant traffic |
| Custom ClusterRole | Restrict what resources tenants can create |

---

## Quick Revision Questions

1. **How does Flux enforce multi-tenancy?**
   > Through ServiceAccount impersonation in Kustomizations — each tenant's Kustomization runs as a tenant-scoped SA that only has permissions in their namespace.

2. **What happens if a tenant tries to deploy to another team's namespace?**
   > The Kubernetes API rejects the request because the impersonated ServiceAccount lacks permissions in the other namespace.

3. **Who manages the tenant configurations in the fleet repo?**
   > The platform team manages the `tenants/` directory (namespaces, RBAC, source references). Each tenant team manages their own Git repository with application configs.

4. **How do you share a HelmRepository across tenants?**
   > Add `accessFrom.namespaceSelectors` to the HelmRepository to allow specific tenant namespaces to reference it.

5. **What additional isolation should you add beyond RBAC?**
   > NetworkPolicies for network isolation, ResourceQuotas for resource limits, and PodSecurityStandards for workload security.

---

[← Previous: Image Automation](05-image-automation.md) | [Back to README](../README.md) | [Next → Application Deployment](../05-GitOps-Workflows/01-application-deployment.md)
