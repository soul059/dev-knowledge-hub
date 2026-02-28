# Chapter 4: Labels, Dimensions & Cardinality

[← Previous: Naming Conventions](03-metric-naming-conventions.md) | [Back to README](../README.md) | [Next: Aggregations →](05-aggregations.md)

---

## Overview

Labels (also called dimensions or tags) are key-value pairs attached to metrics that enable powerful filtering and grouping. However, misusing labels can lead to **cardinality explosions** — one of the most common and dangerous pitfalls in metrics systems.

---

## 4.1 What Are Labels?

```
http_requests_total{method="GET", status="200", endpoint="/api/users"}
                    ──────────────────────────────────────────────────
                                    LABELS

Each unique combination of labels = one TIME SERIES

http_requests_total{method="GET",  status="200"} → Series 1
http_requests_total{method="GET",  status="404"} → Series 2
http_requests_total{method="POST", status="200"} → Series 3
http_requests_total{method="POST", status="500"} → Series 4

Total time series = methods(2) × statuses(4) = 8 series
```

---

## 4.2 Good vs Bad Labels

```
┌─────────────────────────────────────────────────────────────┐
│  ✅ GOOD LABELS (bounded, finite values):                   │
│                                                              │
│  method:    GET, POST, PUT, DELETE         (4 values)       │
│  status:    200, 201, 400, 404, 500        (5 values)       │
│  region:    us-east, us-west, eu-west      (3 values)       │
│  service:   api, auth, payment             (10 values)      │
│  env:       prod, staging, dev             (3 values)       │
│                                                              │
│  Total series ≈ manageable                                  │
├─────────────────────────────────────────────────────────────┤
│  ❌ BAD LABELS (unbounded, infinite values):                │
│                                                              │
│  user_id:      u-001, u-002, ... u-1000000 (1M values!)    │
│  request_id:   unique per request          (infinite!)      │
│  email:        unique per user             (infinite!)      │
│  ip_address:   4 billion possibilities     (huge!)          │
│  timestamp:    unique per observation      (infinite!)      │
│                                                              │
│  ⚠️ CARDINALITY EXPLOSION — system crashes!                │
└─────────────────────────────────────────────────────────────┘
```

---

## 4.3 Cardinality

> **Cardinality** = the total number of unique time series in your metrics system.

```
Cardinality = metric_count × label1_values × label2_values × ...

Example:
  http_requests_total{method, status, endpoint, region}
  
  methods  = 4 (GET, POST, PUT, DELETE)
  statuses = 5 (200, 201, 400, 404, 500)
  endpoints= 20 (distinct API paths)
  regions  = 3

  Cardinality = 1 × 4 × 5 × 20 × 3 = 1,200 time series ✅

With user_id added (100K users):
  Cardinality = 1 × 4 × 5 × 20 × 3 × 100,000 
              = 120,000,000 time series ❌ EXPLOSION!
```

### Cardinality Guidelines

```
┌──────────────────┬──────────────────────────────────┐
│ Time Series      │ Assessment                       │
├──────────────────┼──────────────────────────────────┤
│ < 10,000         │ ✅ Healthy                       │
│ 10,000 - 100,000 │ ⚠️  Watch carefully             │
│ 100K - 1M        │ 🟡 Likely problematic           │
│ > 1M             │ 🔴 Critical — will cause issues │
└──────────────────┴──────────────────────────────────┘

Symptoms of cardinality explosion:
• Prometheus memory usage spikes
• Queries become extremely slow
• Grafana dashboards time out
• Prometheus OOM-killed
```

---

## 4.4 Managing Cardinality

### Strategy 1: Remove High-Cardinality Labels

```python
# ❌ BAD — user_id creates millions of series
request_duration.labels(
    method='GET',
    endpoint='/api/orders',
    user_id=current_user.id  # NEVER DO THIS
).observe(duration)

# ✅ GOOD — bounded labels only
request_duration.labels(
    method='GET',
    endpoint='/api/orders',
    user_type='premium'  # Few known values
).observe(duration)
```

### Strategy 2: Bucket High-Cardinality Values

```python
# ❌ BAD — exact response size
response_size.labels(size=str(len(response))).inc()

# ✅ GOOD — bucketed
def size_bucket(size):
    if size < 1024: return "small"
    if size < 10240: return "medium"
    return "large"

response_size.labels(size_class=size_bucket(len(response))).inc()
```

### Strategy 3: Use Logs/Traces for High-Cardinality Data

```
┌──────────────────────────────────────────────────────┐
│  HIGH-CARDINALITY DATA ROUTING:                      │
│                                                       │
│  Need to search by user_id?    → Use LOGS            │
│  Need request-level detail?    → Use TRACES          │
│  Need aggregated trends?       → Use METRICS (no     │
│                                   high-card labels)  │
│                                                       │
│  Metrics = aggregated view                           │
│  Logs/Traces = detailed, per-event view              │
└──────────────────────────────────────────────────────┘
```

---

## 4.5 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Prometheus OOM | Too many time series | `prometheus_tsdb_head_series` — check count |
| Slow queries | High cardinality in queried metric | Reduce labels or use recording rules |
| Uncontrolled growth | Dynamic labels (URLs with IDs) | Normalize endpoint paths (remove IDs) |
| Scrape timeouts | Target exposes too many metrics | Reduce exported metrics or split targets |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Labels** | Key-value pairs that add dimensions to metrics |
| **Cardinality** | Total unique time series (metric × label combos) |
| **Good labels** | Bounded: method, status, region |
| **Bad labels** | Unbounded: user_id, request_id, email |
| **Safe range** | < 100K time series total |
| **Danger zone** | > 1M time series |

---

## Quick Revision Questions

1. **What is metric cardinality? Why is high cardinality dangerous?**
2. **Calculate the cardinality for a metric with labels: method (4 values), status (5 values), region (3 values), endpoint (50 values).**
3. **Why should `user_id` never be used as a metric label?**
4. **What are three strategies for managing cardinality?**
5. **Where should high-cardinality data go if not in metrics?**
6. **How can you check the current cardinality in Prometheus?**

---

[← Previous: Naming Conventions](03-metric-naming-conventions.md) | [Back to README](../README.md) | [Next: Aggregations →](05-aggregations.md)
