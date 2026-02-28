# Chapter 1.2: SRE vs DevOps

[← Previous: What is SRE?](01-what-is-sre.md) | [Back to README](../README.md) | [Next: SRE Principles →](03-sre-principles.md)

---

## Overview

SRE and DevOps are often mentioned together, but they are different approaches to the same fundamental problem — how to deliver software quickly while maintaining system reliability. DevOps is a broad cultural and philosophical movement; SRE is a concrete, opinionated implementation of those ideas.

> **"Class SRE implements interface DevOps"**
> — Google's way of framing the relationship

---

## 1. DevOps — The Philosophy

DevOps emerged around 2008-2009 as a movement to break down silos between Development and Operations teams.

```
  BEFORE DEVOPS                         WITH DEVOPS
  ┌─────────┐    Wall of     ┌─────┐   ┌─────────────────────┐
  │   Dev   │   Confusion    │ Ops │   │   Dev + Ops = DevOps│
  │         │ ──────────►    │     │   │                     │
  │ "Works  │  "You deal     │"Not │   │  Shared ownership   │
  │  on my  │   with it"     │ my  │   │  Shared tooling     │
  │  machine│                │code"│   │  Shared goals       │
  └─────────┘                └─────┘   └─────────────────────┘
```

### DevOps Core Principles (CALMS)

| Pillar | Description |
|--------|-------------|
| **C**ulture | Break silos, shared responsibility |
| **A**utomation | Automate build, test, deploy, infra |
| **L**ean | Reduce waste, small batch sizes |
| **M**easurement | Measure everything, data-driven decisions |
| **S**haring | Share knowledge, tools, and practices |

---

## 2. SRE — The Implementation

SRE provides **specific, prescriptive practices** for achieving DevOps goals.

```
  ┌──────────────────────────────────────────────┐
  │                  DevOps                      │
  │           (Philosophy / Culture)             │
  │                                              │
  │    ┌──────────────────────────────────┐      │
  │    │            SRE                   │      │
  │    │    (Concrete Implementation)     │      │
  │    │                                  │      │
  │    │  - Error Budgets                 │      │
  │    │  - SLOs / SLIs / SLAs           │      │
  │    │  - 50% Toil Cap                 │      │
  │    │  - Blameless Postmortems        │      │
  │    │  - On-call Engineering          │      │
  │    │  - Automation-first             │      │
  │    └──────────────────────────────────┘      │
  │                                              │
  └──────────────────────────────────────────────┘
```

---

## 3. Side-by-Side Comparison

```
  ┌──────────────────────────────────────────────────────────────┐
  │                    DevOps vs SRE                             │
  ├──────────────────────┬───────────────────────────────────────┤
  │      DevOps          │              SRE                     │
  ├──────────────────────┼───────────────────────────────────────┤
  │  Philosophy          │  Discipline / Job Role               │
  │  "Break down silos"  │  "Software eng for operations"       │
  │  CI/CD focused       │  Reliability focused                 │
  │  General practices   │  Specific practices (SLOs, budgets)  │
  │  Any team can adopt  │  Dedicated SRE team / role           │
  │  Broad community     │  Originated at Google                │
  │  Cultural movement   │  Prescriptive framework              │
  └──────────────────────┴───────────────────────────────────────┘
```

### Detailed Comparison Table

| Aspect | DevOps | SRE |
|--------|--------|-----|
| **Origin** | Community movement (~2008) | Google (~2003) |
| **Nature** | Philosophy / culture | Engineering discipline |
| **Reliability Target** | "Improve reliability" (qualitative) | SLOs with error budgets (quantitative) |
| **Failure approach** | "Shift left," prevent bugs | Error budgets — accept controlled failure |
| **Change management** | CI/CD pipelines | Progressive rollouts, canarying |
| **Ops work** | Automate ops | Cap toil at 50%, engineer the rest |
| **On-call** | Shared on-call (varies) | Structured on-call with compensation |
| **Incident review** | Retrospectives | Blameless postmortems with action items |
| **Team model** | Embedded or platform teams | Dedicated SRE team or embedded SREs |
| **Measurement** | DORA metrics (deploy freq, lead time) | SLIs, SLOs, error budgets |
| **Scaling** | "Automate everything" | "Automate toil, engineer reliability" |

---

## 4. Where They Align

DevOps and SRE share several fundamental beliefs:

```
  DevOps Principle              SRE Equivalent
  ─────────────────────         ─────────────────────────
  Reduce silos             ──►  Share ownership of
                                production with devs

  Accept failure           ──►  Error budgets &
  as normal                     blameless postmortems

  Implement gradual        ──►  Canaries, progressive
  change                        rollouts, feature flags

  Leverage tooling &       ──►  Eliminate toil,
  automation                    automate ops tasks

  Measure everything       ──►  SLIs feed SLOs,
                                data-driven decisions
```

---

## 5. Where They Diverge

### 5.1 Prescriptiveness

- **DevOps** says *"reduce organizational silos"* but doesn't tell you **how**
- **SRE** says *"use error budgets to align dev and ops incentives — when the budget runs out, stop deploying"*

### 5.2 Team Structure

```
  DevOps Model (varies widely):
  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │  Team A  │  │  Team B  │  │  Team C  │
  │ Dev+Ops  │  │ Dev+Ops  │  │ Dev+Ops  │
  │ combined │  │ combined │  │ combined │
  └──────────┘  └──────────┘  └──────────┘

  SRE Model:
  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │  Dev     │  │  Dev     │  │  Dev     │
  │  Team A  │  │  Team B  │  │  Team C  │
  └─────┬────┘  └─────┬────┘  └─────┬────┘
        │              │              │
        └──────────────┼──────────────┘
                       │
               ┌───────▼───────┐
               │   SRE Team    │
               │  (shared or   │
               │   embedded)   │
               └───────────────┘
```

### 5.3 Risk Quantification

- **DevOps**: "Move fast, but be careful"
- **SRE**: "Move fast — you have exactly 0.05% of requests that can fail this quarter"

---

## 6. Real-World Scenario

### [SCENARIO] Choosing Between DevOps and SRE

**Company Profile:** Mid-size SaaS startup, 200 engineers, growing fast.

| Phase | Approach | Why |
|-------|----------|-----|
| **Early stage** (20 devs) | DevOps | Small team, everyone owns everything. CI/CD + cloud-native is enough. |
| **Growth stage** (50-100) | DevOps + SRE principles | Adopt SLOs and error budgets. On-call structure needed. |
| **Scale stage** (200+) | Dedicated SRE team | Dedicated SREs for critical services. Platform team for shared tooling. |

**Key insight:** You don't have to choose one over the other. Most mature organizations practice DevOps-wide and add SRE for critical systems.

---

## 7. Common DevOps Metrics vs SRE Metrics

```
  DevOps (DORA Metrics)                 SRE Metrics
  ┌─────────────────────────┐          ┌────────────────────────┐
  │ Deployment Frequency    │          │ SLI/SLO Compliance     │
  │ Lead Time for Changes   │          │ Error Budget Remaining │
  │ Change Failure Rate     │          │ Toil Percentage        │
  │ MTTR (Mean Time to      │          │ MTTR / MTTD / MTTF    │
  │   Restore)              │          │ Incident Frequency     │
  └─────────────────────────┘          │ On-call Load           │
                                       └────────────────────────┘
```

---

## 8. Can You Have Both?

**Yes, absolutely.** In fact, the best organizations do:

```
  ┌────────────────────────────────────────────────────┐
  │              Organization-wide DevOps              │
  │                                                    │
  │  ┌─────────────┐  ┌─────────────┐  ┌────────────┐│
  │  │ Feature     │  │ Feature     │  │ Feature    ││
  │  │ Team A      │  │ Team B      │  │ Team C     ││
  │  │ (DevOps)    │  │ (DevOps)    │  │ (DevOps)   ││
  │  └──────┬──────┘  └──────┬──────┘  └─────┬──────┘│
  │         │                │                │       │
  │         ▼                ▼                ▼       │
  │  ┌────────────────────────────────────────────┐   │
  │  │          SRE Team (Critical Services)      │   │
  │  │  - SLOs & Error Budgets for Tier-1 services│   │
  │  │  - On-call for production                  │   │
  │  │  - Reliability reviews for launches        │   │
  │  └────────────────────────────────────────────┘   │
  │                                                    │
  │  ┌────────────────────────────────────────────┐   │
  │  │      Platform / Infra Team                 │   │
  │  │  - CI/CD pipelines                         │   │
  │  │  - Kubernetes clusters                     │   │
  │  │  - Observability stack                     │   │
  │  └────────────────────────────────────────────┘   │
  └────────────────────────────────────────────────────┘
```

---

## 9. Troubleshooting Tips

| Issue | Resolution |
|-------|------------|
| "We renamed Ops to SRE but nothing changed" | SRE requires the 50% engineering mandate and SLOs — not just a title change |
| "DevOps and SRE teams conflict" | Align on shared SLOs. SRE provides the reliability framework; DevOps provides the culture and toolchain |
| "Developers resist SRE involvement" | Frame SRE as enabling faster, safer deployments — not as gatekeepers |
| "Which one should we adopt?" | Start with DevOps culture in all teams. Add SRE practices for services where reliability is critical |

---

## Summary Table

| Concept | DevOps | SRE |
|---------|--------|-----|
| **Type** | Cultural movement | Engineering discipline |
| **Focus** | Speed of delivery | Reliability of service |
| **Failure** | Prevent and shift left | Accept, quantify via error budgets |
| **Metrics** | DORA (deploy frequency, lead time) | SLIs, SLOs, error budgets |
| **Ops Work** | Automate | Cap at 50% and engineer away |
| **Postmortems** | Encouraged | Mandatory and blameless |
| **Relationship** | Broad interface | Concrete implementation |
| **Best used** | Organization-wide | Critical, high-SLA services |

---

## Quick Revision Questions

1. **How does Google describe the relationship between SRE and DevOps?**
   > "Class SRE implements interface DevOps" — SRE is a concrete, opinionated implementation of DevOps principles.

2. **What does the CALMS framework stand for in DevOps?**
   > Culture, Automation, Lean, Measurement, Sharing.

3. **Give one example where SRE is more prescriptive than DevOps.**
   > DevOps says "accept failure as normal." SRE quantifies it: "You have a 0.1% error budget this quarter" and enforces consequences when it's spent.

4. **At what organizational size does a dedicated SRE team typically make sense?**
   > Typically at 50-100+ engineers, or when you have critical services where downtime directly impacts revenue or user safety.

5. **Can an organization adopt both DevOps and SRE? How?**
   > Yes. Use DevOps culture and CI/CD across all teams, and add SRE practices (SLOs, error budgets, on-call) for critical Tier-1 services.

6. **What are the DORA metrics, and what is their SRE equivalent?**
   > DORA: deployment frequency, lead time, change failure rate, MTTR. SRE equivalents: SLI/SLO compliance, error budget burn, toil percentage, MTTD/MTTR.

---

[← Previous: What is SRE?](01-what-is-sre.md) | [Back to README](../README.md) | [Next: SRE Principles →](03-sre-principles.md)
