# Chapter 5.6: Rollback Strategies

[← Previous: Feature Flags](05-feature-flags.md) | [Back to README](../README.md) | [Next: Zero-Downtime Deployments →](07-zero-downtime-deployments.md)

---

## Overview

**Rollback** is the process of reverting to a previous working version when a deployment causes issues. A well-planned rollback strategy is as important as the deployment itself — it's your safety net.

---

## Rollback Approaches

```
  ┌──────────────────────────────────────────────────────┐
  │            ROLLBACK STRATEGIES                       │
  │                                                      │
  │  1. REDEPLOY PREVIOUS VERSION                        │
  │     Deploy the last known good version               │
  │     Time: Minutes                                    │
  │                                                      │
  │  2. KUBERNETES ROLLBACK                              │
  │     kubectl rollout undo                             │
  │     Time: Seconds                                    │
  │                                                      │
  │  3. BLUE-GREEN SWITCH                                │
  │     Switch load balancer back                        │
  │     Time: Seconds                                    │
  │                                                      │
  │  4. FEATURE FLAG KILL SWITCH                         │
  │     Disable flag, no redeploy needed                 │
  │     Time: Seconds (instant)                          │
  │                                                      │
  │  5. GIT REVERT + REDEPLOY                            │
  │     git revert + trigger pipeline                    │
  │     Time: Minutes (pipeline must run)                │
  │                                                      │
  │  6. DATABASE ROLLBACK                                │
  │     Reverse migration + app rollback                 │
  │     Time: Minutes to hours ⚠ (most complex)         │
  └──────────────────────────────────────────────────────┘
```

---

## Rollback Decision Tree

```
  SOMETHING WENT WRONG IN PRODUCTION!
  ════════════════════════════════════

         Is it a feature bug?
        ┌───────┴───────┐
       YES              NO
        │                │
   Feature flag?    Is it a crash/500s?
   ┌────┴────┐     ┌────┴────┐
  YES       NO    YES       NO
   │         │     │         │
  Kill     Blue   Rollback  Performance
  switch   Green? K8s/ECS   issue?
  (instant) │      │         │
         ┌──┴──┐  seconds   Scale up
        YES   NO             or revert
         │     │
      Switch  Redeploy
      back    previous
      (secs)  version
              (mins)
```

---

## Kubernetes Rollback

```bash
# View rollout history
kubectl rollout history deployment/myapp

# REVISION  CHANGE-CAUSE
# 1         Initial deploy
# 2         Update to v1.2.0
# 3         Update to v1.3.0  ← current (broken!)

# Rollback to previous revision
kubectl rollout undo deployment/myapp

# Rollback to specific revision
kubectl rollout undo deployment/myapp --to-revision=1

# Check rollback status
kubectl rollout status deployment/myapp

# View what changed
kubectl rollout history deployment/myapp --revision=3
```

```yaml
# Set revision history limit
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  revisionHistoryLimit: 10    # Keep last 10 revisions
  strategy:
    type: RollingUpdate
  template:
    metadata:
      annotations:
        kubernetes.io/change-cause: "Update to v1.3.0"
```

---

## Automated Rollback in CI/CD

```yaml
# GitHub Actions — Deploy with automatic rollback
name: Deploy with Rollback
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Save current version
        run: |
          kubectl get deployment myapp -o jsonpath='{.spec.template.spec.containers[0].image}' \
            > previous-version.txt

      - name: Deploy new version
        run: |
          kubectl set image deployment/myapp \
            myapp=registry.example.com/myapp:${{ github.sha }}
          kubectl rollout status deployment/myapp --timeout=300s

      - name: Health check (post-deploy)
        id: healthcheck
        run: |
          for i in {1..10}; do
            if curl -sf https://myapp.example.com/health; then
              echo "Health check passed ($i/10)"
            else
              echo "Health check FAILED ($i/10)"
              exit 1
            fi
            sleep 5
          done

      - name: Smoke tests
        id: smoke
        run: npm run test:smoke

      - name: Rollback on failure
        if: failure() && (steps.healthcheck.outcome == 'failure' || steps.smoke.outcome == 'failure')
        run: |
          echo "⚠ Deployment failed! Rolling back..."
          kubectl rollout undo deployment/myapp
          kubectl rollout status deployment/myapp --timeout=120s
          echo "✓ Rollback complete"

      - name: Notify on rollback
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "⚠️ Deployment rolled back! Commit: ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Database Rollback Challenges

```
  DATABASE ROLLBACK — THE HARD PROBLEM
  ═════════════════════════════════════

  EASY: Additive schema changes (safe to rollback)
  ┌────────────────────────────────────────────┐
  │  Migration UP:   ALTER TABLE ADD COLUMN    │
  │  Migration DOWN: ALTER TABLE DROP COLUMN   │
  │  ✓ Forward compatible, backward compatible │
  └────────────────────────────────────────────┘

  HARD: Destructive schema changes
  ┌────────────────────────────────────────────┐
  │  Migration UP:   ALTER TABLE DROP COLUMN   │
  │  Migration DOWN: ??? Column data is lost!  │
  │  ✕ Data loss makes rollback impossible     │
  └────────────────────────────────────────────┘

  DATA MIGRATION: Even harder
  ┌────────────────────────────────────────────┐
  │  Migration UP:   Transform user data       │
  │  Migration DOWN: Reverse transformation?   │
  │  ✕ May not be reversible                   │
  └────────────────────────────────────────────┘

  SOLUTION: Expand-and-contract pattern
  Phase 1 (Deploy v2): Add new column, write to both
  Phase 2 (Verify):    Read from new column
  Phase 3 (Cleanup):   Drop old column (later, separate deploy)
  → Rollback is always safe during Phase 1 and 2
```

---

## Rollback Procedures Checklist

```
  ┌──────────────────────────────────────────────────┐
  │          ROLLBACK RUNBOOK                        │
  │                                                  │
  │  BEFORE DEPLOYING:                               │
  │  □ Record current version/image tag              │
  │  □ Verify rollback path works                    │
  │  □ Ensure DB migrations are backward-compatible  │
  │  □ Define rollback criteria (error rate > 5%)    │
  │                                                  │
  │  DURING ISSUE:                                   │
  │  □ Confirm the issue is caused by the deploy     │
  │  □ Assess severity (P1-P4)                       │
  │  □ Decide: rollback vs hotfix                    │
  │                                                  │
  │  EXECUTING ROLLBACK:                             │
  │  □ Execute rollback command                      │
  │  □ Verify health checks pass                     │
  │  □ Confirm metrics return to normal              │
  │  □ Notify team + stakeholders                    │
  │                                                  │
  │  AFTER ROLLBACK:                                 │
  │  □ Root cause analysis                           │
  │  □ Fix the issue on development branch           │
  │  □ Add tests to prevent recurrence               │
  │  □ Document in incident post-mortem              │
  └──────────────────────────────────────────────────┘
```

---

## Rollback Speed Comparison

```
  STRATEGY                ROLLBACK TIME    COMPLEXITY
  ═══════════════════════════════════════════════════

  Feature flag toggle     ██               Seconds
  Blue-Green switch       ███              Seconds
  K8s rollout undo        █████            ~30s
  Canary weight → 0%      ███              Seconds
  ECS task revert         ████████         ~1-2 min
  Redeploy previous       ██████████████   ~5-10 min
  Git revert + pipeline   ████████████████████ ~10-15 min
  DB migration rollback   ████████████████████████████ ~30+ min
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Rollback also fails | Shared dependency changed | Test rollback path before deploying |
| Data written by v2 breaks v1 | Schema incompatibility | Use expand-and-contract migrations |
| Can't identify previous version | No version tracking | Tag images with git SHA; maintain history |
| Cache serves stale/new content | CDN/Redis cache | Invalidate caches during rollback |
| Rollback too slow | Full pipeline redeploy | Use K8s rollout undo or Blue-Green switch |
| Users lost data during rollback | New data in v2 format | Design data models for backward compatibility |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Purpose** | Revert to working version when deployment fails |
| **Fastest** | Feature flags and Blue-Green (seconds) |
| **Standard** | Kubernetes rollback (30 seconds) |
| **Slowest** | Database rollback (30+ minutes) |
| **Key practice** | Always have a tested rollback plan |
| **Hardest part** | Database/data rollback |

---

## Quick Revision Questions

1. **What is a rollback?** *Reverting to a previous working version of an application when a new deployment causes issues.*
2. **What is the fastest rollback method?** *Feature flag kill switch — instantly disables a feature without redeployment.*
3. **How does `kubectl rollout undo` work?** *It reverts a Kubernetes deployment to its previous revision by re-applying the old pod template.*
4. **Why is database rollback the hardest?** *Destructive schema changes (dropped columns) and data transformations may not be reversible.*
5. **What is the expand-and-contract pattern?** *Add new columns alongside old ones (expand), migrate data, then remove old columns (contract) — ensuring rollback safety at each phase.*
6. **What should a rollback runbook include?** *Pre-deploy checks, rollback criteria/triggers, execution steps, verification checks, and post-rollback analysis.*

---

[← Previous: Feature Flags](05-feature-flags.md) | [Back to README](../README.md) | [Next: Zero-Downtime Deployments →](07-zero-downtime-deployments.md)
