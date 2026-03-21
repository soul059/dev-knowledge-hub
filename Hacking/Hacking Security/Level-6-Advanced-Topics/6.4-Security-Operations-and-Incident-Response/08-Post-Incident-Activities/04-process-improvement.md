# Unit 8: Post-Incident Activities — Topic 4: Process Improvement

## Overview

**Process improvement** uses incident insights to systematically enhance detection, response, and prevention capabilities. Every incident reveals gaps — in tools, processes, training, or architecture — and effective process improvement transforms these findings into measurable security enhancements.

---

## 1. Improvement Framework

```
CONTINUOUS IMPROVEMENT CYCLE:

  ┌───────────────┐
  │   INCIDENT    │
  └──────┬────────┘
         │
  ┌──────▼────────┐
  │ LESSONS       │
  │ LEARNED       │
  └──────┬────────┘
         │
  ┌──────▼────────┐
  │ IDENTIFY      │
  │ GAPS          │
  └──────┬────────┘
         │
  ┌──────▼────────┐
  │ PLAN          │
  │ IMPROVEMENTS  │
  └──────┬────────┘
         │
  ┌──────▼────────┐
  │ IMPLEMENT     │
  │ CHANGES       │
  └──────┬────────┘
         │
  ┌──────▼────────┐
  │ VALIDATE      │
  │ EFFECTIVENESS │
  └──────┬────────┘
         │
         └──────▶ (Next incident or exercise)

IMPROVEMENT AREAS:
  → Detection rules and coverage
  → Response playbooks and procedures
  → Tools and technology
  → Team skills and training
  → Communication processes
  → Architecture and controls
  → Policies and governance
  → Third-party management
```

---

## 2. Detection Improvements

```
DETECTION ENHANCEMENT:

POST-INCIDENT DETECTION UPGRADES:
  → Create detection rules for incident TTPs
  → Improve existing rules that missed activity
  → Add log sources that were missing
  → Tune rules to reduce false positives
  → Add threat intelligence indicators
  → Implement behavioral detection

DETECTION GAP ANALYSIS:
  ATT&CK Technique    | Had Detection? | Action
  T1566 Phishing       | Partial        | Improve email rules
  T1059 PowerShell     | No             | Add PS logging + rule
  T1003 Credential Dump| Yes            | Rule worked ✓
  T1021 Remote Services| No             | Add lateral move rules
  T1041 Exfiltration   | No             | Add DLP + proxy rules
  T1053 Scheduled Task | No             | Add Sysmon rule

NEW DETECTION RULES:
  # Rule 1: Suspicious PowerShell download
  index=sysmon EventCode=1 Image="*powershell.exe"
    (CommandLine="*downloadstring*" OR
     CommandLine="*downloadfile*" OR
     CommandLine="*invoke-webrequest*" OR
     CommandLine="*wget*" OR CommandLine="*curl*")
  | alert severity=high

  # Rule 2: Lateral movement via PsExec
  index=sysmon EventCode=1 
    (Image="*psexec*" OR 
     ParentImage="*services.exe" Image="*psexesvc*")
  | alert severity=critical

LOG SOURCE IMPROVEMENTS:
  → Enable PowerShell script block logging
  → Enable Sysmon on all endpoints
  → Enable enhanced audit logging
  → Add DNS query logging
  → Enable cloud audit logs
  → Add email gateway logging
```

---

## 3. Process and Playbook Updates

```
PLAYBOOK IMPROVEMENTS:

UPDATE EXISTING PLAYBOOKS:
  → Add steps that were missed
  → Remove steps that didn't work
  → Update tool commands
  → Update contact information
  → Add decision criteria
  → Include new IOC types

CREATE NEW PLAYBOOKS:
  → If no playbook existed for the incident type
  → Based on actual response steps taken
  → Include lessons learned
  → Peer review before publishing

EXAMPLE PLAYBOOK UPDATE:
  BEFORE (Phishing Playbook):
  1. Check email headers
  2. Block sender
  3. Search for similar emails
  4. Notify affected users

  AFTER (Updated based on incident):
  1. Check email headers
  2. Extract and sandbox URLs
  3. Block sender AND domain
  4. Search for similar emails (last 30 days)
  5. Check if any user clicked/submitted credentials
  6. If credentials submitted:
     a. Disable account immediately
     b. Revoke all active sessions
     c. Check for post-compromise activity
     d. Check for email forwarding rules
     e. Reset password + enable MFA
  7. Block URLs/IPs at proxy and firewall
  8. Notify affected users
  9. Update email filtering rules
  10. Submit to threat intelligence

IR PLAN UPDATES:
  → Update communication tree
  → Update escalation procedures
  → Update severity definitions
  → Update roles and responsibilities
  → Add new incident categories
  → Update regulatory notification procedures
```

---

## 4. Training and Exercises

```
TRAINING IMPROVEMENTS:

INCIDENT-DRIVEN TRAINING:
  → Train on actual incident TTPs
  → Develop scenarios from real incidents
  → Practice with tools used in response
  → Focus on identified skill gaps

TRAINING PLAN:
  Gap Identified        | Training           | Who
  PowerShell analysis   | SANS FOR508         | T2 analysts
  Cloud IR              | AWS IR course       | All IR staff
  Memory forensics      | Volatility training | Forensics team
  Threat hunting        | SANS FOR508         | Senior analysts
  Executive briefing    | Presentation skills | IR leads
  Phishing response     | Updated playbook    | All SOC

EXERCISES:
  Exercise Type    | Frequency | Purpose
  Tabletop         | Quarterly | Test plan, decision-making
  Functional drill | Bi-annual | Test technical procedures
  Full simulation  | Annual    | End-to-end test
  Purple team      | Quarterly | Test detection + response
  Phishing test    | Monthly   | Test user awareness

TABLETOP SCENARIO (from incident):
  "You receive an alert that a user's account
  is making unusual login attempts from an
  overseas IP. Walk through your response:
  - How do you triage?
  - What containment actions do you take?
  - What do you investigate?
  - Who do you notify?
  - How do you determine scope?
  
  This scenario is based on our actual incident
  INC-2024-001. Let's see if we would respond
  better with our improvements."
```

---

## 5. Technology and Architecture Improvements

```
SECURITY ARCHITECTURE IMPROVEMENTS:

POST-INCIDENT SECURITY UPGRADES:
  Finding                | Improvement
  No MFA                 | Deploy MFA for all remote access
  Flat network           | Implement segmentation
  No email filtering     | Deploy advanced email security
  No DLP                 | Implement DLP solution
  Poor logging           | Deploy SIEM + enhanced logging
  No EDR                 | Deploy EDR on all endpoints
  Weak passwords         | Implement PAM + password policy
  No backup testing      | Monthly backup validation

PRIORITIZATION MATRIX:
  Priority | Effort | Impact | Examples
  Quick Win| Low    | High   | Enable MFA, tune rules
  Strategic| High   | High   | Network segmentation
  Tactical | Low    | Medium | New detection rules
  Deferred | High   | Low    | Platform migration

TECHNOLOGY ROADMAP:
  Phase 1 (0-30 days):
  → Deploy MFA
  → Update detection rules
  → Enable enhanced logging
  → Patch critical vulnerabilities

  Phase 2 (30-90 days):
  → Deploy email security
  → Implement network segmentation
  → Deploy DLP
  → Upgrade SIEM capacity

  Phase 3 (90+ days):
  → Implement zero-trust architecture
  → Deploy PAM solution
  → Implement SOAR
  → Build threat hunting program

TRACKING IMPROVEMENTS:
  → Measure before and after metrics
  → Track implementation progress
  → Report improvements to leadership
  → Validate effectiveness through exercises
  → Adjust based on new incidents
```

---

## Summary Table

| Area | Actions | Timing | Validation |
|------|---------|--------|-----------|
| Detection | New rules, tune existing | 1-2 weeks | Purple team test |
| Playbooks | Update or create new | 2-4 weeks | Tabletop exercise |
| Training | Skills gaps, scenarios | 1-3 months | Certification, drill |
| Technology | Tools, architecture | 1-12 months | Vulnerability scan |
| Process | IR plan, communication | 2-4 weeks | Tabletop exercise |

---

## Revision Questions

1. What is the continuous improvement cycle for incident response?
2. How should detection gaps be identified and addressed?
3. What updates should be made to playbooks after an incident?
4. How do exercises validate process improvements?
5. How should technology improvements be prioritized?

---

*Previous: [03-metrics-and-kpis.md](03-metrics-and-kpis.md) | Next: [05-knowledge-management.md](05-knowledge-management.md)*

---

*[Back to README](../README.md)*
