# Chapter 5: SRE Concepts — SLI, SLO, SLA & Error Budgets

[← Previous: Observability Maturity Model](04-observability-maturity-model.md) | [Back to README](../README.md) | [Next: Unit 2 — What Are Metrics? →](../02-Metrics/01-what-are-metrics.md)

---

## Overview

Site Reliability Engineering (SRE) provides a framework for managing reliability through **data-driven decisions**. The core concepts — **SLI, SLO, SLA, and Error Budgets** — form the backbone of how modern organizations balance reliability with feature velocity.

---

## 5.1 The SRE Reliability Hierarchy

```
┌─────────────────────────────────────────────────┐
│                    SLA                           │
│         (Legal/Business Agreement)               │
│    "We promise 99.9% uptime or we pay you"      │
│                                                  │
│    ┌───────────────────────────────────┐         │
│    │              SLO                  │         │
│    │     (Internal Target)             │         │
│    │  "We target 99.95% availability"  │         │
│    │                                   │         │
│    │    ┌─────────────────────┐        │         │
│    │    │        SLI          │        │         │
│    │    │   (Measurement)     │        │         │
│    │    │ "Currently 99.97%"  │        │         │
│    │    └─────────────────────┘        │         │
│    │                                   │         │
│    │    ┌─────────────────────┐        │         │
│    │    │   Error Budget      │        │         │
│    │    │  (Remaining room)   │        │         │
│    │    │ "0.05% left to use" │        │         │
│    │    └─────────────────────┘        │         │
│    └───────────────────────────────────┘         │
└─────────────────────────────────────────────────┘
```

---

## 5.2 SLI — Service Level Indicator

> **SLI** = A carefully defined quantitative measure of some aspect of the level of service provided.

### What Makes a Good SLI?

An SLI is a **ratio** expressed as:

```
SLI = (Good events / Total events) × 100%

Example:
SLI = (Successful HTTP requests / Total HTTP requests) × 100%
SLI = (Requests served < 300ms / Total requests) × 100%
```

### Common SLI Categories

```
┌───────────────────────────────────────────────────────────┐
│                    SLI CATEGORIES                          │
├────────────────┬──────────────────────────────────────────┤
│ Category       │ SLI Definition                           │
├────────────────┼──────────────────────────────────────────┤
│ Availability   │ % of requests that succeed (non-5xx)    │
│                │ Good: status != 5xx / Total requests    │
├────────────────┼──────────────────────────────────────────┤
│ Latency        │ % of requests faster than threshold     │
│                │ Good: latency < 300ms / Total requests  │
├────────────────┼──────────────────────────────────────────┤
│ Throughput     │ % of time system handles expected load   │
│                │ Good: rps > baseline / Total time        │
├────────────────┼──────────────────────────────────────────┤
│ Correctness    │ % of requests returning correct results │
│                │ Good: correct results / Total results    │
├────────────────┼──────────────────────────────────────────┤
│ Freshness      │ % of data updated within threshold      │
│                │ Good: data age < 1min / Total records    │
├────────────────┼──────────────────────────────────────────┤
│ Durability     │ % of data retrievable when expected      │
│                │ Good: data found / Total expected data   │
└────────────────┴──────────────────────────────────────────┘
```

### Measuring SLIs with Prometheus

```promql
# Availability SLI
sum(rate(http_requests_total{status!~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))

# Latency SLI (% requests under 300ms)
sum(rate(http_request_duration_seconds_bucket{le="0.3"}[5m]))
/
sum(rate(http_request_duration_seconds_count[5m]))

# Current SLI value over 30 days
avg_over_time(
  (
    sum(rate(http_requests_total{status!~"5.."}[5m]))
    /
    sum(rate(http_requests_total[5m]))
  )[30d:5m]
)
```

---

## 5.3 SLO — Service Level Objective

> **SLO** = A target value (or range) for a service level as measured by an SLI.

### SLO = SLI + Target + Window

```
SLO = "99.9% of requests succeed over a 30-day rolling window"
       ─────                         ─────────────────────────
       Target                        Measurement Window

SLO = "95% of requests complete in under 300ms over 7 days"
```

### The Nines Table

```
┌──────────┬───────────────┬───────────────┬──────────────────┐
│ Uptime % │ Downtime/Year │ Downtime/Month│ Downtime/Day     │
├──────────┼───────────────┼───────────────┼──────────────────┤
│ 99%      │ 3.65 days     │ 7.31 hours    │ 14.4 minutes     │
│ 99.9%    │ 8.77 hours    │ 43.83 min     │ 1.44 minutes     │
│ 99.95%   │ 4.38 hours    │ 21.92 min     │ 43.2 seconds     │
│ 99.99%   │ 52.6 minutes  │ 4.38 min      │ 8.64 seconds     │
│ 99.999%  │ 5.26 minutes  │ 26.3 sec      │ 864 milliseconds │
└──────────┴───────────────┴───────────────┴──────────────────┘

💡 Each additional "nine" is roughly 10x harder and more expensive!
```

### How to Choose an SLO

```
┌─────────────────────────────────────────────────────────┐
│  Choosing the Right SLO Target:                         │
│                                                          │
│  Too Low (e.g., 99%):                                   │
│  • Users have poor experience                           │
│  • Business loses revenue                               │
│  • Doesn't reflect user expectations                    │
│                                                          │
│  Too High (e.g., 99.999%):                             │
│  • Extremely expensive to achieve                       │
│  • Engineering velocity grinds to a halt               │
│  • False sense of "reliability theater"                 │
│                                                          │
│  Just Right:                                            │
│  • Slightly better than current performance             │
│  • Reflects actual user impact thresholds               │
│  • Allows room for innovation (error budget)            │
│                                                          │
│  Rule of thumb: SLO should be stricter than SLA         │
│  If SLA = 99.9%, set SLO = 99.95%                      │
└─────────────────────────────────────────────────────────┘
```

---

## 5.4 SLA — Service Level Agreement

> **SLA** = A contract with users that includes consequences (typically financial) for meeting or missing the SLO.

### SLA vs SLO

```
┌──────────────────────────────────────────────────────────┐
│                                                           │
│   SLA (External Promise — has consequences)              │
│   ┌────────────────────────────────────────────┐         │
│   │ "We guarantee 99.9% availability.          │         │
│   │  If we fail, we credit your account 10%."  │         │
│   └────────────────────────────────────────────┘         │
│                                                           │
│   SLO (Internal Target — no external consequence)        │
│   ┌────────────────────────────────────────────┐         │
│   │ "We target 99.95% availability internally. │         │
│   │  This gives us buffer above the SLA."      │         │
│   └────────────────────────────────────────────┘         │
│                                                           │
│   SLI (Measurement)                                      │
│   ┌────────────────────────────────────────────┐         │
│   │ "Our current availability is 99.97%."      │         │
│   │  We're meeting both SLO and SLA.           │         │
│   └────────────────────────────────────────────┘         │
│                                                           │
│   Relationship:                                          │
│   SLI ──measures──► SLO ──informs──► SLA                │
│                                                           │
│   Typically:                                             │
│   SLA < SLO < Actual Performance                        │
│   99.9%  < 99.95% < 99.97%                              │
└──────────────────────────────────────────────────────────┘
```

### Common SLA Structures

| Provider | Uptime SLA | Credit |
|----------|-----------|--------|
| AWS EC2 | 99.99% | 10-30% credit for violations |
| Google Cloud | 99.95% | 10-50% credit |
| Azure | 99.95-99.99% | 10-100% credit |
| Stripe API | 99.99% | Not financial (SLO published) |

---

## 5.5 Error Budgets

> **Error Budget** = The maximum amount of unreliability you can tolerate while still meeting your SLO.

### Calculating Error Budget

```
Error Budget = 1 - SLO Target

If SLO = 99.9% availability:
  Error Budget = 1 - 0.999 = 0.1%

Over 30 days:
  Error Budget = 30 days × 24 hours × 60 minutes × 0.001
               = 43.2 minutes of downtime allowed

Over 30 days with 1M requests:
  Error Budget = 1,000,000 × 0.001 = 1,000 failed requests allowed
```

### Error Budget Visualization

```
SLO Target: 99.9% over 30 days
Error Budget: 43.2 minutes

Day 1                                              Day 30
├──────────────────────────────────────────────────────┤

Error Budget Remaining:
█████████████████████████████████████████████░░░░░░░░░░
100%                                    Consumed ──►  0%

Day  5: 5min incident    ─── Budget: 38.2 min remaining (88%)
Day 12: 2min blip        ─── Budget: 36.2 min remaining (84%)
Day 18: 15min outage     ─── Budget: 21.2 min remaining (49%)
Day 22: Deploy freeze!   ─── Budget getting low
Day 25: 10min incident   ─── Budget: 11.2 min remaining (26%)
Day 28: ⚠️ BUDGET CRITICAL ─── Stop risky changes
```

### Error Budget Policy

```
┌─────────────────────────────────────────────────────────┐
│              ERROR BUDGET POLICY                         │
├──────────────────┬──────────────────────────────────────┤
│ Budget > 75%     │ ✅ Normal operations                 │
│                  │ • Deploy freely                      │
│                  │ • Run experiments                    │
│                  │ • Try new features                   │
├──────────────────┼──────────────────────────────────────┤
│ Budget 50-75%    │ ⚠️ Caution                          │
│                  │ • Continue deploying but carefully   │
│                  │ • Prioritize reliability work        │
│                  │ • Review recent incidents            │
├──────────────────┼──────────────────────────────────────┤
│ Budget 25-50%    │ 🟡 Reduced velocity                  │
│                  │ • Only well-tested changes           │
│                  │ • Focus on reliability improvements  │
│                  │ • Increase testing coverage          │
├──────────────────┼──────────────────────────────────────┤
│ Budget < 25%     │ 🔴 Freeze                            │
│                  │ • No new features deployed           │
│                  │ • Only reliability improvements      │
│                  │ • Postmortem all recent incidents     │
├──────────────────┼──────────────────────────────────────┤
│ Budget = 0%      │ 🚫 Full stop                         │
│                  │ • Emergency reliability work only    │
│                  │ • Escalate to leadership             │
│                  │ • SLO review needed                  │
└──────────────────┴──────────────────────────────────────┘
```

---

## 5.6 SLO-Based Alerting

Traditional alerting uses static thresholds. SLO-based alerting uses **burn rate**:

```
Burn Rate = rate of error budget consumption

If burn rate = 1:  Budget consumed exactly on schedule (just meets SLO)
If burn rate = 2:  Budget consumed 2x faster (will exhaust in 15 days)
If burn rate = 10: Budget consumed 10x faster (will exhaust in 3 days)
If burn rate = 36: Budget consumed 36x faster (emergency!)
```

### Multi-Window, Multi-Burn-Rate Alerting

```yaml
# Prometheus alerting rules for SLO burn rate
groups:
  - name: slo-alerts
    rules:
      # Critical: high burn rate over short window
      - alert: HighErrorBudgetBurn
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[1h]))
            / sum(rate(http_requests_total[1h]))
          ) > (14.4 * 0.001)
          and
          (
            sum(rate(http_requests_total{status=~"5.."}[5m]))
            / sum(rate(http_requests_total[5m]))
          ) > (14.4 * 0.001)
        labels:
          severity: critical
        annotations:
          summary: "High error budget burn rate (>14.4x)"
          
      # Warning: moderate burn rate over longer window
      - alert: ErrorBudgetBurnWarning
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[6h]))
            / sum(rate(http_requests_total[6h]))
          ) > (6 * 0.001)
          and
          (
            sum(rate(http_requests_total{status=~"5.."}[30m]))
            / sum(rate(http_requests_total[30m]))
          ) > (6 * 0.001)
        labels:
          severity: warning
        annotations:
          summary: "Elevated error budget burn rate (>6x)"
```

---

## 5.7 Putting It All Together

```
┌──────────────────────────────────────────────────────────────┐
│              SRE RELIABILITY WORKFLOW                         │
│                                                               │
│  1. Define SLIs                                              │
│     "What matters to our users?"                             │
│     ├── Availability: Can they use the service?              │
│     ├── Latency: Is it fast enough?                          │
│     └── Correctness: Are results accurate?                   │
│                                                               │
│  2. Set SLOs                                                  │
│     "What target should we aim for?"                         │
│     ├── 99.9% availability over 30 days                     │
│     └── 95% of requests < 300ms over 30 days                │
│                                                               │
│  3. Calculate Error Budget                                   │
│     "How much room do we have?"                              │
│     └── 43.2 minutes of downtime per month                  │
│                                                               │
│  4. Set Up Alerting                                          │
│     "When should we act?"                                    │
│     ├── Burn rate > 14x → Page on-call (critical)           │
│     └── Burn rate > 6x  → Create ticket (warning)           │
│                                                               │
│  5. Apply Error Budget Policy                                │
│     "What changes based on budget remaining?"                │
│     ├── > 50%: Ship features freely                         │
│     └── < 25%: Freeze features, fix reliability             │
│                                                               │
│  6. Review & Iterate                                         │
│     "Were our SLOs right?"                                   │
│     ├── Monthly SLO review meeting                          │
│     ├── Adjust targets based on user feedback               │
│     └── Postmortem for every SLO violation                  │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.8 Real-World Scenario

🌍 **Scenario: Managing Error Budget for a Payment Service**

```
Service: payment-api
SLO: 99.95% availability over 30 days
Error Budget: 21.6 minutes/month

Week 1:
├── Deployed new payment gateway integration
├── 3-minute outage during rollout (bad config)
├── Budget remaining: 18.6 min (86%)
└── Action: Postmortem, add config validation

Week 2:
├── Normal operations
├── 0 downtime
├── Budget remaining: 18.6 min (86%)
└── Action: Ship new features, run experiments

Week 3:
├── Database failover took 8 minutes
├── Budget remaining: 10.6 min (49%)
├── Entered "caution" zone
└── Action: Prioritize DB failover automation

Week 4:
├── Another 7 minutes of downtime
├── Budget remaining: 3.6 min (17%)
├── BUDGET CRITICAL!
└── Action: Feature freeze, all hands on reliability

Month-end Review:
├── SLO met (barely): actual = 99.96%
├── But used 83% of budget
├── Decision: Invest in automated DB failover
└── Next month target: Reduce incidents by 50%
```

---

## 5.9 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| SLO always violated | Target too aggressive | Lower SLO to achievable level, then improve |
| Error budget never used | SLO too lenient | Tighten SLO to push innovation |
| Team ignores error budget | No policy enforcement | Get management buy-in for error budget policy |
| SLI doesn't reflect user pain | Wrong metric chosen | Measure at user-facing boundary, not internal |
| Burn rate alerts too noisy | Window too short | Use multi-window alerting (5m AND 1h) |

---

## Summary Table

| Concept | Definition | Example |
|---------|-----------|---------|
| **SLI** | Quantitative measure of service performance | 99.97% of requests succeed |
| **SLO** | Target value for an SLI over a time window | 99.9% availability over 30 days |
| **SLA** | Contract with consequences for missing SLO | 99.9% uptime or 10% credit |
| **Error Budget** | Maximum allowed unreliability (1 - SLO) | 43.2 min downtime/month |
| **Burn Rate** | Speed of error budget consumption | 2x = budget gone in 15 days |
| **Error Budget Policy** | Actions based on remaining budget | <25% → feature freeze |

---

## Quick Revision Questions

1. **Define SLI, SLO, and SLA. How do they relate to each other? Draw the hierarchy.**

2. **If your SLO is 99.95% availability over a 30-day window, calculate the error budget in minutes.**

3. **What is a "burn rate" in the context of SLO-based alerting? Why is it superior to simple threshold-based alerts?**

4. **Describe the error budget policy. What actions should a team take when the error budget drops below 25%?**

5. **Why should the SLO always be stricter than the SLA? What problems arise if they're set equal?**

6. **Give an example of how error budgets help balance reliability work with feature development.**

---

[← Previous: Observability Maturity Model](04-observability-maturity-model.md) | [Back to README](../README.md) | [Next: Unit 2 — What Are Metrics? →](../02-Metrics/01-what-are-metrics.md)
