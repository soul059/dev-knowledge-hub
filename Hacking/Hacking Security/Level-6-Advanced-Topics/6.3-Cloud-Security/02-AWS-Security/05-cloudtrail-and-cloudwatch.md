# Unit 2: AWS Security — Topic 5: CloudTrail and CloudWatch

## Overview

**AWS CloudTrail** and **CloudWatch** are the foundation of AWS security monitoring. CloudTrail records every API call made in your AWS account (who did what, when, and from where), while CloudWatch provides metrics, logs, and alarms for AWS resources. Together, they enable threat detection, incident investigation, compliance auditing, and operational monitoring.

---

## 1. AWS CloudTrail

```
CLOUDTRAIL — API ACTIVITY LOGGING:

WHAT IT RECORDS:
  → Every AWS API call
  → Who made the call (identity)
  → When it was made (timestamp)
  → What was called (API action)
  → What was the result (success/failure)
  → Source IP address
  → User agent
  → Request parameters
  → Response elements

EVENT TYPES:
  Management Events:
  → Control plane operations
  → CreateUser, RunInstances, CreateBucket
  → Enabled by default in CloudTrail
  
  Data Events:
  → Data plane operations
  → S3 GetObject/PutObject
  → Lambda function invocations
  → Must be explicitly enabled
  → Higher volume, additional cost
  
  Insights Events:
  → Unusual API activity detection
  → Anomalous call rates
  → Unusual error rates
  → Machine learning-based

EXAMPLE LOG ENTRY:
{
  "eventVersion": "1.08",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAEXAMPLE",
    "arn": "arn:aws:iam::123456789012:user/admin",
    "accountId": "123456789012",
    "userName": "admin",
    "accessKeyId": "AKIAEXAMPLE"
  },
  "eventTime": "2024-01-15T10:30:00Z",
  "eventSource": "s3.amazonaws.com",
  "eventName": "PutBucketPolicy",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "198.51.100.50",
  "userAgent": "aws-cli/2.0",
  "requestParameters": {
    "bucketName": "sensitive-data-bucket",
    "policy": "..."
  },
  "responseElements": null,
  "errorCode": null,
  "errorMessage": null
}

CONFIGURATION:
  # Create trail (all regions)
  aws cloudtrail create-trail \
    --name SecurityTrail \
    --s3-bucket-name cloudtrail-logs-bucket \
    --is-multi-region-trail \
    --enable-log-file-validation
  
  # Start logging
  aws cloudtrail start-logging --name SecurityTrail
  
  # Enable data events for S3
  aws cloudtrail put-event-selectors \
    --trail-name SecurityTrail \
    --event-selectors '[{
      "ReadWriteType": "All",
      "IncludeManagementEvents": true,
      "DataResources": [{
        "Type": "AWS::S3::Object",
        "Values": ["arn:aws:s3"]
      }]
    }]'

BEST PRACTICES:
  → Enable in ALL regions
  → Enable log file validation
  → Store in secure, separate account S3 bucket
  → Enable S3 server access logging on trail bucket
  → Never delete or disable CloudTrail
  → Use SCP to prevent trail deletion
  → Integrate with CloudWatch for real-time alerts
  → Retain logs for at least 1 year
```

---

## 2. Security-Critical Events to Monitor

```
HIGH-PRIORITY CLOUDTRAIL EVENTS:

IAM CHANGES:
  → CreateUser, DeleteUser
  → CreateAccessKey, UpdateAccessKey
  → AttachUserPolicy, DetachUserPolicy
  → CreateRole, UpdateAssumeRolePolicy
  → PutUserPolicy (inline policy)
  → CreateLoginProfile (console access)

AUTHENTICATION:
  → ConsoleLogin (especially failures)
  → AssumeRole (cross-account)
  → GetSessionToken (MFA)
  → FederatedLogin
  → SwitchRole

INFRASTRUCTURE:
  → RunInstances, TerminateInstances
  → CreateSecurityGroup, AuthorizeSecurityGroupIngress
  → CreateVpc, DeleteVpc
  → ModifyInstanceAttribute
  → CreateNetworkAclEntry

DATA ACCESS:
  → PutBucketPolicy, DeleteBucketPolicy
  → PutBucketAcl
  → CreateDBSnapshot (RDS)
  → CopyDBSnapshot (cross-account!)
  → ModifyDBInstance

SECURITY SERVICES:
  → StopLogging (CloudTrail disabled!)
  → DeleteTrail
  → DisableRule (Config/GuardDuty)
  → DeleteDetector (GuardDuty)

RED FLAGS:
  → API calls from unusual IPs
  → Failed authentication attempts
  → Root account usage
  → Access key usage from new locations
  → CloudTrail/GuardDuty being disabled
  → Large-scale resource creation (cryptomining)
  → Cross-region activity (unusual)
```

---

## 3. CloudWatch

```
CLOUDWATCH — MONITORING AND ALERTING:

COMPONENTS:
  CloudWatch Metrics:
  → Numerical time-series data
  → EC2: CPU, Network, Disk
  → RDS: Connections, CPU, Storage
  → Lambda: Invocations, Duration, Errors
  → Custom metrics from applications
  
  CloudWatch Logs:
  → Log data from AWS services
  → Application logs
  → VPC Flow Logs
  → CloudTrail logs
  → OS-level logs (via agent)
  
  CloudWatch Alarms:
  → Alert based on metrics/logs
  → Trigger actions (SNS, Lambda, Auto Scaling)
  → Threshold-based or anomaly-based
  
  CloudWatch Dashboards:
  → Visual display of metrics
  → Custom layouts
  → Cross-account, cross-region

LOG GROUPS AND STREAMS:
  Log Group: /aws/cloudtrail/SecurityTrail
  └── Log Stream: 123456789012_CloudTrail_us-east-1
      └── Log Events (individual entries)

METRIC FILTERS:
  # Create metric from log pattern
  # Detect root account logins
  aws logs put-metric-filter \
    --log-group-name CloudTrail/SecurityTrail \
    --filter-name RootAccountUsage \
    --filter-pattern '{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }' \
    --metric-transformations \
      metricName=RootAccountUsageCount,\
      metricNamespace=SecurityMetrics,\
      metricValue=1

COMMON SECURITY METRIC FILTERS:
  # Unauthorized API calls
  '{ ($.errorCode = "*UnauthorizedAccess*") || ($.errorCode = "AccessDenied*") }'
  
  # Console login without MFA
  '{ ($.eventName = "ConsoleLogin") && ($.additionalEventData.MFAUsed != "Yes") }'
  
  # IAM policy changes
  '{ ($.eventName = CreatePolicy) || ($.eventName = DeletePolicy) || ($.eventName = AttachUserPolicy) }'
  
  # Security group changes
  '{ ($.eventName = AuthorizeSecurityGroupIngress) || ($.eventName = RevokeSecurityGroupIngress) }'
  
  # CloudTrail changes
  '{ ($.eventName = StopLogging) || ($.eventName = DeleteTrail) }'
  
  # S3 bucket policy changes
  '{ ($.eventName = PutBucketPolicy) || ($.eventName = PutBucketAcl) || ($.eventName = DeleteBucketPolicy) }'
```

---

## 4. Alerting and Automation

```
SECURITY ALERTING:

CLOUDWATCH ALARM → SNS:
  # Create alarm for root usage
  aws cloudwatch put-metric-alarm \
    --alarm-name RootAccountUsageAlarm \
    --metric-name RootAccountUsageCount \
    --namespace SecurityMetrics \
    --statistic Sum \
    --period 300 \
    --threshold 1 \
    --comparison-operator GreaterThanOrEqualToThreshold \
    --evaluation-periods 1 \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:SecurityAlerts

AUTOMATED RESPONSE:
  CloudTrail Event → CloudWatch Logs →
  Metric Filter → Alarm → SNS → Lambda → Remediation
  
  Example automated responses:
  → Public S3 bucket detected → Lambda removes public access
  → Open security group → Lambda removes 0.0.0.0/0 rule
  → Root login detected → SNS alert to security team
  → CloudTrail disabled → Lambda re-enables it

EVENTBRIDGE RULES:
  # Real-time event matching
  {
    "source": ["aws.iam"],
    "detail-type": ["AWS API Call via CloudTrail"],
    "detail": {
      "eventName": ["CreateUser", "CreateAccessKey", "AttachUserPolicy"]
    }
  }
  
  # Route to Lambda for automated response
  # Route to SNS for alerts
  # Route to SQS for processing
```

---

## 5. Investigation Workflow

```
INCIDENT INVESTIGATION WITH CLOUDTRAIL:

STEP 1: IDENTIFY COMPROMISED CREDENTIALS
  # Search for access key usage
  aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=AKIA...
  
  # Search by username
  aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=Username,AttributeValue=admin

STEP 2: DETERMINE SCOPE
  # What did the attacker do?
  # Search by time range and user
  aws cloudtrail lookup-events \
    --start-time 2024-01-15T00:00:00Z \
    --end-time 2024-01-16T00:00:00Z \
    --lookup-attributes AttributeKey=Username,AttributeValue=admin

STEP 3: ATHENA QUERIES (For S3-stored logs)
  -- Create Athena table for CloudTrail
  -- Query specific events
  SELECT eventtime, eventname, sourceipaddress,
         useridentity.arn, requestparameters
  FROM cloudtrail_logs
  WHERE useridentity.accesskeyid = 'AKIA...'
    AND eventtime BETWEEN '2024-01-15' AND '2024-01-16'
  ORDER BY eventtime;

STEP 4: IDENTIFY ACTIONS TAKEN
  → Resources created/modified
  → Data accessed/exfiltrated
  → Persistence mechanisms installed
  → Lateral movement attempts

STEP 5: REMEDIATE
  → Disable compromised credentials
  → Revoke active sessions
  → Remove attacker persistence
  → Fix root cause
  → Document and report
```

---

## Summary Table

| Service | Purpose | Key Feature |
|---------|---------|-------------|
| CloudTrail | API call logging | Who did what, when |
| CloudWatch Logs | Log aggregation | Centralized logging |
| CloudWatch Metrics | Performance metrics | Resource monitoring |
| CloudWatch Alarms | Alerting | Threshold-based alerts |
| EventBridge | Event routing | Real-time responses |

---

## Revision Questions

1. What information does a CloudTrail log entry contain?
2. What are the most critical CloudTrail events to monitor?
3. How do CloudWatch metric filters convert log data to actionable alerts?
4. How can automated remediation be implemented using CloudTrail events?
5. What is the workflow for investigating a security incident using CloudTrail?

---

*Previous: [04-vpc-security.md](04-vpc-security.md) | Next: [06-aws-security-services.md](06-aws-security-services.md)*

---

*[Back to README](../README.md)*
