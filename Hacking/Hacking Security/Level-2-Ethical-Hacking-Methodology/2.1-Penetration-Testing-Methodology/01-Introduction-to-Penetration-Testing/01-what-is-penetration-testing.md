# What is Penetration Testing?

## Unit 1 - Topic 1 | Introduction to Penetration Testing

---

## Overview

**Penetration testing** (pen testing) is an **authorized, simulated cyberattack** against a computer system, network, or application to evaluate its security. Unlike malicious hacking, pen testing is conducted with **explicit permission**, follows a **structured methodology**, and produces a **report** with findings and remediation advice.

---

## 1. Definition and Purpose

```
┌──────────────────────────────────────────────────────────────┐
│              WHAT IS PENETRATION TESTING?                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  "A methodical, authorized attempt to evaluate the           │
│   security of an IT infrastructure by safely exploiting      │
│   vulnerabilities."                                          │
│                                                              │
│  KEY ELEMENTS:                                               │
│  ✅ Authorized — written permission required                  │
│  ✅ Simulated — mimics real-world attacks                    │
│  ✅ Systematic — follows a methodology                       │
│  ✅ Documented — every action is recorded                    │
│  ✅ Reported — findings delivered to stakeholders            │
│  ✅ Safe — minimize damage to target systems                 │
│                                                              │
│  PURPOSE:                                                    │
│  • Find vulnerabilities before attackers do                  │
│  • Test security controls effectiveness                      │
│  • Meet compliance requirements (PCI-DSS, HIPAA)            │
│  • Validate incident response capabilities                   │
│  • Provide actionable remediation guidance                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Penetration Testing Lifecycle

```
┌────────────┐    ┌────────────┐    ┌────────────┐
│    PRE-     │    │   RECON    │    │  SCANNING  │
│ ENGAGEMENT  │───►│  & OSINT   │───►│  & ENUM    │
│             │    │            │    │            │
└────────────┘    └────────────┘    └─────┬──────┘
                                          │
┌────────────┐    ┌────────────┐    ┌─────▼──────┐
│  REPORTING  │◄───│   POST-    │◄───│EXPLOITATION│
│  & CLEANUP  │    │EXPLOITATION│    │            │
│             │    │            │    │            │
└────────────┘    └────────────┘    └────────────┘

Phase 1: Pre-engagement — Scope, rules, authorization
Phase 2: Reconnaissance — Gather information about target
Phase 3: Scanning & Enumeration — Find open ports, services
Phase 4: Exploitation — Attempt to exploit vulnerabilities
Phase 5: Post-exploitation — Escalate, pivot, assess impact
Phase 6: Reporting — Document findings, recommend fixes
```

---

## 3. Who Performs Penetration Tests?

| Role | Description |
|------|-------------|
| **Internal team** | Organization's own security team |
| **External firm** | Third-party security consultancy |
| **Independent consultant** | Freelance pen tester |
| **Bug bounty hunter** | Crowd-sourced security researcher |

### Common Job Titles:
- Penetration Tester
- Ethical Hacker
- Security Consultant
- Red Team Operator
- Offensive Security Engineer
- Application Security Tester

---

## 4. When to Conduct Pen Tests

| Trigger | Description |
|---------|-------------|
| **Regulatory** | PCI-DSS requires annual pen test |
| **New deployment** | Before launching new systems |
| **Major changes** | After significant infrastructure changes |
| **After incident** | Validate that fixes are effective |
| **Periodic** | Quarterly or annually as part of security program |
| **M&A** | Before acquiring or merging with another company |

---

## 5. Penetration Testing vs Hacking

| Aspect | Penetration Testing | Malicious Hacking |
|--------|:---:|:---:|
| **Authorization** | ✅ Written permission | ❌ Unauthorized |
| **Intent** | Improve security | Steal/damage |
| **Methodology** | Structured | Varies |
| **Documentation** | Complete records | Minimal/none |
| **Reporting** | Detailed report | No report |
| **Legal** | Legal activity | Criminal activity |
| **Scope** | Defined boundaries | No boundaries |
| **Cleanup** | Remove artifacts | Leave backdoors |

---

## 6. Value of Penetration Testing

```
BUSINESS VALUE:
├── Risk Reduction
│   └── Find and fix vulnerabilities before breach
├── Compliance
│   └── Meet regulatory requirements (PCI, HIPAA, SOX)
├── Customer Trust
│   └── Demonstrate security commitment
├── Cost Savings
│   └── Pen test cost << breach cost ($4.45M average)
├── Security Awareness
│   └── Educate teams about real threats
└── Insurance
    └── May reduce cyber insurance premiums
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Pen testing** | Authorized simulated attack to find vulnerabilities |
| **Authorization** | Must have written permission before any testing |
| **Lifecycle** | Pre-engagement → Recon → Scan → Exploit → Report |
| **Purpose** | Find flaws before real attackers do |
| **Compliance** | Required by PCI-DSS, HIPAA, and other standards |
| **Value** | Risk reduction, compliance, cost savings |

---

## Quick Revision Questions

1. **What distinguishes a pen test from malicious hacking?**
2. **What are the 6 phases of the pen testing lifecycle?**
3. **When should an organization conduct a pen test?**
4. **Why is written authorization essential?**
5. **Name 3 business benefits of penetration testing.**

---

[Next: Types of Penetration Tests →](02-types-of-penetration-tests.md)

---

*Unit 1 - Topic 1 of 5 | [Back to README](../README.md)*
