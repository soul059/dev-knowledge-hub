# Chapter 3: Three Pillars of Observability

[← Previous: Monitoring vs Observability](02-monitoring-vs-observability.md) | [Back to README](../README.md) | [Next: Observability Maturity Model →](04-observability-maturity-model.md)

---

## Overview

The **three pillars of observability** — Metrics, Logs, and Traces — are the fundamental data types that make a system observable. Each pillar provides a different lens into system behavior, and together they provide complete visibility.

---

## 3.1 The Three Pillars Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                             │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐       │
│  │ API GW  │──►│Auth Svc │──►│Order Svc│──►│Pay Svc  │       │
│  └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘       │
│       │              │              │              │             │
│  ┌────┴──────────────┴──────────────┴──────────────┴────┐       │
│  │              INSTRUMENTATION LAYER                    │       │
│  ├──────────┬──────────────┬────────────────────────────┤       │
│  │          │              │                            │       │
│  │  METRICS │    LOGS      │    TRACES                  │       │
│  │          │              │                            │       │
│  │  req/sec │  "Order      │  Trace: abc-123            │       │
│  │  latency │   created    │  ├─ API GW    (5ms)        │       │
│  │  errors  │   for user   │  ├─ Auth Svc  (12ms)       │       │
│  │  CPU %   │   u-456"     │  ├─ Order Svc (45ms)       │       │
│  │          │              │  └─ Pay Svc   (120ms)      │       │
│  └──────────┴──────────────┴────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────┘

         │              │              │
         ▼              ▼              ▼
    ┌─────────┐   ┌──────────┐   ┌──────────┐
    │Promethe-│   │Elasticse-│   │  Jaeger  │
    │us/Mimir │   │arch/Loki │   │  /Tempo  │
    └─────────┘   └──────────┘   └──────────┘
```

---

## 3.2 Pillar 1: Metrics

### What Are Metrics?

Metrics are **numeric measurements** collected at regular intervals. They represent the state or behavior of a system as numbers over time.

```
┌──────────────────────────────────────────────────┐
│                   METRICS                         │
│                                                   │
│   Time Series: metric_name{labels} = value        │
│                                                   │
│   Example:                                        │
│   http_requests_total{method="GET", status="200"} │
│                                                   │
│   Value over time:                                │
│                                                   │
│   requests │      ╱──                             │
│            │    ╱╱                                 │
│            │  ╱╱                                   │
│            │╱╱                                     │
│            └──────────────────                    │
│             t1  t2  t3  t4                        │
└──────────────────────────────────────────────────┘
```

### Characteristics

| Property | Description |
|----------|-------------|
| **Format** | Numeric time-series |
| **Storage** | Compact (just numbers + timestamps) |
| **Aggregation** | Excellent (sum, avg, percentile) |
| **Cardinality** | Low to medium |
| **Best for** | Trends, alerts, capacity planning |
| **Tools** | Prometheus, InfluxDB, DataDog |

### Use Cases
- CPU, memory, disk utilization
- Request rate, error rate, latency (RED)
- Queue depth, connection pool usage
- Business metrics (orders/min, revenue/hour)

---

## 3.3 Pillar 2: Logs

### What Are Logs?

Logs are **immutable, timestamped records** of discrete events that happened in a system. They can be structured (JSON) or unstructured (plain text).

```
┌──────────────────────────────────────────────────────────────┐
│                        LOGS                                   │
│                                                               │
│  Unstructured:                                                │
│  2025-01-15 14:23:01 ERROR [payment-svc] Connection timeout  │
│  to database after 30s. Retrying... attempt 3/5              │
│                                                               │
│  Structured (JSON):                                           │
│  {                                                            │
│    "timestamp": "2025-01-15T14:23:01Z",                      │
│    "level": "ERROR",                                          │
│    "service": "payment-svc",                                  │
│    "message": "Connection timeout to database",               │
│    "duration_ms": 30000,                                      │
│    "retry_attempt": 3,                                        │
│    "max_retries": 5,                                          │
│    "trace_id": "abc-123-def-456"                             │
│  }                                                            │
└──────────────────────────────────────────────────────────────┘
```

### Characteristics

| Property | Description |
|----------|-------------|
| **Format** | Text records (structured or unstructured) |
| **Storage** | Large (verbose, many events) |
| **Aggregation** | Limited (requires parsing first) |
| **Cardinality** | Very high (every event is unique) |
| **Best for** | Debugging, audit trails, understanding context |
| **Tools** | ELK Stack, Loki, Splunk, CloudWatch |

### Use Cases
- Error messages and stack traces
- Audit and compliance trails
- Request/response payloads for debugging
- Application lifecycle events (startup, shutdown, deploys)

---

## 3.4 Pillar 3: Traces

### What Are Traces?

Traces represent the **end-to-end journey of a request** as it flows through a distributed system. Each trace is made up of **spans** — individual units of work.

```
┌──────────────────────────────────────────────────────────────┐
│                       TRACES                                  │
│                                                               │
│  Trace ID: abc-123-def-456                                   │
│                                                               │
│  ┌─ API Gateway ────────────────────────────────────┐ 182ms  │
│  │                                                   │        │
│  │  ┌─ Auth Service ──────────┐ 15ms                │        │
│  │  │  Validate JWT token     │                      │        │
│  │  └─────────────────────────┘                      │        │
│  │                                                   │        │
│  │  ┌─ Order Service ──────────────────────┐ 89ms   │        │
│  │  │                                       │        │        │
│  │  │  ┌─ DB Query ──────┐ 34ms            │        │        │
│  │  │  │ SELECT * FROM   │                  │        │        │
│  │  │  │ orders WHERE... │                  │        │        │
│  │  │  └─────────────────┘                  │        │        │
│  │  │                                       │        │        │
│  │  │  ┌─ Cache Lookup ──┐ 2ms             │        │        │
│  │  │  │ Redis GET       │                  │        │        │
│  │  │  └─────────────────┘                  │        │        │
│  │  └───────────────────────────────────────┘        │        │
│  │                                                   │        │
│  │  ┌─ Payment Service ───────────┐ 76ms            │        │
│  │  │  ┌─ Stripe API Call ──┐ 68ms│                  │        │
│  │  │  │ POST /charges      │     │                  │        │
│  │  │  └────────────────────┘     │                  │        │
│  │  └─────────────────────────────┘                  │        │
│  └───────────────────────────────────────────────────┘        │
└──────────────────────────────────────────────────────────────┘
```

### Characteristics

| Property | Description |
|----------|-------------|
| **Format** | Tree of spans with timing data |
| **Storage** | Medium to large (depends on sampling) |
| **Aggregation** | Moderate (can derive metrics from traces) |
| **Cardinality** | Very high (per-request granularity) |
| **Best for** | Understanding request flow, finding bottlenecks |
| **Tools** | Jaeger, Zipkin, Tempo, X-Ray |

### Use Cases
- Finding slow services in a request chain
- Understanding dependencies between services
- Identifying cascading failures
- Performance optimization

---

## 3.5 How the Pillars Complement Each Other

```
    SCENARIO: "Why is checkout slow?"

    Step 1: METRICS tell you WHAT is happening
    ┌──────────────────────────────────┐
    │ checkout_latency_p99 = 3.2s     │ ◄── Usually 200ms!
    │ payment_svc_errors = 12/min     │ ◄── Spike!
    └──────────────┬───────────────────┘
                   │
    Step 2: TRACES tell you WHERE the problem is
    ┌──────────────▼───────────────────┐
    │ Trace shows:                     │
    │ api-gw(5ms) → auth(10ms)        │
    │   → order(50ms) → payment(2.8s) │ ◄── Bottleneck!
    │     → stripe-api(2.7s)          │ ◄── Root cause!
    └──────────────┬───────────────────┘
                   │
    Step 3: LOGS tell you WHY
    ┌──────────────▼───────────────────┐
    │ payment-svc log:                 │
    │ "Stripe API rate limit exceeded. │
    │  Backing off for 2500ms.         │
    │  Current hour usage: 9850/10000" │ ◄── Aha!
    └──────────────────────────────────┘
```

---

## 3.6 Correlation Across Pillars

The real power comes from **linking** the three pillars together:

```
┌──────────────────────────────────────────────────────────────┐
│                 CORRELATION VIA TRACE ID                      │
│                                                               │
│   Metric Alert                                                │
│   ┌─────────────────────┐                                    │
│   │ error_rate > 5%     │                                    │
│   │ service=payment-svc │──── trace_id: abc-123              │
│   └─────────────────────┘         │                          │
│                                    │                          │
│   Trace                           │                          │
│   ┌─────────────────────┐         │                          │
│   │ trace_id: abc-123   │◄────────┘                          │
│   │ span: payment-svc   │                                    │
│   │ duration: 2800ms    │──── trace_id: abc-123              │
│   └─────────────────────┘         │                          │
│                                    │                          │
│   Log                              │                          │
│   ┌─────────────────────┐         │                          │
│   │ trace_id: abc-123   │◄────────┘                          │
│   │ "Rate limit hit"    │                                    │
│   │ usage: 9850/10000   │                                    │
│   └─────────────────────┘                                    │
│                                                               │
│   Result: Complete understanding without guesswork            │
└──────────────────────────────────────────────────────────────┘
```

📌 **Best Practice:** Always include `trace_id` in your logs and metric labels to enable cross-pillar correlation.

---

## 3.7 Comparison Table

```
┌────────────────┬───────────────┬───────────────┬───────────────┐
│   Dimension    │   METRICS     │    LOGS       │   TRACES      │
├────────────────┼───────────────┼───────────────┼───────────────┤
│ Data type      │ Numbers       │ Text/JSON     │ Span trees    │
│ Granularity    │ Aggregated    │ Per-event     │ Per-request   │
│ Volume         │ Low           │ Very High     │ High          │
│ Cost           │ Low           │ High          │ Medium-High   │
│ Query speed    │ Fast          │ Slow-Medium   │ Medium        │
│ Answers        │ What/When     │ Why/Context   │ Where/Flow    │
│ Alerting       │ Excellent     │ Possible      │ Limited       │
│ Ad-hoc queries │ Good          │ Excellent     │ Good          │
│ Retention      │ Months-Years  │ Days-Weeks    │ Days-Weeks    │
│ Sampling       │ Always 100%  │ Can filter    │ Often sampled │
└────────────────┴───────────────┴───────────────┴───────────────┘
```

---

## 3.8 Beyond the Three Pillars

Modern observability is expanding beyond the classic three pillars:

```
┌─────────────────────────────────────────────────┐
│          EXTENDED OBSERVABILITY                  │
│                                                  │
│  Traditional Three Pillars:                     │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│  │ Metrics │ │  Logs   │ │ Traces  │          │
│  └─────────┘ └─────────┘ └─────────┘          │
│                                                  │
│  Emerging Pillars:                              │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│  │Profiles │ │ Events  │ │Exceptions│          │
│  │(Pyrosc.)│ │(custom) │ │(Sentry) │          │
│  └─────────┘ └─────────┘ └─────────┘          │
│                                                  │
│  Profiles: CPU/Memory flamegraphs               │
│  Events:   Deploys, config changes, incidents   │
│  Exceptions: Grouped error tracking             │
└─────────────────────────────────────────────────┘
```

---

## 3.9 Real-World Scenario

🌍 **Scenario: Debugging a Microservices Timeout**

```
System: Online food delivery platform
Problem: Customers report "order stuck at processing"

Using All Three Pillars:

METRICS (Grafana dashboard):
├── order_processing_duration_seconds p99 = 45s (normally 2s)
├── database_connections_active = 95/100 (near limit!)
└── restaurant_svc_http_requests_total{status="504"} spiking

TRACES (Jaeger):
├── Trace shows: order-svc → restaurant-svc → menu-api
├── menu-api span takes 40s (timeout!)
├── menu-api makes DB call that never returns
└── DB span shows: "waiting for connection from pool"

LOGS (Elasticsearch):
├── menu-api: "Connection pool exhausted. Waiting..."
├── menu-api: "Slow query: SELECT * FROM menu_items JOIN..."
├── Correlated: A schema migration ran 30 min ago
└── Migration added 10M rows without updating indices

ROOT CAUSE: Missing database index after schema migration
FIX: Add index on menu_items table, increase connection pool
```

---

## 3.10 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Metrics show spike but can't find root cause | Missing trace/log correlation | Add trace_id to all log entries |
| Logs are too noisy to search | Unstructured logging, too many DEBUG logs | Use structured logging, adjust log levels |
| Traces show gaps (missing spans) | Incomplete instrumentation | Instrument all service-to-service calls |
| Can't correlate metric to specific request | Metrics are too aggregated | Add higher-cardinality labels or use exemplars |
| High storage costs | Collecting too much data | Apply sampling (traces), reduce log verbosity, use retention policies |

---

## Summary Table

| Pillar | Format | Purpose | Key Question | Best Tool |
|--------|--------|---------|-------------|-----------|
| **Metrics** | Numeric time-series | Detect anomalies, alert, track trends | "WHAT is happening?" | Prometheus, DataDog |
| **Logs** | Text/JSON records | Understand context, debug, audit | "WHY did it happen?" | ELK, Loki, Splunk |
| **Traces** | Span trees | Map request flow, find bottlenecks | "WHERE is the problem?" | Jaeger, Zipkin, Tempo |
| **Combined** | All correlated via trace ID | Full system understanding | "What, why, and where?" | Grafana, DataDog |

---

## Quick Revision Questions

1. **Name the three pillars of observability and describe the primary purpose of each.**

2. **A metric alert fires showing elevated error rates. Explain step-by-step how you would use all three pillars to find the root cause.**

3. **Why are metrics typically the best pillar for alerting? Why aren't logs or traces preferred?**

4. **What is a trace ID and how does it enable correlation across the three pillars?**

5. **Compare the storage costs and retention policies of metrics vs. logs vs. traces. Which is most expensive and why?**

6. **What are the "emerging pillars" beyond the traditional three? Give an example use case for each.**

---

[← Previous: Monitoring vs Observability](02-monitoring-vs-observability.md) | [Back to README](../README.md) | [Next: Observability Maturity Model →](04-observability-maturity-model.md)
