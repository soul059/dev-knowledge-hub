# Chapter 7: Dashboard Best Practices

[← Previous: Alerting in Grafana](06-alerting-grafana.md) | [Back to README](../README.md) | [Next: Provisioning Dashboards →](08-provisioning-dashboards.md)

---

## Overview

Well-designed dashboards reduce mean time to detection (MTTD) and mean time to resolution (MTTR). Poorly designed dashboards add cognitive load and slow down incident response.

---

## 7.1 Dashboard Design Principles

```
┌──────────────────────────────────────────────────────────┐
│            DASHBOARD DESIGN PYRAMID                      │
│                                                          │
│                    ┌──────┐                              │
│                    │Detail│  Drill-down dashboards       │
│                   /│      │\  (per-service, per-node)    │
│                  / └──────┘ \                            │
│                 /             \                           │
│                ┌──────────────┐                          │
│                │  Service     │  Service-level dashboards │
│               /│  Dashboards  │\ (API, DB, Cache)        │
│              / └──────────────┘ \                        │
│             /                    \                        │
│            ┌──────────────────────┐                      │
│            │   Overview / Home    │  High-level overview  │
│            │   Dashboard          │  (all services)       │
│            └──────────────────────┘                      │
│                                                          │
│  Navigation: Overview → Service → Detail                │
└──────────────────────────────────────────────────────────┘
```

---

## 7.2 Layout Guidelines

### The 5-Second Rule
A user should understand system health within 5 seconds of looking at the dashboard.

```
┌──────────────────────────────────────────────────────────┐
│  TOP: Key health indicators (Stat/Gauge panels)         │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│  │ Uptime │ │ Error% │ │ P99 ms │ │ RPS    │           │
│  │ 99.9%  │ │  0.3%  │ │  145   │ │ 1.2k   │           │
│  └────────┘ └────────┘ └────────┘ └────────┘           │
│                                                          │
│  MIDDLE: Trends and time series                         │
│  ┌────────────────────────┐ ┌──────────────────────┐    │
│  │ Request Rate (Graph)    │ │ Error Rate (Graph)   │    │
│  └────────────────────────┘ └──────────────────────┘    │
│                                                          │
│  BOTTOM: Details (tables, logs)                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Top Endpoints by Latency (Table)                  │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘

Flow: Eyes scan top-to-bottom, left-to-right
```

---

## 7.3 Do's and Don'ts

| Do | Don't |
|----|-------|
| Use consistent color schemes | Use random colors per panel |
| Limit to 15-20 panels per dashboard | Cram 50+ panels on one page |
| Use variables for reusability | Hardcode service names in queries |
| Add meaningful panel titles | Use auto-generated titles |
| Include units (req/s, ms, %) | Show raw numbers without context |
| Use collapsible rows | Show everything at once |
| Link to runbooks from panels | Assume everyone knows the fix |
| Set appropriate time ranges | Default to "Last 90 days" |
| Use thresholds for quick status | Rely on users to know what's normal |
| Version control dashboard JSON | Rely on Grafana's internal versioning only |

---

## 7.4 Color Standards

```
Standard Color Conventions:

  🟢 Green   = Healthy / Normal / OK
  🟡 Yellow  = Warning / Degraded / Approaching limit
  🔴 Red     = Critical / Error / Down
  🔵 Blue    = Informational / Neutral
  🟣 Purple  = Special / Custom metric

Threshold Examples:
  Error Rate:    0% → Green,  1% → Yellow,  5% → Red
  CPU Usage:     0% → Green, 70% → Yellow, 90% → Red
  Latency P99:   0  → Green, 200ms → Yellow, 500ms → Red
  Disk Usage:    0% → Green, 75% → Yellow, 90% → Red
```

---

## 7.5 Dashboard Organization

### Folder Structure
```
Grafana Folders:
├── Overview/
│   └── Platform Overview
├── Infrastructure/
│   ├── Node Exporter
│   ├── Kubernetes Cluster
│   └── Network
├── Services/
│   ├── API Server
│   ├── Web Frontend
│   └── Background Workers
├── Databases/
│   ├── PostgreSQL
│   ├── Redis
│   └── Elasticsearch
└── Business/
    ├── Revenue Metrics
    └── User Analytics
```

### Naming Convention
```
Format: [Category] Service - Aspect

Examples:
  [Infra] Kubernetes - Cluster Overview
  [Service] API - Performance
  [DB] PostgreSQL - Queries & Connections
  [Business] Orders - Real-time Metrics
```

---

## 7.6 Dashboard Links & Drill-Down

```yaml
# Panel links - click a panel to drill down
Panel: "Error Rate by Service"
Links:
  - title: "View Service Details"
    url: "/d/service-detail?var-service=${__field.labels.service}"
    targetBlank: false

# Dashboard links - top-of-dashboard navigation
Dashboard Links:
  - type: dashboards
    tags: [production]
    title: "Related Dashboards"

  - type: link
    title: "Runbook"
    url: "https://wiki.example.com/runbooks"
    icon: doc
```

---

## 7.7 Performance Optimization

| Technique | Impact | How |
|-----------|--------|-----|
| Use recording rules | High | Pre-compute expensive queries in Prometheus |
| Limit time range default | Medium | Set default to 1h or 6h, not 30d |
| Use `$__rate_interval` | Medium | Avoids unnecessary granularity |
| Limit panel count | Medium | 15-20 panels max, use rows |
| Avoid `{__name__=~".*"}` | High | Never query all metrics |
| Use `topk()` in tables | Medium | Limit result set size |
| Set min query interval | Low | Prevent sub-second queries |

---

## Summary Table

| Principle | Description |
|-----------|-------------|
| Hierarchy | Overview → Service → Detail |
| 5-Second Rule | Health visible at a glance |
| Consistency | Same colors, units, layouts across dashboards |
| Reusability | Template variables over hardcoded values |
| Context | Units, thresholds, descriptions, runbook links |
| Performance | Recording rules, limited panels, sane time ranges |

---

## Quick Revision Questions

1. **What is the dashboard design pyramid and why is it important?**
2. **What is the 5-second rule for dashboard design?**
3. **How should panels be ordered on a dashboard (top to bottom)?**
4. **What are three techniques to improve dashboard loading performance?**
5. **How do dashboard links enable drill-down navigation?**
6. **What naming convention would you use for organizing dashboards?**

---

[← Previous: Alerting in Grafana](06-alerting-grafana.md) | [Back to README](../README.md) | [Next: Provisioning Dashboards →](08-provisioning-dashboards.md)
