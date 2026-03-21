# Unit 8: Post-Incident Activities — Topic 3: Metrics and KPIs

## Overview

**Metrics and KPIs** (Key Performance Indicators) measure the effectiveness of incident response and security operations. They provide data-driven insights for continuous improvement, resource justification, and demonstrating security program maturity to leadership.

---

## 1. IR Metrics Framework

```
METRICS CATEGORIES:

  ┌──────────────────────────────────────────────┐
  │           IR METRICS FRAMEWORK               │
  │                                              │
  │  DETECTION METRICS                           │
  │  → How well do we find threats?              │
  │                                              │
  │  RESPONSE METRICS                            │
  │  → How fast do we respond?                   │
  │                                              │
  │  EFFECTIVENESS METRICS                       │
  │  → How well do we resolve incidents?         │
  │                                              │
  │  EFFICIENCY METRICS                          │
  │  → How efficiently do we operate?            │
  │                                              │
  │  BUSINESS IMPACT METRICS                     │
  │  → What is the cost/impact?                  │
  └──────────────────────────────────────────────┘
```

---

## 2. Key Metrics

```
DETECTION METRICS:

  Metric                  | Definition               | Target
  Mean Time to Detect     | Time from compromise     | < 24 hours
  (MTTD)                  | to detection             |
  Dwell Time              | Time attacker is present | < 7 days
                          | before detection         |
  Detection Rate          | % of incidents detected  | > 95%
                          | by internal tools        |
  False Positive Rate     | FP / Total alerts        | < 50%
  True Positive Rate      | TP / Total alerts        | > 50%
  Alert-to-Incident Ratio | Alerts per real incident | Track trend
  Detection Source        | How incidents are found  | Internal > external

RESPONSE METRICS:

  Metric                  | Definition               | Target
  Mean Time to Respond    | Detection to first       | < 1 hour
  (MTTR)                  | response action          |
  Mean Time to Contain    | Detection to containment | < 4 hours
  (MTTC)                  |                          |
  Mean Time to Resolve    | Detection to full        | < 72 hours
                          | resolution               |
  Mean Time to Recover    | Start to end of recovery | < 48 hours
  Escalation Time         | Triage to escalation     | < 30 min
  Time to Notify          | Detection to stakeholder | Per policy
                          | notification             |

EFFECTIVENESS METRICS:

  Metric                  | Definition               | Target
  Re-compromise Rate      | Incidents that recur     | < 5%
  Containment Effectiveness| % threats fully contained| > 95%
  Eradication Completeness| % threats fully removed  | 100%
  Scope Accuracy          | Initial vs final scope   | > 80% match
  Data Loss Prevented     | Data saved by response   | Track amount
  Corrective Action       | % LL actions completed   | > 90%
  Completion Rate         |                          |
```

---

## 3. SOC Operational Metrics

```
SOC PERFORMANCE METRICS:

  Metric                  | Definition               | Target
  Alerts Processed/Day    | Volume handled           | Track capacity
  Alerts per Analyst      | Workload distribution    | 50-75/shift
  Alert Backlog           | Unprocessed alerts       | < 24 hours
  Triage Time (average)   | Time to classify alert   | < 15 minutes
  Ticket Closure Time     | Open to close            | Track trend
  Analyst Turnover        | % staff leaving/year     | < 15%
  Training Hours          | Training per analyst     | 40+ hours/yr
  Shift Coverage          | % time with coverage     | 100% critical
  Tool Uptime             | SIEM/EDR availability    | > 99.9%

INCIDENT VOLUME METRICS:

  Metric                  | Tracking
  Total Incidents/Month   | Trend over time
  By Severity             | Critical/High/Med/Low
  By Category             | Phishing/Malware/etc.
  By Business Unit        | Which areas most affected
  By Detection Source     | Internal/external/user
  By Root Cause           | Identify patterns
  Repeat Incidents        | Same root cause recurring

METRICS DASHBOARD EXAMPLE:
  ┌──────────────────────────────────────────────┐
  │ SOC METRICS DASHBOARD — January 2024        │
  │                                              │
  │ INCIDENTS    MTTD     MTTR     MTTC          │
  │ 12 total     18 hrs   4 hrs    2.5 hrs       │
  │                                              │
  │ BY SEVERITY:           BY CATEGORY:          │
  │ Critical: 1            Phishing: 5           │
  │ High: 3                Malware: 3            │
  │ Medium: 5              Account comp: 2       │
  │ Low: 3                 Policy viol: 2        │
  │                                              │
  │ ALERTS:                FP RATE:              │
  │ 3,200 total            38%                   │
  │ 75/analyst/shift       ↓ from 45% last month │
  │                                              │
  │ TRENDS: MTTD improved 22% from Q3           │
  │         FP rate decreased 7% from last month │
  │         Critical incidents down from 3 to 1  │
  └──────────────────────────────────────────────┘
```

---

## 4. Business Impact Metrics

```
BUSINESS IMPACT METRICS:

FINANCIAL METRICS:
  Metric                  | How to Calculate
  Cost per Incident       | Total response cost / incidents
  Total Incident Cost     | Staff time + tools + lost
                          | revenue + remediation
  Cost Avoidance          | Estimated damage prevented
  Recovery Cost           | Hardware + software + labor
  Regulatory Fines        | Actual fines/penalties
  Insurance Premium Impact| Change in cyber insurance
  Legal Costs             | Attorney fees, settlements
  Lost Revenue            | Downtime × revenue/hour

COST CALCULATION EXAMPLE:
  ┌──────────────────────────────────────────────┐
  │ INCIDENT COST ANALYSIS — INC-2024-001       │
  │                                              │
  │ DIRECT COSTS:                                │
  │ IR team labor (5 staff × 80 hrs)  $40,000   │
  │ IT recovery labor                  $15,000   │
  │ Forensic consultant                $25,000   │
  │ Hardware replacement                $5,000   │
  │ Software licenses                   $3,000   │
  │                                              │
  │ INDIRECT COSTS:                              │
  │ Business downtime (3 days)         $90,000   │
  │ Lost productivity (100 users)      $30,000   │
  │ Reputation damage (estimated)      $50,000   │
  │                                              │
  │ REGULATORY/LEGAL:                            │
  │ Legal fees                         $10,000   │
  │ Notification costs                  $5,000   │
  │ Credit monitoring                  $15,000   │
  │ Regulatory fine                    $25,000   │
  │                                              │
  │ TOTAL ESTIMATED COST:             $313,000   │
  └──────────────────────────────────────────────┘

ROI METRICS:
  → Security investment vs incident cost reduction
  → Cost per incident trending down
  → Time to detect trending down
  → Number of prevented incidents
  → Reduced dwell time saves $$
```

---

## 5. Reporting and Trends

```
METRICS REPORTING:

REPORTING CADENCE:
  Report          | Frequency | Audience
  Daily dashboard | Daily     | SOC team
  Weekly summary  | Weekly    | SOC management
  Monthly report  | Monthly   | CISO, IT mgmt
  Quarterly review| Quarterly | Executive team
  Annual report   | Annually  | Board, C-suite

TREND ANALYSIS:
  Track metrics over time to identify:
  → Improvement or degradation
  → Seasonal patterns
  → Impact of new tools/processes
  → Staffing adequacy
  → Training effectiveness
  → Threat landscape changes

MATURITY MEASUREMENT:
  Level | Description         | MTTD    | MTTR
  1     | Reactive            | Weeks   | Days
  2     | Proactive           | Days    | Hours
  3     | Managed             | Hours   | < 4 hrs
  4     | Optimized           | Minutes | < 1 hr
  5     | Adaptive            | Real-time| Minutes

BENCHMARKING:
  → Compare against industry averages
  → Use frameworks (NIST, SANS)
  → Participate in information sharing
  → Track year-over-year improvement
  → Set targets based on risk appetite
```

---

## Summary Table

| Category | Key Metrics | Target | Audience |
|----------|-----------|--------|----------|
| Detection | MTTD, dwell time | < 24 hrs | SOC, CISO |
| Response | MTTR, MTTC | < 4 hrs | IR team |
| Effectiveness | Re-compromise rate | < 5% | Management |
| Efficiency | Alerts/analyst, FP rate | Track trend | SOC |
| Business | Cost per incident | Trend down | Executives |

---

## Revision Questions

1. What are the five categories of IR metrics?
2. What is the difference between MTTD, MTTR, and MTTC?
3. How do you calculate the total cost of an incident?
4. What should a SOC metrics dashboard include?
5. How are metrics used to demonstrate security program maturity?

---

*Previous: [02-report-writing.md](02-report-writing.md) | Next: [04-process-improvement.md](04-process-improvement.md)*

---

*[Back to README](../README.md)*
