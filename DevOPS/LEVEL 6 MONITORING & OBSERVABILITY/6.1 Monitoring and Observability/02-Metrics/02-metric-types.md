# Chapter 2: Metric Types — Counter, Gauge, Histogram, Summary

[← Previous: What Are Metrics?](01-what-are-metrics.md) | [Back to README](../README.md) | [Next: Naming Conventions →](03-metric-naming-conventions.md)

---

## Overview

Prometheus defines four core metric types, each designed for specific measurement patterns. Choosing the right type is crucial for accurate data representation and efficient querying.

---

## 2.1 The Four Metric Types at a Glance

```
┌──────────────────────────────────────────────────────────────┐
│                   METRIC TYPES OVERVIEW                       │
├──────────┬──────────┬────────────────┬───────────────────────┤
│ Counter  │  Gauge   │  Histogram     │  Summary              │
├──────────┼──────────┼────────────────┼───────────────────────┤
│ Only     │ Goes     │ Counts values  │ Calculates            │
│ goes     │ up AND   │ in buckets     │ quantiles             │
│ up       │ down     │ (server-side)  │ (client-side)         │
│          │          │                │                       │
│  ╱       │  /\  /   │  ┌──┐         │  p50 = 120ms          │
│ ╱        │ /  \/    │  │  │┌─┐      │  p90 = 250ms          │
│╱         │/         │  │  ││ │┌┐    │  p99 = 800ms          │
│          │          │  └──┘└─┘└┘    │                       │
├──────────┼──────────┼────────────────┼───────────────────────┤
│ requests │ temp,    │ latency dist., │ latency quantiles,    │
│ errors   │ active   │ size dist.     │ request duration      │
│ bytes    │ conns    │                │                       │
└──────────┴──────────┴────────────────┴───────────────────────┘
```

---

## 2.2 Counter

> A **counter** is a cumulative metric that only **increases** (or resets to zero on restart).

### Behavior

```
Value │
 500  │                                        ╱
 400  │                                   ╱──╱
 300  │                             ╱───╱╱
 200  │                       ╱───╱╱
 100  │                 ╱───╱╱
   0  │───────────╱───╱╱
      └──────────────────────────────────────────
       t0   t1   t2   t3   t4   t5   t6

  ⚠️ Counter NEVER decreases (except on restart → reset to 0)
  ⚠️ Raw value is rarely useful — use rate() or increase()
```

### When to Use
- Total number of requests served
- Total number of errors
- Total bytes transferred
- Total tasks completed

### Example

```python
from prometheus_client import Counter

# Define
http_requests_total = Counter(
    'http_requests_total',
    'Total number of HTTP requests',
    ['method', 'status', 'endpoint']
)

# Use
http_requests_total.labels(method='GET', status='200', endpoint='/api/users').inc()
http_requests_total.labels(method='POST', status='500', endpoint='/api/orders').inc()
```

### Querying Counters

```promql
# WRONG: Raw counter value is just a big number
http_requests_total  → 150234 (not useful!)

# RIGHT: Rate of increase per second
rate(http_requests_total[5m])  → 45.2 req/s

# RIGHT: Total increase over period
increase(http_requests_total[1h])  → 162720 requests in last hour

# Error rate percentage
rate(http_requests_total{status=~"5.."}[5m])
/
rate(http_requests_total[5m])
* 100  → 2.3% error rate
```

---

## 2.3 Gauge

> A **gauge** is a metric that can **increase and decrease** — represents a current value.

### Behavior

```
Value │
  90  │      ╱╲
  75  │     ╱  ╲         ╱╲
  60  │    ╱    ╲       ╱  ╲
  45  │   ╱      ╲    ╱╱    ╲╲
  30  │  ╱        ╲  ╱        ╲
  15  │ ╱          ╲╱          ╲
   0  │╱                        ╲
      └──────────────────────────────
       t0  t1  t2  t3  t4  t5  t6

  ✅ Gauge CAN go up and down
  ✅ Current value IS meaningful
```

### When to Use
- Current temperature
- Current CPU usage (%)
- Active connections / in-flight requests
- Queue size
- Memory usage
- Number of running pods

### Example

```python
from prometheus_client import Gauge

# Define
active_connections = Gauge(
    'db_connections_active',
    'Number of active database connections',
    ['database']
)

cpu_usage = Gauge(
    'node_cpu_usage_percent',
    'Current CPU usage percentage'
)

# Use
active_connections.labels(database='orders_db').set(42)
active_connections.labels(database='orders_db').inc()   # 43
active_connections.labels(database='orders_db').dec()   # 42
cpu_usage.set(67.5)
```

### Querying Gauges

```promql
# Current value (directly useful)
db_connections_active{database="orders_db"}  → 42

# Average over time
avg_over_time(node_cpu_usage_percent[1h])  → 55.3%

# Max over time (peak detection)
max_over_time(node_cpu_usage_percent[24h])  → 92.1%

# Predict future value (linear regression)
predict_linear(node_filesystem_free_bytes[6h], 3600*24)
# "At current rate, disk will have X bytes free in 24h"
```

---

## 2.4 Histogram

> A **histogram** samples observations and counts them in configurable **buckets**. Also provides sum and count.

### Behavior

```
Histogram: http_request_duration_seconds

Bucket boundaries: 0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0

Distribution:
  Count │
   500  │  ┌────┐
   400  │  │    │┌────┐
   300  │  │    ││    │
   200  │  │    ││    │┌────┐
   100  │  │    ││    ││    │┌────┐
     0  │  │    ││    ││    ││    │┌──┐┌─┐
        └──┴────┴┴────┴┴────┴┴────┴┴──┴┴─┴──
           0.01  0.05  0.1  0.25  0.5  1.0  (seconds)

  What Prometheus stores:
  http_request_duration_seconds_bucket{le="0.01"}  = 120
  http_request_duration_seconds_bucket{le="0.05"}  = 620
  http_request_duration_seconds_bucket{le="0.1"}   = 1020
  http_request_duration_seconds_bucket{le="0.25"}  = 1220
  http_request_duration_seconds_bucket{le="0.5"}   = 1320
  http_request_duration_seconds_bucket{le="1.0"}   = 1380
  http_request_duration_seconds_bucket{le="+Inf"}  = 1400
  http_request_duration_seconds_sum                = 187.5
  http_request_duration_seconds_count              = 1400
```

### When to Use
- Request latency distributions
- Response size distributions
- Any measurement where you need percentiles/quantiles
- When you need to **aggregate** percentiles across instances

### Example

```python
from prometheus_client import Histogram

# Define with custom buckets
request_duration = Histogram(
    'http_request_duration_seconds',
    'Request duration in seconds',
    ['method', 'endpoint'],
    buckets=[0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0]
)

# Use
@request_duration.labels(method='GET', endpoint='/api/users').time()
def get_users():
    return db.query("SELECT * FROM users")

# Or manually
start = time.time()
process_order()
request_duration.labels(method='POST', endpoint='/api/orders').observe(
    time.time() - start
)
```

### Querying Histograms

```promql
# Calculate p50 (median) latency
histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))

# Calculate p95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Calculate p99 latency
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))

# Average latency
rate(http_request_duration_seconds_sum[5m])
/
rate(http_request_duration_seconds_count[5m])

# Apdex score (requests below threshold / total)
(
  rate(http_request_duration_seconds_bucket{le="0.3"}[5m])
  +
  rate(http_request_duration_seconds_bucket{le="1.0"}[5m])
) / 2
/
rate(http_request_duration_seconds_count[5m])
```

---

## 2.5 Summary

> A **summary** calculates streaming **quantiles** on the client side over a configurable time window.

### Behavior

```
Summary: rpc_duration_seconds

Client calculates:
  rpc_duration_seconds{quantile="0.5"}   = 0.120  (median)
  rpc_duration_seconds{quantile="0.9"}   = 0.250
  rpc_duration_seconds{quantile="0.99"}  = 0.830
  rpc_duration_seconds_sum               = 1250.5
  rpc_duration_seconds_count             = 10500
```

### When to Use
- When you need **exact quantiles** (not approximated)
- When you don't need to aggregate across instances
- When bucket boundaries are hard to define in advance

### Example

```python
from prometheus_client import Summary

# Define
request_duration = Summary(
    'rpc_duration_seconds',
    'RPC duration in seconds',
    ['service', 'method']
)

# Use
with request_duration.labels(service='user', method='GetUser').time():
    user = user_service.get_user(user_id)
```

---

## 2.6 Histogram vs Summary — When to Use Which?

```
┌────────────────────┬──────────────────────┬──────────────────────┐
│ Aspect             │ Histogram            │ Summary              │
├────────────────────┼──────────────────────┼──────────────────────┤
│ Quantile calc.     │ Server-side          │ Client-side          │
│                    │ (at query time)      │ (at scrape time)     │
├────────────────────┼──────────────────────┼──────────────────────┤
│ Aggregation        │ ✅ Can aggregate     │ ❌ Cannot aggregate  │
│ across instances   │ across instances     │ across instances     │
├────────────────────┼──────────────────────┼──────────────────────┤
│ Accuracy           │ Approximate          │ Exact (for a single  │
│                    │ (depends on buckets) │ instance)            │
├────────────────────┼──────────────────────┼──────────────────────┤
│ Configuration      │ Must choose buckets  │ Must choose quantiles│
│                    │ upfront              │ upfront              │
├────────────────────┼──────────────────────┼──────────────────────┤
│ CPU/Memory cost    │ Low (just counters)  │ Higher (streaming    │
│ on client          │                      │ calculation)         │
├────────────────────┼──────────────────────┼──────────────────────┤
│ Time series count  │ #buckets + 2         │ #quantiles + 2      │
├────────────────────┼──────────────────────┼──────────────────────┤
│ RECOMMENDATION     │ ✅ USE THIS          │ ⚠️ Only if needed   │
│                    │ (default choice)     │                      │
└────────────────────┴──────────────────────┴──────────────────────┘
```

💡 **Rule of thumb:** **Use histograms** unless you have a specific reason to need summaries. Histograms can be aggregated across instances; summaries cannot.

---

## 2.7 Common Patterns

### Choosing the Right Type

```
"How many total X have occurred?"     → COUNTER
  (requests, errors, bytes)

"What is the current value of X?"     → GAUGE
  (temperature, connections, queue)

"What is the distribution of X?"      → HISTOGRAM
  (latency, request size)

"What are exact quantiles of X?"      → SUMMARY
  (rarely needed; prefer histogram)
```

### Real-World Metric Examples

```
┌──────────────────────────────────────────────────────────┐
│ Service: E-Commerce Checkout                              │
│                                                           │
│ COUNTERS:                                                 │
│ ├── checkout_orders_total{status="success|failed"}       │
│ ├── payment_transactions_total{provider="stripe|paypal"} │
│ └── email_notifications_sent_total{type="order|ship"}    │
│                                                           │
│ GAUGES:                                                   │
│ ├── checkout_cart_items_current                           │
│ ├── inventory_stock_available{product_id="..."}          │
│ └── payment_gateway_connections_active                    │
│                                                           │
│ HISTOGRAMS:                                               │
│ ├── checkout_duration_seconds                             │
│ ├── payment_processing_duration_seconds                   │
│ └── order_total_amount_dollars                            │
└──────────────────────────────────────────────────────────┘
```

---

## 2.8 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Counter value decreases | Service restarted | Use `rate()` — it handles resets |
| Histogram quantile looks wrong | Bad bucket boundaries | Choose buckets matching expected range |
| Gauge shows stale value | Not being updated | Ensure code path updates the gauge |
| Summary can't aggregate across pods | By design | Switch to histogram |
| Too many time series from histogram | Too many labels × buckets | Reduce label cardinality or bucket count |

---

## Summary Table

| Type | Direction | Raw Value Useful? | Aggregatable? | Best For |
|------|-----------|-------------------|---------------|----------|
| **Counter** | Only up | No (use `rate()`) | Yes | Totals: requests, errors, bytes |
| **Gauge** | Up and down | Yes | Yes | Current: CPU, memory, connections |
| **Histogram** | N/A (buckets) | Via `histogram_quantile()` | Yes | Distributions: latency, sizes |
| **Summary** | N/A (quantiles) | Yes | No | Exact quantiles (single instance) |

---

## Quick Revision Questions

1. **Name the four Prometheus metric types and give one real-world example for each.**

2. **Why should you never use a raw counter value directly? What functions should you apply to it?**

3. **What is the key difference between a histogram and a summary? When would you choose one over the other?**

4. **You need to track the latency distribution of an API endpoint served by 10 pods. Should you use a histogram or summary? Explain why.**

5. **What happens to a counter metric when the application restarts? How does Prometheus handle this?**

6. **Why are histograms recommended over summaries as the default choice for tracking distributions?**

---

[← Previous: What Are Metrics?](01-what-are-metrics.md) | [Back to README](../README.md) | [Next: Naming Conventions →](03-metric-naming-conventions.md)
