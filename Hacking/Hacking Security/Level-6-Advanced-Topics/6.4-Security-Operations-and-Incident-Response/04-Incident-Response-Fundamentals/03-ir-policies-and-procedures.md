# Unit 4: Incident Response Fundamentals — Topic 3: IR Policies and Procedures

## Overview

**IR policies and procedures** provide the formal framework that governs how an organization detects, responds to, and recovers from security incidents. These documents establish authority, define processes, and ensure consistent incident handling across the organization.

---

## 1. IR Policy Framework

```
IR POLICY HIERARCHY:

  ┌──────────────────────────────────────┐
  │         IR POLICY                     │
  │  Executive-level, mandates IR        │
  │  Updated: Annually                    │
  └─────────────────┬────────────────────┘
                    │
  ┌─────────────────▼────────────────────┐
  │         IR PLAN                       │
  │  Detailed procedures, roles, phases  │
  │  Updated: Semi-annually              │
  └─────────────────┬────────────────────┘
                    │
  ┌─────────────────▼────────────────────┐
  │         PLAYBOOKS / RUNBOOKS          │
  │  Step-by-step for specific scenarios │
  │  Updated: Quarterly or as needed     │
  └─────────────────┬────────────────────┘
                    │
  ┌─────────────────▼────────────────────┐
  │         CHECKLISTS / TEMPLATES        │
  │  Quick-reference operational tools   │
  │  Updated: As needed                  │
  └──────────────────────────────────────┘

IR POLICY CONTENTS:
  → Purpose and scope
  → Definitions (incident, event, breach)
  → Authority and authorization
  → Roles and responsibilities
  → Incident classification
  → Reporting requirements
  → Escalation procedures
  → Communication guidelines
  → Legal and regulatory obligations
  → Training requirements
  → Policy review schedule
  → Compliance requirements
```

---

## 2. IR Plan Components

```
INCIDENT RESPONSE PLAN:

SECTION 1: OVERVIEW
  → Mission statement
  → Scope (what's covered)
  → Objectives
  → Applicable regulations
  → Related policies

SECTION 2: AUTHORITY
  → Who can declare an incident
  → Who can authorize containment
  → Who can communicate externally
  → Pre-authorized actions list
  → Escalation authority chain

SECTION 3: TEAM STRUCTURE
  → Core team members and contacts
  → Extended team and contacts
  → External resources (retainers)
  → On-call schedule
  → Escalation matrix

SECTION 4: INCIDENT CLASSIFICATION
  → Severity levels (Critical/High/Medium/Low)
  → Category definitions
  → Classification criteria
  → Priority matrix
  → Response time requirements

SECTION 5: PROCEDURES BY PHASE
  Preparation:
  → Tool readiness checklist
  → Training schedule
  → Exercise plan
  
  Detection/Analysis:
  → Monitoring procedures
  → Triage workflow
  → Evidence collection steps
  → Analysis methodology
  
  Containment:
  → Pre-authorized containment actions
  → Containment decision matrix
  → Evidence preservation requirements
  
  Eradication/Recovery:
  → Eradication procedures
  → Recovery steps
  → Validation checklist
  
  Post-Incident:
  → Lessons learned process
  → Report template
  → Improvement tracking

SECTION 6: COMMUNICATION
  → Internal notification matrix
  → External notification requirements
  → Regulatory notification timelines
  → Media handling procedures
  → Customer notification templates

SECTION 7: EVIDENCE HANDLING
  → Evidence collection procedures
  → Chain of custody forms
  → Storage requirements
  → Retention periods
  → Legal hold procedures
```

---

## 3. Standard Operating Procedures

```
SOPs FOR COMMON SCENARIOS:

SOP: MALWARE INCIDENT
  1. Isolate affected system from network
  2. Do NOT power off (preserve memory)
  3. Capture memory image if possible
  4. Identify malware (hash, name, family)
  5. Determine infection vector
  6. Scan for lateral spread
  7. Block IOCs (hash, C2 IP/domain)
  8. Clean or reimage affected systems
  9. Reset compromised credentials
  10. Monitor for reinfection

SOP: PHISHING INCIDENT
  1. Identify all recipients
  2. Determine who opened/clicked
  3. Check email gateway for similar messages
  4. Block sender/domain
  5. Remove emails from all mailboxes
  6. Assess if credentials were compromised
  7. Reset passwords if needed
  8. Block malicious URLs/domains
  9. Scan endpoints of affected users
  10. Send awareness notification

SOP: ACCOUNT COMPROMISE
  1. Disable compromised account
  2. Revoke all active sessions/tokens
  3. Review account activity logs
  4. Identify unauthorized actions
  5. Check for persistence (forwarding rules, 
     delegated access, OAuth apps)
  6. Reset password and MFA
  7. Review and remove unauthorized changes
  8. Re-enable with enhanced monitoring
  9. Notify user and manager
  10. Document timeline and actions

SOP: RANSOMWARE
  1. Isolate affected systems IMMEDIATELY
  2. Identify ransomware variant
  3. Determine encryption scope
  4. Preserve encrypted samples
  5. Assess backup availability
  6. Engage legal counsel
  7. Notify executive leadership
  8. Contact law enforcement
  9. Do NOT pay ransom without legal advice
  10. Begin recovery from clean backups
  11. Rebuild affected systems
  12. Harden to prevent recurrence
```

---

## 4. Documentation Standards

```
INCIDENT DOCUMENTATION:

INCIDENT TICKET TEMPLATE:
  ┌──────────────────────────────────────┐
  │ INCIDENT RECORD                      │
  │                                      │
  │ ID: INC-2024-001                    │
  │ Title: Account Compromise           │
  │ Severity: HIGH                       │
  │ Status: Investigating               │
  │ Declared: 2024-01-15 10:30 UTC      │
  │                                      │
  │ Category: Unauthorized Access        │
  │ Affected Systems: mail.company.com  │
  │ Affected Users: jsmith@company.com  │
  │ Data Impact: Email access (TBD)     │
  │                                      │
  │ IC: Jane Doe (IR Manager)           │
  │ Analyst: John Smith                  │
  │ War Room: Teams Channel #INC-001    │
  │                                      │
  │ TIMELINE:                           │
  │ 10:00 - Alert triggered (SIEM)      │
  │ 10:15 - Tier 1 triage, escalated    │
  │ 10:30 - Incident declared           │
  │ 10:45 - Account disabled            │
  │ 11:00 - Investigation started       │
  │                                      │
  │ ACTIONS TAKEN:                      │
  │ - Account disabled                   │
  │ - Sessions revoked                   │
  │ - Log analysis in progress           │
  │                                      │
  │ NEXT STEPS:                         │
  │ - Complete log review                │
  │ - Determine data access scope        │
  │ - Assess lateral movement            │
  └──────────────────────────────────────┘

DOCUMENTATION BEST PRACTICES:
  → Timestamp every action
  → Record who performed each action
  → Include evidence references
  → Use objective language
  → Avoid assumptions in notes
  → Update in real-time
  → Maintain chain of custody
  → Secure incident records
```

---

## 5. Policy Maintenance

```
POLICY LIFECYCLE:

  ┌───────────┐
  │  CREATE   │ ← Initial development
  └─────┬─────┘
        │
  ┌─────▼─────┐
  │  APPROVE  │ ← Executive sign-off
  └─────┬─────┘
        │
  ┌─────▼─────┐
  │ IMPLEMENT │ ← Train team, deploy
  └─────┬─────┘
        │
  ┌─────▼─────┐
  │  EXERCISE │ ← Test with tabletop/live
  └─────┬─────┘
        │
  ┌─────▼─────┐
  │  REVIEW   │ ← Annual review
  └─────┬─────┘
        │
  ┌─────▼─────┐
  │  UPDATE   │ ← Based on lessons learned
  └─────┬─────┘
        │
        └──────▶ (cycle repeats)

REVIEW TRIGGERS:
  → Annual scheduled review
  → After major incident
  → After organizational change
  → After regulatory change
  → After tabletop exercise
  → After audit findings
  → After technology changes

COMPLIANCE REQUIREMENTS:
  Framework    | IR Requirement
  PCI DSS      | Req 12.10: IR plan, testing
  HIPAA        | § 164.308(a)(6): IR procedures
  NIST CSF     | RS.RP: Response planning
  SOC 2        | CC7.3-CC7.5: Response, mitigation
  GDPR         | Art 33: 72-hour breach notification
  ISO 27001    | A.16: Information security incident mgmt
```

---

## Summary Table

| Document | Purpose | Update Frequency |
|----------|---------|-----------------|
| IR Policy | Authority and mandate | Annually |
| IR Plan | Detailed procedures | Semi-annually |
| Playbooks | Scenario-specific steps | Quarterly |
| SOPs | Operational procedures | As needed |
| Templates | Standardized forms | As needed |
| Contact Lists | Emergency contacts | Quarterly |

---

## Revision Questions

1. What is the hierarchy of IR documentation?
2. What should an IR plan's authority section include?
3. What are the key steps in a ransomware SOP?
4. What should be documented in an incident ticket?
5. What regulatory frameworks require formal IR plans?

---

*Previous: [02-ir-team-structure.md](02-ir-team-structure.md) | Next: [04-communication-plans.md](04-communication-plans.md)*

---

*[Back to README](../README.md)*
