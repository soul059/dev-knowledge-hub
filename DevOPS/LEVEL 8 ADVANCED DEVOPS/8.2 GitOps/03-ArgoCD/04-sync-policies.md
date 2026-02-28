# Chapter 3.4: Sync Policies

[← Previous: Application Definitions](03-application-definitions.md) | [Back to README](../README.md) | [Next → Rollouts and Rollbacks](05-rollouts-and-rollbacks.md)

---

## Overview

Sync policies control **how and when** ArgoCD applies changes from Git to the cluster. They determine whether sync is manual or automatic, how pruning works, and how the system handles failures. Getting sync policies right is critical for balancing speed and safety.

---

## Sync Policy Options

```
┌──────────────────────────────────────────────────────────────┐
│                   SYNC POLICY SPECTRUM                       │
│                                                              │
│  Manual ◄─────────────────────────────────────► Automatic    │
│                                                              │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────────┐   │
│  │  Manual  │    │  Auto Sync   │    │  Auto + Prune    │   │
│  │  Sync    │    │  (no prune)  │    │  + Self-Heal     │   │
│  │          │    │              │    │                  │   │
│  │  Human   │    │  Auto-apply  │    │  Full autopilot  │   │
│  │  triggers│    │  new changes │    │  mode            │   │
│  │  sync    │    │  but keep    │    │                  │   │
│  │          │    │  orphans     │    │                  │   │
│  └──────────┘    └──────────────┘    └──────────────────┘   │
│                                                              │
│  Use for:        Use for:            Use for:                │
│  Production      Staging             Dev / Well-tested       │
│  (cautious)      (semi-auto)         production              │
└──────────────────────────────────────────────────────────────┘
```

---

## Manual Sync

```yaml
# No syncPolicy.automated — requires manual trigger
spec:
  syncPolicy: {}
```

```bash
# Trigger sync manually
argocd app sync myapp

# Sync specific resources only
argocd app sync myapp --resource apps:Deployment:myapp

# Dry run (preview what would change)
argocd app sync myapp --dry-run

# Sync and wait for healthy
argocd app sync myapp --wait
```

---

## Automated Sync

```yaml
spec:
  syncPolicy:
    automated:
      prune: false          # Don't delete removed resources
      selfHeal: false       # Don't revert manual changes
```

### With Prune

```yaml
spec:
  syncPolicy:
    automated:
      prune: true           # DELETE resources removed from Git
      selfHeal: false
```

### With Self-Heal

```yaml
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true        # Revert ANY manual changes
```

### Self-Heal Flow

```
┌──────────────────────────────────────────────────────────────┐
│                    SELF-HEAL FLOW                            │
│                                                              │
│  1. Git says: replicas=3, image=v2                           │
│  2. Agent syncs: cluster has replicas=3, image=v2 ✓          │
│  3. Someone runs: kubectl scale --replicas=1                 │
│  4. Agent detects: replicas=1 ≠ replicas=3 (drift!)          │
│  5. Agent reverts: replicas back to 3                        │
│  6. Status: Synced ✓                                          │
│                                                              │
│  Self-heal check interval: every 5 seconds (default)         │
│  This means manual changes are reverted within seconds       │
└──────────────────────────────────────────────────────────────┘
```

---

## Sync Options

```yaml
spec:
  syncPolicy:
    syncOptions:
    - CreateNamespace=true          # Create namespace if missing
    - PrunePropagationPolicy=foreground  # Wait for dependents to delete
    - PruneLast=true                # Prune after all syncs complete
    - Validate=false                # Skip schema validation
    - ApplyOutOfSyncOnly=true       # Only apply changed resources
    - ServerSideApply=true          # Use server-side apply
    - RespectIgnoreDifferences=true # Honor ignoreDifferences during sync
    - Replace=true                  # Replace instead of apply (destructive)
    - FailOnSharedResource=true     # Fail if resource is managed by another app
```

### Sync Options Explained

| Option | Default | Effect |
|--------|---------|--------|
| `CreateNamespace` | false | Auto-create target namespace |
| `PrunePropagationPolicy` | background | foreground = wait for child deletions |
| `PruneLast` | false | Prune only after all creates/updates succeed |
| `Validate` | true | Skip K8s schema validation |
| `ApplyOutOfSyncOnly` | false | Only apply resources that differ |
| `ServerSideApply` | false | Use K8s server-side apply (better conflict handling) |
| `Replace` | false | Replace resource instead of patch (destructive) |

---

## Sync Waves and Hooks

### Sync Waves — Ordering Resources

```
┌──────────────────────────────────────────────────────────────┐
│                   SYNC WAVES                                 │
│                                                              │
│  Wave -1: Namespaces, CRDs, RBAC                            │
│           (must exist before apps)                           │
│            ▼                                                 │
│  Wave 0:  ConfigMaps, Secrets, Services                      │
│           (default wave, no annotation)                      │
│            ▼                                                 │
│  Wave 1:  Deployments, StatefulSets                          │
│           (depend on ConfigMaps)                             │
│            ▼                                                 │
│  Wave 2:  HPA, Ingress, NetworkPolicy                        │
│           (depend on Deployments)                            │
│                                                              │
│  Lower waves sync FIRST. Same wave = parallel.              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
# Wave -1: Create namespace first
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
  annotations:
    argocd.argoproj.io/sync-wave: "-1"

---
# Wave 0: ConfigMap (default)
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  annotations:
    argocd.argoproj.io/sync-wave: "0"

---
# Wave 1: Deployment (after ConfigMap exists)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

### Sync Hooks — Pre/Post Actions

```yaml
# Pre-sync hook: Run database migration before deploying
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: myapp:v2
        command: ["./migrate.sh"]
      restartPolicy: Never

---
# Post-sync hook: Run smoke tests after deploying
apiVersion: batch/v1
kind: Job
metadata:
  name: smoke-test
  annotations:
    argocd.argoproj.io/hook: PostSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
      - name: test
        image: myapp-tests:v2
        command: ["./smoke-test.sh"]
      restartPolicy: Never
```

### Hook Types

| Hook | When It Runs |
|------|-------------|
| `PreSync` | Before sync (DB migrations, backups) |
| `Sync` | During sync (alongside resources) |
| `PostSync` | After sync succeeds (smoke tests, notifications) |
| `SyncFail` | After sync fails (cleanup, alerting) |
| `Skip` | Resource is ignored during sync |

### Hook Delete Policies

| Policy | Behavior |
|--------|----------|
| `HookSucceeded` | Delete after hook succeeds |
| `HookFailed` | Delete after hook fails |
| `BeforeHookCreation` | Delete previous hook before creating new one |

---

## Retry Policy

```yaml
spec:
  syncPolicy:
    retry:
      limit: 5              # Max retry attempts
      backoff:
        duration: 5s         # Initial backoff duration
        factor: 2            # Multiply by 2 each time
        maxDuration: 3m      # Cap at 3 minutes

# Retry sequence: 5s → 10s → 20s → 40s → 80s (capped at 3m)
```

---

## Sync Windows — Time-Based Restrictions

```yaml
# Only allow syncs during business hours
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
spec:
  syncWindows:
  - kind: allow
    schedule: "0 9-17 * * 1-5"    # Mon-Fri 9am-5pm
    duration: 8h
    applications:
    - "*"
  - kind: deny
    schedule: "0 0 * * *"         # Block midnight syncs
    duration: 4h
    applications:
    - "critical-*"
    manualSync: true               # Allow manual override
```

---

## Summary Table

| Policy | Behavior | Use Case |
|--------|----------|----------|
| Manual sync | Human triggers sync | Production (cautious) |
| Auto sync | Auto-apply on Git change | Staging, dev |
| Prune | Delete removed resources | Cleanup old resources |
| Self-heal | Revert manual changes | Prevent config drift |
| Sync waves | Order resource creation | Dependencies (NS → CM → Deploy) |
| PreSync hook | Run before sync | DB migration, backup |
| PostSync hook | Run after sync | Smoke tests, notifications |
| Retry | Auto-retry failed syncs | Transient failures |
| Sync windows | Time-based restrictions | Business hours only |

---

## Quick Revision Questions

1. **What is the difference between prune and self-heal?**
   > Prune: deletes resources that were removed from Git. Self-heal: reverts manual changes to match Git (e.g., someone runs `kubectl edit`).

2. **How do sync waves work?**
   > Resources with lower wave numbers sync first. Same-wave resources sync in parallel. Use annotations like `argocd.argoproj.io/sync-wave: "-1"`.

3. **When would you use a PreSync hook?**
   > For operations that must complete before deployment, such as database migrations, backups, or pre-flight checks.

4. **What does `ApplyOutOfSyncOnly=true` do?**
   > It only applies resources that differ between Git and the cluster, rather than re-applying everything. Improves sync performance.

5. **How does the retry backoff work?**
   > Starts at `duration`, multiplies by `factor` each retry, capped at `maxDuration`. Example: 5s → 10s → 20s → 40s → 80s.

6. **What are sync windows used for?**
   > Time-based restrictions on when syncs can occur — e.g., only during business hours, block weekend deploys.

---

[← Previous: Application Definitions](03-application-definitions.md) | [Back to README](../README.md) | [Next → Rollouts and Rollbacks](05-rollouts-and-rollbacks.md)
