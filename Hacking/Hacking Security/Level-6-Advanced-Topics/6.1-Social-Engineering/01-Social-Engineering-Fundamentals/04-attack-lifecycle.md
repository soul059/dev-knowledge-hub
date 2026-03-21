# Unit 1: Social Engineering Fundamentals — Topic 4: Attack Lifecycle

## Overview

The **social engineering attack lifecycle** follows a structured methodology similar to the penetration testing kill chain. Understanding each phase — from target selection through execution and exit — enables security teams to implement controls at every stage and identify attacks early in the process.

---

## 1. Attack Lifecycle Phases

```
SOCIAL ENGINEERING ATTACK LIFECYCLE:

Phase 1          Phase 2          Phase 3
RECONNAISSANCE   TARGETING        PRETEXTING
  Research       Select           Build
  target org     specific         convincing
  and people     victims          cover story
     │               │               │
     ▼               ▼               ▼
┌─────────┐    ┌─────────┐    ┌─────────┐
│ Gather  │───→│ Profile │───→│ Develop │
│ Intel   │    │ Targets │    │ Pretext │
└─────────┘    └─────────┘    └─────────┘
                                   │
     ┌─────────────────────────────┘
     ▼
Phase 4          Phase 5          Phase 6
ENGAGEMENT       EXPLOITATION     EXIT
  Make           Execute          Cover
  contact        the attack       tracks
     │               │               │
     ▼               ▼               ▼
┌─────────┐    ┌─────────┐    ┌─────────┐
│ Build   │───→│ Extract │───→│ Clean  │
│ Rapport │    │ Info/   │    │ Up     │
│         │    │ Access  │    │        │
└─────────┘    └─────────┘    └─────────┘
```

---

## 2. Phase Details

```
PHASE 1: RECONNAISSANCE (Days to Weeks)

  OBJECTIVES:
  → Identify target organization
  → Understand organizational structure
  → Map key personnel
  → Find communication patterns
  → Discover security controls

  ACTIVITIES:
  → Website analysis (about, team, blog)
  → LinkedIn profiling (employees, roles)
  → Social media monitoring
  → Job posting analysis (technologies used)
  → Press releases and news
  → Public records and filings
  → Google dorking for exposed docs
  → Conference attendee lists

  OUTPUT: Target profile document

---

PHASE 2: TARGETING (Hours to Days)

  OBJECTIVES:
  → Select specific victims
  → Identify most vulnerable individuals
  → Map relationships and authority

  SELECTION CRITERIA:
  → Access to desired information/systems
  → Susceptibility to manipulation
  → Position in organization
  → Communication patterns
  → Technology proficiency (lower = easier)

  HIGH-VALUE TARGETS:
  → Executive assistants (access to executives)
  → Help desk staff (accustomed to helping)
  → New employees (unfamiliar with processes)
  → Finance/accounting (money transfers)
  → IT staff (system access)
  → Reception (physical access)

---

PHASE 3: PRETEXTING (Hours to Days)

  OBJECTIVES:
  → Create believable cover story
  → Prepare supporting materials
  → Set up infrastructure

  PRETEXT ELEMENTS:
  → Who am I? (identity/role)
  → Why am I contacting you? (reason)
  → What do I need? (request)
  → Why is it urgent? (pressure)
  → How can you verify me? (fake validation)

  PREPARATION:
  → Register lookalike domains
  → Create fake badge/ID
  → Set up phishing pages
  → Prepare phone scripts
  → Practice the scenario

---

PHASE 4: ENGAGEMENT (Minutes to Hours)

  OBJECTIVES:
  → Establish initial contact
  → Build trust and rapport
  → Position for exploitation

  TECHNIQUES:
  → Open with context/familiarity
  → Demonstrate knowledge of the org
  → Be helpful, professional, friendly
  → Mirror communication style
  → Build credibility gradually

---

PHASE 5: EXPLOITATION (Seconds to Minutes)

  OBJECTIVES:
  → Execute the actual attack
  → Obtain information or access
  → Achieve the mission goal

  ACTIONS:
  → Credential harvesting (phishing page)
  → Information extraction (questions)
  → Physical access (tailgate/badge)
  → Malware delivery (attachment/USB)
  → Fund transfer (BEC)

---

PHASE 6: EXIT (Minutes)

  OBJECTIVES:
  → End interaction naturally
  → Avoid raising suspicion
  → Clean up evidence
  → Maintain access if needed

  TECHNIQUES:
  → Thank the target
  → Provide reason for ending conversation
  → Leave positive impression (return possible)
  → Remove evidence of contact
  → Establish persistence (backdoor, credentials)
```

---

## 3. Defense at Each Phase

```
DEFENSE MAPPING:

PHASE              DEFENSE CONTROLS

Reconnaissance  →  Limit public information
                   Social media policy
                   Monitor for OSINT exposure

Targeting       →  Awareness training for all roles
                   Identify high-risk positions
                   Extra verification for sensitive roles

Pretexting      →  Domain monitoring (lookalikes)
                   Caller ID verification policies
                   Visitor management systems

Engagement      →  Verify identity through callback
                   Challenge unexpected requests
                   "Trust but verify" culture

Exploitation    →  MFA (prevents credential theft)
                   Email filtering (blocks phishing)
                   Physical access controls
                   Dual authorization for transfers

Exit            →  Logging and monitoring
                   Incident reporting culture
                   Post-incident analysis
```

---

## Summary Table

| Phase | Duration | Attacker Goal | Key Defense |
|-------|----------|--------------|-------------|
| Reconnaissance | Days-Weeks | Gather intelligence | Limit public info |
| Targeting | Hours-Days | Select victims | Role-based awareness |
| Pretexting | Hours-Days | Build cover story | Domain monitoring |
| Engagement | Minutes-Hours | Build trust | Identity verification |
| Exploitation | Seconds-Minutes | Execute attack | MFA, email filtering |
| Exit | Minutes | Cover tracks | Logging, reporting |

---

## Revision Questions

1. What are the six phases of the social engineering attack lifecycle?
2. What activities occur during the reconnaissance phase?
3. Who are the most commonly targeted individuals and why?
4. What elements make up a convincing pretext?
5. How can defenders implement controls at each phase?

---

*Previous: [03-influence-principles-cialdini.md](03-influence-principles-cialdini.md) | Next: [05-why-it-works.md](05-why-it-works.md)*

---

*[Back to README](../README.md)*
