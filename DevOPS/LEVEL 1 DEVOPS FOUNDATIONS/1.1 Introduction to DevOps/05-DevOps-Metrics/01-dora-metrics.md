# Chapter 5.1 — DORA Metrics

[← Previous: Cloud Platforms](../04-DevOps-Tools-Landscape/07-cloud-platforms.md) | [Next: SLIs, SLOs, and SLAs →](02-slis-slos-slas.md)

---

## Overview

**DORA (DevOps Research and Assessment)** metrics are four key measures that indicate the performance of software delivery teams. Backed by years of research (Accelerate book, State of DevOps reports), these metrics reliably predict organizational performance.

---

## The Four DORA Metrics

```
┌──────────────────────────────────────────────────────────┐
│  THE FOUR DORA METRICS                                   │
│                                                          │
│  SPEED (Throughput)              STABILITY (Reliability) │
│  ═══════════════════             ════════════════════     │
│                                                          │
│  1. Deployment Frequency         3. Change Failure Rate  │
│     "How often do we ship?"         "How often do we     │
│                                      break things?"      │
│  2. Lead Time for Changes        4. Mean Time to Recover │
│     "How fast from commit            "How fast do we     │
│      to production?"                  fix failures?"     │
│                                                          │
│  ┌────────────────────────────────────────────────┐      │
│  │  HIGH PERFORMERS:                              │      │
│  │  ├── Deploy multiple times per day             │      │
│  │  ├── Lead time < 1 hour                        │      │
│  │  ├── Change failure rate < 5%                  │      │
│  │  └── Recovery time < 1 hour                    │      │
│  │                                                │      │
│  │  LOW PERFORMERS:                               │      │
│  │  ├── Deploy monthly or less                    │      │
│  │  ├── Lead time > 6 months                      │      │
│  │  ├── Change failure rate > 45%                 │      │
│  │  └── Recovery time > 1 week                    │      │
│  └────────────────────────────────────────────────┘      │
└──────────────────────────────────────────────────────────┘
```

---

## Metric 1: Deployment Frequency

```
┌──────────────────────────────────────────────────────────┐
│  DEPLOYMENT FREQUENCY                                    │
│  "How often does your team deploy to production?"        │
│                                                          │
│  Elite:  ████████████████████  Multiple/day              │
│  High:   █████████████████     Weekly to daily           │
│  Medium: ██████████████        Monthly to weekly         │
│  Low:    █████████             Monthly or less           │
│                                                          │
│  HOW TO MEASURE:                                         │
│  ├── Count production deployments per time period        │
│  ├── Track via CI/CD pipeline (deployment job runs)      │
│  └── Exclude rollbacks and hotfixes from count           │
│                                                          │
│  HOW TO IMPROVE:                                         │
│  ├── Smaller batch sizes (less code per deploy)          │
│  ├── Automate deployment pipeline                        │
│  ├── Trunk-based development                             │
│  ├── Feature flags (decouple deploy from release)        │
│  └── Reduce manual approval gates                        │
└──────────────────────────────────────────────────────────┘
```

---

## Metric 2: Lead Time for Changes

```
┌──────────────────────────────────────────────────────────┐
│  LEAD TIME FOR CHANGES                                   │
│  "Time from first commit to running in production"       │
│                                                          │
│  commit ──► build ──► test ──► review ──► deploy         │
│  │                                             │         │
│  └─────────── Lead Time ───────────────────────┘         │
│                                                          │
│  Elite:  < 1 hour                                        │
│  High:   1 day to 1 week                                 │
│  Medium: 1 week to 1 month                               │
│  Low:    > 6 months                                      │
│                                                          │
│  BOTTLENECK ANALYSIS:                                    │
│  ┌──────────────────────────────────────────┐            │
│  │ Build: 5 min   ✅                        │            │
│  │ Test:  20 min  ✅                        │            │
│  │ Review: 2 DAYS ❌  ← BOTTLENECK          │            │
│  │ Approval: 1 day ⚠️                       │            │
│  │ Deploy: 10 min ✅                        │            │
│  │ Total: ~3 days  (most is waiting!)       │            │
│  └──────────────────────────────────────────┘            │
│                                                          │
│  Fix: Automate approvals, small PRs, faster reviews      │
└──────────────────────────────────────────────────────────┘
```

---

## Metric 3: Change Failure Rate

```
┌──────────────────────────────────────────────────────────┐
│  CHANGE FAILURE RATE (CFR)                               │
│  "% of deployments causing a failure in production"      │
│                                                          │
│  Formula: (failed deployments / total deployments) × 100 │
│                                                          │
│  Elite:  0-5%                                            │
│  High:   5-10%                                           │
│  Medium: 10-15%                                          │
│  Low:    > 45%                                           │
│                                                          │
│  WHAT COUNTS AS FAILURE:                                 │
│  ├── Service outage or degradation                       │
│  ├── Rollback required                                   │
│  ├── Hotfix needed                                       │
│  └── Significant performance regression                  │
│                                                          │
│  HOW TO REDUCE CFR:                                      │
│  ├── Comprehensive automated testing                     │
│  ├── Canary deployments (test on small % first)          │
│  ├── Feature flags (kill switch for bad releases)        │
│  ├── Code review requirements                            │
│  ├── Shift-left testing (test earlier)                   │
│  └── Smaller, incremental changes                        │
└──────────────────────────────────────────────────────────┘
```

---

## Metric 4: Mean Time to Recover (MTTR)

```
┌──────────────────────────────────────────────────────────┐
│  MTTR — MEAN TIME TO RECOVER                             │
│  "How long from failure detection to full recovery?"     │
│                                                          │
│  ┌──────────────── MTTR ─────────────────────┐           │
│  │                                            │           │
│  Failure ──► Detect ──► Diagnose ──► Fix ──► Recover     │
│  occurs      (MTTD)     (root cause)  (deploy) (verified)│
│                                                          │
│  Elite:  < 1 hour                                        │
│  High:   < 1 day                                         │
│  Medium: 1 day to 1 week                                 │
│  Low:    > 6 months                                      │
│                                                          │
│  RELATED METRICS:                                        │
│  ├── MTTD (Mean Time to Detect) — alert fires            │
│  ├── MTTA (Mean Time to Acknowledge) — human responds    │
│  ├── MTTR (Mean Time to Recover) — service restored      │
│  └── MTBF (Mean Time Between Failures) — reliability      │
│                                                          │
│  HOW TO IMPROVE MTTR:                                    │
│  ├── Robust monitoring and alerting (reduce MTTD)        │
│  ├── Runbooks for common incidents                       │
│  ├── One-click rollback capability                       │
│  ├── Blameless postmortem culture                        │
│  └── Chaos engineering (practice recovery)               │
└──────────────────────────────────────────────────────────┘
```

---

## DORA Performance Levels

| Metric | Elite | High | Medium | Low |
|--------|-------|------|--------|-----|
| **Deploy Frequency** | On-demand (multiple/day) | Weekly–daily | Monthly–weekly | Monthly or less |
| **Lead Time** | < 1 hour | 1 day – 1 week | 1 week – 1 month | > 6 months |
| **Change Failure Rate** | 0–5% | 5–10% | 10–15% | > 45% |
| **MTTR** | < 1 hour | < 1 day | 1 day – 1 week | > 6 months |

---

## Measuring DORA in Practice

```yaml
# Example: Tracking DORA metrics with CI/CD data

# Data sources:
deployment_frequency:
  source: "CI/CD pipeline deployment job count"
  tool: "GitHub Actions / GitLab CI"
  query: "COUNT deployments WHERE env=production GROUP BY week"

lead_time:
  source: "Git commit timestamp → deployment timestamp"
  tool: "Git + CI/CD pipeline"
  query: "AVG(deploy_time - first_commit_time) GROUP BY week"

change_failure_rate:
  source: "Incident tickets linked to deployments"
  tool: "PagerDuty + CI/CD pipeline"
  query: "COUNT(failed_deploys) / COUNT(total_deploys) * 100"

mttr:
  source: "Incident open → incident resolved timestamps"
  tool: "PagerDuty / OpsGenie"
  query: "AVG(resolved_at - detected_at) GROUP BY week"

# Dashboard tools:
# - Grafana dashboard with DORA panels
# - Sleuth.io (DORA-specific tool)
# - LinearB, Jellyfish, Swarmia
# - Four Keys (Google open-source project)
```

---

## Real-World Scenario: Improving DORA Metrics

```
BEFORE (Team assessment — Low Performer):
├── Deploy Frequency:  Monthly releases          ❌
├── Lead Time:         3-4 weeks                 ❌
├── Change Failure Rate: 35%                     ❌
└── MTTR:              2-3 days                  ❌

ACTIONS TAKEN (6-month improvement plan):

Month 1-2: Foundation
├── Implemented CI pipeline (automated build + test)
├── Adopted trunk-based development
└── Set up monitoring with Prometheus + Grafana

Month 3-4: Automation
├── Implemented CD pipeline (automated deployments)
├── Added canary deployments
├── Created runbooks for top 10 incidents
└── Introduced feature flags

Month 5-6: Optimization
├── Implemented blameless postmortems
├── Reduced PR size limit to <300 lines
├── Added automated security scanning
└── Practiced chaos engineering

AFTER (6 months later — High Performer):
├── Deploy Frequency:  2-3 times/week            ✅
├── Lead Time:         2-3 days                  ✅
├── Change Failure Rate: 8%                      ✅
└── MTTR:              4 hours                   ✅
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Deploy frequency low | Large batch sizes, manual process | Smaller PRs, automate pipeline, remove approval bottlenecks |
| Lead time high | Waiting for reviews or manual steps | Automate tests, set review SLAs, use trunk-based dev |
| CFR high | Insufficient testing or review | Shift-left testing, canary deployments, staging parity |
| MTTR high | Poor observability, no runbooks | Better alerting, runbooks, one-click rollback, chaos engineering |
| Metrics gamed | Teams optimizing metrics not outcomes | Focus on trends, not absolute numbers; tie to business outcomes |

---

## Summary Table

| DORA Concept | Description |
|-------------|-------------|
| **DORA** | DevOps Research and Assessment — research-backed metrics |
| **Deployment Frequency** | How often code is deployed to production |
| **Lead Time** | Time from commit to running in production |
| **Change Failure Rate** | Percentage of deployments causing failures |
| **MTTR** | Mean Time to Recover from production failures |
| **Elite Performer** | Multiple deploys/day, <1hr lead time, <5% CFR, <1hr MTTR |
| **Accelerate** | Book by Dr. Nicole Forsgren et al. presenting the research |

---

## Quick Revision Questions

1. **What are the four DORA metrics, and what do they measure?**
   <details><summary>Answer</summary>1) Deployment Frequency — how often the team deploys to production (throughput). 2) Lead Time for Changes — time from commit to production (speed). 3) Change Failure Rate — percentage of deployments causing failures (stability). 4) Mean Time to Recover — time to restore service after failure (reliability). The first two measure speed; the last two measure stability.</details>

2. **Why do DORA metrics matter to organizations?**
   <details><summary>Answer</summary>Research (Accelerate, State of DevOps reports) shows that elite-performing teams with strong DORA metrics have: 2× higher profitability, 50% higher market share growth, and higher employee satisfaction. DORA metrics are predictive — improving them correlates with better business outcomes. They also provide a common language for measuring DevOps progress.</details>

3. **What characterizes an "elite" performer according to DORA?**
   <details><summary>Answer</summary>Elite performers: deploy on-demand (multiple times per day), have lead time under 1 hour, change failure rate between 0-5%, and recover from failures in under 1 hour. Key insight: elite teams are both FASTER and MORE STABLE — speed and stability are not tradeoffs; they reinforce each other.</details>

4. **How do you measure Lead Time for Changes in practice?**
   <details><summary>Answer</summary>Track the time from the first commit in a changeset to when that code is running in production. Data sources: Git (commit timestamp) + CI/CD pipeline (deployment timestamp). Calculate: deployment_time - first_commit_time. Track weekly averages and trends. Bottleneck analysis helps identify if wait time is in build, test, review, or deployment stages.</details>

5. **How can a team reduce its Change Failure Rate?**
   <details><summary>Answer</summary>1) Comprehensive automated testing (unit, integration, e2e). 2) Canary or blue-green deployments to limit blast radius. 3) Feature flags to quickly disable broken features. 4) Code review requirements. 5) Shift-left testing (test earlier in the pipeline). 6) Smaller, incremental changes (easier to test and rollback). 7) Staging environment that mirrors production.</details>

6. **What is the relationship between MTTR and MTBF, and which should DevOps teams prioritize?**
   <details><summary>Answer</summary>MTBF (Mean Time Between Failures) measures how often failures occur. MTTR measures how quickly you recover. DevOps teams should prioritize MTTR over MTBF because: 1) Failures are inevitable in complex systems. 2) Fast recovery minimizes user impact. 3) Optimizing for MTBF leads to fear of change and slower deployments. Accept failures, recover fast.</details>

---

[← Previous: Cloud Platforms](../04-DevOps-Tools-Landscape/07-cloud-platforms.md) | [Next: SLIs, SLOs, and SLAs →](02-slis-slos-slas.md)
