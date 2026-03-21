# Unit 9: Advanced Techniques — Topic 1: Business Email Compromise (BEC)

## Overview

**Business Email Compromise (BEC)** is a sophisticated social engineering attack where threat actors impersonate or compromise business email accounts to manipulate employees into transferring funds, sharing sensitive data, or performing other harmful actions. BEC is the most financially devastating form of cybercrime, with the FBI reporting over $2.7 billion in losses annually in the US alone.

---

## 1. BEC Attack Categories

```
FBI-DEFINED BEC TYPES:

TYPE 1: CEO FRAUD
  → Impersonate CEO/executive
  → Request urgent wire transfer
  → Target: Finance/AP department
  → "Confidential acquisition, process immediately"
  → Average loss: $130,000

TYPE 2: ACCOUNT COMPROMISE
  → Compromise actual employee email
  → Send invoices to contacts from real account
  → All email authentication passes
  → Very difficult to detect
  → Target: Vendor relationships

TYPE 3: FALSE INVOICE SCHEME
  → Impersonate vendor/supplier
  → Send fake invoice or updated payment details
  → "Our banking information has changed"
  → Target: AP departments with vendor relationships
  → Often targets long-term vendor relationships

TYPE 4: ATTORNEY IMPERSONATION
  → Impersonate legal counsel
  → Leverage urgency of legal matters
  → "Confidential matter — handle personally"
  → Target: Executives, finance
  → Effective due to legal authority

TYPE 5: DATA THEFT
  → Request W-2 forms, employee data
  → Impersonate HR executive or CEO
  → Target: HR and payroll departments
  → "Send all employee W-2s for audit"
  → Identity theft of all employees
```

---

## 2. BEC Attack Lifecycle

```
BEC ATTACK PHASES:

PHASE 1: TARGET IDENTIFICATION
  → Research company structure
  → Identify CEO, CFO, AP contacts
  → Map vendor/supplier relationships
  → Study communication patterns
  → Monitor for travel/absence

PHASE 2: ACCOUNT COMPROMISE (If applicable)
  → Phish executive credentials
  → Access email account
  → Study communication style
  → Set up inbox rules to hide activity
  → Monitor conversations for opportunities

  INBOX RULES CREATED BY ATTACKERS:
  → Rule: If from [CEO] → move to hidden folder
  → Rule: If contains "wire" → forward to attacker
  → Rule: If from [bank] → mark read + hide
  → Rule: If "suspicious" → auto-delete

PHASE 3: RECONNAISSANCE (From inside)
  → Read email threads for context
  → Identify pending transactions
  → Learn internal processes
  → Understand approval workflows
  → Find the right moment to strike

PHASE 4: EXECUTION
  → Send the fraudulent request
  → Time it for maximum success:
    - When executive is traveling
    - End of business day
    - Before holiday/weekend
    - During busy period
  → Create urgency and confidentiality

PHASE 5: MONEY MOVEMENT
  → Wire transfer to mule account
  → Quickly moved to second account
  → Converted to cryptocurrency
  → Transferred overseas
  → Extremely difficult to recover after 48 hours
```

---

## 3. BEC Indicators

```
RED FLAGS:

EMAIL INDICATORS:
  → Reply-to differs from From address
  → Slight domain variation
  → Unusual sending time (3 AM from CEO)
  → Grammar/style different from normal
  → Requests bypassing normal procedures
  → "Keep this confidential"
  → First-time vendor payment request

BEHAVIORAL INDICATORS:
  → Urgency that prevents verification
  → Request for secrecy
  → Change in payment details
  → Pressure to act immediately
  → Executive unavailable to verify
  → Request to bypass approval process
  → Unfamiliar bank account details

TECHNICAL INDICATORS:
  → Email header anomalies
  → New inbox forwarding rules
  → Login from unusual locations
  → Mail rules auto-deleting responses
  → OAuth app grants to unknown apps
  → VPN from unusual geography
```

---

## 4. Real-World BEC Cases

```
NOTABLE BEC INCIDENTS:

FACEBOOK & GOOGLE ($100M, 2013-2015):
  → Lithuanian attacker impersonated Quanta Computer
  → Sent fake invoices for hardware
  → Both companies paid over $100M combined
  → Attacker: Evaldas Rimasauskas
  → Sentenced to 5 years in prison

UBIQUITI NETWORKS ($46.7M, 2015):
  → Employee impersonation via email
  → Wire transfers to overseas accounts
  → Discovered during routine audit
  → Recovered $14.9M
  → Stock price dropped 47%

TOYOTA BOSHOKU ($37M, 2019):
  → European subsidiary targeted
  → Attacker impersonated business partner
  → Changed payment instructions
  → Single wire transfer of $37M
  → BEC targeting supply chain

PUERTO RICO GOVERNMENT ($2.6M, 2020):
  → Targeted government pension fund
  → Changed bank account for payments
  → Three separate wire transfers
  → Discovered only after investigation
  → Government employee impersonation
```

---

## 5. Defending Against BEC

```
DEFENSE STRATEGIES:

FINANCIAL CONTROLS:
  → Dual authorization for all wire transfers
  → Verbal confirmation on known phone numbers
  → 24-48 hour hold for new payees
  → Separation of duties
  → Daily reconciliation of accounts
  → Limits on wire transfer amounts
  → Pre-approved vendor list

TECHNICAL CONTROLS:
  → DMARC enforcement (p=reject)
  → Anti-impersonation rules
  → Inbox rule monitoring
  → OAuth app audit and restrictions
  → Conditional access policies
  → Suspicious login detection
  → Email forwarding restrictions
  → Advanced BEC detection tools

PROCESS CONTROLS:
  → Vendor payment change verification
  → Out-of-band verification for all changes
  → Regular vendor information audits
  → Employee absence procedures
  → Documented approval workflows
  → Regular financial audits

AWARENESS:
  → Executive-specific BEC training
  → Finance team targeted training
  → Real-world case study presentations
  → Simulated BEC exercises
  → Vendor communication guidelines
```

---

## Summary Table

| BEC Type | Target | Method | Average Loss |
|----------|--------|--------|-------------|
| CEO fraud | Finance/AP | Impersonate executive | $130K |
| Account compromise | Vendor contacts | Compromised email | Variable |
| False invoice | AP department | Fake vendor invoice | $50K-500K |
| Attorney impersonation | Executives | Legal pretext | $100K+ |
| Data theft | HR/Payroll | Request employee data | Varies |

---

## Revision Questions

1. What are the five FBI-defined types of BEC attacks?
2. How do attackers use compromised email accounts to facilitate BEC?
3. What red flags indicate a potential BEC attack?
4. What financial controls specifically prevent BEC wire fraud?
5. Why is BEC the most financially damaging form of cybercrime?

---

*Previous: None (First topic in this unit) | Next: [02-deep-fake-concerns.md](02-deep-fake-concerns.md)*

---

*[Back to README](../README.md)*
