# Chapter 7.2: RBAC Implementation

## Overview

Role-Based Access Control (RBAC) is the primary authorization mechanism in Kubernetes. It determines who can perform which actions on which resources. Proper RBAC implementation follows the principle of least privilege, preventing unauthorized access and limiting blast radius when a service account or user is compromised.

---

## RBAC Architecture

```
KUBERNETES RBAC MODEL:

  WHO                    WHAT                   WHERE
  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
  │ Subjects     │      │ Rules        │      │ Resources    │
  │              │      │              │      │              │
  │ • Users      │      │ • Verbs:     │      │ • pods       │
  │ • Groups     │ ─────│   get, list, │──────│ • services   │
  │ • Service    │      │   create,    │      │ • deployments│
  │   Accounts   │      │   delete,    │      │ • secrets    │
  │              │      │   update,    │      │ • configmaps │
  │              │      │   patch,     │      │ • nodes      │
  │              │      │   watch      │      │              │
  └──────────────┘      └──────────────┘      └──────────────┘
         │                      │
         │        ┌─────────────┘
         │        │
         ▼        ▼
  ┌──────────────────────┐     ┌──────────────────────┐
  │ RoleBinding          │     │ ClusterRoleBinding    │
  │ (namespace-scoped)   │     │ (cluster-wide)        │
  │                      │     │                       │
  │ Binds Role/CR to     │     │ Binds ClusterRole to  │
  │ Subject in a         │     │ Subject across ALL    │
  │ specific namespace   │     │ namespaces            │
  └──────────────────────┘     └──────────────────────┘
         │                              │
         ▼                              ▼
  ┌──────────────┐              ┌──────────────┐
  │ Role         │              │ ClusterRole  │
  │ (namespace)  │              │ (cluster)    │
  └──────────────┘              └──────────────┘
```

---

## Role vs ClusterRole

```yaml
# Role — namespace-scoped permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: pod-reader
rules:
  - apiGroups: [""]            # Core API group
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get"]

---
# ClusterRole — cluster-wide or reusable across namespaces
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get", "list"]
    # NO "create", "update", "delete" — read-only!
```

---

## RoleBinding vs ClusterRoleBinding

```yaml
# RoleBinding — grants Role to subject in ONE namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
  - kind: User
    name: developer@company.com
    apiGroup: rbac.authorization.k8s.io
  - kind: Group
    name: dev-team
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

---
# ClusterRoleBinding — grants ClusterRole across ALL namespaces
# WARNING: Use sparingly!
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
subjects:
  - kind: Group
    name: platform-admins
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin         # Built-in admin role
  apiGroup: rbac.authorization.k8s.io
```

---

## Common RBAC Patterns

### Developer Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: developer
rules:
  # Read pods, logs, events
  - apiGroups: [""]
    resources: ["pods", "pods/log", "events", "services", "configmaps"]
    verbs: ["get", "list", "watch"]
  # Manage deployments
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
  # Port-forward for debugging
  - apiGroups: [""]
    resources: ["pods/portforward"]
    verbs: ["create"]
  # NO access to: secrets, exec, delete pods, cluster resources
```

### CI/CD Service Account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cicd-deployer
  namespace: production

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: deployer
rules:
  # Deploy and rollback
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "update", "patch"]
  - apiGroups: ["apps"]
    resources: ["deployments/rollback"]
    verbs: ["create"]
  # Update configmaps (not secrets directly)
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "update", "patch"]
  # Check pod status
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: production
  name: cicd-deployer-binding
subjects:
  - kind: ServiceAccount
    name: cicd-deployer
    namespace: production
roleRef:
  kind: Role
  name: deployer
  apiGroup: rbac.authorization.k8s.io
```

### Read-Only Auditor

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: auditor
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["get", "list", "watch"]
  # Explicitly deny secrets (handled by NOT including)
  # But we need to exclude secrets from the wildcard:
---
# Better approach: explicit resource list (no wildcards)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: auditor-safe
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps", "namespaces", "nodes", "events"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "statefulsets", "daemonsets"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["networkpolicies", "ingresses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["rbac.authorization.k8s.io"]
    resources: ["roles", "rolebindings", "clusterroles", "clusterrolebindings"]
    verbs: ["get", "list", "watch"]
  # NO secrets, NO exec, NO delete
```

---

## Service Account Security

```yaml
# 1. Disable automounting service account token
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  namespace: production
automountServiceAccountToken: false    # Don't mount token unless needed

---
# 2. Pod-level override (when token IS needed)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      serviceAccountName: my-app
      automountServiceAccountToken: true   # Explicit opt-in
      containers:
        - name: app
          image: myapp:1.0.0

---
# 3. Don't use the default service account
# The default SA in each namespace has no permissions by default
# but CAN be misconfigured. Always create dedicated SAs.
```

```
SERVICE ACCOUNT BEST PRACTICES:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  ✓  Create dedicated SA per workload               │
  │  ✓  Disable automountServiceAccountToken           │
  │  ✓  Use namespace-scoped Roles (not ClusterRoles)  │
  │  ✓  Never bind to cluster-admin                    │
  │  ✓  Rotate tokens (use bound SA tokens in v1.24+)  │
  │  ✓  Audit SA permissions with kubectl              │
  │                                                    │
  │  ✗  Don't use default service account              │
  │  ✗  Don't use wildcards in resources/verbs         │
  │  ✗  Don't grant exec/attach to service accounts    │
  │  ✗  Don't share SAs across workloads               │
  └──────────────────────────────────────────────────┘
```

---

## RBAC Auditing

```bash
# Check who can do what
# Who can create pods in production?
kubectl auth can-i create pods -n production --as=developer@company.com

# What can a service account do?
kubectl auth can-i --list --as=system:serviceaccount:production:cicd-deployer -n production

# Find all ClusterRoleBindings to cluster-admin (high risk!)
kubectl get clusterrolebindings -o json | jq '.items[] | 
  select(.roleRef.name == "cluster-admin") | 
  {name: .metadata.name, subjects: .subjects}'

# Find all service accounts with secrets access
kubectl get rolebindings,clusterrolebindings -A -o json | jq '.items[] |
  select(.roleRef.name != null) |
  {name: .metadata.name, namespace: .metadata.namespace, role: .roleRef.name, subjects: .subjects}'

# List all roles with wildcard permissions (dangerous!)
kubectl get roles,clusterroles -A -o json | jq '.items[] |
  select(.rules[]?.resources[]? == "*" or .rules[]?.verbs[]? == "*") |
  {name: .metadata.name, namespace: .metadata.namespace}'
```

### RBAC Visualization with rakkess

```bash
# Install rakkess (RBAC access matrix)
kubectl krew install access-matrix

# Show what current user can do
kubectl access-matrix

# Show what a service account can do
kubectl access-matrix --as=system:serviceaccount:production:cicd-deployer -n production

# Output:
# NAME                  GET  LIST  WATCH  CREATE  UPDATE  DELETE
# configmaps             ✔    ✔     ✔      ✖       ✔       ✖
# deployments.apps       ✔    ✔     ✖      ✖       ✔       ✖
# pods                   ✔    ✔     ✔      ✖       ✖       ✖
# secrets                ✖    ✖     ✖      ✖       ✖       ✖
```

---

## Common RBAC Mistakes

```
RBAC ANTI-PATTERNS:

  ┌─────────────────────────────────────────────────────────┐
  │                                                          │
  │  ✗ MISTAKE 1: Wildcard everything                        │
  │  rules:                                                  │
  │    - apiGroups: ["*"]                                    │
  │      resources: ["*"]                                    │
  │      verbs: ["*"]           ← This is cluster-admin!    │
  │                                                          │
  │  ✗ MISTAKE 2: ClusterRoleBinding when Role suffices      │
  │  kind: ClusterRoleBinding   ← Grants across ALL NS      │
  │                                                          │
  │  ✗ MISTAKE 3: Granting exec to service accounts          │
  │  resources: ["pods/exec"]                                │
  │  verbs: ["create"]         ← Attacker can exec into pod │
  │                                                          │
  │  ✗ MISTAKE 4: Not auditing inherited permissions         │
  │  A SA with access to Secrets can read DB credentials     │
  │                                                          │
  │  ✗ MISTAKE 5: Too many cluster-admin bindings            │
  │  Every team lead has cluster-admin "for convenience"     │
  └─────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `Forbidden: User cannot list pods` | Missing RoleBinding for user/group | Create RoleBinding linking user to appropriate Role |
| Service account can access secrets | Overly broad Role with `resources: ["*"]` | Use explicit resource list without wildcards |
| `kubectl auth can-i` shows different from actual | Checking wrong namespace or SA | Specify `--as` and `-n` correctly |
| CI/CD deployment fails with 403 | SA missing `update` verb on deployments | Add `update` and `patch` verbs for `deployments` resource |
| Multiple teams conflicting in same namespace | Shared namespace RBAC | Use separate namespaces per team with dedicated Roles |

---

## Summary Table

| Concept | Scope | Purpose |
|---------|-------|---------|
| **Role** | Namespace | Define permissions within a single namespace |
| **ClusterRole** | Cluster | Define permissions across all namespaces or non-namespaced resources |
| **RoleBinding** | Namespace | Grant a Role to subjects within one namespace |
| **ClusterRoleBinding** | Cluster | Grant a ClusterRole to subjects across all namespaces |
| **ServiceAccount** | Namespace | Identity for pods; should be per-workload |
| **automountServiceAccountToken** | Pod/SA | Disable token mounting unless explicitly needed |

---

## Quick Revision Questions

1. **What is the difference between a Role and a ClusterRole?**
   > A Role grants permissions within a specific namespace. A ClusterRole grants permissions across all namespaces (when used with ClusterRoleBinding) or for non-namespaced resources (nodes, PVs). ClusterRoles can also be bound to a single namespace via RoleBinding, making them reusable templates.

2. **Why should you avoid wildcards in RBAC rules?**
   > Wildcards (`*`) in resources or verbs grant access to everything — including secrets, exec, and deletion rights. As new resource types are added to the cluster, wildcards automatically include them. This violates least privilege and makes it hard to audit actual permissions.

3. **What is the risk of using the default service account?**
   > The default SA exists in every namespace and is automatically mounted in pods that don't specify a SA. If someone adds permissions to the default SA, all pods in that namespace inherit those permissions. Always create dedicated SAs per workload.

4. **How do you check what permissions a service account has?**
   > Use `kubectl auth can-i --list --as=system:serviceaccount:NAMESPACE:SA_NAME -n NAMESPACE`. For a visual matrix, use `kubectl access-matrix` (rakkess). Also audit RoleBindings and ClusterRoleBindings that reference the SA.

5. **Why should `automountServiceAccountToken` be disabled by default?**
   > The mounted token allows any process in the pod to authenticate to the Kubernetes API. If an attacker compromises the application, they can use this token to interact with the cluster. Most applications don't need API access, so disabling it by default reduces attack surface.

6. **What is the difference between RoleBinding and ClusterRoleBinding?**
   > A RoleBinding grants permissions in a single namespace — even if it references a ClusterRole, access is limited to that namespace. A ClusterRoleBinding grants permissions across ALL namespaces. Always prefer RoleBinding unless cluster-wide access is genuinely needed.

---

[← Previous: 7.1 Cluster Hardening](01-cluster-hardening.md) | [Next: 7.3 Network Policies →](03-network-policies.md)

[Back to Table of Contents](../README.md)
