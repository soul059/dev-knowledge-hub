# Chapter 3.1: ArgoCD Architecture

[← Previous: Version Control Best Practices](../02-Git-Source-Of-Truth/05-version-control-best-practices.md) | [Back to README](../README.md) | [Next → ArgoCD Installation](02-argocd-installation.md)

---

## Overview

ArgoCD is a **declarative, GitOps continuous delivery tool for Kubernetes**. It is a CNCF graduated project and the most widely adopted GitOps agent. ArgoCD continuously monitors Git repositories, compares the desired state with the actual cluster state, and reconciles differences automatically.

---

## High-Level Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                        ARGOCD ARCHITECTURE                          │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │                      ARGOCD NAMESPACE                       │     │
│  │                                                             │     │
│  │  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   │     │
│  │  │   API        │   │ Repository   │   │ Application  │   │     │
│  │  │   Server     │   │ Server       │   │ Controller   │   │     │
│  │  │              │   │              │   │              │   │     │
│  │  │ • REST/gRPC  │   │ • Clone repos│   │ • Watch apps │   │     │
│  │  │ • Web UI     │   │ • Cache      │   │ • Compare    │   │     │
│  │  │ • Auth       │   │ • Generate   │   │ • Sync       │   │     │
│  │  │ • RBAC       │   │   manifests  │   │ • Reconcile  │   │     │
│  │  └──────┬───────┘   └──────┬───────┘   └──────┬───────┘   │     │
│  │         │                  │                   │           │     │
│  │         └──────────┬───────┘───────────────────┘           │     │
│  │                    │                                       │     │
│  │              ┌─────▼─────┐                                 │     │
│  │              │   Redis   │  (Cache layer)                  │     │
│  │              └───────────┘                                 │     │
│  │                                                             │     │
│  │  ┌──────────────┐   ┌──────────────┐                       │     │
│  │  │ Dex (OIDC)   │   │ Notification │                       │     │
│  │  │ SSO Provider │   │ Controller   │                       │     │
│  │  └──────────────┘   └──────────────┘                       │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  External:                                                           │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                       │
│  │ Git Repo │    │ Helm     │    │ OCI      │                       │
│  │ (GitHub) │    │ Registry │    │ Registry │                       │
│  └──────────┘    └──────────┘    └──────────┘                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Core Components — Deep Dive

### 1. API Server (`argocd-server`)

```
┌──────────────────────────────────────────────────────┐
│              API SERVER                              │
│                                                      │
│  Responsibilities:                                   │
│  ├── Serve Web UI (React frontend)                   │
│  ├── Expose REST API                                 │
│  ├── Expose gRPC API (used by CLI)                   │
│  ├── Handle authentication (local, SSO, OIDC)        │
│  ├── Enforce RBAC policies                           │
│  ├── Serve webhook endpoints                         │
│  └── Manage application CRUD operations              │
│                                                      │
│  Ports:                                              │
│  ├── 443 (HTTPS - Web UI + API)                      │
│  ├── 80  (HTTP - redirect to HTTPS)                  │
│  └── 8083 (Metrics)                                  │
│                                                      │
│  Access Methods:                                     │
│  ├── Web UI:   https://argocd.example.com            │
│  ├── CLI:      argocd login argocd.example.com       │
│  └── REST API: curl https://argocd.example.com/api/  │
└──────────────────────────────────────────────────────┘
```

### 2. Repository Server (`argocd-repo-server`)

```
┌──────────────────────────────────────────────────────┐
│           REPOSITORY SERVER                          │
│                                                      │
│  Responsibilities:                                   │
│  ├── Clone and cache Git repositories                │
│  ├── Generate Kubernetes manifests from:             │
│  │   ├── Plain YAML                                  │
│  │   ├── Kustomize overlays                          │
│  │   ├── Helm charts                                 │
│  │   ├── Jsonnet files                               │
│  │   └── Custom config management plugins            │
│  ├── Return rendered manifests to the controller     │
│  └── Maintain repo credential cache                  │
│                                                      │
│  Flow:                                               │
│  Git Repo ──clone──> Repo Server ──render──> YAML    │
│                                                      │
│  Security: Runs in isolated containers               │
│  Performance: Caches repos to avoid repeated clones  │
└──────────────────────────────────────────────────────┘
```

### 3. Application Controller (`argocd-application-controller`)

```
┌──────────────────────────────────────────────────────┐
│         APPLICATION CONTROLLER                       │
│                                                      │
│  Responsibilities:                                   │
│  ├── Watch Application CRDs in the cluster           │
│  ├── Poll repo server for desired state              │
│  ├── Query Kubernetes API for actual state            │
│  ├── Compute diff (desired vs actual)                │
│  ├── Execute sync operations (apply changes)         │
│  ├── Run health assessments                          │
│  ├── Report sync/health status                       │
│  └── Handle automated sync policies                  │
│                                                      │
│  Key Settings:                                       │
│  ├── Reconciliation interval (default: 3 min)        │
│  ├── Self-heal interval (default: 5 sec)             │
│  └── Status refresh interval                         │
│                                                      │
│  This is the BRAIN of ArgoCD.                        │
└──────────────────────────────────────────────────────┘
```

---

## Data Flow — End to End

```
┌──────────────────────────────────────────────────────────────────┐
│                  ARGOCD DATA FLOW                                │
│                                                                  │
│  ① Developer commits to Git                                      │
│  ┌──────┐     ┌──────────┐                                       │
│  │ Dev  │────>│ Git Repo │                                       │
│  └──────┘     └────┬─────┘                                       │
│                    │                                              │
│  ② Repo Server detects change (poll or webhook)                  │
│                    │                                              │
│               ┌────▼──────────┐                                   │
│               │ Repo Server   │                                   │
│               │ • git pull    │                                   │
│               │ • kustomize   │                                   │
│               │   build       │                                   │
│               │ • Output YAML │                                   │
│               └────┬──────────┘                                   │
│                    │                                              │
│  ③ Controller compares desired vs actual                          │
│               ┌────▼──────────────┐     ┌──────────────┐         │
│               │ App Controller    │────>│ K8s API      │         │
│               │ • Diff engine     │     │ (actual state│         │
│               │ • Three-way merge │     │  query)      │         │
│               └────┬──────────────┘     └──────────────┘         │
│                    │                                              │
│  ④ If OutOfSync: apply changes                                   │
│               ┌────▼──────────────┐     ┌──────────────┐         │
│               │ kubectl apply     │────>│ K8s API      │         │
│               │ (server-side)     │     │ (apply)      │         │
│               └────┬──────────────┘     └──────────────┘         │
│                    │                                              │
│  ⑤ Report status                                                 │
│               ┌────▼──────────────┐                               │
│               │ Status: Synced ✓  │                               │
│               │ Health: Healthy ✓ │                               │
│               └───────────────────┘                               │
└──────────────────────────────────────────────────────────────────┘
```

---

## Custom Resource Definitions (CRDs)

ArgoCD extends Kubernetes with these CRDs:

| CRD | Purpose |
|-----|---------|
| `Application` | Defines a single deployable unit (source → destination) |
| `AppProject` | Groups applications with shared RBAC and source/destination restrictions |
| `ApplicationSet` | Template for generating multiple Applications dynamically |

### Application CRD — Minimal Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd            # Must be in ArgoCD namespace
spec:
  project: default             # AppProject reference

  source:                       # WHERE to get desired state
    repoURL: https://github.com/myorg/config.git
    targetRevision: main
    path: overlays/production

  destination:                  # WHERE to deploy
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:                   # HOW to sync
    automated:
      prune: true
      selfHeal: true
```

---

## Health Assessment Model

```
┌──────────────────────────────────────────────────────────────┐
│               ARGOCD HEALTH MODEL                            │
│                                                              │
│  Resource Health States:                                     │
│                                                              │
│  ┌──────────┐  Pod is Running, all containers ready          │
│  │ Healthy  │  Service has endpoints                         │
│  │    ✓     │  Deployment has desired replicas               │
│  └──────────┘                                                │
│                                                              │
│  ┌──────────┐  Deployment is rolling out                     │
│  │Progressing│  Pod is being scheduled                       │
│  │    ⟳     │  Job is running                                │
│  └──────────┘                                                │
│                                                              │
│  ┌──────────┐  Pod is CrashLoopBackOff                       │
│  │ Degraded │  Deployment has unavailable replicas            │
│  │    ⚠     │  PVC is pending                                │
│  └──────────┘                                                │
│                                                              │
│  ┌──────────┐  Resource type not recognized                  │
│  │ Missing  │  or health check not implemented               │
│  │    ?     │                                                │
│  └──────────┘                                                │
│                                                              │
│  App-level health = worst health of all child resources      │
└──────────────────────────────────────────────────────────────┘
```

---

## Deployment Strategies Supported

| Strategy | Support |
|----------|---------|
| Plain YAML | ✓ Native |
| Kustomize | ✓ Native |
| Helm | ✓ Native |
| Jsonnet | ✓ Native |
| Custom plugins | ✓ Config Management Plugins |
| OCI artifacts | ✓ (Helm OCI, OCI manifests) |

---

## Summary Table

| Component | Role | Key Responsibility |
|-----------|------|-------------------|
| API Server | User interface | Web UI, REST/gRPC API, auth, RBAC |
| Repo Server | Manifest generation | Clone repos, render Helm/Kustomize/YAML |
| App Controller | Reconciliation engine | Diff, sync, health check, self-heal |
| Redis | Cache | Repo cache, session data |
| Dex | Authentication | SSO/OIDC provider integration |
| Notification Controller | Alerts | Slack, webhook, email notifications |
| Application CRD | Config | Defines source → destination mapping |
| AppProject CRD | Governance | RBAC boundaries for groups of apps |
| ApplicationSet CRD | Automation | Template-based app generation |

---

## Quick Revision Questions

1. **What are the three core components of ArgoCD?**
   > API Server (user interface), Repository Server (manifest generation), and Application Controller (reconciliation engine).

2. **What is the role of the Repository Server?**
   > It clones Git repositories, renders manifests from Kustomize/Helm/YAML/Jsonnet, caches repos for performance, and returns rendered YAML to the controller.

3. **What CRDs does ArgoCD introduce to Kubernetes?**
   > Application (single deployable unit), AppProject (RBAC grouping), and ApplicationSet (templated multi-app generation).

4. **Describe the data flow from a Git commit to cluster deployment in ArgoCD.**
   > Developer commits → Repo Server detects and pulls → renders manifests → Controller diffs desired vs actual → applies changes if OutOfSync → reports status.

5. **What are the four health states ArgoCD tracks for resources?**
   > Healthy (running correctly), Progressing (in transition), Degraded (unhealthy), Missing (not found/unknown).

6. **What manifest generation tools does ArgoCD support natively?**
   > Plain YAML, Kustomize, Helm, Jsonnet, and custom config management plugins.

---

[← Previous: Version Control Best Practices](../02-Git-Source-Of-Truth/05-version-control-best-practices.md) | [Back to README](../README.md) | [Next → ArgoCD Installation](02-argocd-installation.md)
