# Unit 1: Security Operations Fundamentals — Topic 3: SOC Roles and Responsibilities

## Overview

A well-functioning SOC requires clearly defined **roles and responsibilities** organized in a tiered structure. Each role has specific skills, duties, and career progression paths. Understanding the SOC organizational structure is essential for building effective teams and developing security career paths.

---

## 1. SOC Organizational Structure

```
SOC TIERED STRUCTURE:

  ┌─────────────────────────────────────────────┐
  │              SOC LEADERSHIP                  │
  │  SOC Director / CISO                         │
  │  Strategy, Budget, Reporting                 │
  └──────────────────────┬──────────────────────┘
                         │
  ┌──────────────────────▼──────────────────────┐
  │              SOC MANAGER                     │
  │  Daily operations, staffing, process         │
  └──────────────────────┬──────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
  ┌─────▼─────┐  ┌──────▼──────┐  ┌─────▼─────┐
  │ TIER 3    │  │ ENGINEERING │  │ THREAT    │
  │ Senior    │  │             │  │ INTEL     │
  │ Analysts  │  │ Detection   │  │           │
  │           │  │ Content     │  │ Research  │
  │ Advanced  │  │ Tool Admin  │  │ Analysis  │
  │ IR, Hunt  │  │ Automation  │  │ Sharing   │
  └───────────┘  └─────────────┘  └───────────┘
        │
  ┌─────▼─────┐
  │ TIER 2    │
  │ Analysts  │
  │           │
  │ Deep      │
  │ Analysis  │
  │ IR        │
  └─────┬─────┘
        │
  ┌─────▼─────┐
  │ TIER 1    │
  │ Analysts  │
  │           │
  │ Monitor   │
  │ Triage    │
  │ Escalate  │
  └───────────┘
```

---

## 2. Tier 1: SOC Analyst (Junior)

```
TIER 1 SOC ANALYST:

  TITLE: SOC Analyst I / Security Monitor / Alert Analyst

  PRIMARY DUTIES:
  → Monitor security dashboards and alerts
  → Initial alert triage (true positive vs false positive)
  → Follow standard operating procedures (SOPs)
  → Execute basic playbook steps
  → Escalate confirmed incidents to Tier 2
  → Create and update tickets
  → Basic log review and analysis
  → Documentation of findings

  DAILY WORKFLOW:
  1. Check SIEM dashboard at shift start
  2. Review and triage new alerts
  3. Follow playbook for each alert type
  4. Document analysis in ticket
  5. Escalate if confirmed or uncertain
  6. Close false positives with notes
  7. End-of-shift handoff report

  REQUIRED SKILLS:
  → Understanding of networking (TCP/IP, DNS, HTTP)
  → Basic knowledge of operating systems
  → Familiarity with SIEM tools
  → Understanding of common attack types
  → Log analysis basics
  → Strong documentation skills
  → Attention to detail

  CERTIFICATIONS:
  → CompTIA Security+
  → CompTIA CySA+
  → Splunk Core Certified User
  → GIAC Security Essentials (GSEC)

  EXPERIENCE: 0-2 years
  SALARY RANGE: $50,000-$75,000
  
  ESCALATION CRITERIA:
  → Confirmed malware execution
  → Successful exploitation
  → Data exfiltration indicators
  → Unauthorized access
  → Alert outside playbook scope
  → Uncertain analysis
```

---

## 3. Tier 2: SOC Analyst (Senior)

```
TIER 2 SOC ANALYST:

  TITLE: SOC Analyst II / Incident Responder / Security Analyst

  PRIMARY DUTIES:
  → Deep-dive investigation of escalated alerts
  → Determine scope and impact of incidents
  → Perform root cause analysis
  → Execute containment actions
  → Coordinate with IT for remediation
  → Develop and refine detection rules
  → Mentor Tier 1 analysts
  → Write incident reports
  → Conduct basic threat hunting

  INVESTIGATION PROCESS:
  1. Receive escalation from Tier 1
  2. Validate and expand investigation
  3. Correlate across multiple data sources
  4. Determine attack scope
  5. Execute containment if needed
  6. Document findings and timeline
  7. Coordinate remediation
  8. Escalate to Tier 3 if needed

  REQUIRED SKILLS:
  → Advanced log analysis
  → Malware analysis basics
  → Network forensics
  → Endpoint forensics
  → Scripting (Python, PowerShell)
  → Advanced SIEM queries (SPL, KQL)
  → Incident response methodology
  → Threat intelligence consumption
  → Strong communication skills

  CERTIFICATIONS:
  → GIAC Certified Incident Handler (GCIH)
  → CompTIA CySA+
  → EC-Council ECIH
  → Splunk Certified Power User
  → SANS FOR508

  EXPERIENCE: 2-5 years
  SALARY RANGE: $75,000-$110,000
```

---

## 4. Tier 3 and Specialized Roles

```
TIER 3 SOC ANALYST / SUBJECT MATTER EXPERT:

  TITLE: Senior Security Analyst / Threat Hunter / IR Lead

  PRIMARY DUTIES:
  → Lead complex incident investigations
  → Proactive threat hunting
  → Advanced malware analysis
  → Develop detection strategies
  → Red team / Purple team exercises
  → Process improvement
  → Strategic threat assessment
  → Mentoring Tier 1 and 2

  EXPERIENCE: 5+ years
  SALARY RANGE: $110,000-$160,000

DETECTION ENGINEER:

  DUTIES:
  → Create and maintain detection rules
  → Develop SIEM correlation logic
  → Write Sigma/YARA rules
  → Tune rules to reduce false positives
  → Map detections to MITRE ATT&CK
  → Validate detection effectiveness
  → Automate detection testing
  → Content lifecycle management

  SKILLS:
  → SIEM query languages (SPL, KQL, Lucene)
  → Sigma rule creation
  → YARA rule writing
  → MITRE ATT&CK framework
  → Scripting (Python)
  → Understanding of attack techniques
  → Log source knowledge

THREAT INTELLIGENCE ANALYST:

  DUTIES:
  → Collect and analyze threat intelligence
  → Produce threat reports
  → Maintain threat intel platform
  → IOC management and enrichment
  → Support detection with intelligence
  → Track threat actors
  → Share intelligence with peers

SOC MANAGER:

  DUTIES:
  → Manage SOC team and operations
  → Define and track metrics
  → Staff scheduling and hiring
  → Budget management
  → Stakeholder communication
  → Process development
  → Technology roadmap
  → Team training and development

  EXPERIENCE: 7+ years
  SALARY RANGE: $130,000-$180,000

SOC ENGINEER / TOOL ADMINISTRATOR:

  DUTIES:
  → Deploy and maintain SOC tools
  → SIEM administration
  → Log source onboarding
  → Parser/connector development
  → SOAR playbook development
  → Performance optimization
  → Integration between tools
```

---

## 5. Career Progression

```
SOC CAREER PATH:

  ┌──────────────────────────────────────────┐
  │  ENTRY LEVEL                              │
  │  Help Desk / IT Support / NOC Analyst     │
  │  0-1 years                                │
  └───────────────────┬──────────────────────┘
                      │
  ┌───────────────────▼──────────────────────┐
  │  TIER 1 SOC ANALYST                       │
  │  Security monitoring, triage              │
  │  1-2 years                                │
  └───────────────────┬──────────────────────┘
                      │
  ┌───────────────────▼──────────────────────┐
  │  TIER 2 SOC ANALYST / IR                  │
  │  Investigation, incident response         │
  │  2-5 years                                │
  └───────────────────┬──────────────────────┘
                      │
        ┌─────────────┼──────────────┐
        │             │              │
  ┌─────▼──────┐ ┌───▼────┐ ┌──────▼──────┐
  │ Threat     │ │ Detection│ │ IR Lead /  │
  │ Hunter     │ │ Engineer │ │ Forensics  │
  └─────┬──────┘ └───┬────┘ └──────┬──────┘
        │             │              │
  ┌─────▼─────────────▼──────────────▼──────┐
  │  SOC MANAGER / IR MANAGER                │
  │  7-10 years                              │
  └───────────────────┬──────────────────────┘
                      │
  ┌───────────────────▼──────────────────────┐
  │  SOC DIRECTOR / CISO                      │
  │  10+ years                                │
  └──────────────────────────────────────────┘

SKILLS DEVELOPMENT:
  Stage         | Focus Areas
  Entry         | Networking, OS, Security fundamentals
  Tier 1        | SIEM, log analysis, common attacks
  Tier 2        | Forensics, IR, scripting, malware
  Specialist    | Deep expertise in chosen area
  Management    | Leadership, strategy, budgeting
```

---

## Summary Table

| Role | Level | Key Duties | Experience |
|------|-------|-----------|------------|
| Tier 1 Analyst | Junior | Monitor, triage, escalate | 0-2 years |
| Tier 2 Analyst | Mid | Investigate, respond, contain | 2-5 years |
| Tier 3 / SME | Senior | Hunt, lead IR, mentor | 5+ years |
| Detection Engineer | Specialist | Create/tune detection rules | 3-5 years |
| Threat Intel Analyst | Specialist | Intelligence analysis | 3-5 years |
| SOC Manager | Management | Team/operations leadership | 7+ years |

---

## Revision Questions

1. What are the primary responsibilities of a Tier 1 SOC analyst?
2. How do Tier 2 analysts differ from Tier 1 in their daily work?
3. What skills does a detection engineer need?
4. What are the escalation criteria from Tier 1 to Tier 2?
5. What career paths are available for SOC analysts?

---

*Previous: [02-soc-models.md](02-soc-models.md) | Next: [04-soc-metrics.md](04-soc-metrics.md)*

---

*[Back to README](../README.md)*
