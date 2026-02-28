# Chapter 4: PromQL Advanced Queries

[← Previous: PromQL Basics](03-promql-basics.md) | [Back to README](../README.md) | [Next: Recording Rules →](05-recording-rules.md)

---

## Overview

This chapter covers advanced PromQL techniques including vector matching, subqueries, label manipulation, and complex real-world query patterns.

---

## 4.1 Vector Matching

### One-to-One Matching

```promql
# Divide errors by total — labels must match exactly
rate(http_requests_total{status=~"5.."}[5m])
/
rate(http_requests_total[5m])
# Only works if BOTH sides have identical label sets

# Use ignoring() to ignore specific labels
rate(http_errors_total[5m])
/ ignoring(error_type)
rate(http_requests_total[5m])

# Use on() to match on specific labels only
method:http_requests:rate5m{method="GET"}
/ on(method)
method:http_errors:rate5m{method="GET"}
```

### Many-to-One / One-to-Many

```promql
# group_left: many-to-one (left side has more series)
rate(http_requests_total[5m])
* on(instance) group_left(node_name)
node_info

# group_right: one-to-many (right side has more series)
node_info
* on(instance) group_right()
rate(http_requests_total[5m])
```

---

## 4.2 Subqueries

```promql
# Subquery syntax: <expr>[<range>:<resolution>]

# Max of 5-minute rates computed every minute over 1 hour
max_over_time(rate(http_requests_total[5m])[1h:1m])

# P99 latency over multiple windows
max_over_time(
  histogram_quantile(0.99,
    sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
  )[1h:5m]
)
```

---

## 4.3 Label Manipulation

```promql
# label_replace — add/modify labels
label_replace(up, "hostname", "$1", "instance", "(.*):.*")
# Extracts hostname from "server1:9090" → hostname="server1"

# label_join — concatenate labels
label_join(up, "full_name", "-", "job", "instance")
# Creates full_name="prometheus-localhost:9090"
```

---

## 4.4 Prediction and Trends

```promql
# predict_linear — predict future value using linear regression
predict_linear(node_filesystem_avail_bytes[6h], 24*3600)
# "What will disk space be in 24 hours based on 6-hour trend?"

# Alert when disk predicted full within 4 hours
predict_linear(node_filesystem_avail_bytes[6h], 4*3600) < 0

# deriv — per-second derivative of a gauge
deriv(node_memory_MemAvailable_bytes[1h])
# Positive = memory increasing, negative = memory decreasing

# changes — count value changes
changes(process_start_time_seconds[1h])
# "How many times did this process restart in the last hour?"

# resets — count counter resets
resets(http_requests_total[1h])
```

---

## 4.5 Advanced Patterns

```promql
# SLO Error Budget burn rate
(
  sum(rate(http_requests_total{status=~"5.."}[1h]))
  / sum(rate(http_requests_total[1h]))
) / 0.001  # 0.001 = 1 - 0.999 (SLO target)
# Result > 1 means burning budget faster than allowed

# Apdex score
(
  sum(rate(http_request_duration_seconds_bucket{le="0.5"}[5m])) +
  sum(rate(http_request_duration_seconds_bucket{le="2.0"}[5m]))
) / 2
/ sum(rate(http_request_duration_seconds_count[5m]))

# Request percentages by status code
sum(rate(http_requests_total[5m])) by (status)
/ ignoring(status) group_left
sum(rate(http_requests_total[5m]))
* 100

# Saturation — ratio of current to max
sum(db_connections_active) by (database)
/ sum(db_connections_max) by (database)
* 100
```

---

## 4.6 Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "many-to-many matching not allowed" | Add `on(label)` or `ignoring(label)` + `group_left/right` |
| Subquery too slow | Reduce resolution or use recording rules |
| `predict_linear` returns NaN | Not enough data points, increase range |
| `label_replace` no effect | Check regex — it uses RE2 syntax |

---

## Summary Table

| Technique | Purpose | Syntax |
|-----------|---------|--------|
| `on()`/`ignoring()` | Control label matching | `A / on(method) B` |
| `group_left/right` | Many-to-one joins | `A * on(x) group_left B` |
| Subqueries | Nested range queries | `max_over_time(rate(...)[1h:5m])` |
| `predict_linear()` | Forecast values | `predict_linear(metric[6h], 86400)` |
| `label_replace()` | Transform labels | `label_replace(m, "new", "$1", "old", "(.*))")` |
| `changes()` | Count value changes | `changes(metric[1h])` |

---

## Quick Revision Questions

1. **What is the difference between `on()` and `ignoring()` in PromQL vector matching?**
2. **When do you need `group_left` or `group_right`?**
3. **Write a query to predict when a disk will be full using `predict_linear()`.**
4. **What is a subquery? Give a practical example.**
5. **How would you calculate the SLO error budget burn rate in PromQL?**

---

[← Previous: PromQL Basics](03-promql-basics.md) | [Back to README](../README.md) | [Next: Recording Rules →](05-recording-rules.md)
