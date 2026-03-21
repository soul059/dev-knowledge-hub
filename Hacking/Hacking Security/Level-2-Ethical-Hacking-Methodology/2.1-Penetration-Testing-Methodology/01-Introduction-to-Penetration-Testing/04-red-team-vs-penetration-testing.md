# Red Team vs Penetration Testing

## Unit 1 - Topic 4 | Introduction to Penetration Testing

---

## Overview

While both **red team engagements** and **penetration tests** involve attacking systems, they differ significantly in **scope**, **objectives**, **duration**, and **approach**. A penetration test focuses on finding technical vulnerabilities; a red team exercise simulates a **real-world adversary** to test the organization's entire security posture — including people and processes.

---

## 1. Core Differences

```
┌──────────────────────────────────────────────────────────────┐
│              PEN TEST vs RED TEAM                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PENETRATION TEST:                                           │
│  "Find as many vulnerabilities as possible"                  │
│  ├── Scope: Defined targets (IPs, apps, networks)           │
│  ├── Duration: 1-3 weeks                                     │
│  ├── Stealth: Not required                                   │
│  ├── Goal: Identify & exploit technical flaws               │
│  └── Blue team: Usually informed                             │
│                                                              │
│  RED TEAM:                                                   │
│  "Achieve specific objectives like a real attacker"          │
│  ├── Scope: Entire organization (few limits)                │
│  ├── Duration: 2-6 months                                    │
│  ├── Stealth: Essential (evade detection)                    │
│  ├── Goal: Test detection & response capabilities           │
│  └── Blue team: NOT informed                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Detailed Comparison

| Aspect | Penetration Test | Red Team |
|--------|:---:|:---:|
| **Scope** | Defined assets | Entire organization |
| **Duration** | 1-4 weeks | 1-6 months |
| **Objective** | Find vulnerabilities | Achieve objectives |
| **Stealth** | Optional | Required |
| **Detection** | Doesn't matter | Must evade |
| **Techniques** | Technical exploitation | Technical + social + physical |
| **Blue team awareness** | Usually knows | Does NOT know |
| **Coverage** | Broad (many vulns) | Deep (specific path) |
| **Cost** | $10K-$100K | $50K-$500K+ |
| **Report focus** | Vulnerability list | Attack narrative |
| **Tests** | Technical controls | People + Process + Technology |

---

## 3. Team Types

```
┌──────────────────────────────────────────────────────────────┐
│              SECURITY TEAM COLORS                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  🔴 RED TEAM (Offense)                                       │
│     Simulate real-world attackers                            │
│     Use TTPs (Tactics, Techniques, Procedures)              │
│     Goal: Break in, achieve objectives                       │
│                                                              │
│  🔵 BLUE TEAM (Defense)                                      │
│     Monitor, detect, and respond                             │
│     SOC analysts, incident responders                        │
│     Goal: Detect and stop attacks                            │
│                                                              │
│  🟣 PURPLE TEAM (Collaboration)                              │
│     Red + Blue work together                                 │
│     Maximize learning from exercises                         │
│     Goal: Improve detection and response                     │
│                                                              │
│  ⬜ WHITE TEAM (Oversight)                                    │
│     Manages the engagement                                   │
│     Sets rules, arbitrates disputes                          │
│     Goal: Ensure safe, productive exercise                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Red Team Objectives (Examples)

| Objective | Description |
|-----------|-------------|
| **Access CEO's email** | Social engineering + credential theft |
| **Exfiltrate customer data** | Full attack chain to database |
| **Deploy ransomware (simulated)** | Test if ransomware would succeed |
| **Access SCADA systems** | Test OT/ICS segmentation |
| **Physical breach** | Enter secure facility without authorization |
| **Compromise domain admin** | Full AD compromise |

---

## 5. MITRE ATT&CK Framework

```
Red teams use MITRE ATT&CK to structure their attacks:

TACTIC             │ EXAMPLE TECHNIQUES
────────────────────┼──────────────────────────────
Initial Access      │ Phishing, public exploit
Execution           │ PowerShell, scheduled task
Persistence         │ Registry run keys, services
Privilege Escalation│ UAC bypass, token manipulation
Defense Evasion     │ Obfuscation, process injection
Credential Access   │ Mimikatz, kerberoasting
Discovery           │ AD enumeration, network scan
Lateral Movement    │ PsExec, RDP, WMI
Collection          │ Keylogging, screenshots
Exfiltration        │ DNS tunneling, HTTPS upload
Impact              │ Ransomware, data destruction
```

---

## 6. When to Choose Each

```
CHOOSE PEN TEST:
├── First security assessment
├── Specific compliance requirement
├── Testing new application/network
├── Limited budget
├── Want comprehensive vulnerability list
└── Time-constrained (1-4 weeks)

CHOOSE RED TEAM:
├── Mature security program
├── Want to test detection capabilities
├── Need realistic adversary simulation
├── Testing incident response
├── Budget allows ($50K+)
└── Organization has a SOC/blue team
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Pen test** | Find all vulnerabilities in defined scope |
| **Red team** | Simulate real attacker to test detection |
| **Purple team** | Red + Blue collaborate for max learning |
| **Stealth** | Red team must evade; pen test doesn't need to |
| **MITRE ATT&CK** | Framework for structuring red team operations |
| **Duration** | Pen test: weeks; Red team: months |

---

## Quick Revision Questions

1. **What is the primary goal of a red team vs a pen test?**
2. **Why is stealth important for red teams but not pen testers?**
3. **What is a purple team?**
4. **When should an organization choose a red team over a pen test?**
5. **What is the MITRE ATT&CK framework?**

---

[← Previous: Pen Testing vs Vulnerability Assessment](03-pentest-vs-vulnerability-assessment.md) | [Next: Bug Bounty Programs →](05-bug-bounty-programs.md)

---

*Unit 1 - Topic 4 of 5 | [Back to README](../README.md)*
