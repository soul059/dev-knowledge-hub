# Chapter 1.1: What is Site Reliability Engineering (SRE)?

[← Back to README](../README.md) | [Next: SRE vs DevOps →](02-sre-vs-devops.md)

---

## Overview

Site Reliability Engineering (SRE) is a discipline that incorporates aspects of software engineering and applies them to infrastructure and operations problems. The main goals are to create scalable and highly reliable software systems. SRE was born at Google around 2003 when Ben Treynor Sloss was tasked with running a production team.

> **"SRE is what happens when you ask a software engineer to design an operations team."**
> — Ben Treynor Sloss, VP of Engineering at Google

---

## 1. The Core Idea

Traditional operations teams often struggle with the tension between development velocity and system stability. SRE resolves this by treating operations as a software problem.

```
  TRADITIONAL OPS                              SRE APPROACH
  ┌──────────────┐                     ┌──────────────────────┐
  │  Manual work │                     │  Automate everything │
  │  Ticket-based│                     │  Code-driven ops     │
  │  Reactive    │                     │  Proactive + reactive│
  │  Siloed teams│                     │  Shared ownership    │
  └──────┬───────┘                     └──────────┬───────────┘
         │                                        │
         v                                        v
  ┌──────────────┐                     ┌──────────────────────┐
  │  Ops grows   │                     │  Ops work stays      │
  │  linearly    │                     │  constant or shrinks │
  │  with load   │                     │  as systems grow     │
  └──────────────┘                     └──────────────────────┘
```

---

## 2. Key Characteristics of SRE

### 2.1 Engineering Over Operations

SREs spend at most **50% of their time on operational (toil) work**. The remaining 50%+ is spent on engineering projects that improve reliability, scalability, and operational efficiency.

```
  ┌────────────────────────────────┐
  │     SRE Time Allocation        │
  ├────────────────────────────────┤
  │                                │
  │  ████████████████ Engineering  │  ≥ 50%
  │  (automation, tooling, design) │
  │                                │
  │  ░░░░░░░░░░░░░░░░ Operations  │  ≤ 50%
  │  (on-call, tickets, toil)     │
  │                                │
  └────────────────────────────────┘
```

### 2.2 Embracing Risk

- 100% reliability is neither practical nor desirable
- SRE introduces **error budgets** — a quantified amount of acceptable unreliability
- Teams can "spend" error budgets on deploying new features

### 2.3 Service Level Objectives (SLOs)

- SRE defines reliability targets through SLOs
- Decisions are driven by data (SLIs feeding into SLOs)
- Example: "99.95% of requests complete within 200ms"

### 2.4 Eliminating Toil

**Toil** = manual, repetitive, automatable, reactive, lacks enduring value

SRE teams actively measure and reduce toil to keep operational load sustainable.

### 2.5 Monitoring and Observability

```
  ┌─────────────────────────────────────────────────────────┐
  │                THREE PILLARS OF OBSERVABILITY            │
  ├──────────────────┬──────────────────┬───────────────────┤
  │     METRICS      │      LOGS        │     TRACES        │
  │                  │                  │                   │
  │  - Numeric data  │  - Event records │  - Request flow   │
  │  - Time-series   │  - Structured/   │  - Span-based     │
  │  - Aggregatable  │    unstructured  │  - Latency map    │
  │  - Prometheus,   │  - ELK, Loki,   │  - Jaeger, Zipkin │
  │    Datadog       │    Splunk        │    Tempo          │
  │                  │                  │                   │
  └──────────────────┴──────────────────┴───────────────────┘
```

### 2.6 Automation

SREs build software to manage systems rather than manually intervening. Common automation targets:

- Deployments / rollbacks
- Capacity adjustments
- Failover procedures
- Health checks and self-healing

---

## 3. SRE in Practice — Real-World Scenario

### [SCENARIO] E-Commerce Platform

A growing e-commerce company experiences frequent outages during flash sales. They hire an SRE team.

**Before SRE:**
- 3-4 major outages per quarter
- Mean Time to Recovery (MTTR): 2 hours
- DevOps team manually scaling servers

**SRE Interventions:**
1. Defined SLOs: 99.95% availability, p99 latency < 500ms
2. Implemented error budgets — froze risky releases when budget ran low
3. Automated horizontal scaling via Kubernetes HPA
4. Built runbooks and automated incident response
5. Introduced chaos engineering (game days)

**After SRE:**
- 0-1 major outages per quarter
- MTTR reduced to 15 minutes
- Auto-scaling handles 10x traffic spikes

---

## 4. SRE Core Responsibilities

```
  ┌──────────────────────────────────────────────────────┐
  │                SRE RESPONSIBILITIES                  │
  ├──────────────────────────────────────────────────────┤
  │                                                      │
  │  ┌──────────────┐   ┌──────────────┐                │
  │  │ Availability │   │  Performance │                │
  │  │ & Reliability│   │  & Latency   │                │
  │  └──────┬───────┘   └──────┬───────┘                │
  │         │                  │                         │
  │         v                  v                         │
  │  ┌──────────────┐   ┌──────────────┐                │
  │  │  Monitoring  │   │   Capacity   │                │
  │  │  & Alerting  │   │   Planning   │                │
  │  └──────┬───────┘   └──────┬───────┘                │
  │         │                  │                         │
  │         v                  v                         │
  │  ┌──────────────┐   ┌──────────────┐                │
  │  │   Incident   │   │  Change &    │                │
  │  │   Response   │   │  Release Mgmt│                │
  │  └──────┬───────┘   └──────┬───────┘                │
  │         │                  │                         │
  │         v                  v                         │
  │  ┌──────────────────────────────────┐               │
  │  │   Automation & Toil Reduction   │               │
  │  └──────────────────────────────────┘               │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

---

## 5. Where SRE Sits in an Organization

```
  ┌────────────────────────────────────────────┐
  │              CTO / VP Engineering          │
  ├──────────┬──────────────┬──────────────────┤
  │          │              │                  │
  │   Dev    │     SRE      │    Platform /    │
  │   Teams  │     Team     │    Infra Team    │
  │          │              │                  │
  │ - Build  │ - Reliability│ - Kubernetes     │
  │   features│ - SLOs      │ - CI/CD Pipelines│
  │ - Ship   │ - Incidents  │ - Cloud Infra    │
  │   code   │ - Automation │ - Networking     │
  │          │ - On-call    │                  │
  └──────────┴──────────────┴──────────────────┘
```

---

## 6. SRE Skills and Knowledge Areas

| Domain | Skills |
|--------|--------|
| **Software Engineering** | Python, Go, Java; algorithms; data structures |
| **Systems** | Linux, networking, DNS, load balancing |
| **Cloud** | AWS, GCP, Azure; IaC (Terraform, Pulumi) |
| **Containers** | Docker, Kubernetes, Helm |
| **Observability** | Prometheus, Grafana, ELK, Jaeger |
| **Incident Management** | PagerDuty, Opsgenie; runbooks |
| **Automation** | Ansible, scripting, CI/CD |
| **Databases** | SQL, NoSQL, replication, sharding |

---

## 7. Troubleshooting Tips

| Problem | Tip |
|---------|-----|
| "We don't need SRE, we have DevOps" | SRE is a *specific implementation* of DevOps principles — they complement each other |
| "SRE is just ops with a new name" | If the team spends >50% on toil, it's ops, not SRE. Ensure engineering time is protected |
| "We can't hire SREs" | Start by training existing ops/dev staff in SRE practices; adopt the principles first |
| "Management won't buy in" | Show business impact: reduced outages → higher revenue, lower customer churn |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **SRE Definition** | Applying software engineering to operations problems |
| **Origin** | Google, ~2003, Ben Treynor Sloss |
| **50% Rule** | SREs spend ≤50% time on toil, ≥50% on engineering |
| **Error Budgets** | Quantified acceptable unreliability to balance velocity and stability |
| **SLOs** | Measurable reliability targets that drive operational decisions |
| **Toil** | Manual, repetitive, automatable operational work |
| **Observability** | Metrics, Logs, Traces — the three pillars |
| **Core Goal** | Reliable, scalable systems with sustainable operations |

---

## Quick Revision Questions

1. **What is the fundamental philosophy behind SRE?**
   > Treat operations as a software engineering problem — automate wherever possible and use data-driven reliability targets.

2. **What is the "50% rule" in SRE and why does it matter?**
   > SREs should spend no more than 50% of their time on operational/toil work. This ensures continuous improvement through engineering projects.

3. **What is an error budget, and how does it balance development velocity with reliability?**
   > An error budget is the allowed amount of unreliability (1 − SLO). Teams can "spend" it on deploying features; if the budget is exhausted, feature launches are frozen until reliability improves.

4. **Name the three pillars of observability and give one tool for each.**
   > Metrics (Prometheus), Logs (ELK/Loki), Traces (Jaeger/Zipkin).

5. **How does SRE differ from traditional operations?**
   > SRE uses software engineering to automate operations, sets quantitative reliability targets (SLOs), and embraces controlled risk through error budgets — unlike traditional ops which is often manual, reactive, and grows linearly with system complexity.

6. **Why does SRE say 100% reliability is the wrong target?**
   > Users cannot distinguish between 99.999% and 100%. Pursuing 100% is infinitely expensive and prevents any changes to the system, halting innovation.

---

[← Back to README](../README.md) | [Next: SRE vs DevOps →](02-sre-vs-devops.md)
