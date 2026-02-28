# Chapter 3: PromQL Basics

[← Previous: Installation](02-installation-configuration.md) | [Back to README](../README.md) | [Next: PromQL Advanced →](04-promql-advanced.md)

---

## Overview

PromQL (Prometheus Query Language) is a functional query language for selecting and aggregating time series data. Understanding PromQL is essential for building dashboards, alerts, and recording rules.

---

## 3.1 Data Types

```
┌──────────────────────────────────────────────────────────┐
│                 PromQL DATA TYPES                        │
├──────────────┬───────────────────────────────────────────┤
│ Instant      │ A set of time series with one value each │
│ Vector       │ at a single point in time                │
│              │ http_requests_total{method="GET"} = 1234 │
├──────────────┼───────────────────────────────────────────┤
│ Range        │ A set of time series with a range of     │
│ Vector       │ values over time                         │
│              │ http_requests_total{method="GET"}[5m]    │
├──────────────┼───────────────────────────────────────────┤
│ Scalar       │ A single numeric floating point value    │
│              │ 42.5                                     │
├──────────────┼───────────────────────────────────────────┤
│ String       │ A single string value (rarely used)      │
│              │ "hello"                                   │
└──────────────┴───────────────────────────────────────────┘
```

---

## 3.2 Selectors and Matchers

```promql
# Exact match
http_requests_total{method="GET"}

# Not equal
http_requests_total{method!="DELETE"}

# Regex match
http_requests_total{status=~"2.."}       # Any 2xx status
http_requests_total{method=~"GET|POST"}  # GET or POST

# Negative regex
http_requests_total{status!~"4.."}       # NOT 4xx

# Multiple matchers (AND logic)
http_requests_total{method="GET", status="200", job="api"}

# Range vector (last 5 minutes of data)
http_requests_total{method="GET"}[5m]

# Time durations: s, m, h, d, w, y
# [30s] [5m] [1h] [7d] [4w] [1y]
```

---

## 3.3 Essential Functions

### rate() — Per-Second Rate of Increase

```promql
# Most important function for counters
rate(http_requests_total[5m])
# "How many requests per second, averaged over last 5 minutes"

# Why 5m?
# • Too short (1m): noisy, spiky
# • Too long (1h): smooths out real issues
# • 5m is a good default (covers at least 4 scrapes at 15s intervals)
```

### irate() — Instantaneous Rate

```promql
# Uses last two data points only — more volatile
irate(http_requests_total[5m])
# Better for graphs, worse for alerts
```

### increase() — Total Increase Over Period

```promql
# Total new requests in last hour
increase(http_requests_total[1h])
# Equivalent to: rate(http_requests_total[1h]) * 3600
```

### Aggregation Operators

```promql
# Sum across all instances
sum(rate(http_requests_total[5m]))

# Average per service
avg(rate(http_requests_total[5m])) by (service)

# Maximum memory across pods
max(container_memory_usage_bytes) by (pod)

# Count of targets
count(up == 1) by (job)
```

---

## 3.4 Common Query Patterns

```promql
# 1. Error Rate (%)
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
* 100

# 2. Request Rate (req/s)  
sum(rate(http_requests_total[5m])) by (service)

# 3. Latency Percentiles (from histogram)
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# 4. Average Latency
rate(http_request_duration_seconds_sum[5m])
/
rate(http_request_duration_seconds_count[5m])

# 5. CPU Usage (%)
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# 6. Memory Usage (%)
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes)
/
node_memory_MemTotal_bytes * 100

# 7. Disk Space Remaining (%)
node_filesystem_avail_bytes{mountpoint="/"}
/
node_filesystem_size_bytes{mountpoint="/"} * 100
```

---

## 3.5 Operators

```
┌─────────────────────────────────────────────────────┐
│ Arithmetic:  +  -  *  /  %  ^                      │
│ Comparison:  ==  !=  >  <  >=  <=                  │
│ Logical:     and  or  unless                       │
│ Aggregation: sum  avg  min  max  count  topk       │
│              bottomk  stddev  quantile             │
└─────────────────────────────────────────────────────┘

# Comparison (filtering)
http_requests_total > 1000      # Only series with value > 1000

# Logical
up == 1 and on(job) http_requests_total > 0
# Targets that are up AND receiving traffic
```

---

## 3.6 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `no data` result | Wrong label values or metric doesn't exist | Check available metrics at /metrics endpoint |
| `rate()` returns nothing | Applied to gauge (not counter) | Use raw gauge value or `deriv()` |
| Spiky graphs | `irate()` or window too short | Use `rate()` with 5m+ window |
| "many-to-many matching" error | Ambiguous vector math | Use `on()` and `group_left/right` |

---

## Summary Table

| Function | Purpose | Example |
|----------|---------|---------|
| `rate()` | Per-second rate over range | `rate(requests_total[5m])` |
| `irate()` | Instantaneous rate (last 2 points) | `irate(requests_total[5m])` |
| `increase()` | Total increase over range | `increase(requests_total[1h])` |
| `histogram_quantile()` | Percentile from histogram | `histogram_quantile(0.99, ...)` |
| `sum() by ()` | Sum grouped by labels | `sum(rate(...)) by (svc)` |
| `avg_over_time()` | Average over time window | `avg_over_time(cpu[1h])` |

---

## Quick Revision Questions

1. **What is the difference between an instant vector and a range vector?**
2. **Why should you use `rate()` instead of the raw counter value?**
3. **Write a PromQL query for the error rate percentage of a service.**
4. **What is the difference between `rate()` and `irate()`? When should you use each?**
5. **How do you calculate the p95 latency from a histogram metric?**

---

[← Previous: Installation](02-installation-configuration.md) | [Back to README](../README.md) | [Next: PromQL Advanced →](04-promql-advanced.md)
