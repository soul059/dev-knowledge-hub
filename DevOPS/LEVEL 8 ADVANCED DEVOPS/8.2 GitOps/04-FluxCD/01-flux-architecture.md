# Chapter 4.1: Flux Architecture

[← Previous: ArgoCD with Helm](../03-ArgoCD/08-argocd-with-helm.md) | [Back to README](../README.md) | [Next → Flux Installation](02-flux-installation.md)

---

## Overview

**Flux** is a CNCF graduated GitOps toolkit that keeps Kubernetes clusters in sync with configuration sources (Git, Helm, OCI). Unlike ArgoCD's monolithic design, Flux uses a **modular controller architecture** — a set of specialized controllers that each handle one concern.

---

## Flux Controller Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    FLUX ARCHITECTURE                              │
│                                                                  │
│  ┌────────────────────────────────────────────┐                  │
│  │                Git / OCI / Helm            │                  │
│  │              (External Sources)            │                  │
│  └───────────────────┬────────────────────────┘                  │
│                      │                                           │
│  ┌───────────────────▼────────────────────────┐                  │
│  │          SOURCE CONTROLLER                 │                  │
│  │  Fetches artifacts from Git, Helm, OCI,    │                  │
│  │  S3 buckets. Produces artifacts.           │                  │
│  │                                            │                  │
│  │  CRDs: GitRepository, HelmRepository,      │                  │
│  │        OCIRepository, Bucket               │                  │
│  └───────────────────┬────────────────────────┘                  │
│                      │ artifact                                  │
│          ┌───────────┼───────────┐                               │
│          ▼           ▼           ▼                               │
│  ┌──────────────┐ ┌────────────┐ ┌──────────────────┐           │
│  │ KUSTOMIZE    │ │   HELM     │ │  IMAGE           │           │
│  │ CONTROLLER   │ │ CONTROLLER │ │  AUTOMATION      │           │
│  │              │ │            │ │  CONTROLLERS     │           │
│  │ Builds and   │ │ Manages    │ │                  │           │
│  │ applies      │ │ Helm       │ │ Scans registries │           │
│  │ Kustomize    │ │ releases   │ │ Updates Git with │           │
│  │ overlays     │ │            │ │ new image tags   │           │
│  │              │ │ CRD:       │ │                  │           │
│  │ CRD:        │ │ HelmRelease│ │ CRDs:            │           │
│  │ Kustomization│ │            │ │ ImageRepository  │           │
│  └──────────────┘ └────────────┘ │ ImagePolicy      │           │
│                                  │ ImageUpdate       │           │
│  ┌───────────────────────────┐   │ Automation        │           │
│  │ NOTIFICATION CONTROLLER   │   └──────────────────┘           │
│  │ Sends alerts to Slack,    │                                   │
│  │ Teams, webhooks, etc.     │                                   │
│  │                           │                                   │
│  │ CRDs: Provider, Alert,   │                                   │
│  │       Receiver            │                                   │
│  └───────────────────────────┘                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## Core Controllers

### 1. Source Controller

Responsible for fetching configuration from external sources and making it available as **artifacts** to other controllers.

| CRD | Purpose | Example Source |
|-----|---------|----------------|
| GitRepository | Fetch from Git | GitHub, GitLab, Bitbucket |
| HelmRepository | Fetch Helm chart index | Helm repo, OCI registry |
| OCIRepository | Fetch OCI artifacts | Docker Hub, ECR, GCR |
| Bucket | Fetch from S3-compatible | AWS S3, MinIO, GCS |

### 2. Kustomize Controller

Builds and applies Kubernetes manifests using **Kustomize**. Even if you don't use Kustomize overlays, this controller handles plain YAML too.

### 3. Helm Controller

Manages the lifecycle of **HelmRelease** resources — install, upgrade, test, rollback, and uninstall Helm charts.

### 4. Image Automation Controllers

Two controllers working together:
- **Image Reflector Controller**: Scans container registries for new tags
- **Image Automation Controller**: Updates Git with new image tags

### 5. Notification Controller

Handles events and alerts:
- **Provider**: Where to send notifications (Slack, Teams, etc.)
- **Alert**: Which events to notify about
- **Receiver**: Incoming webhooks (e.g., GitHub webhook triggers reconciliation)

---

## Reconciliation Loop

```
┌─────────────────────────────────────────────────────┐
│             FLUX RECONCILIATION LOOP                 │
│                                                      │
│  1. SOURCE CONTROLLER                                │
│     │  Poll Git/Helm/OCI at interval (e.g., 1m)     │
│     │  Detect new commit / chart version             │
│     │  Download and store artifact                   │
│     ▼                                                │
│  2. KUSTOMIZE / HELM CONTROLLER                      │
│     │  Watch for new artifacts                       │
│     │  Build manifests (kustomize build / helm       │
│     │  template)                                     │
│     │  Compute diff (desired vs actual)              │
│     │  Apply changes if needed                       │
│     ▼                                                │
│  3. HEALTH ASSESSMENT                                │
│     │  Wait for rollout (Deployments, StatefulSets)  │
│     │  Report Ready / Not Ready status               │
│     ▼                                                │
│  4. NOTIFICATION CONTROLLER                          │
│     │  Emit events                                   │
│     │  Send alerts (Slack, webhook, etc.)             │
│     ▼                                                │
│  5. REPEAT at configured interval                    │
│                                                      │
└─────────────────────────────────────────────────────┘
```

---

## Flux vs ArgoCD Comparison

| Feature | Flux | ArgoCD |
|---------|------|--------|
| Architecture | Modular (multiple controllers) | Monolithic (single app) |
| UI | No built-in UI (use Weave GitOps) | Built-in web UI |
| CRD-based | Yes, fully CRD-driven | Yes, Application CRD |
| Helm support | Native HelmRelease CRD | `helm template` rendering |
| Image automation | Built-in (ImagePolicy) | Requires ArgoCD Image Updater |
| Multi-tenancy | Native (Kustomization per tenant) | AppProject-based |
| Notifications | Built-in Notification Controller | Notification engine or webhooks |
| CNCF status | Graduated | Graduated |
| Learning curve | Steeper (many CRDs) | Easier (UI-driven) |

---

## Key CRDs at a Glance

```
┌─────────────────────────────────────────────────────────┐
│                   FLUX CRDs                              │
│                                                          │
│  Sources:                                                │
│  ├── GitRepository          (watch a Git repo/branch)    │
│  ├── HelmRepository         (watch a Helm chart index)   │
│  ├── OCIRepository          (watch an OCI artifact)      │
│  └── Bucket                 (watch S3 bucket)            │
│                                                          │
│  Reconcilers:                                            │
│  ├── Kustomization          (apply Kustomize overlays)   │
│  └── HelmRelease            (install/upgrade Helm chart) │
│                                                          │
│  Image Automation:                                       │
│  ├── ImageRepository        (scan container registry)    │
│  ├── ImagePolicy            (select image tag)           │
│  └── ImageUpdateAutomation  (commit new tag to Git)      │
│                                                          │
│  Notifications:                                          │
│  ├── Provider               (Slack, Teams, webhook)      │
│  ├── Alert                  (filter events → provider)   │
│  └── Receiver               (incoming webhook trigger)   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Source Controller | Fetches from Git, Helm repos, OCI, S3 |
| Kustomize Controller | Builds and applies manifests |
| Helm Controller | Manages HelmRelease lifecycle |
| Image Automation | Scans registries, updates Git with new tags |
| Notification Controller | Alerts and incoming webhooks |
| Reconciliation | Continuous loop: fetch → build → diff → apply |
| CRD-driven | Everything configured via Kubernetes CRDs |
| Modular | Install only the controllers you need |

---

## Quick Revision Questions

1. **Name the five main controllers in Flux.**
   > Source Controller, Kustomize Controller, Helm Controller, Image Reflector Controller, Image Automation Controller, and Notification Controller.

2. **What does the Source Controller produce?**
   > It fetches configuration from external sources (Git, Helm, OCI, S3) and produces versioned artifacts consumed by other controllers.

3. **How does Flux's architecture differ from ArgoCD?**
   > Flux is modular — each concern (source fetching, kustomize, helm, notifications) is a separate controller. ArgoCD is a single monolithic application.

4. **What CRD do you use to apply Kustomize overlays?**
   > `Kustomization` (from `kustomize.toolkit.fluxcd.io`).

5. **Does Flux have a built-in web UI?**
   > No. Flux is CLI/CRD-driven. You can use Weave GitOps or Capacitor as a UI.

6. **What is the role of the Notification Controller?**
   > It sends alerts to external services (Slack, Teams, webhooks) and receives incoming webhooks to trigger reconciliation.

---

[← Previous: ArgoCD with Helm](../03-ArgoCD/08-argocd-with-helm.md) | [Back to README](../README.md) | [Next → Flux Installation](02-flux-installation.md)
