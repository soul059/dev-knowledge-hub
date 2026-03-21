# Unit 2: Reconnaissance for Social Engineering — Topic 4: Social Media Intelligence

## Overview

**Social media intelligence (SOCMINT)** is the collection and analysis of information from social media platforms to support social engineering operations. People share enormous amounts of personal and professional information on social media, creating a rich data source for building pretexts, identifying targets, and crafting personalized attacks.

---

## 1. Platform-Specific Intelligence

```
WHAT EACH PLATFORM REVEALS:

LINKEDIN (Professional):
  → Current employer and job title
  → Work history and career progression
  → Education and certifications
  → Professional connections
  → Published articles and activity
  → Group memberships
  → Skills and endorsements
  → Recommendations (reveal projects/relationships)
  → Job change notifications
  USE: Impersonation, business pretexts, org mapping

FACEBOOK (Personal):
  → Family members and relationships
  → Birthday and hometown
  → Photos (locations, events, lifestyle)
  → Check-ins and places visited
  → Political/religious views
  → Groups and communities
  → Upcoming events (travel, conferences)
  USE: Personal pretexts, security question answers

TWITTER/X (Opinions):
  → Real-time opinions and complaints
  → Technology preferences
  → Conference attendance (#conference)
  → Interactions with companies
  → Frustrations with work/tools
  → Political views and biases
  USE: Emotional hooks, watering hole targeting

INSTAGRAM (Lifestyle):
  → Travel patterns and locations
  → Daily routines (gym, coffee shops)
  → Family and pet names
  → Hobbies and interests
  → Workplace photos (badges visible!)
  USE: Physical surveillance, pretext details

GITHUB (Technical):
  → Coding skills and languages
  → Email addresses in commits
  → API keys accidentally committed
  → Internal project names
  → Technical interests
  USE: Technical pretexts, credential discovery
```

---

## 2. Intelligence Collection Techniques

```
COLLECTION METHODS:

PASSIVE COLLECTION (No interaction):
  → View public profiles
  → Search posts and comments
  → Monitor hashtags and mentions
  → Use Google to index social profiles
  → Archive pages (Wayback Machine)
  → Use anonymous viewing (LinkedIn)

ACTIVE COLLECTION (Interaction):
  → Send connection/friend requests
  → Join same groups
  → Engage in discussions
  → Create fake profiles (sock puppets)
  → Follow targets on multiple platforms
  ⚠ HIGHER RISK of detection!

AUTOMATED COLLECTION:
  → Social media monitoring tools
  → Automated scrapers (ethical/legal concerns)
  → OSINT frameworks (Maltego, SpiderFoot)
  → Google alerts for target names
  → RSS feeds for company mentions

SEARCH OPERATORS:
  LinkedIn: "current company" AND "job title"
  Twitter: from:username keyword
  Facebook: Photos of [person] / Posts by [person]
  Google: site:linkedin.com "Company Name" "IT Manager"
  Google: site:twitter.com "@company" "password" OR "locked out"
```

---

## 3. Attack Vectors from Social Media

```
TURNING SOCIAL MEDIA INTO ATTACKS:

SCENARIO 1: CONFERENCE ATTENDANCE
  Discovery: Target tweeted "Excited for #DefCon2024!"
  Attack: "Hi, I met you at DefCon. I'm following up 
          on that tool we discussed. Here's the link..."
  Why it works: Shared experience creates trust

SCENARIO 2: COMPANY COMPLAINT
  Discovery: "Ugh, [company tool] went down again 🙄"
  Attack: "Hi, this is [company tool] support. We 
          noticed the outage affected your account..."
  Why it works: Addresses real frustration

SCENARIO 3: LIFE EVENT
  Discovery: "Just started my new role at [Company]!"
  Attack: "Welcome! HR has a few forms for new hires.
          Please complete this onboarding checklist..."
  Why it works: New employees unfamiliar with processes

SCENARIO 4: TRAVEL
  Discovery: CFO posting from vacation in Bali
  Attack: BEC email to finance team: "The CFO is 
          traveling and needs an urgent wire transfer..."
  Why it works: Explains why CEO can't be reached

SCENARIO 5: PET/FAMILY NAMES
  Discovery: Instagram shows dog named "Bailey"
  Attack: Password guessing (Bailey2024!)
  Also: Security question answers visible
  "What's your pet's name?" → Bailey

SCENARIO 6: BADGE PHOTOS
  Discovery: Instagram selfie with company badge visible
  Attack: Clone badge design for impersonation
  Also: Badge number, access areas, photo for fake ID
```

---

## 4. OPSEC for Social Media Recon

```
OPERATIONAL SECURITY:

DO:
  ✓ Use VPN/Tor for browsing
  ✓ Create dedicated research accounts
  ✓ Use private/incognito browsing
  ✓ Screenshot and save locally (don't bookmark)
  ✓ Track what you've accessed
  ✓ Use different accounts per platform

DON'T:
  ✗ Use personal social media accounts
  ✗ Like/react to target's posts
  ✗ View LinkedIn profiles (shows in "who viewed")
  ✗ Follow targets from identifiable accounts
  ✗ Leave traces that link to your real identity
  ✗ Access from company network (IP logged)

SOCK PUPPET ACCOUNTS:
  → Create realistic fake profiles
  → Age the accounts (don't use brand new)
  → Build history and connections
  → Match target's industry/location
  → Use stock photos or AI-generated faces
  → Maintain consistent persona
```

---

## Summary Table

| Platform | Primary Intelligence | SE Application |
|----------|---------------------|---------------|
| LinkedIn | Professional details | Business pretexts |
| Facebook | Personal life, family | Security questions |
| Twitter/X | Opinions, events | Emotional hooks |
| Instagram | Location, routine | Physical access |
| GitHub | Technical skills | Tech impersonation |

---

## Revision Questions

1. What unique intelligence does each social media platform provide?
2. How can conference attendance be leveraged for social engineering?
3. What OPSEC measures should be followed during social media recon?
4. Give three examples of turning social media posts into attack vectors.
5. What risks do badge photos on social media create?

---

*Previous: [03-organizational-reconnaissance.md](03-organizational-reconnaissance.md) | Next: [05-pretexting-research.md](05-pretexting-research.md)*

---

*[Back to README](../README.md)*
