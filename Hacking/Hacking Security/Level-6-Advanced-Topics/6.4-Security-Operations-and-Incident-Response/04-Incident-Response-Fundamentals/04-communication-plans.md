# Unit 4: Incident Response Fundamentals — Topic 4: Communication Plans

## Overview

**Communication plans** define how information flows during security incidents — internally to stakeholders and externally to customers, regulators, and the public. Poor communication during incidents amplifies damage, erodes trust, and can create legal liability. Effective communication is as critical as technical response.

---

## 1. Communication Framework

```
INCIDENT COMMUNICATION STRUCTURE:

  ┌──────────────────────────────────────────┐
  │         INCIDENT COMMANDER               │
  │         (Communication Authority)         │
  └─────────────────┬────────────────────────┘
                    │
       ┌────────────┼────────────────┐
       │            │                │
  ┌────▼────┐ ┌────▼────┐ ┌────────▼────────┐
  │INTERNAL │ │EXTERNAL │ │ REGULATORY      │
  │         │ │         │ │                 │
  │Executive│ │Customers│ │Data protection  │
  │IT teams │ │Partners │ │Law enforcement  │
  │Legal    │ │Media    │ │Industry ISAC    │
  │HR       │ │Public   │ │Cyber insurance  │
  │All staff│ │Vendors  │ │Regulatory bodies│
  └─────────┘ └─────────┘ └─────────────────┘

COMMUNICATION PRINCIPLES:
  → Timely: Share information promptly
  → Accurate: Only share verified information
  → Consistent: One source of truth
  → Need-to-know: Limit sensitive details
  → Documented: Record all communications
  → Secure: Use encrypted channels
  → Coordinated: No unauthorized statements
```

---

## 2. Internal Communications

```
INTERNAL NOTIFICATION MATRIX:

  Severity  | Notify              | Within
  Critical  | CISO, CEO, CTO,     | 15 minutes
            | Legal, Board        |
  High      | CISO, IT Director,  | 1 hour
            | Legal               |
  Medium    | SOC Manager, IT Ops | 4 hours
  Low       | SOC Manager         | Next business day

EXECUTIVE BRIEFING TEMPLATE:
  ┌──────────────────────────────────────────┐
  │ EXECUTIVE INCIDENT BRIEFING              │
  │                                          │
  │ Incident ID: INC-2024-001              │
  │ Severity: CRITICAL                       │
  │ Status: Active - Containment Phase      │
  │ Time: 2024-01-15 14:00 UTC              │
  │                                          │
  │ SITUATION:                               │
  │ Ransomware detected on 15 endpoints in  │
  │ the finance department. Encryption is    │
  │ actively spreading.                      │
  │                                          │
  │ ACTIONS TAKEN:                          │
  │ - Affected network segment isolated     │
  │ - All finance endpoints disconnected    │
  │ - IR team fully activated               │
  │ - External IR firm engaged              │
  │                                          │
  │ IMPACT:                                 │
  │ - Finance operations halted             │
  │ - No evidence of data exfiltration yet  │
  │ - ERP system unaffected                 │
  │                                          │
  │ NEXT STEPS:                             │
  │ - Complete containment by 16:00         │
  │ - Assess backup recovery timeline       │
  │ - Legal to assess notification needs    │
  │                                          │
  │ NEXT UPDATE: 16:00 UTC                  │
  │ CONTACT: Jane Doe, IR Manager           │
  │ WAR ROOM: Teams #incident-response      │
  └──────────────────────────────────────────┘

INTERNAL STATUS UPDATES:
  → Use regular cadence (every 2-4 hours for critical)
  → Keep messages brief and factual
  → Include: Status, actions, next update time
  → Use secure channels only
  → Avoid speculation or blame
```

---

## 3. External Communications

```
EXTERNAL COMMUNICATION:

CUSTOMER NOTIFICATION:
  When Required:
  → Personal data compromised
  → Service disruption affecting customers
  → Regulatory requirement
  → Contractual obligation
  
  Template:
  "We are writing to inform you of a security
  incident that may have affected your data.
  
  What happened: [brief description]
  When: [date/time range]
  What data was involved: [specific types]
  What we're doing: [response actions]
  What you should do: [recommended actions]
  How to contact us: [support details]"

MEDIA COMMUNICATIONS:
  → Designate single spokesperson
  → Prepare holding statement
  → Do NOT speculate about cause or scope
  → Acknowledge the incident factually
  → Emphasize response actions
  → Avoid technical jargon
  → Legal review before release
  
  Holding Statement:
  "We are aware of a security incident and are
  actively investigating. We have engaged 
  cybersecurity experts and are working to
  resolve the situation. We will provide updates
  as more information becomes available."

PARTNER/VENDOR NOTIFICATION:
  → Notify partners at risk
  → Share relevant IOCs
  → Coordinate response if shared infrastructure
  → Document notifications
  → Follow contractual requirements

REGULATORY NOTIFICATION:
  Regulation | Timeframe        | Authority
  GDPR       | 72 hours         | Data Protection Authority
  HIPAA      | 60 days          | HHS/OCR
  PCI DSS    | Varies           | Card brands
  State laws | 30-60 days       | State AG
  SEC        | 4 business days  | SEC
  NIS2       | 24 hours initial | National CSIRT
```

---

## 4. Communication Security

```
SECURE COMMUNICATIONS:

OUT-OF-BAND COMMUNICATION:
  → Assume primary channels may be compromised
  → Use separate communication channels
  → Not accessible to the attacker
  
  Options:
  → Personal cell phones (Signal/WhatsApp)
  → Separate email accounts (not corporate)
  → War room conference bridge
  → Physical meetings
  → Dedicated incident Slack/Teams workspace
  → Encrypted messaging apps

INFORMATION CLASSIFICATION:
  Level           | Share With         | Example
  Restricted      | Core IR team only  | Technical IOCs
  Confidential    | Extended team      | Scope, impact
  Internal        | All employees      | Status updates
  Public          | Anyone             | Press statement

OPSEC DURING INCIDENTS:
  [ ] Don't discuss on monitored channels
  [ ] Don't email details over compromised systems
  [ ] Don't share IOCs publicly during investigation
  [ ] Verify identity before sharing sensitive info
  [ ] Encrypt files containing incident data
  [ ] Limit war room access
  [ ] Sanitize screenshots before sharing
  [ ] Use code names for sensitive incidents
```

---

## 5. Communication Templates

```
TEMPLATES LIBRARY:

INCIDENT DECLARATION:
  "A security incident (INC-YYYY-NNN) has been declared
  at [severity] level. The IR team has been activated.
  Please direct all information to the designated 
  incident channel. Do not discuss externally."

STATUS UPDATE:
  "INC-YYYY-NNN Update #N ([timestamp])
  Status: [Active/Contained/Resolved]
  Summary: [1-2 sentence update]
  Actions since last update: [bullet points]
  Next steps: [planned actions]
  Next update: [time]"

RESOLUTION NOTICE:
  "INC-YYYY-NNN has been resolved at [timestamp].
  Duration: [X hours/days]. Root cause: [brief].
  No further immediate action required.
  Post-incident review scheduled for [date]."

ALL-HANDS NOTIFICATION:
  "Our security team has identified and is responding
  to a security incident. The affected [systems/services]
  are [status]. Your action required: [specific steps].
  This is being handled by the security team. Please
  do not take independent action."

COMMUNICATION CHECKLIST:
  [ ] Communication leads identified
  [ ] Secure channels established
  [ ] Notification matrix followed
  [ ] Executive briefing delivered
  [ ] Legal consulted on external comms
  [ ] Regulatory notification assessed
  [ ] Customer notification drafted (if needed)
  [ ] Media holding statement prepared
  [ ] All communications documented
  [ ] Post-incident communication plan
```

---

## Summary Table

| Audience | When to Notify | Channel | Content Level |
|----------|---------------|---------|--------------|
| Executive | Critical/High incidents | Secure briefing | Strategic impact |
| IT Teams | All incidents | Internal chat | Technical details |
| Legal | Data breaches, criminal | Direct communication | Full details |
| Customers | Data compromise | Email/letter | Impact + actions |
| Regulators | Per regulation | Official channels | Required details |
| Media | Public incidents | Press release | Approved statement |

---

## Revision Questions

1. What are the key principles of incident communication?
2. What should an executive incident briefing include?
3. When is customer notification required during incidents?
4. Why is out-of-band communication important during incidents?
5. What regulatory notification timelines must organizations follow?

---

*Previous: [03-ir-policies-and-procedures.md](03-ir-policies-and-procedures.md) | Next: [05-legal-considerations.md](05-legal-considerations.md)*

---

*[Back to README](../README.md)*
