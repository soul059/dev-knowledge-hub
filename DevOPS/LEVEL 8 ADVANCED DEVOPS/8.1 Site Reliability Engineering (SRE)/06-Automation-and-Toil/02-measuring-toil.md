# Chapter 6.2: Measuring Toil

[← Previous: What is Toil](01-what-is-toil.md) | [Back to README](../README.md) | [Next: Automation Strategies →](03-automation-strategies.md)

---

## Overview

You cannot reduce what you do not measure. Measuring toil provides data-driven evidence for prioritizing automation investments, tracking progress, and justifying engineering time to leadership. Effective toil measurement combines time tracking, categorization, and cost analysis.

---

## 1. Toil Measurement Framework

```
  ┌──────────────────────────────────────────────────────────┐
  │               TOIL MEASUREMENT LIFECYCLE                 │
  │                                                          │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐            │
  │  │ IDENTIFY │──▶│ QUANTIFY │──▶│PRIORITIZE│            │
  │  │          │   │          │   │          │             │
  │  │ • Survey │   │ • Hours  │   │ • Effort │            │
  │  │ • Track  │   │ • Cost   │   │ • Impact │            │
  │  │ • Observe│   │ • Freq   │   │ • ROI    │            │
  │  └──────────┘   └──────────┘   └──────────┘            │
  │       │                             │                    │
  │       │         ┌──────────┐        │                    │
  │       │         │ AUTOMATE │◀───────┘                    │
  │       │         │          │                             │
  │       │         │ • Build  │                             │
  │       │         │ • Deploy │                             │
  │       │         │ • Verify │                             │
  │       │         └────┬─────┘                             │
  │       │              │                                   │
  │       │         ┌────▼─────┐                             │
  │       └────────▶│ MEASURE  │  (repeat — continuous       │
  │                 │  AGAIN   │   improvement loop)         │
  │                 └──────────┘                             │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Toil Survey Template

```yaml
# Weekly toil survey — each team member fills out
toil_survey:
  engineer: "jane.doe"
  week: "2024-W48"
  
  tasks:
    - name: "Manual deployment of payment-service"
      frequency: "3 times this week"
      duration_per_occurrence: "45 min"
      total_time: "2h 15m"
      category: "deployment"
      is_toil: true
      could_automate: true
      automation_difficulty: "medium"  # low / medium / high
      frustration_level: 4            # 1-5 scale
      
    - name: "Investigating false-positive alerts"
      frequency: "~10 times this week"
      duration_per_occurrence: "15 min"
      total_time: "2h 30m"
      category: "monitoring"
      is_toil: true
      could_automate: true
      automation_difficulty: "low"
      frustration_level: 5
      
    - name: "Designing new caching layer"
      frequency: "ongoing"
      total_time: "12h"
      category: "engineering"
      is_toil: false
      notes: "Strategic project work"

  summary:
    total_hours_worked: 40
    toil_hours: 18.5
    engineering_hours: 16
    overhead_hours: 5.5
    toil_percentage: 46.25%   # Under 50% but close
```

---

## 3. Toil Tracking Metrics

```
  ┌──────────────────────────────────────────────────────────┐
  │  METRIC                 │ FORMULA                │ TARGET│
  ├─────────────────────────┼────────────────────────┼───────┤
  │ Toil Percentage         │ toil_hours / total_hrs │ ≤ 50% │
  │                         │  × 100                 │       │
  │                         │                        │       │
  │ Toil per Engineer       │ total_toil_hrs /       │ ≤ 20h │
  │ (weekly)                │  num_engineers         │ /week │
  │                         │                        │       │
  │ Toil Growth Rate        │ (current - previous) / │ ≤ 0%  │
  │                         │  previous × 100        │ (flat │
  │                         │                        │ or ↓) │
  │                         │                        │       │
  │ Automation Rate         │ automated_tasks /      │ ≥ 80% │
  │                         │  total_repetitive_tasks│       │
  │                         │                        │       │
  │ Ticket Volume           │ manual_tickets /       │ ↓ over│
  │ per Service             │  num_services          │  time │
  │                         │                        │       │
  │ Time to Automate (TTA)  │ Days from identifying  │ ≤ 30d │
  │                         │  toil → deploying fix  │       │
  └─────────────────────────┴────────────────────────┴───────┘
```

---

## 4. Toil Cost Calculation

```
  ┌──────────────────────────────────────────────────────────┐
  │  ANNUAL COST OF TOIL                                     │
  │                                                          │
  │  Formula:                                                │
  │  annual_toil_cost = toil_hours_per_week                  │
  │                   × 52 weeks                             │
  │                   × fully_loaded_engineer_cost_per_hour  │
  │                                                          │
  │  EXAMPLE:                                                │
  │  ─────────────────────────────────────────────           │
  │  Team: 6 SREs                                            │
  │  Avg toil: 18 hrs/week per engineer                      │
  │  Fully loaded cost: $100/hr (salary + benefits + infra)  │
  │                                                          │
  │  Annual toil cost = 6 × 18 × 52 × $100                  │
  │                   = $561,600/year spent on toil!          │
  │                                                          │
  │  If automation reduces toil by 60%:                      │
  │  Savings = $561,600 × 0.60 = $336,960/year              │
  │  One-time automation cost: ~$80,000                      │
  │  ROI: 4.2x in first year                                │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Toil Categorization Matrix

```
  ┌──────────────────────────────────────────────────────────┐
  │  Used to prioritize what to automate first               │
  │                                                          │
  │  Frequency ▲                                             │
  │  (per week) │                                            │
  │             │  ┌─────────────┐  ┌──────────────┐         │
  │    HIGH     │  │  AUTOMATE   │  │  AUTOMATE    │         │
  │   (>10)     │  │  SECOND     │  │  FIRST ★★★   │         │
  │             │  │  (frequent  │  │  (frequent   │         │
  │             │  │  but quick) │  │  AND slow)   │         │
  │             │  └─────────────┘  └──────────────┘         │
  │             │                                            │
  │             │  ┌─────────────┐  ┌──────────────┐         │
  │    LOW      │  │  ACCEPT     │  │  AUTOMATE    │         │
  │   (<10)     │  │  (rare and  │  │  THIRD       │         │
  │             │  │  quick)     │  │  (rare but   │         │
  │             │  │             │  │  very slow)  │         │
  │             │  └─────────────┘  └──────────────┘         │
  │             │                                            │
  │             └──────────────────────────────────▶         │
  │                LOW (<30min)     HIGH (>30min)             │
  │                    Duration per occurrence                │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Toil Tracking Tools and Methods

```
  ┌──────────────────────────────────────────────────────────┐
  │  METHOD           │ PROS              │ CONS              │
  ├───────────────────┼───────────────────┼───────────────────┤
  │ Weekly surveys    │ Simple, captures  │ Relies on memory, │
  │ (Google Forms/    │ subjective data   │ under-reporting   │
  │  Typeform)        │ (frustration too) │ common            │
  │                   │                   │                   │
  │ Time tracking     │ Accurate data,    │ Overhead itself   │
  │ (Toggl/Clockify/  │ trends visible    │ becomes toil      │
  │  Jira work logs)  │                   │                   │
  │                   │                   │                   │
  │ Ticket analysis   │ Objective data,   │ Only captures     │
  │ (Jira/ServiceNow  │ already tracked   │ ticketed work     │
  │  labels)          │                   │                   │
  │                   │                   │                   │
  │ Alert analysis    │ Direct correlation│ Doesn't capture   │
  │ (PagerDuty/       │ to on-call burden │ non-alert toil    │
  │  OpsGenie reports)│                   │                   │
  │                   │                   │                   │
  │ Git/CI metrics    │ Shows deploy      │ Narrow scope      │
  │ (deployment freq, │ automation level  │ (deploy only)     │
  │  manual steps)    │                   │                   │
  └───────────────────┴───────────────────┴───────────────────┘
  
  BEST APPROACH: Combine ticket analysis + quarterly survey
  (low overhead but comprehensive)
```

---

## 7. Toil Budget

```yaml
# Toil budget — treat toil like error budget
toil_budget:
  team: "platform-sre"
  quarter: "Q1-2025"
  
  budget:
    total_team_hours_per_week: 240    # 6 engineers × 40h
    max_toil_percentage: 40%           # Stricter than 50%
    max_toil_hours_per_week: 96
    
  current:
    actual_toil_hours: 78
    actual_toil_percentage: 32.5%
    status: "WITHIN BUDGET"
    
  categories:
    deployments:
      budget: 15%
      actual: 8%          # ✅ CI/CD automation working
    monitoring:
      budget: 10%
      actual: 12%         # ⚠️ Over budget — alert tuning needed
    access_management:
      budget: 5%
      actual: 3%          # ✅ SSO working well
    scaling:
      budget: 5%
      actual: 4.5%        # ✅ Near budget — HPA helping
    incident_remediation:
      budget: 5%
      actual: 5%          # ⚠️ At limit — need more self-healing
      
  actions_if_over_budget:
    - "Pause feature work to build automation"
    - "Escalate to management for resources"
    - "Consider handing service back to dev team"  # Google practice
```

---

## 8. Real-World Scenario

### [SCENARIO] Measuring Toil to Justify Automation Investment

```
  SITUATION: Infrastructure team claims they're "too busy."
  Manager asks: "Busy doing what? Show me the data."
  
  Step 1: Ran 4-week toil survey across 8 engineers
  ┌──────────────────────────────────────────────────────────┐
  │  TOIL CATEGORY          │ HOURS/WEEK │  % OF TOTAL      │
  ├─────────────────────────┼────────────┼──────────────────┤
  │ Manual VM provisioning  │    32      │   10.0%          │
  │ SSL certificate renewal │    12      │    3.7%          │
  │ Access request tickets  │    24      │    7.5%          │
  │ Log rotation/cleanup    │     8      │    2.5%          │
  │ Manual deploys          │    40      │   12.5%          │
  │ Alert investigation     │    28      │    8.7%          │
  │ Capacity tickets        │    16      │    5.0%          │
  │ OTHER TOIL              │    12      │    3.7%          │
  ├─────────────────────────┼────────────┼──────────────────┤
  │ TOTAL TOIL              │   172      │   53.7% ← TOO   │
  │ Engineering work        │   108      │   33.8%   HIGH   │
  │ Overhead                │    40      │   12.5%          │
  └─────────────────────────┴────────────┴──────────────────┘
  
  Step 2: Cost analysis presented to leadership
  ├─ Annual toil cost: 172 hrs × 52 weeks × $95/hr = $849K
  ├─ Top 3 items (deploys + VM + access) = 60% of toil
  ├─ Automation cost estimate: $120K (one-time)
  ├─ Expected reduction: 65% of toil
  ├─ Annual savings: $552K
  └─ APPROVED: 2-quarter automation project funded
  
  Result after 6 months:
  ├─ Toil reduced from 53.7% → 22%
  ├─ Team gained 100+ engineering hours per week  
  ├─ Used freed time to build self-healing systems
  └─ Incident rate dropped 35%
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Engineers don't fill out toil surveys" | Make it 5 minutes max. Use auto-populated categories. Run it monthly instead of weekly. |
| "Toil numbers seem too low" | Engineers often undercount. Supplement with ticket/alert data. Shadow engineers for a day. |
| "Can't distinguish toil from engineering" | Apply all six toil criteria. If the task has enduring value or is being done for the first time, it's not toil. |
| "Toil keeps growing despite automation" | New services bring new toil. Require "toil impact assessment" for new service launches. |

---

## Summary Table

| Metric | What It Measures | Target |
|--------|-----------------|--------|
| **Toil Percentage** | % of time on toil | ≤ 50% (ideally ≤ 30%) |
| **Toil per Engineer** | Hours of toil per person per week | ≤ 20 hours |
| **Toil Growth Rate** | How fast toil is increasing | ≤ 0% (flat or decreasing) |
| **Automation Rate** | % of repetitive tasks automated | ≥ 80% |
| **Toil Cost** | Annual dollar cost of toil | Track and reduce |
| **Time to Automate** | Days from identification to automation | ≤ 30 days |

---

## Quick Revision Questions

1. **Why is measuring toil important?**
   > You can't reduce what you don't measure. Data helps prioritize automation, track improvement, and justify investment to leadership.

2. **How do you calculate the annual cost of toil?**
   > toil_hours_per_week × number_of_engineers × 52 weeks × fully_loaded_hourly_cost. This typically reveals six-figure costs that make automation easy to justify.

3. **What is the toil categorization matrix?**
   > A 2×2 matrix of frequency vs. duration. High-frequency + long-duration tasks are automated first (highest ROI). Low-frequency + short-duration tasks may be acceptable.

4. **Name three methods for tracking toil.**
   > Weekly surveys (captures subjective data), ticket analysis (objective, already tracked), and alert/on-call reports (direct correlation to on-call burden). Best approach combines ticket data with periodic surveys.

5. **What is a toil budget?**
   > Similar to an error budget — a predefined maximum percentage of time that can be spent on toil per category. If exceeded, the team pauses feature work to build automation.

6. **What should you do if toil exceeds 50%?**
   > Immediate action: identify top 3 toil sources, build business case with cost data, get approval for automation sprint. Long-term: implement toil budgets and new-service toil assessments.

---

[← Previous: What is Toil](01-what-is-toil.md) | [Back to README](../README.md) | [Next: Automation Strategies →](03-automation-strategies.md)
