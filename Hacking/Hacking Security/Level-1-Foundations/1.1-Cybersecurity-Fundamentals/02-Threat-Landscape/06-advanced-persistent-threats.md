# APT (Advanced Persistent Threats)

## Unit 2 - Topic 6 | Threat Landscape

---

## Overview

An **Advanced Persistent Threat (APT)** is a sophisticated, long-term cyberattack campaign — typically carried out by nation-state actors or highly organized groups — that targets specific organizations to steal data, conduct espionage, or cause disruption. APTs are the most dangerous category of cyber threats due to their resources, patience, and sophistication.

---

## 1. What Makes an APT "Advanced, Persistent, and a Threat"?

```
┌──────────────────────────────────────────────────────────────────┐
│                        A.P.T. BREAKDOWN                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  A = ADVANCED                                                    │
│  ─────────────                                                   │
│  • Custom malware and zero-day exploits                          │
│  • Multiple attack vectors simultaneously                       │
│  • Sophisticated evasion techniques                              │
│  • Deep technical expertise                                      │
│                                                                  │
│  P = PERSISTENT                                                  │
│  ──────────────                                                  │
│  • Long-term presence (months to years)                          │
│  • Multiple persistence mechanisms                               │
│  • Continuous objectives — not one-and-done                      │
│  • Adapt when detected — don't give up                           │
│                                                                  │
│  T = THREAT                                                      │
│  ─────────                                                       │
│  • Organized, well-funded teams                                  │
│  • Clear objectives and motivation                               │
│  • Capability + Intent + Opportunity                             │
│  • Real damage to organizations and nations                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. APT Attack Lifecycle

```
┌──────────────────────────────────────────────────────────────────┐
│                    APT ATTACK LIFECYCLE                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1: RECONNAISSANCE (Weeks-Months)                          │
│  ├── OSINT on target organization                                │
│  ├── Employee profiling via social media                         │
│  ├── Technology stack identification                             │
│  └── Supply chain mapping                                        │
│         │                                                        │
│         ▼                                                        │
│  Phase 2: INITIAL COMPROMISE                                     │
│  ├── Spear phishing key employees                                │
│  ├── Watering hole attacks                                       │
│  ├── Supply chain compromise                                     │
│  └── Zero-day exploitation                                       │
│         │                                                        │
│         ▼                                                        │
│  Phase 3: ESTABLISH FOOTHOLD                                     │
│  ├── Install backdoor / RAT (Remote Access Trojan)               │
│  ├── Create persistence mechanisms                               │
│  ├── Establish C2 communications                                 │
│  └── Evade initial detection                                     │
│         │                                                        │
│         ▼                                                        │
│  Phase 4: ESCALATE PRIVILEGES                                    │
│  ├── Local privilege escalation                                  │
│  ├── Credential harvesting (Mimikatz-style)                      │
│  ├── Kerberoasting / Pass-the-Hash                               │
│  └── Target domain admin accounts                                │
│         │                                                        │
│         ▼                                                        │
│  Phase 5: LATERAL MOVEMENT                                       │
│  ├── Move through network to target systems                      │
│  ├── Compromise additional hosts                                 │
│  ├── Map internal network and resources                          │
│  └── Reach data repositories / crown jewels                      │
│         │                                                        │
│         ▼                                                        │
│  Phase 6: DATA EXFILTRATION / OBJECTIVE                          │
│  ├── Collect and stage target data                               │
│  ├── Exfiltrate via encrypted channels                           │
│  ├── Or: Sabotage / Destroy systems                              │
│  └── Maintain access for future operations                       │
│                                                                  │
│  TOTAL DURATION: Months to YEARS                                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Known APT Groups

| APT Group | Attribution | Primary Targets | Notable Operations |
|-----------|-----------|-----------------|-------------------|
| **APT28** (Fancy Bear) | Russia (GRU) | NATO, elections, defense | DNC hack (2016), Olympics |
| **APT29** (Cozy Bear) | Russia (SVR) | Government, think tanks | SolarWinds (2020) |
| **APT41** (Double Dragon) | China (MSS) | Healthcare, tech, gaming | Dual espionage + crime |
| **APT1** (Comment Crew) | China (PLA) | Defense, aerospace | 100+ companies compromised |
| **Lazarus Group** | North Korea | Financial, crypto | Sony hack, WannaCry, bank heists |
| **APT33** (Elfin) | Iran | Aviation, energy, petrochemical | Shamoon attacks |
| **Equation Group** | USA (attributed/NSA) | Global targets | Stuxnet (attributed) |
| **Turla** (Snake) | Russia (FSB) | Government, embassies | Satellite-based C2 |

---

## 4. APT vs Regular Cyberattack

| Characteristic | Regular Attack | APT |
|---------------|---------------|-----|
| **Duration** | Hours to days | Months to years |
| **Sophistication** | Low to medium | Very high |
| **Targets** | Opportunistic | Specific, researched |
| **Tools** | Public exploit kits | Custom malware, zero-days |
| **Motivation** | Financial (usually) | Espionage, geopolitical |
| **Persistence** | Minimal | Extensive, multiple backdoors |
| **Evasion** | Basic | Advanced (fileless, encrypted C2) |
| **Response to Detection** | Abandon operation | Adapt and continue |
| **Resources** | Limited | Nation-state funding |

---

## 5. APT Indicators

```
┌──────────────────────────────────────────────────────────────┐
│              SIGNS OF AN APT IN YOUR NETWORK                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ⚠️ Unusual outbound traffic (data exfiltration)             │
│  ⚠️ Unexpected login times (off-hours, weekends)             │
│  ⚠️ Large data bundles being moved to unknown locations      │
│  ⚠️ Backdoor trojans detected                                │
│  ⚠️ Spear phishing targeting key employees                   │
│  ⚠️ Unknown processes communicating externally               │
│  ⚠️ Anomalous DNS queries (DNS tunneling)                    │
│  ⚠️ Credential abuse (pass-the-hash, golden ticket)          │
│  ⚠️ Lateral movement detected between systems                │
│  ⚠️ Legitimate tools used maliciously (LOLBins)              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6. Defending Against APTs

| Defense Layer | Techniques |
|-------------|------------|
| **Prevention** | Email security, patch management, MFA, network segmentation |
| **Detection** | EDR, SIEM, threat hunting, behavioral analytics, deception tech |
| **Response** | Incident response plan, forensics capability, threat intelligence |
| **Recovery** | Backups, disaster recovery, business continuity |
| **Intelligence** | Subscribe to threat feeds, join ISACs, monitor dark web |

### Key Frameworks for APT Defense

| Framework | Use for APTs |
|-----------|-------------|
| **MITRE ATT&CK** | Map APT TTPs and build detection rules |
| **Cyber Kill Chain** | Identify attack phases and disrupt early |
| **Diamond Model** | Analyze adversary-infrastructure-victim relationships |
| **NIST CSF** | Organize overall security posture |

---

## 7. Famous APT Case Study: SolarWinds (2020)

```
┌──────────────────────────────────────────────────────────────────┐
│             SOLARWINDS SUPPLY CHAIN ATTACK                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. APT29 compromised SolarWinds build server                    │
│                    │                                             │
│  2. Injected malicious code (SUNBURST) into Orion update         │
│                    │                                             │
│  3. SolarWinds distributed legitimate-looking update             │
│                    │                                             │
│  4. ~18,000 organizations installed the update                   │
│                    │                                             │
│  5. SUNBURST created backdoor, contacted C2 servers              │
│                    │                                             │
│  6. Attackers selected ~100 high-value targets                   │
│     (US government agencies, Fortune 500)                        │
│                    │                                             │
│  7. Deployed additional payloads (TEARDROP, RAINDROP)            │
│                    │                                             │
│  8. Operated undetected for ~9 months                            │
│                                                                  │
│  IMPACT: One of the most significant supply chain attacks        │
│  in history. Compromised US Treasury, DHS, and more.             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Definition** | Sophisticated, long-term targeted attack (usually nation-state) |
| **Duration** | Months to years |
| **Key Difference** | Persistence — APTs adapt and continue when detected |
| **Top Groups** | APT28/29 (Russia), APT41 (China), Lazarus (N. Korea) |
| **Lifecycle** | Recon → Compromise → Foothold → Escalate → Move → Exfiltrate |
| **Defense** | Layered: threat intel, EDR, hunting, segmentation, IR plan |

---

## Quick Revision Questions

1. **What do the letters A, P, and T stand for, and what does each mean?**
2. **Describe the six phases of the APT attack lifecycle.**
3. **Name 4 APT groups and their attributed nations.**
4. **How does an APT differ from a typical ransomware attack?**
5. **What are 5 indicators that an APT may be present in your network?**
6. **Explain the SolarWinds supply chain attack and why it was so effective.**

---

[← Previous: Common Attack Types](05-common-attack-types.md)

---

*Unit 2 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Security Principles →](../03-Security-Principles/01-least-privilege.md)*
