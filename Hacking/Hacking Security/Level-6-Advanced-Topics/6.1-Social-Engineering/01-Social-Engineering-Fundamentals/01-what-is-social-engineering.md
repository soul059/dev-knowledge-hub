# Unit 1: Social Engineering Fundamentals — Topic 1: What is Social Engineering?

## Overview

**Social engineering** is the psychological manipulation of people into performing actions or divulging confidential information. Unlike technical hacking that exploits software vulnerabilities, social engineering exploits the human element — trust, helpfulness, fear, curiosity, and urgency. It is consistently the most effective initial attack vector in real-world breaches.

---

## 1. Defining Social Engineering

```
SOCIAL ENGINEERING:

"The art of manipulating people so they give up
 confidential information or perform actions that
 compromise security."

KEY CHARACTERISTICS:
  → Targets PEOPLE, not technology
  → Exploits human psychology and behavior
  → Can be conducted remotely or in person
  → Often the first step in larger attacks
  → Bypasses even the strongest technical controls
  → Success rate: 60-90% in professional assessments

THE HUMAN VULNERABILITY:
┌──────────────────────────────────────────┐
│     TECHNICAL CONTROLS                   │
│  ┌────────────────────────────────────┐  │
│  │  Firewalls, IDS, Encryption,      │  │
│  │  MFA, Patching, Monitoring...     │  │
│  │                                    │  │
│  │     ┌──────────────────────┐      │  │
│  │     │    THE HUMAN         │      │  │
│  │     │    ELEMENT           │      │  │
│  │     │                      │      │  │
│  │     │  Trust  │ Fear       │      │  │
│  │     │  Greed  │ Curiosity  │      │  │
│  │     │  Urgency│ Helpfulness│      │  │
│  │     │                      │      │  │
│  │     │  ← WEAKEST LINK →   │      │  │
│  │     └──────────────────────┘      │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

---

## 2. Categories of Social Engineering

```
SOCIAL ENGINEERING CATEGORIES:

1. HUMAN-BASED (In-person):
   → Impersonation (pretending to be someone)
   → Tailgating (following through secure doors)
   → Shoulder surfing (watching screens/keyboards)
   → Dumpster diving (searching trash)
   → Eavesdropping (listening to conversations)

2. COMPUTER-BASED (Digital):
   → Phishing (fraudulent emails)
   → Spear phishing (targeted phishing)
   → Watering hole (compromising trusted websites)
   → Pop-up windows (fake alerts)
   → Malicious attachments

3. MOBILE-BASED:
   → Smishing (SMS phishing)
   → Vishing (voice phishing)
   → QR code attacks
   → Malicious apps
   → SIM swapping

4. SOCIAL MEDIA-BASED:
   → Fake profiles / catfishing
   → Targeted friend requests
   → Information harvesting
   → Fake job offers
   → Influence campaigns
```

---

## 3. Social Engineering vs. Technical Attacks

```
COMPARISON:

┌─────────────────┬──────────────────┬──────────────────┐
│ Aspect          │ Social Eng.      │ Technical Attack │
├─────────────────┼──────────────────┼──────────────────┤
│ Target          │ People           │ Systems/software │
│ Exploits        │ Trust, emotions  │ Code bugs        │
│ Tools           │ Psychology       │ Exploits, code   │
│ Cost            │ Low              │ Varies           │
│ Success Rate    │ Very High (60%+) │ Varies           │
│ Detection       │ Very Difficult   │ IDS/IPS possible │
│ Patch Available │ No (human nature)│ Yes (software)   │
│ Speed           │ Minutes-days     │ Seconds-hours    │
│ Skill Level     │ Communication    │ Technical        │
│ Persistence     │ Relies on target │ Can automate     │
│ Legal Evidence  │ Hard to prove    │ Logs available   │
└─────────────────┴──────────────────┴──────────────────┘
```

---

## 4. Real-World Impact

```
NOTABLE SOCIAL ENGINEERING ATTACKS:

TWITTER HACK (2020):
  → Vishing attack on Twitter employees
  → Gained access to internal admin tools
  → Compromised high-profile accounts (Obama, Musk, Gates)
  → Used for cryptocurrency scam
  → $120,000 stolen in hours

SONY PICTURES (2014):
  → Spear phishing emails to employees
  → Credential theft → network access
  → Massive data breach: movies, emails, PII
  → Estimated $100M+ in damages

RSA SECURID (2011):
  → Email: "2011 Recruitment Plan.xls"
  → Sent to small group of employees
  → Excel file with zero-day exploit
  → Stole RSA SecurID seed data
  → Compromised 40,000+ defense contractors

UBIQUITI NETWORKS (2015):
  → Business Email Compromise (BEC)
  → Impersonated executives via email
  → Finance team wired $46.7 million
  → To overseas accounts controlled by attackers

STATS:
  → 98% of cyber attacks involve social engineering
  → Average cost of social engineering attack: $130,000
  → 43% of IT pros targeted by social engineering yearly
  → Phishing is #1 cause of data breaches
```

---

## Summary Table

| Category | Examples | Primary Target |
|----------|----------|---------------|
| Human-based | Impersonation, tailgating | Physical access |
| Computer-based | Phishing, malware | Digital access |
| Mobile-based | Vishing, smishing | Credentials |
| Social media | Fake profiles, harvesting | Information |

---

## Revision Questions

1. What is social engineering and how does it differ from technical hacking?
2. What are the four main categories of social engineering?
3. Why is social engineering so effective against organizations?
4. Name three notable social engineering attacks and their impact.
5. Why can't social engineering be "patched" like software vulnerabilities?

---

*Previous: None (First topic in this unit) | Next: [02-psychology-of-manipulation.md](02-psychology-of-manipulation.md)*

---

*[Back to README](../README.md)*
