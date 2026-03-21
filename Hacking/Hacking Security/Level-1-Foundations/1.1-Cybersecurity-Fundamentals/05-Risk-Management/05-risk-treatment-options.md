# Risk Treatment Options

## Unit 5 - Topic 5 | Risk Management

---

## Overview

Once risks are identified and analyzed, organizations must decide how to handle them. There are four primary **risk treatment** (or risk response) options: **Mitigate, Transfer, Accept, or Avoid**. The choice depends on the risk level, cost of controls, and business context.

---

## 1. The Four Risk Treatment Options

```
┌──────────────────────────────────────────────────────────────────┐
│                  RISK TREATMENT OPTIONS                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │  1. MITIGATE      │  │  2. TRANSFER     │                     │
│  │  (Reduce)         │  │  (Share)         │                     │
│  │                   │  │                  │                     │
│  │  Apply controls   │  │  Shift risk to   │                     │
│  │  to reduce        │  │  third party     │                     │
│  │  likelihood or    │  │  (insurance,     │                     │
│  │  impact           │  │  outsourcing)    │                     │
│  └──────────────────┘  └──────────────────┘                     │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │  3. ACCEPT        │  │  4. AVOID        │                     │
│  │  (Acknowledge)    │  │  (Eliminate)     │                     │
│  │                   │  │                  │                     │
│  │  Acknowledge      │  │  Stop the        │                     │
│  │  risk and take    │  │  activity that   │                     │
│  │  no action        │  │  creates the     │                     │
│  │  (within appetite)│  │  risk entirely   │                     │
│  └──────────────────┘  └──────────────────┘                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Detailed Descriptions

### Mitigate (Reduce)

| Aspect | Details |
|--------|---------|
| **What** | Apply controls to reduce likelihood and/or impact |
| **When** | Risk is above appetite AND cost-effective controls exist |
| **Examples** | Install firewalls, patch systems, implement MFA, train employees |
| **Result** | Reduces risk to acceptable level (residual risk) |
| **Most Common** | This is the most frequently used risk treatment |

### Transfer (Share)

| Aspect | Details |
|--------|---------|
| **What** | Shift risk consequence to a third party |
| **When** | Risk is too expensive to mitigate internally |
| **Examples** | Cyber insurance, outsourcing to MSP/MSSP, SLAs with vendors |
| **Result** | Financial impact shared; operational risk may remain |
| **Note** | You can transfer financial risk but not reputational risk |

### Accept (Acknowledge)

| Aspect | Details |
|--------|---------|
| **What** | Formally acknowledge the risk and take no action |
| **When** | Risk is within risk appetite OR cost to mitigate exceeds potential loss |
| **Examples** | Low-impact risks, risks where controls are too expensive |
| **Requirement** | Must be formally documented and approved by management |
| **Note** | Acceptance ≠ ignorance. It's a conscious, documented decision |

### Avoid (Eliminate)

| Aspect | Details |
|--------|---------|
| **What** | Eliminate the activity or technology that creates the risk |
| **When** | Risk is too high and cannot be adequately mitigated |
| **Examples** | Stop using legacy software, discontinue a risky business process |
| **Result** | Risk eliminated entirely (along with potential benefits) |
| **Tradeoff** | May lose business opportunities |

---

## 3. Decision Matrix

```
                            IMPACT
                    Low              High
               ┌──────────────┬──────────────┐
    High       │   MITIGATE   │    AVOID     │
LIKELIHOOD     │   or         │    or        │
               │   TRANSFER   │    MITIGATE  │
               ├──────────────┼──────────────┤
    Low        │   ACCEPT     │   TRANSFER   │
               │              │   or         │
               │              │   MITIGATE   │
               └──────────────┴──────────────┘
```

---

## 4. Real-World Examples

| Risk | Treatment | Reasoning |
|------|-----------|-----------|
| Ransomware attack on servers | **Mitigate** | Implement backups, EDR, patching, training |
| Earthquake destroys data center | **Transfer** | Buy insurance + use cloud DR |
| Minor website defacement risk | **Accept** | Low impact, monitoring in place |
| Using unsupported Windows XP | **Avoid** | Migrate to supported OS entirely |
| Employee laptop theft | **Mitigate + Transfer** | Encryption + insurance |
| Low-priority dev server vulnerability | **Accept** | Isolated, non-production, monitored |

---

## 5. Residual Risk

```
┌──────────────────────────────────────────────────────────────┐
│                    RESIDUAL RISK                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  INHERENT RISK ──[Apply Controls]──► RESIDUAL RISK           │
│  (Before controls)                   (After controls)        │
│                                                              │
│  Example:                                                    │
│  Inherent Risk (ransomware):    CRITICAL (20)                │
│  Controls Applied:              Backups, EDR, MFA, training  │
│  Residual Risk:                 MEDIUM (6)                   │
│                                                              │
│  If Residual Risk > Risk Appetite → Apply MORE controls      │
│  If Residual Risk ≤ Risk Appetite → ACCEPT residual risk     │
│                                                              │
│  ⚠️ Residual risk can NEVER be zero.                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Treatment | Action | When to Use | Example |
|-----------|--------|-------------|---------|
| **Mitigate** | Reduce risk with controls | Cost-effective controls available | Firewall, MFA, patching |
| **Transfer** | Shift to third party | Too expensive to handle internally | Cyber insurance |
| **Accept** | Acknowledge and monitor | Risk within appetite | Low-impact vulnerabilities |
| **Avoid** | Eliminate the risk source | Risk too high, can't be mitigated | Discontinue legacy system |

---

## Quick Revision Questions

1. **What are the four risk treatment options?**
2. **When would you choose to ACCEPT a risk?**
3. **Give an example where risk TRANSFER is the best option.**
4. **What is residual risk and can it ever be zero?**
5. **Why must risk acceptance be formally documented?**
6. **A company uses Windows XP for a critical system. Which treatment option would you recommend and why?**

---

[← Previous: Risk Calculation](04-risk-calculation.md) | [Next: Business Impact Analysis →](06-business-impact-analysis.md)

---

*Unit 5 - Topic 5 of 6 | [Back to README](../README.md)*
