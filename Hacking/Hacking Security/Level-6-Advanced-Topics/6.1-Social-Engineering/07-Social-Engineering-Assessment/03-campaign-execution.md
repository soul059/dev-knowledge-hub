# Unit 7: Social Engineering Assessment — Topic 3: Campaign Execution

## Overview

**Campaign execution** is the active phase of a social engineering assessment where planned attacks are carried out against the target organization. This topic covers the operational aspects of executing phishing, vishing, physical, and combined campaigns — from infrastructure setup through attack delivery to evidence collection.

---

## 1. Phishing Campaign Execution

```
PHISHING CAMPAIGN WORKFLOW:

STEP 1: INFRASTRUCTURE SETUP
  → Register lookalike domain (2+ weeks before)
  → Configure email server (SPF, DKIM, DMARC)
  → Build/clone landing pages
  → Set up GoPhish or similar framework
  → Configure SSL certificates
  → Test email deliverability
  → Warm up sending IP

STEP 2: TARGET LIST PREPARATION
  → Import employee email list from client
  → Or gather via OSINT (if scope allows)
  → Create groups (by department, location)
  → Remove excluded individuals
  → Verify email addresses

STEP 3: TEMPLATE CREATION
  → Design email template(s)
  → Match organization's email style
  → Include tracking elements
  → Create credential harvesting page
  → Test rendering in multiple clients
  → Prepare multiple scenarios

STEP 4: LAUNCH
  → Send in waves (not all at once)
  → Stagger timing for realism
  → Monitor delivery and bounce rates
  → Track opens, clicks, submissions
  → Be prepared for target responses
  → Watch for security team detection

STEP 5: MONITORING
  → Real-time dashboard monitoring
  → Track campaign metrics
  → Note when security team responds
  → Record timeline of events
  → Identify outlier behaviors

STEP 6: DATA COLLECTION
  → Export all campaign data
  → Credential submissions (hash/redact)
  → Timeline of events
  → Click-through analysis
  → Report generation
```

---

## 2. Vishing Campaign Execution

```
VISHING WORKFLOW:

PREPARATION:
  → Develop call scripts
  → Set up caller ID spoofing
  → Prepare recording setup (if authorized)
  → Create target call list
  → Practice scripts with team

CALL STRUCTURE:
  1. Opening: Identify yourself (pretext)
  2. Rapport: Build connection
  3. Pretext: Explain reason for call
  4. Ask: Make specific request
  5. Handle objections: Respond to pushback
  6. Close: Thank and disconnect

EXECUTION BEST PRACTICES:
  → Call from quiet environment
  → Have script visible but don't read verbatim
  → Maintain consistent character
  → Record outcomes immediately after each call
  → Take breaks between calls (fatigue affects quality)
  → Stop if a call goes poorly — regroup

DOCUMENTATION:
  → Target name and number
  → Date, time, duration
  → Pretext used
  → Information obtained
  → Whether target challenged
  → Whether target reported call
  → Notes on target's demeanor
```

---

## 3. Physical Assessment Execution

```
PHYSICAL CAMPAIGN WORKFLOW:

PRE-EXECUTION:
  → Reconnaissance visit (observe security)
  → Map entry points and camera locations
  → Identify shift changes and schedules
  → Prepare props, clothing, documentation
  → Carry authorization letter at all times
  → Brief team on roles and abort procedures
  → Confirm emergency contacts are available

EXECUTION:
  → Arrive at planned time
  → Execute primary entry method
  → Document everything (discreetly)
  → Photograph accessed areas (authorized only)
  → Plant proof-of-access items (business cards, etc.)
  → Attempt to reach target areas
  → Time all activities
  → Monitor for security response

EVIDENCE COLLECTION:
  → Timestamped photos (show date/time)
  → Video documentation (if authorized)
  → Notes on security controls encountered
  → Notes on employee interactions
  → Access cards/badges collected (for evidence)
  → Documents found (sensitive information)
  → Network access achieved (if in scope)

IF CONFRONTED:
  → Remain calm and professional
  → Produce authorization letter immediately
  → Contact emergency POC
  → Do NOT resist security or law enforcement
  → Do NOT break cover unnecessarily
  → Cooperate fully
  → Document the encounter

IF LAW ENFORCEMENT ARRIVES:
  → Comply immediately with all instructions
  → State: "I'm conducting an authorized 
    security assessment"
  → Present authorization letter
  → Request they contact the listed POC
  → Do NOT argue or debate
  → Accept the situation calmly
  → Document the encounter in detail
```

---

## 4. Combined Campaign Strategies

```
MULTI-VECTOR APPROACH:

SCENARIO: PHISH → VISH → PHYSICAL

WEEK 1 — PHISHING:
  → Send targeted phishing emails
  → Harvest credentials or establish pretext
  → Identify responsive targets
  → Build knowledge of organization

WEEK 2 — VISHING:
  → Call targets who didn't report phishing
  → Reference "the email we sent"
  → Use info gathered to build credibility
  → Attempt credential/info extraction
  → Identify physical security details

WEEK 3 — PHYSICAL:
  → Use gathered intelligence for pretexting
  → Attempt building access
  → Reference real people and projects
  → Use credentials obtained earlier
  → Plant devices or access systems

VALUE OF COMBINED:
  → Most realistic assessment
  → Tests defense across all channels
  → Reveals how attacks chain together
  → Demonstrates real-world attack scenarios
  → Highest quality findings
```

---

## 5. Operational Security

```
OPSEC FOR TESTERS:

COMMUNICATION:
  → Use encrypted channels (Signal, encrypted email)
  → Don't discuss operations on unsecured channels
  → Separate testing infrastructure from personal
  → VPN for all testing activities
  → Dedicated testing devices

IDENTITY PROTECTION:
  → Use assessment-specific identities
  → Separate testing phone numbers
  → Dedicated email accounts
  → Don't use personal social media
  → Cover stories for team members

DATA HANDLING:
  → Encrypt all collected data
  → Secure storage for evidence
  → Don't store credentials in plaintext
  → Delete test data per agreement
  → Secure evidence transfer to client
  → Chain of custody documentation

INFRASTRUCTURE:
  → Dedicated testing infrastructure
  → Separate from other client work
  → Teardown after engagement
  → Ensure no residual access
  → Domain deregistration or transfer
```

---

## Summary Table

| Campaign Type | Setup Time | Execution Time | Evidence Type |
|--------------|-----------|----------------|--------------|
| Phishing | 1-2 weeks | 1-2 weeks | Digital metrics |
| Vishing | 2-3 days | 1-5 days | Call recordings/notes |
| Physical | 1 week | 1-3 days | Photos, timestamps |
| USB drop | 1-2 days | 1-2 weeks monitoring | Callback data |
| Combined | 2-3 weeks | 3-5 weeks | All types |

---

## Revision Questions

1. What are the steps in executing a phishing campaign?
2. How should vishing calls be structured and documented?
3. What procedures should be followed if confronted during physical testing?
4. Why are combined/multi-vector campaigns more valuable?
5. What operational security measures should testers maintain?

---

*Previous: [02-scope-and-authorization.md](02-scope-and-authorization.md) | Next: [04-metrics-and-reporting.md](04-metrics-and-reporting.md)*

---

*[Back to README](../README.md)*
