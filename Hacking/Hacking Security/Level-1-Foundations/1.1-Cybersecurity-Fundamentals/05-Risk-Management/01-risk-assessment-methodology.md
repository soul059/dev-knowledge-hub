# Risk Assessment Methodology

## Unit 5 - Topic 1 | Risk Management

---

## Overview

**Risk assessment** is the systematic process of identifying, analyzing, and evaluating cybersecurity risks to an organization. It answers the questions: "What could go wrong?", "How likely is it?", and "How bad would it be?" Risk assessment is the foundation of all security decision-making.

---

## 1. Risk Management Process

```
┌──────────────────────────────────────────────────────────────────┐
│                  RISK MANAGEMENT LIFECYCLE                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐                                                │
│  │ 1. IDENTIFY  │ What assets? What threats? What vulnerabilities?│
│  └──────┬───────┘                                                │
│         ▼                                                        │
│  ┌──────────────┐                                                │
│  │ 2. ANALYZE   │ How likely? How severe? Qualitative/Quantitative│
│  └──────┬───────┘                                                │
│         ▼                                                        │
│  ┌──────────────┐                                                │
│  │ 3. EVALUATE  │ Prioritize risks against risk appetite         │
│  └──────┬───────┘                                                │
│         ▼                                                        │
│  ┌──────────────┐                                                │
│  │ 4. TREAT     │ Mitigate, transfer, accept, or avoid           │
│  └──────┬───────┘                                                │
│         ▼                                                        │
│  ┌──────────────┐                                                │
│  │ 5. MONITOR   │ Continuously review and update                 │
│  └──────────────┘                                                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Key Risk Terminology

| Term | Definition |
|------|-----------|
| **Asset** | Anything of value (data, systems, people, reputation) |
| **Threat** | Potential cause of harm (hackers, malware, natural disaster) |
| **Vulnerability** | Weakness that can be exploited |
| **Risk** | Probability of a threat exploiting a vulnerability × Impact |
| **Likelihood** | Probability that a risk event will occur |
| **Impact** | Severity of consequences if the risk materializes |
| **Risk Appetite** | Amount of risk an organization is willing to accept |
| **Risk Tolerance** | Acceptable deviation from the risk appetite |
| **Residual Risk** | Risk remaining after controls are applied |
| **Inherent Risk** | Risk before any controls are applied |

```
RISK = THREAT × VULNERABILITY × IMPACT

   ┌──────────┐     ┌──────────────┐     ┌──────────┐
   │  THREAT  │  ×  │VULNERABILITY │  ×  │  IMPACT  │  = RISK
   │(Attacker)│     │ (Weakness)   │     │(Damage)  │
   └──────────┘     └──────────────┘     └──────────┘
```

---

## 3. Qualitative vs Quantitative Risk Assessment

### Qualitative Assessment

Uses descriptive scales (High/Medium/Low) to rate risk:

| Likelihood | Description |
|-----------|-------------|
| **Very High** | Almost certain to occur (>90%) |
| **High** | Likely to occur (70-90%) |
| **Medium** | Possible (30-70%) |
| **Low** | Unlikely (10-30%) |
| **Very Low** | Rare (<10%) |

| Impact | Description |
|--------|-------------|
| **Critical** | Business-ending or regulatory shutdown |
| **High** | Major financial loss, significant data breach |
| **Medium** | Moderate disruption, limited data exposure |
| **Low** | Minor inconvenience, no data loss |
| **Negligible** | Minimal or no impact |

### Risk Matrix (Qualitative)

```
              IMPACT
              Low    Medium   High   Critical
           ┌────────┬────────┬────────┬────────┐
  Very High│ Medium │  High  │Critical│Critical│
           ├────────┼────────┼────────┼────────┤
  High     │  Low   │ Medium │  High  │Critical│
L          ├────────┼────────┼────────┼────────┤
I Medium   │  Low   │ Medium │  High  │  High  │
K          ├────────┼────────┼────────┼────────┤
E Low      │  Low   │  Low   │ Medium │ Medium │
L          ├────────┼────────┼────────┼────────┤
I Very Low │  Low   │  Low   │  Low   │ Medium │
H          └────────┴────────┴────────┴────────┘
O
O          🟢 Low = Accept/Monitor
D          🟡 Medium = Mitigate when resources allow
           🟠 High = Prioritize mitigation
           🔴 Critical = Immediate action required
```

### Quantitative Assessment

Uses financial values to calculate risk:

| Formula | Description |
|---------|-------------|
| **AV** (Asset Value) | Dollar value of the asset |
| **EF** (Exposure Factor) | % of asset lost if threat occurs (0-1) |
| **SLE** (Single Loss Expectancy) | AV × EF |
| **ARO** (Annualized Rate of Occurrence) | How often the threat occurs per year |
| **ALE** (Annualized Loss Expectancy) | SLE × ARO |

### Quantitative Example

```
Asset: Customer Database
AV  = $500,000 (value of the database)
EF  = 0.40 (40% of data compromised in a breach)
SLE = $500,000 × 0.40 = $200,000 (loss per incident)
ARO = 0.5 (expected once every 2 years)
ALE = $200,000 × 0.5 = $100,000/year (expected annual loss)

DECISION: If a security control costs less than $100,000/year
and reduces the risk, it's a good investment.
```

---

## 4. Risk Assessment Steps

| Step | Action | Output |
|------|--------|--------|
| 1. **Scope** | Define what's being assessed | Assessment boundary |
| 2. **Asset Inventory** | List all assets in scope | Asset register |
| 3. **Threat Identification** | List potential threats | Threat catalog |
| 4. **Vulnerability Identification** | Find weaknesses | Vulnerability list |
| 5. **Existing Controls** | Document current security | Control inventory |
| 6. **Likelihood Analysis** | Rate probability | Likelihood scores |
| 7. **Impact Analysis** | Rate severity | Impact scores |
| 8. **Risk Calculation** | Combine likelihood × impact | Risk scores |
| 9. **Risk Prioritization** | Rank risks | Prioritized risk register |
| 10. **Recommendations** | Propose treatments | Treatment plan |

---

## 5. Risk Register

| Risk ID | Threat | Vulnerability | Likelihood | Impact | Risk Level | Treatment |
|---------|--------|--------------|-----------|--------|------------|-----------|
| R-001 | Ransomware | Unpatched systems | High | Critical | 🔴 Critical | Patch + backup |
| R-002 | Phishing | No email filtering | High | High | 🟠 High | Email gateway + training |
| R-003 | Insider threat | No DLP | Medium | High | 🟠 High | DLP + UBA |
| R-004 | Power outage | No UPS | Low | Medium | 🟡 Medium | Install UPS + generator |
| R-005 | Physical theft | No cameras | Low | Low | 🟢 Low | Accept + insure |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Risk Formula** | Risk = Threat × Vulnerability × Impact |
| **Qualitative** | Uses descriptive scales (High/Medium/Low) |
| **Quantitative** | Uses financial calculations (ALE = SLE × ARO) |
| **Risk Register** | Document tracking all identified risks and treatments |
| **Continuous** | Risk assessment is ongoing, not one-time |
| **Purpose** | Inform security investment and prioritization decisions |

---

## Quick Revision Questions

1. **What is the risk formula and what does each component mean?**
2. **What is the difference between qualitative and quantitative risk assessment?**
3. **Calculate ALE given: AV=$1M, EF=0.3, ARO=2.**
4. **What are the five steps of the risk management lifecycle?**
5. **What is a risk register and what information does it contain?**
6. **What is the difference between inherent risk and residual risk?**

---

[Next: Threat Modeling →](02-threat-modeling.md)

---

*Unit 5 - Topic 1 of 6 | [Back to README](../README.md)*
