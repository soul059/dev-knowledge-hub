# Chapter 5: Dashboards and Workbooks

## Overview
**Azure Dashboards** provide customisable portal views with pinned charts, metrics, and resource summaries. **Azure Workbooks** offer richer, interactive reports combining data from metrics, logs, and other sources with parameters, filters, and visualisations.

---

## 5.1 Dashboards vs Workbooks

```
┌────── DASHBOARDS vs WORKBOOKS ─────────────┐
│                                             │
│  AZURE DASHBOARDS:                          │
│  ┌───────────────────────────────────────┐  │
│  │  • Portal-based tile layout           │  │
│  │  • Pin charts from Metrics Explorer   │  │
│  │  • Pin query results from Log Analyt. │  │
│  │  • Resource summaries, links          │  │
│  │  • Shared or private                  │  │
│  │  • Quick overview at a glance         │  │
│  │  • Limited interactivity              │  │
│  └───────────────────────────────────────┘  │
│                                             │
│  AZURE WORKBOOKS:                           │
│  ┌───────────────────────────────────────┐  │
│  │  • Rich interactive reports           │  │
│  │  • Parameters / dropdowns / filters   │  │
│  │  • Mix text, KQL queries, metrics     │  │
│  │  • Conditional formatting             │  │
│  │  • Templates (pre-built + custom)     │  │
│  │  • Drill-down and linking             │  │
│  │  • Better for investigation & reports │  │
│  └───────────────────────────────────────┘  │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 5.2 Dashboard Layout

```
┌────── AZURE DASHBOARD ──────────────────────────────┐
│                                                      │
│  ┌────────────────┐  ┌────────────────┐  ┌────────┐ │
│  │  VM CPU Chart  │  │  App Response  │  │ Active │ │
│  │  ████████░░░   │  │    Time        │  │ Alerts │ │
│  │  ████░░░░░░░   │  │  ┌──────┐     │  │  🔴 2  │ │
│  │  avg: 45%      │  │  │ 230ms│     │  │  🟡 5  │ │
│  └────────────────┘  │  └──────┘     │  │  🟢 12 │ │
│                       └────────────────┘  └────────┘ │
│  ┌────────────────┐  ┌─────────────────────────────┐ │
│  │  Error Rate    │  │  Resource Health Summary     │ │
│  │  ──*──*──*──   │  │  ✅ Web App: Healthy        │ │
│  │     (0.2%)     │  │  ✅ SQL DB: Healthy         │ │
│  │                │  │  ⚠️ Redis: Degraded         │ │
│  └────────────────┘  │  ✅ Storage: Healthy        │ │
│                       └─────────────────────────────┘ │
│  ┌──────────────────────────────────────────────────┐ │
│  │  KQL Query: Top Errors (last 24h)               │ │
│  │  ┌──────────────┬──────────┬──────────────────┐ │ │
│  │  │ Exception    │ Count    │ Last Seen        │ │ │
│  │  ├──────────────┼──────────┼──────────────────┤ │ │
│  │  │ NullRef      │ 42       │ 5 min ago        │ │ │
│  │  │ Timeout      │ 15       │ 12 min ago       │ │ │
│  │  │ Auth Failed  │ 8        │ 1 hr ago         │ │ │
│  │  └──────────────┴──────────┴──────────────────┘ │ │
│  └──────────────────────────────────────────────────┘ │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 5.3 Creating Dashboards

```bash
# Create a shared dashboard via CLI
az portal dashboard create \
  --resource-group myRG \
  --name "ops-dashboard" \
  --input-path dashboard-template.json \
  --location uksouth

# Export existing dashboard to JSON
az portal dashboard show \
  --resource-group myRG \
  --name "ops-dashboard" > dashboard-backup.json
```

### Pinning Charts

```
Steps to pin a chart to dashboard:
1. Go to Azure Monitor → Metrics
2. Select resource and metric (e.g., CPU %)
3. Configure chart (time range, aggregation)
4. Click "Pin to dashboard" 📌
5. Choose existing or new dashboard
```

---

## 5.4 Workbook Structure

```
┌────── WORKBOOK COMPONENTS ────────────────┐
│                                            │
│  Parameters:                               │
│  ┌──────────────────────────────────────┐  │
│  │  [Subscription ▼] [Resource Group ▼] │  │
│  │  [Time Range ▼]   [Environment ▼]    │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Section 1: Overview                       │
│  ┌──────────────────────────────────────┐  │
│  │  Markdown text explaining the report │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Section 2: KQL Query (Grid)               │
│  ┌──────────────────────────────────────┐  │
│  │  requests                           │  │
│  │  | where timestamp > {TimeRange}     │  │
│  │  | summarize count() by name         │  │
│  │  → Renders as interactive table     │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Section 3: Metrics Chart                  │
│  ┌──────────────────────────────────────┐  │
│  │  CPU % over time (line chart)       │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Section 4: Conditional Formatting         │
│  ┌──────────────────────────────────────┐  │
│  │  Green: CPU < 70%                   │  │
│  │  Yellow: CPU 70-90%                  │  │
│  │  Red: CPU > 90%                     │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

---

## 5.5 Workbook KQL Examples

```kql
// Workbook parameter usage
// {TimeRange} and {Subscription} are parameter references

// Request performance by endpoint
requests
| where timestamp {TimeRange}
| summarize 
    AvgDuration = avg(duration),
    p95Duration = percentile(duration, 95),
    FailRate = countif(success == false) * 100.0 / count(),
    TotalRequests = count()
  by name
| order by TotalRequests desc

// VM health summary for workbook grid
Heartbeat
| where TimeGenerated {TimeRange}
| summarize LastHeartbeat = max(TimeGenerated) by Computer, OSType
| extend Status = iff(LastHeartbeat < ago(5m), "🔴 Offline", "🟢 Online")
| project Computer, OSType, Status, LastHeartbeat
```

---

## 5.6 Built-in Workbook Templates

| Template | Purpose |
|----------|---------|
| **VM Insights** | CPU, memory, disk, network for all VMs |
| **Key Vault Insights** | Operations, latency, failures |
| **Storage Insights** | Capacity, transactions, latency |
| **Container Insights** | AKS node and pod performance |
| **Network Insights** | Connectivity, traffic, NSG flow |
| **Failure Analysis** | App Insights exception analysis |

---

## 5.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Dashboard loads slowly | Too many tiles or wide time range | Reduce tiles or narrow time range |
| Pinned chart shows no data | Resource moved or metric discontinued | Re-pin from Metrics Explorer |
| Workbook parameter not working | Wrong parameter syntax `{param}` | Verify parameter name matches reference |
| Can't share dashboard | Permissions not set | Share via RBAC (Reader role on dashboard) |
| Workbook query timeout | KQL too complex or scans too much data | Add time filter first, use `project` to limit |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Dashboards** | Portal tile layout, pinned charts, quick overview |
| **Workbooks** | Rich interactive reports with parameters and drill-down |
| **Pinning** | Pin Metrics Explorer charts or Log queries to dashboards |
| **Parameters** | Dropdown filters in Workbooks (time range, subscription, etc.) |
| **Templates** | Pre-built Workbooks for VMs, Storage, AKS, Network |
| **Sharing** | Dashboards shared via RBAC; Workbooks saved in resource groups |

---

## Quick Revision Questions

1. **What is the difference between Azure Dashboards and Workbooks?**
   > Dashboards are simple portal tile layouts for quick overview. Workbooks are rich, interactive reports with parameters, conditional formatting, and drill-down.

2. **How do you add a chart to a dashboard?**
   > Open Metrics Explorer, configure the chart, then click "Pin to dashboard" to pin it to an existing or new dashboard.

3. **What are Workbook parameters?**
   > Interactive filters (dropdowns, time pickers) that users can change to dynamically update all queries and visualisations in the workbook.

4. **Name three built-in Workbook templates.**
   > VM Insights (compute performance), Container Insights (AKS monitoring), and Key Vault Insights (operations and failures).

5. **How do you share a dashboard with your team?**
   > Publish the dashboard as a shared dashboard and grant team members Reader (or higher) RBAC role on the dashboard resource.

---

[⬅ Previous: Alerts](04-alerts.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Diagnostics ➡](06-diagnostics.md)
