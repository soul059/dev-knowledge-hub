# Unit 5: Preparation Phase — Topic 1: IR Plan Development

## Overview

**IR plan development** is the foundational activity of incident response preparation. A well-developed IR plan ensures the organization can respond effectively, efficiently, and consistently to security incidents. The plan documents authority, procedures, roles, and resources needed for every phase of incident handling.

---

## 1. Plan Development Process

```
IR PLAN DEVELOPMENT LIFECYCLE:

  ┌────────────┐  ┌──────────┐  ┌────────────┐
  │ Assess     │─▶│ Design   │─▶│ Document   │
  │ Current    │  │ Plan     │  │ Plan       │
  │ State      │  │ Structure│  │            │
  └────────────┘  └──────────┘  └─────┬──────┘
                                      │
  ┌────────────┐  ┌──────────┐  ┌─────▼──────┐
  │ Maintain   │◀─│ Test     │◀─│ Approve &  │
  │ & Update   │  │ Plan     │  │ Implement  │
  └────────────┘  └──────────┘  └────────────┘

ASSESSMENT PHASE:
  → Identify organizational assets
  → Determine threat landscape
  → Review regulatory requirements
  → Assess current IR capabilities
  → Identify gaps and deficiencies
  → Benchmark against frameworks (NIST, SANS)

PLAN DOCUMENT STRUCTURE:
  1. Executive Summary
  2. Purpose, Scope, and Objectives
  3. Authority and Governance
  4. Incident Response Team
  5. Incident Classification
  6. Response Procedures (by phase)
  7. Communication Plan
  8. Evidence Handling
  9. Reporting Requirements
  10. Training and Exercises
  11. Plan Maintenance
  Appendices:
  A. Contact Lists
  B. Escalation Matrix
  C. Templates and Forms
  D. Tool Inventory
  E. Third-Party Agreements
```

---

## 2. Key Plan Components

```
CRITICAL PLAN SECTIONS:

SCOPE:
  → Systems and networks covered
  → Types of incidents covered
  → Geographic scope
  → Organizational units included
  → Exclusions (if any)

AUTHORITY:
  Pre-Authorized Actions:
  → Isolate endpoints from network
  → Disable user accounts
  → Block IP addresses at firewall
  → Quarantine email messages
  → Take forensic images
  → Engage external IR firm (retainer)
  
  Requires Approval:
  → Shutdown production systems
  → Engage law enforcement
  → External communications
  → Paying ransom
  → Restoring from backup (production)
  → Hiring additional external resources

CLASSIFICATION:
  Category          | Examples
  Malware           | Ransomware, trojan, worm
  Unauthorized Access| Account compromise, system breach
  Data Breach       | PII exposure, data theft
  DoS/DDoS          | Service disruption
  Insider Threat    | Data theft, sabotage
  Social Engineering| Phishing, vishing, pretexting
  Web App Attack    | SQL injection, XSS
  Supply Chain      | Third-party compromise

RESOURCE REQUIREMENTS:
  → Forensic workstations
  → Jump kits (portable IR kits)
  → Network taps/captures
  → Forensic software licenses
  → Secure communication tools
  → Documentation systems
  → Evidence storage
  → War room access
```

---

## 3. Jump Kit and Resources

```
IR JUMP KIT:

HARDWARE:
  → Forensic laptop (clean, dedicated)
  → Write blockers (hardware)
  → External hard drives (sanitized)
  → USB drives (bootable forensic OS)
  → Network cables and adapters
  → Network tap (passive capture)
  → Labels, markers, evidence bags
  → Camera for photographing evidence
  → Portable printer

SOFTWARE:
  → Forensic imaging (FTK Imager, dd)
  → Memory capture (WinPmem, DumpIt)
  → Analysis (Autopsy, Volatility)
  → Network capture (Wireshark, tcpdump)
  → Malware analysis (sandbox, YARA)
  → Log analysis (grep, jq, ELK)
  → Password tools (hashcat, john)
  → Secure communication (Signal)

DOCUMENTATION:
  → IR plan (printed copy)
  → Contact lists
  → Chain of custody forms
  → Evidence labels
  → Incident report templates
  → Network diagrams
  → System inventory
  → Playbook quick references

CHECKLIST:
  [ ] Jump kit assembled and inventoried
  [ ] Software licenses current
  [ ] Hardware tested and functional
  [ ] Documentation up to date
  [ ] Stored in accessible location
  [ ] Reviewed and restocked quarterly
```

---

## 4. Third-Party Integration

```
EXTERNAL RESOURCES:

IR RETAINER FIRMS:
  → Pre-negotiated engagement
  → Guaranteed response time (2-4 hours)
  → Rate card agreed in advance
  → Scope of services defined
  → NDA and legal agreements
  → Key providers: CrowdStrike, Mandiant, 
    Secureworks, Unit42, Kroll, NCC Group

CYBER INSURANCE:
  → Incident reporting procedures
  → Approved vendor list
  → Coverage scope documented
  → Claims process understood
  → Policy limits known
  → Premium impact considerations

LAW ENFORCEMENT CONTACTS:
  → Local FBI field office
  → Secret Service (financial)
  → CISA (critical infrastructure)
  → State police cyber unit
  → Pre-established relationships

INDUSTRY SHARING:
  → ISAC membership (sector-specific)
  → Threat intel sharing agreements
  → Peer organization contacts
  → Law enforcement partnerships

INTEGRATION CHECKLIST:
  [ ] IR retainer in place and tested
  [ ] Cyber insurance policy reviewed
  [ ] Law enforcement contacts established
  [ ] ISAC membership active
  [ ] Legal counsel identified
  [ ] PR firm/contact identified
  [ ] Backup communication providers
```

---

## 5. Plan Validation

```
PLAN VALIDATION:

VALIDATION METHODS:
  → Document review (completeness check)
  → Tabletop exercise (scenario walkthrough)
  → Functional exercise (partial simulation)
  → Full-scale exercise (realistic simulation)
  → Actual incident review (post-mortem)

DOCUMENT REVIEW CHECKLIST:
  [ ] All required sections present
  [ ] Contact information current
  [ ] Procedures align with current environment
  [ ] Regulatory requirements addressed
  [ ] Escalation paths clear and current
  [ ] Templates usable and up to date
  [ ] External agreements current
  [ ] Tools and technologies accurate
  [ ] Training requirements documented
  [ ] Review schedule established

COMMON PLAN FAILURES:
  → Outdated contact information
  → Procedures don't match reality
  → Unclear authority and decision-making
  → Missing communication procedures
  → No evidence handling guidance
  → Untested backup/recovery procedures
  → No external resource agreements
  → Team unfamiliar with plan contents
  → Plan too long/complex to use in crisis

SUCCESS FACTORS:
  → Executive sponsorship
  → Regular updates and testing
  → Team familiarity through training
  → Integration with business continuity
  → Lessons learned incorporation
  → Practical and actionable format
  → Available offline/printed
```

---

## Summary Table

| Component | Purpose | Update Frequency |
|-----------|---------|-----------------|
| IR Plan | Core response framework | Semi-annually |
| Jump Kit | Ready-to-deploy tools | Quarterly |
| Contact List | Emergency contacts | Monthly |
| Retainer Agreements | External support | Annually |
| Templates | Standardized forms | As needed |

---

## Revision Questions

1. What are the key sections of an IR plan?
2. What actions should be pre-authorized vs. require approval?
3. What should an IR jump kit contain?
4. Why are IR retainer agreements important?
5. What are common reasons IR plans fail during real incidents?

---

*Previous: None (First topic in this unit) | Next: [02-playbook-creation.md](02-playbook-creation.md)*

---

*[Back to README](../README.md)*
