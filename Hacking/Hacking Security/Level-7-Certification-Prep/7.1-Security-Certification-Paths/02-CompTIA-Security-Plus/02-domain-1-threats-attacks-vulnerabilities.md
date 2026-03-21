# Unit 2: CompTIA Security+ — Topic 2: Domain 1 — Threats, Attacks, and Vulnerabilities

## Overview

**Domain 2 (SY0-701): Threats, Vulnerabilities, and Mitigations** is the second-heaviest domain at 22%. It covers threat actors, attack vectors, social engineering, application and network vulnerabilities, and indicators of compromise. This domain tests your ability to identify and understand the threat landscape.

---

## 1. Threat Actors and Motivations

```
THREAT ACTOR TYPES:

  Actor Type        | Sophistication | Motivation
  Nation-state      | Very High      | Espionage, warfare
  APT               | Very High      | Long-term objectives
  Organized crime   | High           | Financial gain
  Hacktivists       | Medium         | Political/social cause
  Insider threats   | Varies         | Revenge, financial
  Script kiddies    | Low            | Notoriety, fun
  Shadow IT         | Low            | Convenience

THREAT ACTOR ATTRIBUTES:
  → Internal vs External
  → Level of sophistication
  → Resources/funding available
  → Intent/motivation
  → Attack surface knowledge

ATTACK VECTORS:
  → Email (phishing, malware)
  → Web (drive-by, watering hole)
  → Removable media (USB drops)
  → Supply chain (compromised vendor)
  → Cloud services (misconfiguration)
  → Social media (reconnaissance)
  → Physical (tailgating, theft)
  → Wireless (evil twin, deauth)
```

---

## 2. Social Engineering Attacks

```
SOCIAL ENGINEERING:

  Type              | Description            | Example
  Phishing          | Email-based deception  | Fake bank email
  Spear phishing    | Targeted phishing      | CEO impersonation
  Whaling           | Executive-targeted     | Board member
  Vishing           | Voice phishing         | IRS phone scam
  Smishing          | SMS phishing           | Package delivery
  Pretexting        | Fabricated scenario    | IT support call
  Baiting           | Tempting offer         | Free USB drive
  Tailgating        | Physical following     | Enter behind badge
  Piggybacking      | Authorized following   | "Hold the door"
  Watering hole     | Compromise trusted site| Industry forum
  Typosquatting     | Similar domain names   | gogle.com
  Impersonation     | Pretend to be someone  | Fake vendor

EXAM KEY POINTS:
  → Know the difference between each type
  → Phishing = email, Vishing = voice, Smishing = SMS
  → Spear phishing = targeted, Whaling = executives
  → Social engineering uses psychological manipulation
  → Mitigations: training, verification, policies
```

---

## 3. Application Vulnerabilities

```
APPLICATION ATTACKS:

  Attack              | Description
  SQL injection       | Inject SQL queries into input
  XSS (Cross-site)    | Inject scripts into web pages
  CSRF                | Force authenticated user actions
  SSRF                | Server-side request forgery
  Directory traversal | Access unauthorized files
  Buffer overflow     | Exceed memory buffer bounds
  Race condition      | Exploit timing vulnerability
  API attacks         | Abuse API endpoints
  Privilege escalation| Gain higher access level
  Replay attacks      | Reuse captured credentials

INJECTION TYPES (exam favorite):
  → SQL injection: ' OR 1=1 --
  → LDAP injection: Manipulate directory queries
  → XML injection: Modify XML data
  → Command injection: Execute OS commands
  → XSS: Inject JavaScript into pages

SECURE CODING CONCEPTS:
  → Input validation
  → Output encoding
  → Parameterized queries
  → Least privilege
  → Error handling
  → Code signing
  → OWASP guidelines
```

---

## 4. Network-Based Attacks

```
NETWORK ATTACKS:

  Attack              | Layer  | Description
  ARP poisoning       | L2     | Redirect traffic
  MAC flooding        | L2     | Overflow switch table
  VLAN hopping        | L2     | Access other VLANs
  DNS poisoning       | L7     | Redirect DNS queries
  DNS amplification   | L7     | DDoS via DNS
  Man-in-the-middle   | L3-L7  | Intercept traffic
  On-path attack      | L3-L7  | Same as MITM (new term)
  DDoS                | All    | Overwhelm resources
  Deauthentication    | Wireless| Disconnect wireless
  Evil twin           | Wireless| Fake access point
  Rogue AP            | Wireless| Unauthorized AP
  Bluejacking         | BT     | Unsolicited messages
  Bluesnarfing        | BT     | Steal BT data

DENIAL OF SERVICE:
  → Volumetric: Flood bandwidth
  → Protocol: Exploit protocol weakness
  → Application: Exhaust app resources
  → Distributed (DDoS): Multiple sources
  → Amplification: DNS, NTP, Memcached

WIRELESS ATTACKS:
  → Evil twin: Fake AP mimicking real one
  → Deauthentication: Force disconnect
  → WPS attacks: PIN brute force
  → Jamming: Signal interference
  → Rogue AP: Unauthorized access point
```

---

## 5. Indicators and Mitigations

```
INDICATORS OF MALICIOUS ACTIVITY:

  Indicator                | Meaning
  Account lockouts         | Brute force attempt
  Concurrent sessions      | Account compromise
  Impossible travel        | Credential theft
  Resource consumption     | Cryptomining/DDoS
  Blocked content          | Malware activity
  Missing logs             | Anti-forensics
  Out-of-cycle logging     | Attacker activity
  Published/documented     | Known vulnerability

MITIGATION TECHNIQUES:
  Attack          | Mitigation
  Phishing        | Email filtering, training, MFA
  Brute force     | Account lockout, MFA, CAPTCHA
  SQL injection   | Parameterized queries, WAF
  XSS             | Input validation, CSP
  MitM            | Encryption (TLS), certificate pinning
  DDoS            | CDN, rate limiting, scrubbing
  Password spray  | MFA, smart lockout
  Malware         | EDR, AV, application whitelisting
  Insider threat  | DLP, access reviews, monitoring

EXAM TIPS FOR DOMAIN 2:
  [ ] Know all social engineering types
  [ ] Understand injection attack types
  [ ] Know wireless attack types
  [ ] Memorize DDoS types
  [ ] Understand mitigation for each attack
  [ ] Know indicator types
```

---

## Summary Table

| Topic Area | Key Concepts | Exam Weight |
|-----------|-------------|-------------|
| Threat Actors | Types, motivations, attributes | Medium |
| Social Engineering | Phishing types, pretexting | High |
| Application Attacks | Injection, XSS, CSRF | High |
| Network Attacks | DDoS, MitM, wireless | Medium |
| Indicators | IoC types, malicious activity | Medium |

---

## Revision Questions

1. What distinguishes spear phishing from regular phishing?
2. What is the difference between SQL injection and XSS?
3. How do volumetric and application-layer DDoS attacks differ?
4. What mitigations address brute force attacks?
5. What indicators suggest account compromise?

---

*Previous: [01-exam-objectives-overview.md](01-exam-objectives-overview.md) | Next: [03-domain-2-technologies-and-tools.md](03-domain-2-technologies-and-tools.md)*

---

*[Back to README](../README.md)*
