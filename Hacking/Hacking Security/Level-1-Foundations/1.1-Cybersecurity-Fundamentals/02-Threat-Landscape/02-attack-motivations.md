# Attack Motivations

## Unit 2 - Topic 2 | Threat Landscape

---

## Overview

Understanding **why** attackers attack is just as important as understanding **how** they attack. Motivations range from financial gain to political ideology, personal revenge to intellectual curiosity. Knowing the motivation helps predict targets, methods, and intensity of attacks.

---

## 1. Primary Motivations

```
┌──────────────────────────────────────────────────────────────────┐
│                    ATTACK MOTIVATIONS                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  💰 FINANCIAL GAIN          Most common motivation               │
│     Ransomware, fraud, theft, cryptojacking                      │
│                                                                  │
│  🕵️ ESPIONAGE               Nation-state / corporate spying      │
│     Intellectual property, classified data, trade secrets        │
│                                                                  │
│  📢 IDEOLOGY / ACTIVISM     Hacktivism, political protest        │
│     Website defacement, data leaks, DDoS campaigns              │
│                                                                  │
│  💣 DISRUPTION / SABOTAGE   Destroy or disable systems           │
│     Critical infrastructure attacks, wipers, DoS                 │
│                                                                  │
│  🏆 FAME / EGO              Proving skills, reputation           │
│     CTF-style bragging, public disclosures                       │
│                                                                  │
│  😡 REVENGE                 Disgruntled employees/individuals    │
│     Data destruction, leaking sensitive info                     │
│                                                                  │
│  ⚔️ CYBERWARFARE            State-sponsored conflict              │
│     Infrastructure damage, election interference                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Motivation-to-Actor Mapping

| Motivation | Primary Actors | Typical Targets | Attack Types |
|-----------|---------------|-----------------|-------------|
| **Financial Gain** | Cybercriminals, organized crime | Banks, e-commerce, healthcare | Ransomware, BEC, fraud |
| **Espionage** | Nation-states, competitors | Government, defense, tech companies | APT, data exfiltration |
| **Ideology** | Hacktivists | Governments, controversial orgs | DDoS, defacement, leaks |
| **Disruption** | Nation-states, terrorists | Critical infrastructure, utilities | Wipers, sabotage |
| **Fame/Ego** | Script kiddies, researchers | Easy targets, high-profile sites | Defacement, public exploits |
| **Revenge** | Insiders | Former employer | Data theft, destruction |
| **Cyberwarfare** | Nation-states | Enemy nation's infrastructure | Stuxnet-style, election interference |
| **Curiosity** | Students, hobbyists | Accessible systems | Unauthorized access, scanning |

---

## 3. Financial Motivation Deep Dive

Financial gain is the **#1 motivation** behind cyberattacks globally.

```
┌──────────────────────────────────────────────────────────────┐
│              FINANCIAL ATTACK ECOSYSTEM                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │Ransomware│───►│ Victim   │───►│ Bitcoin  │              │
│  │  Attack  │    │  Pays    │    │ Payment  │              │
│  └──────────┘    └──────────┘    └──────────┘              │
│                                       │                     │
│  ┌──────────┐    ┌──────────┐    ┌────┴─────┐              │
│  │Data Theft│───►│ Dark Web │───►│ Money    │              │
│  │          │    │  Sale    │    │ Laundering│              │
│  └──────────┘    └──────────┘    └──────────┘              │
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │   BEC    │───►│ Wire     │───►│  Mule    │              │
│  │  Scam    │    │ Transfer │    │ Accounts │              │
│  └──────────┘    └──────────┘    └──────────┘              │
│                                                              │
│  Revenue Streams:                                            │
│  • Ransomware: $20B+ annually                                │
│  • BEC Fraud: $2.7B (FBI IC3, 2022)                          │
│  • Stolen data: $1-$50 per credit card                       │
│  • Cryptojacking: Free compute for mining                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. How Motivation Affects Attack Behavior

| Factor | Financial | Espionage | Ideology | Revenge |
|--------|----------|-----------|----------|---------|
| **Stealth** | Moderate | Maximum | Low (want attention) | Low |
| **Duration** | Days-weeks | Months-years | Hours-days | One-time |
| **Sophistication** | High | Very High | Moderate | Varies |
| **Targets** | Mass/opportunistic | Specific/targeted | Symbolic | Specific |
| **Data Interest** | PII, financials | IP, classified | Embarrassing data | Destructive |
| **Willingness to Destroy** | Low (want payment) | Low (want stealth) | Moderate | High |

---

## 5. Understanding Motivation for Defense

```
┌─────────────────────────────────────────────────────────────────┐
│         MOTIVATION → DEFENSE STRATEGY                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  If motivation = FINANCIAL:                                     │
│    → Prioritize: Ransomware protection, email security,         │
│      backups, fraud detection, employee training                │
│                                                                 │
│  If motivation = ESPIONAGE:                                     │
│    → Prioritize: Data classification, DLP, advanced threat      │
│      detection, insider threat monitoring, encryption           │
│                                                                 │
│  If motivation = DISRUPTION:                                    │
│    → Prioritize: DDoS protection, redundancy, business          │
│      continuity, incident response, resilience                  │
│                                                                 │
│  If motivation = REVENGE (insider):                             │
│    → Prioritize: Access revocation procedures, user behavior    │
│      analytics, least privilege, exit procedures                │
│                                                                 │
│  KEY: Know your likely adversaries and their motivations        │
│  to prioritize your security investments.                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Motivation | Primary Actor | Key Defense |
|-----------|--------------|-------------|
| **Financial** | Cybercriminals | Backups, email security, MFA |
| **Espionage** | Nation-states | DLP, threat hunting, encryption |
| **Ideology** | Hacktivists | DDoS protection, web security |
| **Disruption** | Nation-states/terrorists | Redundancy, business continuity |
| **Revenge** | Insiders | UBA, access controls, exit procedures |
| **Fame** | Script kiddies | Patch management, basic security |

---

## Quick Revision Questions

1. **What is the most common motivation behind cyberattacks globally?**
2. **How does an attacker's motivation affect their stealth level?**
3. **Why do hacktivists often choose low-stealth, high-visibility attacks?**
4. **What defenses would you prioritize against financially motivated attackers?**
5. **How does corporate espionage differ from nation-state espionage?**
6. **Give an example of a cyberattack driven by revenge.**

---

[← Previous: Types of Threat Actors](01-types-of-threat-actors.md) | [Next: Attack Vectors →](03-attack-vectors.md)

---

*Unit 2 - Topic 2 of 6 | [Back to README](../README.md)*
