# Chapter 4.2: Incident Response Process

[← Previous: Incident Classification](01-incident-classification.md) | [Back to README](../README.md) | [Next: On-Call Management →](03-on-call-management.md)

---

## Overview

The incident response process is a structured framework for detecting, triaging, mitigating, and resolving service disruptions. An effective process minimizes Mean Time to Recovery (MTTR) by ensuring clear roles, communication channels, and decision-making authority are established before incidents occur.

---

## 1. Incident Lifecycle

```
  ┌──────────────────────────────────────────────────────────┐
  │              INCIDENT LIFECYCLE                          │
  │                                                          │
  │  DETECT ──▶ TRIAGE ──▶ MITIGATE ──▶ RESOLVE ──▶ LEARN   │
  │                                                          │
  │  ┌──────┐  ┌──────┐   ┌────────┐  ┌───────┐  ┌──────┐  │
  │  │Alert │  │Assess│   │Stop the│  │Fix the│  │Post- │  │
  │  │fires │  │impact│   │bleeding│  │root   │  │mortem│  │
  │  │      │  │assign│   │restore │  │cause  │  │action│  │
  │  │      │  │roles │   │service │  │       │  │items │  │
  │  └──────┘  └──────┘   └────────┘  └───────┘  └──────┘  │
  │                                                          │
  │  MTTD     MTTA      MITIGATION   RESOLUTION   LEARNING  │
  │  ◄───▶    ◄───▶     ◄────────▶   ◄────────▶   ◄──────▶  │
  │                                                          │
  │  ◄──────────────── MTTR ────────────────────▶            │
  │  (Mean Time to Recovery)                                 │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Incident Roles (ICS Model)

```
  ┌──────────────────────────────────────────────────────────┐
  │  ROLE                │ RESPONSIBILITY                    │
  ├──────────────────────┼──────────────────────────────────┤
  │ Incident Commander   │ Owns the incident. Makes          │
  │ (IC)                 │ decisions. Delegates tasks.       │
  │                      │ Runs the war room.                │
  │                      │                                   │
  │ Communications Lead  │ Updates stakeholders, status      │
  │ (Comms)              │ page, customers. Single source    │
  │                      │ of truth for external comms.      │
  │                      │                                   │
  │ Operations Lead      │ Executes technical investigation  │
  │ (Ops)                │ and mitigation. Hands-on-keyboard.│
  │                      │                                   │
  │ Subject Matter       │ Deep expertise in affected system.│
  │ Expert (SME)         │ Called in by IC as needed.        │
  │                      │                                   │
  │ Scribe               │ Documents timeline, actions,      │
  │                      │ decisions in real-time.           │
  └──────────────────────┴──────────────────────────────────┘
  
  ┌──────────────────────────────────────────────────────────┐
  │                 Incident Commander                       │
  │                    (owns incident)                       │
  │              ┌──────┼──────┐                             │
  │              ▼      ▼      ▼                             │
  │          ┌──────┐ ┌────┐ ┌──────┐                       │
  │          │Comms │ │Ops │ │Scribe│                        │
  │          │Lead  │ │Lead│ │      │                        │
  │          └──────┘ └──┬─┘ └──────┘                       │
  │                      │                                   │
  │                 ┌────┴────┐                               │
  │                 ▼         ▼                               │
  │              ┌─────┐  ┌─────┐                            │
  │              │SME 1│  │SME 2│                             │
  │              └─────┘  └─────┘                            │
  └──────────────────────────────────────────────────────────┘
```

---

## 3. Incident Response Checklist

```
  ┌──────────────────────────────────────────────────────────┐
  │  PHASE 1: DETECTION & TRIAGE (first 5 minutes)          │
  │  □ Alert received and acknowledged                       │
  │  □ Initial assessment: Is this real? What's the impact?  │
  │  □ Assign severity level (SEV-1 through SEV-4)           │
  │  □ Declare incident in #incident channel                 │
  │  □ Assign IC (or self-assign)                            │
  │  □ IC assigns Comms Lead, Ops Lead, Scribe               │
  │                                                          │
  │  PHASE 2: INVESTIGATION (5–30 minutes)                   │
  │  □ Check dashboards and recent deployments               │
  │  □ Review logs and traces for affected services           │
  │  □ Identify blast radius (which users/services affected) │
  │  □ Form hypothesis about root cause                      │
  │  □ Comms: Post initial status update                     │
  │                                                          │
  │  PHASE 3: MITIGATION (parallel with investigation)       │
  │  □ Apply immediate mitigation (rollback, restart, scale) │
  │  □ Verify mitigation is working (metrics recovering)     │
  │  □ Comms: Update status page with ETA                    │
  │                                                          │
  │  PHASE 4: RESOLUTION                                     │
  │  □ Confirm service is fully restored                     │
  │  □ Monitor for recurrence (30-60 minutes)                │
  │  □ Update status page: "Resolved"                        │
  │  □ Notify stakeholders                                   │
  │                                                          │
  │  PHASE 5: POST-INCIDENT                                  │
  │  □ Schedule postmortem (within 48 hours for SEV-1/2)     │
  │  □ Write timeline from scribe notes                      │
  │  □ Identify action items                                 │
  │  □ Share learnings with broader team                     │
  └──────────────────────────────────────────────────────────┘
```

---

## 4. Communication Templates

### Incident Declaration
```
@here INCIDENT DECLARED

Title: [Brief description]
Severity: [SEV-1/2/3/4]
Impact: [What users experience]
Affected services: [List]
IC: @[name]
Comms: @[name]
Ops: @[name]
War room: [link to video call]

Status: Investigating
Next update: [time, 15-30 min]
```

### Status Update
```
INCIDENT UPDATE - [Title]
Time: [HH:MM UTC]
Severity: [Current]
Status: Investigating / Mitigating / Monitoring

What we know:
- [Finding 1]
- [Finding 2]

What we're doing:
- [Action 1]
- [Action 2]

User impact: [Current status]
Next update: [time]
```

### Resolution
```
INCIDENT RESOLVED - [Title]
Duration: [Start - End, total time]
Impact: [Summary of what users experienced]
Root cause: [Brief description]
Mitigation: [What fixed it]
Next steps: Postmortem scheduled for [date]
```

---

## 5. Mitigation vs Resolution

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  ALWAYS MITIGATE FIRST, INVESTIGATE SECOND               │
  │                                                          │
  │  MITIGATION (stop the bleeding)    RESOLUTION (cure)     │
  │  ──────────────────────────        ────────────────      │
  │  ├─ Rollback deploy               ├─ Fix the bug        │
  │  ├─ Scale up capacity              ├─ Redesign component │
  │  ├─ Restart hung processes         ├─ Add redundancy     │
  │  ├─ Enable feature flag fallback   ├─ Update runbook     │
  │  ├─ Redirect traffic               ├─ Improve monitoring │
  │  ├─ Block bad traffic (WAF)        ├─ Load testing      │
  │  └─ Failover to DR                 └─ Architecture review│
  │                                                          │
  │  Time:  Minutes                    Time: Days/Weeks      │
  │  Goal:  Restore service            Goal: Prevent recur.  │
  │                                                          │
  │  [!] A common mistake: spending 30 min debugging while   │
  │      users are impacted. Rollback FIRST, debug LATER.    │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Escalation Process

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  Time  ──▶  0min    15min    30min    60min    2hrs      │
  │                                                          │
  │  SEV-1     On-call → Backup  → Manager → Director → VP  │
  │            + IC     on-call   + all SREs  notified       │
  │                                                          │
  │  SEV-2     On-call ──────────▶ Backup  → Manager         │
  │                                on-call                   │
  │                                                          │
  │  SEV-3     On-call ──────────────────────▶ Backup        │
  │                                            (4h)          │
  │                                                          │
  │  Auto-escalation triggers:                               │
  │  ├─ No acknowledgment within response SLA                │
  │  ├─ Impact expanding (more users affected)               │
  │  ├─ Duration exceeding expected MTTR                     │
  │  └─ IC requests additional help                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] SEV-1 Incident: API Gateway Complete Outage

```
  14:02 UTC - PagerDuty alert: API gateway error rate > 50%
  14:03 UTC - On-call acknowledges, checks dashboard
  14:04 UTC - Error rate at 100%. DECLARES SEV-1.
              IC: @alice  Comms: @bob  Ops: @charlie
  14:05 UTC - @bob posts status: "Investigating API issues"
  14:06 UTC - @charlie checks: deployment at 13:55 (new config)
  14:08 UTC - DECISION: Roll back config change
  14:10 UTC - Rollback applied. Error rate dropping.
  14:12 UTC - Error rate at 2% (normal). Monitoring.
  14:15 UTC - @bob updates status page: "Mitigated"
  14:30 UTC - 15 minutes stable. INCIDENT RESOLVED.
  
  Summary:
  ├─ MTTD: 2 minutes (alert to acknowledge)
  ├─ MTTA: 1 minute (acknowledge to IC assigned)
  ├─ MTTM: 8 minutes (IC assigned to mitigation applied)
  ├─ Total MTTR: 28 minutes
  ├─ Root cause: Invalid regex in rate-limiting config
  └─ Action items:
      ├─ Config validation in CI pipeline
      ├─ Canary config changes (roll to 5% → 25% → 100%)
      └─ Add config syntax check to pre-deployment
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Too many people in the war room" | IC controls attendance. Only IC, Ops, Comms, Scribe, and requested SMEs. |
| "No one takes the IC role" | Rotate IC duty on a schedule. Train all senior engineers. Make it a leadership skill. |
| "We spend too long debugging before mitigating" | Rule: If root cause not found in 5 min, mitigate first (rollback/restart). |
| "Communication is chaotic during incidents" | Only Comms Lead posts updates. Use structured templates. All discussion in one channel. |
| "Incidents drag on for hours" | Set time-box milestones. If not mitigated in 30 min, escalate. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **MTTD** | Mean Time to Detect — alert fires |
| **MTTA** | Mean Time to Acknowledge — human responds |
| **MTTM** | Mean Time to Mitigate — bleeding stops |
| **MTTR** | Mean Time to Recovery — full restoration |
| **IC** | Incident Commander — owns the incident |
| **Comms Lead** | Manages stakeholder communication |
| **Mitigation** | Quick fix (minutes): rollback, restart, scale |
| **Resolution** | Permanent fix (days): code fix, redesign |

---

## Quick Revision Questions

1. **Name the five phases of incident response.**
   > Detect, Triage, Mitigate, Resolve, Learn. The key insight is that mitigation (restoring service) should happen before resolution (fixing root cause).

2. **What are the four key incident roles?**
   > Incident Commander (IC), Communications Lead, Operations Lead, and Scribe. SMEs are called in as needed.

3. **Why should you mitigate before investigating root cause?**
   > Users are impacted during investigation. A quick rollback or restart can restore service in minutes, while debugging may take hours. Mitigate first to stop the bleeding.

4. **What triggers auto-escalation?**
   > No acknowledgment within SLA, expanding impact, duration exceeding expected MTTR, or IC requests additional help.

5. **What information should the initial incident declaration include?**
   > Title, severity, user impact, affected services, assigned roles (IC/Comms/Ops), war room link, current status, and next update time.

6. **How does MTTD differ from MTTR?**
   > MTTD is the time from failure occurrence to detection (alert fires). MTTR is the total time from detection to full service restoration. MTTR = MTTD + MTTA + diagnosis + mitigation time.

---

[← Previous: Incident Classification](01-incident-classification.md) | [Back to README](../README.md) | [Next: On-Call Management →](03-on-call-management.md)
