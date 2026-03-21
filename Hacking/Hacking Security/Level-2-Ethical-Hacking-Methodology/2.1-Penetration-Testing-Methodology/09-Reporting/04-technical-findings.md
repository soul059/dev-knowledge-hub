# Technical Findings

## Unit 9 - Topic 4 | Reporting

---

## Overview

**Technical findings** form the core of the penetration test report. Each finding is a detailed, structured writeup of a discovered vulnerability — including proof of exploitation, step-by-step reproduction instructions, and specific remediation guidance for the technical team.

---

## 1. Finding Structure

```
STANDARD FINDING FORMAT:

┌──────────────────────────────────────────────────────────────┐
│  FINDING ID:     F-001                                       │
│  TITLE:          SQL Injection — Login Page                   │
│  SEVERITY:       Critical (CVSS 9.8)                         │
│  STATUS:         Exploited                                    │
│  AFFECTED:       https://app.target.com/login                │
├──────────────────────────────────────────────────────────────┤
│  DESCRIPTION:    What the vulnerability is                    │
│  IMPACT:         What damage could result                     │
│  REPRODUCTION:   Step-by-step instructions                    │
│  EVIDENCE:       Screenshots, output                          │
│  REMEDIATION:    How to fix it                                │
│  REFERENCES:     CVE, CWE, OWASP links                       │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Writing a Technical Finding

```
═══════════════════════════════════════════════
F-003: Weak SSH Configuration
═══════════════════════════════════════════════

SEVERITY: Medium (CVSS 5.3)
CWE: CWE-327 (Use of Broken Crypto Algorithm)
AFFECTED HOST: 10.0.0.5 (web-server-01)
PORT: 22/TCP (OpenSSH 8.9p1)

────────────────────────────────────────────
DESCRIPTION:
────────────────────────────────────────────
The SSH service on web-server-01 accepts deprecated
cryptographic algorithms including diffie-hellman-group1-sha1
and hmac-sha1. Additionally, root login is permitted
via SSH, increasing the risk of brute-force attacks.

────────────────────────────────────────────
IMPACT:
────────────────────────────────────────────
• Deprecated algorithms may be vulnerable to
  cryptographic attacks
• Root login allows direct brute-force of the
  highest-privilege account
• Successful attack yields immediate root access

────────────────────────────────────────────
STEPS TO REPRODUCE:
────────────────────────────────────────────
1. Run SSH audit against the target:
   $ ssh-audit 10.0.0.5

2. Observe deprecated algorithms in output:
   [WARN] diffie-hellman-group1-sha1
   [WARN] hmac-sha1

3. Verify root login is permitted:
   $ ssh root@10.0.0.5
   → Password prompt appears (not rejected)

────────────────────────────────────────────
EVIDENCE:
────────────────────────────────────────────
[Screenshot: ssh-audit output]
[Screenshot: SSH root login prompt]

────────────────────────────────────────────
REMEDIATION:
────────────────────────────────────────────
1. Disable deprecated algorithms in /etc/ssh/sshd_config:
   KexAlgorithms curve25519-sha256,diffie-hellman-group16-sha512
   MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

2. Disable root SSH login:
   PermitRootLogin no

3. Restart SSH service:
   $ sudo systemctl restart sshd

────────────────────────────────────────────
REFERENCES:
────────────────────────────────────────────
• CWE-327: https://cwe.mitre.org/data/definitions/327.html
• SSH Hardening Guide: https://ssh-audit.com/hardening_guides.html
═══════════════════════════════════════════════
```

---

## 3. Finding Status Categories

| Status | Meaning |
|--------|---------|
| **Exploited** | Vulnerability was exploited successfully |
| **Confirmed** | Verified but not exploited (could be too risky) |
| **Probable** | Strong evidence but not fully confirmed |
| **Informational** | Not a direct vulnerability but worth noting |

---

## 4. Organizing Findings

```
FINDINGS ORGANIZATION OPTIONS:

OPTION 1: BY SEVERITY (Recommended)
├── Critical Findings (F-001, F-002)
├── High Findings (F-003 to F-007)
├── Medium Findings (F-008 to F-015)
├── Low Findings (F-016 to F-018)
└── Informational (F-019 to F-022)

OPTION 2: BY HOST
├── 10.0.0.5 (web-server-01)
│   ├── F-001: SQL Injection
│   └── F-003: Weak SSH
├── 10.0.0.10 (db-server-01)
│   └── F-002: Default Credentials
└── 10.0.0.20 (file-server)
    └── F-004: SMB Vulnerability

OPTION 3: BY ATTACK PHASE
├── Reconnaissance Findings
├── Exploitation Findings
└── Post-Exploitation Findings
```

---

## 5. Writing Quality Tips

| Tip | Example |
|-----|---------|
| **Be specific** | "Port 80 on 10.0.0.5" not "the web server" |
| **Include versions** | "Apache 2.4.49" not "Apache" |
| **Number reproduction steps** | 1, 2, 3... not paragraphs |
| **Quantify impact** | "47,832 records" not "many records" |
| **Provide exact remediation** | Config lines, not "fix it" |
| **Link to references** | OWASP, CWE, vendor docs |
| **Include raw evidence** | Full command + output |

---

## Summary Table

| Element | Purpose |
|---------|---------|
| **Finding ID** | Unique reference number |
| **Severity** | CVSS score and rating |
| **Description** | What the vulnerability is |
| **Impact** | What could go wrong |
| **Reproduction** | How to verify the finding |
| **Evidence** | Proof it exists |
| **Remediation** | How to fix it |

---

## Quick Revision Questions

1. **What elements should every technical finding include?**
2. **What is the difference between "Exploited" and "Confirmed" status?**
3. **Why include step-by-step reproduction instructions?**
4. **What is the best way to organize findings?**
5. **Why include references (CWE, OWASP) in findings?**

---

[← Previous: Executive Summary](03-executive-summary.md) | [Next: Risk Ratings →](05-risk-ratings.md)

---

*Unit 9 - Topic 4 of 7 | [Back to README](../README.md)*
