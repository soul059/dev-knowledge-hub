# Unit 3: Phishing — Topic 4: Whaling

## Overview

**Whaling** is a highly targeted phishing attack aimed at senior executives (C-suite), board members, and other high-value targets within an organization. The term "whaling" refers to going after the "big fish." These attacks leverage authority, urgency, and business context to manipulate executives into authorizing wire transfers, sharing sensitive data, or clicking malicious links.

---

## 1. What Makes Whaling Unique

```
WHALING CHARACTERISTICS:

TARGET PROFILE:
  → CEO, CFO, CTO, COO, CISO
  → Board members and directors
  → VPs and senior management
  → Key decision makers
  → High-net-worth individuals

WHY EXECUTIVES ARE TARGETED:
  → Authority to approve large transactions
  → Access to sensitive strategic data
  → Less likely to follow security procedures
  → Often bypass security controls
  → Public profiles (easy OSINT)
  → High-pressure schedules = quick decisions
  → Mobile email = less scrutiny

SPEAR PHISHING vs WHALING:

SPEAR PHISHING:              WHALING:
→ Any specific person         → Senior executives only
→ Various pretexts            → Business-focused pretexts
→ Moderate impact             → Very high impact
→ Technical lures             → Legal/financial lures
→ Impersonates colleagues     → Impersonates executives or
                                legal/regulatory entities
```

---

## 2. Common Whaling Scenarios

```
WHALING PRETEXTS:

LEGAL/REGULATORY:
  → "Subpoena notification — review immediately"
  → "SEC inquiry regarding recent filing"
  → "GDPR compliance violation — urgent response"
  → "Legal action pending — review attached documents"
  → "Regulatory audit — information required"

FINANCIAL:
  → "Wire transfer required for acquisition (confidential)"
  → "Updated banking details from vendor"
  → "Tax document requires signature"
  → "Investor relations — quarterly report review"
  → "Emergency payment authorization"

BOARD/GOVERNANCE:
  → "Board meeting agenda — updated materials"
  → "Shareholder communication — review draft"
  → "Executive compensation review"
  → "Merger/acquisition documents (NDA required)"

PERSONAL:
  → "Your tax return has been flagged"
  → "Issue with your personal investment account"
  → "Travel itinerary update for upcoming trip"
  → "Country club membership renewal"
```

---

## 3. Whaling Attack Examples

```
EXAMPLE 1: CEO FRAUD

FROM: "CEO Name" <ceo@company-secure.com>
TO: CFO
SUBJECT: Confidential — Urgent Wire Transfer

Hi [CFO Name],

I need you to process a wire transfer of $450,000
to the account below. This is for a confidential 
acquisition we discussed. Please handle this 
personally and do not discuss with anyone else 
until the deal closes.

Account: [attacker's account details]
Bank: [attacker's bank]

This needs to go out today. I'm in meetings 
all afternoon so please handle ASAP.

Thanks,
[CEO Name]
Sent from my iPhone

KEY ELEMENTS:
→ Urgency ("needs to go out today")
→ Confidentiality ("do not discuss")
→ Authority (from CEO)
→ Unavailability ("in meetings")
→ Mobile signature (explains brevity)

─────────────────────────────────────────

EXAMPLE 2: LEGAL SUBPOENA

FROM: "Federal Court Clerk" <clerk@uscourts-filing.com>
TO: CEO
SUBJECT: Subpoena — Case No. 2024-CV-4521

Dear [CEO Name],

A civil complaint has been filed against [Company]
in the U.S. District Court. Please review the 
attached subpoena and respond within 48 hours.

Failure to respond may result in a default judgment
against your organization.

[View Court Documents]   ← Malicious link

Office of the Clerk
U.S. District Court

KEY ELEMENTS:
→ Legal authority (federal court)
→ Fear (lawsuit/judgment)
→ Urgency (48 hours)
→ Plausible scenario
```

---

## 4. CEO Fraud / BEC Targeting Executives

```
BUSINESS EMAIL COMPROMISE FLOW:

1. RECONNAISSANCE:
   → Research company structure
   → Identify CEO and CFO
   → Learn executive travel schedule
   → Map financial approval processes

2. IMPERSONATION:
   → Spoof or compromise CEO email
   → Time attack when CEO is traveling
   → CEO is unavailable to verify

3. THE ASK:
   → CFO receives email from "CEO"
   → Urgent wire transfer needed
   → Marked confidential
   → Must be done today

4. EXECUTION:
   → CFO processes transfer
   → Money goes to attacker's account
   → Quickly moved to offshore accounts
   → Discovered days/weeks later

REAL-WORLD LOSSES:
  → Ubiquiti Networks: $46.7 million
  → FACC Operations: $55.8 million  
  → Crelan Bank: $75.8 million
  → Facebook & Google: $100 million
  → Toyota Boshoku: $37 million
```

---

## 5. Defending Against Whaling

```
EXECUTIVE PROTECTION:

TECHNICAL:
  → VIP email filtering rules
  → Domain monitoring for lookalikes
  → External email warnings
  → Advanced threat protection
  → Executive account MFA enforcement

PROCESS:
  → Dual authorization for wire transfers > $X
  → Verbal confirmation for transfer requests
  → No financial actions via email alone
  → Vendor change verification procedures
  → Out-of-band verification (call back on known number)

TRAINING:
  → Executive-specific security briefings
  → Whaling simulation exercises
  → Personal OSINT awareness
  → Travel security protocols
  → PA/EA training (often first point of contact)

MONITORING:
  → Executive email anomaly detection
  → Wire transfer monitoring
  → Brand/domain monitoring
  → Dark web monitoring for exec data
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Target | C-suite, board members, senior executives |
| Pretext | Legal, financial, regulatory, board governance |
| Success factor | Authority + urgency + confidentiality |
| Average loss | $100K - $100M per incident |
| Key defense | Out-of-band verification + dual authorization |
| Detection | Very difficult — highly personalized |

---

## Revision Questions

1. What distinguishes whaling from standard spear phishing?
2. Why are executives particularly vulnerable to phishing?
3. What are the most common whaling pretexts?
4. How does CEO fraud/BEC typically unfold?
5. What defense measures specifically protect against whaling?
6. Name three real-world whaling attacks and their financial impact.

---

*Previous: [03-spear-phishing.md](03-spear-phishing.md) | Next: [05-vishing.md](05-vishing.md)*

---

*[Back to README](../README.md)*
