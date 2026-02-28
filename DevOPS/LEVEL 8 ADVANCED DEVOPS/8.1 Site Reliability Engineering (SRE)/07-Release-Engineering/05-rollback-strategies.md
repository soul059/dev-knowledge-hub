# Chapter 7.5: Rollback Strategies

[← Previous: Feature Flags](04-feature-flags.md) | [Back to README](../README.md) | [Next: Change Management →](06-change-management.md)

---

## Overview

Rollback is the ability to revert a system to a previous known-good state when a deployment causes problems. Fast, reliable rollback is the primary safety net for production deployments. SRE teams must design systems where rollback is always possible, always tested, and faster than fix-forward.

---

## 1. Rollback Decision Framework

```
  ┌──────────────────────────────────────────────────────────┐
  │  INCIDENT DETECTED AFTER DEPLOYMENT                      │
  │                                                          │
  │  Is the issue clearly caused by the deployment?          │
  │  ├─ YES                                                  │
  │  │   Can we fix-forward in < 15 minutes?                │
  │  │   ├─ YES ──▶ Fix-forward (quick hotfix)              │
  │  │   └─ NO                                               │
  │  │       Is rollback safe (no data migration)?          │
  │  │       ├─ YES ──▶ ROLLBACK immediately                │
  │  │       └─ NO  ──▶ Feature flag disable OR             │
  │  │                  partial rollback + data fix          │
  │  │                                                       │
  │  └─ UNCLEAR                                              │
  │      Is the issue SEV-1 or SEV-2?                       │
  │      ├─ YES ──▶ ROLLBACK (assume deployment caused it)  │
  │      └─ NO  ──▶ Investigate first (15 min max)          │
  │                  then rollback if not resolved            │
  │                                                          │
  │  GOLDEN RULE: When in doubt, roll back.                  │
  │  You can always re-deploy after investigating.           │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Rollback Methods

```
  ┌──────────────────────────────────────────────────────────┐
  │  METHOD            │ SPEED    │ HOW IT WORKS             │
  ├────────────────────┼──────────┼──────────────────────────┤
  │ Container image    │ 30-60s   │ Point deployment to      │
  │ revert             │          │ previous image tag       │
  │                    │          │ kubectl rollout undo     │
  │                    │          │                          │
  │ Blue-green swap    │ <10s     │ Switch LB from green     │
  │                    │          │ back to blue             │
  │                    │          │                          │
  │ Feature flag       │ <5s      │ Disable flag, code stays │
  │ disable            │          │ deployed but inactive    │
  │                    │          │                          │
  │ Git revert +       │ 5-15min  │ Revert commit, trigger   │
  │ redeploy           │          │ full CI/CD pipeline      │
  │                    │          │                          │
  │ Infrastructure     │ 2-10min  │ terraform apply with     │
  │ rollback (IaC)     │          │ previous state file      │
  │                    │          │                          │
  │ Database rollback  │ 10-30min │ Run reverse migration    │
  │ (dangerous!)       │          │ or restore from backup   │
  └────────────────────┴──────────┴──────────────────────────┘
```

---

## 3. Kubernetes Rollback

```yaml
# Kubernetes maintains rollout history by default
# View rollout history
# kubectl rollout history deployment/api-service

# ROLLBACK to previous version (one command)
# kubectl rollout undo deployment/api-service

# ROLLBACK to specific revision
# kubectl rollout undo deployment/api-service --to-revision=3

# Deployment config for rollback readiness
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  replicas: 5
  revisionHistoryLimit: 10    # Keep 10 previous ReplicaSets
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0       # Zero downtime rollback
  template:
    spec:
      containers:
        - name: api
          image: api-service:a1b2c3d  # Git SHA tag (immutable)
          
          # Readiness gate ensures pods are ready before 
          # old pods are terminated
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            periodSeconds: 5
            failureThreshold: 3
      
      # Grace period for in-flight requests during rollback
      terminationGracePeriodSeconds: 60
```

---

## 4. Database-Aware Rollback

```
  ┌──────────────────────────────────────────────────────────┐
  │  THE DATABASE ROLLBACK CHALLENGE                         │
  │                                                          │
  │  Code rollback + DB schema change = DANGER               │
  │                                                          │
  │  Timeline (problematic):                                 │
  │  1. v1.0 running (schema v1)                             │
  │  2. Deploy v2.0 with migration (schema v2)               │
  │  3. v2.0 has a bug!                                      │
  │  4. Rollback code to v1.0... but schema is now v2!       │
  │  5. v1.0 code can't work with v2 schema ← BROKEN!       │
  │                                                          │
  │  ═══════════════════════════════════════════════════════  │
  │                                                          │
  │  SOLUTION: Expand-Contract Pattern                       │
  │                                                          │
  │  Phase 1 (EXPAND): Add new column/table alongside old    │
  │  ┌──────────────────────────────┐                        │
  │  │ users table                  │                        │
  │  │ ├─ name (old column)        │                        │
  │  │ ├─ first_name (new column)  │ ← added, nullable     │
  │  │ └─ last_name (new column)   │ ← added, nullable     │
  │  └──────────────────────────────┘                        │
  │  Deploy v2.0: writes to BOTH old and new columns        │
  │  v1.0 rollback safe: still reads from old column        │
  │                                                          │
  │  Phase 2 (MIGRATE): Backfill data, switch reads          │
  │  After v2.0 is stable: v2.1 reads from new columns     │
  │                                                          │
  │  Phase 3 (CONTRACT): Remove old column (separate deploy) │
  │  After v2.1 is stable: remove old column                │
  │                                                          │
  │  RULE: Never break backward compatibility in a single   │
  │  deployment. Always maintain N-1 compatibility.          │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Automated Rollback Triggers

```yaml
# Automated rollback configuration
rollback_policy:
  service: "api-service"
  
  # Automatic rollback triggers
  triggers:
    - name: "error_rate_spike"
      metric: "rate(http_requests_total{status=~'5..'}[5m]) / rate(http_requests_total[5m])"
      threshold: 0.02           # 2% error rate
      comparison: "greater_than"
      evaluation_window: "5m"
      
    - name: "latency_degradation"
      metric: "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))"
      threshold: 0.5            # 500ms p99
      comparison: "greater_than"
      evaluation_window: "5m"
      
    - name: "crash_loop"
      metric: "kube_pod_container_status_restarts_total"
      threshold: 3              # 3 restarts in window
      comparison: "greater_than"
      evaluation_window: "10m"
      
    - name: "health_check_failure"
      metric: "up{job='api-service'}"
      threshold: 0.5            # Less than 50% pods healthy
      comparison: "less_than"
      evaluation_window: "2m"

  # Rollback behavior
  rollback:
    method: "kubectl_rollout_undo"
    max_rollbacks_per_hour: 2    # Prevent rollback loops
    notification_channels:
      - "slack:#deployments"
      - "pagerduty:sre-team"
    cooldown_after_rollback: "30m"  # No deploys for 30 min
```

---

## 6. Rollback Testing

```
  ┌──────────────────────────────────────────────────────────┐
  │  ROLLBACK TESTING STRATEGIES                             │
  │                                                          │
  │  1. PRE-PRODUCTION ROLLBACK TESTS                        │
  │     ├─ Deploy v2.0 to staging                            │
  │     ├─ Run smoke tests                                   │
  │     ├─ Rollback to v1.0                                  │
  │     ├─ Run smoke tests again                             │
  │     └─ Verify: data intact, no errors, full function    │
  │                                                          │
  │  2. ROLLBACK DRILLS (quarterly)                          │
  │     ├─ Intentionally trigger rollback in production      │
  │     ├─ Measure actual rollback time                      │
  │     ├─ Verify monitoring catches the event               │
  │     └─ Document any issues found                        │
  │                                                          │
  │  3. SCHEMA COMPATIBILITY TESTS                           │
  │     ├─ Run v1.0 code against v2.0 schema                │
  │     ├─ Run v2.0 code against v1.0 schema                │
  │     └─ Both must work for safe rollback                 │
  │                                                          │
  │  4. CI/CD PIPELINE VALIDATION                            │
  │     ├─ Every PR must include rollback plan               │
  │     ├─ Breaking schema changes require 2-phase deploy   │
  │     └─ Automated check: "Is this change rollback-safe?" │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] Rollback During a Payments Outage

```
  INCIDENT: Payment service v5.3.0 deployed at 14:00
  
  14:00 — Deploy v5.3.0 via canary (5%)
  14:15 — Canary analysis passes ✓
  14:20 — Rolled to 100%
  14:25 — Alert: Payment success rate drops from 99.8% → 94%
  
  ┌──────────────────────────────────────────────────────────┐
  │  14:25 — INCIDENT DECLARED (SEV-1)                       │
  │                                                          │
  │  14:26 — IC confirms: issue started exactly at deploy    │
  │          Decision: ROLLBACK (not fix-forward)            │
  │          Reason: payments are critical, >5% affected     │
  │                                                          │
  │  14:27 — Check: any database migrations in v5.3.0?      │
  │          Answer: YES — new column added (nullable)       │
  │          Assessment: Safe to rollback (expand phase,     │
  │          old code doesn't read new column)               │
  │                                                          │
  │  14:28 — kubectl rollout undo deployment/payment-service │
  │          Rolling update begins: v5.3.0 → v5.2.0         │
  │                                                          │
  │  14:29 — First v5.2.0 pods receiving traffic            │
  │          Payment success rate: 94% → 96%                │
  │                                                          │
  │  14:31 — All pods on v5.2.0                             │
  │          Payment success rate: 99.7% ← recovered!       │
  │                                                          │
  │  14:35 — Incident resolved. Total impact: 6 minutes     │
  │                                                          │
  │  RCA: v5.3.0 introduced a race condition in the         │
  │  payment idempotency check, causing ~6% of concurrent   │
  │  payments to fail validation.                            │
  │                                                          │
  │  FIX: v5.3.1 with proper locking. Deployed next day     │
  │  with extended canary bake time (2 hours at 5%).        │
  └──────────────────────────────────────────────────────────┘
  
  KEY TAKEAWAYS:
  ├─ Rollback took 3 minutes (14:28 → 14:31)
  ├─ Total user impact: 6 minutes at 94% success rate
  ├─ Expand-contract pattern made DB rollback safe
  └─ Without rollback: could have been 30+ min to fix-forward
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Can't rollback due to database migration" | Use expand-contract pattern. Never drop columns on deploy. Always maintain backward compatibility. |
| "Rollback caused a different problem" | Test rollbacks in staging. Ensure N-1 version compatibility. Check for external state changes (cache, queues). |
| "Automated rollback keeps triggering (flapping)" | Set cooldown periods (no deploy for 30 min after rollback). Set max rollback count per hour. Investigate root cause. |
| "We don't know which version to rollback to" | Use immutable image tags (Git SHA). Maintain `revisionHistoryLimit` > 5 in deployments. Tag production images. |

---

## Summary Table

| Method | Speed | Risk | Best For |
|--------|-------|------|----------|
| **Feature flag toggle** | <5s | Almost none | Feature-level issues |
| **Blue-green swap** | <10s | Low | Full version rollback |
| **K8s rollout undo** | 30-60s | Low | Container workloads |
| **Git revert + redeploy** | 5-15 min | Low | When history matters |
| **DB reverse migration** | 10-30 min | HIGH | Avoid if possible |

---

## Quick Revision Questions

1. **When should you rollback vs fix-forward?**
   > Rollback when: the issue is deployment-related, you can't fix in <15 minutes, or it's a SEV-1/SEV-2 impacting users. Fix-forward when: the fix is simple and faster than rollback (e.g., config typo).

2. **What is the expand-contract pattern and why is it important?**
   > A 3-phase approach to schema changes: (1) Expand — add new columns without removing old ones, (2) Migrate — backfill data, (3) Contract — remove old columns in a later deploy. It ensures backward compatibility so rolling back code won't break against the new schema.

3. **How does Kubernetes support rollback?**
   > `kubectl rollout undo` reverts to the previous ReplicaSet. Kubernetes keeps old ReplicaSets based on `revisionHistoryLimit`. You can rollback to specific revisions.

4. **What should trigger an automatic rollback?**
   > Error rate exceeding threshold (e.g., 2% 5xx), p99 latency exceeding threshold, pod crash loops, or health check failures across >50% of pods.

5. **Why should you test rollbacks regularly?**
   > Untested rollbacks fail when you need them most. Schema changes, configuration drift, or external dependencies can make rollback impossible. Quarterly drills validate that rollback actually works.

6. **What's the golden rule of rollback?**
   > When in doubt, roll back. You can always investigate and redeploy later. Every minute of hesitation during a production issue costs money and user trust.

---

[← Previous: Feature Flags](04-feature-flags.md) | [Back to README](../README.md) | [Next: Change Management →](06-change-management.md)
