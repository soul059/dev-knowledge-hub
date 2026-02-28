# Chapter 10.5: Bug Bounty Programs

## Overview

Bug bounty programs incentivize external security researchers to find and responsibly disclose vulnerabilities in your systems. They complement internal security testing by providing continuous, crowdsourced security assessment from diverse perspectives. This chapter covers program design, platform selection, vulnerability handling, and integration with DevSecOps workflows.

---

## Bug Bounty vs Traditional Pentesting

```
COMPARISON:

  ┌────────────────────────┬────────────────────────────┐
  │   PENETRATION TEST     │    BUG BOUNTY PROGRAM      │
  ├────────────────────────┼────────────────────────────┤
  │ Fixed scope & timeline │ Continuous & ongoing       │
  │ 1-3 testers            │ 100s-1000s of researchers  │
  │ Fixed cost             │ Pay per valid finding      │
  │ Scheduled (quarterly)  │ Always running             │
  │ Structured report      │ Individual submissions     │
  │ Known methodology      │ Diverse, creative approach │
  │ Compliance-satisfying  │ Broader coverage           │
  │ Deep but narrow        │ Wide but variable depth    │
  └────────────────────────┴────────────────────────────┘

  Best Practice: Use BOTH
  ┌──────────────────────────────────────────────────┐
  │ Pentest → Deep, structured assessment            │
  │ Bug Bounty → Continuous crowdsourced testing     │
  │ Together → Maximum vulnerability coverage        │
  └──────────────────────────────────────────────────┘
```

---

## Bug Bounty Program Types

```
PROGRAM MODELS:

  1. PRIVATE PROGRAM
  ┌────────────────────────────────────────────────┐
  │ • Invite-only researchers (vetted)              │
  │ • Limited scope initially                       │
  │ • Lower volume, higher quality                  │
  │ • Recommended to START here                     │
  └────────────────────────────────────────────────┘

  2. PUBLIC PROGRAM
  ┌────────────────────────────────────────────────┐
  │ • Open to all researchers                       │
  │ • Broader scope                                 │
  │ • Higher volume (including noise)               │
  │ • Mature security posture required              │
  └────────────────────────────────────────────────┘

  3. VULNERABILITY DISCLOSURE PROGRAM (VDP)
  ┌────────────────────────────────────────────────┐
  │ • No monetary reward (recognition only)         │
  │ • Safe harbour policy                           │
  │ • Legal protection for researchers              │
  │ • Minimum recommended program                   │
  └────────────────────────────────────────────────┘

  Progression:
  VDP ──▶ Private Bug Bounty ──▶ Public Bug Bounty
```

---

## Bug Bounty Platforms

| Platform | Focus | Features |
|----------|-------|----------|
| **HackerOne** | Largest platform | Managed programs, triage support, compliance |
| **Bugcrowd** | Curated researchers | Managed triage, penetration testing add-ons |
| **Intigriti** | European market | GDPR-compliant, managed programs |
| **YesWeHack** | European platform | French/EU based, ANSSI collaboration |
| **Synack** | Managed + vetted | Vetted researchers only, compliance-focused |
| **Self-hosted** | Full control | security.txt, custom portal |

---

## Program Design

### Scope Definition

```
SCOPE EXAMPLE:

  ┌──────────────────────────────────────────────────────┐
  │ IN SCOPE ✅                                           │
  │                                                        │
  │ Domains:                                               │
  │ • *.example.com                                       │
  │ • api.example.com                                     │
  │ • app.example.com                                     │
  │                                                        │
  │ Applications:                                          │
  │ • Web application (app.example.com)                   │
  │ • REST API (api.example.com/v2/*)                     │
  │ • Mobile app (iOS + Android)                          │
  │                                                        │
  │ Vulnerability Types:                                   │
  │ • OWASP Top 10                                        │
  │ • Business logic flaws                                │
  │ • Authentication/authorization bypass                  │
  │ • Server-side vulnerabilities                         │
  │ • API security issues                                 │
  ├──────────────────────────────────────────────────────┤
  │ OUT OF SCOPE ❌                                        │
  │                                                        │
  │ • Third-party services (CDN, analytics)               │
  │ • Physical security / social engineering              │
  │ • DoS/DDoS attacks                                    │
  │ • Automated scanning without coordination             │
  │ • blog.example.com (WordPress — third-party)          │
  │ • Rate limiting issues                                │
  │ • SPF/DKIM/DMARC misconfiguration                    │
  │ • Self-XSS (no impact on other users)                │
  └──────────────────────────────────────────────────────┘
```

### Reward Structure

```
BOUNTY REWARD TABLE:

  ┌───────────────┬──────────────┬────────────────────────┐
  │  Severity     │  Reward      │  Examples              │
  ├───────────────┼──────────────┼────────────────────────┤
  │  CRITICAL     │  $5,000-     │  RCE, SQLi with data   │
  │  (CVSS 9.0+)  │  $25,000     │  access, auth bypass   │
  ├───────────────┼──────────────┼────────────────────────┤
  │  HIGH         │  $2,000-     │  Stored XSS, IDOR      │
  │  (CVSS 7.0-8.9│  $10,000     │  with data access,     │
  │               │              │  privilege escalation   │
  ├───────────────┼──────────────┼────────────────────────┤
  │  MEDIUM       │  $500-       │  Reflected XSS, CSRF,  │
  │  (CVSS 4.0-6.9│  $3,000      │  information disclosure│
  ├───────────────┼──────────────┼────────────────────────┤
  │  LOW          │  $100-       │  Open redirect, verbose │
  │  (CVSS 0.1-3.9│  $500        │  errors, missing       │
  │               │              │  headers               │
  └───────────────┴──────────────┴────────────────────────┘

  Note: Rewards vary greatly by company size/budget.
  Tech giants pay $10K-$100K+ for critical findings.
```

---

## Vulnerability Handling Workflow

```
BUG BOUNTY SUBMISSION LIFECYCLE:

  Researcher                        Your Team
  ──────────                        ─────────
  Discovers vuln
       │
       ▼
  Submits report ──────────────▶ Receive report
  (via platform)                      │
                                      ▼
                                ┌──────────┐
                                │  TRIAGE   │ (< 24h)
                                │           │
                                │ Valid?    │
                                │ Severity? │
                                │ Duplicate?│
                                └─────┬────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    ▼                 ▼                  ▼
              ┌──────────┐    ┌──────────┐       ┌──────────┐
              │ VALID    │    │ DUPLICATE│       │ INVALID  │
              │          │    │          │       │          │
              │ Assign   │    │ Link to  │       │ Explain  │
              │ to dev   │    │ original │       │ why not  │
              │ team     │    │ Close    │       │ valid    │
              └─────┬────┘    └──────────┘       └──────────┘
                    │
                    ▼
              ┌──────────┐
              │   FIX    │  (SLA: 7d critical,
              │          │   30d high, 90d medium)
              │ Develop  │
              │ Test     │
              │ Deploy   │
              └─────┬────┘
                    │
                    ▼
              ┌──────────┐
              │ VERIFY & │
              │ REWARD   │
              │          │
              │ Confirm  │
              │ fix      │
              │ Pay      │
              │ bounty   │
              └──────────┘
```

---

## Integrating Bug Bounty with DevSecOps

```yaml
# GitHub Actions — Auto-create issue from bounty report
name: Bug Bounty Triage
on:
  repository_dispatch:
    types: [bounty-report]

jobs:
  create-issue:
    runs-on: ubuntu-latest
    steps:
      - name: Create Security Issue
        uses: actions/github-script@v7
        with:
          script: |
            const report = context.payload.client_payload;
            
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `[BOUNTY-${report.severity}] ${report.title}`,
              body: `
              ## Bug Bounty Report
              
              **Platform**: ${report.platform}
              **Report ID**: ${report.id}
              **Severity**: ${report.severity}
              **CVSS**: ${report.cvss}
              
              ### Description
              ${report.description}
              
              ### Steps to Reproduce
              ${report.steps}
              
              ### Impact
              ${report.impact}
              
              ### SLA
              - Critical: Fix within 7 days
              - High: Fix within 30 days
              - Medium: Fix within 90 days
              `,
              labels: ['security', 'bug-bounty', report.severity.toLowerCase()],
              assignees: ['security-team-lead']
            });
```

```
DevSecOps Integration Flow:

  Bug Bounty    ──▶  Triage     ──▶  JIRA/GitHub  ──▶  Developer
  Platform           (Security      Issue              Fixes
                      Team)                              │
                                                         ▼
                                                    CI/CD Pipeline
                                                    ┌────────────┐
                                                    │ Unit Tests │
  Bounty Paid   ◀──  Verified   ◀──  Deployed  ◀── │ SAST/DAST  │
  Report Closed      in Staging      to Prod        │ Regression │
                                                    └────────────┘
```

---

## security.txt Standard

```
# .well-known/security.txt (RFC 9116)
# Place at: https://example.com/.well-known/security.txt

Contact: mailto:security@example.com
Contact: https://hackerone.com/example
Expires: 2025-12-31T23:59:00.000Z
Encryption: https://example.com/.well-known/pgp-key.txt
Acknowledgments: https://example.com/hall-of-fame
Policy: https://example.com/security-policy
Preferred-Languages: en
Canonical: https://example.com/.well-known/security.txt
```

---

## Responsible Disclosure Policy Template

```markdown
# Responsible Disclosure Policy

## Guidelines
We welcome security researchers to help us improve our security.
If you discover a vulnerability, please:

1. **Report** it promptly via [Platform/Email]
2. **Don't** access, modify, or delete user data
3. **Don't** perform DoS attacks
4. **Don't** use automated scanners excessively
5. **Do** provide sufficient details to reproduce

## Our Commitment
- Acknowledge receipt within 24 hours
- Provide triage assessment within 72 hours  
- Keep you updated on remediation progress
- Credit you publicly (if desired)
- Pay bounty within 30 days of fix verification
- No legal action against good-faith researchers

## Safe Harbour
We will not pursue legal action against researchers who:
- Act in good faith following this policy
- Report findings promptly
- Avoid privacy violations
- Don't exploit beyond proof of concept
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Too many duplicate reports | Common vulnerability well-known | Keep program updated with known issues; close duplicates quickly with references |
| Noise from invalid submissions | Low-quality researchers or bots | Use private program initially; require detailed reproduction steps; implement structured submission forms |
| Slow triage response time | Security team overwhelmed | Use platform managed triage; set up automation for classification; hire dedicated triage staff |
| Researchers frustrated by slow fixing | Engineering team doesn't prioritize | Set SLAs by severity; track in same issue system as other bugs; escalate overdue items |
| Budget exceeded | Too many valid findings (good problem!) | Adjust scope; fix root causes (SAST in CI/CD) to reduce volume; adjust reward tiers |

---

## Summary Table

| Aspect | Recommendation |
|--------|---------------|
| **Program Type** | Start with VDP → Private → Public |
| **Platform** | HackerOne, Bugcrowd, or Intigriti |
| **Scope** | Well-defined in-scope/out-of-scope lists |
| **Rewards** | Tiered by CVSS severity |
| **SLAs** | Critical: 7d, High: 30d, Medium: 90d |
| **Integration** | Auto-create tickets; track alongside dev bugs |
| **Disclosure** | security.txt + responsible disclosure policy |
| **Metrics** | Track MTTR, cost per vuln, researcher satisfaction |

---

## Quick Revision Questions

1. **What is a bug bounty program and how does it differ from pentesting?**
   > A bug bounty program incentivizes external security researchers to find and responsibly report vulnerabilities, paying per valid finding. Unlike pentesting (fixed scope, timeline, team), bug bounties are continuous, involve hundreds of diverse researchers, and provide pay-per-result. Use both: pentests for depth, bounties for breadth.

2. **What is the recommended progression for starting a bug bounty program?**
   > Start with a Vulnerability Disclosure Program (VDP) — no monetary rewards, just a safe channel for reporting. Then launch a private bug bounty with vetted, invited researchers and limited scope. Once the security posture is mature and the team can handle volume, open a public bug bounty program.

3. **What should be included in a bug bounty scope?**
   > In-scope: specific domains, applications, APIs, and vulnerability types. Out-of-scope: third-party services, social engineering, DoS/DDoS, excessive automated scanning, specific low-value findings. Clear scope prevents researcher frustration and reduces noise. Update scope regularly as your attack surface changes.

4. **How should bug bounty findings integrate with DevSecOps?**
   > Reports should automatically create tickets in the development issue tracker (JIRA, GitHub Issues) with severity labels and SLAs. Fixes go through the standard CI/CD pipeline with SAST/DAST regression tests. Verified fixes trigger bounty payment. Root cause analysis should inform SAST rule updates to prevent recurring vulnerability classes.

5. **What is a responsible disclosure policy and why is it important?**
   > A responsible disclosure policy provides guidelines for security researchers, commits to specific response timelines (acknowledgement, triage, remediation), offers safe harbour from legal action, and establishes communication channels. It's important because it encourages vulnerability reporting over exploitation and protects both the organization and researchers legally.

---

[← Previous: 10.4 Penetration Testing](04-penetration-testing.md) | [Next: 10.6 Security Compliance →](06-security-compliance.md)

[Back to Table of Contents](../README.md)
