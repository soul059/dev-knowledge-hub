# Report Structure

## Unit 9 - Topic 2 | Reporting

---

## Overview

A **penetration test report** is the primary deliverable of an engagement. It communicates findings to both technical teams and executive management. A well-structured report is clear, actionable, and organized so that different audiences can find the information they need.

---

## 1. Standard Report Structure

```
┌──────────────────────────────────────────────────────────────┐
│              PENETRATION TEST REPORT STRUCTURE                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SECTION 1: COVER PAGE                                       │
│  └── Client name, date, classification, tester info          │
│                                                              │
│  SECTION 2: TABLE OF CONTENTS                                │
│  └── Clickable links to all sections                         │
│                                                              │
│  SECTION 3: EXECUTIVE SUMMARY (1-2 pages)                    │
│  └── For C-suite / non-technical audience                    │
│                                                              │
│  SECTION 4: SCOPE AND METHODOLOGY                            │
│  └── What was tested, how, and when                          │
│                                                              │
│  SECTION 5: FINDINGS SUMMARY                                 │
│  └── Chart/table of all findings by severity                 │
│                                                              │
│  SECTION 6: DETAILED FINDINGS                                │
│  └── Each vulnerability with full details                    │
│                                                              │
│  SECTION 7: REMEDIATION RECOMMENDATIONS                      │
│  └── Prioritized fix list                                    │
│                                                              │
│  SECTION 8: APPENDICES                                       │
│  └── Raw scan data, tools used, methodology details          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Report Sections in Detail

### Cover Page

```
═══════════════════════════════════════════════
        PENETRATION TEST REPORT
═══════════════════════════════════════════════

CLIENT:     Acme Corporation
TEST TYPE:  External Network Penetration Test
DATE:       January 15-19, 2024
VERSION:    1.0

CLASSIFICATION: CONFIDENTIAL

PREPARED BY: Security Testing LLC
LEAD TESTER: [Name], OSCP, CEH
CONTACT:    [email]

═══════════════════════════════════════════════
```

### Scope and Methodology

```
SCOPE:
├── IP Ranges: 203.0.113.0/24
├── Domains: *.acmecorp.com
├── Test Type: External, black-box
├── Exclusions: mail.acmecorp.com (production mail)
├── Testing Window: Jan 15-19, 2024 (09:00-18:00)
└── Source IP: 198.51.100.50

METHODOLOGY:
├── Standard: PTES (Penetration Testing Execution Standard)
├── Phase 1: Reconnaissance
├── Phase 2: Scanning & Enumeration
├── Phase 3: Exploitation
├── Phase 4: Post-Exploitation
└── Phase 5: Reporting
```

---

## 3. Findings Summary Format

```
FINDINGS OVERVIEW:

Severity    │ Count │ Visual
────────────┼───────┼──────────────────
Critical    │   2   │ ██████████
High        │   5   │ █████████████████████████
Medium      │   8   │ ████████████████████████████████████████
Low         │   3   │ ███████████████
Info        │   4   │ ████████████████████

TOTAL FINDINGS: 22

TOP 3 CRITICAL FINDINGS:
1. SQL Injection on login page (Critical)
2. EternalBlue on internal file server (Critical)
3. Default credentials on Tomcat manager (High)
```

---

## 4. Individual Finding Format

```
═══════════════════════════════════════════════
FINDING: F-001 — SQL Injection on Login Page
═══════════════════════════════════════════════

SEVERITY:    Critical (CVSS 9.8)
AFFECTED:    https://app.acmecorp.com/login
CVE:         N/A (application-specific)
CWE:         CWE-89: SQL Injection
STATUS:      Confirmed (exploited)

DESCRIPTION:
The login page is vulnerable to SQL injection in the
username parameter. An attacker can bypass authentication
and extract the entire database.

STEPS TO REPRODUCE:
1. Navigate to https://app.acmecorp.com/login
2. Enter username: ' OR 1=1--
3. Enter any password
4. Click "Login"
5. Observe: Logged in as first admin user

EVIDENCE:
[Screenshot showing successful bypass]
[Screenshot showing database extraction via sqlmap]

IMPACT:
• Complete authentication bypass
• Full database access (47,832 customer records)
• Potential for remote code execution via --os-shell

REMEDIATION:
• Use parameterized queries / prepared statements
• Implement input validation
• Deploy WAF rules for SQL injection patterns
• Conduct code review of all database queries

REFERENCES:
• OWASP SQL Injection: https://owasp.org/...
• CWE-89: https://cwe.mitre.org/data/definitions/89.html
═══════════════════════════════════════════════
```

---

## 5. Report Quality Checklist

| Item | Check |
|------|:-----:|
| Cover page with classification | ☐ |
| Executive summary (non-technical) | ☐ |
| Scope clearly defined | ☐ |
| Methodology referenced | ☐ |
| Findings sorted by severity | ☐ |
| Each finding has reproduction steps | ☐ |
| Screenshots/evidence included | ☐ |
| Remediation for each finding | ☐ |
| CVSS scores assigned | ☐ |
| Report reviewed by peer | ☐ |
| Spell-checked and formatted | ☐ |

---

## Summary Table

| Section | Audience | Content |
|---------|----------|---------|
| **Executive summary** | C-suite, managers | Risk overview, business impact |
| **Scope/Methodology** | Both | What was tested, how |
| **Findings summary** | Both | Chart of all findings |
| **Detailed findings** | Technical team | Steps, evidence, remediation |
| **Appendices** | Technical team | Raw data, tool output |

---

## Quick Revision Questions

1. **What are the main sections of a pen test report?**
2. **Who is the executive summary written for?**
3. **What should each individual finding include?**
4. **Why include steps to reproduce?**
5. **What goes in the appendices?**

---

[← Previous: Documentation Throughout Testing](01-documentation-throughout-testing.md) | [Next: Executive Summary →](03-executive-summary.md)

---

*Unit 9 - Topic 2 of 7 | [Back to README](../README.md)*
