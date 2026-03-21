# Organizational Structure

## Unit 7 - Topic 2 | Social Engineering Reconnaissance

---

## Overview

**Organizational structure reconnaissance** maps the hierarchy, departments, reporting relationships, and business processes of the target organization. This intelligence enables realistic pretexting scenarios and identifies decision-makers, gatekeepers, and weakest links.

---

## 1. Structure Discovery Sources

```
ORGANIZATIONAL INTELLIGENCE SOURCES:

PUBLIC FILINGS:
├── SEC filings (10-K, 10-Q, proxy)  → Executives, subsidiaries
├── Annual reports                    → Structure, divisions
├── Business registrations           → Legal entities
├── Patent filings                   → R&D teams, inventors
└── Government contracts             → Key personnel

COMPANY COMMUNICATIONS:
├── Company website (About Us)       → Leadership, divisions
├── Press releases                   → Org changes, new hires
├── Blog posts                       → Team introductions
├── Investor presentations           → Business unit structure
└── Job postings                     → Departments, teams

THIRD-PARTY SOURCES:
├── LinkedIn Company Page            → Employee count, locations
├── Glassdoor                        → Org culture, departments
├── Crunchbase                       → Funding, leadership
├── ZoomInfo / Apollo                → Company database
└── D&B (Dun & Bradstreet)          → Corporate family tree
```

---

## 2. Organizational Mapping

```
MAPPING THE TARGET ORGANIZATION:

CORPORATE STRUCTURE:
Target Corporation (Parent)
├── Target Solutions LLC (Software Division)
│   ├── Engineering Department (50 employees)
│   ├── Product Department (20 employees)
│   └── QA Department (15 employees)
├── Target Services Inc (Consulting Division)
│   ├── Professional Services (30 employees)
│   └── Customer Support (40 employees)
├── Target Financial (Finance Subsidiary)
│   └── Payment Processing (25 employees)
└── Shared Services
    ├── IT / Infrastructure (20 employees)
    ├── HR / People Operations (10 employees)
    ├── Legal / Compliance (8 employees)
    ├── Finance / Accounting (12 employees)
    └── Marketing (15 employees)

TOTAL: ~245 employees
LOCATIONS: New York (HQ), San Francisco, London
```

---

## 3. Intelligence for Social Engineering

| Structure Element | Social Engineering Use |
|------------------|----------------------|
| **Reporting chain** | "I'm calling on behalf of [boss name]" |
| **Department names** | Impersonate specific departments |
| **Business processes** | Understand approval workflows |
| **Vendor relationships** | Impersonate known vendors |
| **Recent changes** | Exploit confusion during transitions |
| **Remote offices** | "I'm from the [city] office" |
| **Subsidiaries** | Impersonate sister company staff |

---

## 4. Business Process Intelligence

```
KEY BUSINESS PROCESSES TO UNDERSTAND:

FINANCIAL:
├── Who approves purchases?
├── Wire transfer authorization process
├── Invoice submission (vendor impersonation)
├── Expense report system
└── Payroll processing (direct deposit changes)

IT OPERATIONS:
├── How are password resets handled?
├── New employee onboarding (IT provisioning)
├── Software deployment process
├── Incident response procedures
└── Remote access (VPN) provisioning

PHYSICAL SECURITY:
├── Badge/access card issuance
├── Visitor registration process
├── Delivery handling procedures
├── Contractor access procedures
└── After-hours access policies

COMMUNICATION:
├── Internal communication tools (Slack, Teams?)
├── Email conventions and formatting
├── Meeting scheduling tools
├── Company-wide announcement channels
└── Emergency communication procedures
```

---

## 5. Pretexting Scenarios from Structure Intel

```
SCENARIO 1: IT DEPARTMENT IMPERSONATION
────────────────────────────────────────
Intel: IT Director is Patricia Lee, uses ServiceNow
Pretext: "Hi, this is Mark from Patricia's team.
We're migrating email systems tonight and need
you to verify your credentials on our portal."

SCENARIO 2: VENDOR IMPERSONATION
────────────────────────────────────────
Intel: Company uses Salesforce (from job postings)
Pretext: "This is Alex from Salesforce support.
We detected an issue with your instance and
need admin access to apply the fix."

SCENARIO 3: EXECUTIVE AUTHORITY
────────────────────────────────────────
Intel: CFO Michael Chen, urgent wire transfers
Pretext: Email from "M.Chen" requesting urgent
wire transfer for "confidential acquisition"
to finance team member Amy Wong.

SCENARIO 4: NEW EMPLOYEE
────────────────────────────────────────
Intel: Recent job postings for engineering
Pretext: "Hi, I'm the new developer starting
Monday. IT told me to call you for my VPN
credentials. My manager is [real name]."
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Org structure** | Map hierarchy and departments |
| **Subsidiaries** | Separate entities = separate targets |
| **Business processes** | Understand workflows for pretexting |
| **Job postings** | Reveal internal tools and structure |
| **Vendor knowledge** | Enable vendor impersonation |
| **Authority chain** | Know who reports to whom |

---

## Quick Revision Questions

1. **What public sources reveal organizational structure?**
2. **How do subsidiaries create additional attack surface?**
3. **Why is understanding business processes important for social engineering?**
4. **How can job postings reveal internal information?**
5. **Give an example of a pretexting scenario using org structure intel.**

---

[← Previous: Employee Enumeration](01-employee-enumeration.md) | [Next: Social Media Profiling →](03-social-media-profiling.md)

---

*Unit 7 - Topic 2 of 5 | [Back to README](../README.md)*
