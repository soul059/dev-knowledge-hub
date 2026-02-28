# Chapter 6: Trace Analysis & Optimization

[← Previous: Instrumentation Best Practices](05-instrumentation-best-practices.md) | [Back to README](../README.md) | [Next: Unit 8 - Alerting Strategies →](../08-Alerting/01-alerting-strategies.md)

---

## Overview

Collecting traces is only half the battle. This chapter covers how to analyze traces to find performance bottlenecks, understand system behavior, diagnose failures, and optimize your tracing infrastructure for cost and performance.

---

## 6.1 Trace Analysis Workflow

```
┌──────────────────────────────────────────────────────────┐
│         TRACE ANALYSIS WORKFLOW                          │
│                                                          │
│  1. DETECT ──► Alert fires / user reports issue          │
│       │                                                  │
│       ▼                                                  │
│  2. IDENTIFY ──► Find relevant traces                   │
│       │          • Search by service, operation, time    │
│       │          • Filter by error, duration             │
│       │          • Use exemplars from metrics            │
│       ▼                                                  │
│  3. EXAMINE ──► Inspect trace waterfall                 │
│       │          • Find the longest span                 │
│       │          • Check for sequential calls            │
│       │          • Look for error spans                  │
│       ▼                                                  │
│  4. CORRELATE ──► Cross-reference with metrics/logs     │
│       │           • Jump to logs for error details       │
│       │           • Check metrics for system state       │
│       ▼                                                  │
│  5. FIX ──► Implement performance/reliability fix        │
│       │                                                  │
│       ▼                                                  │
│  6. VERIFY ──► Compare before/after traces              │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 Common Performance Patterns

### Sequential vs Parallel Calls

```
┌──────────────────────────────────────────────────────────┐
│  PROBLEM: Sequential calls (total = sum of all calls)    │
│                                                          │
│  checkout ██████████████████████████████████████ 600ms   │
│  ├─ validate  ████ 100ms                                │
│  ├─ inventory      ████ 100ms                           │
│  ├─ pricing             ████ 100ms                      │
│  ├─ payment                  ██████████ 200ms           │
│  └─ shipping                           ████ 100ms       │
│                                                          │
│  FIX: Parallelize independent calls                     │
│                                                          │
│  checkout █████████████████████ 300ms                   │
│  ├─ validate  ████ 100ms                                │
│  ├─ inventory ████ 100ms       (parallel with above)    │
│  ├─ pricing   ████ 100ms       (parallel with above)    │
│  ├─ payment        ██████████ 200ms (needs validation)  │
│  └─ shipping       ████ 100ms (parallel with payment)   │
│                                                          │
│  Result: 600ms → 300ms (50% improvement)                │
└──────────────────────────────────────────────────────────┘
```

### N+1 Query Problem

```
┌──────────────────────────────────────────────────────────┐
│  PROBLEM: N+1 database queries                           │
│                                                          │
│  GET /orders ████████████████████████████████ 500ms      │
│  ├─ SELECT orders        ██ 20ms                        │
│  ├─ SELECT user id=1     █ 10ms                         │
│  ├─ SELECT user id=2     █ 10ms                         │
│  ├─ SELECT user id=3     █ 10ms                         │
│  ├─ ... (47 more)                                       │
│  └─ SELECT user id=50    █ 10ms                         │
│                                                          │
│  Symptom: Many identical DB spans with different params  │
│                                                          │
│  FIX: Batch query or JOIN                               │
│                                                          │
│  GET /orders ████ 40ms                                  │
│  └─ SELECT orders JOIN users ██ 25ms                    │
│                                                          │
│  Result: 500ms → 40ms (92% improvement)                 │
└──────────────────────────────────────────────────────────┘
```

### Missing Cache / Cold Cache

```
┌──────────────────────────────────────────────────────────┐
│  PATTERN: Slow trace shows cache misses                  │
│                                                          │
│  GET /product/123 ████████████████████ 350ms             │
│  ├─ redis GET product:123 █ 2ms   (MISS)               │
│  ├─ postgres SELECT       ████████████ 200ms            │
│  └─ redis SET product:123 █ 3ms   (populate cache)     │
│                                                          │
│  vs cached request:                                      │
│  GET /product/123 ██ 15ms                               │
│  └─ redis GET product:123 █ 2ms   (HIT)                │
│                                                          │
│  Action: Add span attribute "cache.hit" = true/false     │
│  Monitor cache hit rate via derived metrics              │
└──────────────────────────────────────────────────────────┘
```

---

## 6.3 Error Analysis with Traces

### Finding Root Cause Across Services

```
┌──────────────────────────────────────────────────────────┐
│  TRACE: User sees "500 Internal Server Error"            │
│                                                          │
│  api-gateway    ████████████████████ 200ms  ❌ 500      │
│  ├─ auth-svc    ███ 30ms                   ✓ 200       │
│  └─ order-svc   ████████████████ 170ms     ❌ 500      │
│     ├─ db query  ████ 40ms                 ✓ OK        │
│     └─ payment   ████████████ 120ms        ❌ 500      │
│        └─ stripe █████████ 100ms           ❌ 502      │
│           event: "Connection refused to stripe.com"     │
│                                                          │
│  Root Cause: Stripe API is down                         │
│  The 500 error propagated:                               │
│  stripe → payment → order → api-gateway → user          │
│                                                          │
│  Without tracing: "order service is broken"             │
│  With tracing: "Stripe is unreachable"                  │
└──────────────────────────────────────────────────────────┘
```

---

## 6.4 TraceQL Queries for Analysis

```bash
# === LATENCY ANALYSIS ===

# Find slowest traces for a service
{ resource.service.name = "api-gateway" } | traceDuration > 5s

# Find traces where DB is the bottleneck
{ span.db.system = "postgresql" && duration > 1s }

# === ERROR ANALYSIS ===

# Find all error traces
{ status = error }

# Find HTTP 5xx errors in production
{ span.http.status_code >= 500 && resource.deployment.environment = "production" }

# Find timeout errors
{ status = error && span.error.type =~ ".*Timeout.*" }

# === DEPENDENCY ANALYSIS ===

# Find traces crossing specific services
{ resource.service.name = "frontend" } >> { resource.service.name = "payment-service" }

# Find traces where cache miss leads to DB call
{ .name = "redis.GET" && span.cache.hit = false } ~ { span.db.system = "postgresql" }

# === VOLUME ANALYSIS ===

# Count traces by service
{ resource.service.name = "order-service" } | count() > 0

# Find traces with too many spans (potential span explosion)
{ } | count() > 100
```

---

## 6.5 Service Dependency Mapping

```
┌──────────────────────────────────────────────────────────┐
│  SERVICE DEPENDENCY GRAPH (from traces)                  │
│                                                          │
│                   ┌──────────┐                          │
│                   │ Frontend │                          │
│                   └────┬─────┘                          │
│                        │                                 │
│              ┌─────────┼─────────┐                      │
│              ▼         ▼         ▼                      │
│         ┌────────┐ ┌────────┐ ┌────────┐               │
│         │  API   │ │ Search │ │  Auth  │               │
│         │Gateway │ │Service │ │Service │               │
│         └───┬────┘ └───┬────┘ └───┬────┘               │
│             │          │          │                      │
│        ┌────┼────┐     │     ┌────┘                     │
│        ▼    ▼    ▼     ▼     ▼                          │
│   ┌──────┐┌────┐┌───────┐ ┌──────┐                     │
│   │Order ││Cart││Payment│ │Redis │                     │
│   │ Svc  ││Svc ││  Svc  │ │Cache │                     │
│   └──┬───┘└─┬──┘└───┬───┘ └──────┘                     │
│      │      │       │                                    │
│      ▼      ▼       ▼                                   │
│   ┌──────────────────────┐ ┌──────────┐                │
│   │     PostgreSQL       │ │ Stripe   │                │
│   │     (shared DB)      │ │ (extern) │                │
│   └──────────────────────┘ └──────────┘                │
│                                                          │
│  Edge labels show:                                      │
│  • Request rate (req/s)                                 │
│  • Error rate (%)                                       │
│  • p99 latency (ms)                                    │
│                                                          │
│  Generated automatically from trace data in:            │
│  • Grafana (Service Map panel using Tempo)              │
│  • Jaeger (System Architecture / DAG view)              │
└──────────────────────────────────────────────────────────┘
```

---

## 6.6 Comparing Traces (Before/After)

```
┌──────────────────────────────────────────────────────────┐
│  TRACE COMPARISON (Jaeger / Grafana)                     │
│                                                          │
│  BEFORE optimization:                                    │
│  checkout ██████████████████████████████████ 1200ms      │
│  ├─ validate      ████ 100ms                            │
│  ├─ check-inv     ████████ 200ms                        │
│  ├─ calc-price    ██████ 150ms                          │
│  ├─ charge        ████████████████ 400ms                │
│  ├─ create-order  ██████████ 250ms                      │
│  └─ send-email    ████ 100ms                            │
│                                                          │
│  AFTER optimization:                                     │
│  checkout ██████████████████ 550ms                       │
│  ├─ validate      ██ 50ms        (added caching)        │
│  ├─ check-inv     ██ 50ms        (cached inventory)     │
│  ├─ calc-price    ██ 50ms        (cached pricing)       │
│  ├─ charge        ████████ 200ms (async confirm)        │
│  ├─ create-order  ████ 100ms     (batch insert)         │
│  └─ send-email         ── async (fire & forget)         │
│                                                          │
│  Improvement: 1200ms → 550ms (54% faster)               │
└──────────────────────────────────────────────────────────┘
```

---

## 6.7 Tracing Infrastructure Optimization

### Reducing Costs

| Strategy | Impact | Implementation |
|----------|--------|----------------|
| Head-based sampling | 50-90% reduction | Set sample rate in SDK |
| Tail-based sampling | 70-95% reduction (keeps interesting) | OTel Collector tail_sampling processor |
| Drop health checks | 10-30% reduction | Filter `/health`, `/ready` spans |
| Reduce span attributes | 20-40% size reduction | Remove verbose attributes |
| Shorten retention | Storage savings | Set retention to 3-7 days |
| Use Tempo over Jaeger/ES | 60-80% storage cost | Object storage vs indexed |

### Sizing Guidelines

```
┌──────────────────────────────────────────────────────────┐
│  TRACING INFRASTRUCTURE SIZING                           │
│                                                          │
│  Estimation Formula:                                     │
│  daily_spans = services × requests_per_day × avg_spans   │
│                × sample_rate                             │
│                                                          │
│  Example:                                                │
│  • 20 services                                          │
│  • 1M requests/day per service                          │
│  • 5 spans per request average                          │
│  • 10% sampling                                         │
│  = 20 × 1,000,000 × 5 × 0.10 = 10M spans/day          │
│                                                          │
│  Storage estimate:                                       │
│  • ~1 KB per span (average)                             │
│  • 10M × 1KB = ~10 GB/day                              │
│  • 7-day retention = ~70 GB                             │
│                                                          │
│  Tempo (object storage): ~$2/month for 70GB             │
│  Elasticsearch:          ~$200/month for 70GB indexed   │
└──────────────────────────────────────────────────────────┘
```

---

## 6.8 Real-World Debugging Scenarios

### Scenario 1: Intermittent Timeouts

```
Investigation steps using traces:
1. Search: { resource.service.name="api-gateway" && duration > 10s }
2. Find: 80% of slow traces have payment-service span > 9s
3. Check: payment-service → external payment API
4. Correlate: Logs show "connection pool exhausted"
5. Root cause: Connection pool max=10, concurrent requests=50
6. Fix: Increase pool size, add circuit breaker
```

### Scenario 2: Cascading Failures

```
Investigation steps:
1. Alert: Error rate spike on frontend
2. Traces: { status = error } | count() > 5
3. Find: All errors originate from inventory-service
4. Drill down: inventory-service → database timeout
5. Check metrics: DB CPU at 100%, active connections at max
6. Root cause: Missing index on frequently queried column
7. Fix: Add database index, add query timeout
```

---

## Summary Table

| Concept | Details |
|---------|---------|
| Analysis Workflow | Detect → Identify → Examine → Correlate → Fix → Verify |
| Common Patterns | Sequential calls, N+1 queries, cache misses, error propagation |
| Key Tools | TraceQL, Service Maps, Trace Compare, Exemplars |
| Cost Optimization | Sampling, retention, attribute trimming, Tempo over ES |
| Sizing | daily_spans = services × requests × avg_spans × sample_rate |

---

## Quick Revision Questions

1. **How do you identify an N+1 query problem from a trace waterfall?**
2. **What is the benefit of parallelizing sequential upstream calls, and how do traces reveal this opportunity?**
3. **How do you use TraceQL to find traces where a specific service is the performance bottleneck?**
4. **Describe the steps to trace a user-facing 500 error back to its root cause in a microservices system.**
5. **How would you estimate storage costs for your tracing infrastructure?**

---

[← Previous: Instrumentation Best Practices](05-instrumentation-best-practices.md) | [Back to README](../README.md) | [Next: Unit 8 - Alerting Strategies →](../08-Alerting/01-alerting-strategies.md)
