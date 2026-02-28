# Chapter 5.3: Canary Deployments

[← Previous: Blue-Green Deployments](02-blue-green-deployments.md) | [Back to README](../README.md) | [Next: A/B Testing Deployments →](04-ab-testing-deployments.md)

---

## Overview

A **canary deployment** releases the new version to a small subset of users first (the "canary"), monitors key metrics, and gradually increases traffic if everything looks good. Named after the "canary in a coal mine" — an early warning system.

---

## How Canary Deployments Work

```
  CANARY DEPLOYMENT
  ═════════════════

  Step 1: Deploy canary (5% traffic)
  ┌──────────┐     ┌──────────────┐
  │  Users   │────►│ Load Balancer│
  └──────────┘     └──────┬───────┘
                     95%  │  5%
                   ┌──────▼──────┐    ┌───────────┐
                   │ Stable (v1) │    │ Canary(v2)│
                   │ ┌──┐┌──┐┌──┐│   │ ┌──┐      │
                   │ │v1││v1││v1││    │ │v2│      │
                   │ └──┘└──┘└──┘│   │ └──┘      │
                   └─────────────┘    └───────────┘

  Step 2: Monitor (errors, latency, business metrics)
      Error rate v2 < v1?  ✓
      Latency p95 OK?      ✓
      Revenue impact?      ✓
      → Proceed to 25%

  Step 3: Increase to 25%, then 50%, then 100%
  ┌──────────┐     ┌──────────────┐
  │  Users   │────►│ Load Balancer│
  └──────────┘     └──────┬───────┘
                    100%  │
                   ┌──────▼──────┐
                   │  All v2     │
                   │ ┌──┐┌──┐┌──┐│
                   │ │v2││v2││v2││
                   │ └──┘└──┘└──┘│
                   └─────────────┘
```

---

## Canary Rollout Stages

```
  CANARY PROGRESSION
  ══════════════════

  Time ──►
  ┌─────┐  ┌──────┐  ┌───────┐  ┌────────┐  ┌──────────┐
  │  5% │─►│  10% │─►│  25%  │─►│  50%   │─►│  100%    │
  │     │  │      │  │       │  │        │  │ (promote) │
  └──┬──┘  └──┬───┘  └──┬────┘  └──┬─────┘  └──────────┘
     │        │         │          │
  Monitor  Monitor   Monitor    Monitor
  30 min   30 min    1 hour     1 hour

  At ANY stage: if metrics degrade → ROLLBACK to 0%
```

---

## Kubernetes Canary with Istio

```yaml
# VirtualService — Traffic splitting
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts: ["myapp.example.com"]
  http:
    - route:
        - destination:
            host: myapp-stable
            port: { number: 80 }
          weight: 95            # 95% to stable
        - destination:
            host: myapp-canary
            port: { number: 80 }
          weight: 5             # 5% to canary

---
# DestinationRule
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  subsets:
    - name: stable
      labels: { version: v1 }
    - name: canary
      labels: { version: v2 }
```

---

## Argo Rollouts (Automated Canary)

```yaml
# Argo Rollouts — Automated canary with analysis
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 10
  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause: { duration: 5m }         # Wait 5 min
        - analysis:                        # Run automated analysis
            templates:
              - templateName: success-rate
            args:
              - name: service-name
                value: myapp
        - setWeight: 25
        - pause: { duration: 10m }
        - setWeight: 50
        - pause: { duration: 10m }
        - setWeight: 100
      canaryService: myapp-canary
      stableService: myapp-stable
      trafficRouting:
        istio:
          virtualService:
            name: myapp-vsvc

---
# AnalysisTemplate — Automated metric check
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
    - name: service-name
  metrics:
    - name: success-rate
      interval: 1m
      successCondition: result[0] > 0.95      # >95% success rate
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            sum(rate(http_requests_total{
              service="{{args.service-name}}",
              status=~"2..",
              version="canary"
            }[5m])) /
            sum(rate(http_requests_total{
              service="{{args.service-name}}",
              version="canary"
            }[5m]))
```

---

## Metrics to Monitor During Canary

```
  ┌──────────────────────────────────────────────────┐
  │         CANARY MONITORING METRICS                │
  │                                                  │
  │  ERROR RATE                                      │
  │  Canary:  0.3%  ──vs──  Stable: 0.2%  ✓ OK     │
  │  (< 2x stable = acceptable)                     │
  │                                                  │
  │  LATENCY (p95)                                   │
  │  Canary:  180ms ──vs──  Stable: 150ms  ✓ OK     │
  │  (< 1.5x stable = acceptable)                   │
  │                                                  │
  │  SATURATION (CPU/Memory)                         │
  │  Canary:  45%   ──vs──  Stable: 40%   ✓ OK     │
  │                                                  │
  │  BUSINESS METRICS                                │
  │  Conversion:  3.1% ──vs──  Stable: 3.2% ✓ OK   │
  │  (within statistical margin)                     │
  │                                                  │
  │  ✕ FAIL triggers:                                │
  │  - Error rate > 5%                               │
  │  - p95 latency > 500ms                           │
  │  - Any 5xx spike                                 │
  │  - Conversion rate drops > 10%                   │
  └──────────────────────────────────────────────────┘
```

---

## Canary vs Blue-Green vs Rolling

| Aspect | Rolling | Blue-Green | Canary |
|---|---|---|---|
| **Traffic split** | Gradual by instance | All-or-nothing | Percentage-based |
| **Rollback speed** | Minutes | Seconds | Seconds |
| **Infra cost** | Low | 2x | Low-Medium |
| **Risk exposure** | Medium | Low | Very Low |
| **Metric-driven** | No | No | Yes |
| **Complexity** | Low | Medium | High |
| **Best for** | Standard apps | Critical apps | Data-driven teams |

---

## CI/CD Pipeline with Canary

```yaml
# GitHub Actions — Canary deploy
name: Canary Deploy
on:
  push:
    branches: [main]

jobs:
  canary:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build & push
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker push registry.example.com/myapp:${{ github.sha }}

      - name: Deploy canary (5%)
        run: |
          kubectl argo rollouts set image myapp \
            myapp=registry.example.com/myapp:${{ github.sha }}

      - name: Monitor canary
        run: |
          kubectl argo rollouts status myapp --watch --timeout 600s

      # Argo Rollouts handles auto-promotion or rollback
      # based on AnalysisTemplate results
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Not enough traffic to canary | 5% of low traffic = too few requests | Use minimum absolute request count for analysis |
| Metrics noisy at low traffic | Small sample size | Increase observation window; wait longer |
| Canary promoted despite issues | Analysis thresholds too lenient | Tighten success conditions; add more metrics |
| Traffic not splitting correctly | Service mesh misconfigured | Verify Istio/Nginx ingress rules; test with headers |
| Database changes break canary | Migration ran on shared DB | Use expand-and-contract; version API endpoints |
| Rollback doesn't clear effects | Some users got corrupted data | Implement data reconciliation; use feature flags |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **How it works** | Route small % of traffic to new version; increase gradually |
| **Downtime** | Zero |
| **Rollback** | Instant — set canary weight to 0% |
| **Risk** | Very low — only small percentage of users affected |
| **Requires** | Service mesh or smart load balancer; monitoring |
| **Best for** | Teams with good observability; data-driven decisions |

---

## Quick Revision Questions

1. **What is a canary deployment?** *Releasing the new version to a small percentage of traffic first, monitoring metrics, and gradually increasing traffic if metrics are healthy.*
2. **Why is it called "canary"?** *Named after the "canary in a coal mine" — an early warning system that detects problems before they affect everyone.*
3. **What metrics should you monitor during a canary?** *Error rate, latency (p95/p99), CPU/memory usage, and business metrics like conversion rate.*
4. **What tool automates canary deployments in Kubernetes?** *Argo Rollouts — provides automated canary with analysis templates and progressive traffic shifting.*
5. **How does canary differ from Blue-Green?** *Canary gradually shifts traffic by percentage; Blue-Green is all-or-nothing switching between two environments.*
6. **What happens if canary metrics fail?** *Traffic is automatically routed back to 0% canary (all to stable), and the canary deployment is rolled back.*

---

[← Previous: Blue-Green Deployments](02-blue-green-deployments.md) | [Back to README](../README.md) | [Next: A/B Testing Deployments →](04-ab-testing-deployments.md)
