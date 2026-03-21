# Unit 9: A09 - Security Logging and Monitoring Failures — Topic 3: Monitoring and Alerting

## Overview

Logging without monitoring is writing a diary nobody reads. **Security monitoring** involves real-time analysis of security events to detect threats, while **alerting** notifies the security team when suspicious activity is detected. The goal is to reduce **Mean Time To Detect (MTTD)** — the time between a breach occurring and being discovered.

---

## 1. Security Monitoring Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│              MONITORING PIPELINE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [App Servers] ──→ [Log Aggregator] ──→ [SIEM/Analysis]       │
│  [Network]     ──→ [Fluentd/Logstash]──→ [Elasticsearch]      │
│  [Cloud]       ──→ [CloudWatch/etc] ──→ [Kibana/Grafana]      │
│  [Endpoints]   ──→                      [Alert Engine]         │
│                                              │                  │
│                                              ▼                  │
│                                    [Notification Channels]      │
│                                    ├── Slack/Teams              │
│                                    ├── PagerDuty                │
│                                    ├── Email                    │
│                                    └── Phone (Critical)         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Alert Rules for Common Attacks

```yaml
# Alert Rule Examples

# Brute Force Detection
- name: "Brute Force Attack"
  condition: |
    count(event_type == "auth.login_failed" 
          AND same(source_ip)) > 10
    within: 5 minutes
  severity: HIGH
  action: block_ip, notify_security

# Credential Stuffing
- name: "Credential Stuffing"
  condition: |
    count(event_type == "auth.login_failed"
          AND distinct(username) > 50
          AND same(source_ip)) 
    within: 10 minutes
  severity: CRITICAL
  action: block_ip, enable_captcha, notify_security

# Privilege Escalation
- name: "Unauthorized Privilege Escalation"
  condition: |
    event_type == "authorization.role_changed"
    AND actor.role != "admin"
  severity: CRITICAL
  action: revert_change, lock_account, notify_security

# Data Exfiltration
- name: "Bulk Data Export"
  condition: |
    count(event_type == "data.export"
          AND same(user_id)) > 5
    within: 1 hour
  severity: HIGH
  action: rate_limit, notify_security

# SQL Injection Attempt
- name: "SQL Injection Detected"
  condition: |
    event_type == "input.validation_failed"
    AND details.type == "sql_injection"
  severity: HIGH
  action: block_request, log_payload, notify_security

# After-Hours Admin Activity
- name: "Off-Hours Admin Access"
  condition: |
    actor.role == "admin"
    AND time NOT BETWEEN "08:00" AND "20:00"
    AND day NOT IN ["Saturday", "Sunday"]
  severity: MEDIUM
  action: notify_security
```

---

## 3. Key Metrics to Monitor

| Metric | Normal | Alert Threshold | Indicates |
|--------|--------|----------------|-----------|
| Failed logins/min | <5 | >20 | Brute force |
| 403 errors/min | <2 | >10 | Authorization probing |
| 500 errors/min | <1 | >5 | Attack or system issue |
| New admin accounts/day | 0 | >0 | Privilege escalation |
| Data export volume/hr | <100MB | >1GB | Data exfiltration |
| Login from new country | Rare | Any occurrence | Account compromise |
| API calls/min per user | <100 | >500 | Scraping/abuse |

---

## 4. Alert Fatigue Prevention

```
PROBLEM: Too many alerts → Analysts ignore them all
SOLUTION: Tuning and prioritization

STRATEGIES:
1. SEVERITY-BASED ROUTING
   Critical → PagerDuty phone call (immediate)
   High     → Slack alert (respond within 1 hour)
   Medium   → Email digest (review daily)
   Low      → Dashboard only (review weekly)

2. ALERT CORRELATION
   Don't send 100 alerts for 100 failed logins
   → Group into: "Brute force attack from 203.0.113.50 (100 attempts)"

3. SUPPRESS KNOWN PATTERNS
   Automated vulnerability scanner → suppress during scan windows
   Deployment activity → suppress during maintenance

4. METRICS
   Track: Alerts generated vs investigated vs actionable
   Goal: >50% of alerts should be actionable
   If <20% actionable → tune rules aggressively
```

---

## 5. Dashboard Design

```
SECURITY OPERATIONS DASHBOARD:

┌─────────────────────────────────────────────────────────────────┐
│  SECURITY OVERVIEW                           Last 24 hours     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CRITICAL: 0 │ HIGH: 3 │ MEDIUM: 12 │ LOW: 45               │
│                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐                     │
│  │ Auth Events      │  │ Top Source IPs  │                     │
│  │ Success: 12,456  │  │ 1. 203.0.113.50 │                    │
│  │ Failed:    234   │  │ 2. 198.51.100.1 │                    │
│  │ Lockouts:    5   │  │ 3. 192.0.2.100  │                    │
│  └─────────────────┘  └─────────────────┘                     │
│                                                                 │
│  ┌─────────────────────────────────────┐                       │
│  │ Failed Logins Over Time             │                       │
│  │ ▁▂▁▁▃▂▁█████▃▂▁▁▁▂▁▁▁▁▁▁         │                       │
│  │               ↑                     │                       │
│  │         Brute force attack          │                       │
│  └─────────────────────────────────────┘                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Monitoring Type | Purpose | Tools |
|----------------|---------|-------|
| Real-time alerting | Immediate threat detection | PagerDuty, OpsGenie |
| Log analysis | Pattern detection | ELK, Splunk, Datadog |
| Dashboard | Visual overview | Grafana, Kibana |
| Anomaly detection | Unknown threats | ML-based SIEM, UEBA |
| Compliance reporting | Audit requirements | Splunk, Sumo Logic |

---

## Revision Questions

1. Design a security monitoring architecture showing log sources, aggregation, analysis, and alerting.
2. Write alert rules for brute force, privilege escalation, and data exfiltration detection.
3. What key metrics should a security monitoring dashboard display?
4. How do you prevent alert fatigue while maintaining security coverage?
5. What is Mean Time To Detect (MTTD) and how does monitoring reduce it?

---

*Previous: [02-audit-trails.md](02-audit-trails.md) | Next: [04-incident-response-integration.md](04-incident-response-integration.md)*

---

*[Back to README](../README.md)*
