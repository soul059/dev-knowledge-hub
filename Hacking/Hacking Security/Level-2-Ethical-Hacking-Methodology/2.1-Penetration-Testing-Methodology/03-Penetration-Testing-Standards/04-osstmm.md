# OSSTMM (Open Source Security Testing Methodology Manual)

## Unit 3 - Topic 4 | Penetration Testing Standards

---

## Overview

**OSSTMM** is a peer-reviewed, scientific methodology for security testing. Unlike other standards, OSSTMM focuses on **measurable security** through its unique **RAV (Risk Assessment Values)** scoring system. It tests across **5 channels** and provides quantitative security metrics.

---

## 1. OSSTMM Structure

```
┌──────────────────────────────────────────────────────────────┐
│              OSSTMM — 5 SECURITY CHANNELS                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. HUMAN SECURITY                                           │
│     Social engineering, physical social interaction          │
│     Training awareness, trust relationships                  │
│                                                              │
│  2. PHYSICAL SECURITY                                        │
│     Perimeter, access controls, surveillance                 │
│     Environmental design, barriers                           │
│                                                              │
│  3. WIRELESS COMMUNICATIONS                                  │
│     Wi-Fi, Bluetooth, RFID, infrared                        │
│     Signal emanation, electromagnetic                        │
│                                                              │
│  4. TELECOMMUNICATIONS                                       │
│     Phone systems, VoIP, fax, PBX                           │
│     Modems, dial-up, carrier signals                         │
│                                                              │
│  5. DATA NETWORKS                                            │
│     Internet-facing systems, internal networks              │
│     Applications, services, protocols                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. RAV Score (Risk Assessment Values)

```
OSSTMM UNIQUE FEATURE: Quantitative Security Metrics

RAV = ATTACK SURFACE ÷ CONTROLS

Components:
├── POROSITY     — Number of access points (attack surface)
├── CONTROLS     — Security mechanisms in place
├── LIMITATIONS  — Weaknesses in controls
└── RAV SCORE    — Numerical security rating

RAV INTERPRETATION:
├── RAV < 1.0    — More controls than exposure (SECURE)
├── RAV = 1.0    — Controls match exposure (BALANCED)
└── RAV > 1.0    — Exposure exceeds controls (VULNERABLE)

Unlike CVSS (per-vulnerability), RAV measures 
OVERALL security posture of the target.
```

---

## 3. OSSTMM Testing Approach

| Phase | Description |
|-------|-------------|
| **Induction** | Define the scope and channel(s) |
| **Interaction** | Map the target's operational surface |
| **Inquiry** | Gather information about controls |
| **Intervention** | Test controls for weaknesses |
| **Verification** | Confirm findings |
| **Reporting** | Generate RAV scores and report |

---

## 4. Control Categories

| Control Type | Description | Example |
|-------------|-------------|---------|
| **Authentication** | Verify identity | Password, MFA, biometric |
| **Indemnification** | Legal protections | Contracts, SLAs, insurance |
| **Resilience** | Ability to recover | Backups, redundancy, DR |
| **Subjugation** | Control access | ACLs, firewalls, segmentation |
| **Continuity** | Maintain operations | HA clusters, failover |
| **Non-repudiation** | Prove actions | Logging, digital signatures |
| **Confidentiality** | Protect data | Encryption, DLP |
| **Privacy** | Protect personal info | Data masking, anonymization |
| **Integrity** | Ensure accuracy | Checksums, hashing |
| **Alarm** | Detect events | IDS, SIEM, monitoring |

---

## 5. OSSTMM vs Other Standards

| Aspect | OSSTMM | PTES | OWASP | NIST |
|--------|:---:|:---:|:---:|:---:|
| **Focus** | Measurable security | Pen test methodology | Web apps | Government |
| **Channels** | 5 (including physical) | Network/app | Web only | IT systems |
| **Metrics** | RAV scores | Qualitative | Qualitative | Risk-based |
| **Approach** | Scientific | Practical | Practical | Policy |
| **Unique** | Quantitative scoring | Comprehensive phases | Test cases | Compliance |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **OSSTMM** | Scientific, measurable security testing methodology |
| **5 channels** | Human, Physical, Wireless, Telecom, Data |
| **RAV score** | Quantitative security metric |
| **Controls** | 10 categories of security controls |
| **Unique value** | Measures overall security posture numerically |

---

## Quick Revision Questions

1. **What are the 5 OSSTMM testing channels?**
2. **What does RAV measure?**
3. **How is OSSTMM different from PTES?**
4. **What does a RAV score > 1.0 indicate?**
5. **Name 3 OSSTMM control categories.**

---

[← Previous: NIST SP 800-115](03-nist-sp-800-115.md) | [Next: Choosing a Methodology →](05-choosing-a-methodology.md)

---

*Unit 3 - Topic 4 of 5 | [Back to README](../README.md)*
