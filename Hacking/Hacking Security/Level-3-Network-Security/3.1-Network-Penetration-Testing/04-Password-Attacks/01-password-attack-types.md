# Password Attack Types

## Unit 4 - Topic 1 | Password Attacks

---

## Overview

**Password attacks** attempt to discover or bypass authentication credentials. Understanding the different attack types helps select the right approach based on the target, available information, and constraints like lockout policies.

---

## 1. Password Attack Taxonomy

```
PASSWORD ATTACK CATEGORIES:

ONLINE ATTACKS (against live service):
├── Brute Force — try every possible combination
├── Dictionary — try list of common passwords
├── Password Spraying — one password, many accounts
├── Credential Stuffing — reuse breached credentials
└── Default Credentials — try factory defaults

OFFLINE ATTACKS (against captured hashes):
├── Hash Cracking — brute force against hash
├── Dictionary Attack — wordlist against hash
├── Rainbow Tables — precomputed hash lookup
├── Rule-Based — modify dictionary words
└── Hybrid — dictionary + brute force combination

OTHER TECHNIQUES:
├── Credential Capture — MitM, keylogger, phishing
├── Pass-the-Hash — reuse hash without cracking
├── Pass-the-Ticket — reuse Kerberos tickets
├── Token Theft — steal session tokens
└── Social Engineering — ask for the password
```

---

## 2. Online vs Offline Attacks

| Aspect | Online | Offline |
|--------|:------:|:------:|
| **Target** | Live service | Captured hash file |
| **Speed** | Limited by network/service | Limited by hardware |
| **Detection** | Easily detected + logged | Undetectable |
| **Lockout** | Account lockout risk | No lockout |
| **Rate** | ~10-100 attempts/sec | Millions-billions/sec |
| **Tools** | Hydra, Medusa, Burp | Hashcat, John |
| **Requirement** | Network access | Hash dump needed |

---

## 3. Attack Selection Guide

```
WHICH ATTACK TO USE?

HAVE HASH DUMP?
├── Yes → Offline cracking (Hashcat/John)
│   ├── Try common wordlists first
│   ├── Add rules for variations
│   └── Brute force as last resort
└── No → Online attacks
    ├── HAVE BREACHED CREDENTIALS?
    │   └── Yes → Credential stuffing
    ├── LOCKOUT POLICY EXISTS?
    │   ├── Yes → Password spraying (slow, safe)
    │   └── No → Dictionary attack (fast)
    ├── KNOW USERNAME?
    │   ├── Yes → Targeted dictionary/brute force
    │   └── No → Enumerate users first
    └── KNOW NOTHING?
        └── Try default credentials first!
```

---

## 4. Password Complexity vs Attack Effort

| Password | Length | Attack Time (offline) |
|----------|:------:|:--------------------:|
| `123456` | 6 | Instant |
| `password` | 8 | Instant (in wordlist) |
| `P@ssw0rd` | 8 | Minutes (common pattern) |
| `Tr0ub4dor&3` | 12 | Hours (mixed chars) |
| `correct horse battery staple` | 28 | Days (passphrase) |
| `kX$9mP2!qR7vL4` | 15 | Years (random) |

---

## 5. Authentication Protocols to Attack

| Protocol | Port | Tool | Method |
|----------|:----:|------|--------|
| **SSH** | 22 | Hydra, Medusa | Brute/Dictionary |
| **FTP** | 21 | Hydra | Brute/Dictionary |
| **HTTP(S)** | 80/443 | Hydra, Burp | Form brute force |
| **RDP** | 3389 | Hydra, Crowbar | Brute/Spray |
| **SMB** | 445 | CME, Hydra | Spray/Stuffing |
| **SMTP** | 25/587 | Hydra | Brute/Dictionary |
| **MySQL** | 3306 | Hydra | Brute/Dictionary |
| **MSSQL** | 1433 | Hydra | Brute/Dictionary |
| **WinRM** | 5985 | Evil-WinRM | Spray/Pass-Hash |
| **LDAP** | 389 | Hydra | Spray/Brute |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Online** | Against live services (slow, detectable) |
| **Offline** | Against hashes (fast, undetectable) |
| **Brute force** | Try all combinations (slow but thorough) |
| **Dictionary** | Try wordlist (fast for common passwords) |
| **Spraying** | One password, many accounts (avoids lockout) |
| **Selection** | Choose based on hash availability + lockout |

---

## Quick Revision Questions

1. **What is the difference between online and offline password attacks?**
2. **When should you use password spraying instead of brute force?**
3. **Why are offline attacks faster than online attacks?**
4. **What is credential stuffing?**
5. **What should you always try first before complex attacks?**

---

[Next: Brute Force Attacks →](02-brute-force-attacks.md)

---

*Unit 4 - Topic 1 of 7 | [Back to README](../README.md)*
