# Unit 4: Incident Response Fundamentals — Topic 2: IR Team Structure

## Overview

An effective **Incident Response team** requires clearly defined roles, responsibilities, and organizational structure. The team must include technical responders, management support, and cross-functional stakeholders who coordinate during incidents. A well-structured IR team is the difference between a managed incident and a crisis.

---

## 1. IR Team Models

```
IR TEAM ORGANIZATIONAL MODELS:

CENTRALIZED IR TEAM:
  → Single team handles all incidents
  → Dedicated full-time IR staff
  → Consistent processes
  → Best for: Large organizations

DISTRIBUTED IR TEAM:
  → IR responsibilities distributed across teams
  → Coordinated by central point
  → Part-time IR duties
  → Best for: Geographically distributed orgs

HYBRID MODEL:
  → Core team + extended team
  → Core: Full-time IR specialists
  → Extended: IT, Legal, HR, Comms (on-call)
  → Most common model

  ┌──────────────────────────────────────────┐
  │            HYBRID IR STRUCTURE            │
  │                                           │
  │  ┌──────────────────────────────────────┐│
  │  │         CORE IR TEAM                 ││
  │  │  IR Manager │ IR Analysts │ Forensics││
  │  └──────────────────────────────────────┘│
  │                    │                      │
  │  ┌────────────────┬┴──────────────┐      │
  │  │                │               │      │
  │  ▼                ▼               ▼      │
  │  ┌─────┐    ┌──────────┐    ┌────────┐  │
  │  │ IT  │    │ Security │    │ Legal  │  │
  │  │ Ops │    │ Ops/SOC  │    │ HR     │  │
  │  │     │    │          │    │ Comms  │  │
  │  └─────┘    └──────────┘    └────────┘  │
  │                                           │
  │  EXTENDED TEAM (activated as needed)      │
  └──────────────────────────────────────────┘
```

---

## 2. Core IR Roles

```
IR TEAM ROLES:

IR MANAGER / INCIDENT COMMANDER:
  → Leads the IR effort
  → Makes strategic decisions
  → Manages resources and priorities
  → Coordinates with stakeholders
  → Approves containment actions
  → Manages communications
  → Ensures documentation
  → Reports to executive leadership

INCIDENT HANDLER / LEAD ANALYST:
  → Leads technical investigation
  → Coordinates analysis activities
  → Determines scope and impact
  → Guides containment strategy
  → Assigns tasks to team members
  → Technical decision-making
  → Evidence oversight

IR ANALYST:
  → Performs log analysis
  → Conducts endpoint investigation
  → Collects and preserves evidence
  → Develops timeline
  → Identifies indicators of compromise
  → Executes containment actions
  → Documents technical findings

FORENSIC ANALYST:
  → Disk/memory forensics
  → Malware analysis
  → Evidence preservation
  → Chain of custody management
  → Forensic report writing
  → Expert witness support
  → Tool development

THREAT INTELLIGENCE ANALYST:
  → Research threat actors
  → Provide IOCs and context
  → Attribute attacks
  → Track campaigns
  → Brief IR team on threats
  → Inform detection improvements

COMMUNICATIONS LEAD:
  → Internal communications
  → External communications (if needed)
  → Media coordination
  → Customer notification
  → Regulatory notification
  → Status updates
```

---

## 3. Extended Team and Stakeholders

```
EXTENDED TEAM MEMBERS:

IT OPERATIONS:
  → System administration support
  → Network configuration changes
  → Firewall rule updates
  → Account management
  → System restoration
  → Backup/recovery
  Activation: Most incidents

LEGAL COUNSEL:
  → Legal implications assessment
  → Regulatory notification requirements
  → Law enforcement coordination
  → Evidence preservation guidance
  → Liability assessment
  → Contract review (breach notification)
  Activation: Medium+ incidents, data breaches

HUMAN RESOURCES:
  → Insider threat incidents
  → Employee investigations
  → Disciplinary actions
  → Policy enforcement
  → Privacy considerations
  Activation: Insider-related incidents

PUBLIC RELATIONS / COMMUNICATIONS:
  → Press release coordination
  → Media response
  → Customer communications
  → Social media monitoring
  → Brand protection
  Activation: Public-facing incidents

EXECUTIVE MANAGEMENT:
  → Strategic decision-making
  → Resource authorization
  → Business continuity decisions
  → External relationship management
  → Final authority on critical decisions
  Activation: Critical incidents

EXTERNAL RESOURCES:
  Resource              | When to Engage
  IR retainer firm      | Complex/large incidents
  Forensic specialists  | Legal proceedings
  Law enforcement       | Criminal activity
  Cyber insurance       | Breach with financial impact
  Regulatory bodies     | Compliance requirements
  Industry ISAC         | Threat sharing
```

---

## 4. On-Call and Escalation

```
ON-CALL STRUCTURE:

ON-CALL ROTATION:
  → Primary on-call (first responder)
  → Secondary on-call (backup)
  → Weekly or bi-weekly rotation
  → Defined response time SLA
  → After-hours compensation
  → Clear escalation path

  Week 1: Alice (primary) / Bob (secondary)
  Week 2: Bob (primary) / Charlie (secondary)
  Week 3: Charlie (primary) / Alice (secondary)

ESCALATION PROCEDURE:
  ┌──────────┐
  │ Alert    │
  │ Generated│
  └────┬─────┘
       │ 15 min
  ┌────▼─────┐
  │ SOC Tier │──── FP → Close
  │ 1 Triage │
  └────┬─────┘
       │ Confirmed
  ┌────▼─────┐
  │ SOC Tier │──── Contained → Close
  │ 2 Invest │
  └────┬─────┘
       │ Complex/Critical
  ┌────▼─────────┐
  │ IR Team      │──── Activate war room
  │ Activated    │     Notify management
  └────┬─────────┘
       │ Major incident
  ┌────▼─────────┐
  │ Executive    │──── Legal, PR, external
  │ Notification │     Business decisions
  └──────────────┘

WAR ROOM SETUP:
  → Dedicated physical or virtual space
  → Secure communication channels
  → Whiteboard/collaboration tools
  → Access to all required systems
  → Bridge line for remote participants
  → Regular status update cadence
  → Controlled access (need to know)
```

---

## 5. Team Development

```
BUILDING IR CAPABILITY:

HIRING PRIORITIES:
  1. IR Manager (experienced leader)
  2. Senior IR Analyst (technical lead)
  3. IR Analysts (2-3 for coverage)
  4. Forensic Specialist
  5. Threat Intel Analyst

TRAINING REQUIREMENTS:
  Training Type    | Frequency  | Audience
  IR procedures    | Annually   | All IR team
  Forensic tools   | Semi-annual| Forensic analysts
  Tabletop exercise| Quarterly  | Full team
  Live exercise    | Annually   | Core IR team
  Industry training| As needed  | Specialists
  Cert prep        | As needed  | Individual

CERTIFICATIONS:
  → GIAC Certified Incident Handler (GCIH)
  → GIAC Certified Forensic Analyst (GCFA)
  → GIAC Certified Forensic Examiner (GCFE)
  → EC-Council ECIH
  → CompTIA CySA+
  → SANS FOR508 (Advanced IR)
  → SANS FOR500 (Forensic Analysis)

TEAM MATURITY:
  Level 1: Ad-hoc response, no formal team
  Level 2: Defined team, basic procedures
  Level 3: Trained team, playbooks, tools
  Level 4: Tested team, regular exercises
  Level 5: Optimized, continuous improvement

RETAINER AGREEMENTS:
  → External IR firm on retainer
  → Pre-negotiated rates and SLA
  → Activated for major incidents
  → Supplements internal team
  → Often required by cyber insurance
  → Key providers: CrowdStrike, Mandiant, 
    Secureworks, Unit42, Kroll
```

---

## Summary Table

| Role | Responsibility | Activation |
|------|---------------|------------|
| IR Manager | Lead, coordinate, decide | All incidents |
| IR Analyst | Investigate, contain | All incidents |
| Forensics | Evidence, malware analysis | Medium+ |
| Legal | Compliance, liability | Data breaches |
| IT Ops | System changes, recovery | Most incidents |
| Comms | Internal/external messaging | Public incidents |
| Executive | Strategic decisions | Critical incidents |

---

## Revision Questions

1. What are the three IR team organizational models?
2. What are the key responsibilities of an Incident Commander?
3. When should legal counsel be activated during an incident?
4. How should on-call rotations be structured for IR teams?
5. What external resources should be pre-arranged through retainer agreements?

---

*Previous: [01-ir-lifecycle.md](01-ir-lifecycle.md) | Next: [03-ir-policies-and-procedures.md](03-ir-policies-and-procedures.md)*

---

*[Back to README](../README.md)*
