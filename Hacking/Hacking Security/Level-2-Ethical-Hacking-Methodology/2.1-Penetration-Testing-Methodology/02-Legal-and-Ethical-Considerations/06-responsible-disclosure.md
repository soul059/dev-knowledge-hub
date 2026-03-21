# Responsible Disclosure

## Unit 2 - Topic 6 | Legal and Ethical Considerations

---

## Overview

**Responsible disclosure** (also called **coordinated disclosure**) is the practice of privately reporting security vulnerabilities to the affected organization, giving them time to fix the issue before making it public. It balances **public safety** with **vendor/organization interests**.

---

## 1. Disclosure Models

```
┌──────────────────────────────────────────────────────────────┐
│              VULNERABILITY DISCLOSURE MODELS                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FULL DISCLOSURE:                                            │
│  • Vulnerability published immediately                       │
│  • No vendor notification                                    │
│  • ⚠️ Controversial — enables attacks before fix            │
│  • Rationale: Forces vendors to act quickly                  │
│                                                              │
│  RESPONSIBLE/COORDINATED DISCLOSURE:                         │
│  • Vendor notified privately first                           │
│  • Agreed timeline for fix (typically 90 days)              │
│  • Public disclosure after fix or deadline                   │
│  • ✅ Industry standard practice                             │
│                                                              │
│  NO DISCLOSURE:                                              │
│  • Vulnerability kept secret                                 │
│  • Sold on dark web or used privately                        │
│  • ❌ Unethical and potentially illegal                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Coordinated Disclosure Process

```
STEP-BY-STEP:

1. DISCOVER vulnerability
   └── Document fully (steps to reproduce, impact)

2. REPORT to vendor/organization
   ├── Use security@company.com
   ├── Check for security.txt (/.well-known/security.txt)
   ├── Use bug bounty platform if available
   └── Encrypt communication (PGP/GPG)

3. WAIT for acknowledgment
   └── Follow up if no response in 7 days

4. COLLABORATE on fix
   ├── Provide additional details if needed
   ├── Test proposed patches
   └── Agree on disclosure timeline

5. VENDOR releases fix
   └── Typically within 90 days

6. PUBLIC DISCLOSURE
   ├── After fix is available
   ├── CVE assigned if applicable
   └── Credit given to researcher

TIMELINE:
Day 0          Day 7         Day 90        Day 90+
│──────────────│──────────────│──────────────│
Report         Follow-up     Deadline       Public
to vendor      if no reply   (fix or not)   disclosure
```

---

## 3. Industry Disclosure Policies

| Organization | Disclosure Timeline |
|-------------|:---:|
| **Google Project Zero** | 90 days (strict) |
| **Microsoft** | Coordinated, no fixed deadline |
| **CERT/CC** | 45 days |
| **Zero Day Initiative (ZDI)** | 120 days |
| **HackerOne** | Per program policy |
| **Most bug bounties** | 90 days standard |

---

## 4. Finding Disclosure Contacts

```bash
# 1. Check security.txt
curl https://example.com/.well-known/security.txt
# Standard format (RFC 9116):
# Contact: security@example.com
# Encryption: https://example.com/pgp-key.txt
# Policy: https://example.com/security-policy
# Preferred-Languages: en

# 2. Check bug bounty platforms
# HackerOne: hackerone.com/company
# Bugcrowd: bugcrowd.com/company

# 3. Common security email addresses
# security@company.com
# abuse@company.com
# cert@company.com

# 4. CERT/CC can help coordinate
# https://www.kb.cert.org/vuls/report/
```

---

## 5. Writing a Disclosure Report

```
VULNERABILITY REPORT TEMPLATE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Subject: Security Vulnerability in [Product/Service]

SUMMARY:
A [vulnerability type] exists in [product] that 
allows [attacker action] resulting in [impact].

AFFECTED VERSION(S):
[Product name] version X.Y.Z

STEPS TO REPRODUCE:
1. Navigate to...
2. Enter...
3. Observe...

IMPACT:
[What an attacker can achieve]
CVSS Score: [Estimated score]

PROOF OF CONCEPT:
[Minimal PoC — enough to prove, not weaponize]
[Screenshots or sanitized output]

SUGGESTED FIX:
[Remediation recommendations]

RESEARCHER:
[Your name/handle]
[Contact information]
[PGP key fingerprint]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 6. Ethical Considerations

```
DO:
✅ Report privately first
✅ Give reasonable time to fix (90 days minimum)
✅ Provide clear reproduction steps
✅ Suggest remediation
✅ Be patient and professional
✅ Follow up if no response

DO NOT:
❌ Publicly disclose before giving vendor time to fix
❌ Demand payment in exchange for not disclosing
❌ Access more data than needed to prove the vulnerability
❌ Use the vulnerability for personal gain
❌ Share with others before vendor fixes it
❌ Threaten the vendor
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Coordinated disclosure** | Industry standard — notify, wait, disclose |
| **90-day timeline** | Standard deadline for vendor to fix |
| **security.txt** | Standard file for disclosure contact info |
| **CVE** | Common Vulnerabilities and Exposures ID |
| **Ethics** | Report privately, be patient, be professional |

---

## Quick Revision Questions

1. **What is the difference between full and coordinated disclosure?**
2. **What is the standard disclosure timeline?**
3. **How do you find a company's security contact?**
4. **What should a vulnerability report include?**
5. **Is it ethical to demand payment for not disclosing a vulnerability?**

---

[← Previous: Liability Considerations](05-liability-considerations.md)

---

*Unit 2 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Penetration Testing Standards →](../03-Penetration-Testing-Standards/01-ptes.md)*
