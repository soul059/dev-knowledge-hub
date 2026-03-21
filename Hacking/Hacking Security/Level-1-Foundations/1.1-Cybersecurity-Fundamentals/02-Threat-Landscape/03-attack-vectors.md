# Attack Vectors

## Unit 2 - Topic 3 | Threat Landscape

---

## Overview

An **attack vector** is the path or method an attacker uses to gain unauthorized access to a system, network, or data. Understanding attack vectors is critical for both offensive testing and defensive security — you can't protect what you don't know can be attacked.

---

## 1. Attack Vector Categories

```
┌──────────────────────────────────────────────────────────────────────┐
│                       ATTACK VECTORS                                 │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐      │
│  │  HUMAN-BASED VECTORS                                       │      │
│  │  • Phishing (email, voice, SMS)                            │      │
│  │  • Social engineering                                      │      │
│  │  • Physical access / tailgating                            │      │
│  │  • Insider threats                                         │      │
│  └────────────────────────────────────────────────────────────┘      │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐      │
│  │  NETWORK-BASED VECTORS                                     │      │
│  │  • Open ports and services                                 │      │
│  │  • Man-in-the-middle attacks                               │      │
│  │  • DNS attacks                                             │      │
│  │  • Wireless network attacks                                │      │
│  └────────────────────────────────────────────────────────────┘      │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐      │
│  │  SOFTWARE-BASED VECTORS                                    │      │
│  │  • Unpatched vulnerabilities                               │      │
│  │  • Web application flaws (SQLi, XSS)                       │      │
│  │  • Malware (trojan, ransomware)                            │      │
│  │  • Supply chain attacks                                    │      │
│  └────────────────────────────────────────────────────────────┘      │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐      │
│  │  PHYSICAL VECTORS                                          │      │
│  │  • USB drops (malicious drives)                            │      │
│  │  • Physical device theft                                   │      │
│  │  • Rogue hardware implants                                 │      │
│  │  • Dumpster diving                                         │      │
│  └────────────────────────────────────────────────────────────┘      │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 2. Top Attack Vectors Explained

### Email / Phishing

```
┌──────────────────────────────────────────────────────────────┐
│                    PHISHING ATTACK FLOW                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Attacker                                     Victim         │
│     │                                            │           │
│     │ 1. Craft phishing email                    │           │
│     │    (spoofed sender, urgent language)        │           │
│     │─────────────────────────────────────────►  │           │
│     │                                            │           │
│     │                    2. Victim clicks link    │           │
│     │                       or opens attachment   │           │
│     │                                            │           │
│     │                    3. Malware installed     │           │
│     │◄────────────────── OR credentials stolen   │           │
│     │                                            │           │
│     │ 4. Attacker gains access                   │           │
│     │                                            │           │
│  STATS: 91% of cyberattacks start with phishing             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Unpatched Vulnerabilities

| Vulnerability Type | Example | Risk |
|-------------------|---------|------|
| **Zero-Day** | Unknown to vendor, no patch | Critical |
| **N-Day** | Known, patch available but not applied | High |
| **Legacy** | End-of-life software, no patches coming | High |
| **Misconfiguration** | Default settings, open services | Medium-High |

### Web Application Attacks

| Vector | Description | Impact |
|--------|-------------|--------|
| **SQL Injection** | Injecting SQL commands via input fields | Data theft, auth bypass |
| **XSS** | Injecting scripts into web pages | Session hijacking, phishing |
| **CSRF** | Forcing user to perform unwanted actions | Unauthorized transactions |
| **File Upload** | Uploading malicious files to server | Remote code execution |
| **Path Traversal** | Accessing files outside web root | File disclosure |
| **SSRF** | Server makes requests to internal resources | Internal network access |

### Supply Chain Attacks

```
┌──────────────────────────────────────────────────────────────┐
│                SUPPLY CHAIN ATTACK FLOW                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Attacker                                                    │
│     │                                                        │
│     │ 1. Compromise software vendor / open-source library    │
│     │                                                        │
│     ▼                                                        │
│  ┌──────────────┐                                            │
│  │ Vendor's     │ 2. Inject malicious code into update       │
│  │ Build System │                                            │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼  3. Distribute via legitimate update channel       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Customer A  │  │  Customer B  │  │  Customer C  │      │
│  │  (Infected)  │  │  (Infected)  │  │  (Infected)  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                              │
│  Examples: SolarWinds (2020), Codecov (2021), 3CX (2023)     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Attack Vector by Target

| Target | Common Vectors | Defenses |
|--------|---------------|----------|
| **Users** | Phishing, social engineering, credential theft | Training, MFA, email filtering |
| **Network** | Open ports, MITM, wireless attacks | Firewalls, IDS/IPS, segmentation |
| **Web Apps** | SQLi, XSS, auth flaws | WAF, input validation, secure coding |
| **Endpoints** | Malware, USB drops, unpatched software | EDR, patch management, device control |
| **Cloud** | Misconfig, IAM flaws, exposed storage | CSPM, least privilege, encryption |
| **Supply Chain** | Compromised updates, malicious packages | SCA, code signing, vendor assessment |
| **Physical** | Tailgating, theft, rogue devices | Access controls, cameras, visitor policies |

---

## 4. Attack Surface Concept

```
┌──────────────────────────────────────────────────────────────┐
│                    ATTACK SURFACE                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Attack Surface = Sum of ALL possible attack vectors         │
│                                                              │
│  ┌─────────────────────────────────────────────────┐        │
│  │  DIGITAL ATTACK SURFACE                          │        │
│  │  • Open ports and services                       │        │
│  │  • Web applications and APIs                     │        │
│  │  • Email systems                                 │        │
│  │  • Cloud services and storage                    │        │
│  │  • VPN endpoints                                 │        │
│  │  • Third-party integrations                      │        │
│  └─────────────────────────────────────────────────┘        │
│                                                              │
│  ┌─────────────────────────────────────────────────┐        │
│  │  PHYSICAL ATTACK SURFACE                         │        │
│  │  • Office locations and entrances                │        │
│  │  • USB ports on endpoints                        │        │
│  │  • Server rooms and data centers                 │        │
│  │  • Printer and IoT devices                       │        │
│  └─────────────────────────────────────────────────┘        │
│                                                              │
│  ┌─────────────────────────────────────────────────┐        │
│  │  HUMAN ATTACK SURFACE                            │        │
│  │  • Employees susceptible to phishing             │        │
│  │  • Contractors with access                       │        │
│  │  • Executives (whaling targets)                  │        │
│  │  • Help desk (social engineering target)         │        │
│  └─────────────────────────────────────────────────┘        │
│                                                              │
│  GOAL: Minimize attack surface as much as possible           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Vector Category | Top Vectors | Primary Defense |
|----------------|------------|-----------------|
| **Human** | Phishing, social engineering | Awareness training, MFA |
| **Network** | Open ports, MITM, wireless | Firewalls, encryption, segmentation |
| **Software** | Unpatched vulns, web app flaws | Patch management, WAF, secure coding |
| **Physical** | USB drops, theft, tailgating | Physical access controls, device management |
| **Supply Chain** | Compromised updates, libraries | SCA, vendor assessment, code signing |

---

## Quick Revision Questions

1. **What is an attack vector? Give 5 examples.**
2. **Why is phishing considered the most effective attack vector?**
3. **Explain how a supply chain attack works with a real-world example.**
4. **What are the three categories of attack surface?**
5. **How do you reduce the attack surface of a web application?**
6. **What makes zero-day vulnerabilities more dangerous than N-day vulnerabilities?**

---

[← Previous: Attack Motivations](02-attack-motivations.md) | [Next: Threat Intelligence →](04-threat-intelligence.md)

---

*Unit 2 - Topic 3 of 6 | [Back to README](../README.md)*
