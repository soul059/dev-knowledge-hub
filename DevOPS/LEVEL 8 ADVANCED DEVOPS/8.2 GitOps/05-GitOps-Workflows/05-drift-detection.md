# Chapter 5.5: Drift Detection

[← Previous: Secrets Management](04-secrets-management.md) | [Back to README](../README.md) | [Next → Self-Healing](06-self-healing.md)

---

## Overview

**Drift** occurs when the actual state of a cluster diverges from the desired state defined in Git. GitOps operators continuously detect drift by comparing the rendered manifests from Git against the live cluster state. This chapter covers how drift is detected, common causes, and handling strategies.

---

## What Is Drift?

```
┌──────────────────────────────────────────────────────────┐
│              CONFIGURATION DRIFT                         │
│                                                          │
│  DESIRED STATE (Git):          ACTUAL STATE (Cluster):   │
│  ┌──────────────────┐         ┌──────────────────┐      │
│  │ replicas: 3      │         │ replicas: 5      │ ←    │
│  │ image: v1.2.0    │    ≠    │ image: v1.2.0    │      │
│  │ cpu: 500m        │         │ cpu: 1000m       │ ←    │
│  │ env: prod        │         │ env: prod        │      │
│  └──────────────────┘         └──────────────────┘      │
│                                                          │
│  ← = Drift detected                                     │
│                                                          │
│  Causes of drift:                                        │
│  • Manual kubectl edit / kubectl scale                   │
│  • Horizontal Pod Autoscaler (HPA)                      │
│  • Admission webhooks mutating resources                 │
│  • Operators modifying resources they manage             │
│  • Emergency hotfixes applied directly                   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Drift Detection in ArgoCD

### How ArgoCD Detects Drift

```
┌──────────────────────────────────────────────────────────┐
│         ARGOCD DRIFT DETECTION LOOP                      │
│                                                          │
│  1. App Controller fetches manifests from Git             │
│     (via Repo Server)                                    │
│     │                                                    │
│     ▼                                                    │
│  2. Renders templates (Helm/Kustomize/plain YAML)        │
│     │                                                    │
│     ▼                                                    │
│  3. Compares rendered manifests with live state           │
│     (kubectl get from cluster)                           │
│     │                                                    │
│     ├── No diff → Status: Synced ✓                       │
│     │                                                    │
│     └── Diff found → Status: OutOfSync ⚠                │
│         │                                                │
│         ├── selfHeal: true → Auto-sync to fix drift      │
│         │                                                │
│         └── selfHeal: false → Alert, wait for manual     │
│             sync                                         │
│                                                          │
│  Refresh interval: every 3 minutes (default)             │
│  Compare interval: every 3 minutes (configurable)        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### ArgoCD Application Status States

| Status | Meaning |
|--------|---------|
| Synced | Live state matches Git desired state |
| OutOfSync | Drift detected — live ≠ desired |
| Unknown | Cannot determine state (e.g., cluster unreachable) |
| Missing | Resource exists in Git but not in cluster |

### Viewing Drift in ArgoCD

```bash
# Check application sync status
argocd app get my-app

# View the diff (what drifted)
argocd app diff my-app

# Detailed diff with live vs desired
argocd app diff my-app --local ./path/to/manifests
```

In the **ArgoCD UI**, drifted resources appear with a yellow "OutOfSync" badge, and you can click to see the exact diff.

---

## Drift Detection in Flux

### How Flux Detects Drift

```
┌──────────────────────────────────────────────────────────┐
│           FLUX DRIFT DETECTION                           │
│                                                          │
│  Kustomize Controller:                                   │
│  1. Builds manifests from source (kustomize build)       │
│  2. Computes hash of desired state                       │
│  3. Compares with previously applied hash                 │
│     │                                                    │
│     ├── Same hash → skip (source unchanged)              │
│     │                                                    │
│     └── Different hash → server-side apply               │
│                                                          │
│  Drift correction:                                       │
│  • Flux uses server-side apply by default                │
│  • On each reconciliation interval, it re-applies        │
│  • Any manual changes are overwritten                    │
│                                                          │
│  Force drift detection:                                  │
│  spec:                                                   │
│    force: true    # Force apply even if hash matches     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Flux Status and Events

```bash
# Check Kustomization status
flux get kustomizations

# NAME       REVISION     SUSPENDED  READY  MESSAGE
# my-app     main@sha1:a  False      True   Applied revision: main@sha1:abc123

# View events for drift detection
kubectl events --for kustomization/my-app -n flux-system

# Force reconciliation (checks and fixes drift now)
flux reconcile kustomization my-app
```

---

## Handling Legitimate Drift

Some drift is expected (e.g., HPA changing replicas). You need to **ignore** specific fields.

### ArgoCD: ignoreDifferences

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  ignoreDifferences:
  # Ignore replica count (managed by HPA)
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas

  # Ignore annotations added by admission webhooks
  - group: ""
    kind: Service
    jqPathExpressions:
    - .metadata.annotations["cloud.google.com/neg-status"]

  # Ignore all secrets data fields
  - group: ""
    kind: Secret
    jsonPointers:
    - /data
```

### Flux: Field Managers

Flux uses **server-side apply** with field managers. If another controller (like HPA) owns a field, Flux won't overwrite it by default.

```yaml
# Explicitly exclude fields from management
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
spec:
  patches:
  # Remove replicas from management (HPA controls it)
  - patch: |
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: my-app
      spec:
        replicas: null        # null = don't manage this field
```

---

## Drift Detection Monitoring

### Prometheus Metrics (ArgoCD)

```yaml
# ArgoCD exposes drift metrics
# argocd_app_info{sync_status="OutOfSync"} → count of drifted apps

# Alert rule
groups:
- name: argocd-drift
  rules:
  - alert: ApplicationOutOfSync
    expr: argocd_app_info{sync_status="OutOfSync"} == 1
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: "{{ $labels.name }} is out of sync"
      description: "Application {{ $labels.name }} has been OutOfSync for 10 minutes"
```

### Flux Notification

```yaml
# Alert on drift/reconciliation failures
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Alert
metadata:
  name: drift-alert
  namespace: flux-system
spec:
  providerRef:
    name: slack
  eventSeverity: error
  eventSources:
  - kind: Kustomization
    name: '*'
    namespace: flux-system
```

---

## Common Drift Scenarios and Solutions

| Scenario | Cause | Solution |
|----------|-------|----------|
| Replica count changed | HPA scaling | Use `ignoreDifferences` or remove `replicas` from Git |
| Annotations added | Admission webhooks, cloud controllers | Ignore specific annotations |
| Resource limits changed | Manual `kubectl edit` | Enable self-heal to auto-revert |
| Image tag changed | Emergency hotfix via kubectl | Commit the fix to Git, then self-heal reverts |
| Secret data changed | External rotation | Use External Secrets Operator or SOPS |
| CRD status fields | Operators updating status | These are ignored by default |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Drift | Live state differs from Git desired state |
| ArgoCD detection | Compares rendered manifests vs live state every 3 min |
| Flux detection | Server-side apply on each reconciliation interval |
| OutOfSync | ArgoCD status indicating drift |
| ignoreDifferences | ArgoCD: skip specific fields when comparing |
| Server-side apply | Flux: field ownership prevents unintended overwrites |
| Self-heal | Automatically corrects drift (ArgoCD) |
| Monitoring | Prometheus metrics + alerts for persistent drift |

---

## Quick Revision Questions

1. **What is configuration drift?**
   > When the actual state of resources in a cluster diverges from the desired state defined in Git.

2. **How does ArgoCD detect drift?**
   > The App Controller renders manifests from Git and compares them with the live cluster state every 3 minutes. Differences result in an "OutOfSync" status.

3. **How does Flux handle drift?**
   > Flux uses server-side apply on each reconciliation. Manual changes are overwritten when the Kustomization reconciles.

4. **How do you handle expected drift (e.g., HPA replicas)?**
   > In ArgoCD, use `ignoreDifferences` with `jsonPointers`. In Flux, set the field to `null` so it's not managed, or rely on server-side apply field ownership.

5. **Should all drift be automatically corrected?**
   > Not always. Some drift is intentional (HPA, operator-managed fields). Configure ignore rules for expected drift and self-heal for unexpected drift.

6. **How do you monitor for drift?**
   > ArgoCD exposes Prometheus metrics (`argocd_app_info{sync_status="OutOfSync"}`). Flux emits events that can trigger alerts via the Notification Controller.

---

[← Previous: Secrets Management](04-secrets-management.md) | [Back to README](../README.md) | [Next → Self-Healing](06-self-healing.md)
