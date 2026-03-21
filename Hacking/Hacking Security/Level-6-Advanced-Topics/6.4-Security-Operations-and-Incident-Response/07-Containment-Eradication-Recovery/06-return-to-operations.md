# Unit 7: Containment, Eradication, Recovery — Topic 6: Return to Operations

## Overview

**Return to operations** is the final step in the recovery phase, transitioning systems from incident response back to normal business operations. This phase requires careful coordination, staged deployment, enhanced monitoring, and clear communication to ensure a smooth and secure transition.

---

## 1. Return to Operations Process

```
RETURN TO OPERATIONS WORKFLOW:

  ┌────────────────────────────────────────────┐
  │       RETURN TO OPERATIONS                 │
  │                                            │
  │  1. VALIDATION COMPLETE                    │
  │     All systems passed testing             │
  │     Sign-offs obtained                     │
  │                                            │
  │  2. STAGED DEPLOYMENT                      │
  │     Phase 1: Infrastructure (AD, DNS)      │
  │     Phase 2: Critical applications         │
  │     Phase 3: Standard services             │
  │     Phase 4: End-user systems              │
  │                                            │
  │  3. ENHANCED MONITORING                    │
  │     72-hour intensive watch                │
  │     30-day enhanced detection              │
  │                                            │
  │  4. USER COMMUNICATION                     │
  │     System availability notices            │
  │     Password reset instructions            │
  │     Changed process guidance               │
  │                                            │
  │  5. INCIDENT CLOSURE CRITERIA              │
  │     No re-compromise detected              │
  │     All systems stable                     │
  │     Monitoring normalized                  │
  └────────────────────────────────────────────┘
```

---

## 2. Staged Deployment

```
STAGED RETURN PLAN:

PHASE 1 — INFRASTRUCTURE (Day 1):
  → Domain controllers online
  → DNS/DHCP services restored
  → Core network services
  → Authentication working
  → Verify: Users can log in to domain

PHASE 2 — CRITICAL APPS (Day 2-3):
  → Email servers
  → Critical business applications
  → Database servers
  → File servers
  → Verify: Core business functions operational

PHASE 3 — STANDARD SERVICES (Day 3-5):
  → Web servers
  → Secondary applications
  → Print services
  → Development environments
  → Verify: Full service catalog available

PHASE 4 — END USERS (Day 5-7):
  → Reimaged workstations deployed
  → User data restored
  → User access provisioned
  → Verify: Users productive

STAGE GATE CRITERIA:
  Each phase must pass before proceeding:
  [ ] Security validation complete
  [ ] Functional testing passed
  [ ] No incidents detected during phase
  [ ] Monitoring confirmed active
  [ ] Stakeholder approval for next phase
```

---

## 3. Enhanced Monitoring

```
POST-RECOVERY MONITORING PLAN:

72-HOUR INTENSIVE MONITORING:
  → 24/7 SOC analyst coverage
  → Watch for re-compromise indicators
  → Monitor all previously compromised systems
  → Alert on any IOC matches
  → Track all C2-related activity
  → Monitor all privileged access

  Monitoring Focus:
  → Known attacker IOCs
  → Known attacker TTPs
  → C2 communication patterns
  → Credential abuse
  → Lateral movement
  → Data exfiltration
  → Persistence re-establishment
  → New user accounts
  → Privilege escalation

30-DAY ENHANCED DETECTION:
  → New detection rules for incident TTPs
  → Lower alert thresholds
  → Additional log sources enabled
  → Weekly IOC re-sweep
  → Regular threat hunting sessions

  # Custom alert: Re-compromise detection
  index=sysmon ComputerName IN 
    ("JSMITH-PC","FILESVR01","DC01")
    (EventCode=1 OR EventCode=3 OR EventCode=11)
  | where match(Image, "(?i)(mimikatz|psexec|
    cobalt|beacon)")
  | alert

90-DAY MONITORING CHECKLIST:
  Week 1-2:  Daily IOC sweeps
  Week 3-4:  Bi-weekly IOC sweeps + hunt
  Month 2:   Weekly sweeps + hunt
  Month 3:   Bi-weekly sweeps
  Month 3+:  Return to standard monitoring
```

---

## 4. Communication and Handoff

```
STAKEHOLDER COMMUNICATION:

USER NOTIFICATION:
  ┌──────────────────────────────────────────────┐
  │ TO: All Staff                                │
  │ FROM: IT Department                          │
  │ SUBJECT: Systems Restored — Action Required  │
  │                                              │
  │ Systems have been restored following the      │
  │ recent security incident. Please note:       │
  │                                              │
  │ IMMEDIATE ACTIONS REQUIRED:                  │
  │ 1. Reset your password at next login         │
  │ 2. Set up MFA if prompted                    │
  │ 3. Report any unusual system behavior        │
  │                                              │
  │ CHANGES YOU MAY NOTICE:                      │
  │ → New MFA requirement for VPN               │
  │ → Updated email security warnings           │
  │ → Password complexity requirements           │
  │ → Some data restored from [date] backup     │
  │                                              │
  │ SUPPORT:                                     │
  │ Contact Help Desk for any issues             │
  └──────────────────────────────────────────────┘

IR TO OPERATIONS HANDOFF:
  → Transfer monitoring responsibilities
  → Provide IOC watch list
  → Document new detection rules
  → Share enhanced monitoring plan
  → Define escalation criteria
  → Establish check-in schedule
  → Provide incident summary for ops team

MANAGEMENT BRIEFING:
  → Systems restored and validated
  → Timeline to full operations
  → Security improvements implemented
  → Ongoing monitoring plan
  → Remaining action items
  → Lessons learned (high-level)
```

---

## 5. Incident Closure

```
INCIDENT CLOSURE CRITERIA:

  [ ] All systems recovered and validated
  [ ] No re-compromise detected (30+ days)
  [ ] All containment measures lifted or
      converted to permanent controls
  [ ] Corrective actions initiated
  [ ] Lessons learned meeting completed
  [ ] Incident report finalized
  [ ] Evidence properly archived
  [ ] Costs documented
  [ ] Regulatory notifications completed
  [ ] Insurance claims filed (if applicable)

CLOSURE DOCUMENTATION:
  → Final incident report
  → Timeline of incident and response
  → Scope summary
  → Impact assessment
  → Root cause analysis
  → Corrective action plan with status
  → Costs and resource usage
  → Lessons learned document

TRANSITIONING TO NORMAL:
  → Remove temporary firewall rules
  → Remove temporary monitoring rules
  → Return to standard SOC procedures
  → Close incident ticket
  → Archive evidence per retention policy
  → Update runbooks with lessons learned
  → Schedule follow-up on corrective actions
```

---

## Summary Table

| Phase | Activities | Duration | Gate |
|-------|-----------|----------|------|
| Stage 1 | Infrastructure | Day 1 | No incidents |
| Stage 2 | Critical apps | Day 2-3 | Functional tests |
| Stage 3 | Standard services | Day 3-5 | All clean |
| Stage 4 | End users | Day 5-7 | User acceptance |
| Monitoring | Enhanced watch | 30-90 days | No re-compromise |
| Closure | Documentation, handoff | Post-30 days | All criteria met |

---

## Revision Questions

1. Why should return to operations be staged rather than simultaneous?
2. What should be monitored during the 72-hour intensive period?
3. What information should users receive when systems are restored?
4. What criteria must be met before an incident can be closed?
5. How is monitoring responsibility handed off from IR to operations?

---

*Previous: [05-validation-and-testing.md](05-validation-and-testing.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
