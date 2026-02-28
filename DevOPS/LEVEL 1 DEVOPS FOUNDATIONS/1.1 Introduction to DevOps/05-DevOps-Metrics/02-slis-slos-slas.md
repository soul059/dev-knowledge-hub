# Chapter 5.2 — SLIs, SLOs, and SLAs

[← Previous: DORA Metrics](01-dora-metrics.md) | [Next: Error Budgets →](03-error-budgets.md)

---

## Overview

**SLIs, SLOs, and SLAs** form a hierarchy for measuring and agreeing on service reliability. Originating from Google's Site Reliability Engineering (SRE) practices, they provide a data-driven approach to balancing reliability and development velocity.

---

## The SLI → SLO → SLA Hierarchy

```
┌──────────────────────────────────────────────────────────┐
│  SLI → SLO → SLA HIERARCHY                              │
│                                                          │
│  ┌────────────────────────────────────────────────┐      │
│  │  SLA (Service Level Agreement)                 │      │
│  │  "External contract with customers"             │      │
│  │  "99.9% uptime or we give credits"             │      │
│  │  ┌──────────────────────────────────────┐      │      │
│  │  │  SLO (Service Level Objective)       │      │      │
│  │  │  "Internal target for the team"       │      │      │
│  │  │  "99.95% availability (stricter)"    │      │      │
│  │  │  ┌────────────────────────────┐      │      │      │
│  │  │  │  SLI (Service Level       │      │      │      │
│  │  │  │      Indicator)           │      │      │      │
│  │  │  │  "What we actually        │      │      │      │
│  │  │  │   measure"               │      │      │      │
│  │  │  │  "% of successful        │      │      │      │
│  │  │  │   requests"              │      │      │      │
│  │  │  └────────────────────────────┘      │      │      │
│  │  └──────────────────────────────────────┘      │      │
│  └────────────────────────────────────────────────┘      │
│                                                          │
│  SLI = the metric (data)                                 │
│  SLO = the target (threshold for the metric)             │
│  SLA = the contract (consequences for missing target)    │
└──────────────────────────────────────────────────────────┘
```

---

## SLI — Service Level Indicator

```
┌──────────────────────────────────────────────────────────┐
│  SLI = A quantitative measure of service behavior        │
│                                                          │
│  Common SLIs:                                            │
│                                                          │
│  AVAILABILITY:                                           │
│  successful requests / total requests × 100              │
│  Example: 99,950 / 100,000 = 99.95%                     │
│                                                          │
│  LATENCY:                                                │
│  % of requests faster than threshold                     │
│  Example: 95% of requests < 200ms                        │
│                                                          │
│  THROUGHPUT:                                             │
│  Requests processed per second                           │
│  Example: 10,000 RPS sustained                           │
│                                                          │
│  ERROR RATE:                                             │
│  failed requests / total requests × 100                  │
│  Example: 50 / 100,000 = 0.05%                           │
│                                                          │
│  Good SLIs are:                                          │
│  ├── Measurable (concrete number)                        │
│  ├── User-facing (what users experience)                 │
│  ├── Meaningful (tied to user happiness)                 │
│  └── Not vanity metrics ("server CPU" is not a good SLI) │
└──────────────────────────────────────────────────────────┘
```

---

## SLO — Service Level Objective

```
┌──────────────────────────────────────────────────────────┐
│  SLO = SLI + Target + Time Window                        │
│                                                          │
│  Formula: "SLI >= target over time window"               │
│                                                          │
│  EXAMPLES:                                               │
│  ├── "99.95% of requests successful over 30 days"        │
│  ├── "95% of requests < 200ms p95 over 30 days"         │
│  ├── "99.9% of API calls return in < 500ms"             │
│  └── "99.99% of data writes are durable"                 │
│                                                          │
│  NINES TABLE (what the numbers really mean):             │
│  ┌──────────┬──────────────┬──────────────────┐         │
│  │ SLO      │ Downtime/mo  │ Downtime/year    │         │
│  ├──────────┼──────────────┼──────────────────┤         │
│  │ 99%      │ 7.3 hours    │ 3.65 days        │         │
│  │ 99.9%    │ 43.8 min     │ 8.77 hours       │         │
│  │ 99.95%   │ 21.9 min     │ 4.38 hours       │         │
│  │ 99.99%   │ 4.38 min     │ 52.6 minutes     │         │
│  │ 99.999%  │ 26.3 sec     │ 5.26 minutes     │         │
│  └──────────┴──────────────┴──────────────────┘         │
│                                                          │
│  KEY INSIGHT: Going from 99.9% to 99.99% is 10× harder │
│  and 10× more expensive. Choose your SLO carefully!     │
└──────────────────────────────────────────────────────────┘
```

---

## SLA — Service Level Agreement

```
┌──────────────────────────────────────────────────────────┐
│  SLA = SLO + Consequences (legal/financial)              │
│                                                          │
│  EXAMPLE SLA (cloud provider):                           │
│  ┌──────────────────────────────────────────────┐       │
│  │  "We guarantee 99.9% monthly uptime.         │       │
│  │   If we fail:                                 │       │
│  │   ├── 99.0% - 99.9%  → 10% service credit   │       │
│  │   ├── 95.0% - 99.0%  → 25% service credit   │       │
│  │   └── < 95.0%        → 50% service credit"  │       │
│  └──────────────────────────────────────────────┘       │
│                                                          │
│  IMPORTANT RULES:                                        │
│  ├── SLO should be STRICTER than SLA                     │
│  │   SLA: 99.9%  →  SLO: 99.95%                        │
│  │   (buffer to catch issues before SLA breach)         │
│  │                                                       │
│  ├── Not every service needs an SLA                      │
│  │   Internal services: SLOs only                       │
│  │   Customer-facing: SLOs + SLA                        │
│  │                                                       │
│  └── SLA breaches have real financial cost               │
│      (credits, penalties, lost customers)               │
└──────────────────────────────────────────────────────────┘
```

---

## Practical Example: E-Commerce Platform

```
┌──────────────────────────────────────────────────────────┐
│  E-COMMERCE PLATFORM SLI/SLO/SLA EXAMPLE                 │
│                                                          │
│  SERVICE: Checkout API                                   │
│                                                          │
│  SLIs:                                                   │
│  ├── Availability: % of 2xx responses                    │
│  ├── Latency: p99 response time                          │
│  └── Error rate: % of 5xx responses                      │
│                                                          │
│  SLOs:                                                   │
│  ├── Availability: ≥ 99.95% over 30 days                 │
│  ├── Latency: p99 < 500ms over 30 days                   │
│  └── Error rate: ≤ 0.1% over 30 days                     │
│                                                          │
│  SLA:                                                    │
│  "Partners integrating with our checkout API              │
│   are guaranteed 99.9% availability.                     │
│   Breach → 10% credit on monthly invoice"                │
│                                                          │
│  Why SLO (99.95%) > SLA (99.9%):                         │
│  ├── 30-day SLO budget: 21.9 min downtime                │
│  ├── 30-day SLA limit:  43.8 min downtime                │
│  ├── Buffer: 21.9 minutes of "safety margin"             │
│  └── Alert when SLO threatened, fix before SLA breach    │
└──────────────────────────────────────────────────────────┘
```

---

## Measuring SLIs with Prometheus

```yaml
# Prometheus recording rules for SLIs
groups:
  - name: sli-rules
    rules:
      # Availability SLI: % of successful HTTP requests
      - record: sli:availability:ratio
        expr: |
          sum(rate(http_requests_total{status!~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m]))

      # Latency SLI: % of requests under 200ms
      - record: sli:latency:ratio
        expr: |
          sum(rate(http_request_duration_seconds_bucket{le="0.2"}[5m]))
          /
          sum(rate(http_request_duration_seconds_count[5m]))

  - name: slo-alerts
    rules:
      # Alert when approaching SLO violation
      - alert: SLOAvailabilityBreach
        expr: sli:availability:ratio < 0.9995
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Availability SLO at risk"
          description: "Current availability: {{ $value | humanizePercentage }}"
```

---

## Real-World Scenario: SLO-Based Alerting

```
TRADITIONAL ALERTING:             SLO-BASED ALERTING:
═══════════════════               ═══════════════════

"CPU > 80%"                       "Error budget burn rate
"Memory > 90%"                     > 10x normal"
"Disk > 85%"

100 alerts/week                   5 alerts/week
80% are noise                     90% are actionable
Team ignores alerts               Team trusts alerts

SLO-BASED ALERT EXAMPLE:
├── 30-day error budget = 0.05% (for 99.95% SLO)
├── That means ~21.9 minutes of allowed downtime/month
├── Normal burn rate: 1.0 (using budget evenly)
├── Alert when burn rate > 14.4 (burning 1-hour budget)
│   → "At this rate, SLO breached in 1 hour"
├── Alert when burn rate > 6 (burning 6-hour budget)
│   → "At this rate, SLO breached in 6 hours — investigate"
└── This approach = fewer alerts, more meaningful
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Too many SLO breaches | SLO is too aggressive | Start conservative (99.9%), tighten over time |
| SLO never breached | SLO is too lenient | Tighten the SLO or add new dimensions (latency) |
| Team ignores SLOs | No consequences or visibility | Display SLO dashboards; tie to error budgets |
| SLI doesn't reflect users | Measuring wrong thing | Measure from user's perspective (not server CPU) |
| SLA breaches frequent | SLO not stricter than SLA | SLO must have buffer above SLA (e.g., 99.95% vs 99.9%) |
| Alert fatigue from SLOs | Too many burn-rate alerts | Use multi-window burn rate alerting (short + long windows) |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **SLI** | Service Level Indicator — a quantitative measure (e.g., % successful requests) |
| **SLO** | Service Level Objective — a target for the SLI (e.g., ≥ 99.95% over 30 days) |
| **SLA** | Service Level Agreement — a contract with consequences (e.g., credits if < 99.9%) |
| **Nines** | Shorthand for availability (99.9% = "three nines") |
| **Error Budget** | Allowed unreliability = 100% − SLO (e.g., 0.05% for 99.95% SLO) |
| **Burn Rate** | Speed at which error budget is consumed |

---

## Quick Revision Questions

1. **What is the relationship between SLI, SLO, and SLA?**
   <details><summary>Answer</summary>SLI is the metric you measure (e.g., request success rate). SLO is the target for that metric (e.g., 99.95% over 30 days). SLA is an external contract that adds consequences for missing the target (e.g., billing credits). They form a hierarchy: SLI → SLO → SLA. The SLO should always be stricter than the SLA to provide a buffer.</details>

2. **What is the difference between 99.9% and 99.99% availability?**
   <details><summary>Answer</summary>99.9% ("three nines") allows 43.8 minutes of downtime per month (8.77 hours/year). 99.99% ("four nines") allows only 4.38 minutes per month (52.6 minutes/year). That's 10× less downtime and requires significantly more engineering investment — redundancy, automated failover, zero-downtime deployments, etc.</details>

3. **Why should the SLO be stricter than the SLA?**
   <details><summary>Answer</summary>The SLO acts as an early warning. If SLO = 99.95% and SLA = 99.9%, the team gets alerted when availability drops below 99.95% — giving them a 21.9-minute buffer to fix the issue before the SLA is breached. Without this buffer, the team only learns about problems when they're already causing contract violations and financial penalties.</details>

4. **What makes a good SLI?**
   <details><summary>Answer</summary>A good SLI: 1) Measures what users experience (not internal metrics like CPU). 2) Is quantifiable as a ratio or percentage. 3) Is meaningful for service health. 4) Can be measured accurately and automatically. Example good SLI: "% of requests completing in under 200ms." Example bad SLI: "average server CPU usage" (doesn't correlate with user experience).</details>

5. **What is the "nines" system, and why does each additional nine matter?**
   <details><summary>Answer</summary>Nines measure availability: 99% = two nines, 99.9% = three nines, etc. Each additional nine reduces allowed downtime by 10× and typically requires 10× more engineering effort. Going from 99.9% to 99.99% means going from 43.8 min/month to 4.38 min/month — requiring automated failover, multi-region deployment, and extreme operational discipline.</details>

6. **How do you measure SLIs using Prometheus?**
   <details><summary>Answer</summary>Use Prometheus recording rules to calculate SLIs from raw metrics. For availability: `sum(rate(http_requests_total{status!~"5.."}[5m])) / sum(rate(http_requests_total[5m]))`. For latency: use histogram buckets to calculate the percentage of requests under a threshold. Record these as new time series and set alerting rules based on SLO thresholds.</details>

---

[← Previous: DORA Metrics](01-dora-metrics.md) | [Next: Error Budgets →](03-error-budgets.md)
