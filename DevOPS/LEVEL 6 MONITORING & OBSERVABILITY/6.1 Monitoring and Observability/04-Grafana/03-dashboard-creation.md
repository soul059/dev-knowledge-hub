# Chapter 3: Dashboard Creation

[← Previous: Data Sources](02-data-sources.md) | [Back to README](../README.md) | [Next: Panels & Visualizations →](04-panels-visualizations.md)

---

## Overview

Dashboards are the primary way to visualize data in Grafana. A dashboard consists of rows and panels, each querying one or more data sources.

---

## 3.1 Dashboard Anatomy

```
┌──────────────────────────────────────────────────────────┐
│  Dashboard: Production Overview                          │
│  ┌─ Time Picker ─┐  ┌─ Variables ─┐  ┌─ Auto-refresh ─┐│
│  │ Last 6 hours   │  │ env=prod    │  │ Every 30s      ││
│  └────────────────┘  └─────────────┘  └────────────────┘│
│                                                          │
│  ── Row: Service Health ──────────────────────────────   │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │
│  │  Stat Panel  │ │  Gauge Panel │ │  Stat Panel  │     │
│  │  Uptime: 99% │ │  CPU: 45%    │ │  RPS: 1.2k   │     │
│  └──────────────┘ └──────────────┘ └──────────────┘     │
│                                                          │
│  ── Row: Request Metrics ─────────────────────────────   │
│  ┌────────────────────────┐ ┌────────────────────────┐   │
│  │  Time Series Panel     │ │  Time Series Panel     │   │
│  │  ▄▅▆▇█▇▆▅▄▃▂▁▂▃▄▅    │ │  ▁▂▃▄▅▆▇█▇▆▅▄▃▂▁▂    │   │
│  │  Request Rate          │ │  Error Rate            │   │
│  └────────────────────────┘ └────────────────────────┘   │
│                                                          │
│  ── Row: Latency ─────────────────────────────────────   │
│  ┌──────────────────────────────────────────────────┐    │
│  │  Heatmap Panel                                    │    │
│  │  ░░▒▓█▓▒░░░░▒▓█▓▒░░                             │    │
│  │  Request Duration Distribution                    │    │
│  └──────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Creating a Dashboard Step-by-Step

### Step 1: Create New Dashboard
- Click **+** → **Dashboard** → **Add new panel**

### Step 2: Configure the Query
```
# Example: HTTP request rate per status code
Metric: rate(http_requests_total{job="api-server"}[5m])
Legend: {{status_code}}
```

### Step 3: Choose Visualization
- Time series for trends
- Stat for single values
- Table for tabular data

### Step 4: Set Panel Options
- Title, description
- Thresholds (green/yellow/red)
- Unit format (req/s, bytes, percent)

### Step 5: Save Dashboard
- Add tags (e.g., `production`, `api`)
- Choose folder
- Set UID for version control

---

## 3.3 Dashboard JSON Model

Every Grafana dashboard is stored as JSON:

```json
{
  "dashboard": {
    "id": null,
    "uid": "prod-overview",
    "title": "Production Overview",
    "tags": ["production", "overview"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "type": "timeseries",
        "title": "Request Rate",
        "gridPos": { "h": 8, "w": 12, "x": 0, "y": 0 },
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{status}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "reqps",
            "thresholds": {
              "steps": [
                { "color": "green", "value": null },
                { "color": "red", "value": 1000 }
              ]
            }
          }
        }
      }
    ],
    "time": { "from": "now-6h", "to": "now" },
    "refresh": "30s"
  }
}
```

---

## 3.4 Common Dashboard Patterns

### The Four Golden Signals Dashboard

```
 Row 1: Latency
 ┌────────────────────────┐ ┌────────────────────────┐
 │ P50/P90/P99 Latency    │ │ Latency Heatmap        │
 └────────────────────────┘ └────────────────────────┘

 Row 2: Traffic
 ┌────────────────────────┐ ┌────────────────────────┐
 │ Requests/sec           │ │ By Endpoint            │
 └────────────────────────┘ └────────────────────────┘

 Row 3: Errors
 ┌────────────────────────┐ ┌────────────────────────┐
 │ Error Rate %           │ │ Errors by Type         │
 └────────────────────────┘ └────────────────────────┘

 Row 4: Saturation
 ┌────────────────────────┐ ┌────────────────────────┐
 │ CPU / Memory / Disk    │ │ Connection Pool Usage  │
 └────────────────────────┘ └────────────────────────┘
```

### PromQL for Each Signal

```promql
# Latency (P99)
histogram_quantile(0.99, rate(http_duration_seconds_bucket[5m]))

# Traffic
sum(rate(http_requests_total[5m]))

# Errors
sum(rate(http_requests_total{status=~"5.."}[5m]))
/ sum(rate(http_requests_total[5m])) * 100

# Saturation
1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))
```

---

## 3.5 Dashboard Management

| Action | API Call | CLI |
|--------|----------|-----|
| Export JSON | `GET /api/dashboards/uid/:uid` | `grafana-cli dashboard export` |
| Import JSON | `POST /api/dashboards/db` | UI: + → Import |
| List all | `GET /api/search` | — |
| Delete | `DELETE /api/dashboards/uid/:uid` | — |
| Folder list | `GET /api/folders` | — |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "No data" in panel | Wrong time range or query | Check time picker, test query in Explore |
| Dashboard loads slowly | Too many panels or wide time range | Use variables, limit panels to 20-25 |
| Data looks flat/wrong | Wrong aggregation or unit | Check rate() vs raw counter, verify unit |
| Changes not saving | Read-only provisioned dashboard | Set `editable: true` or copy to editable folder |

---

## Quick Revision Questions

1. **What are the key components of a Grafana dashboard?**
2. **How is a dashboard represented internally (what format)?**
3. **What are the Four Golden Signals and what PromQL would you use for each?**
4. **How do you export and import dashboards via the API?**
5. **What causes a "No data" result in a panel and how do you debug it?**
6. **Why might a dashboard load slowly and how would you fix it?**

---

[← Previous: Data Sources](02-data-sources.md) | [Back to README](../README.md) | [Next: Panels & Visualizations →](04-panels-visualizations.md)
