# Authorization Forms

## Unit 4 - Topic 3 | Pre-engagement

---

## Overview

**Authorization forms** provide the **legal permission** to conduct penetration testing. Without them, pen testing is indistinguishable from criminal hacking. These forms must be signed by someone with **legal authority** over the systems being tested and should clearly document the scope and boundaries.

---

## 1. Types of Authorization Documents

| Document | Purpose | Who Signs |
|----------|---------|-----------|
| **Permission to Test** | Legal authorization to conduct the test | Client executive (CEO, CISO, CTO) |
| **Scope Agreement** | Defines what will be tested | Both parties |
| **NDA** | Confidentiality obligations | Both parties |
| **Indemnification** | Liability protection | Both parties |
| **Third-party auth** | Permission for shared/hosted systems | Third-party provider |

---

## 2. Permission to Test Template

```
PENETRATION TESTING AUTHORIZATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ORGANIZATION: _________________________
ADDRESS: _____________________________

I, _________________, in my capacity as 
_________________ (title), hereby authorize 
_________________ (testing firm/individual) 
to perform penetration testing against the 
following systems and networks:

IN-SCOPE TARGETS:
• _________________________________
• _________________________________
• _________________________________

TESTING PERIOD:
From: _____________ To: _____________

TESTING TYPE:
☐ External   ☐ Internal   ☐ Web Application
☐ Wireless   ☐ Social Engineering
☐ Physical   ☐ Other: _______________

TESTING APPROACH:
☐ Black Box  ☐ Gray Box  ☐ White Box

RESTRICTIONS:
• _________________________________
• _________________________________

I confirm that I have the legal authority to 
authorize this testing and that all necessary 
stakeholders have been informed.

Signature: _________________________
Name: _____________________________
Title: ____________________________
Date: _____________________________
Organization seal/stamp: ___________

WITNESS (optional):
Signature: _________________________
Name: _____________________________

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Authority Verification

```
WHO CAN AUTHORIZE TESTING?

MUST HAVE:
├── Legal authority over the target systems
├── Authority to bind the organization
└── Understanding of what they're authorizing

ACCEPTABLE SIGNERS:
├── CEO / Managing Director
├── CTO / CIO / CISO
├── VP of IT / VP of Security
├── IT Director (if delegated)
└── Authorized legal representative

NOT ACCEPTABLE:
├── Junior IT staff
├── Help desk personnel
├── Project managers (without delegation)
├── System administrators (without delegation)
└── Anyone without formal authority

VERIFICATION STEPS:
1. Confirm signer's identity and title
2. Verify their authority (corporate registry, org chart)
3. For subsidiaries: may need parent company approval
4. For hosted systems: may need provider approval
5. Keep signed original securely
```

---

## 4. Multi-Party Authorization

```
COMMON MULTI-PARTY SCENARIOS:

SCENARIO 1: Cloud-hosted application
├── Client authorization: Test the application ✅
├── Cloud provider: Follow their pen test policy ✅
│   ├── AWS: No pre-approval needed (most tests)
│   ├── Azure: No pre-approval needed
│   └── GCP: Follow Acceptable Use Policy
└── Both authorizations needed!

SCENARIO 2: Managed service provider
├── Client: Authorizes testing of their data/apps
├── MSP: Must agree to testing of managed infrastructure
└── Both must sign!

SCENARIO 3: Co-located office
├── Client: Authorizes testing of their network segment
├── Building management: Physical testing authorization
├── Other tenants: Must NOT be affected
└── Careful scope boundaries needed!
```

---

## 5. Carrying Authorization During Testing

```
DURING THE ENGAGEMENT:

FOR REMOTE TESTING:
├── Keep digital copy of signed authorization
├── Have emergency contact numbers ready
├── Have client POC available during testing hours
└── Be ready to provide authorization if ISP contacts you

FOR PHYSICAL TESTING:
├── Carry printed authorization letter ("Get Out of Jail")
├── Carry government-issued ID
├── Have client POC on speed dial
├── Carry business cards
├── Have emergency contacts memorized
└── Be prepared to explain your presence to:
    ├── Security guards
    ├── Police officers
    ├── Employees
    └── Building management

⚠️ Even with authorization, cooperate fully if 
   confronted by law enforcement
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Authorization** | Legal permission — mandatory before ANY testing |
| **Authorized signer** | Must have legal authority (CEO, CISO, CTO) |
| **Third-party** | Cloud/hosting providers may need separate auth |
| **Carry proof** | Always have authorization accessible |
| **Verify authority** | Confirm the signer has the right to authorize |

---

## Quick Revision Questions

1. **Who should sign the authorization form?**
2. **Why is a "Permission to Test" letter critical?**
3. **When do you need multi-party authorization?**
4. **What should you carry during a physical pen test?**
5. **What should you do if confronted by law enforcement?**

---

[← Previous: Rules of Engagement Document](02-rules-of-engagement-document.md) | [Next: Emergency Contacts →](04-emergency-contacts.md)

---

*Unit 4 - Topic 3 of 6 | [Back to README](../README.md)*
