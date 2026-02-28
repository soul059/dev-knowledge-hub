# Chapter 8.2: High Availability Patterns

[← Previous: Scalability Patterns](01-scalability-patterns.md) | [Back to README](../README.md) | [Next: Disaster Recovery →](03-disaster-recovery.md)

---

## Overview

High Availability (HA) is the ability of a system to remain operational and accessible despite component failures. SRE teams design for HA by eliminating single points of failure, implementing redundancy at every layer, and ensuring that failures are detected and recovered from automatically. The goal: users never notice when things break.

---

## 1. Availability Targets and What They Mean

```
  ┌──────────────────────────────────────────────────────────┐
  │  AVAILABILITY TARGETS                                    │
  │                                                          │
  │  Nines  │ Uptime %  │ Downtime/Year │ Downtime/Month   │
  │  ───────┼───────────┼───────────────┼──────────────────  │
  │  2 nines│ 99%       │ 3.65 days     │ 7.3 hours        │
  │  3 nines│ 99.9%     │ 8.76 hours    │ 43.8 minutes     │
  │  4 nines│ 99.99%    │ 52.6 minutes  │ 4.38 minutes     │
  │  5 nines│ 99.999%   │ 5.26 minutes  │ 26.3 seconds     │
  │  6 nines│ 99.9999%  │ 31.5 seconds  │ 2.63 seconds     │
  │                                                          │
  │  COMPOUND AVAILABILITY:                                  │
  │  Serial:   A(total) = A1 × A2 × A3                     │
  │  Example:  0.999 × 0.999 × 0.999 = 0.997 (3 services) │
  │                                                          │
  │  Parallel: A(total) = 1 - (1-A1) × (1-A2)              │
  │  Example:  1 - (0.001)² = 0.999999 (2 redundant)       │
  │                                                          │
  │  KEY INSIGHT: Redundancy converts serial to parallel,   │
  │  dramatically improving overall availability.            │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Redundancy Patterns

```
  ┌──────────────────────────────────────────────────────────┐
  │  REDUNDANCY TYPES                                        │
  │                                                          │
  │  ACTIVE-ACTIVE:                                          │
  │  ┌──────┐  ┌──────┐                                     │
  │  │ Node │  │ Node │  Both serve traffic simultaneously  │
  │  │  A   │  │  B   │  If A fails, B handles 100%        │
  │  │ 50%  │  │ 50%  │  No failover delay                 │
  │  └──────┘  └──────┘  Must handle 2x capacity per node  │
  │                                                          │
  │  ACTIVE-PASSIVE:                                         │
  │  ┌──────┐  ┌──────┐                                     │
  │  │ Node │  │ Node │  A serves all traffic              │
  │  │  A   │  │  B   │  B is standby (warm or hot)        │
  │  │ 100% │  │  0%  │  Failover time: seconds to minutes │
  │  └──────┘  └──────┘  Simpler but wastes standby cost   │
  │                                                          │
  │  N+1 REDUNDANCY:                                         │
  │  ┌───┐ ┌───┐ ┌───┐ ┌───┐                               │
  │  │ 1 │ │ 2 │ │ 3 │ │+1 │  Need 3 nodes for capacity  │
  │  └───┘ └───┘ └───┘ └───┘  Keep 4 (one spare)          │
  │  Any one node can fail without impact                    │
  │                                                          │
  │  N+2 REDUNDANCY:                                         │
  │  For critical systems: survive 2 simultaneous failures  │
  └──────────────────────────────────────────────────────────┘
```

---

## 3. Failure Detection

```
  ┌──────────────────────────────────────────────────────────┐
  │  HEALTH CHECK HIERARCHY                                  │
  │                                                          │
  │  Level 1: LIVENESS                                       │
  │  └─ "Is the process running?"                           │
  │     TCP port check, process exists                       │
  │     Failure → Restart container                         │
  │                                                          │
  │  Level 2: READINESS                                      │
  │  └─ "Can this instance serve traffic?"                  │
  │     HTTP 200 from /ready endpoint                       │
  │     Failure → Remove from load balancer                 │
  │                                                          │
  │  Level 3: DEEP HEALTH                                    │
  │  └─ "Are all dependencies healthy?"                     │
  │     DB connection, cache reachable, disk space OK       │
  │     Failure → Indicate degraded state                   │
  │                                                          │
  │  Level 4: FUNCTIONAL                                     │
  │  └─ "Can we complete a real transaction?"               │
  │     Synthetic monitoring, canary transactions           │
  │     Failure → Page on-call, trigger failover            │
  └──────────────────────────────────────────────────────────┘
```

```yaml
# Comprehensive Kubernetes health check configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  replicas: 5
  template:
    spec:
      containers:
        - name: payment
          image: payment-service:v3.2.1
          
          # Liveness: Is the process healthy?
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 10
            failureThreshold: 3
            timeoutSeconds: 5
          
          # Readiness: Can this pod serve traffic?
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 2
            timeoutSeconds: 3
          
          # Startup: Is the app still starting?
          startupProbe:
            httpGet:
              path: /healthz
              port: 8080
            failureThreshold: 30     # 30 × 10s = 5 min startup
            periodSeconds: 10
```

---

## 4. Circuit Breaker Pattern

```
  ┌──────────────────────────────────────────────────────────┐
  │  CIRCUIT BREAKER STATE MACHINE                           │
  │                                                          │
  │  ┌──────────┐                                            │
  │  │  CLOSED  │ ← Normal operation                        │
  │  │  (allow) │   Track failure count                     │
  │  └────┬─────┘                                            │
  │       │ failures > threshold (e.g., 5 in 30s)           │
  │       ▼                                                  │
  │  ┌──────────┐                                            │
  │  │   OPEN   │ ← Fail fast, don't call downstream       │
  │  │ (reject) │   Return fallback or error immediately    │
  │  └────┬─────┘                                            │
  │       │ timeout expires (e.g., 30 seconds)               │
  │       ▼                                                  │
  │  ┌──────────┐                                            │
  │  │HALF-OPEN │ ← Allow 1 test request through           │
  │  │ (test)   │                                            │
  │  └────┬─────┘                                            │
  │       │                                                  │
  │  ┌────┴────┐                                             │
  │  ▼         ▼                                             │
  │ Success   Failure                                        │
  │ → CLOSED  → OPEN                                        │
  │                                                          │
  │  BENEFITS:                                               │
  │  ├─ Prevents cascade failures                           │
  │  ├─ Gives failing service time to recover               │
  │  ├─ Provides fast failure (no timeout waiting)          │
  │  └─ Enables graceful degradation                        │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Bulkhead Pattern

```
  ┌──────────────────────────────────────────────────────────┐
  │  BULKHEAD ISOLATION (inspired by ship compartments)      │
  │                                                          │
  │  WITHOUT BULKHEADS:                                      │
  │  ┌──────────────────────────────────────┐                │
  │  │  Shared Thread Pool (100 threads)    │                │
  │  │  ┌─────┐ ┌─────┐ ┌─────┐            │                │
  │  │  │ API │ │ Pay │ │ Srch│            │                │
  │  │  │  A  │ │  B  │ │  C  │            │                │
  │  │  └─────┘ └─────┘ └─────┘            │                │
  │  └──────────────────────────────────────┘                │
  │  If Service C is slow, it consumes all 100 threads      │
  │  → Services A and B starved → TOTAL OUTAGE              │
  │                                                          │
  │  WITH BULKHEADS:                                         │
  │  ┌────────────┐ ┌────────────┐ ┌────────────┐           │
  │  │ Pool A     │ │ Pool B     │ │ Pool C     │           │
  │  │ 40 threads │ │ 40 threads │ │ 20 threads │           │
  │  │ ┌─────┐    │ │ ┌─────┐   │ │ ┌─────┐   │           │
  │  │ │ API │    │ │ │ Pay │   │ │ │ Srch│   │           │
  │  │ │  A  │    │ │ │  B  │   │ │ │  C  │   │           │
  │  │ └─────┘    │ │ └─────┘   │ │ └─────┘   │           │
  │  └────────────┘ └────────────┘ └────────────┘           │
  │  If Service C is slow, only Pool C is affected          │
  │  → Services A and B continue operating normally         │
  │                                                          │
  │  BULKHEAD TYPES:                                         │
  │  ├─ Thread pool isolation (per-dependency pools)        │
  │  ├─ Connection pool isolation (per-DB connection limit) │
  │  ├─ Process isolation (separate containers)             │
  │  └─ Infrastructure isolation (separate clusters/AZs)    │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Graceful Degradation

```
  ┌──────────────────────────────────────────────────────────┐
  │  DEGRADATION LEVELS                                      │
  │                                                          │
  │  Level 0: FULL SERVICE                                   │
  │  ├─ All features available                              │
  │  ├─ Real-time recommendations, live chat, analytics     │
  │  └─ Normal operation                                    │
  │                                                          │
  │  Level 1: REDUCED FEATURES                               │
  │  ├─ Disable non-critical features                       │
  │  ├─ Turn off: recommendations, live chat, analytics     │
  │  ├─ Keep: browse, search, checkout                      │
  │  └─ Trigger: >70% CPU or dependency degraded           │
  │                                                          │
  │  Level 2: STATIC CONTENT                                 │
  │  ├─ Serve cached/static versions of pages               │
  │  ├─ Product catalog from CDN cache                      │
  │  ├─ Search with limited/stale results                   │
  │  └─ Trigger: database or core service failure           │
  │                                                          │
  │  Level 3: MAINTENANCE MODE                               │
  │  ├─ Static "We'll be back" page                        │
  │  ├─ Queue incoming requests for later processing        │
  │  └─ Trigger: multi-system catastrophic failure          │
  │                                                          │
  │  RULE: Always degrade gracefully rather than crash.      │
  │  Show users something useful, even if limited.          │
  └──────────────────────────────────────────────────────────┘
```

```yaml
# Degradation policy configuration
degradation_policy:
  service: "e-commerce-platform"
  
  levels:
    - level: 1
      name: "reduced_features"
      triggers:
        - metric: "system_cpu_percent"
          threshold: 70
        - metric: "recommendation_service_available"
          threshold: 0  # service down
      actions:
        - disable_feature: "real_time_recommendations"
          fallback: "popular_items_cache"
        - disable_feature: "live_chat"
          fallback: "email_support_link"
        - disable_feature: "user_activity_tracking"
          fallback: "queue_for_later"
          
    - level: 2
      name: "static_content"
      triggers:
        - metric: "database_available"
          threshold: 0
        - metric: "error_rate_5xx"
          threshold: 0.10  # 10% errors
      actions:
        - switch_to: "cdn_cached_catalog"
        - disable_feature: "search"
          fallback: "category_browse_only"
        - disable_feature: "user_accounts"
          fallback: "guest_checkout"
          
    - level: 3
      name: "maintenance_mode"
      triggers:
        - metric: "healthy_pods_percent"
          threshold: 0.10  # <10% pods healthy
      actions:
        - serve: "static/maintenance.html"
        - queue_requests: true
        - notify: "exec-escalation"
```

---

## 7. Multi-AZ and Multi-Region Architecture

```
  ┌──────────────────────────────────────────────────────────┐
  │  MULTI-REGION ACTIVE-ACTIVE ARCHITECTURE                 │
  │                                                          │
  │  ┌─────────────────────┐     ┌─────────────────────┐    │
  │  │  US-EAST REGION     │     │  EU-WEST REGION     │    │
  │  │                     │     │                     │    │
  │  │  ┌──────┐ ┌──────┐  │     │  ┌──────┐ ┌──────┐ │    │
  │  │  │ AZ-1 │ │ AZ-2 │  │     │  │ AZ-1 │ │ AZ-2 │ │    │
  │  │  │ App  │ │ App  │  │     │  │ App  │ │ App  │ │    │
  │  │  │ DB-P │ │ DB-R │  │     │  │ DB-R │ │ DB-R │ │    │
  │  │  └──────┘ └──────┘  │     │  └──────┘ └──────┘ │    │
  │  │         │            │     │        │           │    │
  │  │    ┌────▼────┐       │     │   ┌────▼────┐      │    │
  │  │    │Regional │       │     │   │Regional │      │    │
  │  │    │  LB     │       │     │   │  LB     │      │    │
  │  │    └─────────┘       │     │   └─────────┘      │    │
  │  └─────────┬───────────┘     └────────┬────────────┘    │
  │            │                          │                  │
  │       ┌────▼──────────────────────────▼────┐             │
  │       │         Global DNS (GeoDNS)        │             │
  │       │   Route users to nearest region    │             │
  │       └────────────────────────────────────┘             │
  │                                                          │
  │  DB-P = Primary (writes)   DB-R = Replica (reads)       │
  │  Cross-region replication: async (10-50ms lag)          │
  │                                                          │
  │  TRADE-OFFS:                                             │
  │  ├─ Complexity: Very high (data consistency, routing)   │
  │  ├─ Cost: 2-3x single region                           │
  │  ├─ Benefit: Survive entire region failure              │
  │  └─ Latency: Users routed to nearest region            │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] Achieving 99.99% Availability for a Banking API

```
  CHALLENGE: Banking API serving 50M users requires 99.99%
  availability (< 52.6 min downtime/year)
  
  PREVIOUS STATE: 99.9% (8.76 hours downtime)
  
  Root causes of downtime:
  ├─ Single AZ deployment: AZ failure = 100% outage
  ├─ No circuit breakers: one slow service cascades
  ├─ Database failover: 3-5 min manual switchover
  └─ Deploy-related: ~2 hours/year from bad deploys
  
  ┌──────────────────────────────────────────────────────────┐
  │  IMPROVEMENTS IMPLEMENTED                                │
  │                                                          │
  │  1. MULTI-AZ DEPLOYMENT (3 AZs)                         │
  │     ├─ Application replicas spread across 3 AZs         │
  │     ├─ Anti-affinity rules prevent co-location          │
  │     ├─ Each AZ can handle 50% (N+1 at AZ level)       │
  │     └─ Impact: AZ failure → zero user impact           │
  │                                                          │
  │  2. CIRCUIT BREAKERS + BULKHEADS                        │
  │     ├─ Circuit breaker on every external call           │
  │     ├─ Bulkhead isolation: separate thread pools        │
  │     ├─ Graceful degradation: static balance if DB slow  │
  │     └─ Impact: cascade failures eliminated              │
  │                                                          │
  │  3. AUTOMATED DATABASE FAILOVER                          │
  │     ├─ RDS Multi-AZ with automatic failover             │
  │     ├─ Failover time: 30-60 seconds (was 3-5 min)      │
  │     ├─ Connection pool with automatic reconnect         │
  │     └─ Impact: DB failures nearly invisible             │
  │                                                          │
  │  4. ZERO-DOWNTIME DEPLOYMENTS                           │
  │     ├─ Canary deployments with automated rollback       │
  │     ├─ Feature flags for all new functionality          │
  │     ├─ Blue-green for infrastructure changes            │
  │     └─ Impact: deploy-related downtime → 0              │
  │                                                          │
  │  5. CHAOS ENGINEERING                                    │
  │     ├─ Monthly AZ failure simulations                   │
  │     ├─ Weekly random pod kills                          │
  │     ├─ Quarterly database failover drills               │
  │     └─ Impact: discovered 12 failure modes in year 1    │
  │                                                          │
  │  RESULTS AFTER 12 MONTHS:                                │
  │  ├─ Availability: 99.997% (achieved better than target) │
  │  ├─ Total downtime: 15.8 minutes (was 8.76 hours)      │
  │  ├─ MTTR: 45 seconds average (was 12 minutes)          │
  │  └─ Zero cascade failures                               │
  └──────────────────────────────────────────────────────────┘
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Split-brain during failover" (both nodes think they're primary) | Use leader election with consensus (etcd, ZooKeeper). Implement fencing (STONITH). Use cloud-native managed services with built-in leader election. |
| "Failover works but recovery takes too long" | Pre-warm standby instances. Test failover regularly. Automate the entire failover+recovery process. |
| "Health checks passing but service is degraded" | Implement deep health checks that test actual functionality. Use synthetic transactions. Check response latency, not just HTTP 200. |
| "Cascade failure despite having redundancy" | Add circuit breakers between all service calls. Implement bulkhead isolation. Set aggressive timeouts. Test with chaos engineering. |
| "Multi-region adds too much complexity" | Start with multi-AZ first (simpler). Use managed services with built-in HA (RDS Multi-AZ, EKS). Only add multi-region when required by SLO. |

---

## Summary Table

| Pattern | Purpose | Key Trade-off |
|---------|---------|---------------|
| **Active-Active** | Both nodes serve traffic | Must handle 2x capacity per node |
| **Active-Passive** | Standby for failover | Wasted standby resources |
| **Circuit Breaker** | Prevent cascade failures | May reject valid requests temporarily |
| **Bulkhead** | Isolate failure domains | Reduced resource utilization |
| **Graceful Degradation** | Partial service > no service | Complexity of degradation levels |
| **Multi-AZ** | Survive AZ failure | 2-3x cost, network complexity |
| **Multi-Region** | Survive region failure | Data consistency challenges |

---

## Quick Revision Questions

1. **What is the difference between active-active and active-passive redundancy?**
   > Active-active: both nodes serve traffic simultaneously, providing instant failover with no delay. Active-passive: one node serves traffic while the other is on standby, requiring a failover process (seconds to minutes).

2. **How does the circuit breaker pattern prevent cascade failures?**
   > When a downstream service starts failing, the circuit breaker "opens" and stops sending requests to it, returning a fallback response immediately. This prevents threads from being blocked waiting on a failing service, protecting the calling service from failure.

3. **What is the bulkhead pattern?**
   > Isolating resources (thread pools, connection pools, processes) per dependency so that a failure or slowdown in one dependency cannot consume resources needed by others. Named after ship compartments that contain flooding.

4. **How do you calculate compound availability for services in series vs parallel?**
   > Serial: $A_{total} = A_1 \times A_2 \times A_3$ (availability decreases). Parallel: $A_{total} = 1 - (1 - A_1)(1 - A_2)$ (availability increases). Redundancy converts serial dependencies to parallel.

5. **What are the four levels of health checks?**
   > Liveness (process alive?), Readiness (can serve traffic?), Deep Health (dependencies OK?), Functional (can complete real transactions?). Each level catches increasingly subtle failure modes.

---

[← Previous: Scalability Patterns](01-scalability-patterns.md) | [Back to README](../README.md) | [Next: Disaster Recovery →](03-disaster-recovery.md)
