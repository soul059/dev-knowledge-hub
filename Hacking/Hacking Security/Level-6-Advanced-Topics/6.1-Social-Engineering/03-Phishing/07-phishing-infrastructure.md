# Unit 3: Phishing — Topic 7: Phishing Infrastructure

## Overview

**Phishing infrastructure** refers to the technical components required to execute phishing campaigns — from domain registration and email servers to credential harvesting pages and tracking systems. Understanding this infrastructure is critical for both offensive security professionals conducting authorized assessments and defenders detecting and blocking phishing attacks.

---

## 1. Phishing Infrastructure Components

```
INFRASTRUCTURE OVERVIEW:

┌──────────────────────────────────────────────────┐
│              PHISHING INFRASTRUCTURE              │
│                                                   │
│  ┌────────────┐  ┌─────────────┐  ┌───────────┐ │
│  │  DOMAINS   │  │   EMAIL     │  │  LANDING  │ │
│  │            │  │   SERVER    │  │   PAGES   │ │
│  │ Lookalike  │  │             │  │           │ │
│  │ Typosquat  │  │ SMTP relay  │  │ Cloned    │ │
│  │ Aged       │  │ Mail agent  │  │ Credential│ │
│  │ Categorized│  │ SPF/DKIM   │  │ Harvester │ │
│  └────────────┘  └─────────────┘  └───────────┘ │
│                                                   │
│  ┌────────────┐  ┌─────────────┐  ┌───────────┐ │
│  │  PAYLOADS  │  │  TRACKING   │  │  C2 /     │ │
│  │            │  │             │  │  EXFIL    │ │
│  │ Macro docs │  │ Open/click  │  │           │ │
│  │ Executables│  │ tracking    │  │ Data      │ │
│  │ HTML files │  │ Unique IDs  │  │ collection│ │
│  │ ISO/IMG    │  │ Analytics   │  │ Storage   │ │
│  └────────────┘  └─────────────┘  └───────────┘ │
└──────────────────────────────────────────────────┘
```

---

## 2. Domain Setup

```
DOMAIN STRATEGIES:

LOOKALIKE DOMAINS:
  Target: microsoft.com
  → micr0soft.com (zero for 'o')
  → microsoftt.com (extra letter)
  → microsoft-security.com (added word)
  → microsft.com (missing letter)
  → rnicrosoft.com (rn looks like m)

HOMOGRAPH ATTACKS (IDN):
  → Use Unicode characters that look identical
  → аpple.com (Cyrillic 'а' vs Latin 'a')
  → Punycode: xn--pple-43d.com
  → Most modern browsers now show punycode

DOMAIN AGING:
  → Fresh domains get flagged by filters
  → Register domains weeks/months in advance
  → Build reputation with benign traffic
  → Categorize domain (business, technology)
  → Age > 30 days significantly reduces detection

EXPIRED DOMAIN ACQUISITION:
  → Purchase recently expired domains
  → Inherits existing reputation
  → May still have valid SPF/DKIM records
  → Existing backlinks add legitimacy

DOMAIN REGISTRATION OPSEC:
  → Use privacy protection (WHOIS guard)
  → Different registrar than target
  → Separate from personal domains
  → Pay with cryptocurrency/prepaid cards (threat actors)
```

---

## 3. Email Infrastructure

```
EMAIL SENDING SETUP:

SMTP SERVER OPTIONS:
  → Postfix/Sendmail on VPS
  → Microsoft 365 (legitimate account)
  → Google Workspace
  → Amazon SES
  → SendGrid, Mailgun

EMAIL AUTHENTICATION:

SPF RECORD:
  → Authorize sending IPs
  → v=spf1 ip4:x.x.x.x -all
  → Prevents SPF failures

DKIM SIGNING:
  → Cryptographic email signing
  → Proves email came from domain
  → Generates key pair, publishes public key in DNS

DMARC POLICY:
  → v=DMARC1; p=none; rua=mailto:reports@domain
  → Aligns SPF and DKIM
  → Start with p=none, move to p=reject

COMPLETE DNS SETUP EXAMPLE:
  ; A Record
  @    A     x.x.x.x
  mail A     x.x.x.x
  
  ; MX Record
  @    MX    10 mail.phish-domain.com
  
  ; SPF
  @    TXT   "v=spf1 ip4:x.x.x.x -all"
  
  ; DKIM
  s1._domainkey  TXT  "v=DKIM1; k=rsa; p=..."
  
  ; DMARC
  _dmarc  TXT  "v=DMARC1; p=quarantine"

DELIVERABILITY TIPS:
  → Warm up sending IP gradually
  → Send test emails first
  → Check spam score (mail-tester.com)
  → Avoid spam trigger words
  → Proper HTML formatting
  → Include unsubscribe link
  → Text/HTML multipart messages
```

---

## 4. Landing Pages / Credential Harvesting

```
PHISHING PAGE SETUP:

CLONING TOOLS:
  → wget --mirror for site cloning
  → HTTrack website copier
  → GoPhish built-in cloner
  → Browser "Save As" for single pages
  → Custom HTML modifications

CREDENTIAL CAPTURE:
  → Form POST to attacker server
  → Log credentials to file/database
  → Redirect to real site after capture
  → Transparent proxy (evilginx2) for session tokens

EVASION TECHNIQUES:
  → JavaScript-based cloaking
  → Check user-agent (block bots/scanners)
  → IP-based access control
  → Time-based availability
  → CAPTCHA before phishing page
  → Redirect crawlers to legitimate site

SSL/TLS:
  → Let's Encrypt free certificates
  → "HTTPS = secure" misconception
  → Green padlock increases trust
  → Wildcard certs for subdomains
```

---

## 5. Phishing Frameworks

```
POPULAR FRAMEWORKS:

GOPHISH (Open Source):
  Features:
    → Web-based management interface
    → Email template editor
    → Landing page cloner
    → Campaign tracking & analytics
    → User/group management
    → Detailed reporting
    → API for automation

KING PHISHER (Open Source):
  Features:
    → Campaign management
    → Jinja2 email templates
    → Web cloning
    → Two-factor authentication
    → Credential harvesting
    → Plugin architecture

SOCIAL ENGINEERING TOOLKIT (SET):
  Features:
    → Multiple attack vectors
    → Website cloner
    → Mass mailer
    → Credential harvester
    → PowerShell attack vectors
    → Integration with Metasploit

EVILGINX2 (MitM Framework):
  Features:
    → Reverse proxy phishing
    → Captures session cookies
    → Bypasses 2FA
    → Real-time credential relay
    → Phishlet templates

COMPARISON:
  Tool        | Complexity | 2FA Bypass | Reporting
  GoPhish     | Low        | No         | Excellent
  King Phisher| Medium     | No         | Good
  SET         | Medium     | No         | Basic
  Evilginx2   | High       | Yes        | Basic
```

---

## 6. Tracking and Analytics

```
CAMPAIGN METRICS:

TRACKING METHODS:
  → Unique URLs per target
  → Tracking pixels (1x1 image)
  → Click tracking via redirectors
  → Form submission logging
  → Browser fingerprinting
  → Geolocation via IP

KEY METRICS:
  → Emails sent / delivered / bounced
  → Open rate (tracking pixel loaded)
  → Click rate (link clicked)
  → Submission rate (credentials entered)
  → Report rate (reported to security team)
  → Time to click (how quickly users click)

REPORTING DASHBOARD (GoPhish Example):
  Campaign: Q1 Security Assessment
  ┌──────────────────────────────────────┐
  │ Sent:        500    (100%)           │
  │ Opened:      312    (62.4%)          │
  │ Clicked:     147    (29.4%)          │
  │ Submitted:    89    (17.8%)          │
  │ Reported:     43    (8.6%)           │
  │ No Action:   188    (37.6%)          │
  └──────────────────────────────────────┘
```

---

## Summary Table

| Component | Purpose | Tools/Examples |
|-----------|---------|---------------|
| Domains | Lookalike URLs | Registrars, IDN, typosquatting |
| Email Server | Send phishing emails | Postfix, GoPhish, SES |
| Authentication | Pass email checks | SPF, DKIM, DMARC |
| Landing Pages | Capture credentials | GoPhish, Evilginx2 |
| Payloads | Deliver malware | Macro docs, HTML, executables |
| Tracking | Monitor campaign | Pixels, unique URLs, analytics |

---

## Revision Questions

1. What are the key components of phishing infrastructure?
2. How do attackers set up lookalike domains and avoid detection?
3. Why are SPF, DKIM, and DMARC important for phishing email delivery?
4. What phishing frameworks are available and how do they compare?
5. How does Evilginx2 bypass two-factor authentication?
6. What metrics are tracked during a phishing campaign?

---

*Previous: [06-smishing.md](06-smishing.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
