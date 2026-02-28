# Chapter 9.5: AWS CloudTrail — API Activity Logging

## Overview

AWS CloudTrail records **all API calls** made in your AWS account — who did what, when, and from where. It's essential for security auditing, compliance, governance, and operational troubleshooting. Every action in the AWS Console, CLI, SDK, or service-to-service call is logged.

---

## CloudTrail Architecture

```
┌──── CloudTrail ──────────────────────────────────────────┐
│                                                           │
│  API Calls (all sources)        CloudTrail                │
│  ┌──────────────────┐          ┌──────────────────────┐  │
│  │ AWS Console      │─────────→│                      │  │
│  │ AWS CLI          │─────────→│  Trail               │  │
│  │ AWS SDKs         │─────────→│  ├── Management      │  │
│  │ Service-to-svc   │─────────→│  │   Events (free)   │  │
│  └──────────────────┘          │  ├── Data Events     │  │
│                                │  │   (S3, Lambda)    │  │
│                                │  └── Insights Events │  │
│                                │      (anomalies)     │  │
│                                └──────────┬───────────┘  │
│                                           │               │
│                     ┌─────────────────────┼───────────┐  │
│                     │                     │           │  │
│                     ▼                     ▼           ▼  │
│              ┌────────────┐     ┌──────────┐  ┌───────┐ │
│              │ S3 Bucket  │     │CloudWatch│  │Event- │ │
│              │ (archive)  │     │ Logs     │  │Bridge │ │
│              │ Encrypted  │     │ (query)  │  │(react)│ │
│              └────────────┘     └──────────┘  └───────┘ │
└───────────────────────────────────────────────────────────┘
```

---

## Event Types

```
┌──── CloudTrail Event Types ──────────────────────────────┐
│                                                          │
│  Management Events (Control Plane):                     │
│  ├── CreateBucket, TerminateInstances, CreateUser       │
│  ├── AttachRolePolicy, CreateVPC, RunInstances          │
│  ├── Logged by default (90-day history free)            │
│  └── Read events + Write events                        │
│                                                          │
│  Data Events (Data Plane):                              │
│  ├── S3: GetObject, PutObject, DeleteObject             │
│  ├── Lambda: Invoke                                     │
│  ├── DynamoDB: GetItem, PutItem, Query                  │
│  ├── NOT logged by default (must enable, extra cost)    │
│  └── High volume — use selectors to filter              │
│                                                          │
│  Insights Events:                                       │
│  ├── Detects unusual API call rates                     │
│  ├── e.g., spike in TerminateInstances calls            │
│  └── Uses ML to establish baseline, alert on anomalies  │
└──────────────────────────────────────────────────────────┘
```

---

## Trail Configuration

```bash
# Create a trail (all regions)
aws cloudtrail create-trail \
  --name org-trail \
  --s3-bucket-name my-cloudtrail-logs \
  --is-multi-region-trail \
  --enable-log-file-validation \
  --include-global-service-events \
  --kms-key-id arn:aws:kms:us-east-1:123456789012:key/abc-123

# Start logging
aws cloudtrail start-logging --name org-trail

# Enable data events (S3 object-level)
aws cloudtrail put-event-selectors \
  --trail-name org-trail \
  --advanced-event-selectors '[{
    "Name": "S3DataEvents",
    "FieldSelectors": [
      {"Field": "eventCategory", "Equals": ["Data"]},
      {"Field": "resources.type", "Equals": ["AWS::S3::Object"]},
      {"Field": "resources.ARN", "StartsWith": ["arn:aws:s3:::my-sensitive-bucket/"]}
    ]
  }]'

# Enable Insights
aws cloudtrail put-insight-selectors \
  --trail-name org-trail \
  --insight-selectors '[
    {"InsightType": "ApiCallRateInsight"},
    {"InsightType": "ApiErrorRateInsight"}
  ]'

# Send to CloudWatch Logs
aws cloudtrail update-trail \
  --name org-trail \
  --cloud-watch-logs-log-group-arn arn:aws:logs:us-east-1:123456789012:log-group:CloudTrail:* \
  --cloud-watch-logs-role-arn arn:aws:iam::role/CloudTrailToCloudWatch
```

---

## CloudTrail Event Structure

```json
{
  "eventVersion": "1.09",
  "userIdentity": {
    "type": "AssumedRole",
    "principalId": "AROA123:admin-session",
    "arn": "arn:aws:sts::123456789012:assumed-role/AdminRole/admin-session",
    "accountId": "123456789012",
    "sessionContext": {
      "sessionIssuer": {
        "type": "Role",
        "arn": "arn:aws:iam::123456789012:role/AdminRole"
      }
    }
  },
  "eventTime": "2024-01-15T14:30:00Z",
  "eventSource": "ec2.amazonaws.com",
  "eventName": "TerminateInstances",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "203.0.113.50",
  "userAgent": "aws-cli/2.15.0",
  "requestParameters": {
    "instancesSet": {
      "items": [{"instanceId": "i-1234567890abcdef0"}]
    }
  },
  "responseElements": {
    "instancesSet": {
      "items": [{
        "instanceId": "i-1234567890abcdef0",
        "currentState": {"name": "shutting-down"}
      }]
    }
  },
  "eventID": "abc123-def456-ghi789"
}
```

---

## Querying with CloudTrail Lake

```sql
-- Find who terminated EC2 instances in the last 7 days
SELECT
  eventTime,
  userIdentity.arn,
  sourceIPAddress,
  requestParameters
FROM
  event_data_store_id
WHERE
  eventName = 'TerminateInstances'
  AND eventTime > '2024-01-08 00:00:00'
ORDER BY eventTime DESC

-- Find unauthorized access attempts
SELECT
  eventTime,
  userIdentity.arn,
  eventName,
  errorCode,
  errorMessage
FROM
  event_data_store_id  
WHERE
  errorCode IN ('AccessDenied', 'UnauthorizedAccess')
  AND eventTime > '2024-01-14 00:00:00'
ORDER BY eventTime DESC
```

---

## EventBridge Integration (Real-Time Reactions)

```bash
# Alert on root user login
aws events put-rule \
  --name "root-user-login" \
  --event-pattern '{
    "source": ["aws.signin"],
    "detail-type": ["AWS Console Sign In via CloudTrail"],
    "detail": {
      "userIdentity": {
        "type": ["Root"]
      }
    }
  }'

# Alert on security group changes
aws events put-rule \
  --name "sg-changes" \
  --event-pattern '{
    "source": ["aws.ec2"],
    "detail-type": ["AWS API Call via CloudTrail"],
    "detail": {
      "eventName": [
        "AuthorizeSecurityGroupIngress",
        "RevokeSecurityGroupIngress",
        "AuthorizeSecurityGroupEgress"
      ]
    }
  }'
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Events delayed | CloudTrail delivers within 15 min | Events aren't real-time; use EventBridge for instant alerts |
| S3 data events missing | Data events not enabled | Enable data events with `put-event-selectors` |
| Can't find event | Looking in wrong region | Use multi-region trail or search each region |
| Log file integrity concern | Potential tampering | Enable log file validation; use `validate-logs` CLI |
| High costs | Data events on all S3 buckets | Use advanced event selectors to scope to specific buckets |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **CloudTrail** | Records all API calls. Who, what, when, where. |
| **Management Events** | Control plane (CreateBucket, RunInstances). Free 90-day history. |
| **Data Events** | Data plane (GetObject, Invoke). Must enable. Extra cost. |
| **Insights** | ML-based anomaly detection for unusual API patterns |
| **CloudTrail Lake** | SQL query engine for long-term event analysis |
| **Log Validation** | Cryptographic hash chain to detect file tampering |

---

## Quick Revision Questions

1. **What does CloudTrail record?**
   > Every API call made in your AWS account — console actions, CLI commands, SDK calls, and service-to-service operations.

2. **What is the difference between management and data events?**
   > Management events are control plane operations (creating/deleting resources). Data events are data plane operations (reading/writing objects). Management events are free; data events cost extra.

3. **How quickly are CloudTrail events available?**
   > Within approximately 15 minutes. For real-time alerting, use EventBridge rules that match CloudTrail events.

4. **What is log file validation?**
   > CloudTrail creates a cryptographic digest file for each log delivery. You can verify that log files haven't been tampered with.

5. **How can you detect unusual API activity?**
   > Enable CloudTrail Insights, which uses ML to establish baselines and alert on anomalous API call rates or error rates.

6. **Should trails be multi-region?**
   > Yes. Multi-region trails capture events in all AWS regions, ensuring you don't miss activity in regions you weren't monitoring.

---

[← Previous: 9.4 X-Ray](04-xray.md) | [Next: 10.1 IAM Advanced →](../10-Security/01-iam-advanced-policies-roles.md)

[← Back to README](../README.md)
