# Unit 3: Phishing — Topic 3: Spear Phishing

## Overview

**Spear phishing** is a targeted phishing attack directed at specific individuals or organizations. Unlike mass phishing, spear phishing uses personalized information gathered through OSINT to craft highly convincing emails that reference the target's name, role, projects, colleagues, and interests. Success rates are 50-65% compared to 3-5% for mass phishing.

---

## 1. What Makes Spear Phishing Different

```
MASS PHISHING vs SPEAR PHISHING:

MASS PHISHING:                SPEAR PHISHING:
→ Generic content             → Personalized to individual
→ Sent to thousands           → Sent to 1-10 targets
→ "Dear Customer"             → "Hi Sarah"
→ Generic pretext             → Reference real projects
→ 3-5% success               → 50-65% success
→ Low effort per email        → High effort per email
→ Volume-based                → Quality-based
→ Easy to detect              → Hard to detect
→ Broad targeting             → Specific targeting
```

---

## 2. Crafting Spear Phishing Emails

```
PERSONALIZATION ELEMENTS:

LEVEL 1 (Basic):
  → Target's name
  → Company name
  → Job title
  → Generic industry reference

LEVEL 2 (Intermediate):
  → Manager's name
  → Department name
  → Specific project reference
  → Company-specific terminology
  → Recent company news reference

LEVEL 3 (Advanced):
  → Reference to recent conversation/meeting
  → Specific colleague mentions
  → Internal tool/system names
  → Current project deadlines
  → Personal interests or events
  → Mimics writing style of known sender

EXAMPLE — LEVEL 3 SPEAR PHISH:

Subject: Re: Q3 Budget Review - Updated Projections

Hi Sarah,

Following up on our discussion with Mark yesterday
about the Q3 projections. I've updated the 
spreadsheet with the revised figures from the 
Portland office.

Can you review and approve before the board 
meeting on Thursday?

Updated projections: [malicious link]

Thanks,
David Chen
VP Finance | Ext. 4521

→ Uses real names (Sarah, Mark, David)
→ References real project (Q3 Budget)
→ Mentions real office (Portland)
→ Creates deadline pressure (Thursday board meeting)
→ Mimics internal communication style
→ Includes realistic signature
```

---

## 3. Spear Phishing Process

```
SPEAR PHISHING WORKFLOW:

PHASE 1: TARGET SELECTION
  → Identify person with desired access
  → Research their role and responsibilities
  → Map their communication patterns
  → Identify potential senders to impersonate

PHASE 2: INTELLIGENCE GATHERING
  → LinkedIn: role, connections, projects
  → Social media: interests, events, writing style
  → Company website: news, team, structure
  → Email format: firstname.lastname@company.com
  → Recent activities: conferences, job changes

PHASE 3: PRETEXT DEVELOPMENT
  → Choose scenario relevant to target's role
  → Select sender to impersonate (boss, colleague, vendor)
  → Create urgency tied to real events
  → Prepare payload (link, attachment, or both)

PHASE 4: INFRASTRUCTURE
  → Register lookalike domain
  → Set up email sending
  → Configure SPF/DKIM for sending domain
  → Build credential harvesting page
  → Test email deliverability

PHASE 5: DELIVERY
  → Send at optimal time (Tuesday-Thursday, 9-11 AM)
  → Monitor for clicks/interactions
  → Follow up if no response
  → Track campaign metrics

PHASE 6: EXPLOITATION
  → Harvest credentials
  → Deliver malware payload
  → Capture session tokens
  → Establish persistence
```

---

## 4. OSINT for Spear Phishing

```
INTELLIGENCE SOURCES:

PROFESSIONAL:
  LinkedIn:
    → Job title, department, responsibilities
    → Connections (who to impersonate)
    → Endorsements (technologies used)
    → Posts (writing style, interests)
    → Job changes (new employee targeting)
  
  Company Website:
    → Organizational structure
    → Press releases (project names)
    → Team pages (names, roles, photos)
    → Contact information
    → Annual reports (financial terminology)

PERSONAL:
  Social Media:
    → Facebook: personal interests, events, family
    → Twitter/X: opinions, complaints, tech preferences
    → Instagram: locations, activities, lifestyle
    → GitHub: technical skills, project involvement

TECHNICAL:
  → Email format discovery (hunter.io)
  → Domain registration (whois)
  → DNS records (MX, SPF)
  → Job postings (technologies used)
  → Data breach databases (prior credentials)
```

---

## 5. Defending Against Spear Phishing

```
DEFENSE LAYERS:

TECHNICAL CONTROLS:
  → External email banners
  → DMARC enforcement (p=reject)
  → Advanced threat protection (ATP)
  → URL rewriting and safe links
  → Attachment sandboxing
  → MFA everywhere

HUMAN CONTROLS:
  → Targeted awareness training
  → Simulated spear phishing exercises
  → Verification procedures for sensitive requests
  → Out-of-band confirmation for financial requests
  → Encourage reporting without punishment

PROCESS CONTROLS:
  → Dual authorization for wire transfers
  → Callback verification for vendor changes
  → Change management for banking details
  → Separation of duties for financial actions
```

---

## Summary Table

| Aspect | Mass Phishing | Spear Phishing |
|--------|--------------|----------------|
| Targeting | Random/broad | Specific individual |
| Research | None/minimal | Extensive OSINT |
| Personalization | None | Highly personalized |
| Success rate | 3-5% | 50-65% |
| Volume | Thousands | 1-10 |
| Detection | Easier | Much harder |
| Defense | Spam filters | Awareness + verification |

---

## Revision Questions

1. What makes spear phishing significantly more effective than mass phishing?
2. What are the three levels of personalization in spear phishing?
3. What OSINT sources are most valuable for crafting spear phishing emails?
4. Describe the six phases of a spear phishing attack.
5. What defense strategies are most effective against spear phishing?

---

*Previous: [02-email-phishing.md](02-email-phishing.md) | Next: [04-whaling.md](04-whaling.md)*

---

*[Back to README](../README.md)*
