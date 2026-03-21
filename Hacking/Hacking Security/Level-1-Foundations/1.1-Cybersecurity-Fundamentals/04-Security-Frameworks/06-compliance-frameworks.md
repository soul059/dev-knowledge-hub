# Compliance Frameworks

## Unit 4 - Topic 6 | Security Frameworks

---

## Overview

**Compliance frameworks** are mandatory regulatory requirements that organizations must follow based on their industry, geography, or the type of data they handle. Unlike voluntary frameworks (NIST CSF, CIS), compliance frameworks carry legal penalties for non-compliance, including fines, lawsuits, and criminal charges.

---

## 1. Compliance vs Security Frameworks

```
┌──────────────────────────────────────────────────────────────────┐
│           COMPLIANCE vs SECURITY FRAMEWORKS                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SECURITY FRAMEWORKS (Voluntary)     COMPLIANCE (Mandatory)      │
│  ───────────────────────────         ────────────────────────    │
│  • NIST CSF                          • PCI-DSS (payment cards)   │
│  • CIS Controls                      • HIPAA (healthcare)        │
│  • ISO 27001 (voluntary cert)        • GDPR (EU data privacy)    │
│  • MITRE ATT&CK                      • SOX (financial reporting) │
│                                      • FERPA (education)         │
│  Best practices — no legal           • CCPA (California privacy) │
│  penalty for non-adoption            • GLBA (financial services) │
│                                                                  │
│  "Should do"                         "MUST do" (or face fines)   │
│                                                                  │
│  NOTE: Being compliant ≠ being secure. Compliance is the         │
│  minimum. Security should exceed compliance requirements.        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Major Compliance Frameworks

### PCI-DSS (Payment Card Industry Data Security Standard)

| Aspect | Details |
|--------|---------|
| **Applies To** | Any org that processes, stores, or transmits credit card data |
| **Managed By** | PCI Security Standards Council (Visa, Mastercard, etc.) |
| **Requirements** | 12 requirements in 6 categories |
| **Penalties** | Fines up to $100K/month, loss of card processing ability |
| **Latest Version** | PCI-DSS v4.0 |

**12 Requirements:**
1. Install and maintain network security controls
2. Apply secure configurations
3. Protect stored account data
4. Encrypt cardholder data in transit
5. Protect against malware
6. Develop secure systems and software
7. Restrict access by business need-to-know
8. Identify users and authenticate access
9. Restrict physical access to cardholder data
10. Log and monitor access
11. Test security regularly
12. Support information security with policies

### HIPAA (Health Insurance Portability and Accountability Act)

| Aspect | Details |
|--------|---------|
| **Applies To** | Healthcare providers, insurers, and their business associates (US) |
| **Protects** | Protected Health Information (PHI) |
| **Key Rules** | Privacy Rule, Security Rule, Breach Notification Rule |
| **Penalties** | $100 to $1.5M per violation category per year |

| HIPAA Rule | Focus |
|-----------|-------|
| **Privacy Rule** | Who can access PHI, patient rights |
| **Security Rule** | Technical, physical, administrative safeguards for ePHI |
| **Breach Notification** | Reporting requirements when PHI is breached |

### GDPR (General Data Protection Regulation)

| Aspect | Details |
|--------|---------|
| **Applies To** | Any org processing EU residents' personal data (worldwide!) |
| **Managed By** | European Union |
| **Key Rights** | Right to access, rectification, erasure ("right to be forgotten"), portability |
| **Penalties** | Up to €20M or 4% of global annual revenue (whichever is higher) |
| **DPO** | Data Protection Officer required for many organizations |

**Key GDPR Principles:**

```
┌──────────────────────────────────────────────────────────────┐
│                    GDPR PRINCIPLES                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Lawfulness, fairness, transparency                       │
│  2. Purpose limitation (collect for stated purpose only)     │
│  3. Data minimization (collect only what's needed)           │
│  4. Accuracy (keep data up to date)                          │
│  5. Storage limitation (don't keep longer than needed)       │
│  6. Integrity and confidentiality (secure the data)          │
│  7. Accountability (demonstrate compliance)                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### SOX (Sarbanes-Oxley Act)

| Aspect | Details |
|--------|---------|
| **Applies To** | Publicly traded companies in the US |
| **Focus** | Financial reporting accuracy and internal controls |
| **Key Section** | Section 404 — IT controls over financial reporting |
| **Penalties** | Criminal penalties for executives, up to 20 years in prison |

---

## 3. Compliance Comparison Table

| Framework | Industry | Geography | Data Protected | Max Penalty |
|-----------|----------|-----------|---------------|-------------|
| **PCI-DSS** | Payment cards | Global | Cardholder data | $100K/month + ban |
| **HIPAA** | Healthcare | US | PHI | $1.5M/year/category |
| **GDPR** | All industries | EU (+ any org processing EU data) | Personal data | €20M or 4% revenue |
| **SOX** | Public companies | US | Financial data | 20 years prison |
| **FERPA** | Education | US | Student records | Loss of federal funding |
| **GLBA** | Financial | US | Consumer financial info | $100K per violation |
| **CCPA/CPRA** | All industries | California (US) | Consumer personal data | $7,500 per violation |

---

## 4. Compliance Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│            COMPLIANCE BEST PRACTICES                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Identify applicable regulations for your organization    │
│  2. Map compliance requirements to security controls         │
│  3. Use security frameworks to EXCEED compliance minimums    │
│  4. Document everything (policies, procedures, evidence)     │
│  5. Conduct regular audits and assessments                   │
│  6. Train employees on compliance requirements               │
│  7. Implement continuous monitoring, not point-in-time       │
│  8. Engage legal counsel for regulatory interpretation       │
│                                                              │
│  REMEMBER: Compliance is the FLOOR, not the CEILING.         │
│  Being compliant doesn't mean you're secure.                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Framework | Focus | Key Requirement | Penalty |
|-----------|-------|----------------|---------|
| **PCI-DSS** | Card payment data | 12 requirements for cardholder data protection | Fines + ban from processing |
| **HIPAA** | Health information | Privacy, Security, and Breach Notification rules | Up to $1.5M/year |
| **GDPR** | EU personal data | Consent, minimization, right to erasure | Up to €20M or 4% revenue |
| **SOX** | Financial reporting | IT controls over financial data | Criminal penalties |

---

## Quick Revision Questions

1. **What is the difference between a compliance framework and a security framework?**
2. **Name 4 major compliance frameworks and what data each protects.**
3. **What are the 7 GDPR principles?**
4. **What are the consequences of PCI-DSS non-compliance?**
5. **Why does "compliant" not necessarily mean "secure"?**
6. **Which compliance framework applies to a US hospital that also serves EU patients?**

---

[← Previous: OWASP](05-owasp.md)

---

*Unit 4 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Risk Management →](../05-Risk-Management/01-risk-assessment-methodology.md)*
