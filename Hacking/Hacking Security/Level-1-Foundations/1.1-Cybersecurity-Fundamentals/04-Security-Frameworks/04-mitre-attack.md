# MITRE ATT&CK

## Unit 4 - Topic 4 | Security Frameworks

---

## Overview

**MITRE ATT&CK** (Adversarial Tactics, Techniques, and Common Knowledge) is a globally-accessible knowledge base of adversary tactics and techniques based on real-world observations. Unlike frameworks that focus on what to defend, ATT&CK focuses on **how adversaries actually attack** — making it invaluable for threat detection, hunting, red teaming, and security assessments.

---

## 1. What is MITRE ATT&CK?

```
┌──────────────────────────────────────────────────────────────────┐
│                      MITRE ATT&CK                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  NOT a compliance framework (like NIST, ISO)                     │
│  NOT a prioritized checklist (like CIS)                          │
│                                                                  │
│  IT IS:                                                          │
│  ✅ A knowledge base of REAL adversary behavior                  │
│  ✅ A common language for describing attacks                     │
│  ✅ A framework for building detection rules                     │
│  ✅ A tool for assessing security coverage                       │
│  ✅ Used by both Red Teams AND Blue Teams                        │
│                                                                  │
│  Organized as: TACTICS ──► TECHNIQUES ──► SUB-TECHNIQUES         │
│                (WHY)        (HOW)         (SPECIFIC HOW)          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. ATT&CK Matrix Structure

### The 14 Tactics (Enterprise)

| # | Tactic | Description |
|---|--------|-------------|
| 1 | **Reconnaissance** | Gathering information for planning the attack |
| 2 | **Resource Development** | Building infrastructure for the attack |
| 3 | **Initial Access** | Getting into the network |
| 4 | **Execution** | Running malicious code |
| 5 | **Persistence** | Maintaining access after reboot/remediation |
| 6 | **Privilege Escalation** | Getting higher-level permissions |
| 7 | **Defense Evasion** | Avoiding detection |
| 8 | **Credential Access** | Stealing credentials |
| 9 | **Discovery** | Learning about the environment |
| 10 | **Lateral Movement** | Moving through the network |
| 11 | **Collection** | Gathering target data |
| 12 | **Command and Control** | Communicating with compromised systems |
| 13 | **Exfiltration** | Stealing data out of the network |
| 14 | **Impact** | Disrupting, destroying, or manipulating systems |

```
ATT&CK MATRIX (Simplified View):

Recon → Resource → Initial → Execution → Persist → Priv Esc
  Dev     Access                                        │
                                                        ▼
Impact ← Exfil ← C2 ← Collection ← Lat Move ← Discovery
                                                    ↑
                                           Def Evasion
                                           Cred Access
```

---

## 3. Tactics → Techniques → Sub-Techniques

```
TACTIC: Initial Access (TA0001)
  │
  ├── TECHNIQUE: Phishing (T1566)
  │     │
  │     ├── SUB-TECHNIQUE: Spearphishing Attachment (T1566.001)
  │     ├── SUB-TECHNIQUE: Spearphishing Link (T1566.002)
  │     └── SUB-TECHNIQUE: Spearphishing via Service (T1566.003)
  │
  ├── TECHNIQUE: Exploit Public-Facing App (T1190)
  │
  ├── TECHNIQUE: Supply Chain Compromise (T1195)
  │     │
  │     ├── SUB-TECHNIQUE: Compromise Software Supply Chain (T1195.002)
  │     └── SUB-TECHNIQUE: Compromise Hardware Supply Chain (T1195.003)
  │
  └── TECHNIQUE: Valid Accounts (T1078)
        │
        ├── SUB-TECHNIQUE: Default Accounts (T1078.001)
        ├── SUB-TECHNIQUE: Domain Accounts (T1078.002)
        └── SUB-TECHNIQUE: Cloud Accounts (T1078.004)
```

---

## 4. How to Use MITRE ATT&CK

| Use Case | Who | How |
|----------|-----|-----|
| **Threat Intelligence** | TI analysts | Map known APTs to ATT&CK techniques |
| **Detection Engineering** | SOC / Detection team | Build detection rules for specific techniques |
| **Red Teaming** | Pen testers / Red team | Plan operations using ATT&CK as a checklist |
| **Gap Assessment** | Security leaders | Map existing detections to ATT&CK and find gaps |
| **Incident Response** | IR team | Classify observed behavior using ATT&CK taxonomy |
| **Vendor Evaluation** | Security architects | Assess if tools cover key ATT&CK techniques |

---

## 5. ATT&CK Matrices Available

| Matrix | Target Environment |
|--------|-------------------|
| **Enterprise** | Windows, macOS, Linux, Cloud, Network, Containers |
| **Mobile** | Android, iOS |
| **ICS** | Industrial Control Systems |

---

## 6. ATT&CK Navigator

The **ATT&CK Navigator** is a web tool for visualizing coverage:

```
┌──────────────────────────────────────────────────────────────┐
│              ATT&CK NAVIGATOR USES                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  • Map your detection coverage (green = covered)             │
│  • Identify gaps (red = no detection)                        │
│  • Compare adversary TTPs to your defenses                   │
│  • Plan red team exercises                                   │
│  • Prioritize detection engineering work                     │
│                                                              │
│  URL: https://mitre-attack.github.io/attack-navigator/      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **What** | Knowledge base of real adversary tactics and techniques |
| **Structure** | 14 Tactics → ~200 Techniques → ~400 Sub-techniques |
| **Matrices** | Enterprise, Mobile, ICS |
| **Key Tool** | ATT&CK Navigator for visualization |
| **Used By** | Red teams, blue teams, threat intel, IR, security leaders |
| **Not** | A compliance framework — it's a threat-based knowledge base |

---

## Quick Revision Questions

1. **What does MITRE ATT&CK stand for?**
2. **What is the difference between a tactic, technique, and sub-technique?**
3. **List the 14 enterprise tactics in order.**
4. **How would a SOC team use ATT&CK?**
5. **How would a red team use ATT&CK?**
6. **What is the ATT&CK Navigator and what is it used for?**

---

[← Previous: CIS Controls](03-cis-controls.md) | [Next: OWASP →](05-owasp.md)

---

*Unit 4 - Topic 4 of 6 | [Back to README](../README.md)*
