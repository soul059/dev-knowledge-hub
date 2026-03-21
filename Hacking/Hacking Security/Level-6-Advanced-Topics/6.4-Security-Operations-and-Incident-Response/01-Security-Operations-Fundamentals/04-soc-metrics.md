# Unit 1: Security Operations Fundamentals — Topic 4: SOC Metrics

## Overview

**SOC metrics** provide quantitative measures of security operations effectiveness, efficiency, and coverage. Well-defined metrics drive continuous improvement, justify budget, demonstrate value to stakeholders, and help identify operational gaps. This topic covers essential SOC metrics, measurement methodologies, and reporting strategies.

---

## 1. Key SOC Metrics

```
CORE SOC METRICS:

DETECTION METRICS:
  ┌─────────────────────────────────────────────┐
  │  MEAN TIME TO DETECT (MTTD)                  │
  │  Time from threat occurrence to detection     │
  │  Target: < 24 hours (advanced: < 1 hour)     │
  │                                               │
  │  DETECTION RATE                               │
  │  % of threats detected vs total threats       │
  │  Target: > 90%                                │
  │                                               │
  │  COVERAGE                                     │
  │  % of MITRE ATT&CK techniques with detection │
  │  Target: > 70% of relevant techniques         │
  │                                               │
  │  FALSE POSITIVE RATE                          │
  │  % of alerts that are false positives         │
  │  Target: < 30%                                │
  │                                               │
  │  TRUE POSITIVE RATE                           │
  │  % of alerts that are real threats             │
  │  Target: > 70%                                │
  └─────────────────────────────────────────────┘

RESPONSE METRICS:
  ┌─────────────────────────────────────────────┐
  │  MEAN TIME TO RESPOND (MTTR)                 │
  │  Time from detection to containment           │
  │  Target: < 4 hours (critical incidents)       │
  │                                               │
  │  MEAN TIME TO CONTAIN (MTTC)                 │
  │  Time from detection to full containment      │
  │  Target: < 8 hours                            │
  │                                               │
  │  MEAN TIME TO REMEDIATE                       │
  │  Time from detection to full remediation      │
  │  Target: varies by severity                   │
  │                                               │
  │  MEAN TIME TO ACKNOWLEDGE (MTTA)             │
  │  Time from alert to analyst acknowledgment    │
  │  Target: < 15 minutes                         │
  └─────────────────────────────────────────────┘

OPERATIONAL METRICS:
  Metric                      | Target
  Alert volume (daily)        | Track trend
  Alerts per analyst (daily)  | < 50-75
  Escalation rate             | < 20%
  Ticket backlog              | < 24 hours
  SLA compliance              | > 95%
  Shift coverage              | 100% 24/7
  Log sources monitored       | > 90% of critical
  Mean time to close ticket   | < SLA
  Analyst utilization         | 70-80%
```

---

## 2. Measuring Detection Effectiveness

```
DETECTION EFFECTIVENESS:

MITRE ATT&CK COVERAGE MAP:
  Tactic           | Techniques | Detected | Coverage
  Initial Access   | 9          | 7        | 78%
  Execution        | 12         | 9        | 75%
  Persistence      | 19         | 12       | 63%
  Priv Escalation  | 13         | 8        | 62%
  Defense Evasion  | 42         | 20       | 48%
  Credential Access| 17         | 12       | 71%
  Discovery        | 31         | 18       | 58%
  Lateral Movement | 9          | 6        | 67%
  Collection       | 17         | 8        | 47%
  Exfiltration     | 9          | 5        | 56%
  Command & Control| 16         | 10       | 63%
  Impact           | 13         | 9        | 69%

DETECTION RULE METRICS:
  → Total active detection rules
  → Rules per data source
  → Rules mapped to ATT&CK
  → Rules without ATT&CK mapping
  → Rules triggered (last 30 days)
  → Rules never triggered (stale)
  → Rules with highest false positive rate
  → Time to create new detection
  → Time to tune detection

DETECTION MATURITY:
  Level 0: No detection
  Level 1: Basic signature matching
  Level 2: Behavioral detection
  Level 3: Advanced analytics, UEBA
  Level 4: Proactive hunting
  Level 5: Predictive/intelligence-led

DWELL TIME TRACKING:
  → Industry average: 200+ days (declining)
  → Mature SOC: < 7 days
  → Advanced SOC: < 24 hours
  → Dwell time = time attacker is present undetected
  → Lower = better detection capability
```

---

## 3. Operational Metrics and SLAs

```
SERVICE LEVEL AGREEMENTS:

INCIDENT SEVERITY SLAs:
  Severity | Acknowledge | Contain  | Remediate | Report
  Critical | 15 min      | 1 hour   | 24 hours  | 48 hours
  High     | 30 min      | 4 hours  | 72 hours  | 1 week
  Medium   | 2 hours     | 24 hours | 1 week    | 2 weeks
  Low      | 8 hours     | 72 hours | 30 days   | Monthly

ALERT HANDLING METRICS:
  → Alerts generated per day/week/month
  → Alerts triaged within SLA
  → Alert distribution by source
  → Alert distribution by severity
  → Alert distribution by category
  → Top 10 noisiest detection rules
  → Alerts requiring investigation
  → Average triage time per alert

ANALYST PERFORMANCE:
  → Alerts handled per shift
  → Average time per alert
  → Escalation accuracy rate
  → False positive identification rate
  → Documentation quality score
  → Training completion
  → Knowledge assessment scores

TOOL EFFECTIVENESS:
  → SIEM uptime (target: 99.9%)
  → Log ingestion rate (EPS)
  → Log source health (% reporting)
  → Query response time
  → Storage utilization
  → Integration health
  → SOAR playbook execution rate
```

---

## 4. Reporting and Dashboards

```
SOC REPORTING CADENCE:

REAL-TIME DASHBOARD:
  → Current active incidents
  → Alert queue status
  → Analyst availability
  → System health indicators
  → Threat level indicator
  → Active investigations

DAILY REPORT:
  → Alerts received and processed
  → Incidents opened/closed
  → SLA compliance
  → Notable events
  → Analyst shift notes
  → Pending escalations

WEEKLY REPORT:
  → Incident trends
  → Top attack vectors
  → Detection rule effectiveness
  → Backlog status
  → Team capacity
  → Key accomplishments

MONTHLY REPORT:
  → Key metrics with trends
  → Incident statistics
  → MTTD/MTTR trends
  → Detection coverage changes
  → Process improvements
  → Training activities
  → Budget utilization
  → Recommendations

EXECUTIVE DASHBOARD:
  → Risk posture (RAG status)
  → Incident count by severity
  → MTTD/MTTR trends
  → Compliance status
  → Top risks
  → Investment ROI
  → Comparison to benchmarks

SAMPLE KPI DASHBOARD:
  ┌─────────────────────────────────────────────┐
  │  SOC PERFORMANCE DASHBOARD - Q4 2024        │
  │                                              │
  │  MTTD: 2.3 hrs ▼ (was 4.1)  ✅ IMPROVING   │
  │  MTTR: 3.8 hrs ▼ (was 6.2)  ✅ IMPROVING   │
  │  FP Rate: 28%  ▼ (was 35%)  ✅ IMPROVING   │
  │  SLA Met: 96%  ▲ (was 92%)  ✅ ON TARGET   │
  │  Coverage: 72%  ▲ (was 65%) ✅ IMPROVING   │
  │  Incidents: 47  (Critical: 3, High: 12)     │
  │  Dwell Time: 4.2 days (was 8.1)             │
  └─────────────────────────────────────────────┘
```

---

## 5. Benchmarking and Improvement

```
INDUSTRY BENCHMARKS:

BENCHMARK SOURCES:
  → SANS SOC Survey
  → Verizon DBIR
  → IBM Cost of a Data Breach
  → Ponemon Institute
  → MITRE ATT&CK Evaluations
  → Gartner SOC Research

COMMON BENCHMARKS:
  Metric              | Average   | Best Practice
  MTTD                | 197 days  | < 24 hours
  MTTR                | 69 days   | < 4 hours
  Cost per incident   | $4.45M    | Varies
  Alert volume (daily)| 10,000+   | Varies by size
  False positive rate | 50%+      | < 30%
  Analyst turnover    | 25-35%    | < 15%

CONTINUOUS IMPROVEMENT:
  1. Measure current baseline
  2. Set improvement targets
  3. Implement changes
  4. Measure impact
  5. Adjust and repeat

IMPROVEMENT AREAS:
  → Reduce MTTD: Better detection, more sources
  → Reduce MTTR: Automation, better playbooks
  → Reduce FP: Rule tuning, ML-based filtering
  → Increase coverage: More log sources, new rules
  → Reduce burnout: Automation, better tools
  → Improve skills: Training, exercises, mentoring

METRICS MATURITY:
  Level 1: No formal metrics
  Level 2: Basic alert counting
  Level 3: KPIs defined and tracked
  Level 4: Trending and benchmarking
  Level 5: Data-driven optimization
```

---

## Summary Table

| Metric Category | Key Metrics | Target |
|----------------|------------|--------|
| Detection | MTTD, coverage, false positive rate | MTTD < 24h, FP < 30% |
| Response | MTTR, MTTC, MTTA | MTTR < 4h, MTTA < 15m |
| Operational | Alert volume, SLA compliance, backlog | SLA > 95% |
| People | Utilization, turnover, training | Turnover < 15% |
| Tool | Uptime, EPS, source health | Uptime 99.9% |

---

## Revision Questions

1. What are the most important SOC metrics and their targets?
2. How is MITRE ATT&CK used to measure detection coverage?
3. What should a monthly SOC report include?
4. How do industry benchmarks help SOC improvement?
5. What is dwell time and why is it a critical metric?

---

*Previous: [03-soc-roles-and-responsibilities.md](03-soc-roles-and-responsibilities.md) | Next: [05-tool-stack-overview.md](05-tool-stack-overview.md)*

---

*[Back to README](../README.md)*
