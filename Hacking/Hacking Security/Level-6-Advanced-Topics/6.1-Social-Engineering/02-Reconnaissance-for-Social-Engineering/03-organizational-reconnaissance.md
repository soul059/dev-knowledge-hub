# Unit 2: Reconnaissance for Social Engineering — Topic 3: Organizational Reconnaissance

## Overview

**Organizational reconnaissance** focuses on understanding the target company's structure, processes, culture, and security controls. This intelligence shapes every aspect of a social engineering campaign — from choosing targets to crafting pretexts to timing attacks for maximum effectiveness.

---

## 1. Organizational Structure Mapping

```
STRUCTURE INTELLIGENCE:

EXECUTIVE LEVEL:
  → CEO, CFO, CTO, CISO, COO
  → Board members and advisors
  → Decision-making authority
  → Public communication style

MANAGEMENT LEVEL:
  → Department heads
  → Project/team managers
  → Who reports to whom
  → Cross-department relationships

OPERATIONAL LEVEL:
  → Help desk / IT support staff
  → Administrative assistants
  → Receptionists / front desk
  → Security guards
  → Facilities management

SOURCES:
  LinkedIn → org chart reconstruction
  Company website → leadership page
  SEC filings → executive compensation, structure
  Glassdoor → internal culture, management style
  Annual reports → strategic direction

MAPPING TOOLS:
  → Maltego (visual relationship mapping)
  → Draw.io (manual org chart creation)
  → LinkedIn Sales Navigator (advanced search)
  → Excel/spreadsheet for target tracking
```

---

## 2. Process and Culture Intelligence

```
PROCESS INTELLIGENCE:

FINANCIAL PROCESSES:
  → How are payments authorized?
  → Who can approve wire transfers?
  → What's the threshold for dual approval?
  → How are vendors onboarded?
  → What accounting software is used?
  → SE APPLICATION: BEC, invoice fraud

IT PROCESSES:
  → How is the help desk contacted?
  → What's the password reset process?
  → Is MFA deployed? How?
  → What ticketing system is used?
  → How are new accounts provisioned?
  → SE APPLICATION: IT impersonation

PHYSICAL SECURITY:
  → Badge access system (RFID? Magnetic?)
  → Visitor sign-in process
  → Delivery procedures
  → After-hours access
  → Camera coverage
  → SE APPLICATION: Physical infiltration

CULTURE INDICATORS:
  → Dress code (formal, business casual, casual)
  → Communication tone (formal emails or Slack casual)
  → Work hours and schedule
  → Remote vs. in-office ratio
  → Employee satisfaction (Glassdoor reviews)
  → Response to authority figures
  → Openness to external contacts
  → SE APPLICATION: Pretext matching
```

---

## 3. Technology Stack Discovery

```
DISCOVERING WHAT TECHNOLOGIES THEY USE:

FROM JOB POSTINGS:
  "Experience with Salesforce, Jira, and Okta required"
  → Know their CRM, project management, and SSO

  "Must know Active Directory, Office 365, Azure"
  → Microsoft ecosystem, cloud migration

  "Python, Django, PostgreSQL, AWS"
  → Development stack and cloud provider

FROM WEBSITE:
  → Check headers, cookies, JavaScript files
  → Identify CMS (WordPress, Drupal)
  → Find analytics tools (Google Analytics ID)
  → Identify CDN (CloudFlare, Akamai)

FROM DNS/EMAIL:
  → MX records → email provider
  → SPF records → what sends email for them
  → TXT records → services used (Google, O365)

FROM ERROR PAGES:
  → Server software (Apache, Nginx, IIS)
  → Framework (ASP.NET, PHP, Ruby)
  → Version information

SE APPLICATION:
  → "I'm from [actual vendor they use] support"
  → "There's an issue with your [actual platform] account"
  → Pretexts that reference real tools are more convincing
```

---

## 4. Security Posture Assessment

```
ASSESSING HOW SECURITY-AWARE THEY ARE:

EMAIL SECURITY:
  dig company.com TXT → Check SPF strictness
  dig _dmarc.company.com TXT → Check DMARC policy
  
  No DMARC / p=none: Easy to spoof emails
  DMARC p=reject: Must use lookalike domain
  
  SPF ~all (softfail): Spoofed emails may deliver
  SPF -all (hardfail): Spoofed emails likely blocked

AWARENESS INDICATORS:
  → Do they have a security awareness program?
  → Is there a "report phishing" button?
  → Have they done phishing simulations?
  → Do they have a security blog/newsletter?
  → Are employees trained on social engineering?
  
  Sources: LinkedIn, job postings for security roles,
  company blog, employee reviews

PHYSICAL SECURITY:
  → Google Street View of entrances
  → Observe badge readers, cameras
  → Note visitor parking areas
  → Check for security guard presence
  → Delivery/loading dock access

MATURITY INDICATORS:
  LOW: No DMARC, no awareness program, no security team
  MEDIUM: Basic awareness, some email controls
  HIGH: Regular simulations, DMARC enforce, SOC team
```

---

## Summary Table

| Intelligence Area | Key Questions | Primary Sources |
|------------------|---------------|----------------|
| Structure | Who works where? Who reports to whom? | LinkedIn, website |
| Processes | How are things done? | Job postings, reviews |
| Culture | How do people communicate? | Glassdoor, social media |
| Technology | What systems do they use? | Job postings, DNS, headers |
| Security | How aware are they? | DMARC, website, news |

---

## Revision Questions

1. How do you map an organization's structure for social engineering?
2. What process information is most valuable for BEC attacks?
3. How do job postings reveal an organization's technology stack?
4. How do you assess a target's security awareness maturity?
5. What does DMARC policy tell you about email spoofing feasibility?

---

*Previous: [02-osint-for-social-engineering.md](02-osint-for-social-engineering.md) | Next: [04-social-media-intelligence.md](04-social-media-intelligence.md)*

---

*[Back to README](../README.md)*
