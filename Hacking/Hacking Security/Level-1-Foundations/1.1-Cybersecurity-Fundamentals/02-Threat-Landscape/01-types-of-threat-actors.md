# Types of Threat Actors

## Unit 2 - Topic 1 | Threat Landscape

---

## Overview

A **threat actor** (also called a **threat agent**) is any individual or group that can carry out a cyber attack. Understanding who the adversaries are, their capabilities, resources, and motivations is essential for building effective defenses.

---

## 1. Threat Actor Categories

```
┌──────────────────────────────────────────────────────────────────────┐
│                     THREAT ACTOR SPECTRUM                            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Low Skill / Low Resource              High Skill / High Resource    │
│  ◄──────────────────────────────────────────────────────────────►    │
│                                                                      │
│  Script       Hacktivists   Cyber      Organized    Nation-State     │
│  Kiddies                   Criminals   Crime Groups  (APT)           │
│                                                                      │
│  ⚡ Low         ⚡⚡ Med      ⚡⚡⚡ High  ⚡⚡⚡⚡ V.High  ⚡⚡⚡⚡⚡ Max    │
│  Threat         Threat       Threat     Threat        Threat         │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 2. Detailed Threat Actor Profiles

### Script Kiddies

```
┌────────────────────────────────────────────────────────────┐
│  SCRIPT KIDDIES                                            │
├────────────────────────────────────────────────────────────┤
│  Skill Level:    ★☆☆☆☆  (Low)                             │
│  Resources:      ★☆☆☆☆  (Minimal)                         │
│  Motivation:     Fun, bragging rights, curiosity           │
│  Targets:        Easy, unpatched systems                   │
│  Tools:          Pre-made exploit scripts, public tools    │
│  Persistence:    Low — give up easily                      │
│  Sophistication: Low — use tools without understanding     │
├────────────────────────────────────────────────────────────┤
│  Example: Teen uses a downloaded DDoS tool against a       │
│  gaming server for revenge after being banned.             │
└────────────────────────────────────────────────────────────┘
```

### Hacktivists

```
┌────────────────────────────────────────────────────────────┐
│  HACKTIVISTS                                               │
├────────────────────────────────────────────────────────────┤
│  Skill Level:    ★★★☆☆  (Moderate)                         │
│  Resources:      ★★☆☆☆  (Volunteer-based)                  │
│  Motivation:     Political, social, ideological cause      │
│  Targets:        Governments, corporations, controversial  │
│  Tools:          DDoS, defacement, data leaks              │
│  Persistence:    Medium — campaign-driven                  │
│  Sophistication: Moderate — coordinated campaigns          │
├────────────────────────────────────────────────────────────┤
│  Examples: Anonymous, LulzSec                              │
│  Attacks: Website defacement, data dumps, DDoS             │
└────────────────────────────────────────────────────────────┘
```

### Cybercriminals

```
┌────────────────────────────────────────────────────────────┐
│  CYBERCRIMINALS                                            │
├────────────────────────────────────────────────────────────┤
│  Skill Level:    ★★★★☆  (High)                             │
│  Resources:      ★★★★☆  (Profit-funded)                    │
│  Motivation:     Financial gain                            │
│  Targets:        Anyone with money/data                    │
│  Tools:          Ransomware, phishing, exploit kits        │
│  Persistence:    High — driven by profit                   │
│  Sophistication: High — RaaS, dark web services            │
├────────────────────────────────────────────────────────────┤
│  Examples: REvil, DarkSide, Conti                          │
│  Model: Ransomware-as-a-Service (RaaS)                     │
│  Revenue: Billions of dollars annually                     │
└────────────────────────────────────────────────────────────┘
```

### Organized Crime Groups

```
┌────────────────────────────────────────────────────────────┐
│  ORGANIZED CRIME GROUPS                                    │
├────────────────────────────────────────────────────────────┤
│  Skill Level:    ★★★★☆  (High)                             │
│  Resources:      ★★★★★  (Well-funded)                      │
│  Motivation:     Financial gain at scale                   │
│  Targets:        Banks, payment systems, large enterprises │
│  Tools:          Custom malware, money laundering infra    │
│  Persistence:    Very High — business operations           │
│  Sophistication: Very High — dedicated teams               │
├────────────────────────────────────────────────────────────┤
│  Structure: CEO, developers, affiliates, money mules       │
│  Run like a business with HR, support, revenue targets     │
└────────────────────────────────────────────────────────────┘
```

### Nation-State Actors (APT)

```
┌────────────────────────────────────────────────────────────┐
│  NATION-STATE ACTORS (APT)                                 │
├────────────────────────────────────────────────────────────┤
│  Skill Level:    ★★★★★  (Elite)                            │
│  Resources:      ★★★★★  (Unlimited government backing)     │
│  Motivation:     Espionage, disruption, geopolitics        │
│  Targets:        Governments, critical infrastructure,     │
│                  defense, intellectual property             │
│  Tools:          Zero-days, custom implants, supply chain  │
│  Persistence:    Maximum — years-long campaigns            │
│  Sophistication: Maximum — dedicated cyber commands         │
├────────────────────────────────────────────────────────────┤
│  Examples:                                                 │
│  • APT28 (Fancy Bear) — Russia                             │
│  • APT29 (Cozy Bear) — Russia                              │
│  • APT41 (Double Dragon) — China                           │
│  • Lazarus Group — North Korea                             │
│  • Equation Group — United States (attributed)             │
└────────────────────────────────────────────────────────────┘
```

### Insider Threats

```
┌────────────────────────────────────────────────────────────┐
│  INSIDER THREATS                                           │
├────────────────────────────────────────────────────────────┤
│  Skill Level:    Varies (already have access)              │
│  Resources:      Legitimate system access                  │
│  Motivation:     Money, revenge, ideology, accidental      │
│  Targets:        Own organization's systems and data       │
│  Advantage:      Bypass perimeter security (already inside)│
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Types:                                                    │
│  ┌──────────────────────────────────────────────────┐     │
│  │  MALICIOUS    - Intentional harm (disgruntled)    │     │
│  │  NEGLIGENT    - Careless actions (clicking phish) │     │
│  │  COMPROMISED  - Account hijacked by external actor│     │
│  └──────────────────────────────────────────────────┘     │
│                                                            │
│  Most damaging because they have trusted access.           │
└────────────────────────────────────────────────────────────┘
```

---

## 3. Threat Actor Comparison Table

| Actor | Skill | Resources | Motivation | Persistence | Example |
|-------|-------|-----------|-----------|-------------|---------|
| **Script Kiddie** | Low | Minimal | Fun, ego | Low | Teen with DDoS tool |
| **Hacktivist** | Moderate | Volunteer | Ideology | Medium | Anonymous |
| **Cybercriminal** | High | Profit-funded | Money | High | Ransomware gangs |
| **Organized Crime** | High | Well-funded | Money at scale | Very High | Carbanak group |
| **Nation-State** | Elite | Government | Espionage/warfare | Maximum | APT28, Lazarus |
| **Insider** | Varies | Legitimate access | Varied | Varies | Disgruntled employee |
| **Competitor** | Moderate | Corporate | Competitive advantage | Medium | Corporate espionage |
| **Terrorist** | Moderate | Group-funded | Fear, disruption | High | Cyber terrorism |

---

## 4. Defending Against Each Actor Type

| Threat Actor | Key Defenses |
|-------------|-------------|
| **Script Kiddies** | Patch regularly, basic firewall rules, strong passwords |
| **Hacktivists** | DDoS protection, web app security, incident response plan |
| **Cybercriminals** | Email security, endpoint protection, backups, MFA |
| **Organized Crime** | Advanced threat detection, threat intelligence, zero trust |
| **Nation-State** | Defense in depth, threat hunting, classified info protection |
| **Insiders** | DLP, user behavior analytics, least privilege, audit logs |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Threat Actor** | Any entity capable of performing a cyberattack |
| **Spectrum** | Ranges from low-skill script kiddies to nation-state APTs |
| **Most Dangerous** | Nation-states (capability) and insiders (access) |
| **Fastest Growing** | Cybercriminals using RaaS models |
| **Hardest to Detect** | Insiders with legitimate access |
| **Defense Strategy** | Tailor controls to the most likely threat actors for your org |

---

## Quick Revision Questions

1. **List 6 types of threat actors and rank them by capability.**
2. **What makes insider threats particularly dangerous?**
3. **What is the difference between a hacktivist and a cybercriminal?**
4. **Name 3 known APT groups and their attributed nation-states.**
5. **How does a Ransomware-as-a-Service (RaaS) model work?**
6. **What defensive strategy would you prioritize against nation-state actors vs script kiddies?**

---

[Next: Attack Motivations →](02-attack-motivations.md)

---

*Unit 2 - Topic 1 of 6 | [Back to README](../README.md)*
