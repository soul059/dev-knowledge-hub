# Chapter 5.3 — Error Budgets

[← Previous: SLIs, SLOs, and SLAs](02-slis-slos-slas.md) | [Next: Measuring DevOps Success →](04-measuring-devops-success.md)

---

## Overview

An **Error Budget** is the maximum amount of unreliability a service is allowed. It's the practical tool that turns SLOs from aspirational targets into actionable constraints. Error budgets create a shared framework between development (who wants to ship fast) and operations (who wants stability).

---

## What Is an Error Budget?

```
┌──────────────────────────────────────────────────────────┐
│  ERROR BUDGET = 100% − SLO                               │
│                                                          │
│  If SLO = 99.95% availability                            │
│  Then Error Budget = 0.05%                               │
│                                                          │
│  In a 30-day month (43,200 minutes):                     │
│  0.05% × 43,200 = 21.6 minutes of allowed downtime      │
│                                                          │
│  ┌──────────────────────────────────────────────┐        │
│  │ █████████████████████████████████████████░░░░ │        │
│  │ ◄──── 99.95% must be good ──────►◄── 0.05%─►│        │
│  │         (available)           (error budget)  │        │
│  └──────────────────────────────────────────────┘        │
│                                                          │
│  Error budget is NOT a target to hit!                    │
│  It's a BUDGET to spend on innovation, features, risk.   │
└──────────────────────────────────────────────────────────┘
```

---

## How Error Budgets Balance Speed and Stability

```
┌──────────────────────────────────────────────────────────┐
│  THE TENSION (without error budgets):                    │
│                                                          │
│  Developers:                 Operations:                 │
│  "Ship faster!"             "Don't break things!"        │
│  ├── More features          ├── Fewer changes            │
│  ├── More experiments       ├── More testing              │
│  └── More risk              └── More caution              │
│                                                          │
│        They fight ← ─── → endlessly                      │
│                                                          │
│  THE SOLUTION (with error budgets):                      │
│                                                          │
│  Error Budget remaining?                                 │
│  ├── YES (budget > 0%)      ├── NO (budget exhausted)    │
│  │   ✅ Ship features       │   🛑 FREEZE features       │
│  │   ✅ Take calculated     │   ✅ Focus on reliability   │
│  │      risks               │   ✅ Fix root causes        │
│  │   ✅ Run experiments      │   ✅ Add monitoring         │
│  │   ✅ Deploy aggressively  │   ✅ Write postmortems      │
│  │                           │                            │
│  └── Data-driven decision   └── Data-driven decision     │
│                                                          │
│  No more subjective arguments — the BUDGET decides.      │
└──────────────────────────────────────────────────────────┘
```

---

## Error Budget Burn Rate

```
┌──────────────────────────────────────────────────────────┐
│  BURN RATE = How fast error budget is consumed           │
│                                                          │
│  Budget    ██████████████████████████  100% (start of    │
│            ████████████████████████░░   90%  month)      │
│  Normal    ██████████████████████░░░░   80%              │
│  burn      ████████████████████░░░░░░   70%              │
│            ██████████████████░░░░░░░░   →  Steady        │
│            ████████████████░░░░░░░░░░       depletion    │
│  Month end ████░░░░░░░░░░░░░░░░░░░░░░  ~5%  ← Perfect!  │
│                                                          │
│  ═══════════════════════════════════════════════          │
│                                                          │
│  Budget    ██████████████████████████  100%              │
│            ████████████░░░░░░░░░░░░░░   45%  ← Incident!│
│  Fast      █████████░░░░░░░░░░░░░░░░░   30%              │
│  burn      ████░░░░░░░░░░░░░░░░░░░░░░   10%              │
│            ░░░░░░░░░░░░░░░░░░░░░░░░░░    0%  ← EXHAUSTED│
│                                         Day 12  FREEZE!  │
│                                                          │
│  Burn Rate = 1.0 → using budget evenly over the month    │
│  Burn Rate = 14.4 → will exhaust budget in ~1 hour!      │
│  Burn Rate = 6.0 → will exhaust budget in ~6 hours       │
└──────────────────────────────────────────────────────────┘
```

---

## Error Budget Policy

```yaml
# error-budget-policy.yaml
# This document defines how the team responds to error budget status

service: checkout-api
slo: 99.95% availability
error_budget_period: 30 days
error_budget_minutes: 21.6

policy:
  # Budget is healthy (> 50% remaining)
  green:
    condition: "error_budget_remaining > 50%"
    actions:
      - "Ship features at normal velocity"
      - "Run experiments and A/B tests"
      - "Perform planned infrastructure changes"
      - "Chaos engineering exercises allowed"

  # Budget is concerning (20-50% remaining)
  yellow:
    condition: "20% < error_budget_remaining <= 50%"
    actions:
      - "Slow down feature releases"
      - "Prioritize reliability improvements"
      - "Review recent incidents for patterns"
      - "No risky infrastructure changes"

  # Budget is critical (< 20% remaining)
  red:
    condition: "error_budget_remaining <= 20%"
    actions:
      - "FEATURE FREEZE — no new features shipped"
      - "All engineering effort on reliability"
      - "Root cause analysis for all recent incidents"
      - "Leadership review required to resume features"

  # Budget exhausted (0%)
  exhausted:
    condition: "error_budget_remaining <= 0%"
    actions:
      - "Complete feature freeze until next budget period"
      - "Incident review with engineering leadership"
      - "Postmortem for every incident that consumed budget"
      - "Reassess SLO if it's unachievable"
```

---

## Error Budget Dashboard

```
┌──────────────────────────────────────────────────────────┐
│  ERROR BUDGET DASHBOARD — Checkout API (November 2024)   │
│                                                          │
│  SLO: 99.95% | Budget: 21.6 min | Day: 18 of 30        │
│                                                          │
│  Budget Consumed: ████████████░░░░░░░░  58% (12.5 min)  │
│  Budget Remaining: ░░░░░░░░██████████  42% (9.1 min)    │
│                                                          │
│  Status: 🟡 YELLOW — Slow down releases                 │
│                                                          │
│  INCIDENTS THIS MONTH:                                   │
│  ┌─────────┬────────┬──────────┬────────────────────┐   │
│  │ Date    │Minutes │ Budget % │ Cause              │   │
│  ├─────────┼────────┼──────────┼────────────────────┤   │
│  │ Nov 3   │ 5.2    │ 24%      │ Bad deploy rollback│   │
│  │ Nov 10  │ 3.1    │ 14%      │ DB connection pool │   │
│  │ Nov 15  │ 4.2    │ 20%      │ Third-party API    │   │
│  ├─────────┼────────┼──────────┼────────────────────┤   │
│  │ TOTAL   │ 12.5   │ 58%      │                    │   │
│  └─────────┴────────┴──────────┴────────────────────┘   │
│                                                          │
│  PROJECTION: At current burn rate, budget exhausted      │
│  by Nov 26. Recommend reliability focus.                 │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Error Budget in Action

```
MONTH 1: Budget Healthy (Team ships aggressively)
├── Budget: 21.6 min | Used: 3.2 min (15%)
├── Shipped 12 features
├── Ran 2 chaos engineering experiments
├── Performed Kubernetes upgrade
└── Status: 🟢 GREEN

MONTH 2: Incident consumes budget
├── Budget: 21.6 min
├── Day 5: Major incident — 14 minutes of downtime
│   └── Cause: Memory leak in new feature from Month 1
├── Budget used: 14 min (65%) — jumped to YELLOW
├── Team action: Paused features, focused on reliability
├── Day 20: Another 4 min incident
├── Budget used: 18 min (83%) — now RED
└── Status: 🔴 FEATURE FREEZE

MONTH 3: Reliability sprint
├── Budget: 21.6 min (fresh budget)
├── Team improvements during Month 2 freeze:
│   ├── Added memory monitoring and alerts
│   ├── Implemented canary deployments
│   ├── Fixed 3 known reliability issues
│   └── Added load testing to CI pipeline
├── Budget used: 1.1 min (5%)
├── Shipped 8 features (slower but safer)
└── Status: 🟢 GREEN — improvements paid off!
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Budget always exhausted | SLO too aggressive | Loosen SLO to something achievable; tighten over time |
| Budget never consumed | SLO too lenient | Tighten SLO; team could be shipping faster |
| Team ignores error budget | No enforcement mechanism | Tie budget to feature freeze policy; get leadership buy-in |
| Single incident exhausts budget | No blast radius limits | Use canary deploys, feature flags; improve MTTD/MTTR |
| External dependencies drain budget | Third-party outages count | Track external vs internal causes; adjust SLO or add redundancy |
| Teams game the budget | Excluding incidents from count | Clear criteria for what counts; independent tracking |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Error Budget** | Allowed unreliability = 100% − SLO |
| **Burn Rate** | Speed at which error budget is consumed |
| **Budget Policy** | Rules for team behavior based on budget remaining |
| **Feature Freeze** | Stop shipping features when budget is exhausted |
| **Green/Yellow/Red** | Traffic light system for budget health status |
| **Budget Period** | Rolling window for error budget (usually 30 days) |
| **Key Insight** | Error budgets resolve Dev vs Ops conflict with data |

---

## Quick Revision Questions

1. **What is an error budget, and how is it calculated?**
   <details><summary>Answer</summary>An error budget is the maximum amount of unreliability allowed by an SLO. Calculated as: Error Budget = 100% − SLO. For a 99.95% SLO, the error budget is 0.05%. Over 30 days (43,200 minutes), that's 21.6 minutes of allowed downtime. It's not a target to hit — it's a budget to spend on innovation, features, and calculated risks.</details>

2. **How do error budgets resolve the tension between developers and operations?**
   <details><summary>Answer</summary>Without error budgets, devs want to ship fast and ops want stability — a subjective conflict. With error budgets: if budget remains (service is reliable), ship features aggressively. If budget is exhausted (service was unreliable), freeze features and focus on reliability. The data decides, not opinions. Both teams share a common metric.</details>

3. **What is burn rate, and why does it matter?**
   <details><summary>Answer</summary>Burn rate measures how quickly the error budget is being consumed. A burn rate of 1.0 means the budget will last exactly the full period. A burn rate of 14.4 means the budget will be exhausted in ~1 hour. Burn rate is used for alerting: high burn rates trigger immediate attention, while sustained moderate burn rates trigger investigation.</details>

4. **What should happen when the error budget is exhausted?**
   <details><summary>Answer</summary>When the error budget hits 0%: 1) Feature freeze — no new features shipped. 2) All engineering effort directed to reliability improvements. 3) Root cause analysis for every incident that consumed budget. 4) Leadership review to assess and approve next steps. Features resume only when a new budget period starts or leadership approves based on improvements.</details>

5. **What is an error budget policy, and what should it include?**
   <details><summary>Answer</summary>An error budget policy is a documented agreement on how the team responds to different budget levels. It should include: thresholds (green/yellow/red), actions at each level, escalation paths, definition of what counts as an incident, who can authorize exceptions, and how external dependencies are handled. It must be agreed upon by both dev and ops leadership.</details>

6. **How do you handle third-party outages that consume your error budget?**
   <details><summary>Answer</summary>Options: 1) Track external vs internal causes separately for accountability. 2) Add redundancy for critical third-party services (fallback providers). 3) Use circuit breakers to gracefully degrade when dependencies fail. 4) Adjust SLO to account for dependency reliability. 5) Build async processing to reduce real-time dependency. External causes still count toward user experience.</details>

---

[← Previous: SLIs, SLOs, and SLAs](02-slis-slos-slas.md) | [Next: Measuring DevOps Success →](04-measuring-devops-success.md)
