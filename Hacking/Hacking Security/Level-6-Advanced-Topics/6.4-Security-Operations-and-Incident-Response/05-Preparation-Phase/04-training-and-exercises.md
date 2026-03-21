# Unit 5: Preparation Phase — Topic 4: Training and Exercises

## Overview

**Training and exercises** build the skills, confidence, and coordination needed for effective incident response. Regular training ensures team members can execute playbooks under pressure, while exercises test the organization's ability to detect, respond to, and recover from realistic scenarios.

---

## 1. Training Program

```
IR TRAINING FRAMEWORK:

  ┌──────────────────────────────────────┐
  │        TRAINING PYRAMID              │
  │                                      │
  │           /\  Advanced               │
  │          /  \ (Specialized)          │
  │         /    \                       │
  │        /──────\ Intermediate         │
  │       / Incident\ (Role-specific)    │
  │      /  Response  \                  │
  │     /──────────────\ Foundation      │
  │    / Security        \ (Everyone)    │
  │   /  Awareness         \             │
  │  /──────────────────────\            │
  └──────────────────────────────────────┘

TRAINING LEVELS:

  FOUNDATION (All Employees):
  → Security awareness
  → Phishing recognition
  → Incident reporting
  → Password hygiene
  → Data handling
  Frequency: Annual + monthly micro-training

  INTERMEDIATE (IT/Security Staff):
  → IR process and procedures
  → Tool usage and proficiency
  → Log analysis fundamentals
  → Evidence handling basics
  → Communication during incidents
  Frequency: Semi-annual

  ADVANCED (IR Team):
  → Advanced forensics
  → Malware analysis
  → Threat intelligence
  → Advanced SIEM queries
  → Specialized certifications
  Frequency: Ongoing + annual courses

TRAINING RESOURCES:
  Platform        | Focus           | Cost
  SANS courses    | IR, Forensics   | $$$$
  SANS Cyber Ranges| Hands-on labs  | $$$
  CyberDefenders | Blue team CTFs  | Free/Paid
  LetsDefend     | SOC analyst sim | Free/Paid
  TryHackMe      | Various paths   | Free/Paid
  Blue Team Labs | Investigations  | Free/Paid
  RangeForce     | SOC training    | $$$
```

---

## 2. Exercise Types

```
EXERCISE TYPES:

TABLETOP EXERCISE (TTX):
  → Discussion-based scenario
  → No technical execution
  → Tests processes and decision-making
  → Involves leadership and IR team
  → 2-4 hours duration
  → Low cost, high value
  → Frequency: Quarterly

WALKTHROUGH:
  → Step-by-step procedure review
  → Verify playbook accuracy
  → Identify gaps in documentation
  → 1-2 hours duration
  → Frequency: Monthly

FUNCTIONAL EXERCISE:
  → Simulated technical response
  → Uses actual tools and systems
  → Tests specific capabilities
  → 4-8 hours duration
  → Moderate cost
  → Frequency: Semi-annually

FULL-SCALE EXERCISE:
  → Realistic attack simulation
  → Red team vs Blue team
  → Tests entire response chain
  → Multiple days duration
  → High cost, highest value
  → Frequency: Annually

PURPLE TEAM EXERCISE:
  → Collaborative red/blue team
  → Red executes, blue detects
  → Real-time feedback
  → Detection improvement focus
  → 2-5 days
  → Frequency: Quarterly

COMPARISON:
  Type       | Realism | Cost | Duration | Participants
  Tabletop   | Low     | $    | 2-4 hrs  | 10-20
  Walkthrough| Low     | $    | 1-2 hrs  | 5-10
  Functional | Medium  | $$   | 4-8 hrs  | 5-15
  Full-scale | High    | $$$$ | 1-3 days | 20-50+
  Purple team| High    | $$$  | 2-5 days | 5-15
```

---

## 3. Tabletop Exercise Design

```
TABLETOP EXERCISE TEMPLATE:

PRE-EXERCISE:
  → Define objectives
  → Select scenario
  → Identify participants
  → Prepare injects (scenario events)
  → Brief facilitator
  → Send calendar invites
  → Prepare observation forms

SCENARIO EXAMPLE: RANSOMWARE

  Inject 1 (10:00):
  "SOC receives an alert from CrowdStrike 
  indicating suspicious file encryption on
  FINSVR01 in the finance department."
  Questions:
  → What is your first action?
  → Who do you notify?
  → What information do you need?

  Inject 2 (10:30):
  "Three more servers show encryption activity.
  Users report they cannot access shared drives.
  A ransom note demands 50 BTC."
  Questions:
  → How do you contain the spread?
  → Who needs to be notified now?
  → What is the business impact?

  Inject 3 (11:00):
  "Investigation reveals the attacker has been
  in the network for 2 weeks. They exfiltrated
  customer PII before encrypting."
  Questions:
  → How does this change your response?
  → What are the notification requirements?
  → Do you engage law enforcement?

  Inject 4 (11:30):
  "Media contacts your PR team asking about
  the incident. A customer posts on Twitter
  about receiving a breach notification."
  Questions:
  → What is your media response?
  → How do you manage public communications?

FACILITATION TIPS:
  → Stay on schedule
  → Encourage participation from everyone
  → No blame or finger-pointing
  → Focus on process, not technical details
  → Document all observations
  → Note gaps and improvement areas
```

---

## 4. Measuring Exercise Effectiveness

```
EXERCISE METRICS:

OBSERVATION AREAS:
  Category        | Questions
  Detection       | Was the incident detected? How quickly?
  Communication   | Were the right people notified?
  Decision-making | Were decisions timely and appropriate?
  Coordination    | Did teams work together effectively?
  Containment     | Were containment actions effective?
  Documentation   | Was the incident properly documented?
  Recovery        | Was recovery plan adequate?
  Legal/Compliance| Were notification requirements met?

SCORING RUBRIC:
  Score | Description
  1     | Not demonstrated / significant gap
  2     | Partially demonstrated / improvement needed
  3     | Demonstrated with minor issues
  4     | Fully demonstrated / effective
  5     | Exceptional / exceeded expectations

AFTER-ACTION REPORT:
  1. Exercise Overview
     → Scenario, date, participants
  2. Objectives and Evaluation
     → Met/not met for each objective
  3. Key Observations
     → What went well
     → What needs improvement
  4. Findings by Category
     → Detection, communication, etc.
  5. Recommendations
     → Specific, actionable improvements
  6. Improvement Plan
     → Action items with owners and dates

COMMON FINDINGS:
  → Contact lists out of date
  → Unclear escalation authority
  → Communication gaps between teams
  → Tools not accessible or unfamiliar
  → Legal team not engaged early enough
  → External communication plan missing
  → Evidence preservation not followed
  → Business continuity not integrated
```

---

## 5. Continuous Improvement

```
TRAINING AND EXERCISE CALENDAR:

  Month    | Activity
  January  | Tabletop: Ransomware
  February | Tool training: Forensics
  March    | Walkthrough: Phishing playbook
  April    | Tabletop: Data breach
  May      | Purple team exercise
  June     | Tool training: SIEM advanced
  July     | Tabletop: Insider threat
  August   | Functional exercise
  September| Walkthrough: Account compromise
  October  | Tabletop: Supply chain
  November | Full-scale exercise
  December | Annual review and planning

SKILL DEVELOPMENT TRACKING:
  Skill             | Analyst A | Analyst B | Analyst C
  SIEM queries      | ████░     | ███░░     | █████
  Forensic imaging  | ██░░░     | ████░     | ███░░
  Memory analysis   | █░░░░     | ███░░     | ████░
  Malware triage    | ███░░     | ██░░░     | ████░
  IR leadership     | ████░     | ██░░░     | ███░░
  Report writing    | █████     | ███░░     | ████░

IMPROVEMENT TRACKING:
  Finding ID | Description            | Owner | Status
  TTX-001    | Update contact list    | Admin | Complete
  TTX-002    | Create media playbook  | Comms | In progress
  TTX-003    | Test backup recovery   | IT    | Scheduled
  FUNC-001   | Fix EDR isolation bug  | Eng   | Complete
  FUNC-002   | Train on Volatility    | Train | Scheduled
```

---

## Summary Table

| Exercise Type | Frequency | Duration | Key Benefit |
|--------------|-----------|----------|-------------|
| Tabletop | Quarterly | 2-4 hours | Process validation |
| Walkthrough | Monthly | 1-2 hours | Playbook accuracy |
| Functional | Semi-annual | 4-8 hours | Technical readiness |
| Full-scale | Annual | 1-3 days | End-to-end testing |
| Purple team | Quarterly | 2-5 days | Detection improvement |

---

## Revision Questions

1. What are the different levels of IR training?
2. How do tabletop exercises differ from full-scale exercises?
3. What should a tabletop exercise scenario include?
4. How is exercise effectiveness measured?
5. What does a year-long training and exercise calendar look like?

---

*Previous: [03-tool-preparation.md](03-tool-preparation.md) | Next: [05-threat-intelligence-integration.md](05-threat-intelligence-integration.md)*

---

*[Back to README](../README.md)*
