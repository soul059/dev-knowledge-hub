# Employee Enumeration

## Unit 7 - Topic 1 | Social Engineering Reconnaissance

---

## Overview

**Employee enumeration** identifies individuals within the target organization — their names, roles, departments, and contact information. This intelligence is critical for crafting targeted social engineering attacks, spear phishing, and identifying high-value targets with privileged access.

---

## 1. Employee Discovery Sources

```
EMPLOYEE INFORMATION SOURCES:

PUBLIC SOURCES:
├── LinkedIn — company page, employee profiles
├── Company website — About Us, Team, Leadership
├── Press releases — quoted employees, executives
├── Conference talks — speakers, presenters
├── GitHub — code contributors with company email
├── Job postings — reveal team structure, tech stack
├── Court records — named individuals in filings
└── SEC filings — executives, board members

SOCIAL MEDIA:
├── Twitter/X — employees posting about work
├── Facebook — company page, employee groups
├── Instagram — office photos, events
├── Reddit — employees discussing company
└── Glassdoor — employee reviews, salary info

TECHNICAL SOURCES:
├── Email harvesting — theHarvester, hunter.io
├── WHOIS records — admin/tech contacts
├── Document metadata — author fields
├── DNS SOA records — admin email
└── SSL certificates — organizational contacts
```

---

## 2. LinkedIn Enumeration

```bash
# === LINKEDIN TECHNIQUES ===

# Manual LinkedIn Search:
# site:linkedin.com/in "Target Corporation"
# site:linkedin.com "works at Target Corporation"

# === LINKEDIN SCRAPING TOOLS ===

# CrossLinked — scrape LinkedIn for employee names
crosslinked -f '{first}.{last}@target.com' \
  -t 'company:"Target Corporation"' \
  -o employees.txt

# linkedin2username — generate usernames from LinkedIn
linkedin2username -c "Target Corporation" -o names.txt

# === INFORMATION TO COLLECT ===
# For each employee:
# ├── Full name
# ├── Job title / role
# ├── Department
# ├── Location
# ├── Skills / certifications
# ├── Previous employment
# ├── Education
# ├── Connections (other employees)
# └── Posts and activity

# === EMPLOYEE DATABASE FORMAT ===
# Name, Title, Department, Email, LinkedIn URL, Notes
# John Smith, CISO, Security, john.smith@target.com, linkedin.com/in/jsmith, Decision maker
# Jane Doe, Sysadmin, IT, jane.doe@target.com, linkedin.com/in/jdoe, Has admin access
```

---

## 3. Role-Based Target Prioritization

| Role | Value | Why |
|------|:-----:|-----|
| **CEO/CFO** | ★★★★★ | Authority for wire transfers, BEC target |
| **CISO/CTO** | ★★★★★ | Security decisions, admin access |
| **System Admin** | ★★★★★ | Root/admin privileges |
| **Help Desk** | ★★★★ | Social engineering target (password resets) |
| **Developer** | ★★★★ | Code access, may have production creds |
| **HR/Recruiting** | ★★★★ | Opens attachments, processes applications |
| **Finance** | ★★★★ | Wire transfer authority |
| **New Employees** | ★★★ | Less security awareness |
| **Executives' Assistants** | ★★★ | Access to executive accounts |
| **Interns** | ★★ | Minimal training, eager to help |

---

## 4. Building Employee Profiles

```
EMPLOYEE PROFILE: John Smith
━━━━━━━━━━━━━━━━━━━━━━━━━━━

IDENTITY:
├── Name: John Smith
├── Title: Senior System Administrator
├── Department: IT Infrastructure
├── Location: New York, NY
├── Email: john.smith@target.com
└── Phone: (found on company directory)

TECHNICAL PROFILE:
├── Certifications: CISSP, MCSE
├── Skills: Active Directory, VMware, Azure
├── GitHub: github.com/jsmith-admin
└── Technology access: likely Domain Admin

SOCIAL PROFILE:
├── LinkedIn: linkedin.com/in/john-smith-admin
├── Twitter: @jsmith_tech (posts about work)
├── Hobbies: mentioned cycling, dogs
├── Recent post: "Just upgraded to Windows 11"
└── Travels: posted from DEF CON 2024

ATTACK ANGLES:
├── Phishing: tech-related lure (VMware update)
├── Pretexting: vendor call about Azure issue
├── Social: cycling event invitation
└── Value: Domain Admin credentials
```

---

## 5. Organizational Chart Building

```
TARGET CORPORATION — ORG CHART (Partial):

CEO: Sarah Johnson
├── CFO: Michael Chen
│   ├── Controller: ...
│   └── AP Manager: Amy Wong (wire transfers!)
├── CTO: Robert Davis
│   ├── VP Engineering: ...
│   │   ├── Dev Team Lead: ...
│   │   └── DevOps Lead: ...
│   └── IT Director: Patricia Lee
│       ├── Sysadmin: John Smith ← TARGET
│       ├── Helpdesk: Tom Brown  ← TARGET
│       └── Network Eng: ...
└── CISO: David Kim
    ├── SOC Manager: ...
    └── Pen Tester: ...

INTELLIGENCE VALUE:
├── Reporting structure = authority for pretexting
├── IT staff = privileged access targets
├── Finance = BEC/wire transfer targets
└── Help desk = password reset social engineering
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **LinkedIn** | Primary source for employee enumeration |
| **CrossLinked** | Automated LinkedIn scraping → emails |
| **Prioritization** | Target by role and access level |
| **Profiles** | Build detailed dossiers per employee |
| **Org chart** | Map reporting structure for pretexting |
| **Metadata** | Document authors reveal employees |

---

## Quick Revision Questions

1. **What sources provide employee information?**
2. **Why is role-based prioritization important?**
3. **How does CrossLinked generate email addresses?**
4. **What makes help desk staff high-value targets?**
5. **How does an org chart help social engineering?**

---

[Next: Organizational Structure →](02-organizational-structure.md)

---

*Unit 7 - Topic 1 of 5 | [Back to README](../README.md)*
