# Scoping Discussions

## Unit 4 - Topic 1 | Pre-engagement

---

## Overview

**Scoping** is the process of defining exactly what will be tested, how, and to what extent. Proper scoping prevents misunderstandings, ensures accurate pricing, and establishes clear boundaries. It's often the first meeting between the pen test team and the client.

---

## 1. Scoping Meeting Agenda

```
┌──────────────────────────────────────────────────────────────┐
│              SCOPING MEETING STRUCTURE                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. BUSINESS OBJECTIVES (Why?)                               │
│     What is the client trying to achieve?                    │
│     Compliance? Risk reduction? Due diligence?               │
│                                                              │
│  2. ASSETS & TARGETS (What?)                                 │
│     IP ranges, domains, applications, infrastructure         │
│                                                              │
│  3. TEST TYPE (How?)                                         │
│     Black/gray/white box? External/internal?                │
│     Social engineering? Physical? Wireless?                  │
│                                                              │
│  4. CONSTRAINTS (Limits?)                                    │
│     Testing hours, excluded systems, methods                 │
│                                                              │
│  5. LOGISTICS (When/Who?)                                    │
│     Timeline, contacts, VPN access, credentials              │
│                                                              │
│  6. DELIVERABLES (Output?)                                   │
│     Report format, presentation, retesting                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Key Scoping Questions

| Category | Questions to Ask |
|----------|-----------------|
| **Business** | Why is the pen test needed? Compliance or proactive? |
| **Scope** | How many IPs? Domains? Applications? |
| **Environment** | Production or staging? Cloud or on-prem? |
| **Sensitivity** | What data types are present? (PII, PHI, PCI) |
| **Previous tests** | Has pen testing been done before? Findings? |
| **Architecture** | Network diagrams? Technology stack? |
| **Restrictions** | Any systems we absolutely cannot touch? |
| **Timing** | When can we test? Maintenance windows? |
| **Access** | Will we get VPN access? User credentials? |
| **Contacts** | Who do we call if something goes wrong? |

---

## 3. Scope Sizing

```
SCOPE SIZE ESTIMATION:

EXTERNAL PEN TEST:
├── Small:   1-10 external IPs
├── Medium:  10-50 external IPs
├── Large:   50-100+ external IPs
└── Pricing factors: IPs, domains, web apps

INTERNAL PEN TEST:
├── Small:   1-50 internal hosts
├── Medium:  50-500 internal hosts
├── Large:   500+ internal hosts
└── Pricing factors: hosts, AD complexity, locations

WEB APPLICATION:
├── Small:   Simple app, <20 pages/endpoints
├── Medium:  Complex app, 20-100 endpoints
├── Large:   Enterprise app, 100+ endpoints, APIs
└── Pricing factors: complexity, roles, API count

SOCIAL ENGINEERING:
├── Phishing: Number of targets (50-500+)
├── Vishing: Number of calls
├── Physical: Number of locations
└── Pricing factors: target count, scenario complexity
```

---

## 4. Common Scoping Mistakes

| Mistake | Impact | Prevention |
|---------|--------|-----------|
| **Vague scope** | Disputes, missed systems | Document everything specifically |
| **Scope too large** | Rushed testing, poor quality | Be realistic about time |
| **Missing cloud assets** | Incomplete test | Ask about cloud infrastructure |
| **Forgotten APIs** | Untested attack surface | Map all endpoints |
| **No exclusions** | Accidental impact | Explicitly list out-of-scope |
| **Unclear test type** | Wrong approach | Confirm black/gray/white box |

---

## 5. Scoping Document Template

```
SCOPE DOCUMENT:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CLIENT: ________________
DATE: ________________
PROJECT: External Penetration Test

TEST TYPE:
☑ External  ☐ Internal  ☐ Web App
☐ Wireless  ☐ Social Engineering

KNOWLEDGE LEVEL:
☐ Black Box  ☑ Gray Box  ☐ White Box

IN-SCOPE TARGETS:
• 203.0.113.0/24 (external range)
• www.example.com
• api.example.com
• mail.example.com

OUT-OF-SCOPE:
• Partner systems (*.partner.com)
• Production database direct access
• DoS/DDoS testing

TESTING WINDOW:
Start: 2024-01-15
End: 2024-02-15
Hours: Mon-Fri, 6 PM - 6 AM EST

CREDENTIALS PROVIDED: ☑ Yes  ☐ No
• Standard user account for web app
• VPN credentials for internal testing

DELIVERABLES:
☑ Executive report
☑ Technical report
☑ Debrief presentation
☑ One round of retesting
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Scoping** | Define exactly what will be tested |
| **Key questions** | Business goals, targets, restrictions, timing |
| **Sizing** | IP count, app complexity, target count |
| **Document** | Written scope agreement signed by both parties |
| **Mistakes** | Vague scope, missing cloud, forgotten APIs |

---

## Quick Revision Questions

1. **What is the purpose of a scoping meeting?**
2. **Name 5 key questions to ask during scoping.**
3. **How do you estimate the size of an external pen test?**
4. **What common scoping mistakes should be avoided?**
5. **Why must out-of-scope items be explicitly documented?**

---

[Next: Rules of Engagement Document →](02-rules-of-engagement-document.md)

---

*Unit 4 - Topic 1 of 6 | [Back to README](../README.md)*
