# Unit 9: Security

[← Previous: Scheduling](../08-Scheduling/01-scheduling.md) | [Back to README](../README.md) | [Next: Observability →](../10-Observability/01-observability.md)

---

## Chapter Overview

Kubernetes security spans multiple layers: cluster infrastructure, API access, workload isolation, and network segmentation. This unit covers **RBAC**, **Service Accounts**, **Pod Security Standards**, **Security Contexts**, **Network Policies**, **Secrets management**, and **image security** — the essential knowledge for securing production clusters.

---

## 9.1 The 4C's of Cloud Native Security

```
┌──────────────────────────────────────────────────────────────┐
│              THE 4C's OF SECURITY                             │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ CLOUD (Infrastructure)                               │    │
│  │  - Cloud provider security (IAM, VPC, firewalls)     │    │
│  │  - Physical security, network perimeter              │    │
│  │                                                      │    │
│  │  ┌──────────────────────────────────────────────┐    │    │
│  │  │ CLUSTER                                      │    │    │
│  │  │  - API server auth, RBAC, admission control  │    │    │
│  │  │  - etcd encryption, audit logging            │    │    │
│  │  │  - Network policies, pod security            │    │    │
│  │  │                                              │    │    │
│  │  │  ┌──────────────────────────────────────┐    │    │    │
│  │  │  │ CONTAINER                            │    │    │    │
│  │  │  │  - Image scanning, signed images     │    │    │    │
│  │  │  │  - Minimal base images, no root      │    │    │    │
│  │  │  │  - Read-only filesystem              │    │    │    │
│  │  │  │                                      │    │    │    │
│  │  │  │  ┌──────────────────────────────┐    │    │    │    │
│  │  │  │  │ CODE                         │    │    │    │    │
│  │  │  │  │  - Input validation          │    │    │    │    │
│  │  │  │  │  - Dependency scanning       │    │    │    │    │
│  │  │  │  │  - Secret handling           │    │    │    │    │
│  │  │  │  │  - TLS everywhere            │    │    │    │    │
│  │  │  │  └──────────────────────────────┘    │    │    │    │
│  │  │  └──────────────────────────────────────┘    │    │    │
│  │  └──────────────────────────────────────────────┘    │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 9.2 Authentication and Authorization Flow

```
┌──────────────────────────────────────────────────────────────┐
│           API REQUEST FLOW                                    │
│                                                              │
│  kubectl / API call                                          │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────────────┐                                        │
│  │ 1. AUTHENTICATION│  WHO are you?                          │
│  │    (AuthN)       │  - Client certificates (x509)          │
│  │                  │  - Bearer tokens                       │
│  │                  │  - ServiceAccount tokens                │
│  │                  │  - OIDC (e.g., Google, Azure AD)       │
│  └────────┬─────────┘                                        │
│           │ Identity confirmed                               │
│           ▼                                                  │
│  ┌──────────────────┐                                        │
│  │ 2. AUTHORIZATION │  WHAT can you do?                      │
│  │    (AuthZ)       │  - RBAC (Role-Based Access Control)    │
│  │                  │  - ABAC (Attribute-Based)              │
│  │                  │  - Webhook                             │
│  │                  │  - Node authorization                  │
│  └────────┬─────────┘                                        │
│           │ Action permitted                                 │
│           ▼                                                  │
│  ┌──────────────────┐                                        │
│  │ 3. ADMISSION     │  Should we MODIFY or REJECT?           │
│  │    CONTROL       │  - Mutating webhooks                   │
│  │                  │  - Validating webhooks                 │
│  │                  │  - Built-in controllers                │
│  └────────┬─────────┘                                        │
│           │ Request approved                                 │
│           ▼                                                  │
│  ┌──────────────────┐                                        │
│  │ 4. PERSIST       │  Write to etcd                         │
│  └──────────────────┘                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 9.3 RBAC (Role-Based Access Control)

RBAC controls **who** can do **what** on **which resources** in Kubernetes.

```
┌──────────────────────────────────────────────────────────────┐
│                 RBAC COMPONENTS                               │
│                                                              │
│  ┌─────────────┐     ┌──────────────┐    ┌──────────────┐   │
│  │   Subject    │     │  RoleBinding  │    │    Role      │   │
│  │             │────▶│              │───▶│             │   │
│  │ - User      │     │ Binds subject│    │ Defines     │   │
│  │ - Group     │     │ to role      │    │ permissions │   │
│  │ - Service   │     │              │    │ (verbs +    │   │
│  │   Account   │     │              │    │  resources) │   │
│  └─────────────┘     └──────────────┘    └──────────────┘   │
│                                                              │
│  NAMESPACE-SCOPED:                                           │
│  Role + RoleBinding = permissions within ONE namespace       │
│                                                              │
│  CLUSTER-SCOPED:                                             │
│  ClusterRole + ClusterRoleBinding = permissions cluster-wide │
│                                                              │
│  ┌────────────────┐  ┌─────────────────┐                     │
│  │ Role           │  │ ClusterRole     │                     │
│  │ (namespace)    │  │ (cluster-wide)  │                     │
│  ├────────────────┤  ├─────────────────┤                     │
│  │ RoleBinding    │  │ClusterRole-     │                     │
│  │ (namespace)    │  │   Binding       │                     │
│  │                │  │ (cluster-wide)  │                     │
│  └────────────────┘  └─────────────────┘                     │
│                                                              │
│  Tip: ClusterRole + RoleBinding = reuse ClusterRole          │
│       permissions in a specific namespace                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Role (Namespace-Scoped)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
- apiGroups: [""]              # "" = core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

- apiGroups: [""]
  resources: ["pods/log"]      # Sub-resource
  verbs: ["get"]

- apiGroups: ["apps"]          # apps API group
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
```

### ClusterRole (Cluster-Scoped)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]           # Cluster-scoped resource
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get", "list"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["nodes", "pods"]
  verbs: ["get", "list"]
```

### RoleBinding and ClusterRoleBinding

```yaml
# Namespace-scoped binding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-dev
  namespace: dev
subjects:
- kind: User
  name: jane                    # User name (from certificate CN)
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: developers              # Group name
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: monitoring-sa           # ServiceAccount in namespace
  namespace: monitoring
roleRef:
  kind: Role                    # or ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
---
# Cluster-scoped binding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: global-node-reader
subjects:
- kind: Group
  name: ops-team
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

### RBAC Verbs Reference

```
┌──────────────────────────────────────────────────────────────┐
│                    RBAC VERBS                                 │
│                                                              │
│  ┌──────────┬──────────────────────────────────────┐         │
│  │ Verb     │ Description                          │         │
│  ├──────────┼──────────────────────────────────────┤         │
│  │ get      │ Read a single resource               │         │
│  │ list     │ List all resources of a type          │         │
│  │ watch    │ Watch for changes (streaming)         │         │
│  │ create   │ Create a new resource                 │         │
│  │ update   │ Replace entire resource               │         │
│  │ patch    │ Modify fields of a resource           │         │
│  │ delete   │ Delete a single resource              │         │
│  │ deletecollection │ Delete multiple resources     │         │
│  │ impersonate │ Act as another user/group          │         │
│  │ bind     │ Bind to a Role/ClusterRole            │         │
│  │ escalate │ Allow escalation of roles             │         │
│  │ *        │ All verbs (wildcard)                  │         │
│  └──────────┴──────────────────────────────────────┘         │
│                                                              │
│  Common Patterns:                                            │
│  Read-only:  ["get", "list", "watch"]                        │
│  Read-write: ["get", "list", "watch", "create", "update",   │
│               "patch", "delete"]                             │
│  Admin:      ["*"]                                           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### RBAC Commands

```bash
# Check permissions
kubectl auth can-i create pods --namespace dev          # For yourself
kubectl auth can-i create pods --as jane --namespace dev # For another user
kubectl auth can-i '*' '*'                              # Am I cluster-admin?
kubectl auth can-i list secrets --as system:serviceaccount:dev:my-sa

# List RBAC resources
kubectl get roles -n dev
kubectl get rolebindings -n dev
kubectl get clusterroles
kubectl get clusterrolebindings

# Create imperatively
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n dev
kubectl create rolebinding read-pods --role=pod-reader --user=jane -n dev
kubectl create clusterrole node-reader --verb=get,list --resource=nodes
kubectl create clusterrolebinding read-nodes --clusterrole=node-reader --group=ops

# View who has access
kubectl describe rolebinding read-pods -n dev
```

---

## 9.4 Service Accounts

Service Accounts provide identity for **pods** (processes in containers). Every pod runs under a ServiceAccount.

```
┌──────────────────────────────────────────────────────────────┐
│              SERVICE ACCOUNTS                                 │
│                                                              │
│  ┌─── Namespace: production ──────────────────────┐          │
│  │                                                │          │
│  │  ServiceAccount: default                       │          │
│  │  ├── Auto-created in every namespace           │          │
│  │  └── Used by pods with no explicit SA          │          │
│  │                                                │          │
│  │  ServiceAccount: app-sa                        │          │
│  │  ├── Custom SA for app pods                    │          │
│  │  ├── Has specific RBAC permissions             │          │
│  │  └── Token auto-mounted (unless disabled)      │          │
│  │                                                │          │
│  │  Pod ──▶ ServiceAccount ──▶ RoleBinding ──▶ Role│         │
│  │                                                │          │
│  └────────────────────────────────────────────────┘          │
│                                                              │
│  Token location in pod:                                      │
│  /var/run/secrets/kubernetes.io/serviceaccount/               │
│  ├── token      (JWT token)                                  │
│  ├── ca.crt     (cluster CA certificate)                     │
│  └── namespace  (current namespace)                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
# Create ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: production
automountServiceAccountToken: true   # Default: true
---
# Use in Pod
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: production
spec:
  serviceAccountName: app-sa        # Use custom SA
  automountServiceAccountToken: false  # Disable token mount (if not needed)
  containers:
  - name: app
    image: my-app:1.0
---
# Bind Role to ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-sa-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: production
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
# ServiceAccount commands
kubectl create serviceaccount app-sa -n production
kubectl get serviceaccounts -n production
kubectl describe serviceaccount app-sa -n production

# Create token (K8s 1.24+, TokenRequest API)
kubectl create token app-sa -n production --duration=1h
```

### Best Practice: Disable Default SA Token

```yaml
# Disable auto-mount for the default SA in a namespace
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
  namespace: production
automountServiceAccountToken: false
```

---

## 9.5 Pod Security Standards (PSS)

Pod Security Standards replace the deprecated PodSecurityPolicy (PSP). They define three security levels enforced via the **Pod Security Admission** controller.

```
┌──────────────────────────────────────────────────────────────┐
│           POD SECURITY STANDARDS                              │
│                                                              │
│  ┌─────────────┬──────────────────────────────────────────┐  │
│  │  Level      │  Description                             │  │
│  ├─────────────┼──────────────────────────────────────────┤  │
│  │ Privileged  │  Unrestricted. No security restrictions. │  │
│  │             │  For system/infrastructure pods.         │  │
│  ├─────────────┼──────────────────────────────────────────┤  │
│  │ Baseline    │  Minimally restrictive. Prevents known   │  │
│  │             │  privilege escalations. Allows most      │  │
│  │             │  common workloads.                       │  │
│  ├─────────────┼──────────────────────────────────────────┤  │
│  │ Restricted  │  Heavily restricted. Best practices.     │  │
│  │             │  Non-root, read-only FS, drop caps.      │  │
│  └─────────────┴──────────────────────────────────────────┘  │
│                                                              │
│  Modes (per namespace):                                      │
│  ┌──────────┬──────────────────────────────────────────┐     │
│  │ enforce  │ Reject pods that violate the standard    │     │
│  │ audit    │ Log violations (but allow pods)          │     │
│  │ warn     │ Show warnings to user (but allow pods)   │     │
│  └──────────┴──────────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Applying PSS via Namespace Labels

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    # Enforce restricted standard
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: v1.28

    # Audit baseline violations
    pod-security.kubernetes.io/audit: baseline
    pod-security.kubernetes.io/audit-version: v1.28

    # Warn about baseline violations
    pod-security.kubernetes.io/warn: baseline
    pod-security.kubernetes.io/warn-version: v1.28
```

```bash
# Apply to existing namespace
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/warn=restricted

# Test dry-run (check which pods would fail)
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  --dry-run=server
```

---

## 9.6 Security Contexts

Security Contexts define privilege and access control settings at the **pod** or **container** level.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  # Pod-level security context (applies to all containers)
  securityContext:
    runAsUser: 1000               # Run as non-root UID
    runAsGroup: 3000              # Primary GID
    fsGroup: 2000                 # Volume ownership GID
    runAsNonRoot: true            # Forbid running as root
    seccompProfile:               # Seccomp profile
      type: RuntimeDefault
    supplementalGroups: [4000]    # Additional GIDs

  containers:
  - name: app
    image: my-app:1.0

    # Container-level security context (overrides pod-level)
    securityContext:
      runAsUser: 1000
      runAsNonRoot: true
      readOnlyRootFilesystem: true     # Read-only root FS
      allowPrivilegeEscalation: false  # Block setuid/setgid
      capabilities:
        drop:
        - ALL                          # Drop all Linux capabilities
        add:
        - NET_BIND_SERVICE             # Only add what's needed

    # Writable directories via emptyDir volumes
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /app/cache

  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
```

```
┌──────────────────────────────────────────────────────────────┐
│          KEY SECURITY CONTEXT FIELDS                          │
│                                                              │
│  ┌──────────────────────────┬────────────────────────────┐   │
│  │ Field                    │ Effect                     │   │
│  ├──────────────────────────┼────────────────────────────┤   │
│  │ runAsUser: 1000          │ Container runs as UID 1000 │   │
│  │ runAsNonRoot: true       │ Reject if image runs root  │   │
│  │ readOnlyRootFilesystem   │ Root FS is read-only       │   │
│  │ allowPrivilegeEscalation │ Block setuid binaries      │   │
│  │ capabilities.drop: ALL   │ Remove all Linux caps      │   │
│  │ capabilities.add: [...]  │ Add only needed caps       │   │
│  │ fsGroup: 2000            │ Volumes owned by GID 2000  │   │
│  │ seccompProfile           │ System call filtering      │   │
│  │ privileged: true         │ Full host access (DANGER!) │   │
│  │ hostNetwork: true        │ Use host's network (RISK)  │   │
│  │ hostPID: true            │ See host processes (RISK)  │   │
│  └──────────────────────────┴────────────────────────────┘   │
│                                                              │
│  Best Practice Minimum:                                      │
│  - runAsNonRoot: true                                        │
│  - readOnlyRootFilesystem: true                              │
│  - allowPrivilegeEscalation: false                           │
│  - capabilities.drop: ALL                                    │
│  - No privileged, hostNetwork, hostPID                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 9.7 Network Policies (Security Perspective)

Network Policies act as a **firewall** for pod-to-pod communication. By default, all pods can communicate with all other pods.

```
┌──────────────────────────────────────────────────────────────┐
│         NETWORK POLICY SECURITY                               │
│                                                              │
│  DEFAULT: All pods can talk to all pods (flat network)       │
│                                                              │
│  ┌───────────┐     ┌───────────┐     ┌───────────┐          │
│  │ Frontend  │◄───▶│ Backend   │◄───▶│ Database  │          │
│  │           │     │           │     │           │          │
│  │  ◄───────▶│     │  ◄───────▶│     │           │          │
│  └───────────┘     └───────────┘     └───────────┘          │
│  Everything can talk to everything — BAD!                    │
│                                                              │
│  WITH NETWORK POLICIES:                                      │
│  ┌───────────┐     ┌───────────┐     ┌───────────┐          │
│  │ Frontend  │────▶│ Backend   │────▶│ Database  │          │
│  │           │     │           │     │           │          │
│  │  ✗ ──────▶│     │  ✗ ──────▶│     │           │          │
│  └───────────┘     └───────────┘     └───────────┘          │
│  Frontend → Backend ✓     Backend → Database ✓               │
│  Frontend → Database ✗    External → Database ✗              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Default Deny All (Starting Point)

```yaml
# Deny all ingress traffic in namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}               # Applies to ALL pods
  policyTypes:
  - Ingress                     # No ingress rules = deny all
---
# Deny all egress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-egress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Egress                      # No egress rules = deny all
---
# Deny all traffic (both directions)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### Allow Specific Traffic

```yaml
# Allow frontend to talk to backend on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: backend              # Apply to backend pods
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend         # Only from frontend pods
    ports:
    - protocol: TCP
      port: 8080
---
# Allow backend to talk to database on port 5432
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-db
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - protocol: TCP
      port: 5432
---
# Allow DNS resolution for all pods (essential!)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

---

## 9.8 Secrets Management Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│         SECRETS MANAGEMENT STRATEGY                           │
│                                                              │
│  Level 1: Kubernetes Secrets (basic)                         │
│  ├── Base64 encoded (NOT encrypted)                          │
│  ├── Stored in etcd (enable encryption at rest!)             │
│  └── RBAC to restrict access                                 │
│                                                              │
│  Level 2: Encryption at Rest                                 │
│  ├── EncryptionConfiguration for API server                  │
│  ├── Encrypts secrets in etcd                                │
│  └── Key managed by cluster admin                            │
│                                                              │
│  Level 3: External Secret Managers                           │
│  ├── HashiCorp Vault                                         │
│  ├── AWS Secrets Manager / SSM Parameter Store               │
│  ├── Azure Key Vault                                         │
│  ├── Google Secret Manager                                   │
│  └── Tools: External Secrets Operator, Sealed Secrets        │
│                                                              │
│  Level 4: Sealed Secrets (GitOps-friendly)                   │
│  ├── Encrypt secrets before committing to Git                │
│  ├── Only cluster can decrypt (asymmetric encryption)        │
│  └── Safe to store in version control                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Encryption at Rest Configuration

```yaml
# /etc/kubernetes/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <base64-encoded-32-byte-key>  # Generate with:
        # head -c 32 /dev/urandom | base64
  - identity: {}     # Fallback: unencrypted (for reading old secrets)
```

```bash
# Verify encryption is working
kubectl create secret generic test-enc --from-literal=key=value
# Read directly from etcd — should be encrypted
ETCDCTL_API=3 etcdctl get /registry/secrets/default/test-enc | hexdump -C
```

---

## 9.9 Image Security

```
┌──────────────────────────────────────────────────────────────┐
│            IMAGE SECURITY CHECKLIST                            │
│                                                              │
│  ✓ Use specific image tags (not :latest)                     │
│    image: nginx:1.25.3  ✓                                    │
│    image: nginx:latest  ✗                                    │
│    image: nginx         ✗  (implies :latest)                 │
│                                                              │
│  ✓ Use image digests for immutability                        │
│    image: nginx@sha256:abc123...                             │
│                                                              │
│  ✓ Use minimal base images                                   │
│    - distroless, alpine, scratch                             │
│    - Smaller attack surface                                  │
│                                                              │
│  ✓ Scan images for vulnerabilities                           │
│    - Trivy, Snyk, Grype, Clair                               │
│                                                              │
│  ✓ Use private registries                                    │
│    - Pull from trusted registries only                       │
│    - Use imagePullSecrets                                    │
│                                                              │
│  ✓ Sign and verify images                                    │
│    - Cosign, Notary / TUF                                    │
│    - Admission policies to enforce signatures                │
│                                                              │
│  ✓ Set imagePullPolicy: Always for mutable tags              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Private Registry Authentication

```yaml
# Create registry secret
# kubectl create secret docker-registry regcred \
#   --docker-server=registry.example.com \
#   --docker-username=user \
#   --docker-password=pass

apiVersion: v1
kind: Pod
metadata:
  name: private-app
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: registry.example.com/my-app:1.0
    imagePullPolicy: Always
```

---

## 9.10 Audit Logging

```yaml
# Audit policy — what to log
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
# Don't log read-only requests to certain endpoints
- level: None
  resources:
  - group: ""
    resources: ["endpoints", "services", "services/status"]

# Log Secret access at Metadata level (no body)
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]

# Log all other resources at Request level
- level: Request
  resources:
  - group: ""
  - group: "apps"
  - group: "batch"
```

```
┌──────────────────────────────────────────────────────────────┐
│  Audit Levels:                                               │
│  ┌────────────┬──────────────────────────────────────────┐   │
│  │ None       │ Don't log this event                     │   │
│  │ Metadata   │ Log request metadata (user, time, verb)  │   │
│  │ Request    │ Log metadata + request body              │   │
│  │ RequestResponse │ Log metadata + request + response   │   │
│  └────────────┴──────────────────────────────────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Securing a Multi-Team Cluster

```yaml
# Namespace for Team A with security policies
apiVersion: v1
kind: Namespace
metadata:
  name: team-a
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/warn: restricted
---
# ServiceAccount for Team A's apps
apiVersion: v1
kind: ServiceAccount
metadata:
  name: team-a-app
  namespace: team-a
automountServiceAccountToken: false
---
# Role: Team A can manage their own resources
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: team-a-developer
  namespace: team-a
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "deployments", "services", "configmaps", "jobs"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods/log", "pods/exec"]
  verbs: ["get", "create"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]        # Read secrets, but not create/update
---
# Bind role to Team A group
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-a-developers-binding
  namespace: team-a
subjects:
- kind: Group
  name: team-a-developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: team-a-developer
  apiGroup: rbac.authorization.k8s.io
---
# Network: Default deny + allow required traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: team-a
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-and-internal
  namespace: team-a
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: team-a
---
# ResourceQuota for Team A
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-quota
  namespace: team-a
spec:
  hard:
    requests.cpu: "8"
    requests.memory: "16Gi"
    limits.cpu: "16"
    limits.memory: "32Gi"
    pods: "30"
```

---

## Troubleshooting Tips

| Issue | Investigation | Solution |
|-------|---------------|---------|
| "Forbidden" error on API call | `kubectl auth can-i <verb> <resource>` | Add missing Role/RoleBinding/ClusterRole |
| Pod rejected by PSS | Check namespace labels and pod securityContext | Adjust pod spec to meet the security standard |
| ServiceAccount token not mounted | Check `automountServiceAccountToken` | Set to `true` on SA or Pod if needed |
| NetworkPolicy not working | Check if CNI supports policies (e.g., Calico, Cilium) | Flannel does NOT support NetworkPolicies |
| Secret visible in plain text | `kubectl get secret -o yaml` | Enable encryption at rest for etcd |
| Can't pull from private registry | Check imagePullSecrets | Ensure Secret exists in same namespace as Pod |

```bash
# Security debugging commands
kubectl auth can-i --list                        # What can I do?
kubectl auth can-i --list --as jane              # What can jane do?
kubectl get networkpolicy -n production          # List policies
kubectl describe networkpolicy <name>            # Policy details
kubectl get psp                                  # PodSecurityPolicies (deprecated)
kubectl label ns production --list               # Check PSS labels
kubectl exec <pod> -- id                         # Check running user
kubectl exec <pod> -- cat /proc/1/status         # Process details
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Authentication** | Identity verification: certificates, tokens, OIDC |
| **Authorization (RBAC)** | Role-based permissions: verbs + resources |
| **Role / RoleBinding** | Namespace-scoped permissions |
| **ClusterRole / ClusterRoleBinding** | Cluster-wide permissions |
| **Service Accounts** | Pod identity for API access; disable token mount when unnecessary |
| **Pod Security Standards** | Privileged / Baseline / Restricted (enforced per namespace) |
| **Security Context** | Pod/container privilege settings (runAsNonRoot, drop caps, readOnlyFS) |
| **Network Policies** | Pod-level firewall rules (requires CNI support) |
| **Secrets Encryption** | Enable encryption at rest for etcd; use external managers |
| **Image Security** | Pin versions, scan, sign, use minimal base images |
| **Audit Logging** | Track who accessed what, when |
| **Admission Controllers** | Mutate/validate requests before persistence |

---

## Quick Revision Questions

1. **What are the four RBAC resources and how do they relate to each other?**
   <details><summary>Answer</summary>1) **Role** — defines permissions (verbs + resources) within a namespace. 2) **ClusterRole** — defines permissions cluster-wide (or reusable across namespaces). 3) **RoleBinding** — binds a Role or ClusterRole to subjects (users/groups/SAs) within a namespace. 4) **ClusterRoleBinding** — binds a ClusterRole to subjects cluster-wide. A Role + RoleBinding grants namespace-scoped access. A ClusterRole + ClusterRoleBinding grants cluster-wide access. A ClusterRole + RoleBinding grants the ClusterRole's permissions only within that namespace.</details>

2. **What command checks if user "jane" can delete pods in the "dev" namespace?**
   <details><summary>Answer</summary>`kubectl auth can-i delete pods --as jane -n dev` — Returns "yes" or "no". The `--as` flag uses impersonation (requires impersonate permission). To see all permissions: `kubectl auth can-i --list --as jane -n dev`.</details>

3. **What are the three Pod Security Standards levels, and which is most restrictive?**
   <details><summary>Answer</summary>1) **Privileged** — no restrictions (system-level pods). 2) **Baseline** — prevents known privilege escalations (blocks hostNetwork, privileged containers, etc.). 3) **Restricted** — most restrictive: requires non-root, drop all capabilities, read-only root FS, seccomp profile. Applied via namespace labels (`pod-security.kubernetes.io/enforce: restricted`).</details>

4. **Why should you set `automountServiceAccountToken: false` on pods that don't need API access?**
   <details><summary>Answer</summary>If a pod is compromised, the mounted SA token gives the attacker access to the Kubernetes API with whatever permissions that ServiceAccount has. Disabling the auto-mount reduces the attack surface. Only pods that actually need to interact with the K8s API should have tokens mounted.</details>

5. **What's the first step when implementing NetworkPolicies in a namespace?**
   <details><summary>Answer</summary>Create a **default deny all** policy: `podSelector: {}` with `policyTypes: [Ingress, Egress]` and no ingress/egress rules. This blocks all traffic by default. Then create specific allow policies for required communication paths. Also ensure your CNI plugin supports NetworkPolicies (Calico, Cilium do; Flannel does not).</details>

6. **How does Kubernetes encrypt Secrets, and why is the default insufficient?**
   <details><summary>Answer</summary>By default, Secrets are stored as base64 in etcd — which is encoding, NOT encryption. Anyone with etcd access can decode them. To encrypt: create an EncryptionConfiguration with a provider (aescbc, aesgcm, or kms) and pass it to the API server via `--encryption-provider-config`. For production, use KMS providers (AWS KMS, Azure Key Vault, GCP KMS) or external secret managers like HashiCorp Vault.</details>

---

[← Previous: Scheduling](../08-Scheduling/01-scheduling.md) | [Back to README](../README.md) | [Next: Observability →](../10-Observability/01-observability.md)
