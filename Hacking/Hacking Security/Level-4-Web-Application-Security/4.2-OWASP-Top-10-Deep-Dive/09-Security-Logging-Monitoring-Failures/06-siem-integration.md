# Unit 9: A09 - Security Logging and Monitoring Failures — Topic 6: SIEM Integration

## Overview

A SIEM (Security Information and Event Management) system **aggregates, correlates, and analyzes security events** from across the entire infrastructure to detect threats, support incident investigation, and meet compliance requirements. Proper SIEM integration ensures that security-relevant logs from all sources are centralized, searchable, and actionable.

---

## 1. SIEM Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    SIEM ARCHITECTURE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LOG SOURCES:                                                  │
│  ├── Web applications (app logs)                               │
│  ├── Web servers (access/error logs)                           │
│  ├── Databases (audit logs)                                    │
│  ├── Firewalls (connection logs)                               │
│  ├── IDS/IPS (alert logs)                                      │
│  ├── Cloud services (CloudTrail, Activity Log)                 │
│  ├── Endpoints (EDR telemetry)                                 │
│  └── Identity providers (auth logs)                            │
│           │                                                    │
│           ▼                                                    │
│  ┌─────────────────────┐                                       │
│  │ LOG COLLECTION       │  Agents, syslog, API ingestion       │
│  │ Fluentd, Logstash,  │  Normalize format (CEF, ECS)         │
│  │ Filebeat, Vector    │                                       │
│  └────────┬────────────┘                                       │
│           ▼                                                    │
│  ┌─────────────────────┐                                       │
│  │ SIEM ENGINE          │  Correlation, detection rules,       │
│  │ Splunk, Elastic,    │  threat intelligence matching,        │
│  │ Sentinel, QRadar    │  anomaly detection (ML/UEBA)          │
│  └────────┬────────────┘                                       │
│           ▼                                                    │
│  ┌─────────────────────┐                                       │
│  │ OUTPUT               │  Dashboards, alerts, reports,         │
│  │ Alerting, Reports,  │  incident tickets, compliance         │
│  │ SOAR integration    │                                       │
│  └─────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. SIEM Platforms

| SIEM | Type | Best For | Log Volume |
|------|------|----------|-----------|
| **Splunk** | Commercial | Large enterprises | Unlimited ($$) |
| **Elastic SIEM** | Open source + commercial | Flexible/affordable | High |
| **Microsoft Sentinel** | Cloud-native | Azure environments | High |
| **IBM QRadar** | Commercial | Enterprise compliance | High |
| **Google Chronicle** | Cloud-native | GCP environments | Unlimited |
| **Wazuh** | Open source | Small/medium orgs | Medium |
| **Graylog** | Open source | Log management focus | Medium |

---

## 3. Log Shipping Configuration

### Filebeat to Elasticsearch

```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    paths:
      - /var/log/app/security.log
    json:
      keys_under_root: true
      add_error_key: true
    fields:
      environment: production
      service: api-server

  - type: log
    paths:
      - /var/log/nginx/access.log
    fields:
      service: web-server

output.elasticsearch:
  hosts: ["https://siem.example.com:9200"]
  username: "filebeat"
  password: "${ELASTIC_PASSWORD}"
  ssl:
    certificate_authorities: ["/etc/pki/ca.pem"]
    verification_mode: full

# Ensure logs are encrypted in transit (TLS)
```

### Application to Cloud SIEM

```python
# Send logs to AWS CloudWatch
import boto3
import json
import time

cloudwatch_logs = boto3.client('logs')

def send_to_cloudwatch(log_group, log_stream, events):
    cloudwatch_logs.put_log_events(
        logGroupName=log_group,
        logStreamName=log_stream,
        logEvents=[{
            'timestamp': int(time.time() * 1000),
            'message': json.dumps(event)
        } for event in events]
    )

# Azure Monitor
# Use Azure Monitor Agent or direct API integration

# Google Cloud Logging
from google.cloud import logging as gcp_logging
client = gcp_logging.Client()
logger = client.logger("security-events")
logger.log_struct({"event": "login_failed", "user": "jsmith"})
```

---

## 4. Correlation Rules

```
SIEM correlation combines events from multiple sources:

EXAMPLE 1: ACCOUNT COMPROMISE DETECTION
  Rule: 
    Event A: >5 failed logins (from auth service)
    FOLLOWED BY
    Event B: Successful login (from auth service)
    FOLLOWED BY
    Event C: Sensitive data access (from database audit)
    ALL within 30 minutes, same user
  Alert: "Possible account compromise — brute force → data access"

EXAMPLE 2: LATERAL MOVEMENT
  Rule:
    Event A: Login from internal IP (from auth service)
    FOLLOWED BY
    Event B: Port scan activity (from IDS)
    FOLLOWED BY
    Event C: Login to different internal host (from auth service)
    ALL from same source IP within 1 hour
  Alert: "Possible lateral movement detected"

EXAMPLE 3: DATA EXFILTRATION
  Rule:
    Event A: Large query result set (from database audit)
    AND
    Event B: High outbound data volume (from firewall)
    SAME user/IP within 30 minutes
  Alert: "Possible data exfiltration"
```

---

## 5. SIEM Use Cases

| Use Case | Log Sources | Detection Method |
|----------|-------------|-----------------|
| Brute force | Auth logs | Threshold (>N failures) |
| Account takeover | Auth + access logs | Sequence (fail → success → unusual activity) |
| Insider threat | All logs | UEBA baseline deviation |
| Malware | EDR + network | IOC matching |
| Data exfiltration | DB audit + firewall | Volume anomaly |
| Compliance | All logs | Scheduled reports |

---

## 6. SIEM Integration Checklist

```
LOG SOURCES:
□ Application security logs connected
□ Web server logs connected
□ Database audit logs connected
□ Authentication service logs connected
□ Cloud provider logs connected (CloudTrail, etc.)
□ Network device logs connected (firewall, IDS)
□ Endpoint detection logs connected

CONFIGURATION:
□ Log format normalized (ECS, CEF, or custom schema)
□ Timestamps synchronized (NTP across all sources)
□ Correlation rules for top 10 attack scenarios
□ Dashboards for security operations team
□ Alert routing to appropriate channels
□ Retention policies configured
□ Log volume capacity planned

OPERATIONS:
□ Alert rules tuned to minimize false positives
□ Threat intelligence feeds integrated
□ Regular rule review and updates
□ SOAR integration for automated response
□ Compliance reports automated
□ Team trained on SIEM queries and investigation
```

---

## Summary Table

| SIEM Component | Purpose | Tools |
|---------------|---------|-------|
| Collection | Gather logs from all sources | Filebeat, Fluentd, agents |
| Normalization | Standardize format | Logstash, parsing pipelines |
| Storage | Retain and index logs | Elasticsearch, Splunk |
| Analysis | Correlate and detect | SIEM rules engine, ML |
| Alerting | Notify on threats | PagerDuty, Slack, email |
| Response | Automate remediation | SOAR platforms |

---

## Revision Questions

1. What is a SIEM and what are its core functions?
2. Compare three SIEM platforms — features, pricing model, and best use case.
3. Configure log shipping from an application to Elasticsearch using Filebeat.
4. Write three correlation rules that combine events from multiple log sources.
5. Design a SIEM integration plan for an organization with web apps, databases, cloud, and network devices.
6. How do you tune SIEM rules to reduce false positives while maintaining detection coverage?

---

*Previous: [05-log-injection-prevention.md](05-log-injection-prevention.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
