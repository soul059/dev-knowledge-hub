# Chapter 4: Panels & Visualizations

[← Previous: Dashboard Creation](03-dashboard-creation.md) | [Back to README](../README.md) | [Next: Variables & Templating →](05-variables-templating.md)

---

## Overview

Panels are the building blocks of Grafana dashboards. Each panel supports different visualization types optimized for specific data representations.

---

## 4.1 Panel Types

```
┌───────────────────────────────────────────────────────────────┐
│                  GRAFANA PANEL TYPES                          │
├──────────────────┬────────────────────────────────────────────┤
│                  │                                            │
│  TIME SERIES     │  ▁▂▃▄▅▆▇█▇▆▅▄   Line/Area/Bar over time │
│                  │                                            │
│  STAT            │  ┌─────────┐     Single big number        │
│                  │  │  99.9%  │                               │
│                  │  └─────────┘                               │
│                  │                                            │
│  GAUGE           │  ╭───╮           Circular gauge 0-100     │
│                  │  │45%│                                     │
│                  │  ╰───╯                                     │
│                  │                                            │
│  BAR GAUGE       │  ████████░░░░   Horizontal/vertical bars  │
│                  │                                            │
│  TABLE           │  ┌──┬──┬──┐     Tabular data             │
│                  │  ├──┼──┼──┤                               │
│                  │  └──┴──┴──┘                               │
│                  │                                            │
│  HEATMAP         │  ░▒▓█▓▒░        Distribution over time   │
│                  │                                            │
│  PIE CHART       │  ◕             Proportional breakdown     │
│                  │                                            │
│  LOGS            │  ≡≡≡            Log lines viewer          │
│                  │                                            │
│  NODE GRAPH      │  ●──●──●       Service topology          │
│                  │                                            │
│  GEOMAP          │  🗺            Geographic data            │
│                  │                                            │
└──────────────────┴────────────────────────────────────────────┘
```

---

## 4.2 Choosing the Right Visualization

| Data Type | Best Panel | Example |
|-----------|-----------|---------|
| Trend over time | Time Series | CPU usage over 24h |
| Current value | Stat | Current error rate |
| Percentage/ratio | Gauge | Disk usage % |
| Comparison | Bar Chart/Bar Gauge | RPS per service |
| Distribution | Heatmap | Request latency distribution |
| Tabular | Table | Top 10 endpoints by latency |
| Proportional | Pie Chart | Traffic by region |
| Logs | Logs panel | Application log stream |
| Topology | Node Graph | Service dependency map |

---

## 4.3 Time Series Panel (Most Common)

### Configuration Options

```yaml
Panel Settings:
  Title: "Request Rate"
  Description: "HTTP requests per second by status code"

Query:
  Expression: rate(http_requests_total{job="api"}[5m])
  Legend: "{{method}} {{status}}"
  Min step: 15s

Display:
  Style: Lines           # Lines, Bars, Points
  Line width: 2
  Fill opacity: 10
  Point size: 5
  Stack series: None     # None, Normal, 100%

Axis:
  Scale: Linear          # Linear, Log (base 2), Log (base 10)
  Label: "req/s"
  Decimals: 2

Thresholds:
  - 0    → Green
  - 500  → Yellow (Warning)
  - 1000 → Red (Critical)

Overrides:               # Per-series customization
  - matcher: "5xx"
    properties:
      color: red
      lineWidth: 3
```

---

## 4.4 Stat Panel

```
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   🟢 99.9%   │ │   🟡 78%     │ │   🔴 45%     │
│   Uptime     │ │   CPU        │ │   Memory Free │
│   ▁▂▃▂▁▂▃▂  │ │   ▅▆▇▆▅▆▇▆  │ │   ▇▆▅▄▃▂▁▁  │
│   Sparkline  │ │   Sparkline  │ │   Sparkline  │
└──────────────┘ └──────────────┘ └──────────────┘

Config:
  Calculation: Last (not null)
  Color mode: Background   # Value, Background, None
  Graph mode: Area          # None, Area
  Text mode: Auto           # Auto, Value, Name, Value+Name
  Orientation: Auto
```

---

## 4.5 Table Panel

```promql
# Query for table
topk(10,
  sum by (handler) (rate(http_requests_total[5m]))
)
```

Table transforms:
```
Transform Pipeline:
  1. Reduce → Calculate "Last" for each series
  2. Organize fields → Reorder/rename columns
  3. Sort by → Descending by "Value"

Column Overrides:
  - "handler" → Rename to "Endpoint"
  - "Value"   → Unit: req/s, Decimals: 2
  - "Value"   → Cell display mode: Color background
```

---

## 4.6 Heatmap Panel

Best for histogram distributions over time:

```promql
# Query: Use raw histogram buckets
sum(increase(http_duration_seconds_bucket[5m])) by (le)

# Data format: Heatmap
# Bucket bound: Upper
```

```
  Heatmap showing latency distribution:

  2s  │ ░░░░░░░░░▒▓█████▓▒░░░░░░░░░░
  1s  │ ░░░▒▓████████████████▓▒░░░░░░
  500ms│ ▒▓████████████████████████▓▒░
  200ms│ █████████████████████████████
  100ms│ ▓████████████████████████████
      └──────────────────────────────
       00:00     06:00    12:00   18:00

  Color: Yellow-Red (Oregon)
  ░ = few requests  █ = many requests
```

---

## 4.7 Transformations

Transformations process query results before visualization:

| Transform | Purpose | Example |
|-----------|---------|---------|
| Reduce | Aggregate values per series | Calc mean/max/last |
| Filter by name | Remove series | Hide internal metrics |
| Organize fields | Rename/reorder columns | Clean table display |
| Join by field | Merge queries | Combine metrics from 2 queries |
| Add field from calc | Computed columns | Percentage calculation |
| Sort by | Order rows | Top N by value |
| Group by | Group and aggregate | Sum by category |
| Concatenate | Merge frames | Combine multiple queries |

---

## 4.8 Value Mappings & Overrides

### Value Mappings
Map numeric values to text/colors:
```
0 → "DOWN"  (color: red)
1 → "UP"    (color: green)
2 → "DEGRADED" (color: yellow)

Range: 0-50 → "Low"
Range: 51-80 → "Medium"  
Range: 81-100 → "High"
```

### Field Overrides
Per-series customization:
```
Override 1:
  Match: Fields with name = "errors"
  Properties:
    Color: Red
    Line width: 3
    Fill: 20%

Override 2:
  Match: Fields matching regex /p99/
  Properties:
    Style: Dashed line
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Panel shows "No data" | Query returns empty result | Test in Explore, check time range |
| Graph looks jagged | Resolution too low | Decrease min step interval |
| Heatmap shows nothing | Wrong data format | Ensure histogram buckets with `le` label |
| Colors not matching | Threshold misconfigured | Check threshold values and sort order |
| Table columns wrong order | Default field ordering | Use "Organize fields" transform |

---

## Quick Revision Questions

1. **When would you use a Stat panel versus a Gauge panel?**
2. **How do you visualize latency distributions over time?**
3. **What are transformations and name three common ones?**
4. **How do value mappings work and give an example use case?**
5. **What is the difference between thresholds and field overrides?**
6. **How would you build a "Top 10 endpoints" table panel?**

---

[← Previous: Dashboard Creation](03-dashboard-creation.md) | [Back to README](../README.md) | [Next: Variables & Templating →](05-variables-templating.md)
