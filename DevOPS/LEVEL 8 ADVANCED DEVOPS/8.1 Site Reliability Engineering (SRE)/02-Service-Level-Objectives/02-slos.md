# Chapter 2.2: SLOs (Service Level Objectives)

[← Previous: SLIs](01-slis.md) | [Back to README](../README.md) | [Next: SLAs →](03-slas.md)

---

## Overview

A Service Level Objective (SLO) is a **target value or range for an SLI**. It represents the desired level of reliability for a service. SLOs are the most important concept in SRE — they are the decision-making framework that determines how fast a team can ship features and when to prioritize reliability.

> **SLOs answer: "How reliable should our service be?"**

---

## 1. SLO Definition

```
  SLO = SLI target over a time window

  Format:  "X% of [events] will [meet criteria] over [time window]"

  Examples:
  ┌──────────────────────────────────────────────────────┐
  │  "99.9% of HTTP requests will succeed                │
  │   over a 30-day rolling window"                      │
  │                                                      │
  │  "95% of search queries will complete in < 200ms     │
  │   over a 30-day rolling window"                      │
  │                                                      │
  │  "99.99% of stored objects will be retrievable       │
  │   over a calendar quarter"                           │
  └──────────────────────────────────────────────────────┘
```

---

## 2. Why SLOs Matter

```
  WITHOUT SLOs                       WITH SLOs
  ┌──────────────────────┐          ┌──────────────────────────┐
  │ "Is the service      │          │ "The service is at       │
  │  reliable enough?"   │          │  99.87% availability     │
  │                      │          │  against a 99.9% SLO.    │
  │  Team A: "Yes"       │          │                          │
  │  Team B: "No"        │          │  Error budget: 0.03%     │
  │  Manager: "Depends"  │          │  remaining = 13 min.     │
  │                      │          │                          │
  │  → Debates, no       │          │  → Clear: freeze risky   │
  │    resolution        │          │    deploys, fix issues"  │
  └──────────────────────┘          └──────────────────────────┘
```

### SLOs Enable:

1. **Objective decision-making** — no more subjective "is it reliable enough?"
2. **Prioritization** — error budget drives whether to focus on features or reliability
3. **Alignment** — dev, SRE, and business agree on a shared target
4. **Risk management** — quantified failure tolerance

---

## 3. Components of an SLO

```
  ┌────────────────────────────────────────────────────┐
  │                 SLO COMPONENTS                     │
  │                                                    │
  │  1. SLI (what are we measuring?)                   │
  │     e.g., request success rate                     │
  │                                                    │
  │  2. Target (what value do we aim for?)             │
  │     e.g., 99.9%                                    │
  │                                                    │
  │  3. Time Window (over what period?)                │
  │     e.g., 30-day rolling window                    │
  │                                                    │
  │  4. Error Budget (how much failure is allowed?)    │
  │     = 1 - SLO target                               │
  │     e.g., 0.1% = 43.2 min/month                   │
  │                                                    │
  └────────────────────────────────────────────────────┘
```

---

## 4. Time Windows

### Rolling vs Calendar Windows

```
  ROLLING WINDOW (recommended)
  ───────────────────────────────────────────────────►
  │◄──── 30 days ────►│
       │◄──── 30 days ────►│
            │◄──── 30 days ────►│
  
  • Slides forward continuously
  • Smooths out short-term volatility
  • Most useful for ongoing operations
  

  CALENDAR WINDOW
  ┌─────────┐┌─────────┐┌─────────┐
  │  Jan    ││  Feb    ││  Mar    │
  │         ││         ││         │
  └─────────┘└─────────┘└─────────┘
  
  • Fixed periods (month, quarter)
  • Resets at period boundary
  • Useful for reporting, business reviews
  
  [!] A bad day near month-end can burn through a 
      fresh calendar budget. Rolling windows prevent this.
```

---

## 5. Setting SLO Targets

### The Nines Table

```
  ┌──────────┬───────────────────┬──────────────────────┐
  │  SLO     │ Allowed Downtime  │ Allowed Downtime     │
  │  Target  │ per Month         │ per Year             │
  ├──────────┼───────────────────┼──────────────────────┤
  │  99%     │  7.2 hours        │  3.65 days           │
  │  99.5%   │  3.6 hours        │  1.83 days           │
  │  99.9%   │  43.2 minutes     │  8.76 hours          │
  │  99.95%  │  21.6 minutes     │  4.38 hours          │
  │  99.99%  │  4.32 minutes     │  52.6 minutes        │
  │  99.999% │  25.9 seconds     │  5.26 minutes        │
  └──────────┴───────────────────┴──────────────────────┘
```

### Guidelines for Choosing a Target

```
  ┌─────────────────────────────────────────────────────────┐
  │  CHOOSING THE RIGHT SLO TARGET                          │
  │                                                         │
  │  Start with current performance:                        │
  │    Measure actual SLI for 2-4 weeks                     │
  │    Set SLO slightly below observed performance          │
  │                                                         │
  │  Consider:                                              │
  │    ┌──────────────────────────────────────────┐         │
  │    │ Factor              │ Push Higher/Lower  │         │
  │    ├─────────────────────┼────────────────────┤         │
  │    │ Direct revenue      │ Higher SLO         │         │
  │    │ Safety-critical     │ Higher SLO         │         │
  │    │ Internal tool       │ Lower SLO ok       │         │
  │    │ Dependency SLOs     │ Can't exceed deps  │         │
  │    │ User expectations   │ Match competitors  │         │
  │    │ Cost of more nines  │ Balance cost/value │         │
  │    └─────────────────────┴────────────────────┘         │
  │                                                         │
  │  [!] You CANNOT have a higher SLO than your least       │
  │      reliable critical dependency!                      │
  │                                                         │
  │  If your DB is 99.9%, your service SLO cannot be 99.99% │
  └─────────────────────────────────────────────────────────┘
```

---

## 6. Multi-Window SLOs

To catch both slow burns and sudden outages, use multiple time windows:

```
  ┌──────────────────────────────────────────────────────┐
  │           MULTI-WINDOW SLO ALERTING                  │
  │                                                      │
  │  Window       │  Purpose                             │
  │  ─────────    │  ──────────────────────────────      │
  │  1-hour       │  Catch catastrophic failures fast    │
  │  6-hour       │  Catch sustained degradation         │
  │  3-day        │  Catch slow burns                    │
  │  30-day       │  Overall SLO compliance              │
  │                                                      │
  │  Alert when:                                         │
  │  ┌───────────────────────────────────────────┐      │
  │  │  Short window (1h) burn rate > 14.4x     │ PAGE │
  │  │  Medium window (6h) burn rate > 6x       │ PAGE │
  │  │  Long window (3d) burn rate > 1x         │TICKET│
  │  └───────────────────────────────────────────┘      │
  │                                                      │
  │  Burn rate = rate of error budget consumption        │
  │  1x = entire budget consumed in exactly 30 days      │
  └──────────────────────────────────────────────────────┘
```

### Burn Rate Alert Configuration (Prometheus)

```yaml
# Alert: High error budget burn rate
groups:
- name: slo-burn-rate
  rules:
  # Page: consuming budget 14.4x faster than sustainable (exhausted in ~2h)
  - alert: SLOBurnRateCritical
    expr: |
      (
        1 - (sum(rate(http_requests_total{status!~"5.."}[1h]))
        / sum(rate(http_requests_total[1h])))
      ) > (14.4 * 0.001)
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High SLO burn rate - budget exhausted in ~2 hours"

  # Ticket: consuming budget 3x faster than sustainable
  - alert: SLOBurnRateWarning
    expr: |
      (
        1 - (sum(rate(http_requests_total{status!~"5.."}[6h]))
        / sum(rate(http_requests_total[6h])))
      ) > (3 * 0.001)
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Elevated SLO burn rate - budget at risk"
```

---

## 7. SLO Document Template

```yaml
# SLO Document — Product Search API
service: product-search-api
owner: search-team
sre_contact: sre-platform
date: 2026-02-01

slos:
  - name: "Availability"
    sli: "Proportion of non-5xx HTTP responses"
    target: 99.9%
    window: 30-day rolling
    error_budget: "43.2 minutes/month"
    measurement: "Load balancer access logs"
    
  - name: "Latency (p99)"
    sli: "Proportion of requests completing in < 500ms"
    target: 99.0%
    window: 30-day rolling
    error_budget: "1% of requests may exceed 500ms"
    measurement: "Prometheus histogram"
    
  - name: "Latency (p50)"
    sli: "Proportion of requests completing in < 100ms"
    target: 95.0%
    window: 30-day rolling
    measurement: "Prometheus histogram"

error_budget_policy:
  budget_remaining_gt_50pct: "Normal development velocity"
  budget_remaining_20_50pct: "Extra review for risky changes"
  budget_remaining_lt_20pct: "Only reliability improvements"
  budget_exhausted: "Feature freeze until budget recovers"

review_cadence: "Monthly SLO review meeting"
```

---

## 8. Real-World Scenario

### [SCENARIO] SLO for a Payment Gateway

```
  ┌──────────────────────────────────────────────────────────┐
  │  Service: Payment Processing API                         │
  │  Business context: Each failure = lost revenue + trust   │
  │                                                          │
  │  SLO 1: Availability                                     │
  │    Target: 99.99% (4.32 min downtime/month)              │
  │    Why: Payment failures directly lose revenue            │
  │                                                          │
  │  SLO 2: Latency                                          │
  │    Target: 99% of transactions complete < 2 seconds      │
  │    Why: Users abandon checkout after 3 seconds            │
  │                                                          │
  │  SLO 3: Correctness                                      │
  │    Target: 100% (no error budget for wrong charges)       │
  │    Why: Charging wrong amounts has legal implications     │
  │                                                          │
  │  Dependencies:                                           │
  │    • Bank API: 99.95% availability (limits our SLO)      │
  │    • Database: 99.99% availability                       │
  │    • Network: 99.99% availability                        │
  │                                                          │
  │  Max achievable: min(99.95%, 99.99%, 99.99%)             │
  │                = 99.95% (limited by Bank API)             │
  │                                                          │
  │  [!] We set 99.99% but must accept that bank outages     │
  │      will consume most of our error budget                │
  └──────────────────────────────────────────────────────────┘
```

---

## 9. Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "SLOs are too strict, always violated" | Set SLO slightly below current measured performance. Tighten over time. |
| "SLOs are too loose, never violated" | Users expect better. Measure user complaints and tighten SLO. |
| "Nobody looks at SLO dashboards" | Include SLO status in weekly team meetings. Automate Slack reports. |
| "SLO doesn't reflect user pain" | Redefine SLIs to measure user journey, not just API responses. |
| "Dependency failures burn our budget" | Track budget burn by cause. Escalate dependency SLOs to provider. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **SLO Definition** | Target value for an SLI over a time window |
| **Format** | "X% of events meet criteria over Y time window" |
| **Error Budget** | 1 − SLO target (allowed failure) |
| **Time Windows** | Rolling (recommended) or Calendar |
| **Burn Rate** | Speed of error budget consumption (1x = 30-day burn) |
| **Multi-Window** | Short (1h) + Medium (6h) + Long (3d) for different alert severities |
| **Dependency Rule** | SLO ≤ min(dependency SLOs) |
| **Key Insight** | SLOs are the decision framework for balancing velocity and reliability |

---

## Quick Revision Questions

1. **Write the standard SLO format with an example.**
   > "X% of [events] will [criteria] over [window]." Example: "99.9% of HTTP requests will return non-5xx status over a 30-day rolling window."

2. **What is a burn rate, and what does 14.4x burn rate mean?**
   > Burn rate is how fast error budget is consumed relative to sustainable pace. 14.4x means the budget will be fully consumed in ~2 hours instead of 30 days.

3. **Why are rolling windows recommended over calendar windows?**
   > Rolling windows prevent a bad period near the start of a new calendar period from being masked by a budget reset.

4. **Why can't your service SLO exceed your least reliable dependency?**
   > If a dependency is 99.9% reliable, your service can't be more reliable than 99.9% since the dependency's downtime counts against your SLO.

5. **What are the four components of an SLO?**
   > SLI (what you measure), Target (the goal), Time Window (evaluation period), and Error Budget (allowed failure margin).

6. **What should happen when an error budget is exhausted?**
   > Feature deployments should freeze. The team focuses on reliability improvements until the budget recovers.

---

[← Previous: SLIs](01-slis.md) | [Back to README](../README.md) | [Next: SLAs →](03-slas.md)
