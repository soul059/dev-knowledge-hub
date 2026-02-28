# Chapter 1: What Are Metrics?

[← Previous: SRE Concepts](../01-Observability-Concepts/05-sre-concepts.md) | [Back to README](../README.md) | [Next: Metric Types →](02-metric-types.md)

---

## Overview

Metrics are **numeric measurements** collected at regular time intervals that describe the behavior of a system. They are the most fundamental pillar of observability — compact, fast to query, and ideal for alerting and trend analysis.

---

## 1.1 Definition

> **Metric** = A numeric value measured at a point in time, with associated metadata (name, labels, timestamp).

```
┌─────────────────────────────────────────────────────────────┐
│                    ANATOMY OF A METRIC                       │
│                                                              │
│   metric_name{label1="val1", label2="val2"}  value  @time   │
│   ───────────  ──────────────────────────── ─────  ────    │
│      Name              Labels              Value  Timestamp │
│                                                              │
│   Example:                                                   │
│   http_requests_total{method="GET", status="200"} 15230     │
│                                                 @1706000000 │
│                                                              │
│   Meaning: "As of this timestamp, 15,230 GET requests       │
│            returned HTTP 200"                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 1.2 Time Series Data

Metrics produce **time series** — ordered sequences of (timestamp, value) pairs:

```
http_requests_total{method="GET"}

  Value │
  1500  │                                    ╱
  1200  │                              ╱───╱
   900  │                        ╱───╱╱
   600  │                 ╱────╱╱
   300  │          ╱────╱╱
     0  │────────╱╱
        └──────────────────────────────────────
         t0   t1   t2   t3   t4   t5   t6

  Each point = one scrape (e.g., every 15 seconds)
  
  Time series = unique combination of metric name + label set
  
  http_requests_total{method="GET"}    → Time Series 1
  http_requests_total{method="POST"}   → Time Series 2
  http_requests_total{method="DELETE"} → Time Series 3
```

---

## 1.3 Why Metrics Matter

```
┌─────────────────────────────────────────────────────────┐
│              WHY METRICS ARE ESSENTIAL                    │
├─────────────┬───────────────────────────────────────────┤
│ Compact     │ Just numbers — very storage efficient     │
│ Fast        │ Numeric queries are extremely fast        │
│ Aggregatable│ Sum, avg, percentile across dimensions    │
│ Alertable   │ Threshold and trend-based alerting        │
│ Historical  │ Retain months/years cheaply               │
│ Universal   │ Every system can emit numbers             │
└─────────────┴───────────────────────────────────────────┘
```

---

## 1.4 Metrics vs Other Data Types

```
┌──────────────┬───────────┬───────────┬───────────┐
│              │  Metrics  │   Logs    │  Traces   │
├──────────────┼───────────┼───────────┼───────────┤
│ Data type    │  Number   │   Text    │Span tree  │
│ Size/event   │  ~16 bytes│  ~1 KB    │  ~5 KB    │
│ Query speed  │  Fast     │  Slow     │  Medium   │
│ Alerting     │ ★★★★★    │  ★★☆☆☆   │  ★★☆☆☆   │
│ Context      │  Low      │  High     │  High     │
│ Retention    │  Months   │  Weeks    │  Days     │
│ Cost/event   │  Cheapest │ Expensive │  Medium   │
└──────────────┴───────────┴───────────┴───────────┘
```

---

## 1.5 The Four Golden Signals (Google SRE)

Google's SRE book defines four critical metrics every service should track:

```
┌─────────────────────────────────────────────────────────┐
│            THE FOUR GOLDEN SIGNALS                       │
│                                                          │
│  ┌────────────┐  ┌────────────┐                         │
│  │  LATENCY   │  │  TRAFFIC   │                         │
│  │            │  │            │                         │
│  │ How long   │  │ How much   │                         │
│  │ requests   │  │ demand on  │                         │
│  │ take       │  │ the system │                         │
│  │            │  │            │                         │
│  │ p50, p95,  │  │ req/sec,   │                         │
│  │ p99        │  │ sessions   │                         │
│  └────────────┘  └────────────┘                         │
│                                                          │
│  ┌────────────┐  ┌────────────┐                         │
│  │   ERRORS   │  │ SATURATION │                         │
│  │            │  │            │                         │
│  │ Rate of    │  │ How "full" │                         │
│  │ failed     │  │ the service│                         │
│  │ requests   │  │ is         │                         │
│  │            │  │            │                         │
│  │ 5xx rate,  │  │ CPU %, mem │                         │
│  │ error %    │  │ queue depth│                         │
│  └────────────┘  └────────────┘                         │
└─────────────────────────────────────────────────────────┘
```

---

## 1.6 RED and USE Methods

### RED Method (For Services)

```
R = Rate       — Requests per second
E = Errors     — Failed requests per second  
D = Duration   — Distribution of request latency

Best for: Request-driven services (APIs, web servers)
```

### USE Method (For Resources)

```
U = Utilization — % of resource capacity being used
S = Saturation  — Amount of work queued/waiting
E = Errors      — Count of error events

Best for: System resources (CPU, memory, disk, network)
```

```
┌─────────────────────────────────────────────────────┐
│    RED for Services │  USE for Resources             │
├─────────────────────┼───────────────────────────────┤
│   API Gateway       │      CPU                       │
│   ├─ Rate: 500 rps  │   ├─ Utilization: 75%         │
│   ├─ Errors: 2%     │   ├─ Saturation: 0 (no queue) │
│   └─ Duration:      │   └─ Errors: 0                │
│      p50=10ms       │                                │
│      p99=150ms      │      Memory                    │
│                     │   ├─ Utilization: 60%          │
│   Payment Service   │   ├─ Saturation: 0 swap       │
│   ├─ Rate: 50 rps   │   └─ Errors: 0 OOMs           │
│   ├─ Errors: 0.5%   │                                │
│   └─ Duration:      │      Disk I/O                  │
│      p50=120ms      │   ├─ Utilization: 35%          │
│      p99=800ms      │   ├─ Saturation: 2 in queue   │
│                     │   └─ Errors: 0                │
└─────────────────────┴───────────────────────────────┘
```

---

## 1.7 Metric Collection Approaches

```
┌─────────────────────────────────────────────────────────────────┐
│                  COLLECTION APPROACHES                           │
│                                                                  │
│  PULL Model (Prometheus)              PUSH Model (StatsD)        │
│  ┌──────────┐                        ┌──────────┐               │
│  │Prometheus│──── scrape ──►App /metrics  │  App   │──── push  │
│  │          │                        │         │──────►StatsD   │
│  │          │──── scrape ──►App /metrics  └─────────┘              │
│  └──────────┘                                                    │
│                                                                  │
│  ✅ Pull: Central control           ✅ Push: Short-lived jobs   │
│  ✅ Pull: Easy to detect "down"     ✅ Push: Behind firewalls   │
│  ❌ Pull: Needs network access      ❌ Push: Can overwhelm      │
│  ❌ Pull: Bad for short jobs        ❌ Push: Hard to detect down│
│                                                                  │
│  HYBRID Model (OpenTelemetry)                                    │
│  ┌──────────┐    ┌───────────┐    ┌──────────────┐              │
│  │   App    │───►│OTel Agent │───►│  Backend     │              │
│  │(push SDK)│    │(collector)│    │(pull/push)   │              │
│  └──────────┘    └───────────┘    └──────────────┘              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1.8 Real-World Scenario

🌍 **Scenario: Setting Up Metrics for a New Microservice**

```bash
# Step 1: Instrument your application (Python/Flask example)
from prometheus_client import Counter, Histogram, Gauge, start_http_server

# Define metrics
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency in seconds',
    ['method', 'endpoint'],
    buckets=[0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0]
)

ACTIVE_REQUESTS = Gauge(
    'http_requests_active',
    'Currently active HTTP requests'
)

# Step 2: Use in request handler
@app.route('/api/orders', methods=['POST'])
def create_order():
    ACTIVE_REQUESTS.inc()
    start_time = time.time()
    try:
        result = process_order()
        REQUEST_COUNT.labels('POST', '/api/orders', '200').inc()
        return result, 200
    except Exception as e:
        REQUEST_COUNT.labels('POST', '/api/orders', '500').inc()
        raise
    finally:
        REQUEST_LATENCY.labels('POST', '/api/orders').observe(
            time.time() - start_time
        )
        ACTIVE_REQUESTS.dec()

# Step 3: Expose /metrics endpoint
start_http_server(8000)  # Metrics available at :8000/metrics
```

---

## 1.9 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Metric not appearing | Metric never incremented/set | Ensure code path is executed at least once |
| Values look wrong | Using wrong metric type | Counter for totals, Gauge for current values |
| Too many time series | High-cardinality labels | Remove unbounded labels (user_id, request_id) |
| Gaps in metric graph | Target down or scrape timeout | Check target health, increase scrape timeout |
| Metric value resets | Service restarted (counter reset) | Use `rate()` or `increase()` to handle resets |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Metric** | Numeric measurement with name, labels, timestamp |
| **Time Series** | Unique combination of metric name + label values |
| **Golden Signals** | Latency, Traffic, Errors, Saturation |
| **RED Method** | Rate, Errors, Duration (for services) |
| **USE Method** | Utilization, Saturation, Errors (for resources) |
| **Pull Model** | Server scrapes targets (Prometheus) |
| **Push Model** | App pushes to collector (StatsD, OTLP) |

---

## Quick Revision Questions

1. **What is a metric in the context of monitoring? What components make up a metric data point?**

2. **Explain the difference between the Pull and Push models for metric collection. Give a tool example for each.**

3. **What are the Four Golden Signals? Why does Google SRE recommend tracking these for every service?**

4. **Compare the RED and USE methods. When would you use each one?**

5. **Why are metrics considered the most cost-effective pillar of observability?**

6. **A counter metric suddenly drops to zero. What likely happened, and how should your queries account for this?**

---

[← Previous: SRE Concepts](../01-Observability-Concepts/05-sre-concepts.md) | [Back to README](../README.md) | [Next: Metric Types →](02-metric-types.md)
