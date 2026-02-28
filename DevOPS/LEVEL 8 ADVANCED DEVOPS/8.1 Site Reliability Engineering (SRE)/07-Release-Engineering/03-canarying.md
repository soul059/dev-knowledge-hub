# Chapter 7.3: Canarying

[← Previous: Progressive Rollouts](02-progressive-rollouts.md) | [Back to README](../README.md) | [Next: Feature Flags →](04-feature-flags.md)

---

## Overview

Canarying is a release technique named after the "canary in a coal mine" — a small group of users or servers receives the new version first, serving as an early warning system. If the canary is healthy, the release proceeds. If not, it's rolled back before most users are affected. Canarying is the most effective technique for catching production-only bugs.

---

## 1. Canary Deployment Architecture

```
  ┌──────────────────────────────────────────────────────────┐
  │  CANARY DEPLOYMENT                                       │
  │                                                          │
  │         Incoming Traffic (100%)                           │
  │                │                                         │
  │                ▼                                         │
  │     ┌──────────────────────┐                             │
  │     │    Load Balancer     │                              │
  │     │  (traffic splitting) │                              │
  │     └─────┬──────────┬─────┘                             │
  │           │          │                                   │
  │      95% ▼          ▼ 5%                                 │
  │   ┌──────────┐  ┌──────────┐                             │
  │   │ STABLE   │  │ CANARY   │                             │
  │   │ v1.0     │  │ v2.0     │                             │
  │   │          │  │          │                             │
  │   │ 9 pods   │  │ 1 pod    │                             │
  │   └──────────┘  └──────────┘                             │
  │        │              │                                  │
  │        │              ▼                                  │
  │        │     ┌──────────────────┐                        │
  │        │     │ Canary Analysis  │                        │
  │        │     │ • Error rate     │                        │
  │        │     │ • Latency        │                        │
  │        │     │ • CPU/Memory     │                        │
  │        │     │ • Business KPIs  │                        │
  │        │     └────────┬─────────┘                        │
  │        │              │                                  │
  │        │     ┌────────┴─────────┐                        │
  │        │     │                  │                        │
  │        │  ✓ Pass            ✗ Fail                       │
  │        │  Promote           Rollback                     │
  │        ▼  to 100%          to v1.0                       │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Traffic Splitting Methods

```
  ┌──────────────────────────────────────────────────────────┐
  │  METHOD           │ HOW                  │ PROS/CONS     │
  ├───────────────────┼──────────────────────┼───────────────┤
  │ Pod ratio         │ 9 stable pods,       │ Simple but    │
  │ (Kubernetes)      │ 1 canary pod         │ coarse (10%   │
  │                   │ = 10% canary traffic │ minimum)      │
  │                   │                      │               │
  │ Service mesh      │ Istio/Linkerd route  │ Fine-grained  │
  │ (Istio)           │ exact percentage     │ (1% possible) │
  │                   │                      │ More complex  │
  │                   │                      │               │
  │ Ingress-based     │ Nginx ingress        │ No mesh       │
  │ (Nginx)           │ canary annotations   │ needed.       │
  │                   │                      │ Limited rules │
  │                   │                      │               │
  │ Header-based      │ Route by header      │ Good for      │
  │                   │ (X-Canary: true)     │ internal      │
  │                   │                      │ testing       │
  │                   │                      │               │
  │ User-segment      │ Route by user ID,    │ Consistent    │
  │                   │ geography, cohort    │ user experience│
  └───────────────────┴──────────────────────┴───────────────┘
```

---

## 3. Istio Canary Configuration

```yaml
# Istio VirtualService — precise traffic splitting
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: api-service
spec:
  hosts:
    - api-service
  http:
    - route:
        - destination:
            host: api-service
            subset: stable
          weight: 95
        - destination:
            host: api-service
            subset: canary
          weight: 5

---
# DestinationRule — define subsets
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: api-service
spec:
  host: api-service
  subsets:
    - name: stable
      labels:
        version: v1.0
    - name: canary
      labels:
        version: v2.0

---
# Nginx Ingress canary (simpler, no mesh required)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-canary
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "5"
    # Or route by header:
    # nginx.ingress.kubernetes.io/canary-by-header: "X-Canary"
spec:
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service-canary
                port:
                  number: 80
```

---

## 4. Canary Analysis Techniques

```
  ┌──────────────────────────────────────────────────────────┐
  │  CANARY ANALYSIS METHODS                                 │
  │                                                          │
  │  1. THRESHOLD-BASED                                      │
  │     "Error rate must be < 1%"                            │
  │     Simple but misses subtle regressions.                │
  │     Good for: obvious failures.                          │
  │                                                          │
  │  2. RELATIVE COMPARISON                                  │
  │     "Canary error rate must be within 1.5× of baseline" │
  │     Adapts to current conditions.                        │
  │     Good for: most scenarios.                            │
  │                                                          │
  │  3. STATISTICAL ANALYSIS (best)                          │
  │     ┌─────────────────────────────────────────┐          │
  │     │  Canary Distribution                    │          │
  │     │    ╱╲                                   │          │
  │     │   ╱  ╲        Baseline Distribution     │          │
  │     │  ╱    ╲          ╱╲                     │          │
  │     │ ╱      ╲        ╱  ╲                    │          │
  │     │╱        ╲      ╱    ╲                   │          │
  │     └─────────────────────────────────────────┘          │
  │     │                                                    │
  │     Use Mann-Whitney U test or Kolmogorov-Smirnov       │
  │     test to determine if distributions are               │
  │     statistically different. p-value < 0.05 = fail.     │
  │                                                          │
  │  4. BUSINESS METRIC ANALYSIS                             │
  │     Compare conversion rates, revenue per user,          │
  │     engagement metrics between canary and baseline.      │
  │     Requires enough traffic for significance.            │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Canary Pitfalls

```
  ┌──────────────────────────────────────────────────────────┐
  │  COMMON CANARY MISTAKES                                  │
  │                                                          │
  │  1. TOO LITTLE TRAFFIC                                   │
  │     ├─ Canary at 1% with 100 RPS = 1 RPS on canary     │
  │     ├─ Not enough data for statistical significance      │
  │     └─ FIX: Ensure canary gets >50 RPS minimum          │
  │                                                          │
  │  2. TOO SHORT BAKE TIME                                  │
  │     ├─ Memory leaks won't show in 5 minutes             │
  │     ├─ Time-dependent bugs (midnight cron, weekly job)   │
  │     └─ FIX: Minimum 30 min for most changes             │
  │                                                          │
  │  3. WRONG METRICS                                        │
  │     ├─ Only monitoring server errors, missing client     │
  │     │   errors, timeouts, or business metric drops       │
  │     └─ FIX: Monitor errors, latency, AND business KPIs  │
  │                                                          │
  │  4. INCONSISTENT TRAFFIC                                 │
  │     ├─ Canary gets different user segment than baseline  │
  │     ├─ E.g., canary gets only mobile, baseline desktop  │
  │     └─ FIX: Random traffic splitting, not segment-based │
  │                                                          │
  │  5. DATABASE SCHEMA MISMATCH                             │
  │     ├─ Canary needs new DB column that doesn't exist    │
  │     ├─ Or old version breaks with new schema            │
  │     └─ FIX: Forward-compatible migrations always        │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Automated Canary Pipeline

```
  ┌──────────────────────────────────────────────────────────┐
  │  FULLY AUTOMATED CANARY FLOW                             │
  │                                                          │
  │  PR Merged                                               │
  │      │                                                   │
  │      ▼                                                   │
  │  CI builds + tests image                                 │
  │      │                                                   │
  │      ▼                                                   │
  │  Deploy canary (5% traffic)                              │
  │      │                                                   │
  │      ▼                                                   │
  │  Wait 10 min (bake time)                                 │
  │      │                                                   │
  │  ┌───▼───────────────────────────────────────────┐       │
  │  │  Canary Analyzer runs:                       │       │
  │  │  query prometheus for canary vs baseline     │       │
  │  │                                               │       │
  │  │  Metrics checked:                             │       │
  │  │  ├─ success_rate >= 99.5%          ✓ PASS    │       │
  │  │  ├─ p99_latency <= 300ms           ✓ PASS    │       │
  │  │  ├─ memory_usage < 500MB           ✓ PASS    │       │
  │  │  └─ error_log_rate < 10/min        ✓ PASS    │       │
  │  │                                               │       │
  │  │  Result: ALL PASS                             │       │
  │  └───┬───────────────────────────────────────────┘       │
  │      │                                                   │
  │      ▼                                                   │
  │  Promote: 5% → 25% → 50% → 100%                        │
  │  (each stage with health check)                          │
  │      │                                                   │
  │      ▼                                                   │
  │  Notify Slack: "v2.1.0 fully deployed ✓"                │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] Canary Detects Subtle Performance Regression

```
  RELEASE: Search service v4.1.0 (query optimizer change)
  
  Canary deployment: 5% traffic
  Initial metrics after 15 minutes:
  ┌──────────────────────────────────────────────────────────┐
  │  METRIC          │ BASELINE (v4.0) │ CANARY (v4.1)      │
  ├──────────────────┼─────────────────┼────────────────────┤
  │ Error rate       │ 0.08%           │ 0.07%    ✓ OK      │
  │ p50 latency      │ 45ms            │ 48ms     ✓ OK      │
  │ p99 latency      │ 180ms           │ 350ms    ✗ FAIL    │
  │ p999 latency     │ 450ms           │ 2,100ms  ✗✗ FAIL   │
  │ CPU usage        │ 35%             │ 52%      ⚠ WARNING │
  └──────────────────┴─────────────────┴────────────────────┘
  
  Analysis:
  ├─ Error rate looks fine — wouldn't catch with basic checks
  ├─ p50 looks fine — median users see normal performance
  ├─ p99 is 2× worse — tail latency heavily degraded
  ├─ p999 is 4.7× worse — worst-case queries are terrible
  
  Root cause investigation:
  ├─ New query optimizer performs well for simple queries
  ├─ BUT for complex multi-join queries (top 1% of traffic),
  │  it generates suboptimal execution plans
  └─ These complex queries now scan 10× more rows
  
  Action:
  ├─ Canary automatically rolled back at 10:17
  ├─ v4.1.1 shipped with optimizer handling for complex queries
  ├─ Re-deployed with canary: all percentiles improved
  └─ Full rollout completed successfully
  
  KEY LESSON: Always monitor tail latency (p99/p999),
  not just averages/medians. Canary saved 100% of users
  from experiencing 2x worse tail latency.
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Canary passes but full rollout breaks" | Canary may not have covered all code paths. Add business-specific tests. Extend bake time. |
| "Canary traffic is too low for analysis" | Increase canary weight or use header-based routing to direct specific test traffic. |
| "A/B metrics don't match canary results" | Ensure consistent user segmentation. Canary = technical validation, A/B = product validation. |
| "Canary rollback caused a brief spike in errors" | Use graceful draining. Set `terminationGracePeriodSeconds` to allow in-flight requests to complete. |

---

## Summary Table

| Aspect | Recommendation |
|--------|---------------|
| **Traffic weight** | Start at 1-5%, increase in 2-4 stages |
| **Minimum bake time** | 30 min for most changes |
| **Key metrics** | Error rate, p50/p99/p999 latency, CPU, memory |
| **Analysis method** | Relative comparison or statistical tests |
| **Rollback trigger** | >2× baseline on any key metric |
| **Traffic splitting** | Service mesh (Istio) for precision, Nginx canary for simplicity |

---

## Quick Revision Questions

1. **What is canarying and why is it named that?**
   > Named after canaries used in coal mines as early warning systems. A small percentage of traffic (canary) receives the new version. If the canary is unhealthy, the release is rolled back before most users are affected.

2. **What's the difference between pod-ratio and service-mesh based canary?**
   > Pod-ratio uses the number of pods (1 canary out of 10 = 10% traffic) — it's coarse-grained. Service mesh (Istio) can route exact percentages (e.g., 1%) regardless of pod count — it's fine-grained.

3. **Why is monitoring tail latency (p99/p999) critical during canarying?**
   > Regressions often affect only a subset of requests (complex queries, edge cases). Medians and averages hide these issues. p99/p999 catches regressions that affect the worst 1-0.1% of requests.

4. **What are the common pitfalls of canary deployment?**
   > Too little traffic (no statistical significance), too short bake time (misses slow bugs), wrong metrics (only errors, not latency or business metrics), inconsistent traffic, and schema mismatches.

5. **How does statistical canary analysis work?**
   > It compares the distribution of metrics (latency, error rate) between canary and baseline using statistical tests (Mann-Whitney U). If distributions are statistically different (p < 0.05), the canary fails.

---

[← Previous: Progressive Rollouts](02-progressive-rollouts.md) | [Back to README](../README.md) | [Next: Feature Flags →](04-feature-flags.md)
