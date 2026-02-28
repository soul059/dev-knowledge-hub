# Chapter 5.6: Self-Healing

[← Previous: Drift Detection](05-drift-detection.md) | [Back to README](../README.md) | [Next → Repository Organization](../06-GitOps-Best-Practices/01-repository-organization.md)

---

## Overview

**Self-healing** is the GitOps superpower: when someone manually changes a resource in the cluster, the GitOps operator automatically reverts it to the desired state defined in Git. This ensures the cluster always matches its declared configuration, eliminating "configuration snowflakes."

---

## How Self-Healing Works

```
┌──────────────────────────────────────────────────────────┐
│              SELF-HEALING FLOW                           │
│                                                          │
│  1. Manual change made to cluster                        │
│     kubectl scale deploy/my-app --replicas=10            │
│     │                                                    │
│     ▼                                                    │
│  2. Cluster state diverges from Git                      │
│     Desired (Git): replicas=3                            │
│     Actual (Cluster): replicas=10                        │
│     │                                                    │
│     ▼                                                    │
│  3. GitOps operator detects drift                        │
│     (next reconciliation cycle)                          │
│     │                                                    │
│     ▼                                                    │
│  4. Operator re-applies desired state from Git           │
│     replicas: 10 → 3        (reverted!)                  │
│     │                                                    │
│     ▼                                                    │
│  5. Cluster matches Git again ✓                          │
│                                                          │
│  Timeline: < 3 minutes (ArgoCD) / < interval (Flux)     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Self-Healing in ArgoCD

### Enabling Self-Heal

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  syncPolicy:
    automated:
      selfHeal: true    # ← Automatically corrects drift
      prune: true       # ← Deletes resources removed from Git
```

### What selfHeal: true Does

```
┌──────────────────────────────────────────────────────────┐
│          ARGOCD SELF-HEAL BEHAVIOR                       │
│                                                          │
│  selfHeal: true                                          │
│  ├── Detects OutOfSync state                             │
│  ├── Automatically triggers a sync                       │
│  ├── Applies the desired state from Git                  │
│  └── Repeats every reconciliation cycle (3 min)          │
│                                                          │
│  selfHeal: false (default)                               │
│  ├── Detects OutOfSync state                             │
│  ├── Shows OutOfSync in UI / CLI                         │
│  └── Waits for manual sync                               │
│                                                          │
│  prune: true                                             │
│  ├── Resources deleted from Git are deleted from cluster │
│  └── Without prune, orphaned resources remain            │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Self-Heal with Specific Sync Options

```yaml
spec:
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
    syncOptions:
    - Validate=true              # Validate manifests before apply
    - CreateNamespace=true       # Create NS if missing
    - PrunePropagationPolicy=foreground  # Wait for dependents
    - PruneLast=true             # Prune after sync completes
    retry:
      limit: 5                  # Retry on failure
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

---

## Self-Healing in Flux

Flux self-heals by default through its reconciliation loop — it **re-applies** manifests at every interval.

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 5m              # Re-apply every 5 minutes
  sourceRef:
    kind: GitRepository
    name: my-app
  path: ./deploy
  prune: true               # Delete resources removed from Git
  force: false               # Set true to force-apply (overwrite conflicts)
  wait: true                 # Wait for rollout health
```

### Flux Force Reconciliation

```bash
# Immediately trigger reconciliation (fix drift now)
flux reconcile kustomization my-app

# Force apply (overwrite even conflicting field managers)
flux reconcile kustomization my-app --force
```

---

## Self-Healing Scenarios

```
┌──────────────────────────────────────────────────────────┐
│           SELF-HEALING SCENARIOS                         │
│                                                          │
│  Scenario 1: Manual kubectl edit                         │
│  ┌────────────────────────────────────────┐              │
│  │ Admin runs: kubectl edit deploy my-app │              │
│  │ Changes CPU limit to 2000m             │              │
│  │                                        │              │
│  │ Result: Self-heal reverts to Git value │              │
│  │ (500m) within next reconciliation      │              │
│  └────────────────────────────────────────┘              │
│                                                          │
│  Scenario 2: Resource deleted                            │
│  ┌────────────────────────────────────────┐              │
│  │ Admin runs: kubectl delete svc my-app  │              │
│  │                                        │              │
│  │ Result: Self-heal recreates the Service│              │
│  │ from Git within next reconciliation    │              │
│  └────────────────────────────────────────┘              │
│                                                          │
│  Scenario 3: Namespace tampered                          │
│  ┌────────────────────────────────────────┐              │
│  │ Admin adds a rogue Deployment directly │              │
│  │                                        │              │
│  │ Result: prune=true deletes it (it's    │              │
│  │ not in Git). prune=false leaves it.    │              │
│  └────────────────────────────────────────┘              │
│                                                          │
│  Scenario 4: ConfigMap modified                          │
│  ┌────────────────────────────────────────┐              │
│  │ Someone patches ConfigMap data         │              │
│  │                                        │              │
│  │ Result: Self-heal restores original    │              │
│  │ values from Git                        │              │
│  └────────────────────────────────────────┘              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## When NOT to Self-Heal

| Scenario | Why Not | Solution |
|----------|---------|----------|
| HPA-managed replicas | HPA legitimately changes replica count | Ignore replicas field |
| Operator-managed CRs | Controllers update status/spec | Ignore managed fields |
| Emergency hotfixes | Need immediate manual fix | Suspend auto-sync, apply fix, commit to Git, resume |
| Feature flags | Toggled at runtime | Use external config (ConfigMap from external source) |

### Handling Emergencies

```bash
# ArgoCD: Suspend auto-sync for emergency
argocd app set my-app --sync-policy none

# Apply emergency fix manually
kubectl set image deploy/my-app my-app=myorg/my-app:hotfix-v1.2.1

# THEN: commit the fix to Git
cd config-repo
# update image tag to hotfix-v1.2.1
git commit -am "hotfix: emergency fix for CVE-2024-xxxx"
git push

# Re-enable auto-sync
argocd app set my-app --self-heal --auto-prune

# ---

# Flux: Suspend reconciliation for emergency
flux suspend kustomization my-app

# Apply manual fix
kubectl set image deploy/my-app my-app=myorg/my-app:hotfix-v1.2.1

# Commit to Git, then resume
flux resume kustomization my-app
```

---

## Self-Healing + Notifications

```yaml
# ArgoCD: Notification on self-heal event
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  trigger.on-sync-succeeded: |
    - description: Sync succeeded (may be self-heal)
      send: [app-sync-succeeded]
      when: app.status.operationState.phase in ['Succeeded']
  template.app-sync-succeeded: |
    message: |
      Application {{.app.metadata.name}} synced successfully.
      Revision: {{.app.status.sync.revision}}
```

```yaml
# Flux: Alert on reconciliation (self-heal)
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Alert
metadata:
  name: self-heal-alert
  namespace: flux-system
spec:
  providerRef:
    name: slack
  eventSeverity: info
  eventSources:
  - kind: Kustomization
    name: my-app
  inclusionList:
  - ".*Applied revision.*"
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Self-healing | Auto-reverts manual changes to match Git state |
| ArgoCD | `selfHeal: true` in syncPolicy.automated |
| Flux | Default behavior — re-applies on every reconciliation |
| Prune | Deletes resources not in Git (`prune: true`) |
| Emergency override | Suspend auto-sync, fix manually, commit to Git, resume |
| Ignore fields | HPA replicas, operator-managed fields |
| Notifications | Alert on self-heal events for visibility |
| Detection time | ArgoCD: ~3 min, Flux: configurable interval |

---

## Quick Revision Questions

1. **What is self-healing in GitOps?**
   > The automatic correction of manual changes to cluster resources, restoring them to the desired state defined in Git.

2. **How do you enable self-healing in ArgoCD?**
   > Set `syncPolicy.automated.selfHeal: true` in the Application spec.

3. **Does Flux self-heal by default?**
   > Yes — Flux re-applies manifests via server-side apply on every reconciliation interval, overwriting any manual changes.

4. **What happens if someone adds a resource directly to the cluster?**
   > With `prune: true`, the GitOps operator deletes it (it's not in Git). Without prune, it remains as an untracked resource.

5. **How do you handle emergency fixes that need to bypass self-healing?**
   > Suspend auto-sync (`argocd app set --sync-policy none` or `flux suspend`), apply the fix manually, commit the change to Git, then re-enable auto-sync.

6. **Why should you notify on self-heal events?**
   > Self-heal events indicate someone made unauthorized manual changes. Notifications provide visibility and help identify process violations.

---

[← Previous: Drift Detection](05-drift-detection.md) | [Back to README](../README.md) | [Next → Repository Organization](../06-GitOps-Best-Practices/01-repository-organization.md)
