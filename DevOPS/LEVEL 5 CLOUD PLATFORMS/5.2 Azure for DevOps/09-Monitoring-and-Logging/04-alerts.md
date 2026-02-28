# Chapter 4: Alerts

## Overview
**Azure Monitor Alerts** proactively notify you when conditions are met in your metrics, logs, or activity log. Alerts fire **action groups** that can send emails, SMS, call webhooks, trigger Logic Apps, or create ITSM tickets.

---

## 4.1 Alert Architecture

```
┌──────────────── ALERT SYSTEM ────────────────────┐
│                                                    │
│  Signal Sources                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │  Metrics (e.g., CPU > 90%)                   │  │
│  │  Logs (KQL query returns results)            │  │
│  │  Activity Log (resource deleted)             │  │
│  │  Resource Health (VM unavailable)            │  │
│  │  Service Health (Azure outage)               │  │
│  └──────────────────────────────────────────────┘  │
│           │                                        │
│           ▼                                        │
│  ┌──────────────────────────────────────────────┐  │
│  │         ALERT RULE                           │  │
│  │  ┌────────────────────────────────────────┐  │  │
│  │  │  Target: myapp (App Service)           │  │  │
│  │  │  Signal: Http5xx                       │  │  │
│  │  │  Condition: Count > 10 in 5 min        │  │  │
│  │  │  Severity: Sev 1 (Error)               │  │  │
│  │  └────────────────────────────────────────┘  │  │
│  │                                              │  │
│  │  Evaluation Period: every 1 min              │  │
│  └──────────────────────────────────────────────┘  │
│           │ (condition met)                        │
│           ▼                                        │
│  ┌──────────────────────────────────────────────┐  │
│  │         ACTION GROUP                         │  │
│  │  ├── Email: ops-team@company.com             │  │
│  │  ├── SMS: +44 7xxx xxx xxx                   │  │
│  │  ├── Webhook: https://slack.com/webhook      │  │
│  │  ├── Azure Function: auto-scale-handler      │  │
│  │  ├── Logic App: create-ticket                │  │
│  │  └── ITSM: ServiceNow integration            │  │
│  └──────────────────────────────────────────────┘  │
│           │                                        │
│           ▼                                        │
│  ┌──────────────────────────────────────────────┐  │
│  │  Alert States:                               │  │
│  │  🔴 Fired → 🟡 Acknowledged → 🟢 Closed     │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 4.2 Alert Types

| Alert Type | Signal | Evaluation | Use Case |
|-----------|--------|------------|----------|
| **Metric Alert** | Numeric metric | Every 1-15 min | CPU > 90%, Response time > 5s |
| **Log Alert** | KQL query result | Every 5-15 min | Error count > 10, exception patterns |
| **Activity Log Alert** | Control plane event | Real-time | Resource deleted, policy violation |
| **Resource Health Alert** | Resource availability | Real-time | VM became unavailable |
| **Service Health Alert** | Azure service status | Real-time | Azure outage in your region |
| **Smart Detection** | Anomaly (App Insights) | Automatic | Failure rate spike, performance degradation |

---

## 4.3 Severity Levels

| Severity | Name | Description |
|----------|------|-------------|
| **Sev 0** | Critical | Business-critical system is down |
| **Sev 1** | Error | Significant problem affecting users |
| **Sev 2** | Warning | Potential issue, needs attention |
| **Sev 3** | Informational | Noteworthy but not urgent |
| **Sev 4** | Verbose | For debugging / low priority |

---

## 4.4 Creating Alerts via CLI

```bash
# Create Action Group
az monitor action-group create \
  --resource-group myRG \
  --name "ops-team" \
  --short-name "OpsTeam" \
  --action email ops-email ops@company.com \
  --action sms ops-sms 44 7123456789 \
  --action webhook slack-hook "https://hooks.slack.com/services/xxx"

# Create Metric Alert (CPU > 90% for 5 minutes)
az monitor metrics alert create \
  --resource-group myRG \
  --name "high-cpu-alert" \
  --scopes "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM" \
  --condition "avg Percentage CPU > 90" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --severity 2 \
  --action "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Insights/actionGroups/ops-team" \
  --description "CPU exceeds 90% for 5 minutes"

# Create Log Alert (more than 10 errors in 5 minutes)
az monitor scheduled-query create \
  --resource-group myRG \
  --name "error-spike-alert" \
  --scopes "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/myWorkspace" \
  --condition "count > 10" \
  --condition-query "AppExceptions | where severityLevel >= 3" \
  --window-size 5m \
  --evaluation-frequency 5m \
  --severity 1 \
  --action "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Insights/actionGroups/ops-team"

# Create Activity Log Alert (resource deleted)
az monitor activity-log alert create \
  --resource-group myRG \
  --name "resource-deleted-alert" \
  --condition category=Administrative and operationName=Microsoft.Resources/subscriptions/resourceGroups/delete \
  --action-group "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Insights/actionGroups/ops-team"
```

---

## 4.5 Alert Processing Rules

```
┌────── ALERT PROCESSING RULES ─────────────┐
│                                            │
│  Suppress alerts during maintenance:       │
│  ┌──────────────────────────────────────┐  │
│  │  Rule: "Weekend Maintenance"        │  │
│  │  Scope: Resource Group "production" │  │
│  │  Schedule: Sat 02:00 - Sat 06:00    │  │
│  │  Action: Suppress notifications     │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Add action group to all alerts:           │
│  ┌──────────────────────────────────────┐  │
│  │  Rule: "Notify Security Team"       │  │
│  │  Scope: All Sev 0 + Sev 1 alerts   │  │
│  │  Action: Add action group "security"│  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

---

## 4.6 Real-World Alert Strategy

```
┌────── RECOMMENDED ALERTS ─────────────────┐
│                                            │
│  Infrastructure:                           │
│  ├── VM CPU > 90% (5 min) → Sev 2        │
│  ├── VM Disk > 95% → Sev 1               │
│  ├── VM Unavailable → Sev 0              │
│  └── VM Memory < 10% free → Sev 2        │
│                                            │
│  Application:                              │
│  ├── 5xx errors > 10/5min → Sev 1        │
│  ├── Response time > 5s (avg) → Sev 2    │
│  ├── Exception spike → Smart Detection   │
│  └── Availability test fails → Sev 0     │
│                                            │
│  Database:                                 │
│  ├── DTU > 85% → Sev 2                   │
│  ├── Deadlocks > 5/hr → Sev 2            │
│  └── Connection failures → Sev 1         │
│                                            │
│  Security:                                 │
│  ├── Resource deleted → Sev 1             │
│  ├── Role assignment changed → Sev 2      │
│  └── Policy non-compliance → Sev 2        │
│                                            │
└────────────────────────────────────────────┘
```

---

## 4.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Alert never fires | Threshold too high or wrong metric | Test with lower threshold, verify metric in Explorer |
| Too many alerts (noise) | Threshold too low or missing suppression | Tune thresholds, use alert processing rules |
| Email not received | Action group misconfigured | Test action group, check spam folder |
| Alert stuck in "Fired" | Condition still true (stateful alert) | Resolve underlying issue or auto-resolve setting |
| Webhook not triggered | URL unreachable or payload mismatch | Test webhook URL, check action group logs |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Alert Types** | Metric, Log, Activity Log, Resource Health, Service Health |
| **Action Groups** | Email, SMS, Webhook, Function, Logic App, ITSM |
| **Severity** | Sev 0 (Critical) → Sev 4 (Verbose) |
| **Metric Alerts** | Near-real-time, evaluate every 1-15 min |
| **Log Alerts** | KQL-based, evaluate every 5-15 min |
| **Processing Rules** | Suppress during maintenance, add extra action groups |

---

## Quick Revision Questions

1. **What are the three main types of Azure Monitor alerts?**
   > Metric alerts (numeric thresholds), Log alerts (KQL query results), and Activity Log alerts (control-plane events).

2. **What is an Action Group?**
   > A collection of notification actions (email, SMS, webhook, Azure Function, Logic App) triggered when an alert fires.

3. **What is the difference between metric and log alerts?**
   > Metric alerts evaluate numeric time-series data near real-time. Log alerts evaluate KQL queries against log data every 5-15 minutes.

4. **What are alert processing rules?**
   > Rules that modify alert behaviour — suppress notifications during maintenance windows or add additional action groups to alerts.

5. **What severity levels exist in Azure Monitor?**
   > Sev 0 (Critical), Sev 1 (Error), Sev 2 (Warning), Sev 3 (Informational), Sev 4 (Verbose).

---

[⬅ Previous: Application Insights](03-application-insights.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Dashboards ➡](05-dashboards.md)
