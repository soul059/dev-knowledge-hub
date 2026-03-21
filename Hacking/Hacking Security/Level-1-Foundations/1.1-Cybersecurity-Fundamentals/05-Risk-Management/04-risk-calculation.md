# Risk Calculation

## Unit 5 - Topic 4 | Risk Management

---

## Overview

**Risk calculation** translates identified threats, vulnerabilities, and impacts into measurable values — enabling organizations to prioritize risks and make data-driven security investment decisions.

---

## 1. Fundamental Risk Formula

```
┌──────────────────────────────────────────────────────────────┐
│                   RISK FORMULAS                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Basic:     RISK = LIKELIHOOD × IMPACT                       │
│                                                              │
│  Extended:  RISK = THREAT × VULNERABILITY × ASSET VALUE      │
│                                                              │
│  Financial: ALE = SLE × ARO                                  │
│             SLE = AV × EF                                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Quantitative Calculations

| Term | Formula | Description |
|------|---------|-------------|
| **AV** | — | Asset Value ($) |
| **EF** | — | Exposure Factor (% loss, 0-1) |
| **SLE** | AV × EF | Single Loss Expectancy |
| **ARO** | — | Annualized Rate of Occurrence |
| **ALE** | SLE × ARO | Annualized Loss Expectancy |
| **Cost of Control** | — | Annual cost to implement a safeguard |
| **Value of Control** | ALE(before) - ALE(after) - Cost | Whether the control is worth it |

### Worked Example

```
SCENARIO: E-commerce database breach risk

Asset Value (AV):         $2,000,000
Exposure Factor (EF):     30% (0.30)
Single Loss Expectancy:   $2,000,000 × 0.30 = $600,000
Annualized Rate (ARO):    0.5 (once every 2 years)
Annual Loss Expectancy:   $600,000 × 0.5 = $300,000

PROPOSED CONTROL: WAF + Database Encryption
Cost: $80,000/year
New ALE after control: $50,000/year

Value of Control = $300,000 - $50,000 - $80,000 = $170,000 SAVED
✅ Control is worth implementing (positive ROI)
```

### When Control is NOT Worth It

```
SCENARIO: Low-value asset, expensive control

ALE before control:    $10,000/year
Control cost:          $50,000/year
ALE after control:     $2,000/year

Value = $10,000 - $2,000 - $50,000 = -$42,000 LOSS
❌ Control costs more than the risk — NOT worth it
→ Consider cheaper alternatives or accept the risk
```

---

## 3. Qualitative Risk Matrix

```
                          IMPACT
                 Low    Medium    High    Critical
              ┌────────┬────────┬────────┬────────┐
    High      │   5    │   10   │   15   │   20   │
              ├────────┼────────┼────────┼────────┤
L   Medium    │   3    │   6    │   12   │   16   │
I             ├────────┼────────┼────────┼────────┤
K   Low       │   1    │   4    │   8    │   12   │
E             ├────────┼────────┼────────┼────────┤
L   Very Low  │   1    │   2    │   4    │   8    │
I             └────────┴────────┴────────┴────────┘
H
O   Score Ranges:
O   1-4:   🟢 LOW RISK    → Accept / Monitor
D   5-8:   🟡 MEDIUM RISK → Mitigate when possible
    9-15:  🟠 HIGH RISK   → Prioritize mitigation
    16-20: 🔴 CRITICAL    → Immediate action
```

---

## 4. Risk Appetite and Tolerance

```
┌──────────────────────────────────────────────────────────────┐
│              RISK APPETITE vs TOLERANCE                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  RISK APPETITE:                                              │
│  "How much risk are we WILLING to take?"                     │
│  Set by the board / executive leadership                     │
│  Example: "We accept up to $500K annual cyber risk"          │
│                                                              │
│  RISK TOLERANCE:                                             │
│  "How much deviation from appetite is ACCEPTABLE?"           │
│  Operational flexibility around the appetite                 │
│  Example: "Tolerance of ±$100K around the $500K appetite"    │
│                                                              │
│  RISK THRESHOLD:                                             │
│  "At what point must we ESCALATE?"                           │
│  Trigger for management action                               │
│  Example: "Any single risk >$200K requires board approval"   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Return on Security Investment (ROSI)

```
ROSI = (ALE_before - ALE_after - Cost_of_Control) / Cost_of_Control × 100%

Example:
ALE before:   $300,000
ALE after:    $50,000
Control cost: $80,000

ROSI = ($300,000 - $50,000 - $80,000) / $80,000 × 100% = 212.5%

For every $1 spent on the control, we save $2.13 in risk reduction.
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Basic Formula** | Risk = Likelihood × Impact |
| **Quantitative** | ALE = SLE × ARO (financial values) |
| **Qualitative** | Risk matrix with descriptive scales |
| **Value of Control** | ALE(before) - ALE(after) - Cost |
| **ROSI** | ROI calculation for security investments |
| **Risk Appetite** | Org's acceptable level of risk |

---

## Quick Revision Questions

1. **Calculate ALE: AV=$1M, EF=0.25, ARO=3. Is a $200K/year control worth it if it reduces ALE to $50K?**
2. **What is the difference between risk appetite and risk tolerance?**
3. **When is a security control NOT worth implementing?**
4. **How do you calculate Return on Security Investment (ROSI)?**
5. **What is the difference between inherent risk and residual risk?**

---

[← Previous: Vulnerability Assessment](03-vulnerability-assessment.md) | [Next: Risk Treatment Options →](05-risk-treatment-options.md)

---

*Unit 5 - Topic 4 of 6 | [Back to README](../README.md)*
