# Unit 5: Physical Social Engineering — Topic 1: Tailgating

## Overview

**Tailgating** (also called **piggybacking**) is a physical social engineering technique where an unauthorized person follows an authorized person through a secured entrance. It exploits social norms of politeness — people naturally hold doors open for others. Tailgating bypasses physical access controls like badge readers, key cards, and PIN pads without requiring any technical hacking.

---

## 1. How Tailgating Works

```
TAILGATING FLOW:

  Authorized Employee              Attacker
       │                              │
       │    Approaches secured door    │
       │    ┌─────────┐               │
       │───▶│  BADGE   │               │
       │    │  READER  │               │
       │    └─────────┘               │
       │    Door opens                 │
       │         │                     │
       │    ┌────▼────┐               │
       │    │  DOOR   │               │
       │    │  OPENS  │◄──── Follows   │
       │    └─────────┘    closely     │
       │                               │
       │    Employee holds door         │
       │    or attacker catches door    │
       │                               │
       ▼    Both now inside secured     ▼
            area

VARIATIONS:

CLASSIC TAILGATING:
  → Walk closely behind authorized person
  → Catch the door before it closes
  → No interaction needed

PIGGYBACKING (Social):
  → Ask the person to hold the door
  → "Excuse me, I forgot my badge upstairs"
  → "My hands are full, could you hold the door?"
  → Social politeness ensures compliance

OPPORTUNISTIC:
  → Wait near secured entrance
  → Watch for groups entering/exiting
  → Join the group naturally
  → Blend in with crowd
```

---

## 2. Tailgating Techniques

```
TECHNIQUE 1: HANDS FULL
  → Carry boxes, coffee cups, laptop bags
  → Look like you're struggling
  → People naturally open doors to help
  → "Thanks! Moving day is always crazy"
  → Almost 100% success rate

TECHNIQUE 2: PHONE CONVERSATION
  → Pretend to be on an important call
  → Walk with purpose toward the door
  → Person holds door — you nod thanks
  → Being "busy" discourages questions
  → Provides cover for not badging in

TECHNIQUE 3: SMOKING AREA
  → Join employees in the smoking area
  → Build rapport with small talk
  → Walk back inside with the group
  → No one questions a "fellow smoker"
  → Often leads to secondary entrances

TECHNIQUE 4: DELIVERY PERSON
  → Carry packages with company logo visible
  → "Delivery for [department name]"
  → Delivery people routinely let in
  → Uniform adds credibility
  → Often not logged or tracked

TECHNIQUE 5: TIMING
  → Rush hour: many people entering/exiting
  → Just after lunch: groups returning
  → After meetings: conference room exodus
  → Shift changes: flow of people
  → Large groups = easy to blend in

TECHNIQUE 6: EMERGENCY PRETEXT
  → "I need to get to the server room — alarm went off"
  → "Plumber — there's a leak on the third floor"
  → Urgency overrides access control procedures
  → People won't delay an "emergency"
```

---

## 3. Why Tailgating Succeeds

```
PSYCHOLOGICAL FACTORS:

SOCIAL POLITENESS:
  → Holding doors is ingrained social behavior
  → Not holding doors feels "rude"
  → People avoid confrontation
  → Challenging someone feels uncomfortable

ASSUMPTIONS:
  → "They must work here"
  → "They probably just forgot their badge"
  → "Security would have stopped them"
  → "It's not my responsibility"

BYSTANDER EFFECT:
  → "Someone else will check them"
  → In groups, responsibility is diffused
  → No one feels individually responsible
  → The more people present, the less likely challenge

AUTHORITY COMPLIANCE:
  → Well-dressed person = probably management
  → Uniform = must be authorized
  → Confident demeanor = belongs here
  → Clipboard/tools = working here
```

---

## 4. Real-World Tailgating Scenarios

```
SCENARIO 1: CORPORATE OFFICE
  → Attacker arrives at 8:30 AM (rush hour)
  → Joins group of employees entering building
  → Walks with purpose, phone to ear
  → Nods to badge reader as if they badged
  → No one notices they didn't actually scan

SCENARIO 2: DATA CENTER
  → Attacker poses as cable installer
  → Carries coil of cable and tool bag
  → Arrives when shift change happens
  → Walks in with outgoing/incoming staff
  → Once inside, has access to racks

SCENARIO 3: MULTI-TENANT BUILDING
  → Shared entrance, different companies per floor
  → Badge required for lobby → elevator
  → Follow any tenant through lobby
  → Take elevator to target floor
  → Floor doors may or may not be secured

SCENARIO 4: SECURE FACILITY
  → Wait at loading dock (less secured)
  → Help unload a delivery truck
  → Walk inside with legitimate delivery staff
  → Loading docks rarely have mantraps
  → Often leads directly to internal areas
```

---

## 5. Defending Against Tailgating

```
TECHNICAL CONTROLS:
  → Mantraps/airlocks (one person at a time)
  → Turnstiles (individual authentication)
  → Tailgate detection sensors
  → Security cameras at entry points
  → Badge-in + badge-out tracking
  → Visitor management systems

PROCEDURAL CONTROLS:
  → No tailgating policy (enforced)
  → Challenge unknown persons
  → All visitors must be escorted
  → Badge must be visible at all times
  → Report tailgating attempts
  → Regular security audits

CULTURAL CONTROLS:
  → Train employees to challenge politely
  → "I'm sorry, you'll need to badge in yourself"
  → Remove stigma from challenging
  → Reward reporting of incidents
  → Security awareness campaigns
  → Lead by example (management challenges too)

PHYSICAL DESIGN:
  → Single-person entry points
  → Clear signage about badge requirements
  → Barriers preventing group entry
  → Reception desk with line of sight to doors
  → Badge readers that require individual scan
```

---

## Summary Table

| Technique | Success Rate | Detection Difficulty | Defense |
|-----------|-------------|---------------------|---------|
| Hands full | Very high | Easy to miss | Mantrap |
| Phone conversation | High | Medium | Turnstile |
| Smoking area | High | Hard | Separate re-entry |
| Delivery person | High | Medium | Visitor management |
| Rush hour blend | Very high | Very hard | Tailgate sensors |

---

## Revision Questions

1. What is the difference between tailgating and piggybacking?
2. Why is the "hands full" technique so effective?
3. What psychological factors make tailgating successful?
4. What physical controls can prevent tailgating?
5. How should employees be trained to handle potential tailgating?

---

*Previous: None (First topic in this unit) | Next: [02-impersonation.md](02-impersonation.md)*

---

*[Back to README](../README.md)*
