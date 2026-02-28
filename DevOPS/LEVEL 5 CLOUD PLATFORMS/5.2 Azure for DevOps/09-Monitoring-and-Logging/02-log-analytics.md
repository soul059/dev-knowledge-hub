# Chapter 2: Log Analytics

## Overview
**Log Analytics** is the query and analysis engine within Azure Monitor. All log data is stored in a **Log Analytics Workspace**, and you use **KQL (Kusto Query Language)** to query, filter, aggregate, and visualise data for troubleshooting, security, and reporting.

---

## 2.1 Log Analytics Architecture

```
┌──────────── LOG ANALYTICS ────────────────────┐
│                                                │
│  Data Sources                                  │
│  ┌──────────────────────────────────────────┐  │
│  │  Azure Resources (diagnostic settings)   │  │
│  │  VMs (Azure Monitor Agent / MMA)         │  │
│  │  Application Insights                    │  │
│  │  Microsoft Defender for Cloud            │  │
│  │  Azure AD (sign-in logs)                 │  │
│  │  Custom Logs (REST API / agent)          │  │
│  └──────────────────────────────────────────┘  │
│           │                                    │
│           ▼                                    │
│  ┌──────────────────────────────────────────┐  │
│  │     LOG ANALYTICS WORKSPACE              │  │
│  │                                          │  │
│  │  Tables:                                 │  │
│  │  ├── AzureActivity                       │  │
│  │  ├── Heartbeat                           │  │
│  │  ├── Perf                                │  │
│  │  ├── Syslog                              │  │
│  │  ├── SecurityEvent                       │  │
│  │  ├── AppRequests                         │  │
│  │  ├── AppExceptions                       │  │
│  │  ├── ContainerLog                        │  │
│  │  └── Custom_CL (custom tables)           │  │
│  │                                          │  │
│  │  Query with KQL ─────────────────────►   │  │
│  │                                          │  │
│  └──────────────────────────────────────────┘  │
│           │                                    │
│           ▼                                    │
│  ┌──────────────────────────────────────────┐  │
│  │  Outputs:                                │  │
│  │  ├── Alerts (log-based)                  │  │
│  │  ├── Workbooks (interactive dashboards)  │  │
│  │  ├── Power BI                            │  │
│  │  └── Export to Storage / Event Hub        │  │
│  └──────────────────────────────────────────┘  │
│                                                │
└────────────────────────────────────────────────┘
```

---

## 2.2 KQL (Kusto Query Language) Essentials

### Basic Queries

```kql
// Get last 10 heartbeat records
Heartbeat
| take 10

// Filter by specific computer
Heartbeat
| where Computer == "web-server-01"
| take 10

// Search across all tables
search "error"
| take 50

// Time filter (last 24 hours)
Perf
| where TimeGenerated > ago(24h)
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
```

### Aggregation

```kql
// Average CPU per computer (last 1 hour)
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer
| order by AvgCPU desc

// Count of errors per hour
AppExceptions
| where TimeGenerated > ago(24h)
| summarize ErrorCount = count() by bin(TimeGenerated, 1h)
| render timechart

// Top 5 most common exceptions
AppExceptions
| summarize Count = count() by ExceptionType
| top 5 by Count desc
```

### Joins and Projections

```kql
// Join VM heartbeat with performance data
Heartbeat
| where TimeGenerated > ago(1h)
| distinct Computer, OSType
| join kind=inner (
    Perf
    | where TimeGenerated > ago(1h)
    | where CounterName == "% Processor Time"
    | summarize AvgCPU = avg(CounterValue) by Computer
) on Computer
| project Computer, OSType, AvgCPU

// Extend with calculated column
Perf
| where CounterName == "% Processor Time"
| extend Status = iff(CounterValue > 90, "Critical", "Normal")
```

---

## 2.3 KQL Quick Reference

| Operator | Purpose | Example |
|----------|---------|---------|
| `where` | Filter rows | `| where State == "Active"` |
| `summarize` | Aggregate | `| summarize count() by Category` |
| `project` | Select columns | `| project Name, Value` |
| `extend` | Add calculated column | `| extend Total = A + B` |
| `order by` | Sort results | `| order by Count desc` |
| `top` | Top N rows | `| top 5 by Count desc` |
| `take` / `limit` | Limit rows | `| take 100` |
| `join` | Join tables | `| join kind=inner (T2) on Key` |
| `render` | Visualise | `| render timechart` |
| `bin()` | Time bucketing | `bin(TimeGenerated, 1h)` |
| `ago()` | Relative time | `ago(24h)`, `ago(7d)` |
| `distinct` | Unique values | `| distinct Computer` |
| `parse` | Extract from strings | `| parse Message with * "error:" err` |
| `mv-expand` | Expand arrays | `| mv-expand Tags` |

---

## 2.4 Workspace Setup

```bash
# Create Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group myRG \
  --workspace-name myWorkspace \
  --location uksouth \
  --retention-time 90 \
  --sku PerGB2018

# Get workspace ID (needed for agents)
az monitor log-analytics workspace show \
  --resource-group myRG \
  --workspace-name myWorkspace \
  --query "customerId" -o tsv

# Get workspace key
az monitor log-analytics workspace get-shared-keys \
  --resource-group myRG \
  --workspace-name myWorkspace

# Run a KQL query via CLI
az monitor log-analytics query \
  --workspace "<workspace-id>" \
  --analytics-query "Heartbeat | summarize count() by Computer" \
  --timespan P1D
```

---

## 2.5 Data Collection Rules (DCR)

```
┌────── DATA COLLECTION RULES ─────────────┐
│                                           │
│  DCR defines:                             │
│  ┌────────────────────────────────────┐   │
│  │  What to collect:                 │   │
│  │  ├── Windows Event Logs           │   │
│  │  ├── Syslog (Linux)               │   │
│  │  ├── Performance Counters         │   │
│  │  └── Custom Text Logs             │   │
│  │                                   │   │
│  │  Where to send:                   │   │
│  │  ├── Log Analytics Workspace      │   │
│  │  └── Azure Monitor Metrics        │   │
│  │                                   │   │
│  │  Transform (optional):            │   │
│  │  └── KQL transformation query     │   │
│  │      (filter/reshape at ingest)   │   │
│  └────────────────────────────────────┘   │
│                                           │
│  Replaces legacy agent configuration      │
│  Managed via Azure Monitor Agent (AMA)    │
│                                           │
└───────────────────────────────────────────┘
```

---

## 2.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| No data in workspace | Diagnostic settings or agent not configured | Verify data source is sending to workspace |
| KQL query returns empty | Wrong table name or time range | Check table exists and expand `ago()` range |
| High ingestion cost | Too much verbose data | Use DCR transforms to filter at ingestion |
| Slow queries | Scanning too much data | Add time filters first, use `project` to limit columns |
| Agent not reporting | AMA not installed or DCR not associated | Install Azure Monitor Agent and link DCR |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Log Analytics** | Query engine for Azure Monitor logs |
| **Workspace** | Central store for log data from all sources |
| **KQL** | Query language: where, summarize, join, render |
| **Tables** | Heartbeat, Perf, Syslog, AppRequests, etc. |
| **DCR** | Data Collection Rules define what/where to collect |
| **Retention** | Configurable 30-730 days; archive for longer |

---

## Quick Revision Questions

1. **What is KQL?**
   > Kusto Query Language — a read-only language for querying structured data in Log Analytics. Uses pipe-based operators like `where`, `summarize`, `project`.

2. **What is a Log Analytics Workspace?**
   > A central data store in Azure that collects logs from multiple sources (VMs, apps, Azure resources) for querying and analysis.

3. **How do you filter rows in KQL?**
   > Use the `where` operator: `Heartbeat | where Computer == "web-01" | where TimeGenerated > ago(1h)`

4. **What are Data Collection Rules (DCR)?**
   > Rules that define what data to collect from sources, optional transforms, and which workspace to send data to. Used with Azure Monitor Agent.

5. **How do you reduce Log Analytics costs?**
   > Filter data at ingestion using DCR transforms, exclude verbose log categories, set shorter retention, and archive old data.

6. **What does `summarize count() by bin(TimeGenerated, 1h)` do?**
   > Groups records into 1-hour buckets and counts the number of records in each bucket — useful for trending.

---

[⬅ Previous: Azure Monitor](01-azure-monitor.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Application Insights ➡](03-application-insights.md)
