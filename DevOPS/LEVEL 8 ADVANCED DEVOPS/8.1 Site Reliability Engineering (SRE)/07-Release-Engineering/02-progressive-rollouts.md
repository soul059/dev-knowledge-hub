# Chapter 7.2: Progressive Rollouts

[← Previous: Release Management](01-release-management.md) | [Back to README](../README.md) | [Next: Canarying →](03-canarying.md)

---

## Overview

Progressive rollouts gradually expose new software versions to increasing percentages of users or traffic, allowing teams to detect issues early with minimal blast radius. Instead of deploying to 100% of users at once, traffic shifts incrementally — 1% → 5% → 25% → 50% → 100% — with automated health checks at each stage.

---

## 1. Progressive Rollout Flow

```
  ┌──────────────────────────────────────────────────────────┐
  │  PROGRESSIVE ROLLOUT STAGES                              │
  │                                                          │
  │  Stage 1     Stage 2     Stage 3     Stage 4     Stage 5 │
  │  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐  │
  │  │  1%  │──▶│  5%  │──▶│ 25%  │──▶│ 50%  │──▶│ 100% │  │
  │  └──────┘   └──────┘   └──────┘   └──────┘   └──────┘  │
  │  10 min      15 min      30 min      30 min     done    │
  │  │            │            │            │                │
  │  Check:      Check:      Check:      Check:             │
  │  • Errors    • Errors    • Errors    • Errors           │
  │  • Latency   • Latency   • Latency   • Latency         │
  │  • Logs      • Logs      • CPU/mem   • CPU/mem          │
  │                                                          │
  │  At ANY stage, if metrics degrade:                       │
  │      ┌──────────────────┐                                │
  │      │ AUTOMATIC ROLLBACK│                               │
  │      │ to previous version│                              │
  │      └──────────────────┘                                │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Rollout Strategy by Risk Level

```
  ┌──────────────────────────────────────────────────────────┐
  │  CHANGE TYPE      │ STRATEGY              │ BAKE TIME    │
  ├───────────────────┼───────────────────────┼──────────────┤
  │ Low risk          │ 10% → 50% → 100%     │ 15 min/stage │
  │ (minor bugfix,    │                       │              │
  │  dependency bump) │                       │              │
  │                   │                       │              │
  │ Medium risk       │ 1% → 5% → 25% →     │ 30 min/stage │
  │ (new feature,     │ 50% → 100%           │              │
  │  refactoring)     │                       │              │
  │                   │                       │              │
  │ High risk         │ 1% → 5% → 10% →     │ 1 hr/stage   │
  │ (database change, │ 25% → 50% → 100%    │              │
  │  core logic)      │                       │              │
  │                   │                       │              │
  │ Critical          │ Shadow → 1% → 5% →  │ 4 hr/stage   │
  │ (payment logic,   │ 10% → 25% → 50% →   │              │
  │  auth changes)    │ 100%                  │              │
  └───────────────────┴───────────────────────┴──────────────┘
```

---

## 3. Kubernetes Rolling Update Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%           # Max 3 new pods above desired
      maxUnavailable: 0       # Zero downtime — never reduce
  template:
    spec:
      containers:
        - name: api
          image: api-service:v2.1.0
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            periodSeconds: 10
      # Draining time for in-flight requests
      terminationGracePeriodSeconds: 60
```

---

## 4. Argo Rollouts — Progressive Delivery

```yaml
# Argo Rollouts — canary with automated analysis
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: api-service
spec:
  replicas: 10
  strategy:
    canary:
      steps:
        # Step 1: Send 5% traffic to canary
        - setWeight: 5
        - pause: { duration: 10m }
        
        # Step 2: Run automated analysis
        - analysis:
            templates:
              - templateName: success-rate
            args:
              - name: service-name
                value: api-service
        
        # Step 3: Increase to 25%
        - setWeight: 25
        - pause: { duration: 15m }
        
        # Step 4: Another analysis check
        - analysis:
            templates:
              - templateName: success-rate
        
        # Step 5: Increase to 50%
        - setWeight: 50
        - pause: { duration: 15m }
        
        # Step 6: Full rollout
        - setWeight: 100

      # Automatic rollback conditions
      abortScaleDownDelaySeconds: 30
      
      # Traffic routing (Istio/Nginx)
      trafficRouting:
        istio:
          virtualService:
            name: api-service-vsvc
            routes:
              - primary

---
# Analysis template — automated health check
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
    - name: service-name
  metrics:
    - name: success-rate
      interval: 60s
      count: 5
      successCondition: result[0] >= 0.99  # 99% success
      failureLimit: 2
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            sum(rate(http_requests_total{
              service="{{args.service-name}}",
              status=~"2.."
            }[5m])) /
            sum(rate(http_requests_total{
              service="{{args.service-name}}"
            }[5m]))
    
    - name: p99-latency
      interval: 60s
      count: 5
      successCondition: result[0] <= 0.5   # p99 < 500ms
      failureLimit: 2
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            histogram_quantile(0.99,
              sum(rate(http_request_duration_seconds_bucket{
                service="{{args.service-name}}"
              }[5m])) by (le)
            )
```

---

## 5. Multi-Region Rollout

```
  ┌──────────────────────────────────────────────────────────┐
  │  MULTI-REGION PROGRESSIVE ROLLOUT                        │
  │                                                          │
  │  Phase 1: Internal testing region                        │
  │  ┌────────────────────────────────────────┐              │
  │  │  Canary Region (us-east-1-canary)      │  1% traffic  │
  │  │  Internal traffic only                 │              │
  │  │  Bake time: 2 hours                    │              │
  │  └────────────────────────────────────────┘              │
  │                    │ ✓ pass                              │
  │                    ▼                                     │
  │  Phase 2: Low-traffic region                             │
  │  ┌────────────────────────────────────────┐              │
  │  │  us-west-2 (smallest region)           │  10% traffic │
  │  │  Bake time: 4 hours                    │              │
  │  └────────────────────────────────────────┘              │
  │                    │ ✓ pass                              │
  │                    ▼                                     │
  │  Phase 3: Medium-traffic regions                         │
  │  ┌────────────────────────────────────────┐              │
  │  │  eu-west-1, ap-southeast-1             │  40% traffic │
  │  │  Bake time: 4 hours                    │              │
  │  └────────────────────────────────────────┘              │
  │                    │ ✓ pass                              │
  │                    ▼                                     │
  │  Phase 4: Primary region                                 │
  │  ┌────────────────────────────────────────┐              │
  │  │  us-east-1 (largest region)            │  100% traffic│
  │  │  Bake time: 24 hours monitoring        │              │
  │  └────────────────────────────────────────┘              │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Rollout Health Signals

```
  ┌──────────────────────────────────────────────────────────┐
  │  WHAT TO MONITOR DURING PROGRESSIVE ROLLOUT              │
  │                                                          │
  │  METRIC              │ COMPARE AGAINST  │ ABORT IF       │
  ├──────────────────────┼──────────────────┼────────────────┤
  │ Error rate (5xx)     │ Baseline (old ver) │ > 2× baseline │
  │ Error rate (4xx)     │ Baseline         │ > 1.5× baseline│
  │ p50 latency          │ Baseline         │ > 1.5× baseline│
  │ p99 latency          │ Baseline         │ > 2× baseline  │
  │ CPU usage            │ Expected range   │ > 90%          │
  │ Memory usage         │ Expected range   │ > 85%          │
  │ Request throughput   │ Expected (±10%)  │ Drop > 20%     │
  │ Business metrics     │ Baseline         │ Drop > 5%      │
  │ (conversion, signups)│                  │                │
  └──────────────────────┴──────────────────┴────────────────┘
  │                                                          │
  │  KEY PRINCIPLE: Compare canary vs baseline (old version) │
  │  NOT canary vs historical data. Both must receive        │
  │  similar traffic patterns for fair comparison.           │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] Progressive Rollout Catches a Memory Leak

```
  RELEASE: API service v3.2.0 (new caching layer)
  
  Timeline:
  ├─ 10:00 — Deploy canary (5% traffic)
  │          All metrics look good initially
  │
  ├─ 10:15 — Analysis pass #1: ✓
  │          Error rate: 0.05% (baseline: 0.08%) ← better!
  │          p99 latency: 180ms (baseline: 195ms) ← better!
  │          Memory: 380MB (baseline: 350MB) ← slightly higher
  │
  ├─ 10:30 — Promote to 25%
  │          Memory: 420MB ... 480MB ... growing
  │
  ├─ 10:45 — Analysis pass #2: ⚠️
  │          Memory: 580MB (and climbing at 50MB/15min)
  │          Projected OOM in 2.5 hours
  │
  ├─ 10:50 — AUTOMATED ABORT triggered
  │          Reason: memory_usage trajectory exceeds threshold
  │          Action: Rollback to v3.1.0
  │
  ├─ 10:51 — Canary pods terminated, 100% on v3.1.0
  │          Zero user impact (only 25% saw new version)
  │
  ├─ 11:00 — RCA: New caching layer had unbounded cache
  │          without TTL. Objects accumulated in memory.
  │
  ├─ 11:30 — Fix: Added TTL + max cache size
  │          v3.2.1 prepared with fix
  │
  └─ 14:00 — v3.2.1 deployed via same progressive rollout
             Memory stable at 370MB over 4 hours. Success.
  
  WITHOUT progressive rollout: Full outage after ~3 hours
  WITH progressive rollout: Zero user-visible impact
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Rollout takes too long" | Adjust bake times based on risk. Low-risk changes can use 10% → 50% → 100% with 10-min bakes. |
| "False positive rollbacks" | Widen success thresholds. Use statistical significance (need enough traffic). Increase analysis window. |
| "Can't compare canary vs baseline" | Ensure both versions receive similar traffic patterns. Use traffic mirroring or consistent hash routing. |
| "Multi-region rollout coordination is complex" | Use a rollout orchestrator (Argo Rollouts, Spinnaker). Automate region-by-region with gates. |

---

## Summary Table

| Aspect | Best Practice |
|--------|--------------|
| **Starting weight** | 1-5% for medium/high risk changes |
| **Bake time** | 10-60 min per stage (longer for higher risk) |
| **Health signals** | Error rate, latency, CPU, memory, business metrics |
| **Comparison method** | Canary vs baseline (not historical) |
| **Abort criteria** | >2× baseline error rate, >2× p99 latency |
| **Rollback speed** | <60 seconds to previous version |
| **Multi-region** | Smallest region first, primary last |

---

## Quick Revision Questions

1. **What is a progressive rollout?**
   > A deployment strategy that gradually increases the percentage of traffic routed to a new version (e.g., 1% → 5% → 25% → 100%), with automated health checks at each stage and automatic rollback if metrics degrade.

2. **Why compare canary to baseline instead of historical data?**
   > Traffic patterns vary by time of day and day of week. Comparing canary to the currently-running baseline ensures both see the same traffic mix, giving accurate performance comparison.

3. **What metrics should trigger an automatic rollback?**
   > Error rate exceeding 2× baseline, p99 latency exceeding 2× baseline, memory or CPU exceeding safe thresholds, or significant drops in business metrics.

4. **How does Argo Rollouts automate progressive delivery?**
   > It defines rollout steps (weight percentages and pause durations) with AnalysisTemplates that query Prometheus. If analysis fails, it automatically aborts and rolls back.

5. **What is the benefit of multi-region progressive rollout?**
   > Issues are caught in smaller, less critical regions before reaching the primary region. Each region serves as a gate, limiting blast radius geographically.

---

[← Previous: Release Management](01-release-management.md) | [Back to README](../README.md) | [Next: Canarying →](03-canarying.md)
