# Chapter 2.1 — CALMS Framework

[← Previous: Benefits and Challenges](../01-What-is-DevOps/05-benefits-and-challenges.md) | [Next: The Three Ways →](02-three-ways.md)

---

## Overview

The **CALMS** framework, coined by Jez Humble, is a widely used model for assessing and guiding DevOps adoption. It stands for **Culture, Automation, Lean, Measurement, and Sharing**. Each pillar represents a critical dimension that organizations must address.

---

## The CALMS Framework at a Glance

```
┌─────────────────────────────────────────────────────────┐
│                  CALMS FRAMEWORK                        │
│                                                         │
│   ┌───────┐ ┌───────────┐ ┌──────┐ ┌───────────┐ ┌───────┐
│   │       │ │           │ │      │ │           │ │       │
│   │   C   │ │     A     │ │  L   │ │     M     │ │   S   │
│   │       │ │           │ │      │ │           │ │       │
│   │Culture│ │Automation │ │ Lean │ │Measurement│ │Sharing│
│   │       │ │           │ │      │ │           │ │       │
│   └───┬───┘ └─────┬─────┘ └──┬───┘ └─────┬─────┘ └───┬───┘
│       │           │          │            │           │
│       ▼           ▼          ▼            ▼           ▼
│   Collab-     Automate    Reduce     Data-driven  Knowledge
│   ration &   repetitive  waste &    decisions    transfer &
│   trust      tasks       batch size              transparency
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## C — Culture

Culture is the foundation. Without the right culture, tools and processes will not deliver results.

```
┌──────────────────────────────────────────────────────┐
│  CULTURE                                             │
│  ═══════                                             │
│                                                      │
│  Core Values:                                        │
│  ┌─────────────────────────────────────────────┐     │
│  │  Trust          → Teams trust each other    │     │
│  │  Ownership      → "You build it, you run it"│     │
│  │  Blamelessness  → Learn from failures       │     │
│  │  Transparency   → Open metrics & processes  │     │
│  │  Collaboration  → Cross-functional teams    │     │
│  └─────────────────────────────────────────────┘     │
│                                                      │
│  Assessment Questions:                               │
│  ├── Do teams share goals and metrics?               │
│  ├── Are postmortems blameless?                      │
│  ├── Can anyone see the deployment pipeline?         │
│  └── Do developers participate in on-call?           │
└──────────────────────────────────────────────────────┘
```

**Practical Example — Building Culture:**
```
# Slack channel structure for collaboration
#team-platform          → Cross-functional team discussions
#incidents              → Real-time incident coordination
#postmortems            → Shared postmortem documents
#deployments            → Deployment notifications (automated)
#tech-talks             → Weekly knowledge sharing
```

---

## A — Automation

Automate everything that is repetitive, error-prone, or time-consuming.

```
┌──────────────────────────────────────────────────────┐
│  AUTOMATION LANDSCAPE                                │
│  ════════════════════                                │
│                                                      │
│  Code ──► Build ──► Test ──► Deploy ──► Monitor      │
│   │         │        │         │          │          │
│   ▼         ▼        ▼         ▼          ▼          │
│  Linting  Compile  Unit     Rolling    Auto-alert   │
│  Commit   Package  Integr.  Blue-Green Auto-scale   │
│  Hooks    Docker   E2E      Canary     Self-heal    │
│  SAST     Artifact Security Feature                  │
│           Store    Perf     Flags                    │
│                                                      │
│  AUTOMATE IF:                                        │
│  ├── Done more than twice                            │
│  ├── Error-prone when manual                         │
│  ├── Time-consuming                                  │
│  └── Needs consistency                               │
└──────────────────────────────────────────────────────┘
```

**Example — Automated CI/CD Pipeline:**
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: docker build -t myapp:${{ github.sha }} .
      - name: Unit Tests
        run: docker run myapp:${{ github.sha }} npm test
      - name: Security Scan
        run: trivy image myapp:${{ github.sha }}
      - name: Push to Registry
        run: docker push myapp:${{ github.sha }}

  deploy:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        run: kubectl set image deployment/myapp myapp=myapp:${{ github.sha }}
```

---

## L — Lean

Borrowed from Lean Manufacturing (Toyota Production System), Lean in DevOps focuses on reducing waste and optimizing flow.

```
┌──────────────────────────────────────────────────────┐
│  LEAN PRINCIPLES IN DEVOPS                           │
│  ═════════════════════════                           │
│                                                      │
│  7 Types of Waste (adapted for software):            │
│                                                      │
│  ┌──────────────────┬──────────────────────────────┐ │
│  │ Waste Type       │ DevOps Example               │ │
│  ├──────────────────┼──────────────────────────────┤ │
│  │ Overproduction   │ Building features nobody uses│ │
│  │ Waiting          │ Waiting for approvals/builds │ │
│  │ Transportation   │ Handoffs between teams       │ │
│  │ Over-processing  │ Unnecessary documentation    │ │
│  │ Inventory        │ Undeployed code branches     │ │
│  │ Motion           │ Context switching            │ │
│  │ Defects          │ Bugs found late in cycle     │ │
│  └──────────────────┴──────────────────────────────┘ │
│                                                      │
│  Lean Solution: Small Batch Sizes                    │
│                                                      │
│  BIG BATCH:   ████████████████████ → HIGH RISK       │
│  SMALL BATCH: ██ ██ ██ ██ ██ ██ ██ → LOW RISK        │
│                                                      │
│  Smaller changes = easier to review, test, deploy,   │
│  and rollback                                        │
└──────────────────────────────────────────────────────┘
```

**Key Lean Concepts:**
- **Value Stream Mapping:** Visualize the flow from idea to production; identify bottlenecks
- **Work in Progress (WIP) Limits:** Don't start too many things at once
- **Pull-based systems:** Work is pulled by the next stage, not pushed

---

## M — Measurement

You can't improve what you don't measure. DevOps emphasizes data-driven decision making.

```
┌──────────────────────────────────────────────────────┐
│  MEASUREMENT                                         │
│  ═══════════                                         │
│                                                      │
│  Key Metrics to Track:                               │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │  DORA Metrics (Four Key Metrics):               │ │
│  │  ├── Deployment Frequency        (Speed)        │ │
│  │  ├── Lead Time for Changes       (Speed)        │ │
│  │  ├── Mean Time to Recovery       (Stability)    │ │
│  │  └── Change Failure Rate         (Stability)    │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │  Operational Metrics:                           │ │
│  │  ├── Uptime / Availability (SLA)                │ │
│  │  ├── Error rates                                │ │
│  │  ├── Response time (p50, p95, p99)              │ │
│  │  └── Infrastructure costs                      │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │  Team Metrics:                                  │ │
│  │  ├── Developer satisfaction (surveys)           │ │
│  │  ├── Toil ratio (manual vs automated)           │ │
│  │  └── On-call burden                             │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ⚠️ Avoid Vanity Metrics:                           │
│  ├── Lines of code written                          │
│  ├── Number of commits                              │
│  └── Hours worked                                   │
└──────────────────────────────────────────────────────┘
```

**Example — Grafana Dashboard Configuration:**
```yaml
# Prometheus alert rule for high error rate
groups:
  - name: application-alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected (> 5%)"
          description: "{{ $labels.instance }} has error rate of {{ $value }}"
```

---

## S — Sharing

Knowledge sharing and transparency are force multipliers.

```
┌──────────────────────────────────────────────────────┐
│  SHARING                                             │
│  ═══════                                             │
│                                                      │
│  What to Share:                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │                                                │  │
│  │  Code ──► Open-source internally (InnerSource) │  │
│  │  Docs ──► Wikis, runbooks, decision records    │  │
│  │  Metrics ► Shared dashboards (Grafana)         │  │
│  │  Learnings► Postmortems, tech talks            │  │
│  │  Tools ──► Internal platform / golden paths    │  │
│  │  On-call ► Shared rotation, shared context     │  │
│  │                                                │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  Sharing Mechanisms:                                 │
│  ├── Internal wiki / Confluence / Notion             │
│  ├── Weekly tech talks / lunch-and-learns            │
│  ├── Pair programming / mob programming              │
│  ├── ChatOps (Slack bots for deploy notifications)   │
│  ├── Architecture Decision Records (ADRs)            │
│  └── InnerSource (open PRs across teams)             │
└──────────────────────────────────────────────────────┘
```

**Example — Architecture Decision Record (ADR):**
```markdown
# ADR-001: Use PostgreSQL over MySQL

## Status: Accepted
## Date: 2026-02-15

## Context
We need a relational database for the user service.

## Decision
We will use PostgreSQL 16 because:
- Better JSON support (needed for flexible user profiles)
- Superior concurrency handling (MVCC)
- Strong community and cloud support

## Consequences
- Team needs PostgreSQL training
- Existing MySQL scripts need migration
- Monitoring dashboards need updating
```

---

## CALMS Self-Assessment

Use this to evaluate your organization's DevOps maturity:

```
┌──────────┬──────────────────┬──────────────────┬──────────────────┐
│ Pillar   │ Level 1 (Basic)  │ Level 2 (Inter.) │ Level 3 (Advanced│
├──────────┼──────────────────┼──────────────────┼──────────────────┤
│ Culture  │ Blame culture    │ Some blameless   │ Fully generative │
│          │ Silos exist      │ practices        │ culture          │
├──────────┼──────────────────┼──────────────────┼──────────────────┤
│ Automat. │ Manual builds/   │ CI in place      │ Full CI/CD,      │
│          │ deploys          │ Some CD           │ self-service     │
├──────────┼──────────────────┼──────────────────┼──────────────────┤
│ Lean     │ Big batch,       │ Smaller batches  │ Continuous flow, │
│          │ long cycles      │ WIP awareness    │ WIP limits       │
├──────────┼──────────────────┼──────────────────┼──────────────────┤
│ Measure  │ No metrics       │ Basic monitoring │ DORA metrics,    │
│          │                  │ Some dashboards  │ data-driven      │
├──────────┼──────────────────┼──────────────────┼──────────────────┤
│ Sharing  │ Knowledge in     │ Some docs/wikis  │ InnerSource,     │
│          │ people's heads   │ Occasional talks │ ADRs, ChatOps    │
└──────────┴──────────────────┴──────────────────┴──────────────────┘
```

---

## Real-World Scenario: Applying CALMS

### 🏢 Scenario: Startup Adopts CALMS

```
CULTURE:     Introduced blameless retros every Friday.
             Shared Slack channels for all teams.

AUTOMATION:  Set up GitHub Actions for CI.
             Dockerized all services.
             Automated staging deployments.

LEAN:        Switched from 2-week sprints to continuous flow.
             Introduced WIP limit of 3 per developer.
             Stopped building features with <10% user demand.

MEASUREMENT: Deployed Prometheus + Grafana.
             Tracked DORA metrics monthly.
             Reduced MTTR from 2 hours to 10 minutes.

SHARING:     Started weekly "Show & Tell" sessions.
             Created internal wiki with runbooks.
             Adopted ADRs for all architectural decisions.

Result: Deployment frequency went from weekly to 5x daily
        in 6 months.
```

---

## Troubleshooting Tips

| Pillar | Problem | Solution |
|--------|---------|----------|
| **Culture** | Leadership says "DevOps" but acts traditional | Align incentives; measure team outcomes, not individual output |
| **Automation** | Automated pipeline exists but nobody trusts it | Improve test coverage; make pipeline results visible |
| **Lean** | Teams start too many things | Enforce WIP limits on your Kanban board |
| **Measurement** | Metrics are collected but not acted upon | Review metrics in retros; set improvement targets |
| **Sharing** | Knowledge stays in one person's head | Pair programming, documentation sprints, bus factor reviews |

---

## Summary Table

| Pillar | Focus | Key Practice |
|--------|-------|-------------|
| **Culture** | Collaboration, trust, blamelessness | Blameless postmortems, shared goals |
| **Automation** | Eliminate manual repetitive work | CI/CD pipelines, IaC, auto-scaling |
| **Lean** | Reduce waste, small batches | Value stream mapping, WIP limits |
| **Measurement** | Data-driven decisions | DORA metrics, dashboards, alerts |
| **Sharing** | Transparency, knowledge transfer | Tech talks, ADRs, InnerSource, ChatOps |

---

## Quick Revision Questions

1. **What does CALMS stand for, and who coined it?**
   <details><summary>Answer</summary>CALMS stands for Culture, Automation, Lean, Measurement, and Sharing. It was coined by Jez Humble as a framework for assessing DevOps adoption readiness and maturity.</details>

2. **Why is Culture listed first in the CALMS framework?**
   <details><summary>Answer</summary>Culture is the foundation of DevOps. Without the right culture (collaboration, blamelessness, shared ownership), automation and tools will not deliver their full potential. Culture change is the hardest and most important part of DevOps transformation.</details>

3. **Name the 7 types of waste in Lean and give a DevOps example for each.**
   <details><summary>Answer</summary>1) Overproduction: building unused features. 2) Waiting: waiting for approvals. 3) Transportation: handoffs between teams. 4) Over-processing: unnecessary documentation. 5) Inventory: undeployed code. 6) Motion: context switching. 7) Defects: bugs found late.</details>

4. **What are vanity metrics, and why should they be avoided?**
   <details><summary>Answer</summary>Vanity metrics are measurements that look good but don't indicate actual improvement — like lines of code written, number of commits, or hours worked. They can be gamed and don't correlate with software delivery performance. Better alternatives are DORA metrics.</details>

5. **Give three practical examples of the "Sharing" pillar.**
   <details><summary>Answer</summary>1) Architecture Decision Records (ADRs) to document and share design choices. 2) Weekly tech talks where teams present learnings. 3) ChatOps — using Slack bots to automatically share deployment statuses, alerts, and build results with everyone.</details>

6. **How would you assess your organization's CALMS maturity?**
   <details><summary>Answer</summary>Use the CALMS self-assessment matrix: evaluate each pillar at Level 1 (Basic), Level 2 (Intermediate), or Level 3 (Advanced). For example, Culture: is blame common (L1), or are blameless postmortems standard (L3)? Automation: are builds manual (L1) or fully CI/CD with self-service (L3)?</details>

---

[← Previous: Benefits and Challenges](../01-What-is-DevOps/05-benefits-and-challenges.md) | [Next: The Three Ways →](02-three-ways.md)
