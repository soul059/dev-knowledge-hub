# Unit 2: Reconnaissance for Social Engineering — Topic 1: Target Profiling

## Overview

**Target profiling** is the systematic process of gathering and organizing information about an organization and its employees to identify the most effective social engineering attack vectors. A thorough profile reveals organizational structure, key personnel, communication patterns, security posture, and individual vulnerabilities.

---

## 1. Organizational Profiling

```
ORGANIZATIONAL PROFILE ELEMENTS:

COMPANY INFORMATION:
  → Full legal name and subsidiaries
  → Industry and market position
  → Size (employees, revenue, locations)
  → Recent news (mergers, layoffs, expansion)
  → Technology stack (from job postings)
  → Partners, vendors, clients
  → Corporate culture and values
  → Physical locations and layout

ORGANIZATIONAL STRUCTURE:
  → Executive team (CEO, CFO, CTO, CISO)
  → Department heads and managers
  → Reporting hierarchies
  → Key decision-makers for financial/IT
  → Administrative assistants to executives
  → Help desk / IT support structure

COMMUNICATION PATTERNS:
  → Email format (first.last@company.com?)
  → Internal tools (Slack, Teams, Zoom?)
  → Communication style (formal/informal)
  → Frequency of all-staff emails
  → External communication channels

SECURITY POSTURE:
  → Security team size and structure
  → Awareness training programs
  → Physical security measures
  → Remote work policies
  → Visitor management procedures
  → Badge/ID systems used
```

---

## 2. Individual Profiling

```
INDIVIDUAL TARGET PROFILE:

PERSONAL INFORMATION:
  → Full name and known aliases
  → Job title and responsibilities
  → Department and manager
  → Contact info (email, phone, office)
  → Employment history
  → Education background
  → Professional certifications

SOCIAL MEDIA PRESENCE:
  → LinkedIn: career details, connections, activity
  → Twitter/X: opinions, interests, activity times
  → Facebook: personal info, friends, events
  → Instagram: lifestyle, locations, routines
  → GitHub: technical skills, projects
  → Personal website/blog

PERSONALITY INDICATORS:
  → Communication style (formal, casual)
  → Interests and hobbies
  → Values and motivations
  → Potential emotional triggers
  → Technical proficiency level
  → Openness to social interaction

VULNERABILITY FACTORS:
  → New to the organization (less suspicious)
  → High workload (less attention to verify)
  → Helpful personality (hard to say no)
  → Authority-respecting (follows orders)
  → Isolated role (fewer people to verify with)
  → Access to sensitive systems/data
```

---

## 3. Profiling Methodology

```
TARGET PROFILING WORKFLOW:

1. DEFINE OBJECTIVES
   → What do we need? (credentials, access, info)
   → What systems/data are the goal?
   → Who has access to those systems?

2. CAST WIDE NET
   → Identify all potential targets
   → Map organizational structure
   → List departments with access
   → Include support staff, vendors

3. NARROW DOWN
   → Score targets on: access, vulnerability, likelihood
   → Prioritize high-access + high-vulnerability
   → Consider backup targets if primary fails

4. DEEP DIVE
   → Intensive research on top 3-5 targets
   → Full social media analysis
   → Communication pattern study
   → Relationship mapping

5. DEVELOP APPROACH
   → Design pretext tailored to target
   → Choose communication channel
   → Plan timing (day of week, time)
   → Prepare fallback scenarios

SCORING MATRIX:
┌──────────────┬───────┬───────────────┬───────┐
│ Target       │Access │Vulnerability  │ Score │
├──────────────┼───────┼───────────────┼───────┤
│ Exec Assist. │ High  │ High (helpful)│ 9/10  │
│ Help Desk    │ High  │ High (trained)│ 8/10  │
│ New Employee │ Low   │ Very High     │ 5/10  │
│ CFO          │ V.High│ Low (aware)   │ 6/10  │
│ IT Admin     │ V.High│ Low (tech)    │ 5/10  │
│ Receptionist │ Med   │ High          │ 7/10  │
└──────────────┴───────┴───────────────┴───────┘
```

---

## 4. Profile Documentation

```
TARGET PROFILE TEMPLATE:

═══════════════════════════════════════
TARGET PROFILE: [Name]
═══════════════════════════════════════

BASIC INFO:
  Name:       [Full Name]
  Title:      [Job Title]
  Department: [Department]
  Location:   [Office / Remote]
  Email:      [email@company.com]
  Phone:      [Office / Mobile]

BACKGROUND:
  Education:  [School, Degree]
  Experience: [Years at company, previous roles]
  Skills:     [Technical, management, etc.]

ONLINE PRESENCE:
  LinkedIn:   [Profile URL, activity level]
  Twitter:    [Handle, topics of interest]
  Facebook:   [Privacy level, visible info]
  Other:      [Any other profiles]

PERSONALITY ASSESSMENT:
  Communication: [Formal/Casual/Friendly]
  Tech Level:    [Expert/Intermediate/Basic]
  Helpfulness:   [High/Medium/Low]
  Risk Profile:  [Conservative/Open/Trusting]

CONNECTIONS:
  Reports to: [Manager name]
  Manages:    [Team members]
  Works with: [Key colleagues]
  Vendors:    [External contacts]

APPROACH STRATEGY:
  Pretext:    [What story to use]
  Channel:    [Email/Phone/In-person]
  Timing:     [Best day/time]
  Hook:       [What will get attention]
  Ask:        [What we need from them]

RISK ASSESSMENT:
  Detection Risk:  [Low/Medium/High]
  Escalation Risk: [Low/Medium/High]
  Success Chance:  [Low/Medium/High]
═══════════════════════════════════════
```

---

## Summary Table

| Profiling Area | Key Data Points | Sources |
|---------------|----------------|---------|
| Organization | Structure, culture, tech | Website, LinkedIn, news |
| Individual | Role, personality, habits | Social media, OSINT |
| Vulnerability | Access, susceptibility | Job postings, behavior |
| Approach | Pretext, channel, timing | Analysis of all data |

---

## Revision Questions

1. What elements make up a comprehensive organizational profile?
2. How do you score and prioritize social engineering targets?
3. What individual factors increase vulnerability to social engineering?
4. What sources provide the most useful profiling information?
5. Why is profiling essential before attempting a social engineering attack?

---

*Previous: None (First topic in this unit) | Next: [02-osint-for-social-engineering.md](02-osint-for-social-engineering.md)*

---

*[Back to README](../README.md)*
