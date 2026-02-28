# Chapter 3.3: Risk Assessment

[← Previous: Measuring Reliability](02-measuring-reliability.md) | [Back to README](../README.md) | [Next: Failure Modes →](04-failure-modes.md)

---

## Overview

Risk assessment in SRE is the systematic process of identifying, analyzing, and prioritizing risks to service reliability. It bridges the gap between intuition-driven operations and data-driven reliability engineering. By quantifying risks, SRE teams can allocate resources effectively and make informed decisions about where to invest in reliability improvements.

---

## 1. Risk Assessment Framework

```
  ┌──────────────────────────────────────────────────────────┐
  │           RISK ASSESSMENT PROCESS                        │
  │                                                          │
  │  1. IDENTIFY ──► What could go wrong?                    │
  │       │          (brainstorm failures)                   │
  │       ▼                                                  │
  │  2. ANALYZE  ──► How likely? How severe?                 │
  │       │          (probability × impact)                  │
  │       ▼                                                  │
  │  3. EVALUATE ──► Which risks are acceptable?             │
  │       │          (compare to risk tolerance)             │
  │       ▼                                                  │
  │  4. TREAT   ──► Mitigate, transfer, accept, or avoid    │
  │       │                                                  │
  │       ▼                                                  │
  │  5. MONITOR ──► Track risk indicators continuously       │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Risk Matrix

```
  ┌──────────────────────────────────────────────────────────┐
  │              RISK PRIORITY MATRIX                        │
  │                                                          │
  │  Impact ▲                                                │
  │         │                                                │
  │  Cata-  │  MEDIUM    │   HIGH     │  CRITICAL            │
  │  strophic│           │            │                      │
  │         │            │            │                      │
  │  Major  │  LOW       │   MEDIUM   │  HIGH                │
  │         │            │            │                      │
  │         │            │            │                      │
  │  Minor  │  VERY LOW  │   LOW      │  MEDIUM              │
  │         │            │            │                      │
  │         └────────────┴────────────┴──────────────►       │
  │           Unlikely     Possible     Likely               │
  │                                                          │
  │                    Likelihood                             │
  │                                                          │
  │  Risk Score = Likelihood (1-5) × Impact (1-5)            │
  │  1-5: Low | 6-12: Medium | 13-19: High | 20-25: Critical│
  └──────────────────────────────────────────────────────────┘
```

---

## 3. Risk Quantification for Services

### Expected Risk = Probability × Impact

```
  ┌─────────────────────────────────────────────────────────────┐
  │                                                             │
  │  Risk: Database primary failure                             │
  │                                                             │
  │  Probability: 2 incidents/year = 2/365 per day              │
  │  Impact: 30-minute outage × $10,000/min = $300,000          │
  │  Expected annual risk: 2 × $300,000 = $600,000/year         │
  │                                                             │
  │  Mitigation: Automated failover ($50K to implement)         │
  │  Post-mitigation MTTR: 2 minutes (vs 30)                    │
  │  New annual risk: 2 × (2 min × $10K) = $40,000/year        │
  │                                                             │
  │  ROI of mitigation: ($600K - $40K) - $50K = $510K/year     │
  │                                                             │
  │  [!] Data-driven justification for reliability investment   │
  └─────────────────────────────────────────────────────────────┘
```

---

## 4. Risk Register

```yaml
# Risk Register Template
risks:
  - id: RISK-001
    title: "Primary database failure"
    category: "Infrastructure"
    likelihood: 3  # 1=Rare, 5=Almost Certain
    impact: 5      # 1=Negligible, 5=Catastrophic
    risk_score: 15 # High
    description: "Single primary DB instance failure causes full outage"
    current_controls:
      - "Daily backups"
      - "Manual failover procedure (30 min MTTR)"
    proposed_mitigations:
      - "Automated failover with read replicas"
      - "Multi-AZ deployment"
    owner: "SRE Team"
    status: "Open"
    
  - id: RISK-002
    title: "Deployment causes regression"
    category: "Change Management"
    likelihood: 4
    impact: 3
    risk_score: 12 # Medium
    current_controls:
      - "Unit tests, integration tests"
    proposed_mitigations:
      - "Canary deployments"
      - "Automated rollback on SLO violation"
    owner: "SRE + Dev Team"
    status: "In Progress"
```

---

## 5. Techniques for Identifying Risks

```
  ┌──────────────────────────────────────────────────────────┐
  │  TECHNIQUE            │ DESCRIPTION                      │
  ├───────────────────────┼──────────────────────────────────┤
  │ Pre-mortem analysis   │ "Imagine the service failed.     │
  │                       │  Why did it fail?"               │
  │                       │                                  │
  │ Dependency mapping    │ Map all dependencies and assess  │
  │                       │ failure impact of each           │
  │                       │                                  │
  │ Historical analysis   │ Review past incidents for        │
  │                       │ recurring patterns               │
  │                       │                                  │
  │ Chaos engineering     │ Deliberately inject failures     │
  │                       │ to discover weaknesses           │
  │                       │                                  │
  │ Architecture review   │ Identify single points of failure│
  │                       │ and bottlenecks                  │
  │                       │                                  │
  │ Threat modeling       │ Systematic identification of     │
  │                       │ security and reliability threats │
  └───────────────────────┴──────────────────────────────────┘
```

---

## 6. Risk Treatment Strategies

```
  ┌──────────────────────────────────────────────────────────┐
  │  STRATEGY   │ WHEN TO USE         │ EXAMPLE             │
  ├─────────────┼─────────────────────┼─────────────────────┤
  │ AVOID       │ Risk is too high    │ Don't use untested  │
  │             │ and preventable     │ technology in prod  │
  │             │                     │                     │
  │ MITIGATE    │ Risk is high but    │ Add redundancy,     │
  │             │ can be reduced      │ improve monitoring  │
  │             │                     │                     │
  │ TRANSFER    │ Risk is manageable  │ Managed DB service, │
  │             │ by another party    │ insurance, SLAs     │
  │             │                     │                     │
  │ ACCEPT      │ Risk is low or cost │ Internal tool with  │
  │             │ of mitigation > risk│ 99% SLO is fine     │
  └─────────────┴─────────────────────┴─────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] Risk Assessment for Cloud Migration

```
  Company migrating from on-prem to AWS
  
  ┌──────────────────────────────────────────────────────┐
  │ Risk                │ Score │ Treatment              │
  ├─────────────────────┼───────┼────────────────────────┤
  │ Data loss during    │  20   │ Mitigate: parallel     │
  │ migration           │       │ run, backup verification│
  │                     │       │                        │
  │ Network latency     │  12   │ Mitigate: CDN + edge   │
  │ increase            │       │ caching                │
  │                     │       │                        │
  │ Cloud provider      │   8   │ Accept: multi-AZ       │
  │ region outage       │       │ (transfer via SLA)     │
  │                     │       │                        │
  │ Team skill gap      │  15   │ Mitigate: training +   │
  │ with cloud          │       │ gradual migration      │
  │                     │       │                        │
  │ Cost overrun        │   9   │ Mitigate: cost alerts, │
  │                     │       │ reserved instances     │
  └─────────────────────┴───────┴────────────────────────┘
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "We can't quantify risk probability" | Use historical incident data. If unavailable, use industry benchmarks. |
| "Risk assessment feels subjective" | Use structured scoring (1-5 scales) and calibrate across the team. |
| "Too many risks to address" | Prioritize by score. Focus on top 5. Accept or monitor the rest. |
| "Risks change faster than we assess" | Make risk review a quarterly process, not a one-time exercise. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Risk Score** | Likelihood × Impact (1-25 scale) |
| **Expected Risk** | Probability × Financial Impact |
| **Treatment Strategies** | Avoid, Mitigate, Transfer, Accept |
| **Pre-mortem** | Imagine failure and work backward to find causes |
| **Risk Register** | Living document tracking all identified risks |
| **ROI Calculation** | (Risk reduction) − (Mitigation cost) = Value |

---

## Quick Revision Questions

1. **What is the formula for risk scoring in SRE?**
   > Risk Score = Likelihood (1-5) × Impact (1-5). Expected financial risk = Probability × Impact cost.

2. **Name the four risk treatment strategies with examples.**
   > Avoid (don't use risky tech), Mitigate (add redundancy), Transfer (use managed services/SLAs), Accept (low-risk internal tools).

3. **What is a pre-mortem analysis?**
   > Imagine the service has already failed. Work backward to identify why — reveals risks that forward-thinking often misses.

4. **How do you calculate ROI for a reliability investment?**
   > ROI = (Annual risk before - Annual risk after) - Mitigation cost.

5. **Why should risk assessment be an ongoing process?**
   > Systems, dependencies, and traffic patterns change. Quarterly reviews keep the risk register current.

---

[← Previous: Measuring Reliability](02-measuring-reliability.md) | [Back to README](../README.md) | [Next: Failure Modes →](04-failure-modes.md)
