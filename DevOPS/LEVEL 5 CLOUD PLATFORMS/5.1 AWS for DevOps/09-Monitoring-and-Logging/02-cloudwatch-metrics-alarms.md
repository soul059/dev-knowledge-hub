# Chapter 9.2: CloudWatch Metrics and Alarms

## Overview

CloudWatch Metrics collect **numeric time-series data** from AWS resources. Alarms watch metrics and **trigger actions** (SNS notifications, Auto Scaling, EC2 actions) when thresholds are breached. Together, they form the foundation of AWS monitoring.

---

## Metrics Architecture

```
┌──── CloudWatch Metrics ──────────────────────────────────┐
│                                                           │
│  AWS Services (auto)     Custom Metrics (you send)       │
│  ┌──────────────────┐    ┌──────────────────────┐        │
│  │ EC2: CPUUtil,     │    │ App: RequestLatency, │        │
│  │   NetworkIn/Out   │    │   ActiveUsers,       │        │
│  │ RDS: ReadIOPS     │    │   QueueDepth         │        │
│  │ ALB: RequestCount │    │                      │        │
│  │ Lambda: Duration  │    │ Agent: MemoryUsed,   │        │
│  │ S3: BucketSize    │    │   DiskUsed           │        │
│  └────────┬─────────┘    └──────────┬───────────┘        │
│           │                         │                     │
│           ▼                         ▼                     │
│  ┌──────────────────────────────────────────────┐        │
│  │         CloudWatch Metrics Store             │        │
│  │                                              │        │
│  │  Namespace/Metric/Dimensions → Data Points   │        │
│  │                                              │        │
│  │  Retention:                                  │        │
│  │  • 1-sec data → 3 hours                     │        │
│  │  • 1-min data → 15 days                     │        │
│  │  • 5-min data → 63 days                     │        │
│  │  • 1-hour data → 455 days (15 months)       │        │
│  └──────────────────────────────────────────────┘        │
└───────────────────────────────────────────────────────────┘
```

---

## Key EC2 Metrics

```
┌──── Built-in EC2 Metrics (5-min default, 1-min detailed) ┐
│                                                           │
│  CPU:                                                     │
│  ├── CPUUtilization           (percent)                  │
│  ├── CPUCreditUsage           (T-series burstable)       │
│  └── CPUCreditBalance                                    │
│                                                           │
│  Network:                                                 │
│  ├── NetworkIn                (bytes)                    │
│  ├── NetworkOut               (bytes)                    │
│  └── NetworkPacketsIn/Out                                │
│                                                           │
│  Disk (instance store only):                              │
│  ├── DiskReadOps / DiskWriteOps                          │
│  └── DiskReadBytes / DiskWriteBytes                      │
│                                                           │
│  Status:                                                  │
│  ├── StatusCheckFailed_Instance                          │
│  └── StatusCheckFailed_System                            │
│                                                           │
│  ⚠ NOT built-in (need CloudWatch Agent):                │
│  ├── Memory utilization                                  │
│  ├── Disk space utilization                              │
│  └── Process-level metrics                               │
└───────────────────────────────────────────────────────────┘
```

---

## Custom Metrics

```bash
# Publish a custom metric
aws cloudwatch put-metric-data \
  --namespace "MyApp" \
  --metric-name "RequestLatency" \
  --value 145 \
  --unit Milliseconds \
  --dimensions Service=API,Environment=prod

# Publish with high-resolution (1-second)
aws cloudwatch put-metric-data \
  --namespace "MyApp" \
  --metric-name "ActiveConnections" \
  --value 42 \
  --storage-resolution 1

# Publish multiple data points (batch)
aws cloudwatch put-metric-data \
  --namespace "MyApp" \
  --metric-data '[
    {"MetricName": "Errors", "Value": 3, "Unit": "Count",
     "Dimensions": [{"Name": "Service", "Value": "API"}]},
    {"MetricName": "Latency", "Value": 234, "Unit": "Milliseconds",
     "Dimensions": [{"Name": "Service", "Value": "API"}]}
  ]'
```

### Standard vs High-Resolution Metrics

| Feature | Standard | High-Resolution |
|---------|----------|----------------|
| Period | 60 seconds (minimum) | 1 second |
| Cost | Included / lower | Higher |
| Use case | Normal monitoring | Real-time alerting |
| Storage resolution | 1 | 1 |

---

## CloudWatch Alarms

```
┌──── Alarm States ────────────────────────────────────────┐
│                                                          │
│  ┌──────────┐     ┌──────────┐     ┌──────────────┐    │
│  │   OK     │ ←→  │ ALARM    │ ←→  │ INSUFFICIENT │    │
│  │          │     │          │     │ DATA         │    │
│  │ Below    │     │ Above    │     │ Not enough   │    │
│  │ threshold│     │ threshold│     │ data points  │    │
│  └──────────┘     └────┬─────┘     └──────────────┘    │
│                        │                                 │
│                   ┌────▼─────────────────┐               │
│                   │ Actions:             │               │
│                   │ • SNS notification   │               │
│                   │ • Auto Scaling       │               │
│                   │ • EC2 action (stop,  │               │
│                   │   terminate, reboot) │               │
│                   │ • Systems Manager    │               │
│                   └──────────────────────┘               │
└──────────────────────────────────────────────────────────┘
```

### Create Alarms

```bash
# Simple alarm: CPU > 80% for 5 minutes
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPU-WebServer" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:ops-alerts \
  --ok-actions arn:aws:sns:us-east-1:123456789012:ops-resolved \
  --treat-missing-data missing

# ALB target response time > 2 seconds
aws cloudwatch put-metric-alarm \
  --alarm-name "HighLatency-API" \
  --metric-name TargetResponseTime \
  --namespace AWS/ApplicationELB \
  --statistic p99 \
  --period 60 \
  --threshold 2 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 3 \
  --dimensions \
    Name=LoadBalancer,Value=app/my-alb/1234567890 \
    Name=TargetGroup,Value=targetgroup/my-tg/9876543210 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:ops-alerts

# Composite alarm (AND/OR logic)
aws cloudwatch put-composite-alarm \
  --alarm-name "CriticalServiceDown" \
  --alarm-rule 'ALARM("HighCPU-WebServer") AND ALARM("HighLatency-API")' \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:pager-duty
```

---

## treat-missing-data Options

| Option | Behavior |
|--------|----------|
| `missing` | Maintain current alarm state (default) |
| `notBreaching` | Treat as within threshold (good) |
| `breaching` | Treat as exceeding threshold (alarm) |
| `ignore` | Skip evaluation until data returns |

---

## Anomaly Detection

```bash
# Create anomaly detection model
aws cloudwatch put-anomaly-detector \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --stat Average

# Alarm based on anomaly detection band
aws cloudwatch put-metric-alarm \
  --alarm-name "AnomalousCPU" \
  --metrics '[
    {"Id":"m1","MetricStat":{"Metric":{"Namespace":"AWS/EC2",
     "MetricName":"CPUUtilization",
     "Dimensions":[{"Name":"InstanceId","Value":"i-12345"}]},
     "Period":300,"Stat":"Average"}},
    {"Id":"ad1","Expression":"ANOMALY_DETECTION_BAND(m1, 2)"}
  ]' \
  --comparison-operator LessThanLowerOrGreaterThanUpperThreshold \
  --threshold-metric-id ad1 \
  --evaluation-periods 3 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:ops-alerts
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| No metrics for EC2 memory | Not a default metric | Install CloudWatch Agent with memory collection |
| Alarm stays in INSUFFICIENT_DATA | No data points in period | Check metric exists; verify dimensions match |
| Alarm not triggering notifications | SNS subscription not confirmed | Confirm email/endpoint subscription |
| Delayed alarm | Evaluation period too long | Reduce period and evaluation periods |
| Custom metric not appearing | Wrong namespace or dimensions | Verify `put-metric-data` parameters exactly |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Metrics** | Time-series data. Built-in (EC2, RDS) + custom (your app). |
| **Namespaces** | Group metrics: `AWS/EC2`, `AWS/RDS`, or custom `MyApp` |
| **Dimensions** | Key-value pairs to filter: `InstanceId`, `Service` |
| **Alarms** | Watch metrics → trigger SNS, Auto Scaling, EC2 actions |
| **Composite Alarms** | Combine multiple alarms with AND/OR logic |
| **Anomaly Detection** | ML-based bands that adapt to metric patterns |

---

## Quick Revision Questions

1. **What EC2 metrics are NOT available by default?**
   > Memory utilization and disk space utilization. These require the CloudWatch Agent.

2. **What are the three alarm states?**
   > OK (below threshold), ALARM (above threshold), and INSUFFICIENT_DATA (not enough data points).

3. **What actions can an alarm trigger?**
   > SNS notifications, Auto Scaling policies, EC2 actions (stop/terminate/reboot), and Systems Manager automation.

4. **What is a composite alarm?**
   > An alarm that evaluates a combination of other alarms using AND/OR logic, reducing alert noise.

5. **How long are CloudWatch metrics retained?**
   > 1-second data for 3 hours, 1-minute for 15 days, 5-minute for 63 days, 1-hour for 455 days (15 months).

---

[← Previous: 9.1 CloudWatch Logs](01-cloudwatch-logs.md) | [Next: 9.3 CloudWatch Dashboards →](03-cloudwatch-dashboards.md)

[← Back to README](../README.md)
