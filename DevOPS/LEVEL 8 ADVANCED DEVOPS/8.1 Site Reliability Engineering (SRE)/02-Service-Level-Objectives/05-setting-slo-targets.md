# Chapter 2.5: Setting SLO Targets

[← Previous: Choosing SLIs](04-choosing-slis.md) | [Back to README](../README.md) | [Next: Error Budgets →](06-error-budgets.md)

---

## Overview

Setting the right SLO target is a balance between user expectations, business needs, engineering cost, and dependency constraints. Too strict and you'll spend all resources on reliability; too loose and users will leave. This chapter provides a practical methodology for setting and iterating on SLO targets.

---

## 1. The SLO Target Methodology

```
  ┌─────────────────────────────────────────────────────┐
  │          SLO TARGET SETTING PROCESS                 │
  │                                                     │
  │  Step 1: MEASURE current performance (2-4 weeks)    │
  │     └── "What is the service actually delivering?"  │
  │                                                     │
  │  Step 2: ASSESS user expectations                   │
  │     └── Surveys, support tickets, competitor SLAs   │
  │                                                     │
  │  Step 3: MAP dependency SLOs                        │
  │     └── Your SLO ≤ min(dependency SLOs)             │
  │                                                     │
  │  Step 4: SET initial target (conservative)          │
  │     └── Slightly below current performance          │
  │                                                     │
  │  Step 5: IMPLEMENT error budget policy              │
  │     └── What happens when budget runs out?          │
  │                                                     │
  │  Step 6: ITERATE quarterly                          │
  │     └── Tighten or loosen based on data             │
  └─────────────────────────────────────────────────────┘
```

---

## 2. Step 1: Measure Current Performance

```bash
# Prometheus query: 30-day availability
# (run against your Prometheus/Grafana)

# Current availability over 30 days
avg_over_time(sli:api_availability:ratio_5m[30d]) * 100

# Example output: 99.94%
# → This means your service currently delivers ~99.94% availability
```

```
  Measurement Results (Example):
  
  ┌──────────────┬─────────────┬───────────────────────┐
  │ SLI          │ Measured    │ Notes                 │
  │              │ Value       │                       │
  ├──────────────┼─────────────┼───────────────────────┤
  │ Availability │ 99.94%      │ 2 outages in 30 days  │
  │ Latency p50  │ 45ms        │ Well within targets   │
  │ Latency p99  │ 380ms       │ Occasionally spikes   │
  │ Error rate   │ 0.06%       │ Mostly 503s           │
  └──────────────┴─────────────┴───────────────────────┘
  
  [!] Don't set SLO at current performance.
      Set it BELOW to allow for natural variation.
      
      Measured: 99.94%  →  Initial SLO: 99.9%
```

---

## 3. Step 2: Assess User Expectations

```
  ┌───────────────────────────────────────────────────────┐
  │        USER EXPECTATION SOURCES                       │
  │                                                       │
  │  1. Support tickets                                   │
  │     "How many reliability complaints do we get?"      │
  │     "At what SLI level do complaints spike?"          │
  │                                                       │
  │  2. User research / surveys                           │
  │     "How fast do users expect pages to load?"         │
  │     "How many errors will they tolerate?"             │
  │                                                       │
  │  3. Competitor benchmarks                             │
  │     "What SLAs do competitors publish?"               │
  │     "What uptime do industry leaders deliver?"        │
  │                                                       │
  │  4. Business context                                  │
  │     "What does downtime cost in revenue?"             │
  │     "Are there regulatory requirements?"              │
  │                                                       │
  │  ┌─────────────────────────────────────────┐          │
  │  │  Complaint Rate vs Availability         │          │
  │  │                                         │          │
  │  │  Complaints ▲                           │          │
  │  │             │     *                     │          │
  │  │             │     *                     │          │
  │  │             │      *                    │          │
  │  │             │       **                  │          │
  │  │             │         ***               │          │
  │  │             │            *****          │          │
  │  │             └────────────────────► SLI  │          │
  │  │              99%  99.5% 99.9% 99.99%   │          │
  │  │                                         │          │
  │  │  "Happiness cliff" — the point where    │          │
  │  │  complaints spike dramatically          │          │
  │  └─────────────────────────────────────────┘          │
  └───────────────────────────────────────────────────────┘
```

---

## 4. Step 3: Dependency Analysis

```
  ┌──────────────────────────────────────────────────────┐
  │            DEPENDENCY SLO CHAIN                      │
  │                                                      │
  │  Your Service                                        │
  │       │                                              │
  │       ├──► Database (SLO: 99.99%)                    │
  │       │                                              │
  │       ├──► Auth Service (SLO: 99.95%)   ◄── weakest │
  │       │                                              │
  │       ├──► Cache Layer (SLO: 99.99%)                 │
  │       │                                              │
  │       └──► CDN (SLA: 99.9%)                          │
  │                                                      │
  │  Serial dependencies multiply:                       │
  │  Max achievable = 99.99% × 99.95% × 99.99% × 99.9% │
  │                 = ~99.83%                             │
  │                                                      │
  │  [!] Your SLO cannot realistically exceed ~99.83%    │
  │      Set SLO at or below this ceiling                │
  └──────────────────────────────────────────────────────┘
```

### Serial vs Parallel Dependencies

```
  SERIAL (all must work):
  A ──► B ──► C
  Combined SLO = SLO_A × SLO_B × SLO_C
  99.9% × 99.9% × 99.9% = 99.7%

  PARALLEL (any one works, with failover):
  A ──┐
      ├──► Result
  B ──┘
  Combined SLO = 1 - (1-SLO_A)(1-SLO_B)
  1 - (0.001)(0.001) = 99.9999%
```

---

## 5. Step 4: Set the Initial Target

### Decision Matrix

```
  ┌──────────────────────────────────────────────────────────┐
  │                 SLO TARGET GUIDE                         │
  │                                                          │
  │  Service Type           │ Typical Availability SLO       │
  │  ──────────────────     │ ────────────────────────       │
  │  Internal dashboard     │ 99.0% - 99.5%                 │
  │  Internal API           │ 99.5% - 99.9%                 │
  │  Customer-facing web    │ 99.9% - 99.95%                │
  │  Payment/financial      │ 99.95% - 99.99%               │
  │  Healthcare/safety      │ 99.99%+                        │
  │                                                          │
  │  Service Type           │ Typical Latency SLO            │
  │  ──────────────────     │ ────────────────────────       │
  │  User-facing page load  │ 95% of requests < 200ms       │
  │  API response           │ 99% of requests < 500ms       │
  │  Batch processing       │ 95% of jobs complete < 10min  │
  │  Search results         │ 99% of queries < 300ms        │
  │  Real-time streaming    │ 99.9% of events < 100ms       │
  └──────────────────────────────────────────────────────────┘
```

### The "Start Conservative" Rule

```
  Current measured: 99.94%
  
  ✗ Don't set SLO at 99.94%  (no room for variation)
  ✗ Don't set SLO at 99.99%  (aspirational but unrealistic)
  ✓ Set SLO at 99.9%         (gives healthy error budget)
  
  Why?
  ┌─────────────────────────────────────────┐
  │  99.94% measured                        │
  │  99.9%  SLO        ← buffer for        │
  │                       natural variation │
  │                                         │
  │  Error budget = 0.1% = 43 min/month     │
  │  Room for 1-2 incidents                 │
  │  Room for risky deployments             │
  └─────────────────────────────────────────┘
```

---

## 6. Step 5: Error Budget Policy

```yaml
# Error Budget Policy Document
error_budget_policy:
  service: "product-api"
  slo: "99.9% availability, 30-day rolling"
  
  thresholds:
    green:
      condition: "Error budget > 50%"
      actions:
        - "Normal development velocity"
        - "Standard change approval process"
        - "Experimentation encouraged"
        
    yellow:
      condition: "Error budget 20-50%"
      actions:
        - "Risky changes require additional review"
        - "Reduce deployment frequency"
        - "Investigate ongoing reliability issues"
        
    orange:
      condition: "Error budget 5-20%"
      actions:
        - "Only critical fixes and reliability improvements"
        - "All changes require SRE review"
        - "Rollback poorly performing recent changes"
        
    red:
      condition: "Error budget < 5% or exhausted"
      actions:
        - "Feature freeze"
        - "All hands on reliability"
        - "Postmortem required if budget exhausted by incident"
        - "Executive review weekly"
        
  escalation:
    - "Budget < 20%: SRE Manager notified"
    - "Budget exhausted: VP Engineering review"
    - "Budget repeatedly exhausted: re-evaluate SLO or architecture"
```

---

## 7. Step 6: Iterate and Refine

```
  ┌──────────────────────────────────────────────────────┐
  │           SLO ITERATION CYCLE (Quarterly)            │
  │                                                      │
  │  Quarter 1: Initial SLO set                          │
  │     │                                                │
  │     ▼                                                │
  │  Quarter 2: Review                                   │
  │     ├── SLO violated frequently? → Loosen target     │
  │     │   or invest in reliability                     │
  │     ├── SLO never violated? → Maybe tighten          │
  │     │   (but only if users expect better)            │
  │     └── SLO matched user expectations? → Keep it     │
  │     │                                                │
  │     ▼                                                │
  │  Quarter 3: Adjust & re-communicate                  │
  │     │                                                │
  │     ▼                                                │
  │  Repeat...                                           │
  │                                                      │
  │  [!] NEVER tighten SLO just because you can.         │
  │      Only tighten if users/business need it.         │
  │      Each nine costs 10x more to maintain.           │
  └──────────────────────────────────────────────────────┘
```

### SLO Review Meeting Agenda

```
  SLO Review Meeting (30 min, monthly)
  
  1. SLO Status Dashboard Review          (5 min)
     - Current SLI values vs SLO targets
     - Error budget remaining
     
  2. Incidents & Budget Burns              (10 min)
     - What consumed error budget?
     - Postmortem action items progress
     
  3. User Feedback & Complaints            (5 min)
     - Support ticket trends
     - NPS/satisfaction related to reliability
     
  4. SLO Target Adjustments (if any)       (5 min)
     - Propose changes with justification
     
  5. Action Items                          (5 min)
     - Reliability investments needed
     - Process changes
```

---

## 8. Real-World Scenario

### [SCENARIO] Setting SLOs for a Multi-Tier Application

```
  Application: Online Banking Platform
  
  Tier 1 — Login & Authentication
  ┌────────────────────────────────────────────────────┐
  │  Measured availability: 99.97%                     │
  │  User expectation: "Login should always work"      │
  │  Business impact: $50K per minute of downtime      │
  │  Dependencies: IAM service (99.99%)                │
  │                                                    │
  │  → SLO: 99.95% availability                        │
  │  → Error budget: 21.6 min/month                    │
  │  → Latency SLO: 99% < 1 second                    │
  └────────────────────────────────────────────────────┘
  
  Tier 2 — Transaction Processing
  ┌────────────────────────────────────────────────────┐
  │  Measured availability: 99.99%                     │
  │  User expectation: "Transfers must not fail"       │
  │  Business impact: Regulatory + financial risk      │
  │  Dependencies: Core banking API (99.95%)           │
  │                                                    │
  │  → SLO: 99.95% availability                        │
  │  → CORRECTNESS SLO: 100% (no tolerance)            │
  │  → Error budget: 21.6 min/month                    │
  └────────────────────────────────────────────────────┘
  
  Tier 3 — Statement Generation
  ┌────────────────────────────────────────────────────┐
  │  Measured availability: 99.8%                      │
  │  User expectation: "Statements ready within 24h"   │
  │  Business impact: Low (delayed, not lost data)     │
  │  Dependencies: Data warehouse (99.9%)              │
  │                                                    │
  │  → SLO: 99.5% availability                         │
  │  → Freshness SLO: 95% generated within 4 hours    │
  │  → Error budget: 3.6 hours/month                   │
  └────────────────────────────────────────────────────┘
```

---

## 9. Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "Stakeholders demand five nines" | Calculate cost: show that 99.999% costs ~10x more than 99.99%. Show dependency ceiling. |
| "SLO seems arbitrary" | Base it on measured performance + user expectations. Show data. |
| "Different services need different SLOs" | That's correct! Tier services by business criticality. |
| "SLO never violated, team is complacent" | SLO may be too loose. Tighten or add latency SLOs. |
| "SLO always violated, team is demoralized" | Loosen SLO to achievable level, then improve incrementally. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Measurement First** | Measure 2-4 weeks before setting any target |
| **Conservative Start** | Set SLO below measured performance |
| **Dependency Ceiling** | SLO ≤ product of dependency SLOs |
| **Business Alignment** | Target matches user expectations and business impact |
| **Error Budget Policy** | Graduated response to budget consumption |
| **Quarterly Iteration** | Review and adjust SLOs based on data |
| **Cost of Nines** | Each additional nine costs ~10x more |
| **Never Over-Tighten** | Only tighten SLO if users/business demand it |

---

## Quick Revision Questions

1. **What is the recommended approach for setting an initial SLO?**
   > Measure actual performance for 2-4 weeks, then set the SLO slightly below the measured value to allow room for natural variation.

2. **How do serial dependencies affect your maximum achievable SLO?**
   > Serial dependencies multiply: if three dependencies are each 99.9%, the combined availability is 99.9%^3 ≈ 99.7%.

3. **What should happen during an SLO quarterly review?**
   > Review SLI performance vs targets, error budget consumption, user feedback, and decide whether to tighten, loosen, or maintain current SLOs.

4. **Why should you NOT tighten an SLO just because the team is exceeding it?**
   > Each additional nine costs ~10x more to maintain. Only tighten if users or business genuinely need higher reliability.

5. **What is a 'happiness cliff' and how does it help set SLOs?**
   > The point where user complaints spike dramatically as reliability drops. Set the SLO above this cliff to keep users satisfied.

6. **Describe the escalation tiers in an error budget policy.**
   > Green (>50% budget: normal), Yellow (20-50%: caution), Orange (5-20%: reliability-only changes), Red (<5%: feature freeze).

---

[← Previous: Choosing SLIs](04-choosing-slis.md) | [Back to README](../README.md) | [Next: Error Budgets →](06-error-budgets.md)
