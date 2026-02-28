# Chapter 3: Metric Naming Conventions

[← Previous: Metric Types](02-metric-types.md) | [Back to README](../README.md) | [Next: Labels, Dimensions & Cardinality →](04-labels-dimensions-cardinality.md)

---

## Overview

Consistent metric naming is critical for maintainability, discoverability, and cross-team collaboration. Prometheus and OpenMetrics define clear conventions that should be followed.

---

## 3.1 Naming Rules

```
┌──────────────────────────────────────────────────────────┐
│              METRIC NAMING FORMAT                         │
│                                                           │
│   <namespace>_<subsystem>_<name>_<unit>                  │
│                                                           │
│   Example:                                                │
│   myapp_http_requests_total                              │
│   ─────  ────  ────────  ─────                           │
│   namespace subsystem name   unit/suffix                 │
│                                                           │
│   More examples:                                          │
│   myapp_http_request_duration_seconds                    │
│   myapp_db_connections_active                            │
│   node_filesystem_avail_bytes                            │
│   process_cpu_seconds_total                              │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Naming Best Practices

| Rule | Good | Bad |
|------|------|-----|
| Use snake_case | `http_request_duration` | `httpRequestDuration` |
| Include unit as suffix | `_seconds`, `_bytes`, `_total` | `_time`, `_size` |
| Use base units | `_seconds` (not `_milliseconds`) | `_ms`, `_minutes` |
| Use `_total` for counters | `http_requests_total` | `http_requests` |
| Use descriptive names | `http_request_duration_seconds` | `latency` |
| Add namespace prefix | `myapp_http_requests_total` | `http_requests_total` |
| Avoid type in name | `http_requests_total` | `http_requests_counter` |

### Standard Unit Suffixes

```
┌───────────────┬─────────────────────────────────────┐
│ Unit          │ Suffix                              │
├───────────────┼─────────────────────────────────────┤
│ Time          │ _seconds (not ms, us, min)          │
│ Bytes         │ _bytes (not kb, mb, gb)             │
│ Temperature   │ _celsius                            │
│ Ratio/Percent │ _ratio (0-1) or _percent (0-100)   │
│ Count/total   │ _total (for counters)               │
│ Information   │ _info (metadata gauge, value=1)     │
└───────────────┴─────────────────────────────────────┘
```

---

## 3.3 Examples by Category

```
┌─────────────────────────────────────────────────────────────┐
│              WELL-NAMED METRICS                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│ HTTP/API Metrics:                                            │
│   http_requests_total{method, status, endpoint}             │
│   http_request_duration_seconds{method, endpoint}           │
│   http_request_size_bytes{method, endpoint}                 │
│   http_response_size_bytes{method, endpoint}                │
│                                                              │
│ Database Metrics:                                            │
│   db_connections_active{database}                           │
│   db_connections_max{database}                              │
│   db_query_duration_seconds{query_type, database}           │
│   db_errors_total{database, error_type}                     │
│                                                              │
│ Queue/Messaging Metrics:                                     │
│   queue_messages_total{queue, action}                       │
│   queue_depth{queue}                                        │
│   queue_processing_duration_seconds{queue}                  │
│                                                              │
│ Business Metrics:                                            │
│   orders_created_total{region, payment_method}              │
│   orders_amount_dollars{region}                             │
│   users_registered_total{source}                            │
│   cart_abandonment_total{step}                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 3.4 Anti-Patterns

```
❌ BAD:                              ✅ GOOD:
────────                             ───────
request_time_ms                      http_request_duration_seconds
numErrors                            http_errors_total
cpu                                  process_cpu_seconds_total
diskUsage                            node_filesystem_used_bytes
HTTPRequestsCounter                  http_requests_total
my-metric-name                       my_metric_name
latency_p99                          Use histogram_quantile() instead
error_rate                           Derive from errors_total / requests_total
```

---

## Summary Table

| Convention | Rule | Example |
|-----------|------|---------|
| Case | snake_case only | `http_request_duration_seconds` |
| Units | Base units as suffix | `_seconds`, `_bytes` |
| Counters | End with `_total` | `http_requests_total` |
| Namespace | Prefix with app name | `myapp_http_requests_total` |
| Avoid | Don't include type name | Not `_counter`, `_gauge` |

---

## Quick Revision Questions

1. **What is the correct naming format for Prometheus metrics?**
2. **Why should you use base units (seconds, bytes) instead of derived units (milliseconds, kilobytes)?**
3. **What suffix should all counter metrics have?**
4. **Fix this metric name: `apiRequestTimeMs` — what should it be?**
5. **Why is including a namespace prefix important for metric names?**

---

[← Previous: Metric Types](02-metric-types.md) | [Back to README](../README.md) | [Next: Labels, Dimensions & Cardinality →](04-labels-dimensions-cardinality.md)
