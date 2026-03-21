# Breached Data Sources

## Unit 6 - Topic 5 | Email Reconnaissance

---

## Overview

**Breached data sources** contain credentials, personal information, and other sensitive data from past security incidents. Analyzing breach data reveals valid credentials, password patterns, and email addresses that can be used in credential stuffing, password spraying, and social engineering attacks.

---

## 1. Types of Breached Data

```
BREACH DATA CATEGORIES:

CREDENTIAL DUMPS:
├── Email + Password combinations
├── Email + Password hash
├── Username + Password
└── Often from: database compromises, phishing campaigns

PERSONAL INFORMATION:
├── Full names, addresses, phone numbers
├── Date of birth, SSN
├── Security questions and answers
└── From: social media breaches, data brokers

CORPORATE DATA:
├── Internal emails and documents
├── VPN credentials
├── API keys and tokens
└── From: targeted attacks, insider threats

COMBO LISTS:
├── Aggregated from multiple breaches
├── Millions to billions of entries
├── Sorted/deduplicated for attackers
└── Used for automated credential stuffing
```

---

## 2. Breach Checking Tools

```bash
# === HAVE I BEEN PWNED (HIBP) ===
# haveibeenpwned.com — search by email
# Shows: which breaches an email appeared in
# API: https://haveibeenpwned.com/API/v3

# Example API query:
curl -H "hibp-api-key: YOUR_KEY" \
  "https://haveibeenpwned.com/api/v3/breachedaccount/user@target.com"
# Returns: breach names, dates, data types

# === DEHASHED ===
# dehashed.com — search by email, username, IP, name
# Returns actual credentials from breaches
# Paid service with API access

# === INTELLIGENCE X ===
# intelx.io — search across breaches, darknet, paste sites
# Comprehensive OSINT search engine

# === SNUSBASE ===
# snusbase.com — breach data search
# Search by: email, username, name, hash, password, IP

# === LEAKCHECK ===
# leakcheck.io — breach credential checking

# === BREACH DIRECTORY ===
# breachdirectory.org — free breach search
```

---

## 3. Password Analysis from Breaches

```
BREACH PASSWORD INTELLIGENCE:

COMMON PATTERNS FOR TARGET ORG:
├── Company name + year:  Target2023!
├── Season + year:        Summer2023!
├── City + numbers:       NewYork123
├── Keyboard patterns:    Qwerty123!
├── Default patterns:     Welcome1!
└── Reused passwords:     Same across services

ANALYSIS WORKFLOW:
1. Find breached credentials for target emails
2. Analyze password patterns
3. Build custom password list based on patterns
4. Use for password spraying attacks

EXAMPLE:
├── john.smith@target.com : Target2022!
├── jane.doe@target.com   : TargetCorp1!
├── bob.j@target.com      : Target@2023
└── PATTERN: "Target" + year/symbol combinations
    → Generate: Target2024!, Target@2024, TargetCorp2024!
```

---

## 4. Credential Stuffing vs Password Spraying

| Technique | Approach | Risk |
|-----------|----------|------|
| **Credential Stuffing** | Try known email:password pairs | Low lockout risk |
| **Password Spraying** | One password across many accounts | Must be slow to avoid lockout |
| **Dictionary Attack** | Many passwords per account | High lockout risk |

```bash
# === CREDENTIAL STUFFING (Breach Data) ===
# If breach shows: john@target.com:MyP@ss123
# Try same password on:
# ├── VPN portal
# ├── Email login (OWA, Gmail)
# ├── Corporate SSO
# └── Cloud services (AWS, Azure)

# People reuse passwords across services!

# === PASSWORD SPRAY (Pattern-Based) ===
# From breach analysis: pattern = CompanyYear!
# Spray:
# ├── Target2024!
# ├── Target2023!
# ├── Welcome2024!
# ├── Spring2024!
# └── ... across all known email addresses
```

---

## 5. Ethical and Legal Considerations

```
⚠️ IMPORTANT LEGAL AND ETHICAL BOUNDARIES:

LEGAL TO DO:
├── Check if target emails appear in HIBP
├── Report breach exposure to the client
├── Use findings to demonstrate risk
├── Recommend password policy improvements
└── Use within authorized penetration test scope

ILLEGAL / UNETHICAL:
├── Downloading full breach databases
├── Using breached data outside scope
├── Sharing breached data with others
├── Accessing personal data without authorization
└── Using data for personal gain

REPORTING:
├── "X employees found in Y breaches"
├── "Password patterns suggest Z weakness"
├── Do NOT include actual passwords in reports
├── Recommend: password rotation, MFA, monitoring
└── Recommend: credential monitoring service
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **HIBP** | Check if emails appear in breaches |
| **DeHashed** | Search for actual breached credentials |
| **Password patterns** | Analyze for custom wordlist generation |
| **Credential stuffing** | Reuse breached passwords on other services |
| **Password spraying** | Pattern-based single password across accounts |
| **Ethics** | Stay within authorized scope |

---

## Quick Revision Questions

1. **What is Have I Been Pwned and how does it help?**
2. **What is the difference between credential stuffing and password spraying?**
3. **How do you analyze breach data for password patterns?**
4. **What are the ethical boundaries when using breach data?**
5. **Why is password reuse a significant security risk?**

---

[← Previous: SPF, DKIM, and DMARC Analysis](04-spf-dkim-dmarc-analysis.md)

---

*Unit 6 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Social Engineering Reconnaissance →](../07-Social-Engineering-Reconnaissance/01-employee-enumeration.md)*
