# Unit 3: Phishing — Topic 5: Vishing (Voice Phishing)

## Overview

**Vishing** (Voice Phishing) uses phone calls to socially engineer victims into revealing sensitive information, authorizing transactions, or performing actions that compromise security. Vishing is particularly effective because it allows real-time interaction, emotional manipulation, and exploitation of the human tendency to trust voice communication.

---

## 1. How Vishing Works

```
VISHING ATTACK FLOW:

PREPARATION:
  ┌─────────────────────────────────────┐
  │ 1. Research target (OSINT)          │
  │ 2. Develop pretext/scenario         │
  │ 3. Set up spoofed caller ID         │
  │ 4. Prepare script and objections    │
  │ 5. Set up recording (if authorized) │
  └─────────────────────────────────────┘
              │
              ▼
EXECUTION:
  ┌─────────────────────────────────────┐
  │ 1. Call target                      │
  │ 2. Establish identity/authority     │
  │ 3. Build rapport/urgency            │
  │ 4. Make the request                 │
  │ 5. Handle objections                │
  │ 6. Extract information/action       │
  │ 7. Close call naturally             │
  └─────────────────────────────────────┘
              │
              ▼
EXPLOITATION:
  ┌─────────────────────────────────────┐
  │ 1. Use gathered credentials         │
  │ 2. Access compromised accounts      │
  │ 3. Leverage for further attacks     │
  │ 4. Document findings (if pentest)   │
  └─────────────────────────────────────┘
```

---

## 2. Common Vishing Scenarios

```
VISHING PRETEXTS:

IT SUPPORT / HELPDESK:
  "Hi, this is Dave from IT. We're seeing some 
   unusual activity on your account. I need to 
   verify your identity to check if your account 
   has been compromised. Can you confirm your 
   username and the last password you used?"

BANK / FINANCIAL:
  "This is the fraud department at [Bank]. We've 
   detected a suspicious transaction of $2,500 on
   your account. To verify this wasn't you, I need
   your account number and the security code on 
   the back of your card."

GOVERNMENT / IRS:
  "This is Agent Williams from the IRS. There is 
   an enforcement action pending against your 
   Social Security number. To resolve this, you 
   need to make an immediate payment or face 
   arrest. Press 1 to speak with a case officer."

VENDOR / SUPPLIER:
  "Hi, this is accounting at [Supplier]. Our 
   banking details have changed. I'm sending over
   the updated wire transfer information. Can you
   confirm you'll update your records?"

CEO / EXECUTIVE:
  "This is [CEO Name]. I'm traveling and need 
   you to process an urgent payment. I can't do
   it from here. Can you handle this for me? 
   I'll explain when I'm back."

TECH SUPPORT SCAM:
  "This is Microsoft Support. We've detected 
   malware on your computer sending data to 
   hackers. I need you to open Remote Desktop
   so I can remove it for you."
```

---

## 3. Caller ID Spoofing

```
SPOOFING TECHNOLOGY:

HOW IT WORKS:
  → VoIP services allow setting any caller ID
  → SIP headers contain caller ID information
  → Attackers set header to any number
  → Victim's phone displays spoofed number

TOOLS AND SERVICES:
  → VoIP providers (some allow custom caller ID)
  → SIP trunking with configurable headers
  → Spoofing apps and services
  → PBX systems with outbound caller ID settings

SPOOFING STRATEGIES:
  → Spoof target's own bank number
  → Spoof internal company numbers
  → Spoof government agency numbers
  → Spoof local area codes (neighbor spoofing)
  → Spoof known contact numbers

LIMITATIONS:
  → STIR/SHAKEN protocol reducing effectiveness
  → Some carriers flag unverified calls
  → Callback reveals actual number
  → Doesn't work on all carriers
```

---

## 4. Vishing Techniques

```
PSYCHOLOGICAL TECHNIQUES:

AUTHORITY:
  → "This is the fraud department..."
  → "I'm calling from the CEO's office..."
  → "This is Agent [Name] from the FBI..."
  → Use title, badge number, case number

URGENCY:
  → "Your account will be frozen in 30 minutes"
  → "We need to act now before the hacker..."
  → "The payment must go out before 5 PM"
  → Time pressure prevents careful thinking

FEAR:
  → "You will face legal consequences"
  → "Your identity has been stolen"
  → "Malware is actively stealing your data"
  → Fear overrides rational thinking

HELPFULNESS:
  → "I'm here to help resolve this"
  → "Let me walk you through fixing this"
  → "I want to make sure your account is safe"
  → Victim feels grateful, not suspicious

RECIPROCITY:
  → Provide something first (fake alert)
  → "I noticed this issue and wanted to help"
  → Creates obligation to cooperate
  → "I went out of my way to call you"

VOICE TECHNIQUES:
  → Professional, calm tone builds trust
  → Match speaking pace of target
  → Use industry-specific jargon
  → Background noise (call center ambiance)
  → Hold music for "verification steps"
```

---

## 5. Defending Against Vishing

```
DEFENSE STRATEGIES:

INDIVIDUAL:
  → Never give credentials over phone
  → Hang up and call back on known number
  → Verify caller identity independently
  → Be suspicious of urgent requests
  → Don't trust caller ID

ORGANIZATIONAL:
  → Establish verification procedures
  → Train employees on vishing tactics
  → Implement callback verification
  → Create "safe words" for internal calls
  → Document and report all suspicious calls
  → Conduct vishing simulations

TECHNICAL:
  → Implement STIR/SHAKEN
  → Call recording for sensitive departments
  → VoIP security hardening
  → Robocall blocking services
  → Number verification services
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Definition | Phone-based social engineering |
| Success rate | 30-40% |
| Key advantage | Real-time interaction and emotional manipulation |
| Common pretexts | IT support, bank, IRS, vendor, executive |
| Technology used | Caller ID spoofing via VoIP |
| Primary defense | Hang up and call back on known number |

---

## Revision Questions

1. What makes vishing particularly effective compared to email phishing?
2. What are the most common vishing pretexts?
3. How does caller ID spoofing work and what are its limitations?
4. What psychological techniques are used during vishing calls?
5. What is the single most effective defense against vishing?

---

*Previous: [04-whaling.md](04-whaling.md) | Next: [06-smishing.md](06-smishing.md)*

---

*[Back to README](../README.md)*
