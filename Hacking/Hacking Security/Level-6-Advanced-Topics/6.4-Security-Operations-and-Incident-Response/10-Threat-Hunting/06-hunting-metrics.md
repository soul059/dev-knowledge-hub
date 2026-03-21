# Unit 10: Threat Hunting — Topic 6: Hunting Metrics

## Overview

**Hunting metrics** quantify the effectiveness, efficiency, and impact of threat hunting operations. Metrics justify the hunting program's value, guide resource allocation, demonstrate maturity progression, and drive continuous improvement. Without metrics, hunting programs cannot demonstrate ROI or identify areas for improvement.

---

## 1. Core Hunting Metrics

```
HUNTING PROGRAM METRICS:

  ┌────────────────────────────────────────────┐
  │       HUNTING METRICS FRAMEWORK            │
  │                                            │
  │  ACTIVITY METRICS                          │
  │  → How much hunting are we doing?          │
  │                                            │
  │  EFFECTIVENESS METRICS                     │
  │  → Are hunts finding threats?              │
  │                                            │
  │  EFFICIENCY METRICS                        │
  │  → How efficiently do we hunt?             │
  │                                            │
  │  IMPACT METRICS                            │
  │  → What value does hunting provide?        │
  │                                            │
  │  MATURITY METRICS                          │
  │  → How mature is our program?              │
  └────────────────────────────────────────────┘

ACTIVITY METRICS:
  Metric                    | Target
  Hunts conducted/month     | 4-8 per month
  Hypotheses tested/quarter | 12-24
  ATT&CK coverage hunted   | 60%+ techniques
  Hours spent hunting/week  | 10-20 hours
  Data sources utilized     | 80%+ available
  Hunters active            | Track over time

EFFECTIVENESS METRICS:
  Metric                    | Target
  Threats discovered        | Track over time
  True positive rate        | > 10% of hunts
  New IOCs discovered       | Track over time
  Dwell time reduction      | Trending down
  Detections created        | 2-5 per hunt
  Misconfigs found          | Track over time
  False positive reduction  | From tuning

EFFICIENCY METRICS:
  Metric                    | Target
  Average hunt duration     | 4-8 hours
  Time to complete hunt     | < 1 week
  Hypothesis backlog age    | < 30 days
  Query reuse rate          | > 50%
  Automation rate           | Trending up
  Hunts per hunter/month    | 2-4
```

---

## 2. Impact Metrics

```
MEASURING HUNTING IMPACT:

THREAT DISCOVERY:
  → Incidents found by hunting
  → Severity of discovered threats
  → Dwell time of discovered threats
  → Damage prevented (estimated)
  → Threats missed by automated detection

DETECTION IMPROVEMENT:
  → New detection rules created from hunts
  → Existing rules tuned from hunt findings
  → Detection gap coverage improvement
  → ATT&CK coverage increase
  → False positive reduction from tuning

SECURITY POSTURE:
  → Misconfigurations discovered
  → Vulnerabilities identified
  → Policy violations found
  → Shadow IT discovered
  → Security recommendations implemented

QUANTIFYING VALUE:
  ┌──────────────────────────────────────────────┐
  │ HUNTING ROI EXAMPLE                          │
  │                                              │
  │ COSTS:                                       │
  │ Hunter salary (1 FTE)         $120,000/yr   │
  │ Tools and infrastructure       $30,000/yr   │
  │ Training                       $10,000/yr   │
  │ Total annual cost:            $160,000/yr   │
  │                                              │
  │ VALUE GENERATED:                             │
  │ Threat discovered (Q1):                      │
  │ → Active C2 beacon on server                │
  │ → Estimated damage prevented: $500,000      │
  │                                              │
  │ Detection improvements (Year):               │
  │ → 48 new detection rules                    │
  │ → 15% reduction in MTTD                     │
  │ → Estimated value: $200,000                 │
  │                                              │
  │ Misconfigs found:                            │
  │ → 12 critical misconfigurations             │
  │ → Prevented potential breaches              │
  │ → Estimated value: $300,000                 │
  │                                              │
  │ ROI: ($1,000,000 - $160,000) / $160,000    │
  │    = 525% ROI                               │
  └──────────────────────────────────────────────┘
```

---

## 3. Tracking and Reporting

```
HUNTING DASHBOARD:

  ┌──────────────────────────────────────────────┐
  │ THREAT HUNTING DASHBOARD — Q1 2024          │
  │                                              │
  │ HUNTS CONDUCTED: 12       THREATS FOUND: 2   │
  │ HYPOTHESES TESTED: 15    DETECTIONS CREATED: 8│
  │                                              │
  │ HUNT ACTIVITY:                               │
  │ Jan: ████████ (5 hunts)                     │
  │ Feb: ██████ (4 hunts)                       │
  │ Mar: ██████ (3 hunts)                       │
  │                                              │
  │ ATT&CK COVERAGE HUNTED:                     │
  │ IA: ██████░░░░ 60%                          │
  │ EX: ████████░░ 80%                          │
  │ PE: ████░░░░░░ 40%                          │
  │ CA: ██████░░░░ 60%                          │
  │ LM: ████████░░ 80%                          │
  │ EF: ██░░░░░░░░ 20%                          │
  │                                              │
  │ KEY FINDINGS:                                │
  │ → Cobalt Strike beacon on WEBSVR03          │
  │ → 3 misconfigured admin accounts            │
  │ → DNS tunneling test tool on dev server     │
  │                                              │
  │ DETECTIONS CREATED: 8 new rules             │
  │ → PS download cradle detection              │
  │ → WMI persistence detection                 │
  │ → Unusual LSASS access detection            │
  │ → 5 IOC-based detections                    │
  └──────────────────────────────────────────────┘

REPORTING CADENCE:
  Report          | Frequency | Audience
  Hunt summary    | After each| Hunt team
  Monthly roll-up | Monthly   | SOC management
  Quarterly report| Quarterly | CISO
  Annual review   | Annually  | Executive team

HUNT LOG:
  ID    | Date    | Hypothesis      | Result  | Detections
  H-001 | Jan 5   | PS downloads    | Clean   | 2 rules
  H-002 | Jan 10  | Sched tasks     | Clean   | 1 rule
  H-003 | Jan 15  | LSASS access    | THREAT  | 1 rule + IR
  H-004 | Jan 22  | DNS tunneling   | Clean   | 1 rule
  H-005 | Jan 28  | WMI lateral     | Misconfig| 0
  ...
```

---

## 4. Maturity Assessment

```
HUNTING MATURITY MODEL:

LEVEL 0 — INITIAL:
  Characteristics:
  → No formal hunting program
  → Reactive only
  → No dedicated resources
  → Limited data collection
  
  Metrics: None tracked

LEVEL 1 — MINIMAL:
  Characteristics:
  → IOC-based searching only
  → Ad-hoc, not systematic
  → Basic data collection
  → Limited tool capability
  
  Metrics:
  → IOC searches conducted
  → Matches found

LEVEL 2 — PROCEDURAL:
  Characteristics:
  → Follow established procedures
  → Regular hunting schedule
  → Multiple data sources
  → Hypothesis-based hunting
  → Document findings
  
  Metrics:
  → Hunts per month
  → Hypotheses tested
  → Threats found
  → Detections created

LEVEL 3 — INNOVATIVE:
  Characteristics:
  → Create own hypotheses
  → Custom analytics
  → Threat intel driven
  → Automate successful hunts
  → Contribute to community
  
  Metrics:
  → All Level 2 metrics
  → ATT&CK coverage
  → Automation rate
  → Detection gap reduction
  → Dwell time impact

LEVEL 4 — LEADING:
  Characteristics:
  → Continuous hunting operations
  → Advanced analytics (ML/AI)
  → Proactive threat research
  → Industry leadership
  → Threat intel production
  
  Metrics:
  → All Level 3 metrics
  → Predictive capability
  → Community contribution
  → Innovation metrics

MATURITY PROGRESSION PLAN:
  Current Level → Target Level → Actions Needed
  0 → 1: Deploy SIEM, start IOC searching
  1 → 2: Establish process, hypothesis-based
  2 → 3: Custom analytics, automation
  3 → 4: ML/AI, predictive hunting
```

---

## 5. Continuous Improvement

```
IMPROVING HUNTING OPERATIONS:

AFTER EACH HUNT:
  [ ] Document findings and methodology
  [ ] Create detection rules from findings
  [ ] Update hypothesis backlog
  [ ] Share knowledge with team
  [ ] Update hunting playbooks
  [ ] Archive queries for reuse

MONTHLY REVIEW:
  [ ] Review hunt metrics
  [ ] Assess ATT&CK coverage
  [ ] Update hypothesis priorities
  [ ] Review data source adequacy
  [ ] Identify training needs
  [ ] Plan next month's hunts

QUARTERLY REVIEW:
  [ ] Assess program maturity
  [ ] Review ROI and impact
  [ ] Evaluate tools and processes
  [ ] Update hunting strategy
  [ ] Present to management
  [ ] Benchmark against industry

ANNUAL REVIEW:
  [ ] Full maturity assessment
  [ ] Program roadmap update
  [ ] Budget and resource planning
  [ ] Tool evaluation
  [ ] Training plan
  [ ] Strategic goals

FEEDBACK LOOPS:
  Hunt Findings → New Detections → Better Monitoring
       ↓
  IR Findings → New Hypotheses → Better Hunts
       ↓
  Threat Intel → Priority Updates → Focused Hunts
       ↓
  Exercises → Gap Identification → Targeted Hunts
```

---

## Summary Table

| Metric Category | Key Metrics | Reporting | Audience |
|----------------|-----------|-----------|----------|
| Activity | Hunts/month, hours spent | Monthly | SOC management |
| Effectiveness | Threats found, TP rate | Monthly | SOC management |
| Impact | Detections created, damage prevented | Quarterly | CISO |
| Efficiency | Hunt duration, query reuse | Monthly | Hunt team |
| Maturity | HMM level, ATT&CK coverage | Annually | Executive team |

---

## Revision Questions

1. What are the five categories of hunting metrics?
2. How do you calculate the ROI of a hunting program?
3. What should a hunting dashboard display?
4. What are the levels of the Hunting Maturity Model?
5. What continuous improvement activities sustain a hunting program?

---

*Previous: [05-tools-and-platforms.md](05-tools-and-platforms.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
