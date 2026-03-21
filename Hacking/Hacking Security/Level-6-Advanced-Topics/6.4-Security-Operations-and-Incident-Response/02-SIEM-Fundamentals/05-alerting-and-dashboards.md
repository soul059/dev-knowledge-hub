# Unit 2: SIEM Fundamentals — Topic 5: Alerting and Dashboards

## Overview

**Alerting and dashboards** are the primary interfaces through which SOC analysts interact with the SIEM. Well-designed alerts ensure the right people are notified about the right threats at the right time, while dashboards provide situational awareness and investigation context. Poor alerting leads to alert fatigue; poor dashboards lead to missed threats.

---

## 1. Alerting Framework

```
ALERT DESIGN PRINCIPLES:

GOOD ALERT CHARACTERISTICS:
  ✓ Actionable — analyst can take specific action
  ✓ True positive rate > 70%
  ✓ Clearly titled with context
  ✓ Appropriate severity
  ✓ Includes relevant evidence
  ✓ Mapped to ATT&CK technique
  ✓ Has associated playbook/runbook
  ✓ Suppression to prevent duplicates

BAD ALERT CHARACTERISTICS:
  ✗ Vague title ("Suspicious Activity")
  ✗ No context or evidence
  ✗ High false positive rate
  ✗ No playbook reference
  ✗ Wrong severity
  ✗ Not actionable
  ✗ No suppression (floods queue)

ALERT SEVERITY LEVELS:
  Level    | Description                | Response Time
  Critical | Active breach, data loss   | Immediate
  High     | Confirmed compromise       | < 1 hour
  Medium   | Suspicious activity        | < 4 hours
  Low      | Policy violation, info     | < 24 hours
  Info     | Informational only         | Review in batch

ALERT CONTENT:
  ┌──────────────────────────────────────────┐
  │  ALERT: Brute Force with Successful     │
  │         Login Detected                   │
  │                                          │
  │  Severity: HIGH                          │
  │  Time: 2024-01-15 10:30:45 UTC          │
  │  ATT&CK: T1110.001 - Password Guessing │
  │                                          │
  │  Source IP: 203.0.113.50                │
  │  Target User: admin@company.com         │
  │  Target Host: dc01.company.local        │
  │  Failed Attempts: 47                     │
  │  Successful Login: Yes                   │
  │                                          │
  │  Enrichment:                             │
  │  → GeoIP: Russia                        │
  │  → Threat Intel: Known brute force IP   │
  │  → Asset: Domain Controller (Critical)  │
  │                                          │
  │  Playbook: SOC-PB-001-BruteForce       │
  └──────────────────────────────────────────┘
```

---

## 2. Alert Routing and Notification

```
ALERT ROUTING:

NOTIFICATION CHANNELS:
  Channel          | Use Case
  SIEM Dashboard   | All alerts (default)
  Email            | Medium/High severity
  Slack/Teams      | Team channel notifications
  PagerDuty        | Critical (on-call paging)
  SMS              | Critical (backup)
  SOAR/Ticketing   | Auto-create tickets
  Phone call       | Critical escalation

ROUTING LOGIC:
  ┌─────────────┐
  │  NEW ALERT  │
  └──────┬──────┘
         │
    ┌────▼────┐
    │Critical?│──Yes──▶ Page on-call + Slack + Ticket
    └────┬────┘
         │ No
    ┌────▼────┐
    │  High?  │──Yes──▶ Slack + Email + Ticket
    └────┬────┘
         │ No
    ┌────▼────┐
    │ Medium? │──Yes──▶ Ticket + Dashboard
    └────┬────┘
         │ No
    ┌────▼────┐
    │  Low?   │──Yes──▶ Dashboard only
    └─────────┘

ESCALATION MATRIX:
  Time Without Response | Action
  15 minutes           | Re-notify assigned analyst
  30 minutes           | Escalate to Tier 2
  1 hour               | Escalate to SOC Manager
  2 hours              | Escalate to CISO
  4 hours              | Executive notification

ALERT SUPPRESSION:
  → Prevent duplicate alerts for same incident
  → Time-based: Suppress for X minutes
  → Count-based: Alert once per N occurrences
  → Group-based: One alert per entity
  → Example: Brute force — alert once per source IP
    per hour, not per attempt
```

---

## 3. Dashboard Design

```
SOC DASHBOARD HIERARCHY:

TIER 1: EXECUTIVE DASHBOARD
  → Overall risk posture (RAG status)
  → Incident count trend (30 days)
  → MTTD/MTTR trends
  → Top risk areas
  → Compliance status
  → Cost/impact metrics

TIER 2: SOC MANAGER DASHBOARD
  → Active incidents by severity
  → Analyst workload distribution
  → SLA compliance
  → Alert queue status
  → Shift staffing
  → Weekly/monthly trends

TIER 3: ANALYST OPERATIONAL DASHBOARD
  ┌─────────────────────────────────────────────┐
  │  SOC OPERATIONS - REAL TIME                  │
  │                                              │
  │  ┌─────────────┐  ┌──────────────────────┐  │
  │  │ OPEN ALERTS │  │ ALERT TREND (24h)    │  │
  │  │             │  │  ████▓▓▒▒░░          │  │
  │  │ Crit:  2    │  │  ▲ Peak at 14:00     │  │
  │  │ High:  8    │  │                      │  │
  │  │ Med:  23    │  └──────────────────────┘  │
  │  │ Low:  45    │                             │
  │  └─────────────┘  ┌──────────────────────┐  │
  │                    │ TOP SOURCES          │  │
  │  ┌─────────────┐  │ EDR: 34%             │  │
  │  │ INCIDENTS   │  │ Firewall: 28%        │  │
  │  │ Active: 5   │  │ SIEM: 22%            │  │
  │  │ Pending: 3  │  │ Email: 16%           │  │
  │  │ Closed: 142 │  └──────────────────────┘  │
  │  └─────────────┘                             │
  │                    ┌──────────────────────┐  │
  │  ┌─────────────┐  │ GEOGRAPHIC MAP       │  │
  │  │ HEALTH      │  │ [World map with      │  │
  │  │ SIEM: ✅    │  │  attack origins]     │  │
  │  │ EDR:  ✅    │  │                      │  │
  │  │ FW:   ✅    │  └──────────────────────┘  │
  │  │ DNS:  ⚠️   │                             │
  │  └─────────────┘                             │
  └─────────────────────────────────────────────┘

INVESTIGATION DASHBOARD:
  → Timeline view of events
  → Entity search (IP, user, host)
  → Related events panel
  → Threat intelligence overlay
  → Process tree visualization
  → Network connection map
```

---

## 4. Dashboard Best Practices

```
DESIGN PRINCIPLES:

LAYOUT:
  → Most important information top-left
  → Group related panels
  → Use consistent color coding
  → Limit to 6-8 panels per dashboard
  → Use drill-down for details
  → Refresh rate appropriate for data

COLOR CODING:
  Color  | Meaning
  Red    | Critical / Immediate attention
  Orange | High severity / Warning
  Yellow | Medium / Caution
  Green  | Normal / Good
  Blue   | Informational
  Gray   | Inactive / Not applicable

VISUALIZATION TYPES:
  Data Type           | Best Visualization
  Counts/totals       | Single value / Gauge
  Trends over time    | Line chart / Area chart
  Comparisons         | Bar chart
  Proportions         | Pie/Donut chart
  Geographic          | Map
  Status              | RAG indicators
  Lists               | Table
  Relationships       | Network graph
  Sequences           | Timeline

COMMON MISTAKES:
  ✗ Too many panels (information overload)
  ✗ Wrong chart type for data
  ✗ No drill-down capability
  ✗ Stale data (slow refresh)
  ✗ No context for numbers
  ✗ Rainbow color schemes
  ✗ Tiny fonts and cramped layout
  ✗ No documentation

DASHBOARD CATALOG:
  Dashboard Name          | Audience    | Refresh
  Executive Overview      | Leadership  | Daily
  SOC Operations          | Analysts    | 1 min
  Incident Management     | IR Team     | 5 min
  Threat Intelligence     | Intel Team  | 15 min
  Alert Queue             | Tier 1      | 1 min
  Investigation           | Tier 2      | On-demand
  Compliance              | Audit       | Daily
  Health Monitoring        | Engineering | 1 min
```

---

## 5. Alert Management Lifecycle

```
ALERT LIFECYCLE:

  NEW → ACKNOWLEDGED → INVESTIGATING →
  → ESCALATED/CLOSED

  Status       | Description
  New          | Alert generated, unassigned
  Acknowledged | Analyst accepted, reviewing
  Investigating| Active investigation
  Escalated    | Sent to higher tier
  Contained    | Threat contained
  Resolved     | Incident resolved
  Closed       | Ticket closed with notes
  False Positive| Documented and rule tuned

ALERT TRIAGE PROCESS:
  1. Read alert details and context
  2. Check enrichment data
  3. Look up entity reputation
  4. Review related events
  5. Determine: TP, FP, or needs escalation
  6. Take action per playbook
  7. Document findings
  8. Close or escalate

ALERT FATIGUE PREVENTION:
  Strategy              | Implementation
  Tune noisy rules      | Regular FP review
  Risk-based alerting   | Score accumulation
  Automation            | SOAR for repetitive tasks
  Consolidation         | Group related alerts
  Suppression           | Deduplicate alerts
  Tiered routing        | Right alert to right tier
  Regular review        | Quarterly rule review
  Feedback loop         | Analyst input on rules

METRICS:
  → Alerts per day (trend)
  → Alerts per analyst per shift
  → Mean triage time
  → False positive rate by rule
  → Alert-to-incident ratio
  → Time in queue before assignment
  → Analyst satisfaction score
```

---

## Summary Table

| Component | Purpose | Key Metric |
|-----------|---------|------------|
| Alert design | Actionable notifications | FP rate < 30% |
| Alert routing | Right person, right time | MTTA < 15 min |
| Suppression | Prevent duplicates | Reduction ratio |
| Dashboards | Situational awareness | Refresh rate |
| Triage | Validate and act | Triage time |

---

## Revision Questions

1. What makes a security alert actionable and effective?
2. How should alerts be routed based on severity?
3. What information should be included on a SOC operations dashboard?
4. How can alert fatigue be prevented?
5. What is the alert management lifecycle?

---

*Previous: [04-correlation-rules.md](04-correlation-rules.md) | Next: [06-common-siem-platforms.md](06-common-siem-platforms.md)*

---

*[Back to README](../README.md)*
