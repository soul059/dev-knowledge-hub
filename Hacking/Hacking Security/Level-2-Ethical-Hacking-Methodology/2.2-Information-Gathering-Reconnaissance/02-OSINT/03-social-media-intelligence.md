# Social Media Intelligence

## Unit 2 - Topic 3 | OSINT (Open Source Intelligence)

---

## Overview

**Social Media Intelligence (SOCMINT)** involves gathering information from social media platforms to build profiles of target organizations and individuals. People share an extraordinary amount of information online — office photos, technology discussions, travel plans, and even credentials.

---

## 1. Key Platforms for Intelligence

| Platform | Intel Value | What to Find |
|----------|:---:|------------|
| **LinkedIn** | ★★★★★ | Employees, roles, tech stack, org structure |
| **Twitter/X** | ★★★★ | Announcements, opinions, tech discussions |
| **GitHub** | ★★★★ | Source code, secrets, developer identities |
| **Facebook** | ★★★ | Personal info, events, group memberships |
| **Instagram** | ★★ | Office photos, events, badge designs |
| **Reddit** | ★★★ | Tech discussions, employee complaints |
| **Stack Overflow** | ★★★ | Tech questions (reveal internal details) |
| **Glassdoor** | ★★★ | Company culture, tech stack, interview Q's |

---

## 2. LinkedIn Intelligence

```
LINKEDIN INTELLIGENCE GATHERING:

EMPLOYEE ENUMERATION:
├── Search: "target company" on LinkedIn
├── Filter by: Current employees
├── Collect: Names, titles, departments
├── Identify: IT staff, security team, executives
├── Note: Tech skills listed in profiles

EMAIL FORMAT DISCOVERY:
├── Find 1-2 verified emails (Hunter.io)
├── Determine format: first.last@, flast@, etc.
├── Apply format to all discovered names
└── Validate with email verification tools

TECHNOLOGY IDENTIFICATION:
├── Job postings list required skills
├── Employee profiles show tech expertise
├── "We use [technology]" in posts/articles
└── Certifications reveal products in use

ORGANIZATIONAL STRUCTURE:
├── Map reporting relationships
├── Identify key decision makers
├── Find recently hired employees (easier targets)
└── Identify departing employees (potential grievances)
```

---

## 3. Extracting Technical Intel

```
EXAMPLE: JOB POSTING ANALYSIS

"Senior DevOps Engineer — Target Corp"
Required skills:
• Kubernetes and Docker         → Container infrastructure
• AWS (EC2, S3, Lambda)        → Cloud provider = AWS
• Terraform                     → Infrastructure as code
• Jenkins CI/CD                 → Build pipeline
• PostgreSQL and Redis          → Database stack
• Datadog monitoring            → Monitoring tool
• Python and Go                 → Programming languages

INTEL DERIVED:
├── Cloud: AWS
├── Containers: Kubernetes + Docker
├── Database: PostgreSQL + Redis
├── CI/CD: Jenkins
├── Monitoring: Datadog
├── Languages: Python, Go
└── IaC: Terraform
```

---

## 4. Social Media OPSEC Failures

```
COMMON INFORMATION LEAKS:

OFFICE PHOTOS:
├── Badge designs visible → clone physical badges
├── Monitor screens visible → internal tools/URLs
├── Whiteboard content → project details
├── Door codes visible → physical access
└── Network equipment visible → model numbers

PERSONAL POSTS:
├── Vacation announcements → office empty, targets unavailable
├── "Working late on [project]" → project timeline intel
├── Complaints about [system] → target weaknesses
├── Conference attendance → social engineering opportunity
└── Geotagged photos → physical locations

TECHNICAL DISCUSSIONS:
├── "Just migrated to [platform]" → infrastructure changes
├── "Debugging [issue] at work" → system details
├── Stack Overflow questions from @target.com → code/config details
└── GitHub commits with company email → code repositories
```

---

## 5. Social Media Recon Tools

```bash
# === LINKEDIN ===
# CrossLinked — LinkedIn scraping for email generation
crosslinked -f '{first}.{last}@target.com' -t 'Target Company'

# === TWITTER/X ===
# twint — Twitter scraping
twint -s "target.com" -o output.txt

# === GENERAL ===
# Sherlock — find usernames across platforms
sherlock username

# Holehe — check email registration on sites
holehe email@target.com

# Maltego — visual link analysis
# Connects people, organizations, and digital assets

# SpiderFoot — automated OSINT
spiderfoot -s target.com -m all
```

---

## Summary Table

| Platform | Primary Intel | Best For |
|----------|-------------|---------|
| **LinkedIn** | Employees, tech stack | Org mapping, email enumeration |
| **GitHub** | Source code, secrets | Technical vulnerabilities |
| **Twitter** | Real-time intel | Current events, opinions |
| **Job postings** | Technology stack | Identifying target infrastructure |
| **Instagram** | Physical intel | Badge cloning, office layout |

---

## Quick Revision Questions

1. **Why is LinkedIn the most valuable platform for OSINT?**
2. **How do job postings reveal technical infrastructure?**
3. **What intelligence can be gathered from office photos?**
4. **How do you derive email format from social media?**
5. **Name 3 tools for social media reconnaissance.**

---

[← Previous: Search Engine Techniques](02-search-engine-techniques.md) | [Next: Public Records →](04-public-records.md)

---

*Unit 2 - Topic 3 of 6 | [Back to README](../README.md)*
