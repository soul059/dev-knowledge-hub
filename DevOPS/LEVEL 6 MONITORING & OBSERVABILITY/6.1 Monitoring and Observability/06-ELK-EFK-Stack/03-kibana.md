# Chapter 3: Kibana

[← Previous: Logstash & Fluentd](02-logstash-fluentd.md) | [Back to README](../README.md) | [Next: Filebeat & Agents →](04-filebeat-agents.md)

---

## Overview

Kibana is the visualization layer for the Elastic Stack, providing log exploration, dashboards, and analytics for data stored in Elasticsearch.

---

## 3.1 Kibana Architecture

```
┌──────────────────────────────────────────────────────────┐
│                  KIBANA COMPONENTS                        │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │                   KIBANA UI                       │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐         │   │
│  │  │ Discover  │ │Dashboard │ │ Lens     │         │   │
│  │  │ (Log      │ │ (Custom  │ │ (Drag &  │         │   │
│  │  │  explore) │ │  views)  │ │  Drop)   │         │   │
│  │  └──────────┘ └──────────┘ └──────────┘         │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐         │   │
│  │  │ Canvas   │ │ Maps     │ │ Alerts   │         │   │
│  │  └──────────┘ └──────────┘ └──────────┘         │   │
│  └──────────────────────┬───────────────────────────┘   │
│                         │ queries                        │
│                         ▼                                │
│  ┌──────────────────────────────────────────────────┐   │
│  │              ELASTICSEARCH                        │   │
│  │              (data storage & search)               │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Discover — Log Exploration

```
┌──────────────────────────────────────────────────────────┐
│  DISCOVER VIEW                                           │
│  ┌─ Time Range ─┐  ┌─ Data View ──┐                    │
│  │ Last 15 min  ▼│  │ logs-*      ▼│                    │
│  └──────────────┘  └──────────────┘                    │
│                                                          │
│  Search: level:ERROR AND service:api-server             │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Log Volume Histogram                             │   │
│  │  ▁▂▃▅▇▅▃▂▁▁▂▃▅▇▅▃▂▁                            │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────┬──────────┬────────────────────────────────┐   │
│  │ Time │ Level    │ Message                         │   │
│  ├──────┼──────────┼────────────────────────────────┤   │
│  │10:24 │ ERROR    │ Payment timeout after 30s       │   │
│  │10:23 │ ERROR    │ DB connection refused            │   │
│  │10:22 │ WARN     │ Retry attempt 3/5               │   │
│  │10:21 │ ERROR    │ Queue consumer lag detected      │   │
│  └──────┴──────────┴────────────────────────────────┘   │
│                                                          │
│  Fields sidebar:     Click to add columns               │
│  │ timestamp                                             │
│  │ level          ← Filter: level:ERROR                 │
│  │ service        ← Filter: service:api-server          │
│  │ message                                               │
│  │ trace_id       ← Click to view trace                 │
└──────────────────────────────────────────────────────────┘
```

---

## 3.3 KQL — Kibana Query Language

```
# Simple field search
level:ERROR

# Multiple conditions (AND)
level:ERROR AND service:api-server

# OR condition
level:ERROR OR level:WARN

# Wildcard
message:*timeout*

# Range
status >= 400 AND status < 500

# Date range
@timestamp >= "2024-01-15" AND @timestamp < "2024-01-16"

# Nested field
kubernetes.namespace:production

# NOT
NOT level:DEBUG

# Combined
level:ERROR AND service:api-server AND NOT message:*health*
```

### Lucene Query Syntax (Alternative)

```
# Phrase search
message:"connection refused"

# Fuzzy search
message:timout~     (matches "timeout")

# Regex
message:/pay.*fail/

# Exists
_exists_:trace_id
```

---

## 3.4 Creating Dashboards

### Visualization Types in Kibana

| Type | Use Case |
|------|----------|
| Lens | Drag-and-drop, auto-suggest (recommended) |
| Line/Area/Bar | Time series trends |
| Pie/Donut | Proportional breakdown |
| Metric | Single number display |
| Data Table | Tabular aggregations |
| Heatmap | Distribution over time |
| Maps | Geographic data |
| TSVB | Advanced time series |
| Timelion | Time series expressions |
| Canvas | Pixel-perfect reports |

### Dashboard Best Practices

```
Layout (similar to Grafana):
  Top:     High-level metrics (Metric panels)
  Middle:  Time series trends (Line/Area charts)
  Bottom:  Detail tables and logs

  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
  │ Total  │ │ Error  │ │ P99    │ │ Active │
  │ Logs:  │ │ Rate:  │ │ Latency│ │ Users: │
  │ 1.2M   │ │ 2.3%   │ │ 145ms  │ │ 5,432  │
  └────────┘ └────────┘ └────────┘ └────────┘
  ┌────────────────────────────────────────────┐
  │  Errors Over Time (Area Chart)             │
  └────────────────────────────────────────────┘
  ┌────────────────────────────────────────────┐
  │  Top Error Messages (Table)                │
  └────────────────────────────────────────────┘
```

---

## 3.5 Data Views (Index Patterns)

```
# Data views define which indices Kibana queries

Data View: logs-*
  Matches: logs-2024.01.14, logs-2024.01.15, ...
  Time field: @timestamp
  
Data View: metrics-*
  Matches: metrics-2024.01.14, metrics-2024.01.15, ...
  Time field: @timestamp

# Create via API:
POST /api/data_views/data_view
{
  "data_view": {
    "title": "logs-*",
    "timeFieldName": "@timestamp"
  }
}
```

---

## 3.6 Kibana Spaces & Security

```
Spaces:
  ├── Production     (ops team)
  │   ├── Dashboards: Prod Overview, Alerts
  │   └── Data Views: logs-prod-*
  │
  ├── Development    (dev team)
  │   ├── Dashboards: Dev Debug
  │   └── Data Views: logs-dev-*
  │
  └── Security       (security team)
      ├── Dashboards: Audit, Threat
      └── Data Views: audit-*, security-*

RBAC Roles:
  viewer:  Read-only access to dashboards
  editor:  Create/edit dashboards and visualizations
  admin:   Manage spaces, users, data views
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "No results" in Discover | Wrong time range or data view | Check time picker, verify index pattern |
| Fields not showing | Mapping not refreshed | Refresh data view field list |
| Dashboard slow | Heavy aggregations on large data | Use date filters, pre-aggregate with transforms |
| "Shard failures" | Cluster overloaded | Check ES cluster health, add resources |
| Saved objects missing | Wrong Kibana space | Switch spaces, export/import objects |

---

## Quick Revision Questions

1. **What is the Discover feature and how do you use it for log exploration?**
2. **Write a KQL query to find all errors from the API service in the last hour.**
3. **What are Data Views (index patterns) and why are they needed?**
4. **What are Kibana Spaces and how do they help with multi-team access?**
5. **Compare Lens vs TSVB for creating visualizations.**
6. **How does Kibana differ from Grafana for log visualization?**

---

[← Previous: Logstash & Fluentd](02-logstash-fluentd.md) | [Back to README](../README.md) | [Next: Filebeat & Agents →](04-filebeat-agents.md)
