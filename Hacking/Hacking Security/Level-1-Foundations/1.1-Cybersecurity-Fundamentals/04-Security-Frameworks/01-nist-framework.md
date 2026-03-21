# NIST Cybersecurity Framework

## Unit 4 - Topic 1 | Security Frameworks

---

## Overview

The **NIST Cybersecurity Framework (CSF)** is a widely adopted, voluntary framework developed by the National Institute of Standards and Technology (NIST) to help organizations manage and reduce cybersecurity risk. It provides a common language, set of standards, and best practices applicable to organizations of all sizes and sectors.

---

## 1. Framework Structure

```
┌──────────────────────────────────────────────────────────────────────┐
│                   NIST CSF CORE FUNCTIONS                            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────┐ │
│  │ IDENTIFY │─►│ PROTECT  │─►│ DETECT   │─►│ RESPOND  │─►│RECOVER│ │
│  │   (ID)   │  │   (PR)   │  │   (DE)   │  │   (RS)   │  │  (RC) │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └───────┘ │
│                                                                      │
│  "What do    "How do we   "How do we    "What do     "How do       │
│   we have?"   safeguard?"  find attacks?" we do?"     we restore?" │
│                                                                      │
│  NIST CSF 2.0 adds a 6th function:                                  │
│  ┌──────────┐                                                       │
│  │  GOVERN  │  Organizational context, risk strategy, oversight     │
│  │   (GV)   │                                                       │
│  └──────────┘                                                       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 2. The Five Core Functions Detailed

### IDENTIFY (ID)

| Category | Description |
|----------|-------------|
| **Asset Management** | Identify and manage all hardware, software, data |
| **Business Environment** | Understand org's mission, objectives, stakeholders |
| **Governance** | Policies, procedures, legal/regulatory requirements |
| **Risk Assessment** | Identify and evaluate cybersecurity risks |
| **Risk Management Strategy** | Define risk tolerance and prioritization |
| **Supply Chain Risk** | Identify and manage third-party risks |

### PROTECT (PR)

| Category | Description |
|----------|-------------|
| **Identity Management & Access Control** | MFA, RBAC, least privilege |
| **Awareness & Training** | Security awareness programs |
| **Data Security** | Encryption, DLP, backup |
| **Information Protection Processes** | Configuration management, change control |
| **Maintenance** | Patch management, system maintenance |
| **Protective Technology** | Firewalls, IDS/IPS, endpoint protection |

### DETECT (DE)

| Category | Description |
|----------|-------------|
| **Anomalies & Events** | Baseline establishment, anomaly detection |
| **Security Continuous Monitoring** | SIEM, log monitoring, network monitoring |
| **Detection Processes** | Roles, responsibilities, testing detection capabilities |

### RESPOND (RS)

| Category | Description |
|----------|-------------|
| **Response Planning** | Incident response plan execution |
| **Communications** | Internal and external stakeholder communication |
| **Analysis** | Investigation, forensics, impact assessment |
| **Mitigation** | Containment, eradication actions |
| **Improvements** | Lessons learned, process updates |

### RECOVER (RC)

| Category | Description |
|----------|-------------|
| **Recovery Planning** | Execute recovery procedures |
| **Improvements** | Update recovery plans based on lessons learned |
| **Communications** | Coordinate restoration with stakeholders |

---

## 3. Framework Tiers

The tiers describe an organization's cybersecurity risk management maturity:

| Tier | Name | Description |
|------|------|-------------|
| **Tier 1** | Partial | Ad hoc, reactive; limited awareness of cyber risk |
| **Tier 2** | Risk Informed | Some risk awareness but not org-wide; informal processes |
| **Tier 3** | Repeatable | Formal, org-wide policies; regularly updated practices |
| **Tier 4** | Adaptive | Agile, risk-informed; adapts to changing threat landscape |

```
Maturity ──────────────────────────────────────────────►

Tier 1          Tier 2          Tier 3          Tier 4
PARTIAL     RISK INFORMED    REPEATABLE      ADAPTIVE
┌─────┐     ┌──────────┐    ┌──────────┐    ┌──────────┐
│Ad hoc│    │  Some     │    │  Formal  │    │  Agile & │
│React │───►│  awareness│───►│  policies│───►│  adaptive│
│ive   │    │  informal │    │  org-wide│    │  proactive│
└─────┘     └──────────┘    └──────────┘    └──────────┘
```

---

## 4. Framework Profiles

| Profile Type | Purpose |
|-------------|---------|
| **Current Profile** | "Where are we now?" — Current security posture |
| **Target Profile** | "Where do we want to be?" — Desired security state |
| **Gap Analysis** | Difference between current and target = action plan |

```
Current Profile ─────── GAP ─────── Target Profile
(Where we are)         (What to      (Where we want
                        improve)       to be)
```

---

## 5. How to Use NIST CSF

```
Step 1: Prioritize and Scope
   └── Define business objectives and priorities

Step 2: Orient
   └── Identify systems, assets, regulatory requirements

Step 3: Create Current Profile
   └── Assess current state against framework categories

Step 4: Conduct Risk Assessment
   └── Identify threats and likelihood

Step 5: Create Target Profile
   └── Define desired cybersecurity outcomes

Step 6: Determine Gaps
   └── Compare current vs target profiles

Step 7: Implement Action Plan
   └── Prioritize and address gaps based on risk
```

---

## Summary Table

| Component | Description |
|-----------|-------------|
| **Core Functions** | Identify, Protect, Detect, Respond, Recover (+ Govern in 2.0) |
| **Tiers** | Partial → Risk Informed → Repeatable → Adaptive |
| **Profiles** | Current state vs Target state → Gap analysis |
| **Applicability** | All industries, all sizes |
| **Cost** | Free, voluntary, widely adopted |
| **Latest Version** | NIST CSF 2.0 (2024) |

---

## Quick Revision Questions

1. **What are the five core functions of NIST CSF?**
2. **What function was added in NIST CSF 2.0?**
3. **Describe the four maturity tiers.**
4. **What is the purpose of a Framework Profile?**
5. **How do you perform a gap analysis using NIST CSF?**
6. **Is NIST CSF mandatory or voluntary? Who uses it?**

---

[Next: ISO 27001/27002 →](02-iso-27001-27002.md)

---

*Unit 4 - Topic 1 of 6 | [Back to README](../README.md)*
