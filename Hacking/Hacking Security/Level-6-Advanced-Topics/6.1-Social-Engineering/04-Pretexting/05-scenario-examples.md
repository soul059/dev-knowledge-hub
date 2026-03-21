# Unit 4: Pretexting — Topic 5: Scenario Examples

## Overview

This topic provides comprehensive, real-world pretexting scenarios that demonstrate how social engineers combine identity, narrative, psychological triggers, and technical knowledge to achieve their objectives. Each scenario includes the pretext setup, execution, and the principles being exploited. These examples serve as both learning material for understanding attacks and templates for authorized security assessments.

---

## 1. Scenario: IT Support Password Reset

```
SCENARIO DETAILS:

PRETEXT:
  Identity: IT Help Desk Technician
  Name: "Alex from IT Support"
  Objective: Obtain user credentials
  Channel: Phone (vishing)

PREPARATION:
  → Research: Company IT department structure
  → OSINT: Help desk phone number format
  → Caller ID: Spoofed to internal IT extension
  → Knowledge: Internal ticketing system name

SCRIPT:

  Attacker: "Hi, this is Alex from IT Support. 
  We're migrating to a new email server this 
  weekend, and I'm calling to verify accounts 
  that need to be transferred. Is this [Target Name]?"

  Target: "Yes, that's me."

  Attacker: "Great. I have your account flagged 
  for migration. I just need to verify a few 
  details. Can you confirm your email address?"

  Target: "[provides email]"

  Attacker: "Perfect, that matches. Now, to ensure 
  your account transfers correctly, I need to 
  verify your current password. This is to make 
  sure it meets the new system's requirements."

  Target: "[provides password]"

  Attacker: "Excellent. Your password meets the 
  requirements. The migration will happen Saturday
  night. You may need to log in again Monday 
  morning. If you have any issues, call the help 
  desk. Have a great day!"

PRINCIPLES EXPLOITED:
  → Authority (IT department)
  → Legitimacy (real process — server migration)
  → Helpfulness (assisting with transition)
  → Social proof (implying others have cooperated)

SUCCESS RATE: ~40-60%
```

---

## 2. Scenario: Vendor Maintenance Access

```
SCENARIO DETAILS:

PRETEXT:
  Identity: HVAC Maintenance Technician
  Name: "Mike from ABC Climate Solutions"
  Objective: Physical access to server room
  Channel: In-person

PREPARATION:
  → Identify HVAC vendor (LinkedIn, social media, dumpster diving)
  → Create fake work order with vendor branding
  → Wear appropriate uniform (polo, work boots)
  → Carry clipboard and basic tools
  → Register lookalike phone number for "dispatch"

EXECUTION:

  [Walks up to front desk]

  Attacker: "Morning! I'm Mike from ABC Climate 
  Solutions. We got a call about a temperature 
  alert in your server room. Work order #4521."

  [Shows clipboard with official-looking work order]

  Receptionist: "I didn't get any notice about this."

  Attacker: "Yeah, the alert came in overnight. 
  Your facility manager probably hasn't had a 
  chance to update you. The system flagged high 
  temps that could damage your equipment. We need 
  to check the CRAC unit before it causes problems."

  Receptionist: "Let me call facilities..."

  Attacker: "Sure thing. You can also call our 
  dispatch at [spoofed number] to verify the 
  work order. Time is kind of critical though — 
  if the temps keep climbing, you could lose servers."

  [Receptionist calls spoofed number, gets "dispatch"]
  [Receptionist escorts to server room]

  Attacker: [Takes photos, installs rogue device, 
  notes physical security controls]

PRINCIPLES EXPLOITED:
  → Authority (vendor with work order)
  → Urgency (equipment could be damaged)
  → Fear (server downtime)
  → Verification offered (controlled callback)
  → Industry knowledge (CRAC unit terminology)
```

---

## 3. Scenario: New Employee Social Engineering

```
SCENARIO DETAILS:

PRETEXT:
  Identity: New Employee (first week)
  Name: "Jordan — new in Marketing"
  Objective: Access badges, network access, internal info
  Channel: In-person + email

DAY 1 — BUILDING RAPPORT:

  Attacker: [Approaches employee area, looking lost]
  
  "Hi! I'm Jordan, just started in Marketing 
  this week. I'm completely lost — can't even 
  find the break room. Can you help?"

  [Small talk, builds rapport]

  "This is such a nice office! My last place was 
  terrible. Everyone here seems so helpful."

  Target: "Welcome! Let me show you around."

  Attacker: "Thanks so much! By the way, how do 
  I get on the WiFi? IT hasn't set me up yet."

  Target: [Provides WiFi password]

DAY 2 — EXPANDING ACCESS:

  Attacker: "Hey [Name]! Thanks again for yesterday. 
  Quick question — I left my badge at home and 
  need to get to the third floor for a meeting. 
  Can you swipe me in?"

  Target: [Badges attacker through door]

  Attacker: [Takes note of badge type, access points]

DAY 3 — INFORMATION GATHERING:

  Attacker: "What ticketing system do you guys use? 
  At my last company we used Jira."

  Target: "We use ServiceNow for IT and Asana for 
  project management."

  Attacker: "Oh nice. And for VPN — do I just need 
  to install the client, or...?"

  Target: "Yeah, it's Cisco AnyConnect. IT will set 
  you up with a profile."

PRINCIPLES EXPLOITED:
  → Liking (friendly, relatable new person)
  → Reciprocity (showed vulnerability, gets help)
  → Helpfulness (people naturally help new employees)
  → Social proof (everyone helps new people)
  → Gradual escalation (WiFi → badge → systems info)
```

---

## 4. Scenario: Executive Urgency

```
SCENARIO DETAILS:

PRETEXT:
  Identity: CEO's Executive Assistant
  Name: "Lisa, from [CEO's] office"
  Objective: Wire transfer authorization
  Channel: Phone + email

EXECUTION:

  [Phone call to Accounts Payable]

  Attacker: "Hi, this is Lisa from [CEO Name]'s 
  office. [CEO] is traveling in Europe and needs 
  an urgent wire transfer processed. He asked me 
  to call you directly because he can't access 
  the system from overseas."

  AP Staff: "I usually need email authorization..."

  Attacker: "I understand. He sent an email about 
  30 minutes ago — check your inbox. The transfer 
  is for a time-sensitive acquisition that's under 
  NDA, so please keep this confidential. He'll be 
  in meetings for the next 6 hours, so he won't 
  be reachable."

  [Spoofed email from "CEO" is already in inbox]

  AP Staff: "I see the email. What are the details?"

  Attacker: "The amount is $87,500 to [bank account].
  It needs to go out today to secure the deal. 
  [CEO] said he'll approve it formally when he's 
  back, but the window closes at 5 PM."

  AP Staff: [Processes the transfer]

PRINCIPLES EXPLOITED:
  → Authority (CEO's office)
  → Urgency (time-sensitive, deal closing)
  → Confidentiality (NDA prevents verification)
  → Unavailability (CEO in meetings, can't verify)
  → Multi-channel (phone + email confirmation)
```

---

## 5. Scenario: Security Audit Pretext

```
SCENARIO DETAILS:

PRETEXT:
  Identity: Third-party Security Auditor
  Name: "David from [Audit Firm]"
  Objective: Access to systems, policies, configurations
  Channel: Email → Phone → In-person

PHASE 1 — EMAIL:
  Subject: Upcoming Security Compliance Audit

  "Dear [Target],

  As part of [Company]'s annual security compliance
  review, [Audit Firm] has been engaged to conduct
  an assessment of your IT infrastructure. 

  I will be on-site next Tuesday to begin the 
  review. Please have the following prepared:
  - Network diagrams
  - System access for audit account
  - Firewall rule sets
  - Recent vulnerability scan reports

  Please confirm receipt and availability.

  David Williams, CISA, CISSP
  Senior Auditor | [Audit Firm]"

PHASE 2 — PHONE FOLLOW-UP:
  "Hi, this is David from [Audit Firm]. I sent an
  email last week about the security audit. I wanted
  to confirm the logistics for Tuesday."

PHASE 3 — IN-PERSON:
  → Arrives in suit with audit firm branding
  → Professional demeanor, clipboard
  → Requests access to systems for "testing"
  → Installs "audit tools" (actually persistence)
  → Reviews configurations (exports data)

PRINCIPLES EXPLOITED:
  → Authority (auditor role + certifications)
  → Institutional (compliance requirement)
  → Multi-phase (builds credibility over time)
  → Fear (non-compliance consequences)
  → Preparation (email establishes legitimacy)
```

---

## Summary Table

| Scenario | Target | Objective | Key Principle |
|----------|--------|-----------|--------------|
| IT Support | Any employee | Credentials | Authority + legitimacy |
| Vendor | Reception/facilities | Physical access | Urgency + verification |
| New Employee | Helpful colleagues | Network/system info | Liking + helpfulness |
| Executive Urgency | Finance/AP | Wire transfer | Authority + urgency |
| Security Audit | IT staff | System access | Authority + institutional |

---

## Revision Questions

1. What makes the IT support pretext so commonly successful?
2. How does the vendor maintenance scenario use controlled verification?
3. Why is the "new employee" approach effective for gradual information gathering?
4. What multi-channel techniques make the executive urgency scenario convincing?
5. How does the security audit pretext build credibility over multiple phases?

---

*Previous: [04-building-rapport.md](04-building-rapport.md) | Next: [06-voice-manipulation.md](06-voice-manipulation.md)*

---

*[Back to README](../README.md)*
