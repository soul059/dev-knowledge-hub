# Chapter 9.1: Amazon CloudWatch Logs

## Overview

Amazon CloudWatch Logs enables you to **centralize, monitor, and analyze logs** from AWS services, applications, and on-premises servers. It's the primary logging service in AWS, integrating natively with Lambda, ECS, EC2, API Gateway, and dozens of other services.

---

## CloudWatch Logs Architecture

```
┌──── CloudWatch Logs ─────────────────────────────────────┐
│                                                           │
│  Sources                    CloudWatch Logs              │
│  ┌──────────────┐          ┌──────────────────────────┐  │
│  │ Lambda       │─────────→│                          │  │
│  │ ECS/Fargate  │─────────→│  Log Group               │  │
│  │ EC2 (agent)  │─────────→│  └── Log Stream 1        │  │
│  │ API Gateway  │─────────→│      └── Log Events      │  │
│  │ VPC Flow Logs│─────────→│  └── Log Stream 2        │  │
│  │ CloudTrail   │─────────→│      └── Log Events      │  │
│  │ RDS          │─────────→│  └── Log Stream N        │  │
│  │ Custom Apps  │─────────→│      └── Log Events      │  │
│  └──────────────┘          └───────────┬──────────────┘  │
│                                        │                  │
│  Destinations:                         │                  │
│  ┌─────────────────────────────────────┘                  │
│  │                                                        │
│  ▼                                                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │ S3       │  │Kinesis   │  │Lambda    │  │OpenSearch│ │
│  │ Export   │  │Firehose  │  │(process) │  │(analyze) │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
└───────────────────────────────────────────────────────────┘
```

---

## Core Concepts

```
┌──── Hierarchy ───────────────────────────────────────────┐
│                                                          │
│  Log Group:  /aws/lambda/my-function                    │
│  ├── Retention: 30 days                                  │
│  ├── KMS Encryption: optional                           │
│  │                                                       │
│  ├── Log Stream: 2024/01/15/[$LATEST]abc123             │
│  │   ├── Log Event: [timestamp] START RequestId: ...    │
│  │   ├── Log Event: [timestamp] {"level":"info",...}    │
│  │   └── Log Event: [timestamp] END RequestId: ...     │
│  │                                                       │
│  └── Log Stream: 2024/01/15/[$LATEST]def456             │
│      └── Log Events...                                   │
│                                                          │
│  Log Group  = Container (retention, permissions)        │
│  Log Stream = Sequence of events from one source        │
│  Log Event  = Single log entry (timestamp + message)    │
└──────────────────────────────────────────────────────────┘
```

---

## Log Group Management

```bash
# Create log group
aws logs create-log-group \
  --log-group-name /myapp/production/api

# Set retention policy
aws logs put-retention-policy \
  --log-group-name /myapp/production/api \
  --retention-in-days 30

# Available retention values:
# 1, 3, 5, 7, 14, 30, 60, 90, 120, 150, 180, 365,
# 400, 545, 731, 1096, 1827, 2192, 2557, 2922, 3283, 3653
# Or: never expire (default)

# Delete log group
aws logs delete-log-group \
  --log-group-name /myapp/production/api

# List log groups
aws logs describe-log-groups --log-group-name-prefix /myapp/
```

---

## CloudWatch Logs Agent (EC2)

```bash
# Install unified CloudWatch agent on EC2
sudo yum install -y amazon-cloudwatch-agent

# Configure agent (wizard)
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

```json
// /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/myapp/application.log",
            "log_group_name": "/myapp/production/api",
            "log_stream_name": "{instance_id}/application",
            "timezone": "UTC",
            "multi_line_start_pattern": "^\\d{4}-\\d{2}-\\d{2}"
          },
          {
            "file_path": "/var/log/myapp/error.log",
            "log_group_name": "/myapp/production/errors",
            "log_stream_name": "{instance_id}/errors"
          }
        ]
      }
    }
  }
}
```

```bash
# Start the agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
  -s
```

---

## CloudWatch Logs Insights (Query Language)

```sql
-- Find the 20 most recent errors
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20

-- Count errors per hour
fields @timestamp
| filter @message like /ERROR/
| stats count(*) as errorCount by bin(1h)

-- Parse JSON log and filter
fields @timestamp, @message
| parse @message '{"level":"*","service":"*","msg":"*"}' as level, service, msg
| filter level = "error"
| stats count(*) by service

-- P99 latency from structured logs
fields @timestamp, @message
| parse @message '"duration":*,' as duration
| stats avg(duration), pct(duration, 95), pct(duration, 99), max(duration)

-- Find Lambda cold starts
fields @timestamp, @type, @billedDuration, @initDuration
| filter ispresent(@initDuration)
| sort @initDuration desc
| limit 20

-- Top 10 most expensive Lambda invocations
fields @timestamp, @billedDuration, @memorySize, @maxMemoryUsed
| sort @billedDuration desc
| limit 10
```

---

## Metric Filters

```bash
# Create metric filter — count ERROR occurrences
aws logs put-metric-filter \
  --log-group-name /myapp/production/api \
  --filter-name ErrorCount \
  --filter-pattern "ERROR" \
  --metric-transformations \
    metricName=ApplicationErrors,metricNamespace=MyApp,metricValue=1

# Pattern syntax examples:
# "ERROR"                  — exact word match
# [ip, user, ..., status_code = 5*]  — space-delimited with wildcard
# { $.level = "error" }     — JSON filter
# { $.statusCode >= 500 }   — JSON numeric comparison
# { $.latency > 1000 && $.method = "POST" }  — compound JSON filter
```

---

## Subscription Filters (Real-Time Streaming)

```bash
# Stream logs to Lambda for processing
aws logs put-subscription-filter \
  --log-group-name /myapp/production/api \
  --filter-name ErrorStream \
  --filter-pattern "ERROR" \
  --destination-arn arn:aws:lambda:us-east-1:123456789012:function:log-processor

# Stream to Kinesis Data Firehose → S3/OpenSearch
aws logs put-subscription-filter \
  --log-group-name /myapp/production/api \
  --filter-name AllLogs \
  --filter-pattern "" \
  --destination-arn arn:aws:firehose:us-east-1:123456789012:deliverystream/log-stream \
  --role-arn arn:aws:iam::role/CWLogsToFirehoseRole

# Limit: 2 subscription filters per log group
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Logs not appearing | Missing IAM permissions | Ensure `logs:CreateLogGroup`, `logs:PutLogEvents` on role |
| High costs | No retention policy (never expire) | Set retention to 30, 60, or 90 days |
| Agent not sending logs | Agent not running or misconfigured | Check agent status; verify config JSON and file paths |
| Insights query slow | Scanning too many log groups | Narrow time range; query specific log groups |
| Metric filter not counting | Wrong filter pattern syntax | Test pattern with `filter-log-events` CLI first |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Log Groups** | Container for log streams. Set retention, encryption. |
| **Log Streams** | Sequence of events from one source (instance, container) |
| **Logs Insights** | SQL-like query language for searching and analyzing logs |
| **Metric Filters** | Convert log patterns to CloudWatch metrics |
| **Subscription Filters** | Real-time streaming to Lambda, Kinesis, OpenSearch |
| **Agent** | Unified CloudWatch agent for EC2/on-premises log collection |

---

## Quick Revision Questions

1. **What is the relationship between log groups and log streams?**
   > A log group contains multiple log streams. Each stream is a sequence of events from a single source (e.g., one EC2 instance or Lambda invocation).

2. **How do you control log storage costs?**
   > Set a retention policy on log groups (e.g., 30 or 90 days). Without a policy, logs are stored indefinitely.

3. **What are metric filters used for?**
   > They scan incoming log data for specific patterns and publish matching occurrences as CloudWatch custom metrics, which can trigger alarms.

4. **How many subscription filters can a log group have?**
   > A maximum of 2 subscription filters per log group.

5. **What is CloudWatch Logs Insights?**
   > A purpose-built query language for searching, parsing, and visualizing log data interactively with SQL-like syntax.

---

[← Previous: 8.4 Terraform](../08-Infrastructure-as-Code/04-terraform-with-aws.md) | [Next: 9.2 CloudWatch Metrics & Alarms →](02-cloudwatch-metrics-alarms.md)

[← Back to README](../README.md)
