# Choosing a Methodology

## Unit 3 - Topic 5 | Penetration Testing Standards

---

## Overview

No single methodology fits every engagement. **Choosing the right methodology** depends on the **target type**, **client requirements**, **compliance needs**, and **engagement scope**. Most experienced testers combine elements from multiple standards to create a tailored approach.

---

## 1. Methodology Selection Matrix

| Factor | PTES | OWASP | NIST 800-115 | OSSTMM |
|--------|:---:|:---:|:---:|:---:|
| **Network pen test** | ✅ Best | ⚠️ Partial | ✅ Good | ✅ Good |
| **Web app pen test** | ✅ Good | ✅ Best | ⚠️ Partial | ⚠️ Partial |
| **Government/compliance** | ⚠️ Partial | ⚠️ Partial | ✅ Best | ⚠️ Partial |
| **Physical security** | ⚠️ Partial | ❌ | ❌ | ✅ Best |
| **Quantitative metrics** | ❌ | ❌ | ❌ | ✅ Best |
| **Beginner-friendly** | ✅ | ✅ | ✅ | ❌ Complex |
| **Free/Open** | ✅ | ✅ | ✅ | ✅ |

---

## 2. Decision Framework

```
CHOOSING YOUR METHODOLOGY:

What are you testing?
│
├── Web Application?
│   └── OWASP Testing Guide (primary)
│       + PTES for engagement structure
│
├── Network Infrastructure?
│   └── PTES (primary)
│       + NIST 800-115 for documentation
│
├── Government/Federal System?
│   └── NIST 800-115 (primary, required)
│       + PTES for technical depth
│
├── Full Organization (including physical)?
│   └── OSSTMM (primary)
│       + PTES for pen test phases
│
├── Mobile Application?
│   └── OWASP Mobile Testing Guide
│       + PTES for engagement structure
│
├── API?
│   └── OWASP API Security Top 10
│       + OWASP Testing Guide
│
└── Red Team Engagement?
    └── MITRE ATT&CK Framework
        + PTES for engagement structure
```

---

## 3. Hybrid Approach (Most Common)

```
REAL-WORLD METHODOLOGY (Hybrid):

┌─────────────────────────────────────────────┐
│         TYPICAL PEN TEST METHODOLOGY         │
├─────────────────────────────────────────────┤
│                                             │
│  PRE-ENGAGEMENT ──── from PTES              │
│  ├── Scoping, RoE, authorization            │
│  └── Contracts, communication plan          │
│                                             │
│  RECONNAISSANCE ──── from PTES + OSSTMM     │
│  ├── OSINT, DNS, WHOIS                      │
│  └── Social media, public records           │
│                                             │
│  SCANNING ──── from NIST 800-115            │
│  ├── Network discovery                       │
│  ├── Port scanning                           │
│  └── Vulnerability scanning                  │
│                                             │
│  WEB APP TESTING ──── from OWASP            │
│  ├── OWASP Top 10 test cases                │
│  └── Business logic testing                  │
│                                             │
│  EXPLOITATION ──── from PTES                │
│  ├── Verified exploitation                   │
│  └── Post-exploitation                       │
│                                             │
│  REPORTING ──── from PTES + NIST            │
│  ├── Executive + technical report            │
│  └── Risk ratings + remediation              │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 4. Methodology Maturity

```
PROGRESSION FOR PEN TESTERS:

BEGINNER:
└── Follow PTES step-by-step
    └── Use OWASP checklist for web apps

INTERMEDIATE:
└── Combine PTES + OWASP + NIST
    └── Develop personal checklists

ADVANCED:
└── Custom methodology based on experience
    └── Pull from all standards as needed
    └── Adapt to each unique engagement

EXPERT:
└── Contribute to methodology development
    └── Create internal standard for team
    └── Mentor others on methodology
```

---

## 5. Certification Alignment

| Certification | Primary Methodology |
|---------------|:---:|
| **CEH** | EC-Council methodology (similar to PTES) |
| **OSCP** | Practical (custom, closest to PTES) |
| **GPEN** | PTES-based |
| **CompTIA PenTest+** | Mix of PTES + NIST |
| **eJPT** | PTES-based |
| **CREST** | OWASP + PTES |
| **PNPT** | Practical (TCM methodology, PTES-based) |

---

## Summary Table

| Methodology | Best For |
|-------------|----------|
| **PTES** | General pen testing (most universal) |
| **OWASP** | Web application testing |
| **NIST 800-115** | Government compliance |
| **OSSTMM** | Quantitative/scientific assessment |
| **MITRE ATT&CK** | Red team operations |
| **Hybrid** | Real-world engagements (most common) |

---

## Quick Revision Questions

1. **Which methodology is best for web application testing?**
2. **What methodology should you use for US government systems?**
3. **Why do most testers use a hybrid approach?**
4. **Which framework is best for red team operations?**
5. **How does methodology choice change as you gain experience?**

---

[← Previous: OSSTMM](04-osstmm.md)

---

*Unit 3 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Pre-engagement →](../04-Pre-engagement/01-scoping-discussions.md)*
