# What is Cybersecurity?

## Unit 1 - Topic 1 | Introduction to Cybersecurity

---

## Overview

Cybersecurity is the practice of protecting systems, networks, programs, and data from digital attacks, unauthorized access, damage, or theft. It encompasses technologies, processes, and practices designed to safeguard the digital world we depend on every day.

---

## 1. Defining Cybersecurity

### The Core Definition

> **Cybersecurity** (also written as **cyber security**) is the body of technologies, processes, and practices designed to protect networks, devices, programs, and data from attack, damage, or unauthorized access.

### Breaking It Down

```
┌─────────────────────────────────────────────────────────────────┐
│                    CYBERSECURITY UMBRELLA                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  Information  │  │   Network    │  │   Application        │  │
│  │  Security     │  │   Security   │  │   Security           │  │
│  │  (InfoSec)    │  │              │  │                      │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  Operational  │  │   Cloud      │  │   Endpoint           │  │
│  │  Security     │  │   Security   │  │   Security           │  │
│  │  (OpSec)      │  │              │  │                      │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  Disaster     │  │   Identity   │  │   IoT                │  │
│  │  Recovery     │  │   Security   │  │   Security           │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Cybersecurity vs Information Security

| Aspect | Cybersecurity | Information Security |
|--------|--------------|---------------------|
| **Scope** | Digital/cyber threats only | All forms of information (digital + physical) |
| **Focus** | Protecting systems from cyber attacks | Protecting data confidentiality, integrity, availability |
| **Medium** | Cyberspace | All media (paper, digital, verbal) |
| **Includes** | Network, app, cloud, endpoint security | Data governance, classification, physical security |
| **Relationship** | Subset of InfoSec | Broader discipline |

---

## 2. Why Cybersecurity Matters

### The Digital Dependency

Modern society is deeply dependent on digital systems:

```
┌─────────────────────────────────────────────────────┐
│              DIGITAL DEPENDENCY MAP                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  🏦 Banking & Finance ──── Online transactions      │
│  🏥 Healthcare ──────────── Patient records (EHR)   │
│  🏭 Critical Infrastructure── Power grids, water    │
│  🏛️ Government ──────────── National security       │
│  🛒 E-Commerce ──────────── Online shopping         │
│  📱 Personal ────────────── Social media, email     │
│  🎓 Education ───────────── Online learning         │
│  🚗 Transportation ──────── Autonomous vehicles     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### The Cost of Cybercrime

- **Global cybercrime cost**: Estimated $10.5 trillion annually by 2025
- **Average data breach cost**: $4.45 million (IBM, 2023)
- **Ransomware attack frequency**: Every 11 seconds
- **Identity theft victims**: Millions per year globally

### Real-World Impact Examples

| Incident | Year | Impact |
|----------|------|--------|
| WannaCry Ransomware | 2017 | 200,000+ computers in 150 countries; hospitals, banks affected |
| Equifax Breach | 2017 | 147 million people's personal data exposed |
| SolarWinds Supply Chain | 2020 | US government agencies and Fortune 500 companies compromised |
| Colonial Pipeline | 2021 | Fuel shortage across US East Coast; $4.4M ransom paid |
| MOVEit Transfer | 2023 | 2,500+ organizations, 67+ million individuals affected |

---

## 3. Domains of Cybersecurity

### The Major Domains

```
                    ┌─────────────────────┐
                    │   CYBERSECURITY     │
                    │     DOMAINS         │
                    └────────┬────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   OFFENSIVE     │ │   DEFENSIVE     │ │   GOVERNANCE    │
│   (Red Team)    │ │   (Blue Team)   │ │   Risk & Comp.  │
├─────────────────┤ ├─────────────────┤ ├─────────────────┤
│ • Pen Testing   │ │ • SOC Analysis  │ │ • Risk Mgmt     │
│ • Vuln Research │ │ • IR (Incident  │ │ • Compliance     │
│ • Exploit Dev   │ │   Response)     │ │ • Audit          │
│ • Social Eng.   │ │ • Threat Hunt   │ │ • Policy Dev     │
│ • Red Teaming   │ │ • Forensics     │ │ • Privacy        │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### Domain Descriptions

**1. Network Security**
- Protecting the network infrastructure from unauthorized access, misuse, or theft
- Technologies: Firewalls, IDS/IPS, VPNs, Network segmentation

**2. Application Security**
- Finding, fixing, and preventing security vulnerabilities in software applications
- Practices: Secure coding, code review, SAST/DAST, penetration testing

**3. Cloud Security**
- Securing data, applications, and infrastructure in cloud environments
- Concepts: Shared responsibility, cloud-native security tools, CASB

**4. Endpoint Security**
- Protecting devices (laptops, phones, servers) that connect to the network
- Tools: EDR, antivirus, device management, patch management

**5. Identity & Access Management (IAM)**
- Ensuring the right people have the right access at the right time
- Components: SSO, MFA, RBAC, Privileged Access Management

**6. Operational Security (OpSec)**
- Protecting day-to-day operations and processes
- Includes: Change management, incident management, business continuity

---

## 4. The Cybersecurity Mindset

### Think Like an Attacker, Defend Like a Guardian

```
┌───────────────────────────────────────────────────────────┐
│                  CYBERSECURITY MINDSET                     │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  🔴 Attacker Mindset          🔵 Defender Mindset         │
│  ─────────────────            ─────────────────           │
│  • What can I break?          • What needs protecting?    │
│  • Where's the weak link?     • What if this fails?       │
│  • How do I stay hidden?      • How do I detect this?     │
│  • What's the reward?         • What's the impact?        │
│  • Can I bypass this?         • Is this truly secure?     │
│                                                           │
│  🟣 Security Professional = BOTH mindsets combined        │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Core Assumptions in Cybersecurity

1. **Assume breach**: Operate as if an attacker is already inside
2. **No system is 100% secure**: Security is about reducing risk, not eliminating it
3. **The human factor**: People are often the weakest link
4. **Defense in depth**: Multiple layers of security are essential
5. **Continuous improvement**: The threat landscape constantly evolves

---

## 5. Key Terminology

| Term | Definition |
|------|-----------|
| **Asset** | Anything of value to an organization (data, hardware, people) |
| **Threat** | Potential cause of an unwanted incident |
| **Vulnerability** | Weakness that can be exploited by a threat |
| **Exploit** | Code or technique that takes advantage of a vulnerability |
| **Risk** | Likelihood of a threat exploiting a vulnerability × Impact |
| **Attack Surface** | Total sum of vulnerabilities accessible to an attacker |
| **Payload** | The component of an attack that performs the malicious action |
| **Zero-Day** | Vulnerability unknown to the vendor (no patch available) |
| **Indicator of Compromise (IoC)** | Evidence that a security breach has occurred |
| **Threat Actor** | Individual or group that performs cyberattacks |

### The Vulnerability-Threat-Risk Relationship

```
┌──────────────┐         ┌──────────────┐
│   THREAT     │         │ VULNERABILITY │
│  (Attacker)  │────────►│  (Weakness)   │
└──────────────┘         └──────┬───────┘
                                │
                         Exploitation
                                │
                                ▼
                        ┌──────────────┐
                        │    RISK      │
                        │  (Impact)    │
                        └──────┬───────┘
                               │
                        Mitigation
                               │
                               ▼
                        ┌──────────────┐
                        │   CONTROL    │
                        │ (Safeguard)  │
                        └──────────────┘

    RISK = Threat × Vulnerability × Impact
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Definition | Protecting systems, networks, and data from cyber threats |
| Scope | Network, application, cloud, endpoint, identity, operational security |
| Why it matters | $10.5T cybercrime cost; critical infrastructure dependency |
| Mindset | Think like attacker + defend like guardian |
| Key formula | Risk = Threat × Vulnerability × Impact |
| Core assumption | No system is 100% secure — reduce risk continuously |

---

## Quick Revision Questions

1. **What is the difference between cybersecurity and information security?**
2. **Name 5 domains of cybersecurity and briefly describe each.**
3. **What is the relationship between threat, vulnerability, and risk?**
4. **Why is the "assume breach" mentality important in modern cybersecurity?**
5. **List 3 real-world cyber incidents and their impact.**
6. **What does "attack surface" mean, and why should it be minimized?**

---

[Next: CIA Triad →](02-cia-triad.md)

---

*Unit 1 - Topic 1 of 6 | [Back to README](../README.md)*
