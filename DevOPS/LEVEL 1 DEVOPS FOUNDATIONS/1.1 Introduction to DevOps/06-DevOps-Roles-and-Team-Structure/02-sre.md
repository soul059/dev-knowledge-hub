# Chapter 6.2 — Site Reliability Engineer (SRE)

[← Previous: DevOps Engineer](01-devops-engineer.md) | [Next: Platform Engineer →](03-platform-engineer.md)

---

## Overview

**Site Reliability Engineering (SRE)** is a discipline created at Google that applies software engineering principles to operations problems. An SRE's mission is to create **scalable, reliable, and efficient systems**. SRE is often described as "a specific implementation of DevOps" — the two philosophies overlap but have distinct emphases: DevOps focuses on culture and collaboration, while SRE provides concrete practices for reliability.

---

## SRE vs DevOps

```
┌──────────────────────────────────────────────────────────┐
│  SRE vs DEVOPS RELATIONSHIP                              │
│                                                          │
│                    ┌─────────────────┐                   │
│              ┌─────│    SHARED       │─────┐             │
│              │     │  ─ Automation   │     │             │
│  ┌───────────┴──┐  │  ─ Monitoring  │  ┌──┴──────────┐ │
│  │   DEVOPS     │  │  ─ IaC        │  │    SRE       │ │
│  │              │  │  ─ CI/CD      │  │              │ │
│  │ ─ Culture    │  │  ─ Blameless  │  │ ─ SLIs/SLOs  │ │
│  │ ─ Collaboration│ │  culture     │  │ ─ Error budgets││
│  │ ─ CALMS      │  │              │  │ ─ Toil budget │ │
│  │ ─ Three Ways │  │              │  │ ─ On-call     │ │
│  │ ─ Shift Left │  │              │  │ ─ Capacity    │ │
│  │              │  │              │  │   planning    │ │
│  └──────────────┘  └─────────────┘  └──────────────┘  │
│                                                          │
│  "class SRE implements DevOps" — Seth Vargo, Google      │
└──────────────────────────────────────────────────────────┘
```

| Aspect | DevOps | SRE |
|--------|--------|-----|
| **Origin** | Community-driven movement | Google (2003, Ben Treynor) |
| **Focus** | Culture + collaboration | Reliability + engineering |
| **Metrics** | DORA metrics | SLOs + Error Budgets |
| **Failure approach** | Blameless postmortems | Error budget policy |
| **Balance** | Speed + Quality | Reliability + Innovation |
| **Team model** | Embedded or enabling | Dedicated SRE team |

---

## Core SRE Principles

```
┌──────────────────────────────────────────────────────────┐
│  GOOGLE'S 7 SRE PRINCIPLES                              │
│                                                          │
│  1. EMBRACE RISK                                         │
│     100% reliability is wrong target. Balance reliability │
│     with feature velocity using error budgets.           │
│                                                          │
│  2. SERVICE LEVEL OBJECTIVES (SLOs)                      │
│     Define reliability targets. Measure with SLIs.       │
│     SLOs drive engineering decisions.                    │
│                                                          │
│  3. ELIMINATE TOIL                                       │
│     Toil = manual, repetitive, automatable work.         │
│     Cap toil at 50% of SRE time.                         │
│                                                          │
│  4. MONITORING & ALERTING                                │
│     Alerts should be actionable. No alert fatigue.       │
│     Three outputs: pages, tickets, logs.                 │
│                                                          │
│  5. RELEASE ENGINEERING                                  │
│     Safe, reliable, frequent releases.                   │
│     Canary deployments, progressive rollouts.            │
│                                                          │
│  6. SIMPLICITY                                           │
│     "Boring" is good. Simple systems fail less.          │
│     Remove unnecessary complexity.                       │
│                                                          │
│  7. ENGINEERING APPROACH                                 │
│     Treat operations as a software problem.              │
│     Write code to automate away operational work.        │
└──────────────────────────────────────────────────────────┘
```

---

## Toil Management

```
┌──────────────────────────────────────────────────────────┐
│  TOIL vs ENGINEERING WORK                                │
│                                                          │
│  TOIL (reduce to < 50%):           ENGINEERING (> 50%): │
│  ├── Manual server restarts        ├── Build auto-healing│
│  ├── Ticket-based access grants    ├── Self-service IAM  │
│  ├── Copy-paste config changes     ├── Config automation │
│  ├── Manual capacity adjustments   ├── Auto-scaling      │
│  ├── Repetitive incident response  ├── Runbook automation│
│  └── Manual certificate renewal    └── Cert-manager      │
│                                                          │
│  TOIL CHARACTERISTICS:                                   │
│  ├── Manual — requires human intervention                │
│  ├── Repetitive — done over and over                     │
│  ├── Automatable — could be done by software             │
│  ├── Tactical — interrupt-driven, reactive               │
│  ├── No lasting value — doesn't improve the system       │
│  └── Scales linearly — grows with service size           │
│                                                          │
│  RULE: If you do it twice, automate it the third time.   │
└──────────────────────────────────────────────────────────┘
```

---

## SRE On-Call Structure

```
┌──────────────────────────────────────────────────────────┐
│  ON-CALL BEST PRACTICES                                  │
│                                                          │
│  ROTATION:                                               │
│  ├── Minimum 8 people in rotation                        │
│  ├── Maximum 2 incidents per 12-hour shift               │
│  ├── Follow-the-sun model for global teams               │
│  └── 25% engineering time spent on-call (max)            │
│                                                          │
│  INCIDENT RESPONSE FLOW:                                 │
│                                                          │
│  Alert Fires                                             │
│     │                                                    │
│     ▼                                                    │
│  Acknowledge (< 5 min)                                   │
│     │                                                    │
│     ▼                                                    │
│  Assess Severity (P1-P4)                                 │
│     │                                                    │
│     ├──[P1/P2]──► Incident Commander assigned            │
│     │              ├── War room opened                   │
│     │              ├── Stakeholders notified              │
│     │              └── All hands on deck                 │
│     │                                                    │
│     └──[P3/P4]──► On-call resolves or creates ticket     │
│                                                          │
│  POST-INCIDENT:                                          │
│  ├── Blameless postmortem within 48 hours                │
│  ├── Action items tracked to completion                  │
│  └── Learnings shared with wider team                    │
└──────────────────────────────────────────────────────────┘
```

---

## SRE Error Budget Policy (Example)

```yaml
# error-budget-policy.yaml
service: payment-api
slo_target: 99.95%   # monthly

policy:
  budget_healthy:        # > 50% remaining
    actions:
      - "Feature development proceeds normally"
      - "SRE focuses on automation and tooling"
      - "Experiment with new deployment strategies"

  budget_warning:        # 20-50% remaining
    actions:
      - "Increase code review rigor"
      - "Add more integration tests before deploy"
      - "SRE reviews all production changes"

  budget_critical:       # 5-20% remaining
    actions:
      - "Only critical fixes and reliability work"
      - "Rollback non-essential recent changes"
      - "Daily SRE-dev sync on stability"

  budget_exhausted:      # 0% remaining
    actions:
      - "Feature freeze until budget regenerates"
      - "All engineering effort on reliability"
      - "Post-incident review of budget burn"
      - "Leadership briefing on recovery plan"

  review_cadence: "Weekly SRE-dev error budget review"
```

---

## SRE Engagement Models

```
┌──────────────────────────────────────────────────────────┐
│  SRE TEAM ENGAGEMENT MODELS                              │
│                                                          │
│  MODEL 1: EMBEDDED SRE                                   │
│  ┌─────────────────────────┐                             │
│  │  Product Team A          │                             │
│  │  ├── 6 Developers        │                             │
│  │  └── 1 SRE (embedded)   │  ← SRE is part of team     │
│  └─────────────────────────┘                             │
│  Best for: Critical services needing deep context        │
│                                                          │
│  MODEL 2: CONSULTING SRE                                 │
│  ┌──────────┐  reviews   ┌──────────┐                   │
│  │ SRE Team │──────────►│ Team A    │                   │
│  │          │──────────►│ Team B    │                   │
│  │          │──────────►│ Team C    │                   │
│  └──────────┘ advises    └──────────┘                   │
│  Best for: Many teams, limited SRE headcount             │
│                                                          │
│  MODEL 3: PLATFORM SRE                                   │
│  ┌──────────────────┐                                    │
│  │ SRE Platform Team │ ← Builds reliability tooling     │
│  │ ├── SLO dashboards│                                   │
│  │ ├── Alerting rules│                                   │
│  │ └── Runbook engine│                                   │
│  └────────┬─────────┘                                    │
│           │ self-service                                 │
│     ┌─────┼─────┐                                       │
│     ▼     ▼     ▼                                       │
│  Team A  Team B  Team C  ← Teams use SRE platform       │
│  Best for: Large orgs scaling SRE practices              │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: SRE Transformation

```
SCENARIO: E-commerce platform — 99.5% availability (too many outages)

BEFORE:
├── 4-5 P1 incidents per month
├── MTTR: 4+ hours (manual investigation)
├── On-call: 2 overworked ops engineers
├── No SLOs — "just keep it running"
└── Every deploy is scary → monthly deploys

SRE TRANSFORMATION:

Phase 1 — Define SLOs (Month 1):
├── SLI: Request success rate (non-5xx / total)
├── SLO: 99.9% success rate (monthly)
├── Error Budget: 0.1% = 43.8 minutes/month
└── Dashboard showing real-time budget consumption

Phase 2 — Reduce Toil (Month 2-3):
├── Automated runbooks for top 5 incident types
├── Self-healing: auto-restart crashed pods
├── Auto-scaling based on traffic patterns
└── Result: 60% reduction in pages

Phase 3 — Improve On-Call (Month 4):
├── Grew rotation to 6 people
├── Actionable alerts only (retired 40% of alerts)
├── Created decision trees for common incidents
└── Result: < 2 incidents per on-call shift

Phase 4 — Release Engineering (Month 5-6):
├── Canary deployments (5% → 25% → 100%)
├── Automated rollback on error rate spike
├── Feature flags for risky changes
└── Result: Weekly deploys → daily deploys

AFTER:
├── Availability: 99.95% (from 99.5%)
├── P1 incidents: 0-1 per month (from 4-5)
├── MTTR: 15 minutes (from 4 hours)
├── Deploy frequency: Daily (from monthly)
└── On-call burden: Sustainable and fair
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Alert fatigue | Too many non-actionable alerts | Review all alerts; delete or auto-resolve noisy ones |
| SRE team is bottleneck | All prod changes go through SRE | Empower dev teams; SRE enables, not gatekeeps |
| Error budget always exhausted | SLO too aggressive or system too flaky | Reassess SLO; invest in reliability before raising target |
| SRE doing only ops work | No toil budget enforcement | Enforce 50% engineering time; track toil metrics |
| Dev teams ignore SLOs | No consequences for budget burn | Implement error budget policy with graduated responses |
| Postmortems not happening | No process or follow-through | Schedule within 48hrs; assign action item owners |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **SRE** | Applies software engineering to operations problems |
| **SRE vs DevOps** | "class SRE implements DevOps" — concrete practices for reliability |
| **SLOs** | Reliability targets that drive engineering decisions |
| **Error Budget** | Allowed unreliability; balances features vs reliability |
| **Toil** | Manual, repetitive, automatable work — cap at 50% |
| **On-Call** | Structured rotation with actionable alerts |
| **Engagement Models** | Embedded, Consulting, or Platform SRE |
| **Postmortems** | Blameless learning from incidents |

---

## Quick Revision Questions

1. **What is SRE and how does it relate to DevOps?**
   <details><summary>Answer</summary>SRE (Site Reliability Engineering) is a discipline created at Google that applies software engineering principles to operations problems. It's often described as "a specific implementation of DevOps" — DevOps provides the philosophy (culture, collaboration, automation), while SRE provides concrete practices (SLOs, error budgets, toil management, on-call). As Seth Vargo put it: "class SRE implements DevOps."</details>

2. **What are Google's core SRE principles?**
   <details><summary>Answer</summary>Seven principles: 1) Embrace risk — 100% reliability is wrong target. 2) Service Level Objectives — define and measure reliability. 3) Eliminate toil — cap manual work at 50%. 4) Monitoring & alerting — actionable alerts only. 5) Release engineering — safe, frequent releases. 6) Simplicity — simple systems fail less. 7) Engineering approach — treat operations as a software problem.</details>

3. **What is toil and why is it important to manage?**
   <details><summary>Answer</summary>Toil is manual, repetitive, automatable, tactical work that produces no lasting value and scales linearly with service size (e.g., manual restarts, ticket-based access grants). SRE caps toil at 50% of time because: unmanaged toil grows until 100% of time is wasted on repetitive work, leaving no time for engineering improvements. The rule: "If you do it twice, automate it the third time."</details>

4. **What are the three SRE engagement models?**
   <details><summary>Answer</summary>1) Embedded SRE — an SRE joins a product team directly, best for critical services needing deep context. 2) Consulting SRE — a central SRE team advises multiple product teams, best when SRE headcount is limited. 3) Platform SRE — builds reliability tooling (SLO dashboards, alerting, runbooks) as self-service for all teams, best for large organizations scaling SRE practices.</details>

5. **How does the error budget policy work in practice?**
   <details><summary>Answer</summary>The error budget policy defines graduated responses based on remaining budget: Healthy (>50%): normal feature development. Warning (20-50%): increased review rigor and testing. Critical (5-20%): only critical fixes, rollback non-essential changes. Exhausted (0%): feature freeze until budget regenerates. Weekly SRE-dev reviews assess budget status and adjust plans accordingly.</details>

6. **What makes a good on-call practice?**
   <details><summary>Answer</summary>Key practices: 1) Minimum 8 people in rotation. 2) Max 2 incidents per 12-hour shift. 3) Follow-the-sun for global teams. 4) Max 25% engineering time on-call. 5) Only actionable alerts (no alert fatigue). 6) Clear incident severity levels (P1-P4). 7) Incident commander for P1/P2. 8) Blameless postmortem within 48 hours. 9) Action items tracked to completion.</details>

---

[← Previous: DevOps Engineer](01-devops-engineer.md) | [Next: Platform Engineer →](03-platform-engineer.md)
