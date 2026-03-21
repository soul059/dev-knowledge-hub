# Unit 8: Post-Incident Activities — Topic 2: Report Writing

## Overview

**Incident report writing** documents the complete incident lifecycle for stakeholders, legal, compliance, and organizational learning. A well-written report serves multiple audiences — from executives needing impact summaries to technical teams needing detailed findings. Quality reports are essential for regulatory compliance, insurance claims, and preventing future incidents.

---

## 1. Report Types

```
INCIDENT REPORT TYPES:

  Report Type          | Audience         | Length  | Detail
  Executive Summary    | C-suite, Board   | 1-2 pg  | High-level
  Management Report    | Directors, Mgrs  | 3-5 pg  | Moderate
  Technical Report     | IR team, IT      | 10-30pg | Detailed
  Regulatory Report    | Regulators       | Varies  | As required
  Legal/Forensic Report| Legal, law enf.  | 20-50pg | Very detailed
  Lessons Learned      | All stakeholders | 3-5 pg  | Process focus

REPORT CONTENT MATRIX:
  Section              | Exec | Mgmt | Tech | Legal
  Summary              |  ✓   |  ✓   |  ✓   |  ✓
  Timeline             | Brief| Brief| Full | Full
  Impact               |  ✓   |  ✓   |  ✓   |  ✓
  Technical details    |  ✗   | Some |  ✓   |  ✓
  IOCs                 |  ✗   |  ✗   |  ✓   |  ✓
  Root cause           | Brief|  ✓   |  ✓   |  ✓
  Evidence             |  ✗   |  ✗   |  ✓   |  ✓
  Recommendations      |  ✓   |  ✓   |  ✓   |  ✓
  Costs                |  ✓   |  ✓   |  ✗   |  ✓
  Legal implications   |  ✓   |  ✓   |  ✗   |  ✓
```

---

## 2. Report Structure

```
TECHNICAL INCIDENT REPORT TEMPLATE:

  1. EXECUTIVE SUMMARY (1 page)
     → Incident type and severity
     → Date of compromise and detection
     → Brief impact statement
     → Current status
     → Key findings (3-5 bullets)
     → Critical recommendations (3-5 bullets)

  2. INCIDENT OVERVIEW
     → Detection: How and when discovered
     → Classification: Incident type/category
     → Severity: Level and justification
     → Scope: Systems, users, data affected
     → Status: Open, contained, resolved

  3. TIMELINE
     → Chronological event sequence
     → Key milestones highlighted
     → Both attacker and defender actions
     → Evidence referenced for each entry

  4. TECHNICAL ANALYSIS
     → Initial access vector
     → Execution details
     → Persistence mechanisms
     → Lateral movement
     → Data access/exfiltration
     → Tools and techniques used
     → ATT&CK technique mapping

  5. IMPACT ASSESSMENT
     → Systems affected (count, criticality)
     → Data affected (type, volume, sensitivity)
     → Users affected
     → Business operations impact
     → Financial impact estimate
     → Regulatory/compliance impact
     → Reputational impact

  6. INDICATORS OF COMPROMISE
     → IP addresses
     → Domain names
     → File hashes
     → File paths
     → Registry keys
     → Email addresses
     → User agents

  7. ROOT CAUSE ANALYSIS
     → Primary root cause
     → Contributing factors
     → How the vulnerability was exploited

  8. RESPONSE ACTIONS
     → Containment actions taken
     → Eradication steps
     → Recovery activities
     → Timeline of response

  9. RECOMMENDATIONS
     → Immediate (0-30 days)
     → Short-term (30-90 days)
     → Long-term (90+ days)
     → Strategic improvements

  10. APPENDICES
      → Detailed IOC list
      → Log excerpts
      → Screenshots
      → Tool output
      → Evidence inventory
```

---

## 3. Writing Best Practices

```
REPORT WRITING GUIDELINES:

CLARITY:
  ✓ Use clear, concise language
  ✓ Define technical terms
  ✓ Use consistent terminology
  ✓ One idea per paragraph
  ✓ Use bullet points for lists
  ✓ Include visual aids (diagrams, tables)
  ✗ Don't use jargon without explanation
  ✗ Don't write excessively long paragraphs
  ✗ Don't assume reader's technical level

ACCURACY:
  ✓ Support claims with evidence
  ✓ Reference specific log entries
  ✓ Include timestamps (UTC)
  ✓ Distinguish facts from analysis
  ✓ Note confidence levels
  ✓ Cite sources and tools used
  ✗ Don't speculate without stating so
  ✗ Don't overstate findings
  ✗ Don't omit relevant findings

PROFESSIONALISM:
  ✓ Objective tone
  ✓ Blameless language
  ✓ Focus on facts and process
  ✓ Proofread for errors
  ✓ Consistent formatting
  ✗ Don't assign blame to individuals
  ✗ Don't use emotional language
  ✗ Don't include unnecessary opinions

EXAMPLE — GOOD vs BAD:

  BAD:  "The user stupidly clicked a phishing link
         and got us all compromised."

  GOOD: "The initial compromise occurred when an
         employee opened a phishing email containing
         a credential-harvesting link. The email
         bypassed existing email filtering controls.
         This indicates a gap in email security
         that should be addressed."
```

---

## 4. Executive Summary Writing

```
EXECUTIVE SUMMARY TEMPLATE:

  ┌──────────────────────────────────────────────┐
  │ SECURITY INCIDENT REPORT                     │
  │ EXECUTIVE SUMMARY                            │
  │                                              │
  │ Incident: Data Breach via Phishing           │
  │ Severity: HIGH                               │
  │ Date Range: January 10-15, 2024             │
  │ Status: RESOLVED                             │
  │                                              │
  │ WHAT HAPPENED:                               │
  │ On January 10, a targeted phishing email     │
  │ was delivered to an employee, resulting in    │
  │ credential compromise. The attacker used     │
  │ stolen credentials to access internal        │
  │ systems and exfiltrate approximately 2.3 GB  │
  │ of data including customer records.          │
  │                                              │
  │ IMPACT:                                      │
  │ → 5 systems compromised                     │
  │ → ~15,000 customer records exposed          │
  │ → 3-day business disruption                 │
  │ → Estimated cost: $250,000                  │
  │                                              │
  │ KEY FINDINGS:                                │
  │ → Initial access: Phishing email            │
  │ → No MFA enabled on VPN/email               │
  │ → Flat network allowed lateral movement     │
  │ → Data exfiltrated to external cloud storage│
  │ → 3-day dwell time before detection         │
  │                                              │
  │ RECOMMENDATIONS:                             │
  │ 1. Deploy MFA for all remote access         │
  │ 2. Implement network segmentation           │
  │ 3. Enhance email security filtering         │
  │ 4. Deploy data loss prevention              │
  │ 5. Conduct employee awareness training      │
  │                                              │
  │ REGULATORY:                                  │
  │ Breach notification required under GDPR.    │
  │ Legal team coordinating 72-hour notice.     │
  └──────────────────────────────────────────────┘

TIPS FOR EXEC SUMMARY:
  → Lead with impact, not technical details
  → Use business language, not security jargon
  → Include financial impact
  → Be honest about what happened
  → Present clear recommendations
  → Keep to 1-2 pages maximum
```

---

## 5. Report Distribution and Handling

```
REPORT HANDLING:

CLASSIFICATION:
  → Mark report as CONFIDENTIAL
  → Limit distribution to need-to-know
  → Control copies (numbered if sensitive)
  → Encrypt electronic copies
  → Secure storage with access controls

DISTRIBUTION:
  Recipient        | Report Type    | Format
  CISO             | Full report    | Encrypted PDF
  CEO/Board        | Exec summary   | Encrypted PDF
  Legal            | Full + legal   | Encrypted PDF
  IT Management    | Technical      | Encrypted PDF
  SOC team         | Technical      | Internal wiki
  Regulators       | As required    | Secure channel
  Insurance        | Full + costs   | Encrypted PDF
  Law enforcement  | Forensic       | Secure transfer

RETENTION:
  → Keep incident reports per retention policy
  → Minimum 3-7 years (varies by regulation)
  → Store securely with access controls
  → Include in evidence archive
  → Available for future reference
```

---

## Summary Table

| Report Element | Purpose | Audience | Key Content |
|---------------|---------|----------|-------------|
| Exec Summary | Decision support | Leadership | Impact, recommendations |
| Timeline | What happened | All | Chronological events |
| Technical Analysis | Root cause | Technical | TTPs, evidence |
| Impact Assessment | Business effect | Management | Costs, scope |
| Recommendations | Prevention | All | Prioritized actions |

---

## Revision Questions

1. What are the different types of incident reports and their audiences?
2. What sections should a technical incident report include?
3. What makes an effective executive summary?
4. What writing best practices should be followed?
5. How should incident reports be handled and distributed?

---

*Previous: [01-lessons-learned.md](01-lessons-learned.md) | Next: [03-metrics-and-kpis.md](03-metrics-and-kpis.md)*

---

*[Back to README](../README.md)*
