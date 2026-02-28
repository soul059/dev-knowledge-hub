# Chapter 1: Azure Monitor

## Overview
**Azure Monitor** is a comprehensive monitoring platform that collects, analyses, and acts on telemetry from Azure, on-premises, and multi-cloud resources. It provides metrics, logs, alerts, dashboards, and autoscale — the central hub for all observability in Azure.

---

## 1.1 Azure Monitor Architecture

```
┌──────────────── AZURE MONITOR ──────────────────────┐
│                                                      │
│  DATA SOURCES                                        │
│  ┌──────────────────────────────────────────────┐    │
│  │  Applications (App Insights SDK)             │    │
│  │  Operating Systems (agents)                  │    │
│  │  Azure Resources (platform metrics/logs)     │    │
│  │  Azure Subscriptions (Activity Log)          │    │
│  │  Azure AD (sign-in/audit logs)               │    │
│  │  Custom Sources (REST API, agents)           │    │
│  └──────────────────────────────────────────────┘    │
│           │                          │               │
│           ▼                          ▼               │
│  ┌─────────────────┐    ┌──────────────────────┐    │
│  │   METRICS        │    │      LOGS            │    │
│  │  (Numeric data)  │    │  (Structured text)   │    │
│  │  Near real-time  │    │  Log Analytics       │    │
│  │  Time-series DB  │    │  Workspace (KQL)     │    │
│  └────────┬────────┘    └──────────┬───────────┘    │
│           │                        │                 │
│           ▼                        ▼                 │
│  ┌──────────────────────────────────────────────┐    │
│  │              CONSUME / ACT                   │    │
│  │  ├── Alerts (metric, log, activity log)      │    │
│  │  ├── Dashboards (Azure Portal)               │    │
│  │  ├── Workbooks (interactive reports)          │    │
│  │  ├── Autoscale                               │    │
│  │  ├── Power BI / Grafana                      │    │
│  │  └── Export (Event Hub, Storage, API)         │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 1.2 Metrics vs Logs

| Aspect | Metrics | Logs |
|--------|---------|------|
| **Data type** | Numeric time-series | Structured text records |
| **Latency** | Near real-time (1 min) | Minutes (ingestion delay) |
| **Storage** | Time-series database | Log Analytics Workspace |
| **Query** | Metrics Explorer | KQL (Kusto Query Language) |
| **Retention** | 93 days (platform) | Configurable (30-730 days) |
| **Cost** | Included with resources | Pay per GB ingested |
| **Use case** | Dashboards, autoscale, fast alerts | Deep analysis, correlation, troubleshooting |

---

## 1.3 Key Metrics Categories

```
┌────── COMMON AZURE METRICS ──────────────┐
│                                           │
│  VM Metrics:                              │
│  ├── Percentage CPU                       │
│  ├── Available Memory Bytes               │
│  ├── Disk Read/Write Bytes                │
│  └── Network In/Out                       │
│                                           │
│  App Service Metrics:                     │
│  ├── Http Server Errors (5xx)             │
│  ├── Response Time                        │
│  ├── Requests                             │
│  └── CPU Percentage                       │
│                                           │
│  SQL Database Metrics:                    │
│  ├── DTU Percentage                       │
│  ├── CPU Percentage                       │
│  ├── Data IO Percentage                   │
│  └── Deadlocks                            │
│                                           │
│  Storage Metrics:                         │
│  ├── Transactions                         │
│  ├── Ingress/Egress                       │
│  ├── Availability                         │
│  └── Latency (E2E, Server)               │
│                                           │
└───────────────────────────────────────────┘
```

---

## 1.4 Azure Monitor CLI Commands

```bash
# List available metrics for a resource
az monitor metrics list-definitions \
  --resource "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM"

# Get CPU metric for a VM (last 1 hour)
az monitor metrics list \
  --resource "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM" \
  --metric "Percentage CPU" \
  --interval PT1H

# Enable diagnostic settings (send logs to Log Analytics)
az monitor diagnostic-settings create \
  --resource "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Web/sites/myapp" \
  --name "send-to-law" \
  --workspace "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/myWorkspace" \
  --logs '[{"category":"AppServiceHTTPLogs","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'

# List Activity Log events
az monitor activity-log list \
  --resource-group myRG \
  --start-time 2024-01-01T00:00:00Z \
  --offset 7d
```

---

## 1.5 Diagnostic Settings

```
┌────── DIAGNOSTIC SETTINGS ────────────────┐
│                                            │
│  Azure Resource (e.g., App Service)        │
│       │                                    │
│       ├──► Log Analytics Workspace         │
│       │    (query with KQL)                │
│       │                                    │
│       ├──► Storage Account                 │
│       │    (long-term archive)             │
│       │                                    │
│       └──► Event Hub                       │
│            (stream to SIEM / Splunk)       │
│                                            │
│  You can send to multiple destinations     │
│  simultaneously.                           │
│                                            │
└────────────────────────────────────────────┘
```

---

## 1.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| No metrics showing | Resource just created | Wait 5-10 minutes for data to appear |
| Logs not arriving | Diagnostic settings not configured | Enable diagnostic settings for the resource |
| High Log Analytics cost | Too much data ingested | Use data collection rules, filter out verbose logs |
| Metrics gaps | Resource stopped or deallocated | Metrics only emit when resource is running |
| Can't query logs | Wrong workspace selected | Verify Log Analytics workspace scope |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Azure Monitor** | Central platform for metrics, logs, alerts, dashboards |
| **Metrics** | Numeric time-series, near real-time, 93-day retention |
| **Logs** | Structured records in Log Analytics, queried with KQL |
| **Diagnostic Settings** | Route logs/metrics to Workspace, Storage, Event Hub |
| **Activity Log** | Subscription-level events (who did what, when) |
| **Data Sources** | Apps, VMs, Azure resources, subscriptions, Azure AD |

---

## Quick Revision Questions

1. **What is the difference between Azure Monitor metrics and logs?**
   > Metrics are numeric time-series data (fast, 93-day retention). Logs are structured text stored in Log Analytics (queried with KQL, configurable retention).

2. **What are diagnostic settings?**
   > Configuration that routes platform logs and metrics from Azure resources to destinations like Log Analytics, Storage Accounts, or Event Hubs.

3. **What is the Activity Log?**
   > A subscription-level log recording control-plane operations (e.g., who created/deleted a resource, when, from where).

4. **What query language does Log Analytics use?**
   > KQL (Kusto Query Language) — a read-only language for exploring and analysing log data.

5. **How do you reduce Azure Monitor costs?**
   > Use data collection rules to filter ingested data, set appropriate retention periods, and archive to cheaper storage tiers.

---

[⬅ Previous: YAML Pipelines](../08-Azure-DevOps-Services/06-yaml-pipelines.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Log Analytics ➡](02-log-analytics.md)
