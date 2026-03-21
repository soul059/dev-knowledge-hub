# Business Impact Analysis

## Unit 5 - Topic 6 | Risk Management

---

## Overview

A **Business Impact Analysis (BIA)** identifies and evaluates the potential effects of an interruption to critical business operations. It determines which systems and processes are most important and defines recovery priorities, objectives, and resource requirements.

---

## 1. What is BIA?

```
┌──────────────────────────────────────────────────────────────────┐
│                  BUSINESS IMPACT ANALYSIS                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BIA answers:                                                    │
│                                                                  │
│  "If this system/process goes down..."                           │
│  ├── How BADLY does it hurt the business?                        │
│  ├── How QUICKLY must we recover it?                             │
│  ├── How much DATA LOSS can we tolerate?                         │
│  └── What RESOURCES are needed for recovery?                     │
│                                                                  │
│  BIA is the FOUNDATION of:                                       │
│  • Business Continuity Planning (BCP)                            │
│  • Disaster Recovery Planning (DRP)                              │
│  • Incident Response prioritization                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Key BIA Metrics

| Metric | Definition | Example |
|--------|-----------|---------|
| **RTO** (Recovery Time Objective) | Maximum acceptable downtime | "Email must be back within 4 hours" |
| **RPO** (Recovery Point Objective) | Maximum acceptable data loss | "Can lose up to 1 hour of data" |
| **MTPD** (Maximum Tolerable Period of Disruption) | Absolute max downtime before catastrophic damage | "Beyond 24 hours, we lose major clients" |
| **MTTR** (Mean Time To Repair) | Average time to fix and restore | "Usually takes 2 hours to restore" |
| **MTBF** (Mean Time Between Failures) | Average time between system failures | "Server fails approximately once a year" |

### RTO vs RPO Visualization

```
              DATA LOSS (RPO)           DOWNTIME (RTO)
          ◄──────────────────── ──────────────────────►
          │                    │                       │
──────────┤                    ├───────────────────────┤──────────
          │                    │                       │
     Last Backup           DISASTER              System Back
     (or sync point)       OCCURS                Online

RPO: How far BACK can we go? (data loss tolerance)
RTO: How QUICKLY must we recover? (downtime tolerance)
```

---

## 3. BIA Process

```
Step 1: IDENTIFY CRITICAL PROCESSES
   └── List all business processes and rank by importance

Step 2: ASSESS IMPACT OF DISRUPTION
   └── Financial, operational, legal, reputational impact over time

Step 3: DETERMINE RECOVERY PRIORITIES
   └── Which processes must be restored first?

Step 4: DEFINE RTO AND RPO
   └── Set recovery objectives for each critical process

Step 5: IDENTIFY DEPENDENCIES
   └── What systems, people, vendors does each process depend on?

Step 6: DETERMINE RESOURCE REQUIREMENTS
   └── What's needed for recovery? (people, technology, facilities)

Step 7: DOCUMENT AND VALIDATE
   └── Create BIA report, validate with stakeholders
```

---

## 4. Impact Categories

| Category | Description | Example |
|----------|-------------|---------|
| **Financial** | Revenue loss, fines, recovery costs | $50K/hour downtime cost |
| **Operational** | Inability to deliver services | Can't process orders |
| **Legal/Regulatory** | Compliance violations, lawsuits | HIPAA breach notification |
| **Reputational** | Customer trust, brand damage | News coverage of breach |
| **Health & Safety** | Physical harm to people | Hospital systems offline |
| **Contractual** | SLA violations | Penalty clauses triggered |

### Impact Over Time

```
Impact Severity
     ▲
     │                                    ╱
     │                               ╱
     │                          ╱
     │                     ╱         ← Impact increases
     │                ╱                 over time!
     │           ╱
     │      ╱
     │ ╱
     └──────────────────────────────────────► Time
     0    1hr   4hr   8hr   24hr  48hr  1wk

     Critical processes: Impact is immediate
     Important processes: Impact grows gradually
     Low-priority: Impact is minimal even after days
```

---

## 5. BIA Example Table

| Process | Criticality | Financial Impact/hr | RTO | RPO | Dependencies |
|---------|:-:|--------|-----|-----|---|
| Online Payment System | 🔴 Critical | $100K | 1 hour | 0 (zero loss) | Database, network, payment gateway |
| Customer Email | 🟠 High | $10K | 4 hours | 1 hour | Mail server, DNS, Internet |
| Internal Wiki | 🟡 Medium | $500 | 24 hours | 24 hours | Web server |
| Marketing Blog | 🟢 Low | $50 | 72 hours | 48 hours | CMS, CDN |

---

## 6. BIA → BCP → DRP Relationship

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  BIA (Business Impact Analysis)                              │
│  "What's critical and what are the recovery objectives?"     │
│              │                                               │
│              ▼                                               │
│  BCP (Business Continuity Plan)                              │
│  "How do we keep operating during a disruption?"             │
│  (People, processes, alternative sites)                      │
│              │                                               │
│              ▼                                               │
│  DRP (Disaster Recovery Plan)                                │
│  "How do we restore IT systems after a disaster?"            │
│  (Backups, failover, recovery procedures)                    │
│                                                              │
│  BIA feeds into BOTH BCP and DRP.                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **BIA** | Identifies critical processes and recovery priorities |
| **RTO** | Maximum acceptable downtime |
| **RPO** | Maximum acceptable data loss |
| **Impact grows** | Over time — the longer the outage, the worse it gets |
| **Feeds into** | Business Continuity Plan (BCP) and Disaster Recovery Plan (DRP) |
| **Frequency** | Should be updated annually or after major changes |

---

## Quick Revision Questions

1. **What is a Business Impact Analysis and why is it important?**
2. **What is the difference between RTO and RPO?**
3. **If RPO = 4 hours, what does that mean for your backup strategy?**
4. **List 5 impact categories that BIA evaluates.**
5. **How does BIA relate to BCP and DRP?**
6. **For an online banking system, what would you set as RTO and RPO?**

---

[← Previous: Risk Treatment Options](05-risk-treatment-options.md)

---

*Unit 5 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Security Policies →](../06-Security-Policies/01-policy-types.md)*
