# Authorization and Scope

## Unit 2 - Topic 1 | Legal and Ethical Considerations

---

## Overview

**Authorization** is the single most important legal requirement in penetration testing. Testing without proper authorization is a **criminal offense** in virtually every jurisdiction. **Scope** defines exactly what can and cannot be tested. Together, authorization and scope form the legal foundation of every engagement.

---

## 1. Why Authorization Matters

```
┌──────────────────────────────────────────────────────────────┐
│              AUTHORIZED vs UNAUTHORIZED TESTING               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  WITH AUTHORIZATION:                                         │
│  ✅ Legal penetration test                                    │
│  ✅ Protected by contract                                     │
│  ✅ Covered by insurance                                      │
│  ✅ Professional security service                             │
│                                                              │
│  WITHOUT AUTHORIZATION:                                      │
│  ❌ Computer crime (felony in most countries)                 │
│  ❌ Up to 10+ years imprisonment                              │
│  ❌ Heavy fines ($250K+)                                      │
│  ❌ Civil liability                                            │
│  ❌ Career-ending                                              │
│                                                              │
│  ⚠️ "I was testing their security" is NOT a legal defense    │
│  ⚠️ Good intentions do NOT provide authorization             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Authorization Requirements

```
AUTHORIZATION CHECKLIST:
├── ✅ Written permission (signed document)
├── ✅ Authorized by someone with legal authority
│   └── CEO, CISO, VP of IT, or authorized representative
├── ✅ Specific scope defined
│   ├── In-scope IP ranges/domains
│   ├── In-scope applications
│   └── Out-of-scope assets
├── ✅ Testing window defined
│   ├── Start date and time
│   └── End date and time
├── ✅ Testing methods approved
│   ├── Social engineering? (yes/no)
│   ├── DoS testing? (yes/no)
│   └── Physical testing? (yes/no)
├── ✅ Emergency contacts listed
└── ✅ Both parties sign the document
```

---

## 3. Defining Scope

| Scope Element | Example |
|---------------|---------|
| **In-scope IPs** | 10.0.0.0/24, 192.168.1.0/24 |
| **In-scope domains** | *.example.com, app.example.com |
| **In-scope apps** | Customer portal, API v2 |
| **Out-of-scope** | Production database, partner systems |
| **Testing hours** | Monday-Friday, 9 PM - 6 AM |
| **Allowed methods** | Port scanning, exploitation, phishing |
| **Prohibited methods** | DoS attacks, social engineering via phone |
| **Credentials provided** | Standard user: test@example.com |

---

## 4. Third-Party Considerations

```
⚠️ CRITICAL: You may NOT have authority to test:

SCENARIO: Client owns app hosted on AWS
├── Client authorizes testing of THEIR application ✅
├── Testing AWS infrastructure itself ❌ (need AWS approval)
└── Solution: Follow AWS penetration testing policy

SCENARIO: Client's website uses Cloudflare CDN
├── Testing client's website ✅
├── Testing/attacking Cloudflare ❌
└── Solution: May need to test from behind CDN

SCENARIO: Client shares office network with other tenants
├── Testing client's systems ✅
├── Testing shared network/other tenants ❌
└── Solution: Clearly define network boundaries

CLOUD PROVIDER PEN TEST POLICIES:
├── AWS: No pre-approval needed for most tests
├── Azure: No pre-approval needed
├── GCP: Follow Acceptable Use Policy
└── All: DoS testing generally prohibited
```

---

## 5. "Get Out of Jail" Letter

```
AUTHORIZATION LETTER (Key Elements):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Company Name: Acme Corporation
Authorizing Person: Jane Smith, CISO
Contact: jsmith@acme.com, +1-555-0100

This letter authorizes [Tester/Firm] to conduct
penetration testing against the following assets:

IN SCOPE:
• 203.0.113.0/24 (external network)
• *.acme.com (web applications)
• VPN gateway: vpn.acme.com

OUT OF SCOPE:
• Production database (10.0.0.50)
• Partner APIs
• Physical security

TESTING WINDOW: Jan 15 - Feb 15, 2024
TESTING HOURS: 6 PM - 6 AM EST (weekdays)

PROHIBITED ACTIVITIES:
• Denial of Service attacks
• Social engineering of employees
• Modification of production data

Signed: ________________  Date: ____________
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Authorization** | Must be written and signed by authorized person |
| **Scope** | Explicitly defines what CAN and CANNOT be tested |
| **Third parties** | Cloud providers, shared infra need separate auth |
| **Get out of jail letter** | Carry during physical pen tests |
| **No authorization = crime** | Even with good intentions |

---

## Quick Revision Questions

1. **Why is written authorization essential?**
2. **Who should sign the authorization document?**
3. **What happens if you test without authorization?**
4. **Why do cloud-hosted apps require special consideration?**
5. **What should a penetration testing scope document include?**

---

[Next: Rules of Engagement →](02-rules-of-engagement.md)

---

*Unit 2 - Topic 1 of 6 | [Back to README](../README.md)*
