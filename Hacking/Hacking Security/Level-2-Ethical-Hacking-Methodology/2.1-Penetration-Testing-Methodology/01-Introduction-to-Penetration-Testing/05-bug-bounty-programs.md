# Bug Bounty Programs

## Unit 1 - Topic 5 | Introduction to Penetration Testing

---

## Overview

**Bug bounty programs** invite security researchers to find and report vulnerabilities in exchange for **rewards (bounties)**. They provide continuous security testing through crowd-sourced efforts, complementing traditional pen tests and vulnerability assessments.

---

## 1. How Bug Bounty Programs Work

```
┌──────────────────────────────────────────────────────────────┐
│              BUG BOUNTY WORKFLOW                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Organization defines scope & rules                       │
│  2. Researchers find vulnerabilities                         │
│  3. Researcher submits report                                │
│  4. Organization triages the report                          │
│  5. Vulnerability is confirmed/rejected                      │
│  6. Bounty is paid (if valid & in scope)                    │
│  7. Organization fixes the vulnerability                     │
│  8. Researcher may disclose after fix                        │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │RESEARCHER│─►│  SUBMIT  │─►│  TRIAGE  │─►│  BOUNTY  │   │
│  │  finds   │  │  report  │  │  by org  │  │  paid $$  │   │
│  │  vuln    │  │          │  │          │  │          │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Major Bug Bounty Platforms

| Platform | Notable Programs | Min/Max Bounties |
|----------|-----------------|:---:|
| **HackerOne** | US DoD, GitHub, Uber, Shopify | $50 - $250K+ |
| **Bugcrowd** | Mastercard, Tesla, Twitch | $50 - $100K+ |
| **Intigriti** | European programs | €50 - €50K+ |
| **Synack** | Invite-only, vetted researchers | Private |
| **YesWeHack** | French government, EU programs | €50 - €50K+ |

### Record Bounties
- **Apple**: $100,000 - $2,000,000 (iOS kernel zero-click)
- **Google**: $1,500 - $250,000+ (Chrome, Android)
- **Microsoft**: $500 - $250,000 (Azure, Windows)
- **Facebook/Meta**: $500 - $300,000+

---

## 3. Bounty Amounts by Severity

| Severity | CVSS | Typical Bounty |
|----------|:----:|:--------------:|
| **Critical** | 9.0-10.0 | $5,000 - $100,000+ |
| **High** | 7.0-8.9 | $2,000 - $25,000 |
| **Medium** | 4.0-6.9 | $500 - $5,000 |
| **Low** | 0.1-3.9 | $100 - $1,000 |
| **Informational** | N/A | $0 - $100 (or swag) |

---

## 4. Bug Bounty vs Traditional Pen Test

| Aspect | Bug Bounty | Penetration Test |
|--------|:---:|:---:|
| **Testers** | Hundreds/thousands | 1-5 professionals |
| **Duration** | Ongoing (continuous) | Fixed timeframe |
| **Cost model** | Pay per valid bug | Fixed project fee |
| **Diversity** | Many perspectives | Limited team |
| **Depth** | Varies by researcher | Structured methodology |
| **Coverage** | May focus on easy wins | Systematic coverage |
| **Reporting** | Individual reports | Comprehensive report |
| **Legal protection** | Platform terms | Detailed contract |

---

## 5. Starting in Bug Bounty

```
BEGINNER ROADMAP:
│
├── 1. LEARN THE BASICS
│   ├── OWASP Top 10
│   ├── Web application security
│   ├── Networking fundamentals
│   └── Linux & scripting
│
├── 2. PRACTICE
│   ├── HackTheBox, TryHackMe
│   ├── PortSwigger Web Security Academy (free)
│   ├── OWASP Juice Shop
│   └── DVWA, bWAPP
│
├── 3. CHOOSE A PLATFORM
│   ├── HackerOne (largest)
│   ├── Bugcrowd
│   └── Start with programs that accept beginners
│
├── 4. START SIMPLE
│   ├── Look for XSS, IDOR, info disclosure
│   ├── Read disclosed reports for inspiration
│   └── Focus on one program deeply
│
└── 5. WRITE GREAT REPORTS
    ├── Clear reproduction steps
    ├── Impact assessment
    ├── Screenshots/video proof
    └── Remediation suggestions
```

### Essential Tools for Bug Bounty

| Tool | Purpose |
|------|---------|
| **Burp Suite** | Web proxy & scanner |
| **Amass** | Subdomain enumeration |
| **Nuclei** | Automated vulnerability scanning |
| **ffuf** | Web fuzzing |
| **httpx** | HTTP probing |
| **Subfinder** | Subdomain discovery |
| **Nmap** | Network scanning |

---

## 6. Writing a Good Bug Report

```
REPORT TEMPLATE:
├── Title: [Severity] Vulnerability Type in [Feature]
├── Summary: 1-2 sentence description
├── Severity: Critical/High/Medium/Low
├── Steps to Reproduce:
│   1. Navigate to...
│   2. Enter payload...
│   3. Click submit...
│   4. Observe...
├── Impact: What an attacker can do
├── Affected URL/Endpoint: https://...
├── Screenshots/Video: Proof of concept
└── Remediation: How to fix it
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Bug bounty** | Crowd-sourced vulnerability discovery for rewards |
| **Platforms** | HackerOne, Bugcrowd, Intigriti |
| **Scope** | Only test what's explicitly in scope |
| **Reports** | Clear steps, impact, proof, remediation |
| **Continuous** | Ongoing testing vs point-in-time pen test |

---

## Quick Revision Questions

1. **What is a bug bounty program?**
2. **Name 3 major bug bounty platforms.**
3. **How does a bug bounty differ from a penetration test?**
4. **What makes a good bug bounty report?**
5. **What should a beginner focus on when starting bug bounty?**

---

[← Previous: Red Team vs Penetration Testing](04-red-team-vs-penetration-testing.md)

---

*Unit 1 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Legal and Ethical Considerations →](../02-Legal-and-Ethical-Considerations/01-authorization-and-scope.md)*
