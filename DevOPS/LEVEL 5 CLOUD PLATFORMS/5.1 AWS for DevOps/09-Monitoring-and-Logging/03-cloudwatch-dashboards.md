# Chapter 9.3: CloudWatch Dashboards

## Overview

CloudWatch Dashboards provide **customizable visual displays** of your AWS metrics, alarms, and log insights. They help teams monitor the health and performance of applications and infrastructure from a single pane of glass.

---

## Dashboard Architecture

```
┌──── CloudWatch Dashboard ────────────────────────────────┐
│                                                           │
│  ┌─── Production Overview ──────────────────────────────┐│
│  │                                                       ││
│  │  ┌──── EC2 CPU ────┐  ┌──── ALB Requests ──────┐    ││
│  │  │  ████            │  │       ▄▄                │    ││
│  │  │  ████▄▄          │  │    ▄▄████              │    ││
│  │  │  ████████▄       │  │  ▄▄██████▄▄            │    ││
│  │  │  ████████████    │  │  ████████████           │    ││
│  │  └──────────────────┘  └────────────────────────┘    ││
│  │                                                       ││
│  │  ┌──── RDS Connections─┐  ┌──── Alarm Status ────┐   ││
│  │  │  Current: 45/100    │  │  ✓ API Latency: OK   │   ││
│  │  │  ████████▓▓▓▓░░░░░  │  │  ✓ CPU Usage: OK     │   ││
│  │  │                     │  │  ⚠ Disk Space: ALARM │   ││
│  │  └─────────────────────┘  │  ✓ Error Rate: OK    │   ││
│  │                           └──────────────────────┘   ││
│  │  ┌──── Logs Insights Query ──────────────────────┐   ││
│  │  │  Top Errors (last 1 hour):                     │   ││
│  │  │  NullPointerException: 23                      │   ││
│  │  │  TimeoutException: 12                          │   ││
│  │  │  AuthenticationError: 5                        │   ││
│  │  └────────────────────────────────────────────────┘   ││
│  └───────────────────────────────────────────────────────┘│
│                                                           │
│  Features: Auto-refresh, cross-account, shareable link   │
└───────────────────────────────────────────────────────────┘
```

---

## Widget Types

| Widget | Purpose | Data Source |
|--------|---------|-------------|
| **Line** | Time-series trends | Metrics |
| **Stacked area** | Composition over time | Metrics |
| **Number** | Single current value | Metric stat |
| **Gauge** | Value vs threshold | Metric |
| **Bar** | Comparison across dimensions | Metrics |
| **Pie** | Proportional breakdown | Metrics |
| **Text** | Markdown annotations | Static text |
| **Log table** | Query results | Logs Insights |
| **Alarm status** | Alarm state display | Alarms |
| **Explorer** | Dynamic resource discovery | Tag-based metrics |

---

## Creating Dashboards

```bash
# Create dashboard with JSON body
aws cloudwatch put-dashboard \
  --dashboard-name "Production-Overview" \
  --dashboard-body file://dashboard.json
```

### Dashboard JSON Structure

```json
{
  "widgets": [
    {
      "type": "metric",
      "x": 0, "y": 0, "width": 12, "height": 6,
      "properties": {
        "title": "EC2 CPU Utilization",
        "view": "timeSeries",
        "stacked": false,
        "region": "us-east-1",
        "period": 300,
        "stat": "Average",
        "metrics": [
          ["AWS/EC2", "CPUUtilization", "InstanceId", "i-1234567890"],
          ["AWS/EC2", "CPUUtilization", "InstanceId", "i-0987654321"]
        ],
        "yAxis": { "left": { "min": 0, "max": 100 } },
        "annotations": {
          "horizontal": [
            { "label": "Warning", "value": 70, "color": "#ff9900" },
            { "label": "Critical", "value": 90, "color": "#d13212" }
          ]
        }
      }
    },
    {
      "type": "metric",
      "x": 12, "y": 0, "width": 6, "height": 6,
      "properties": {
        "title": "Active Connections",
        "view": "singleValue",
        "metrics": [
          ["AWS/RDS", "DatabaseConnections", "DBInstanceIdentifier", "prod-db"]
        ],
        "period": 60,
        "stat": "Average"
      }
    },
    {
      "type": "alarm",
      "x": 18, "y": 0, "width": 6, "height": 6,
      "properties": {
        "title": "Alarm Status",
        "alarms": [
          "arn:aws:cloudwatch:us-east-1:123456789012:alarm:HighCPU",
          "arn:aws:cloudwatch:us-east-1:123456789012:alarm:HighLatency",
          "arn:aws:cloudwatch:us-east-1:123456789012:alarm:ErrorRate"
        ]
      }
    },
    {
      "type": "log",
      "x": 0, "y": 6, "width": 24, "height": 6,
      "properties": {
        "title": "Recent Errors",
        "query": "SOURCE '/aws/lambda/api-handler'\n| fields @timestamp, @message\n| filter @message like /ERROR/\n| sort @timestamp desc\n| limit 20",
        "region": "us-east-1",
        "view": "table"
      }
    },
    {
      "type": "text",
      "x": 0, "y": 12, "width": 24, "height": 2,
      "properties": {
        "markdown": "## Runbook\n- [High CPU Runbook](https://wiki.example.com/runbook/high-cpu)\n- [Error Rate Runbook](https://wiki.example.com/runbook/errors)\n- **On-call**: DevOps Team (#ops-alerts)"
      }
    }
  ]
}
```

---

## Cross-Account Dashboards

```
┌──── Cross-Account Monitoring ────────────────────────────┐
│                                                          │
│  Monitoring Account (central)                            │
│  ┌──────────────────────────────────────────┐            │
│  │ Dashboard: "All Environments"            │            │
│  │ ┌──── Dev (Acct A) ─┐ ┌── Prod (Acct B)┐│            │
│  │ │ CPU: 25%           │ │ CPU: 65%        ││            │
│  │ │ Errors: 2          │ │ Errors: 0       ││            │
│  │ └───────────────────┘ └────────────────┘││            │
│  └──────────────────────────────────────────┘            │
│                                                          │
│  Setup:                                                  │
│  1. Source accounts share data with monitoring account   │
│  2. Use CloudWatch cross-account observability           │
│  3. Dashboard queries metrics from multiple accounts     │
└──────────────────────────────────────────────────────────┘
```

```bash
# Enable cross-account in monitoring account
aws cloudwatch put-metric-data \
  --namespace "CrossAccount" \
  --metric-name "Test" \
  --value 1

# In dashboard JSON, specify accountId:
# ["AWS/EC2", "CPUUtilization", "InstanceId", "i-123", {"accountId": "222222222222"}]
```

---

## Dashboard Best Practices

```
┌──── Recommended Dashboard Layout ────────────────────────┐
│                                                          │
│  Row 1: High-Level KPIs (Number/Gauge widgets)          │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│  │Requests│ │ P99    │ │ Error  │ │ Active │           │
│  │/sec    │ │Latency │ │ Rate   │ │ Users  │           │
│  │ 1,234  │ │ 45ms   │ │ 0.1%   │ │ 5,678  │           │
│  └────────┘ └────────┘ └────────┘ └────────┘           │
│                                                          │
│  Row 2: Time-Series (Line charts)                       │
│  ┌─────────────────────┐ ┌──────────────────────┐       │
│  │ Request Rate        │ │ Response Time (P50,  │       │
│  │ (last 24 hours)     │ │ P95, P99)            │       │
│  └─────────────────────┘ └──────────────────────┘       │
│                                                          │
│  Row 3: Infrastructure (CPU, Memory, Disk)              │
│  Row 4: Alarms + Logs Insights                          │
│  Row 5: Runbook links (Markdown text widget)            │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Widget shows "No data" | Wrong namespace or dimensions | Verify metric exists with `list-metrics` |
| Dashboard not auto-refreshing | Auto-refresh disabled | Enable auto-refresh (10s, 1m, 5m, etc.) |
| Cross-account data missing | Sharing not configured | Enable cross-account observability in both accounts |
| Slow dashboard load | Too many widgets/metrics | Limit to 50 widgets; reduce time range |
| Can't share dashboard | No shareable link enabled | Enable sharing in dashboard settings |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Dashboards** | Visual displays of metrics, alarms, logs. Customizable. |
| **Widgets** | Line, number, gauge, bar, alarm, log, text (markdown) |
| **Layout** | Grid-based (x, y, width, height). Max 24 columns wide. |
| **Cross-Account** | Monitor multiple accounts from one central dashboard |
| **Auto-Refresh** | 10 sec to 15 min intervals |
| **Cost** | First 3 dashboards free; $3/month per additional dashboard |

---

## Quick Revision Questions

1. **What widget types are available in CloudWatch Dashboards?**
   > Line, stacked area, number, gauge, bar, pie, text (markdown), log table, alarm status, and explorer widgets.

2. **How do you add log query results to a dashboard?**
   > Use a "log" widget type with a Logs Insights query and specify the source log group.

3. **What is the pricing for CloudWatch Dashboards?**
   > First 3 dashboards are free. Each additional dashboard costs $3/month, regardless of the number of widgets.

4. **How can you monitor multiple AWS accounts on one dashboard?**
   > Enable cross-account observability and reference metrics from other accounts using the `accountId` property in widget configuration.

5. **What are horizontal annotations used for?**
   > They draw reference lines on line/area charts to indicate thresholds (e.g., warning at 70%, critical at 90%).

---

[← Previous: 9.2 Metrics & Alarms](02-cloudwatch-metrics-alarms.md) | [Next: 9.4 X-Ray →](04-xray.md)

[← Back to README](../README.md)
