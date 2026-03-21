# Unit 4: Incident Response Fundamentals — Topic 1: IR Lifecycle (NIST/SANS)

## Overview

The **Incident Response (IR) lifecycle** provides a structured approach to handling security incidents from detection through resolution and improvement. The two most widely adopted frameworks — NIST SP 800-61 and SANS — define phases that guide organizations through effective incident handling.

---

## 1. NIST IR Lifecycle

```
NIST SP 800-61 Rev. 2: INCIDENT HANDLING LIFECYCLE

  ┌─────────────────────────────────────────────────┐
  │                                                  │
  │   ┌──────────────┐     ┌──────────────────────┐ │
  │   │ 1. PREPARE   │────▶│ 2. DETECT & ANALYZE  │ │
  │   │              │     │                      │ │
  │   │ Plans        │     │ Monitor              │ │
  │   │ Tools        │     │ Triage               │ │
  │   │ Training     │     │ Investigate          │ │
  │   └──────────────┘     └──────────┬───────────┘ │
  │          ▲                        │              │
  │          │                        ▼              │
  │   ┌──────┴───────┐     ┌──────────────────────┐ │
  │   │ 4. POST-     │◀────│ 3. CONTAIN, ERAD,    │ │
  │   │    INCIDENT  │     │    RECOVER           │ │
  │   │              │     │                      │ │
  │   │ Lessons      │     │ Contain threat       │ │
  │   │ Improve      │     │ Remove threat        │ │
  │   │ Report       │     │ Restore systems      │ │
  │   └──────────────┘     └──────────────────────┘ │
  │                                                  │
  │   ← Continuous cycle of improvement →            │
  └─────────────────────────────────────────────────┘

NIST PHASES DETAIL:

PHASE 1: PREPARATION
  → IR policy and plan
  → Team formation and training
  → Tool acquisition and configuration
  → Communication procedures
  → Documentation templates
  → Relationships (legal, HR, PR, law enforcement)

PHASE 2: DETECTION AND ANALYSIS
  → Monitor security events
  → Triage alerts
  → Validate incidents
  → Determine scope and impact
  → Collect and preserve evidence
  → Document findings
  → Prioritize based on impact

PHASE 3: CONTAINMENT, ERADICATION, AND RECOVERY
  → Containment: Stop the spread
  → Eradication: Remove the threat
  → Recovery: Restore operations
  → Validation: Verify systems are clean
  → Monitor for recurrence

PHASE 4: POST-INCIDENT ACTIVITY
  → Lessons learned meeting
  → IR report writing
  → Process improvement
  → Update detection rules
  → Knowledge base update
```

---

## 2. SANS IR Framework

```
SANS INCIDENT RESPONSE FRAMEWORK (PICERL):

  P → Preparation
  I → Identification
  C → Containment
  E → Eradication
  R → Recovery
  L → Lessons Learned

  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐
  │  P  │─▶│  I  │─▶│  C  │─▶│  E  │─▶│  R  │─▶│  L  │
  │     │  │     │  │     │  │     │  │     │  │     │
  │Prep │  │Ident│  │Cont │  │Erad │  │Recv │  │Less │
  │     │  │ify  │  │ain  │  │icate│  │over │  │ons  │
  └─────┘  └─────┘  └─────┘  └─────┘  └─────┘  └─────┘

NIST vs SANS COMPARISON:
  NIST Phase                    | SANS Phase
  1. Preparation                | 1. Preparation
  2. Detection and Analysis     | 2. Identification
  3. Containment, Erad, Recovery| 3. Containment
                                | 4. Eradication
                                | 5. Recovery
  4. Post-Incident Activity     | 6. Lessons Learned

KEY DIFFERENCES:
  → NIST: 4 phases (combines C, E, R)
  → SANS: 6 phases (separates C, E, R)
  → Both emphasize continuous improvement
  → Both are widely accepted standards
  → NIST is more detailed in documentation
  → SANS is more practical/tactical
```

---

## 3. Incident Classification

```
INCIDENT CATEGORIES (NIST):

  Category              | Example
  Unauthorized Access   | Account compromise, system breach
  Denial of Service     | DDoS, service disruption
  Malicious Code        | Malware, ransomware, virus
  Improper Usage        | Policy violation, insider abuse
  Scans/Probes          | Port scanning, vulnerability scanning
  Investigation         | Forensic or legal investigation

INCIDENT SEVERITY:

  Level    | Description              | Impact
  Critical | Active breach, data loss | Business-stopping
  High     | Confirmed compromise     | Significant impact
  Medium   | Suspicious activity      | Moderate impact
  Low      | Minor policy violation   | Minimal impact
  Info     | No immediate threat      | No impact

SEVERITY DETERMINATION FACTORS:
  → Current impact on business operations
  → Potential future impact
  → Number of systems/users affected
  → Data sensitivity involved
  → Regulatory implications
  → Recoverability (ease of recovery)
  → Threat actor sophistication

INCIDENT vs EVENT vs ALERT:
  EVENT: Any observable occurrence in a system
  → Example: User logged in

  ALERT: An event that matches a detection rule
  → Example: Failed login threshold exceeded

  INCIDENT: A confirmed security violation
  → Example: Attacker gained unauthorized access
  → Requires formal IR process

  Flow: Events → Alerts → Investigation → Incident
```

---

## 4. Incident Prioritization

```
PRIORITIZATION MATRIX:

  Impact ↓ / Urgency →  | High      | Medium   | Low
  High                   | Critical  | High     | Medium
  Medium                 | High      | Medium   | Low
  Low                    | Medium    | Low      | Info

FUNCTIONAL IMPACT:
  Category | Description
  None     | No effect on ability to provide services
  Low      | Minimal effect, workarounds available
  Medium   | Services affected for subset of users
  High     | Unable to provide critical services

INFORMATION IMPACT:
  Category          | Description
  None              | No information compromised
  Privacy Breach    | PII exposed
  Proprietary       | Trade secrets/IP exposed
  Integrity Loss    | Data modified without authorization
  Confidentiality   | Sensitive data accessed by unauthorized

RECOVERABILITY:
  Category      | Description
  Regular       | Recoverable with existing resources
  Supplemented  | Needs additional resources
  Extended      | Time to recover is unpredictable
  Not Recoverable| Cannot recover (data permanently lost)

RESPONSE TIMES BY PRIORITY:
  Priority  | Initial Response | Containment | Resolution
  Critical  | 15 minutes       | 1 hour      | 24 hours
  High      | 1 hour           | 4 hours     | 72 hours
  Medium    | 4 hours          | 24 hours    | 1 week
  Low       | Next business day| 72 hours    | 30 days
```

---

## 5. IR Lifecycle in Practice

```
REAL-WORLD IR FLOW:

  DAY 1: DETECTION
  10:00 - SIEM alert: Multiple failed logins from
          unusual IP, followed by successful login
  10:15 - Tier 1 analyst triages, escalates to Tier 2
  10:30 - Tier 2 validates: Confirmed account compromise
  10:45 - Incident declared: INC-2024-001 (HIGH)
  11:00 - IR team activated, War room established

  DAY 1: INVESTIGATION
  11:30 - Scope assessment: 1 user account compromised
  12:00 - Evidence: Attacker accessed email, SharePoint
  13:00 - Additional compromise found: 2 more accounts
  14:00 - Scope expanded to HIGH-CRITICAL
  15:00 - Data access audit: Sensitive docs accessed

  DAY 1-2: CONTAINMENT
  15:30 - Compromised accounts disabled
  16:00 - MFA enforced for all admin accounts
  17:00 - Attacker IP blocked at firewall
  18:00 - Session tokens revoked
  Day 2  - Monitoring for lateral movement

  DAY 2-3: ERADICATION
  - Password resets for all affected accounts
  - Review of any persistence mechanisms
  - Removal of unauthorized access

  DAY 3-5: RECOVERY
  - Accounts re-enabled with new credentials
  - MFA enrollment verified
  - Enhanced monitoring in place
  - Systems validated clean

  DAY 10: POST-INCIDENT
  - Lessons learned meeting
  - Final incident report
  - Detection rule improvements
  - Policy updates recommended

CRITICAL SUCCESS FACTORS:
  [ ] Pre-established IR plan
  [ ] Clear escalation procedures
  [ ] Pre-authorized containment actions
  [ ] Communication templates ready
  [ ] Evidence collection procedures known
  [ ] Legal counsel engaged early
  [ ] Management informed promptly
  [ ] Post-incident review conducted
```

---

## Summary Table

| Phase (NIST) | Phase (SANS) | Key Activities |
|-------------|-------------|----------------|
| Preparation | Preparation | Plan, train, tool up |
| Detection & Analysis | Identification | Monitor, triage, validate |
| Containment, Erad, Recovery | Containment | Stop the spread |
| (included above) | Eradication | Remove the threat |
| (included above) | Recovery | Restore operations |
| Post-Incident | Lessons Learned | Improve processes |

---

## Revision Questions

1. What are the four phases of the NIST incident response lifecycle?
2. How does the SANS PICERL framework differ from NIST?
3. How should incidents be classified by severity?
4. What factors determine incident prioritization?
5. What does a real-world incident response timeline look like?

---

*Previous: None (First topic in this unit) | Next: [02-ir-team-structure.md](02-ir-team-structure.md)*

---

*[Back to README](../README.md)*
