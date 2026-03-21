# Social Media Profiling

## Unit 7 - Topic 3 | Social Engineering Reconnaissance

---

## Overview

**Social media profiling** gathers personal and professional information about target employees from social media platforms. This intelligence enables highly personalized social engineering attacks that exploit interests, relationships, and behavioral patterns.

---

## 1. Platform-Specific Intelligence

| Platform | Intelligence Available |
|----------|----------------------|
| **LinkedIn** | Job history, skills, certifications, connections, endorsements |
| **Twitter/X** | Opinions, interests, work complaints, tech preferences |
| **Facebook** | Family, friends, hobbies, location check-ins, events |
| **Instagram** | Photos (office, badge, screens), travel, lifestyle |
| **GitHub** | Code, repos, email, technical ability, projects |
| **Reddit** | Anonymous posts about employer, technical problems |
| **Stack Overflow** | Technical questions reveal internal stack |
| **Strava/Fitbit** | Running routes, gym locations, schedules |

---

## 2. Social Media OSINT Techniques

```bash
# === AUTOMATED SOCIAL MEDIA TOOLS ===

# Sherlock — find usernames across 400+ platforms
sherlock john_smith
# Found: twitter.com/john_smith, github.com/john_smith, reddit.com/u/john_smith

# Social-Analyzer — profile search across platforms
social-analyzer --username "john_smith" --metadata

# Maltego — visual relationship mapping
# Transform: Person → Social Media Profiles → Posts → Connections

# === GOOGLE DORKS FOR SOCIAL MEDIA ===
# Find target's posts across platforms:
"John Smith" site:twitter.com "Target Corporation"
"John Smith" site:reddit.com "target" OR "work"
"john.smith" site:github.com

# === PROFILE ANALYSIS ===
# For each found profile, collect:
# ├── Username (same across platforms?)
# ├── Bio/description
# ├── Profile photo (reverse image search)
# ├── Posts/tweets mentioning work
# ├── Connections/friends (co-workers?)
# ├── Groups/communities joined
# ├── Posted photos (EXIF data?)
# └── Check-ins/locations
```

---

## 3. Intelligence Extraction

```
SOCIAL MEDIA INTELLIGENCE CATEGORIES:

PERSONAL INTERESTS:
├── Hobbies: cycling, gaming, photography
├── Sports teams: favorite team
├── Music/entertainment preferences
├── Food/restaurant preferences
├── Travel destinations visited
└── USE: Personalized phishing lures

PROFESSIONAL DETAILS:
├── Current projects discussed
├── Technologies mentioned
├── Conferences attended
├── Certifications earned
├── Work frustrations shared
└── USE: Technical pretexting

RELATIONSHIP MAP:
├── Close colleagues (tagged in posts)
├── Manager/reports (LinkedIn connections)
├── Family members (Facebook)
├── Significant other
├── Friend groups
└── USE: Impersonation, trust exploitation

BEHAVIORAL PATTERNS:
├── Posting schedule (when they're online)
├── Communication style (formal vs casual)
├── Emoji usage, language patterns
├── Opinions and biases
└── USE: Craft matching communication style
```

---

## 4. Photo Intelligence (IMINT)

```
WHAT PHOTOS REVEAL:

OFFICE PHOTOS:
├── Employee badges (name, photo, access level)
├── Computer screens (applications, data)
├── Whiteboards (project plans, credentials!)
├── Office layout (physical security intel)
├── Door codes / keypad numbers
└── Sticky notes on monitors (passwords!)

PERSONAL PHOTOS:
├── EXIF metadata (GPS coordinates, device info)
├── Home location (security system visible?)
├── Vehicle (license plate for pretext)
├── Vacation patterns (empty house/office)
└── Social circle (who to impersonate)

CORPORATE EVENT PHOTOS:
├── Badge designs (for cloning)
├── Guest/visitor badge appearance
├── Physical security measures
├── Dress code (for physical pen test)
└── Catering (vendor impersonation)

# Remove EXIF data from your own photos!
exiftool -all= photo.jpg
```

---

## 5. Building a Social Engineering Pretext

```
FROM SOCIAL MEDIA INTELLIGENCE TO ATTACK:

TARGET: Jane Doe (Sysadmin at Target Corp)

SOCIAL MEDIA FINDINGS:
├── LinkedIn: MCSE certified, manages Azure
├── Twitter: Posts about cats, complains about Jira
├── Facebook: Runs with local running club Saturdays
├── Instagram: Photos from "Brew Fest 2024"
└── GitHub: Contributes to an open-source monitoring tool

PERSONALIZED ATTACK OPTIONS:

OPTION 1: Phishing email
Subject: "Jira Critical Security Update Required"
→ Exploits known frustration with Jira
→ Higher click-through than generic email

OPTION 2: Watering hole
→ Compromise the running club's website
→ She visits regularly → malware delivery

OPTION 3: Physical approach
→ Attend Saturday running club
→ Build rapport over weeks
→ Eventually discuss work topics

OPTION 4: Technical pretext
→ Call as "Azure support"
→ Reference her MCSE certification
→ "I see you manage the Azure tenant..."
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Sherlock** | Find usernames across 400+ platforms |
| **LinkedIn** | Professional details, connections |
| **Twitter/Reddit** | Unfiltered opinions, work complaints |
| **Photos** | Badges, screens, office layout |
| **EXIF data** | GPS coordinates in photo metadata |
| **Personalization** | Tailored attacks have highest success |

---

## Quick Revision Questions

1. **What tool finds usernames across multiple platforms?**
2. **What intelligence can office photos reveal?**
3. **How do social media posts help craft phishing emails?**
4. **What is EXIF data and why does it matter?**
5. **How does relationship mapping aid social engineering?**

---

[← Previous: Organizational Structure](02-organizational-structure.md) | [Next: Behavioral Patterns →](04-behavioral-patterns.md)

---

*Unit 7 - Topic 3 of 5 | [Back to README](../README.md)*
