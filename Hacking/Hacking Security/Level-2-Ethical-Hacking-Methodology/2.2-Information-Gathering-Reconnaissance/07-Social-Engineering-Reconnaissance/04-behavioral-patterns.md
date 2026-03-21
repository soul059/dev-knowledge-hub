# Behavioral Patterns

## Unit 7 - Topic 4 | Social Engineering Reconnaissance

---

## Overview

**Behavioral pattern analysis** studies how target employees act, communicate, and respond to different situations. Understanding behavioral patterns enables attackers to time attacks optimally, craft messages that match the target's communication style, and exploit psychological triggers.

---

## 1. Types of Behavioral Patterns

```
BEHAVIORAL INTELLIGENCE CATEGORIES:

TEMPORAL PATTERNS:
├── Work hours (when they're online)
├── Lunch break timing
├── Meeting schedules (busy/free times)
├── Travel patterns (business trips)
├── Vacation schedules
└── Best attack window = high-stress busy times

COMMUNICATION PATTERNS:
├── Email response time (quick? slow?)
├── Communication style (formal vs casual)
├── Preferred channels (email, Slack, phone)
├── Language patterns (jargon, abbreviations)
├── Signature format
└── Use to craft matching messages

DECISION PATTERNS:
├── How they handle urgent requests
├── Authority compliance tendency
├── Process adherence vs shortcuts
├── Verification habits (do they check?)
├── Escalation behavior
└── Exploit: urgency + authority = action
```

---

## 2. Psychological Triggers

| Trigger | Description | Attack Application |
|---------|-------------|-------------------|
| **Authority** | Obey perceived authority figures | Impersonate CEO/manager |
| **Urgency** | Act quickly without thinking | "Immediate action required!" |
| **Scarcity** | Fear of missing out | "Limited time offer" |
| **Social proof** | Follow what others do | "Your colleagues already updated" |
| **Reciprocity** | Feel obligated to return favors | Give something first, then ask |
| **Liking** | Help people we like | Build rapport before attack |
| **Fear** | Avoid negative consequences | "Your account will be locked" |
| **Curiosity** | Need to know | "Someone shared a photo of you" |

---

## 3. Communication Style Analysis

```
ANALYZING TARGET'S COMMUNICATION STYLE:

FORMAL COMMUNICATOR:
├── Uses: "Dear Mr./Ms.", "Best regards"
├── Full sentences, proper grammar
├── Avoids abbreviations and emojis
├── Attack: Match formal tone exactly
└── "Dear Jane, Please review the attached..."

CASUAL COMMUNICATOR:
├── Uses: "Hey", "Cheers", "Thanks!"
├── Short messages, abbreviations
├── Uses emojis and GIFs
├── Attack: Match casual tone
└── "Hey Jane! Can you check this out? 👍"

TECHNICAL COMMUNICATOR:
├── Uses jargon freely
├── References specific tools/systems
├── Expects technical competence
├── Attack: Use correct technical terms
└── "The SIEM flagged an anomaly in the
     Azure AD tenant. Can you pull the
     audit logs from Sentinel?"

HOW TO DETERMINE STYLE:
├── Read their social media posts
├── Check professional blog posts
├── Review public email exchanges
├── Observe LinkedIn activity
└── Check conference presentations
```

---

## 4. Timing Attacks

```
OPTIMAL ATTACK TIMING:

MONDAY MORNING:
├── Inbox overload
├── Rushing through emails
├── Less scrutiny per message
└── ★★★★ Good for phishing

FRIDAY AFTERNOON:
├── Mental fatigue
├── Ready to leave for weekend
├── Quicker to click and move on
└── ★★★★ Good for phishing

LUNCH TIME:
├── Mobile device usage
├── Smaller screen = less context
├── Distracted while eating
└── ★★★ Moderate for phishing

AFTER VACATION:
├── Hundreds of unread emails
├── Catching up quickly
├── Less careful review
└── ★★★★★ Best for phishing

DURING CRISIS:
├── Stress reduces critical thinking
├── Urgent requests seem normal
├── People bypass procedures
└── ★★★★★ Best for all social engineering

WORST TIMING:
├── Right after security training
├── Mid-morning (fresh and alert)
├── During dedicated focus time
└── ★ Low success rate
```

---

## 5. Pretext Optimization

```
BEHAVIORAL PATTERN → ATTACK OPTIMIZATION:

FINDING: Target responds quickly to authority
OPTIMIZE: Impersonate C-level executive
EMAIL: "From: CEO <ceo@target.com> — Urgent request"

FINDING: Target is helpful to new employees
OPTIMIZE: Pretend to be a new hire
CALL: "Hi, I just started and I'm locked out..."

FINDING: Target frequently shares on social media
OPTIMIZE: Use personal details in lure
PHISH: "Your Strava account needs verification"

FINDING: Target works late and responds to after-hours email
OPTIMIZE: Send phishing email at 8 PM
REASON: Tired, fewer colleagues to verify with

FINDING: Target uses informal language with team
OPTIMIZE: Match casual tone in emails
PHISH: "Hey! Check this doc when you get a sec 😊"
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Temporal patterns** | Time attacks for max success |
| **Psychological triggers** | Authority + urgency = action |
| **Communication style** | Match the target's tone exactly |
| **Best timing** | Friday PM, Monday AM, post-vacation |
| **Worst timing** | After security training |
| **Personalization** | Use behavioral intel to customize attacks |

---

## Quick Revision Questions

1. **What are the 6 psychological triggers used in social engineering?**
2. **When is the best time to send a phishing email?**
3. **Why is matching communication style important?**
4. **How does urgency override critical thinking?**
5. **How do you analyze a target's behavioral patterns?**

---

[← Previous: Social Media Profiling](03-social-media-profiling.md) | [Next: Physical Security Reconnaissance →](05-physical-security-reconnaissance.md)

---

*Unit 7 - Topic 4 of 5 | [Back to README](../README.md)*
