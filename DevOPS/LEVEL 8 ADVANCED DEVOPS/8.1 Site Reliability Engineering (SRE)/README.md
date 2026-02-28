# 📘 Site Reliability Engineering (SRE) — Comprehensive Study Notes

> **Level 8 — Advanced DevOps**
> A complete guide covering SRE principles, practices, and real-world implementation.

---

## Course Overview

Site Reliability Engineering (SRE) is a discipline that applies software engineering principles to infrastructure and operations problems. Originally pioneered at Google, SRE has become the industry standard for building and operating reliable, scalable systems at scale.

These notes cover everything from foundational concepts (SLIs, SLOs, error budgets) through advanced topics (system design patterns, automation strategies, release engineering) with hands-on examples, architecture diagrams, configuration snippets, and real-world scenarios.

```
+------------------------------------------------------------------+
|                  SITE RELIABILITY ENGINEERING                     |
+------------------------------------------------------------------+
|                                                                  |
|   Fundamentals ──> SLOs ──> Reliability ──> Incidents            |
|        │              │          │               │               |
|        v              v          v               v               |
|   Capacity ──> Automation ──> Releases ──> System Design         |
|                                                                  |
|   ┌──────────────────────────────────────────────────────────┐   |
|   │  GOAL: Balance velocity of innovation with reliability   │   |
|   └──────────────────────────────────────────────────────────┘   |
+------------------------------------------------------------------+
```

---

## How to Use These Notes

| Symbol | Meaning |
|--------|---------|
| `>>>` | Hands-on command / config example |
| `[!]` | Important concept or gotcha |
| `[?]` | Revision question |
| `[TIP]` | Best practice tip |
| `[SCENARIO]` | Real-world scenario |

---

## Complete Table of Contents

### Unit 1: SRE Fundamentals
| # | Chapter | File |
|---|---------|------|
| 1.1 | What is SRE? | [01-what-is-sre.md](01-SRE-Fundamentals/01-what-is-sre.md) |
| 1.2 | SRE vs DevOps | [02-sre-vs-devops.md](01-SRE-Fundamentals/02-sre-vs-devops.md) |
| 1.3 | SRE Principles | [03-sre-principles.md](01-SRE-Fundamentals/03-sre-principles.md) |
| 1.4 | Google's SRE Model | [04-googles-sre-model.md](01-SRE-Fundamentals/04-googles-sre-model.md) |
| 1.5 | SRE Team Structure | [05-sre-team-structure.md](01-SRE-Fundamentals/05-sre-team-structure.md) |

### Unit 2: Service Level Objectives
| # | Chapter | File |
|---|---------|------|
| 2.1 | SLIs (Service Level Indicators) | [01-slis.md](02-Service-Level-Objectives/01-slis.md) |
| 2.2 | SLOs (Service Level Objectives) | [02-slos.md](02-Service-Level-Objectives/02-slos.md) |
| 2.3 | SLAs (Service Level Agreements) | [03-slas.md](02-Service-Level-Objectives/03-slas.md) |
| 2.4 | Choosing SLIs | [04-choosing-slis.md](02-Service-Level-Objectives/04-choosing-slis.md) |
| 2.5 | Setting SLO Targets | [05-setting-slo-targets.md](02-Service-Level-Objectives/05-setting-slo-targets.md) |
| 2.6 | Error Budgets | [06-error-budgets.md](02-Service-Level-Objectives/06-error-budgets.md) |

### Unit 3: Reliability
| # | Chapter | File |
|---|---------|------|
| 3.1 | Availability Calculations | [01-availability-calculations.md](03-Reliability/01-availability-calculations.md) |
| 3.2 | Measuring Reliability | [02-measuring-reliability.md](03-Reliability/02-measuring-reliability.md) |
| 3.3 | Risk Assessment | [03-risk-assessment.md](03-Reliability/03-risk-assessment.md) |
| 3.4 | Failure Modes | [04-failure-modes.md](03-Reliability/04-failure-modes.md) |
| 3.5 | Designing for Reliability | [05-designing-for-reliability.md](03-Reliability/05-designing-for-reliability.md) |
| 3.6 | Redundancy and Replication | [06-redundancy-and-replication.md](03-Reliability/06-redundancy-and-replication.md) |

### Unit 4: Incident Management
| # | Chapter | File |
|---|---------|------|
| 4.1 | Incident Classification | [01-incident-classification.md](04-Incident-Management/01-incident-classification.md) |
| 4.2 | Incident Response Process | [02-incident-response-process.md](04-Incident-Management/02-incident-response-process.md) |
| 4.3 | On-Call Management | [03-on-call-management.md](04-Incident-Management/03-on-call-management.md) |
| 4.4 | Postmortems / RCAs | [04-postmortems-rcas.md](04-Incident-Management/04-postmortems-rcas.md) |
| 4.5 | Blameless Culture | [05-blameless-culture.md](04-Incident-Management/05-blameless-culture.md) |
| 4.6 | Learning from Failures | [06-learning-from-failures.md](04-Incident-Management/06-learning-from-failures.md) |

### Unit 5: Capacity Planning
| # | Chapter | File |
|---|---------|------|
| 5.1 | Capacity Forecasting | [01-capacity-forecasting.md](05-Capacity-Planning/01-capacity-forecasting.md) |
| 5.2 | Load Testing | [02-load-testing.md](05-Capacity-Planning/02-load-testing.md) |
| 5.3 | Performance Modeling | [03-performance-modeling.md](05-Capacity-Planning/03-performance-modeling.md) |
| 5.4 | Scaling Strategies | [04-scaling-strategies.md](05-Capacity-Planning/04-scaling-strategies.md) |
| 5.5 | Cost Optimization | [05-cost-optimization.md](05-Capacity-Planning/05-cost-optimization.md) |

### Unit 6: Automation and Toil
| # | Chapter | File |
|---|---------|------|
| 6.1 | What is Toil? | [01-what-is-toil.md](06-Automation-and-Toil/01-what-is-toil.md) |
| 6.2 | Measuring Toil | [02-measuring-toil.md](06-Automation-and-Toil/02-measuring-toil.md) |
| 6.3 | Automation Strategies | [03-automation-strategies.md](06-Automation-and-Toil/03-automation-strategies.md) |
| 6.4 | Reducing Operational Burden | [04-reducing-operational-burden.md](06-Automation-and-Toil/04-reducing-operational-burden.md) |
| 6.5 | Self-Healing Systems | [05-self-healing-systems.md](06-Automation-and-Toil/05-self-healing-systems.md) |
| 6.6 | Automation ROI | [06-automation-roi.md](06-Automation-and-Toil/06-automation-roi.md) |

### Unit 7: Release Engineering
| # | Chapter | File |
|---|---------|------|
| 7.1 | Release Management | [01-release-management.md](07-Release-Engineering/01-release-management.md) |
| 7.2 | Progressive Rollouts | [02-progressive-rollouts.md](07-Release-Engineering/02-progressive-rollouts.md) |
| 7.3 | Canarying | [03-canarying.md](07-Release-Engineering/03-canarying.md) |
| 7.4 | Feature Flags | [04-feature-flags.md](07-Release-Engineering/04-feature-flags.md) |
| 7.5 | Rollback Strategies | [05-rollback-strategies.md](07-Release-Engineering/05-rollback-strategies.md) |
| 7.6 | Change Management | [06-change-management.md](07-Release-Engineering/06-change-management.md) |

### Unit 8: System Design
| # | Chapter | File |
|---|---------|------|
| 8.1 | Scalability Patterns | [01-scalability-patterns.md](08-System-Design/01-scalability-patterns.md) |
| 8.2 | High Availability Patterns | [02-high-availability-patterns.md](08-System-Design/02-high-availability-patterns.md) |
| 8.3 | Disaster Recovery | [03-disaster-recovery.md](08-System-Design/03-disaster-recovery.md) |
| 8.4 | Data Integrity | [04-data-integrity.md](08-System-Design/04-data-integrity.md) |
| 8.5 | Service Architecture | [05-service-architecture.md](08-System-Design/05-service-architecture.md) |

---

## Quick Reference: Key Formulas

```
Availability = Uptime / (Uptime + Downtime)

Error Budget = 1 - SLO
  e.g., 99.9% SLO → 0.1% error budget → ~43.2 min/month

MTTR = Total Downtime / Number of Incidents
MTBF = Total Uptime / Number of Failures
MTTF = Total Operating Time / Number of Failures

Availability = MTBF / (MTBF + MTTR)

Nines of Availability:
  99%     = 3.65 days/year downtime     ("two nines")
  99.9%   = 8.76 hours/year downtime    ("three nines")
  99.95%  = 4.38 hours/year downtime
  99.99%  = 52.6 minutes/year downtime  ("four nines")
  99.999% = 5.26 minutes/year downtime  ("five nines")
```

---

## Recommended Reading

| Book | Author |
|------|--------|
| *Site Reliability Engineering* | Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Murphy |
| *The Site Reliability Workbook* | Betsy Beyer, Niall Murphy, David Rensin, et al. |
| *Seeking SRE* | David Blank-Edelman |
| *Implementing Service Level Objectives* | Alex Hidalgo |
| *The Phoenix Project* | Gene Kim, Kevin Behr, George Spafford |

---

*Last updated: February 2026*
