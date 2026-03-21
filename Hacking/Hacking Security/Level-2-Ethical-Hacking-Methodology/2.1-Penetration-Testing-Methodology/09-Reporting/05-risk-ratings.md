# Risk Ratings

## Unit 9 - Topic 5 | Reporting

---

## Overview

**Risk ratings** assign a severity level to each finding, helping organizations prioritize remediation. The most widely used system is **CVSS** (Common Vulnerability Scoring System), but pen testers must also consider **business context** to provide meaningful risk assessments.

---

## 1. CVSS (Common Vulnerability Scoring System)

```
CVSS v3.1 SEVERITY RATINGS:

Score    │ Rating     │ Meaning
─────────┼────────────┼──────────────────────────────────
0.0      │ None       │ No vulnerability
0.1-3.9  │ Low        │ Minor issue, limited impact
4.0-6.9  │ Medium     │ Moderate impact, some exploitation
7.0-8.9  │ High       │ Significant impact, likely exploitable
9.0-10.0 │ Critical   │ Severe impact, easily exploitable

CVSS VECTOR COMPONENTS:

BASE SCORE:
├── Attack Vector (AV):    Network / Adjacent / Local / Physical
├── Attack Complexity (AC): Low / High
├── Privileges Required (PR): None / Low / High
├── User Interaction (UI):  None / Required
├── Scope (S):             Unchanged / Changed
├── Confidentiality (C):   None / Low / High
├── Integrity (I):         None / Low / High
└── Availability (A):      None / Low / High

EXAMPLE:
SQL Injection (no auth required, remote, full data access):
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H = 9.8 (Critical)
```

---

## 2. Risk = Likelihood × Impact

```
┌──────────────────────────────────────────────────────────────┐
│              RISK CALCULATION MATRIX                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│            │         IMPACT                                  │
│            │  Low    │ Medium  │ High   │ Critical            │
│  LIKELIHOOD├─────────┼─────────┼────────┼─────────            │
│  High      │ Medium  │ High    │ Critical│ Critical           │
│  Medium    │ Low     │ Medium  │ High   │ Critical            │
│  Low       │ Info    │ Low     │ Medium │ High                │
│  Very Low  │ Info    │ Info    │ Low    │ Medium              │
│                                                              │
└──────────────────────────────────────────────────────────────┘

LIKELIHOOD FACTORS:
├── Is there a public exploit? (higher if yes)
├── How complex is exploitation? (higher if easy)
├── Does it require authentication? (lower if yes)
├── Is the service internet-facing? (higher if yes)
└── Are there compensating controls? (lower if yes)

IMPACT FACTORS:
├── What data is at risk? (PII, financial, etc.)
├── What systems could be compromised?
├── Regulatory implications (GDPR, HIPAA)?
├── Business operations disruption?
└── Reputational damage potential?
```

---

## 3. Contextual Risk Assessment

```
TECHNICAL CVSS vs CONTEXTUAL RISK:

EXAMPLE 1: SQL Injection on Internal Dev Server
• CVSS: 9.8 (Critical)
• Context: Internal only, no real data, dev environment
• Contextual Risk: MEDIUM
• Reasoning: No business impact, but shows code weakness

EXAMPLE 2: Default Credentials on Internet-Facing Admin Panel
• CVSS: 7.5 (High)
• Context: Manages production infrastructure, internet-facing
• Contextual Risk: CRITICAL
• Reasoning: Easy exploitation, high business impact

EXAMPLE 3: Missing HTTP Security Header
• CVSS: 3.0 (Low)
• Context: Banking application with sensitive data
• Contextual Risk: MEDIUM
• Reasoning: Could enable clickjacking on financial transactions

LESSON: CVSS alone doesn't tell the full story.
Always add business context to your risk ratings.
```

---

## 4. Alternative Rating Systems

| System | Used By | Description |
|--------|---------|-------------|
| **CVSS** | Industry standard | Base + Temporal + Environmental |
| **OWASP Risk Rating** | Web applications | Likelihood × Impact matrix |
| **DREAD** | Microsoft (legacy) | Damage, Reproducibility, Exploitability, Affected Users, Discoverability |
| **Custom** | Consulting firms | Internal 1-5 or color-coded scale |

---

## 5. Presenting Risk Ratings

```
FINDINGS BY RISK RATING:

ID    │ Finding                          │ CVSS │ Risk
──────┼──────────────────────────────────┼──────┼─────────
F-001 │ SQL Injection — Login Page       │ 9.8  │ CRITICAL
F-002 │ EternalBlue — File Server        │ 9.8  │ CRITICAL
F-003 │ Default Tomcat Credentials       │ 8.6  │ HIGH
F-004 │ Weak SSH Configuration           │ 5.3  │ MEDIUM
F-005 │ Missing HSTS Header              │ 4.3  │ MEDIUM
F-006 │ Server Version Disclosure        │ 3.0  │ LOW

REMEDIATION PRIORITY:
1st — Critical findings (F-001, F-002): Fix within 24-48 hours
2nd — High findings (F-003): Fix within 1 week
3rd — Medium findings (F-004, F-005): Fix within 30 days
4th — Low findings (F-006): Fix within 90 days
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **CVSS** | Industry standard, 0-10 scale |
| **Critical** | 9.0-10.0, immediate action needed |
| **Risk formula** | Likelihood × Impact |
| **Context matters** | CVSS alone is insufficient |
| **Business impact** | Translate to dollars, records, compliance |
| **Prioritization** | Critical→24h, High→1wk, Medium→30d |

---

## Quick Revision Questions

1. **What is CVSS and what are its severity levels?**
2. **How is risk calculated?**
3. **Why might contextual risk differ from CVSS score?**
4. **What factors determine likelihood?**
5. **How should remediation be prioritized by risk level?**

---

[← Previous: Technical Findings](04-technical-findings.md) | [Next: Remediation Recommendations →](06-remediation-recommendations.md)

---

*Unit 9 - Topic 5 of 7 | [Back to README](../README.md)*
