# Chapter 5: Aggregations

[← Previous: Labels & Cardinality](04-labels-dimensions-cardinality.md) | [Back to README](../README.md) | [Next: Unit 3 — Prometheus Architecture →](../03-Prometheus/01-prometheus-architecture.md)

---

## Overview

Aggregation is the process of combining multiple time series into fewer, more meaningful values. It's essential for dashboards, alerts, and understanding system behavior at different levels of granularity.

---

## 5.1 Why Aggregate?

```
Without aggregation:
  http_requests_total{instance="pod-1", method="GET", status="200"} = 1500
  http_requests_total{instance="pod-2", method="GET", status="200"} = 1600
  http_requests_total{instance="pod-3", method="GET", status="200"} = 1400
  ... (hundreds more)

With aggregation:
  Total GET 200 requests across all pods = 4500
  Average requests per pod = 1500
```

---

## 5.2 Common Aggregation Functions

```
┌──────────────┬─────────────────────────────┬────────────────────────┐
│ Function     │ Description                 │ Use Case               │
├──────────────┼─────────────────────────────┼────────────────────────┤
│ sum()        │ Total of all values         │ Total requests/sec     │
│ avg()        │ Average of all values       │ Avg CPU across pods    │
│ min()        │ Smallest value              │ Identify coldest node  │
│ max()        │ Largest value               │ Find hottest pod       │
│ count()      │ Number of time series       │ How many pods running  │
│ stddev()     │ Standard deviation          │ Detect uneven load     │
│ topk(k,...)  │ Top K time series by value  │ Top 5 busiest services │
│ bottomk(k,..)│ Bottom K by value           │ Least utilized nodes   │
│ quantile(φ,.)│ φ-quantile over dimensions  │ p90 across instances   │
└──────────────┴─────────────────────────────┴────────────────────────┘
```

### PromQL Examples

```promql
# Total request rate across all pods
sum(rate(http_requests_total[5m]))

# Average CPU per node
avg(node_cpu_usage_percent) by (node)

# Top 5 services by error rate
topk(5, sum(rate(http_requests_total{status=~"5.."}[5m])) by (service))

# Count of running pods per deployment
count(up == 1) by (deployment)

# Max memory usage across cluster
max(container_memory_usage_bytes) by (container)
```

---

## 5.3 Aggregation with `by` and `without`

```promql
# by() — keep only specified labels
sum(rate(http_requests_total[5m])) by (method, status)
# Result: aggregated BY method and status, everything else collapsed

# without() — remove specified labels (keep everything else)
sum(rate(http_requests_total[5m])) without (instance, pod)
# Result: same as above if only extra labels were instance and pod
```

```
Before (4 series):
  rate{instance="pod-1", method="GET", status="200"} = 10
  rate{instance="pod-2", method="GET", status="200"} = 12
  rate{instance="pod-1", method="POST",status="200"} = 5
  rate{instance="pod-2", method="POST",status="200"} = 7

After sum() by (method, status):
  rate{method="GET",  status="200"} = 22   (10+12)
  rate{method="POST", status="200"} = 12   (5+7)
```

---

## 5.4 Over-Time Aggregations

```
┌──────────────────────┬───────────────────────────────┐
│ Function             │ Description                   │
├──────────────────────┼───────────────────────────────┤
│ avg_over_time()      │ Average over time range       │
│ max_over_time()      │ Max over time range           │
│ min_over_time()      │ Min over time range           │
│ sum_over_time()      │ Sum over time range           │
│ count_over_time()    │ Count samples in range        │
│ stddev_over_time()   │ Std dev over time range       │
│ quantile_over_time() │ Quantile over time range      │
│ last_over_time()     │ Most recent value in range    │
└──────────────────────┴───────────────────────────────┘
```

```promql
# Average CPU over last 24 hours
avg_over_time(node_cpu_usage_percent[24h])

# Peak memory in last week
max_over_time(container_memory_usage_bytes[7d])

# 95th percentile latency over last hour
quantile_over_time(0.95, http_request_duration_seconds[1h])
```

---

## 5.5 Real-World Aggregation Patterns

```promql
# 1. Service Error Rate (%)
sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
/
sum(rate(http_requests_total[5m])) by (service)
* 100

# 2. Cluster-wide Request Rate
sum(rate(http_requests_total[5m]))

# 3. p99 Latency per Service
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
)

# 4. Disk Fill Prediction (hours until full)
predict_linear(node_filesystem_avail_bytes[6h], 3600*24) < 0
```

---

## Summary Table

| Aggregation Type | Functions | Use Case |
|-----------------|-----------|----------|
| **Across series** | `sum`, `avg`, `max`, `min`, `count` | Combine pods, instances |
| **Over time** | `avg_over_time`, `max_over_time` | Historical analysis |
| **Selection** | `topk`, `bottomk` | Find outliers |
| **Grouping** | `by()`, `without()` | Control output labels |

---

## Quick Revision Questions

1. **What is the difference between `sum() by (service)` and `sum() without (instance)`?**
2. **Write a PromQL query to find the top 3 services by error rate.**
3. **When would you use `avg_over_time()` vs `avg()`?**
4. **How do you calculate the percentage of 5xx errors for each endpoint?**
5. **What does `count(up == 1) by (job)` tell you?**

---

[← Previous: Labels & Cardinality](04-labels-dimensions-cardinality.md) | [Back to README](../README.md) | [Next: Unit 3 — Prometheus Architecture →](../03-Prometheus/01-prometheus-architecture.md)
