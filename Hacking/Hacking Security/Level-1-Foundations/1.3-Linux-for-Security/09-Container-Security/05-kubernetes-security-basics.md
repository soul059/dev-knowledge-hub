# Kubernetes Security Basics

## Unit 9 - Topic 5 | Container Security

---

## Overview

**Kubernetes (K8s)** is the dominant container orchestration platform. Its complexity introduces a vast attack surface — from the **API server** and **etcd** to **pod configurations** and **network policies**. Understanding Kubernetes security is essential as organizations increasingly deploy containerized workloads.

---

## 1. Kubernetes Architecture & Attack Surface

```
┌──────────────────────────────────────────────────────────────┐
│              KUBERNETES ATTACK SURFACE                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CONTROL PLANE:                                              │
│  ┌──────────────────────────────────────┐                   │
│  │ API Server (kube-apiserver)          │ ← Authentication  │
│  │ etcd (cluster state/secrets)        │ ← Encryption      │
│  │ Scheduler                            │ ← Authorization   │
│  │ Controller Manager                   │ ← Admission Ctrl  │
│  └──────────────────────────────────────┘                   │
│                                                              │
│  WORKER NODES:                                               │
│  ┌──────────────────────────────────────┐                   │
│  │ kubelet (node agent)                │ ← Node security   │
│  │ kube-proxy (networking)              │ ← Network policy  │
│  │ Container Runtime                    │ ← Container sec   │
│  │ ┌────────┐ ┌────────┐ ┌────────┐   │                   │
│  │ │ Pod    │ │ Pod    │ │ Pod    │   │ ← Pod security    │
│  │ └────────┘ └────────┘ └────────┘   │                   │
│  └──────────────────────────────────────┘                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Pod Security

```yaml
# === INSECURE POD ===
apiVersion: v1
kind: Pod
metadata:
  name: insecure-pod
spec:
  containers:
  - name: app
    image: myapp:latest           # ❌ Mutable tag
    securityContext:
      privileged: true            # ❌ Full host access
      runAsUser: 0                # ❌ Running as root

# === SECURE POD ===
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true            # ✅ Enforce non-root
    runAsUser: 1000               # ✅ Specific UID
    runAsGroup: 1000              # ✅ Specific GID
    fsGroup: 1000                 # ✅ Volume ownership
  containers:
  - name: app
    image: myapp:v1.2.3@sha256:abc...  # ✅ Pinned + digest
    securityContext:
      allowPrivilegeEscalation: false  # ✅ No escalation
      readOnlyRootFilesystem: true     # ✅ Immutable FS
      capabilities:
        drop: ["ALL"]                   # ✅ Drop all caps
        add: ["NET_BIND_SERVICE"]       # ✅ Add only needed
    resources:
      limits:
        memory: "256Mi"           # ✅ Memory limit
        cpu: "500m"               # ✅ CPU limit
      requests:
        memory: "128Mi"
        cpu: "250m"
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}
```

---

## 3. RBAC (Role-Based Access Control)

```yaml
# === PRINCIPLE OF LEAST PRIVILEGE ===

# Role — defines permissions within a namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]      # Read-only!

# RoleBinding — assigns role to user/service account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: production
  name: read-pods
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
# === RBAC AUDIT COMMANDS ===
# Check current permissions
kubectl auth can-i create pods
kubectl auth can-i create pods --as developer
kubectl auth can-i '*' '*'                        # Am I cluster-admin?

# List roles and bindings
kubectl get roles -A
kubectl get rolebindings -A
kubectl get clusterroles
kubectl get clusterrolebindings
```

---

## 4. Network Policies

```yaml
# Default: ALL pods can talk to ALL pods (no isolation!)
# Network Policy: Restrict pod communication

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}                     # Apply to all pods
  policyTypes:
  - Ingress
  - Egress
  # No ingress/egress rules = deny all!

---
# Allow only specific traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-to-api
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web
    ports:
    - port: 8080
      protocol: TCP
```

---

## 5. Secrets Management

```bash
# === KUBERNETES SECRETS ===
# ⚠️ Secrets are base64 encoded, NOT encrypted by default!

# Create secret
kubectl create secret generic db-creds \
    --from-literal=username=admin \
    --from-literal=password=S3cur3P@ss

# View secret (base64 encoded)
kubectl get secret db-creds -o yaml
# data:
#   password: UzNjdXIzUEBzcw==     ← Just base64!
echo "UzNjdXIzUEBzcw==" | base64 -d   # → S3cur3P@ss

# === SECURE SECRETS ===
# 1. Enable etcd encryption at rest
# 2. Use external secret managers:
#    • HashiCorp Vault
#    • AWS Secrets Manager
#    • Azure Key Vault
#    • Sealed Secrets (Bitnami)
# 3. Limit RBAC access to secrets
# 4. Never commit secrets to Git
```

---

## 6. Security Scanning Tools

| Tool | Purpose |
|------|---------|
| **kube-bench** | CIS Kubernetes benchmark |
| **kube-hunter** | Penetration testing for K8s |
| **kubeaudit** | Security audit tool |
| **Polaris** | Best practices validation |
| **OPA/Gatekeeper** | Policy enforcement |
| **Falco** | Runtime threat detection |

```bash
# kube-bench — CIS benchmark
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
kubectl logs job/kube-bench

# kube-hunter — pen test K8s
kube-hunter --remote <cluster-ip>
kube-hunter --pod                     # Run from inside cluster
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Pod security** | Non-root, read-only FS, drop capabilities |
| **RBAC** | Least privilege — limit what users/pods can do |
| **Network policies** | Default deny, allow only needed traffic |
| **Secrets** | Base64 ≠ encryption — use Vault or encrypt etcd |
| **kube-bench** | CIS benchmark compliance check |
| **API server** | Secure authentication, enable audit logging |

---

## Quick Revision Questions

1. **What is the default network policy in Kubernetes?**
2. **How are Kubernetes secrets stored by default?**
3. **What does RBAC control in Kubernetes?**
4. **Name 3 pod security context settings.**
5. **What does kube-bench check?**
6. **Why is `allowPrivilegeEscalation: false` important?**

---

[← Previous: Runtime Security](04-runtime-security.md)

---

*Unit 9 - Topic 5 of 5 | [Back to README](../README.md)*

---

*🎉 Congratulations! You have completed Subject 1.3: Linux for Security Professionals!*
