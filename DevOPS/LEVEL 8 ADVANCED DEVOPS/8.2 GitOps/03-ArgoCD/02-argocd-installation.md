# Chapter 3.2: ArgoCD Installation

[← Previous: ArgoCD Architecture](01-argocd-architecture.md) | [Back to README](../README.md) | [Next → Application Definitions](03-application-definitions.md)

---

## Overview

This chapter covers installing ArgoCD on Kubernetes — from basic installations to production-grade HA deployments. You'll learn multiple installation methods, initial configuration, and how to access the ArgoCD UI and CLI.

---

## Installation Methods

```
┌──────────────────────────────────────────────────────────────┐
│              INSTALLATION METHODS                            │
│                                                              │
│  1. Plain Manifests    Quick start, non-HA                   │
│  2. HA Manifests       Production-ready, multi-replica       │
│  3. Helm Chart         Customizable, template-based          │
│  4. Kustomize          Overlay-based customization           │
│  5. Operator           OLM-managed lifecycle                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Method 1: Plain Manifests (Quick Start)

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD (non-HA)
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify installation
kubectl get pods -n argocd
# NAME                                  READY   STATUS    RESTARTS
# argocd-application-controller-0       1/1     Running   0
# argocd-repo-server-xxx                1/1     Running   0
# argocd-server-xxx                     1/1     Running   0
# argocd-redis-xxx                      1/1     Running   0
# argocd-dex-server-xxx                 1/1     Running   0
# argocd-notifications-controller-xxx   1/1     Running   0
```

---

## Method 2: HA Installation (Production)

```bash
# Install HA version (multiple replicas for each component)
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml

# HA differences:
# - argocd-server: 2+ replicas
# - argocd-repo-server: 2+ replicas
# - argocd-application-controller: runs as StatefulSet with sharding
# - Redis: HA Redis with Sentinel
```

### HA vs Non-HA Comparison

```
  NON-HA (Dev/Test)                HA (Production)
  ═══════════════                  ═══════════════
  
  API Server: 1 replica            API Server: 2+ replicas
  Repo Server: 1 replica           Repo Server: 2+ replicas
  Controller: 1 replica            Controller: StatefulSet (sharded)
  Redis: 1 replica                 Redis: 3 replicas + Sentinel
  
  Single point of failure          Fault-tolerant
  Low resource usage               Higher resource usage
  For dev/lab environments          For production workloads
```

---

## Method 3: Helm Chart Installation

```bash
# Add the ArgoCD Helm repository
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Install with default values
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace

# Install with custom values
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  -f values.yaml
```

### Production Helm Values

```yaml
# values.yaml
global:
  image:
    tag: v2.13.0    # Pin to specific version

server:
  replicas: 2
  ingress:
    enabled: true
    ingressClassName: nginx
    hosts:
      - argocd.example.com
    tls:
      - secretName: argocd-tls
        hosts:
          - argocd.example.com
  resources:
    requests:
      cpu: 250m
      memory: 256Mi
    limits:
      cpu: 500m
      memory: 512Mi

controller:
  replicas: 2
  resources:
    requests:
      cpu: 500m
      memory: 512Mi
    limits:
      cpu: "1"
      memory: 1Gi

repoServer:
  replicas: 2
  resources:
    requests:
      cpu: 250m
      memory: 256Mi
    limits:
      cpu: 500m
      memory: 512Mi

redis-ha:
  enabled: true

configs:
  params:
    server.insecure: false
    timeout.reconciliation: 60
```

---

## Accessing ArgoCD

### Get Initial Admin Password

```bash
# The initial password is auto-generated and stored as a Secret
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

# Username: admin
# Password: <output from above>
```

### Access Methods

```
┌──────────────────────────────────────────────────────────────┐
│              ACCESS METHODS                                  │
│                                                              │
│  1. Port Forward (quick/dev)                                 │
│     kubectl port-forward svc/argocd-server -n argocd 8080:443│
│     → Open https://localhost:8080                            │
│                                                              │
│  2. LoadBalancer Service                                     │
│     kubectl patch svc argocd-server -n argocd \              │
│       -p '{"spec": {"type": "LoadBalancer"}}'                │
│                                                              │
│  3. Ingress (recommended for production)                     │
│     See Ingress configuration below                          │
│                                                              │
│  4. CLI                                                      │
│     argocd login argocd.example.com                          │
└──────────────────────────────────────────────────────────────┘
```

### Ingress Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - argocd.example.com
    secretName: argocd-tls
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443
```

---

## Install ArgoCD CLI

```bash
# Linux
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd && sudo mv argocd /usr/local/bin/

# macOS
brew install argocd

# Windows (PowerShell)
choco install argocd-cli
# OR
Invoke-WebRequest -Uri "https://github.com/argoproj/argo-cd/releases/latest/download/argocd-windows-amd64.exe" `
  -OutFile "argocd.exe"

# Login
argocd login argocd.example.com
# Username: admin
# Password: <initial password>

# Change the admin password
argocd account update-password
```

---

## Post-Installation Configuration

### Register a Git Repository

```bash
# HTTPS with token
argocd repo add https://github.com/myorg/config.git \
  --username git \
  --password <github-token>

# SSH key
argocd repo add git@github.com:myorg/config.git \
  --ssh-private-key-path ~/.ssh/id_rsa
```

### Repository Credential via Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-repo-creds
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
type: Opaque
stringData:
  type: git
  url: https://github.com/myorg/config.git
  username: git
  password: ghp_xxxxxxxxxxxxxxxxxxxx
```

### Configure SSO (OIDC)

```yaml
# ArgoCD ConfigMap for SSO
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  url: https://argocd.example.com
  oidc.config: |
    name: Okta
    issuer: https://mycompany.okta.com/oauth2/default
    clientID: xxxxxxxxxxxxxxxxx
    clientSecret: $oidc.okta.clientSecret
    requestedScopes:
      - openid
      - profile
      - email
      - groups
```

---

## Verify Installation

```bash
# Check all pods are running
kubectl get pods -n argocd

# Check ArgoCD version
argocd version

# Check server health
argocd admin dashboard  # Opens web UI

# List registered clusters
argocd cluster list

# List registered repos
argocd repo list

# Create a test application
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Sync the test app
argocd app sync guestbook

# Check status
argocd app get guestbook
```

---

## Troubleshooting Installation

| Problem | Diagnosis | Fix |
|---------|-----------|-----|
| Pods not starting | `kubectl describe pod -n argocd` | Check resource limits, image pull |
| Can't get admin password | Secret may be deleted | `argocd admin initial-password -n argocd` |
| Ingress not working | Check annotations | Ensure SSL passthrough is enabled |
| Repo not accessible | `argocd repo list` | Check credentials, SSH keys |
| OIDC login fails | Check argocd-cm | Verify issuer URL, client ID/secret |
| High memory usage | Too many apps | Increase controller memory, enable sharding |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Namespace | `argocd` (default) |
| Non-HA install | Single `kubectl apply` of install.yaml |
| HA install | Separate HA manifest or Helm with `redis-ha: enabled` |
| Helm chart | `argo/argo-cd` from argoproj Helm repo |
| Default user | `admin` |
| Initial password | Stored in `argocd-initial-admin-secret` Secret |
| Access UI | Port-forward, LoadBalancer, or Ingress |
| CLI install | `brew install argocd` / `choco install argocd-cli` |
| First test | Deploy `guestbook` example app |

---

## Quick Revision Questions

1. **What are the two main installation modes for ArgoCD?**
   > Non-HA (single replica, for dev/test) and HA (multi-replica, for production).

2. **How do you retrieve the initial admin password?**
   > `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

3. **What is the recommended way to expose ArgoCD in production?**
   > Via an Ingress controller with TLS and SSL passthrough, rather than port-forwarding or LoadBalancer.

4. **Name three ways to install ArgoCD.**
   > Plain manifests (`kubectl apply`), Helm chart (`argo/argo-cd`), and Kustomize overlays.

5. **How do you register a private Git repository with ArgoCD?**
   > Using the CLI: `argocd repo add <url> --username --password` or by creating a Kubernetes Secret with the `argocd.argoproj.io/secret-type: repository` label.

6. **What should you check if ArgoCD pods are not starting?**
   > Run `kubectl describe pod -n argocd` to check for image pull errors, resource limit issues, or missing dependencies.

---

[← Previous: ArgoCD Architecture](01-argocd-architecture.md) | [Back to README](../README.md) | [Next → Application Definitions](03-application-definitions.md)
