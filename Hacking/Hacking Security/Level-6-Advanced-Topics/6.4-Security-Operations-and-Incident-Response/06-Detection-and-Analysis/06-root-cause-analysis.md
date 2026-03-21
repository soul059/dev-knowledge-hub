# Unit 6: Detection and Analysis — Topic 6: Root Cause Analysis

## Overview

**Root cause analysis (RCA)** identifies the fundamental reason a security incident occurred, going beyond the immediate technical cause to uncover the systemic issues that allowed the attack to succeed. Effective RCA drives meaningful security improvements and prevents recurrence.

---

## 1. RCA Methodology

```
ROOT CAUSE ANALYSIS PROCESS:

  ┌──────────────────────────────────────┐
  │        ROOT CAUSE ANALYSIS           │
  │                                      │
  │  1. Define the Problem               │
  │     What happened? Impact?           │
  │                                      │
  │  2. Collect Data                     │
  │     Timeline, evidence, interviews   │
  │                                      │
  │  3. Identify Contributing Factors    │
  │     Technical, process, human        │
  │                                      │
  │  4. Determine Root Cause             │
  │     Why did it happen ultimately?    │
  │                                      │
  │  5. Recommend Corrective Actions     │
  │     How to prevent recurrence?       │
  │                                      │
  │  6. Implement and Track              │
  │     Execute fixes, verify effective  │
  └──────────────────────────────────────┘

RCA TECHNIQUES:

  5 WHYS:
  Problem: Ransomware encrypted 50 servers
  Why 1: Malware executed on servers
  Why 2: Attacker moved laterally from workstation
  Why 3: Workstation compromised via phishing
  Why 4: User clicked malicious link in email
  Why 5: No email filtering for malicious URLs
  → Root Cause: Inadequate email security controls

  FISHBONE (ISHIKAWA) DIAGRAM:
  People──────┐
  Process─────┤
  Technology──┤───▶ [INCIDENT]
  Policy──────┤
  Environment─┘
```

---

## 2. Root Cause Categories

```
COMMON ROOT CAUSES:

TECHNICAL ROOT CAUSES:
  → Unpatched vulnerability exploited
  → Misconfigured security controls
  → Weak authentication (no MFA)
  → Inadequate network segmentation
  → Missing endpoint protection
  → Insufficient logging/monitoring
  → Outdated software/OS
  → Default credentials in use
  → Exposed management interfaces

PROCESS ROOT CAUSES:
  → No patch management process
  → Inadequate change management
  → No security review for changes
  → Missing access review process
  → No vulnerability management program
  → Inadequate onboarding/offboarding
  → No third-party risk management

PEOPLE ROOT CAUSES:
  → Insufficient security awareness
  → Social engineering susceptibility
  → Insider threat (malicious or negligent)
  → Skills gap in security team
  → Understaffed security function
  → Burnout/fatigue leading to errors

POLICY/GOVERNANCE ROOT CAUSES:
  → No security policy
  → Policy not enforced
  → No risk assessment process
  → Inadequate budget for security
  → Lack of executive support
  → No incident response plan

ROOT CAUSE MATRIX:
  Incident Type    | Common Root Cause
  Phishing         | No email security + awareness gaps
  Ransomware       | Unpatched + no segmentation
  Data breach      | Weak access controls + no DLP
  Account takeover | No MFA + weak passwords
  Web app attack   | Unpatched + no WAF + no code review
  Insider threat   | No access reviews + no monitoring
```

---

## 3. Contributing Factor Analysis

```
CONTRIBUTING FACTORS:

  Root cause is the primary reason, but contributing
  factors are conditions that enabled or worsened
  the incident.

EXAMPLE ANALYSIS:
  INCIDENT: Ransomware outbreak affecting 50 servers

  ROOT CAUSE:
  → Unpatched VPN vulnerability (CVE-2024-XXXX)

  CONTRIBUTING FACTORS:
  → Patch management: Patch available for 60 days
     but not applied (process gap)
  → Network segmentation: Flat network allowed
     lateral movement (architecture gap)
  → Backup: Backups not tested, 30% failed
     recovery (process gap)
  → Monitoring: No alert for exploitation attempt
     (detection gap)
  → Access: Admin credentials shared among team
     (policy gap)
  → Response: IR plan outdated, team untrained
     (preparation gap)

ANALYSIS TABLE:
  Factor              | Category  | Impact | Preventable?
  Unpatched VPN       | Technical | Primary| Yes (patching)
  Flat network        | Technical | High   | Yes (segment)
  No exploit detection| Detection | Medium | Yes (IDS rule)
  Shared admin creds  | Policy    | Medium | Yes (PAM)
  Untested backups    | Process   | High   | Yes (testing)
  Outdated IR plan    | Process   | Medium | Yes (exercises)
```

---

## 4. Corrective Actions

```
CORRECTIVE ACTION PLAN:

  For each root cause and contributing factor,
  define corrective actions:

  Finding                | Corrective Action       | Priority
  Unpatched VPN          | Implement patch SLA:     | Critical
                         | Critical = 72 hours      |
  Flat network           | Implement microsegment   | High
                         | at ion for server VLANs  |
  No exploit detection   | Deploy IDS rules for     | High
                         | known CVEs               |
  Shared admin creds     | Implement PAM solution   | High
                         | Unique credentials       |
  Untested backups       | Monthly backup restore   | High
                         | testing                  |
  Outdated IR plan       | Update plan, quarterly   | Medium
                         | tabletop exercises       |

ACTION TYPES:
  → Immediate: Fix the specific vulnerability
  → Short-term: Address contributing factors
  → Long-term: Fix systemic issues
  → Strategic: Organizational/cultural changes

TRACKING:
  Action ID | Description         | Owner  | Due    | Status
  RCA-001   | Patch VPN appliance | IT Ops | 48hrs  | Done
  RCA-002   | Network segmentation| NetEng | 90 days| In progress
  RCA-003   | Deploy IDS rules    | SecOps | 7 days | Done
  RCA-004   | Implement PAM       | SecEng | 120days| Planning
  RCA-005   | Backup testing      | IT Ops | 30 days| In progress
  RCA-006   | Update IR plan      | IR Mgr | 60 days| In progress
```

---

## 5. Lessons Learned

```
LESSONS LEARNED MEETING:

TIMING: 1-2 weeks after incident closure
ATTENDEES: All involved parties
FACILITATOR: Neutral party (not IC)
DURATION: 1-2 hours

AGENDA:
  1. Incident summary (timeline, impact)
  2. What went well?
  3. What didn't go well?
  4. Root cause findings
  5. Corrective actions review
  6. Process improvement opportunities
  7. Detection improvements needed
  8. Open discussion

GROUND RULES:
  → Blameless culture
  → Focus on process, not people
  → Document all observations
  → Action items with owners
  → Follow up on action items

KEY QUESTIONS:
  → Were we prepared for this type of incident?
  → How quickly did we detect the incident?
  → Was the IR plan effective?
  → Were playbooks adequate?
  → Did communication work well?
  → Were the right tools available?
  → What would we do differently?
  → What training is needed?

OUTPUT:
  → Lessons learned report
  → Corrective action plan
  → Updated detection rules
  → Updated playbooks
  → Training plan
  → Process improvements
  → Strategic recommendations
```

---

## Summary Table

| RCA Component | Purpose | Output |
|--------------|---------|--------|
| 5 Whys | Find root cause | Primary cause identified |
| Contributing factors | Identify enablers | Factor list with impact |
| Corrective actions | Fix and prevent | Action plan with owners |
| Lessons learned | Organizational learning | Improvement roadmap |

---

## Revision Questions

1. What is the 5 Whys technique and how is it applied to RCA?
2. What are the main categories of root causes?
3. How do contributing factors differ from root causes?
4. What should a corrective action plan include?
5. How should a lessons learned meeting be conducted?

---

*Previous: [05-timeline-development.md](05-timeline-development.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
