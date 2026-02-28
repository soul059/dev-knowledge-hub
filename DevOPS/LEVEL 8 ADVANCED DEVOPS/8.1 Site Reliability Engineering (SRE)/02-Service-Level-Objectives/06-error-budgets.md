# Chapter 2.6: Error Budgets

[← Previous: Setting SLO Targets](05-setting-slo-targets.md) | [Back to README](../README.md) | [Next: Availability Calculations →](../03-Reliability/01-availability-calculations.md)

---

## Overview

Error budgets are the most powerful concept in SRE. They transform the subjective question "Is our service reliable enough?" into an objective, data-driven framework. An error budget is simply the **amount of unreliability your SLO allows** — and it's the mechanism that aligns dev velocity with operational stability.

> **Error Budget = 1 − SLO**

---

## 1. Error Budget Fundamentals

```
  ┌──────────────────────────────────────────────────────────┐
  │                  ERROR BUDGET MATH                       │
  │                                                          │
  │  SLO = 99.9%                                             │
  │  Error Budget = 1 - 0.999 = 0.001 = 0.1%                │
  │                                                          │
  │  In a 30-day month (43,200 minutes):                     │
  │  Budget = 43,200 × 0.001 = 43.2 minutes                 │
  │                                                          │
  │  ┌────────────────────────────────────────────┐          │
  │  │  SLO      │ Budget/Month │ Budget/Year     │          │
  │  ├───────────┼──────────────┼─────────────────┤          │
  │  │ 99%       │ 7.2 hours    │ 3.65 days       │          │
  │  │ 99.5%     │ 3.6 hours    │ 1.83 days       │          │
  │  │ 99.9%     │ 43.2 min     │ 8.76 hours      │          │
  │  │ 99.95%    │ 21.6 min     │ 4.38 hours      │          │
  │  │ 99.99%    │ 4.32 min     │ 52.6 min        │          │
  │  │ 99.999%   │ 25.9 sec     │ 5.26 min        │          │
  │  └───────────┴──────────────┴─────────────────┘          │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. How Error Budgets Work

```
  ┌──────────────────────────────────────────────────────────┐
  │           ERROR BUDGET LIFECYCLE                         │
  │                                                          │
  │  Day 1: Budget starts at 100%                            │
  │  ┌──────────────────────────────────────────────┐        │
  │  │█████████████████████████████████████████ 100% │        │
  │  └──────────────────────────────────────────────┘        │
  │                                                          │
  │  Day 5: Deployment causes 10 min of errors               │
  │  ┌────────────────────────────────────────┐              │
  │  │████████████████████████████████████ 77% │              │
  │  └────────────────────────────────────────┘              │
  │  Budget spent: 10/43.2 = 23.1%                           │
  │                                                          │
  │  Day 12: Another incident, 15 min of errors              │
  │  ┌──────────────────────────────┐                        │
  │  │████████████████████████ 42%  │                        │
  │  └──────────────────────────────┘                        │
  │  Budget spent: (10+15)/43.2 = 57.9%                      │
  │                                                          │
  │  Day 15: Bad deploy, 20 min of errors                    │
  │  ┌────────┐                                              │
  │  │████ 4% │  ← DANGER: Near exhaustion                   │
  │  └────────┘                                              │
  │  Budget spent: 45/43.2 = 104% → BUDGET EXHAUSTED!       │
  │                                                          │
  │  → FEATURE FREEZE until budget recovers                  │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 3. Error Budget as Alignment Tool

The error budget resolves the eternal conflict between dev and ops:

```
  WITHOUT ERROR BUDGETS:
  ┌─────────────────┐         ┌─────────────────┐
  │   Development   │  vs.    │   Operations     │
  │   "Ship faster!"│         │   "Change is     │
  │                 │ ◄─────► │    risk!"        │
  │   "Move fast,   │         │   "Slow down!"   │
  │    break things"│         │                  │
  └─────────────────┘         └─────────────────┘
  Result: Constant conflict, no resolution

  WITH ERROR BUDGETS:
  ┌─────────────────────────────────────────────┐
  │   Dev + SRE: Aligned by Shared Metric       │
  │                                             │
  │   Budget remaining → Ship features freely   │
  │   Budget spent → Focus on reliability       │
  │                                             │
  │   Both teams WANT the budget to be healthy: │
  │   Dev: more budget = more releases          │
  │   SRE: more budget = fewer incidents        │
  │                                             │
  │   The DATA decides, not opinions.           │
  └─────────────────────────────────────────────┘
```

---

## 4. Error Budget Burn Rate

Burn rate measures how fast you're consuming the error budget:

```
  Burn Rate = (actual error rate) / (SLO-allowed error rate)

  Example:
    SLO = 99.9% → allowed error rate = 0.1%
    Current error rate (last 1 hour) = 1.44%
    
    Burn rate = 1.44% / 0.1% = 14.4x
    
    At 14.4x burn, the entire 30-day budget will be
    consumed in: 30 days / 14.4 = ~2.08 hours
```

### Burn Rate Reference Table

```
  ┌────────────┬────────────────────┬──────────────────────┐
  │ Burn Rate  │ Budget Exhausted   │ Severity             │
  ├────────────┼────────────────────┼──────────────────────┤
  │ 1x         │ In 30 days         │ Sustainable          │
  │ 2x         │ In 15 days         │ Watch                │
  │ 3x         │ In 10 days         │ Warning (ticket)     │
  │ 6x         │ In 5 days          │ High (page off-hours)│
  │ 14.4x      │ In 2 hours         │ Critical (page NOW)  │
  │ 36x        │ In 20 hours        │ Severe (all hands)   │
  │ 720x       │ In 1 hour          │ Total outage         │
  └────────────┴────────────────────┴──────────────────────┘
```

---

## 5. Multi-Window, Multi-Burn-Rate Alerting

Google's recommended approach for SLO-based alerting:

```
  ┌────────────────────────────────────────────────────────┐
  │  MULTI-WINDOW BURN RATE ALERTS                        │
  │                                                        │
  │  Alert fires when BOTH windows confirm the burn rate:  │
  │                                                        │
  │  ┌──────────────────────────────────────────────────┐  │
  │  │ Severity │ Long Window │ Short Window │ Action   │  │
  │  ├──────────┼─────────────┼──────────────┼──────────┤  │
  │  │ CRITICAL │ 1h at 14.4x │ 5m at 14.4x  │ PAGE     │  │
  │  │ HIGH     │ 6h at 6x    │ 30m at 6x    │ PAGE     │  │
  │  │ MEDIUM   │ 3d at 1x    │ 6h at 1x     │ TICKET   │  │
  │  └──────────┴─────────────┴──────────────┴──────────┘  │
  │                                                        │
  │  Why two windows?                                      │
  │  • Long window: confirms sustained issue               │
  │  • Short window: confirms issue is current (not past)  │
  │                                                        │
  │  Prevents:                                             │
  │  • False pages from short spikes (long window catches) │
  │  • Stale alerts from past events (short confirms now)  │
  └────────────────────────────────────────────────────────┘
```

### Prometheus Alert Rules

```yaml
groups:
- name: error-budget-alerts
  rules:
  # CRITICAL: 14.4x burn rate over 1 hour AND 5 minutes
  - alert: ErrorBudgetBurnCritical
    expr: |
      (
        (1 - sum(rate(http_requests_total{code!~"5.."}[1h]))
             / sum(rate(http_requests_total[1h])))
        > (14.4 * 0.001)
      )
      and
      (
        (1 - sum(rate(http_requests_total{code!~"5.."}[5m]))
             / sum(rate(http_requests_total[5m])))
        > (14.4 * 0.001)
      )
    labels:
      severity: critical
    annotations:
      summary: "Error budget burn rate critical (14.4x)"
      description: "At this rate, the monthly error budget will be exhausted in ~2 hours"
      
  # HIGH: 6x burn rate over 6 hours AND 30 minutes
  - alert: ErrorBudgetBurnHigh
    expr: |
      (
        (1 - sum(rate(http_requests_total{code!~"5.."}[6h]))
             / sum(rate(http_requests_total[6h])))
        > (6 * 0.001)
      )
      and
      (
        (1 - sum(rate(http_requests_total{code!~"5.."}[30m]))
             / sum(rate(http_requests_total[30m])))
        > (6 * 0.001)
      )
    labels:
      severity: high
    annotations:
      summary: "Error budget burn rate high (6x)"
```

---

## 6. Error Budget Tracking Dashboard

```
  ┌──────────────────────────────────────────────────────────────┐
  │  ERROR BUDGET DASHBOARD                                      │
  │  Service: payment-api | SLO: 99.9% | Window: 30-day rolling │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  Current SLI:    99.87%     [!] Below SLO                   │
  │  SLO Target:     99.90%                                     │
  │  Error Budget:   ████░░░░░░░░░░  32% remaining (13.8 min)  │
  │  Budget Used:    ████████████░░  68% (29.4 min of 43.2)    │
  │  Days Remaining: 18 of 30                                   │
  │  Burn Rate:      1.13x (slightly above sustainable)         │
  │                                                              │
  │  Budget Burn Over Time:                                      │
  │  100% ┤                                                      │
  │       │ ╲                                                    │
  │   75% ┤   ╲                                                  │
  │       │     ╲___                                             │
  │   50% ┤         ╲                                            │
  │       │           ╲_____                                     │
  │   32% ┤                  ╲ ← current                        │
  │       │                   ╲                                  │
  │    0% ┤                    ╲??                               │
  │       └────────────────────────────────────                  │
  │       Day 1    Day 7    Day 12   Day 18   Day 30            │
  │                                                              │
  │  Top Budget Consumers:                                       │
  │  1. Deploy v2.4.1 (Day 5)      — 15 min (34.7%)            │
  │  2. DB failover (Day 12)       — 10 min (23.1%)            │
  │  3. Background errors          —  4.4 min (10.2%)           │
  └──────────────────────────────────────────────────────────────┘
```

### Grafana Dashboard JSON (Key Panel)

```json
{
  "title": "Error Budget Remaining (%)",
  "type": "gauge",
  "targets": [
    {
      "expr": "1 - ((1 - avg_over_time(sli:api_availability:ratio_5m[30d])) / (1 - 0.999))",
      "legendFormat": "Budget Remaining"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "thresholds": {
        "steps": [
          { "value": 0, "color": "red" },
          { "value": 20, "color": "orange" },
          { "value": 50, "color": "yellow" },
          { "value": 75, "color": "green" }
        ]
      },
      "max": 100,
      "min": 0,
      "unit": "percent"
    }
  }
}
```

---

## 7. Error Budget Policy in Practice

```
  ┌──────────────────────────────────────────────────────────┐
  │        ERROR BUDGET DECISION FLOWCHART                   │
  │                                                          │
  │  Check error budget ──► Budget > 50%?                    │
  │                              │                           │
  │                    YES ◄─────┼─────► NO                  │
  │                     │        │        │                   │
  │                     ▼        │        ▼                   │
  │              Ship features   │   Budget > 20%?           │
  │              freely          │        │                   │
  │                              │  YES ◄─┼─► NO             │
  │                              │   │    │    │              │
  │                              │   ▼    │    ▼              │
  │                              │  Extra │   Budget > 0%?   │
  │                              │  review│        │          │
  │                              │  for   │  YES ◄─┼─► NO    │
  │                              │  risky │   │    │    │     │
  │                              │  changes│   ▼    │    ▼    │
  │                              │        │ Reliab │ FREEZE   │
  │                              │        │ only   │ ALL      │
  │                              │        │ changes│ DEPLOYS  │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Error Budget Exceptions

Not all budget consumption is equal. Track the source:

```
  ┌──────────────────────────────────────────────────────────┐
  │           ERROR BUDGET ATTRIBUTION                       │
  │                                                          │
  │  Source               │ Counts Against │ Example         │
  │  ─────────────────    │ Error Budget?  │                 │
  │                       │                │                 │
  │  Bad deployment       │ YES            │ Bug in v2.4     │
  │  Infrastructure       │ YES            │ Node failure    │
  │  Dependency failure   │ YES (usually)  │ DB outage       │
  │  Planned maintenance  │ Excluded       │ Scheduled       │
  │  External attack      │ Excluded       │ DDoS            │
  │  Customer misuse      │ Excluded       │ API abuse       │
  │                       │                │                 │
  │  [!] Track attribution to understand WHERE budget goes.  │
  │      "Are we spending budget on deploys or infra?"       │
  └──────────────────────────────────────────────────────────┘
```

---

## 9. Real-World Scenario

### [SCENARIO] Error Budget Drives Feature Freeze

```
  Company: StreamFlix (video streaming)
  Service: Content Delivery API
  SLO: 99.9% availability (43.2 min budget/month)
  
  Timeline:
  ┌──────────────────────────────────────────────────────┐
  │ Day 3:  Deploy new transcoding pipeline              │
  │         15-minute outage during migration            │
  │         Budget: 65% remaining (28.1 min left)        │
  │         Status: GREEN ✓                              │
  │                                                      │
  │ Day 10: CDN configuration error                      │
  │         20-minute degraded performance               │
  │         Budget: 19% remaining (8.1 min left)         │
  │         Status: ORANGE ⚠                             │
  │         Action: Feature freeze, SRE review required  │
  │                                                      │
  │ Day 15: Dev team wants to deploy v3.0 (major release)│
  │         SRE: "Budget at 19%. Denied without          │
  │               additional testing + canary."          │
  │         Compromise: Deploy to 1% canary, monitor 24h │
  │                                                      │
  │ Day 16: Canary looks good. Proceed to 10%.           │
  │ Day 18: Full rollout. No errors. Budget holds at 19%.│
  │                                                      │
  │ Day 30: Month ends. Budget: 12% remaining.           │
  │         New month: Budget resets to 100%.             │
  │                                                      │
  │ Postmortem actions:                                  │
  │ • CDN change process needs review gate               │
  │ • Transcoding migration needs blue-green approach    │
  └──────────────────────────────────────────────────────┘
```

---

## 10. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Dev team ignores error budget" | Get executive sponsorship. Make budget status a weekly reporting metric. |
| "Error budget is always at 0%" | SLO is too tight. Loosen to achievable level and improve iteratively. |
| "Error budget is always at 100%" | SLO may be too loose. Tighten or add latency/correctness SLOs. |
| "One big incident wipes the whole budget" | That's expected! It forces focus on preventing recurrence. |
| "Dependency outages eat our budget" | Track attribution. Escalate dependency SLOs. Build resilience (retries, circuit breakers). |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Error Budget Formula** | 1 − SLO target (e.g., 0.1% for 99.9% SLO) |
| **Time Budget** | Budget × minutes in window (e.g., 43.2 min/month) |
| **Burn Rate** | actual error rate / SLO-allowed error rate |
| **Multi-Window Alerting** | Long window confirms sustained; short window confirms current |
| **Budget Policy** | Graduated response: green → yellow → orange → red (freeze) |
| **Alignment** | Shared metric that aligns dev velocity and operational stability |
| **Attribution** | Track what consumes budget (deploys vs infra vs deps) |
| **Key Insight** | Error budgets are the currency of innovation — spend wisely |

---

## Quick Revision Questions

1. **Calculate the error budget for a 99.95% SLO over a 30-day month.**
   > Error Budget = 1 − 0.9995 = 0.05%. In 30 days (43,200 min): 43,200 × 0.0005 = 21.6 minutes.

2. **What is a burn rate of 14.4x, and how long until the budget is exhausted?**
   > The error budget is being consumed 14.4 times faster than sustainable. 30 days / 14.4 ≈ 2 hours until exhaustion.

3. **How do error budgets align development and operations teams?**
   > Both teams share the same metric: dev wants budget (to ship features) and SRE wants budget (fewer incidents). The data decides, not opinions.

4. **Explain the two-window alerting approach (e.g., 1h + 5m).**
   > The long window (1h) confirms a sustained issue, the short window (5m) confirms it's happening now (not a past resolved event). Both must trigger for the alert to fire.

5. **What types of events are typically excluded from error budget consumption?**
   > Planned maintenance, external attacks (DDoS), and customer misuse.

6. **What should happen when a single incident exhausts the entire error budget?**
   > Feature freeze, mandatory postmortem, focus all efforts on reliability, and investigate how to prevent recurrence.

---

[← Previous: Setting SLO Targets](05-setting-slo-targets.md) | [Back to README](../README.md) | [Next: Availability Calculations →](../03-Reliability/01-availability-calculations.md)
