# Unit 3: Phishing — Topic 1: Phishing Types

## Overview

**Phishing** is the most common social engineering attack vector, accounting for over 90% of initial compromise in cyber attacks. Phishing uses fraudulent communications — primarily email — to trick recipients into clicking links, opening attachments, or providing sensitive information. This topic covers all major phishing categories and their characteristics.

---

## 1. Phishing Taxonomy

```
PHISHING TYPES:

┌─────────────────────────────────────────────────┐
│                    PHISHING                      │
│                                                  │
│  ┌───────────────────────────────────────────┐  │
│  │ MASS PHISHING (Spray & Pray)              │  │
│  │ → Generic emails to thousands             │  │
│  │ → Low effort, low success per email       │  │
│  │ → Volume compensates for low rate         │  │
│  └───────────────────────────────────────────┘  │
│                                                  │
│  ┌───────────────────────────────────────────┐  │
│  │ SPEAR PHISHING (Targeted)                 │  │
│  │ → Personalized to specific individual     │  │
│  │ → High effort, high success rate          │  │
│  │ → Uses OSINT for personalization          │  │
│  └───────────────────────────────────────────┘  │
│                                                  │
│  ┌───────────────────────────────────────────┐  │
│  │ WHALING (Executive Targeting)             │  │
│  │ → Targets C-suite executives              │  │
│  │ → Very high effort, very high impact      │  │
│  │ → Business-focused pretexts               │  │
│  └───────────────────────────────────────────┘  │
│                                                  │
│  ┌───────────────────────────────────────────┐  │
│  │ VISHING (Voice Phishing)                  │  │
│  │ → Phone-based attacks                     │  │
│  │ → Real-time social engineering            │  │
│  │ → Caller ID spoofing                      │  │
│  └───────────────────────────────────────────┘  │
│                                                  │
│  ┌───────────────────────────────────────────┐  │
│  │ SMISHING (SMS Phishing)                   │  │
│  │ → Text message based                      │  │
│  │ → Short URLs, urgency                     │  │
│  │ → High open rate (98% SMS read)           │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

---

## 2. Additional Phishing Variants

```
SPECIALIZED PHISHING TYPES:

CLONE PHISHING:
  → Copy a legitimate email previously received
  → Replace links/attachments with malicious ones
  → "Re-sending due to updated attachment"
  → Very convincing (looks like real conversation)

PHARMING:
  → DNS poisoning to redirect to fake site
  → No email needed — user types correct URL
  → DNS server or hosts file compromised
  → Difficult for users to detect

ANGLER PHISHING:
  → Attacker monitors social media for complaints
  → Responds as fake customer support
  → "Sorry about your issue! Click here to resolve"
  → Targets frustrated customers

SEARCH ENGINE PHISHING:
  → SEO poisoning to rank fake sites
  → User searches for legitimate service
  → Clicks top result — attacker's page
  → Especially effective for login pages

WATERING HOLE:
  → Compromise website target visits regularly
  → Industry forum, supplier portal, news site
  → Drive-by download or credential harvest
  → Targets specific group/organization

BUSINESS EMAIL COMPROMISE (BEC):
  → Compromise or spoof executive email
  → Request wire transfer or sensitive data
  → Most financially damaging phishing type
  → Average loss: $125,000+ per incident

CONSENT PHISHING (OAuth):
  → Malicious app requests OAuth permissions
  → User grants access to email, files, contacts
  → No credentials stolen — access granted!
  → Hard to detect, persistent access
```

---

## 3. Phishing Statistics

```
PHISHING BY THE NUMBERS:

PREVALENCE:
  → 3.4 billion phishing emails sent daily
  → 36% of breaches involve phishing (Verizon DBIR)
  → Average organization receives 1,295 phishing
    emails per year per employee

SUCCESS RATES:
  → Mass phishing: 3-5% click rate
  → Spear phishing: 50-65% click rate
  → Vishing: 30-40% success rate

FINANCIAL IMPACT:
  → BEC losses: $2.7 billion/year (FBI IC3)
  → Average cost per phishing attack: $4.76M
  → Ransomware via phishing: additional millions

DETECTION TIME:
  → Average time to identify phishing: 213 days
  → Average time to contain: 80 days
  → 60% of phishing sites exist < 24 hours
```

---

## Summary Table

| Type | Target | Effort | Success Rate | Impact |
|------|--------|--------|-------------|--------|
| Mass phishing | Everyone | Low | 3-5% | Low per target |
| Spear phishing | Individuals | High | 50-65% | High |
| Whaling | Executives | Very High | 40-50% | Very High |
| Vishing | Anyone | Medium | 30-40% | High |
| Smishing | Mobile users | Low | 10-20% | Medium |
| BEC | Finance teams | High | Variable | Very High |
| Clone phishing | Prior recipients | Medium | 60%+ | High |

---

## Revision Questions

1. What are the main types of phishing and how do they differ?
2. What makes spear phishing more effective than mass phishing?
3. What is Business Email Compromise and why is it so damaging?
4. How does consent phishing (OAuth) differ from traditional phishing?
5. What statistics highlight the severity of the phishing problem?

---

*Previous: None (First topic in this unit) | Next: [02-email-phishing.md](02-email-phishing.md)*

---

*[Back to README](../README.md)*
