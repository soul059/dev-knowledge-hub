# Common Attack Types Overview

## Unit 2 - Topic 5 | Threat Landscape

---

## Overview

This topic provides a comprehensive overview of the most common attack types in cybersecurity. Understanding these attacks is essential for both offensive security professionals (who simulate them) and defensive professionals (who detect and prevent them).

---

## 1. Attack Classification

```
┌──────────────────────────────────────────────────────────────────┐
│                    ATTACK CLASSIFICATION                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BY TARGET:              BY METHOD:            BY IMPACT:        │
│  ┌──────────┐           ┌──────────┐          ┌──────────┐      │
│  │ Network  │           │ Passive  │          │ Disruptive│      │
│  │ System   │           │ Active   │          │ Destructive│     │
│  │ App      │           │ Close-in │          │ Exfiltrate│      │
│  │ Human    │           │ Insider  │          │ Financial │      │
│  │ Physical │           │ Remote   │          │ Reputational│    │
│  └──────────┘           └──────────┘          └──────────┘      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Major Attack Types

### Malware Attacks

| Malware Type | Behavior | Spread Method |
|-------------|----------|---------------|
| **Virus** | Attaches to legitimate files, replicates | User opens infected file |
| **Worm** | Self-replicating, no user interaction needed | Network propagation |
| **Trojan** | Disguised as legitimate software | Social engineering, downloads |
| **Ransomware** | Encrypts data, demands ransom payment | Phishing, exploits |
| **Spyware** | Monitors user activity secretly | Bundled software, exploits |
| **Rootkit** | Hides deep in OS, evades detection | Exploits, insider |
| **Keylogger** | Records keystrokes | Malware payload, physical device |
| **Adware** | Displays unwanted advertisements | Software bundles |
| **Botnet** | Network of compromised machines (zombies) | Malware infection |
| **Wiper** | Destroys data permanently | Nation-state attacks |

### Social Engineering Attacks

```
┌──────────────────────────────────────────────────────────────┐
│              SOCIAL ENGINEERING ATTACKS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PHISHING          Fake emails to steal credentials/install  │
│  ├─ Spear Phishing   Targeted at specific individuals       │
│  ├─ Whaling          Targeted at executives                  │
│  ├─ Vishing          Voice phishing (phone calls)            │
│  └─ Smishing         SMS phishing (text messages)            │
│                                                              │
│  PRETEXTING        Creating a fake scenario to gain trust    │
│  BAITING           Leaving malicious USB drives              │
│  TAILGATING        Following authorized person into building │
│  QUID PRO QUO      Offering something in exchange for info  │
│  WATERING HOLE     Compromising websites target visits       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Network Attacks

| Attack | Description | Impact |
|--------|-------------|--------|
| **DDoS** | Overwhelm target with traffic from multiple sources | Service unavailability |
| **Man-in-the-Middle** | Intercept communications between two parties | Data theft, manipulation |
| **ARP Spoofing** | Forge ARP messages to intercept traffic | Traffic interception |
| **DNS Spoofing** | Redirect DNS queries to malicious servers | Phishing, data theft |
| **Port Scanning** | Probe for open ports and services | Reconnaissance |
| **SYN Flood** | Exhaust server resources with half-open connections | DoS |
| **Packet Sniffing** | Capture network traffic passively | Data interception |

### Web Application Attacks

```
┌──────────────────────────────────────────────────────────────┐
│                WEB APPLICATION ATTACKS                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  INJECTION ATTACKS:                                          │
│  ├─ SQL Injection ──── Manipulate database queries           │
│  ├─ XSS ──────────── Inject scripts into web pages          │
│  ├─ Command Injection ── Execute OS commands                 │
│  └─ LDAP Injection ──── Manipulate directory queries         │
│                                                              │
│  AUTH/ACCESS ATTACKS:                                        │
│  ├─ Brute Force ──────── Try all password combinations       │
│  ├─ Credential Stuffing ── Use breached credentials          │
│  ├─ Session Hijacking ── Steal session tokens                │
│  └─ IDOR ─────────────── Access other users' resources       │
│                                                              │
│  OTHER WEB ATTACKS:                                          │
│  ├─ CSRF ─────────────── Force user to perform actions       │
│  ├─ SSRF ─────────────── Server-side request forgery         │
│  ├─ XXE ──────────────── XML external entity injection       │
│  └─ File Upload ──────── Upload malicious files              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Password Attacks

| Attack | Description | Speed | Countermeasure |
|--------|-------------|-------|----------------|
| **Brute Force** | Try every possible combination | Slow | Account lockout, complexity |
| **Dictionary** | Try common words/passwords | Medium | Strong unique passwords |
| **Rainbow Table** | Pre-computed hash lookup | Fast | Salted hashes |
| **Password Spraying** | Few passwords across many accounts | Slow (stealthy) | Lockout + monitoring |
| **Credential Stuffing** | Use breached credentials | Fast | MFA, breach monitoring |
| **Keylogging** | Record keystrokes | N/A | Anti-malware, virtual keyboards |

### Denial of Service (DoS/DDoS)

```
┌──────────────────────────────────────────────────────────────┐
│                    DDoS ATTACK                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────┐                                                  │
│  │Attacker│                                                  │
│  └───┬────┘                                                  │
│      │ Commands botnet                                       │
│      ▼                                                       │
│  ┌──────────────────────────────┐                            │
│  │        BOTNET                │                            │
│  │  ┌───┐ ┌───┐ ┌───┐ ┌───┐   │                            │
│  │  │Bot│ │Bot│ │Bot│ │Bot│   │   Thousands of              │
│  │  └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘   │   compromised              │
│  │    │     │     │     │     │   machines                   │
│  └────┼─────┼─────┼─────┼─────┘                            │
│       │     │     │     │                                    │
│       └─────┼─────┼─────┘                                    │
│             │     │                                          │
│             ▼     ▼                                          │
│       ┌─────────────────┐                                    │
│       │   TARGET SERVER │  Overwhelmed with traffic          │
│       │   (OFFLINE! ❌)  │  Legitimate users can't access    │
│       └─────────────────┘                                    │
│                                                              │
│  Types: Volumetric, Protocol, Application Layer              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Attack Types by CIA Impact

| Attack | Confidentiality | Integrity | Availability |
|--------|:-:|:-:|:-:|
| **Data Breach** | ❌ | - | - |
| **Ransomware** | ❌ | ❌ | ❌ |
| **DDoS** | - | - | ❌ |
| **MITM** | ❌ | ❌ | - |
| **SQL Injection** | ❌ | ❌ | ❌ |
| **Defacement** | - | ❌ | - |
| **Wiper Malware** | - | ❌ | ❌ |
| **Phishing** | ❌ | - | - |
| **Keylogger** | ❌ | - | - |

---

## 4. Attack Kill Chain Mapping

```
┌───────────────┬───────────────────────────────────────────┐
│ Kill Chain     │ Common Attack Types                       │
│ Phase          │                                           │
├───────────────┼───────────────────────────────────────────┤
│ Reconnaissance │ OSINT, port scanning, social media recon  │
│ Weaponization  │ Exploit creation, malware development     │
│ Delivery       │ Phishing, USB drop, watering hole         │
│ Exploitation   │ Buffer overflow, SQL injection, RCE       │
│ Installation   │ Trojan, backdoor, rootkit                 │
│ C2 (Command)   │ Botnet C2, reverse shell, DNS tunneling   │
│ Actions        │ Data exfiltration, ransomware, wiper      │
└───────────────┴───────────────────────────────────────────┘
```

---

## Summary Table

| Category | Key Attacks | Primary Defense |
|----------|------------|-----------------|
| **Malware** | Ransomware, trojan, worm | AV/EDR, patching, email security |
| **Social Engineering** | Phishing, pretexting | Awareness training, email filtering |
| **Network** | DDoS, MITM, ARP spoofing | IDS/IPS, encryption, DDoS protection |
| **Web App** | SQLi, XSS, CSRF | WAF, secure coding, input validation |
| **Password** | Brute force, credential stuffing | MFA, lockout policies, password managers |
| **Supply Chain** | Compromised updates | SCA, vendor assessment, code signing |

---

## Quick Revision Questions

1. **List 5 types of malware and explain how each spreads.**
2. **What is the difference between DoS and DDoS?**
3. **How does a Man-in-the-Middle attack work?**
4. **Name 3 types of injection attacks in web applications.**
5. **Which attacks primarily impact Availability in the CIA Triad?**
6. **Map a ransomware attack to the Cyber Kill Chain phases.**

---

[← Previous: Threat Intelligence](04-threat-intelligence.md) | [Next: Advanced Persistent Threats →](06-advanced-persistent-threats.md)

---

*Unit 2 - Topic 5 of 6 | [Back to README](../README.md)*
