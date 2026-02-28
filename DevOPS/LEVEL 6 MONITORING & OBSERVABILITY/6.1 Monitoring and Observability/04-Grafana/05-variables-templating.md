# Chapter 5: Variables & Templating

[← Previous: Panels & Visualizations](04-panels-visualizations.md) | [Back to README](../README.md) | [Next: Alerting in Grafana →](06-alerting-grafana.md)

---

## Overview

Template variables make dashboards interactive and reusable. Instead of hardcoding values like service names or environments, variables allow users to select from dropdowns, making one dashboard serve many use cases.

---

## 5.1 How Variables Work

```
┌──────────────────────────────────────────────────────────┐
│  Dashboard: Service Overview                             │
│                                                          │
│  ┌─ env ──────┐  ┌─ service ──────┐  ┌─ instance ───┐  │
│  │ production ▼│  │ api-server   ▼│  │ All         ▼│  │
│  └────────────┘  └────────────────┘  └──────────────┘  │
│                                                          │
│  Queries use: $env, $service, $instance                 │
│  rate(http_requests_total{env="$env",                   │
│       service="$service", instance=~"$instance"}[5m])   │
│                                                          │
│  When user changes dropdown → all panels re-query       │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 Variable Types

| Type | Description | Example |
|------|------------|---------|
| **Query** | Populated from data source query | `label_values(up, job)` |
| **Custom** | Hardcoded list of values | `dev, staging, prod` |
| **Constant** | Single fixed value | `prometheus` |
| **Interval** | Time intervals | `1m, 5m, 15m, 1h` |
| **Data source** | List of data sources by type | All Prometheus instances |
| **Text box** | Free-form user input | Custom regex filter |
| **Ad hoc** | Auto-generates key/value filters | Dynamic label filters |

---

## 5.3 Query Variable Examples

### Prometheus Label Values

```promql
# All job names
label_values(up, job)

# All instances for a specific job
label_values(up{job="$job"}, instance)

# All namespaces in Kubernetes
label_values(kube_pod_info, namespace)

# Services filtered by environment
label_values(http_requests_total{env="$env"}, service)
```

### Chained Variables (Cascading)

```
Variable 1: env
  Query: label_values(http_requests_total, env)
  Values: [dev, staging, prod]

Variable 2: service  (depends on $env)
  Query: label_values(http_requests_total{env="$env"}, service)
  Values: [api, web, worker]   ← changes when env changes

Variable 3: instance  (depends on $env AND $service)
  Query: label_values(http_requests_total{env="$env", service="$service"}, instance)
  Values: [10.0.1.1:8080, 10.0.1.2:8080]
```

```
  env=prod ──────┐
                  ├──→ service=api ──────┐
                  │                       ├──→ instance=10.0.1.1
                  │                       └──→ instance=10.0.1.2
                  ├──→ service=web
                  └──→ service=worker
```

---

## 5.4 Using Variables in Queries

### Single Value
```promql
rate(http_requests_total{job="$job"}[5m])
```

### Multi-Value (with regex)
When "Multi-value" is enabled:
```promql
# Use =~ for regex matching of multiple selected values
rate(http_requests_total{job=~"$job"}[5m])
# $job expands to: api|web|worker
```

### Variable in Legend
```
Legend: {{method}} - $service
```

### Variable in Panel Title
```
Title: Request Rate - $service ($env)
```

---

## 5.5 Interval Variable & `$__interval`

```yaml
# Custom interval variable
Name: interval
Type: Interval
Values: 1m, 5m, 15m, 30m, 1h

# Usage in query
rate(http_requests_total[${interval}])

# Auto interval (Grafana calculates based on time range)
rate(http_requests_total[$__interval])

# Built-in variables
$__interval     # Auto-calculated step (e.g., 15s, 1m)
$__interval_ms  # Same in milliseconds
$__range        # Selected time range duration
$__range_s      # Range in seconds
$__rate_interval # Safe rate interval (max of $__interval + scrape_interval)
```

---

## 5.6 Variable Formatting Options

```
$var            → value1      (default)
${var}          → value1      (explicit syntax)
${var:csv}      → value1,value2,value3
${var:pipe}     → value1|value2|value3
${var:regex}    → (value1|value2|value3)
${var:glob}     → {value1,value2,value3}
${var:json}     → ["value1","value2"]
${var:queryparam} → var=value1&var=value2
```

---

## 5.7 Practical Example: Multi-Service Dashboard

```yaml
# Variable definitions
Variables:
  - name: datasource
    type: datasource
    query: prometheus

  - name: env
    type: query
    datasource: $datasource
    query: label_values(http_requests_total, env)
    multi: false
    includeAll: false

  - name: service
    type: query
    datasource: $datasource
    query: label_values(http_requests_total{env="$env"}, service)
    multi: true
    includeAll: true
    allValue: ".*"

  - name: interval
    type: interval
    values: "1m,5m,15m,30m,1h"
    auto: true
    auto_min: "10s"

# Panel query using all variables
Query: |
  sum by (service) (
    rate(http_requests_total{
      env="$env",
      service=~"$service"
    }[$interval])
  )
Legend: "{{service}}"
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Dropdown is empty | Query returns no values | Test `label_values()` in Explore |
| Multi-value not working | Using `=` instead of `=~` | Use regex match: `{job=~"$job"}` |
| "All" option not working | Missing `allValue` config | Set All value to `.*` |
| Variable changes not reflected | Query caching | Decrease cache timeout or refresh |
| Chained variable stale | Update order wrong | Set correct variable ordering |

---

## Quick Revision Questions

1. **What is the purpose of template variables in Grafana?**
2. **How do chained (cascading) variables work?**
3. **What is the difference between `$var` and `${var:regex}`?**
4. **When would you use `$__interval` versus a custom interval variable?**
5. **How do you configure "All" option for a multi-value variable?**
6. **What is the `$__rate_interval` variable and why is it preferred over `$__interval` for rate()?**

---

[← Previous: Panels & Visualizations](04-panels-visualizations.md) | [Back to README](../README.md) | [Next: Alerting in Grafana →](06-alerting-grafana.md)
