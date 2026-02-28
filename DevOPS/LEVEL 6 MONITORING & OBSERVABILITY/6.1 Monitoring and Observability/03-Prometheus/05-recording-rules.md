# Chapter 5: Recording Rules

[← Previous: PromQL Advanced](04-promql-advanced.md) | [Back to README](../README.md) | [Next: Alerting Rules →](06-alerting-rules.md)

---

## Overview

Recording rules let you **precompute** expensive PromQL expressions and save the result as a new time series. This improves dashboard performance and simplifies complex queries.

---

## 5.1 Why Recording Rules?

```
Without recording rules:
  Dashboard loads → runs 20 complex PromQL queries → 15 seconds to load ❌

With recording rules:
  Prometheus precomputes every 15s → stores result as simple metric
  Dashboard loads → reads precomputed values → instant ✅
```

---

## 5.2 Naming Convention

```
level:metric:operations

Examples:
  job:http_requests:rate5m           # per-job request rate
  instance:node_cpu:avg_rate5m       # per-instance CPU avg
  cluster:http_errors:ratio_rate5m   # cluster-wide error ratio
```

---

## 5.3 Configuration

```yaml
# recording_rules.yml
groups:
  - name: http_recording_rules
    interval: 15s  # Evaluation interval (optional, defaults to global)
    rules:
      # Request rate per job
      - record: job:http_requests_total:rate5m
        expr: sum(rate(http_requests_total[5m])) by (job)

      # Error ratio per service
      - record: service:http_errors:ratio_rate5m
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
          /
          sum(rate(http_requests_total[5m])) by (service)

      # P99 latency per service
      - record: service:http_request_duration_seconds:p99_rate5m
        expr: |
          histogram_quantile(0.99,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
          )

  - name: node_recording_rules
    rules:
      # CPU usage per instance
      - record: instance:node_cpu_utilization:avg_rate5m
        expr: |
          1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)

      # Memory usage per instance
      - record: instance:node_memory_utilization:ratio
        expr: |
          1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
```

---

## 5.4 Validation

```bash
# Validate rules file
./promtool check rules recording_rules.yml

# Output: SUCCESS if valid
# Checking recording_rules.yml
#   SUCCESS: 5 rules found
```

---

## 5.5 When to Use

| Use Recording Rules | Don't Use Recording Rules |
|--------------------|--------------------------|
| Dashboards are slow | Simple, fast queries |
| Same expensive query used in multiple places | One-off exploration |
| Alert rules depend on complex expressions | Testing new queries |
| Federation needs precomputed results | Metrics with very short retention |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Purpose** | Precompute expensive queries |
| **Naming** | `level:metric:operations` |
| **Config** | `recording_rules.yml` → `rule_files` in prometheus.yml |
| **Validation** | `promtool check rules <file>` |
| **Benefit** | Faster dashboards, simpler alert expressions |

---

## Quick Revision Questions

1. **What is a recording rule and why would you use one?**
2. **What is the naming convention for recording rules?**
3. **Write a recording rule for the error rate of each service.**
4. **How do you validate recording rules before deploying?**
5. **When should you NOT use recording rules?**

---

[← Previous: PromQL Advanced](04-promql-advanced.md) | [Back to README](../README.md) | [Next: Alerting Rules →](06-alerting-rules.md)
