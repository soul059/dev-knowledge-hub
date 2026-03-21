# Threat Modeling

## Unit 5 - Topic 2 | Risk Management

---

## Overview

**Threat modeling** is a structured process for identifying potential threats, vulnerabilities, and attack vectors in a system — then determining countermeasures. It's best done during the design phase of development but is valuable at any stage.

---

## 1. What is Threat Modeling?

```
┌──────────────────────────────────────────────────────────────────┐
│                    THREAT MODELING                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Four Key Questions:                                             │
│                                                                  │
│  1. "What are we building?"         → System Model               │
│  2. "What can go wrong?"            → Threat Identification      │
│  3. "What are we going to do?"      → Mitigations                │
│  4. "Did we do a good enough job?"  → Validation                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. STRIDE Threat Model

The most widely used threat classification model:

| Letter | Threat | CIA Impact | Example |
|--------|--------|-----------|---------|
| **S** | Spoofing | Authentication | Fake login page, ARP spoofing |
| **T** | Tampering | Integrity | Modifying data in transit, SQL injection |
| **R** | Repudiation | Non-repudiation | Denying an action without audit trail |
| **I** | Information Disclosure | Confidentiality | Data breach, directory listing |
| **D** | Denial of Service | Availability | DDoS, resource exhaustion |
| **E** | Elevation of Privilege | Authorization | Privilege escalation, IDOR |

### Applying STRIDE

```
For each component in the system, ask:

┌────────────────┬─────────────────────────────────────────┐
│  Component     │  STRIDE Analysis                         │
├────────────────┼─────────────────────────────────────────┤
│  Login Page    │  S: Can someone impersonate a user?      │
│                │  T: Can login data be tampered?           │
│                │  R: Can a user deny they logged in?       │
│                │  I: Can credentials be intercepted?       │
│                │  D: Can login be made unavailable?        │
│                │  E: Can a regular user become admin?      │
├────────────────┼─────────────────────────────────────────┤
│  Database      │  S: Can someone connect as DB admin?     │
│                │  T: Can records be modified?              │
│                │  R: Are DB changes logged?                │
│                │  I: Can data be extracted?                │
│                │  D: Can DB be overwhelmed?                │
│                │  E: Can app user escalate to DBA?         │
└────────────────┴─────────────────────────────────────────┘
```

---

## 3. Other Threat Modeling Methodologies

| Method | Focus | Best For |
|--------|-------|----------|
| **STRIDE** | Threat classification per component | Developers, design phase |
| **PASTA** | Process for Attack Simulation & Threat Analysis (7 stages) | Risk-centric, business aligned |
| **DREAD** | Damage, Reproducibility, Exploitability, Affected users, Discoverability | Prioritizing threats |
| **Attack Trees** | Visual tree of attack paths | Complex attack scenarios |
| **LINDDUN** | Privacy threat modeling | Privacy-focused applications |
| **VAST** | Visual, Agile, Simple Threat modeling | Agile development teams |

### DREAD Scoring

| Factor | Score (1-10) | Question |
|--------|:-:|-----------|
| **Damage** | ? | How much damage if exploited? |
| **Reproducibility** | ? | How easy to reproduce? |
| **Exploitability** | ? | How easy to exploit? |
| **Affected Users** | ? | How many users impacted? |
| **Discoverability** | ? | How easy to discover? |

Score = Average of all 5 factors. Higher = more critical.

---

## 4. Threat Modeling Process

```
Step 1: DEFINE SCOPE
   └── What system/feature are we modeling?

Step 2: CREATE DATA FLOW DIAGRAM (DFD)
   └── Show components, data flows, trust boundaries
   
   Example DFD:
   ┌───────────┐    HTTPS     ┌────────────┐   SQL    ┌──────────┐
   │  Browser  │─────────────►│  Web App   │─────────►│ Database │
   │  (User)   │              │  Server    │          │          │
   └───────────┘              └────────────┘          └──────────┘
         │                         │
   ══════╪═════════════════════════╪═══════ Trust Boundary
         │                         │
     External                  Internal

Step 3: IDENTIFY THREATS (using STRIDE)
   └── For each component and data flow

Step 4: DETERMINE MITIGATIONS
   └── What controls address each threat?

Step 5: VALIDATE
   └── Review model, test mitigations, iterate
```

---

## 5. When to Threat Model

| When | Why |
|------|-----|
| **New application design** | Catch threats before code is written (cheapest) |
| **New feature addition** | Assess impact of changes on security |
| **Architecture changes** | New components may introduce new attack surfaces |
| **After a security incident** | Update model with newly discovered threats |
| **Regularly (periodic review)** | Threat landscape evolves continuously |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Definition** | Structured process to identify threats during design |
| **Key Questions** | What are we building? What can go wrong? What to do? Did we do enough? |
| **STRIDE** | Spoofing, Tampering, Repudiation, Info Disclosure, DoS, Elevation |
| **DREAD** | Scoring model to prioritize threats |
| **DFD** | Data Flow Diagram showing components and trust boundaries |
| **Best Time** | During design phase (but valuable anytime) |

---

## Quick Revision Questions

1. **What are the four key questions in threat modeling?**
2. **What does STRIDE stand for and what does each letter mean?**
3. **How does DREAD help prioritize threats?**
4. **What is a Data Flow Diagram and why is it important?**
5. **Apply STRIDE to a login form — identify one threat per letter.**
6. **When should threat modeling be performed?**

---

[← Previous: Risk Assessment](01-risk-assessment-methodology.md) | [Next: Vulnerability Assessment →](03-vulnerability-assessment.md)

---

*Unit 5 - Topic 2 of 6 | [Back to README](../README.md)*
