# Chapter 4.4: Postmortems & Root Cause Analysis

[← Previous: On-Call Management](03-on-call-management.md) | [Back to README](../README.md) | [Next: Blameless Culture →](05-blameless-culture.md)

---

## Overview

A postmortem is a structured analysis conducted after a significant incident to understand what happened, why it happened, and how to prevent recurrence. Root Cause Analysis (RCA) is the investigative technique used within postmortems. Together, they transform incidents from painful events into organizational learning opportunities.

---

## 1. When to Write a Postmortem

```
  ┌──────────────────────────────────────────────────────────┐
  │  POSTMORTEM TRIGGERS                                     │
  │                                                          │
  │  Always (mandatory):                                     │
  │  ├─ SEV-1 or SEV-2 incident                              │
  │  ├─ Data loss of any kind                                │
  │  ├─ Security breach                                      │
  │  ├─ Error budget exhausted                               │
  │  ├─ Customer-visible outage > 30 minutes                 │
  │  └─ Incident requiring on-call escalation                │
  │                                                          │
  │  Optional (team decides):                                │
  │  ├─ Interesting near-miss                                │
  │  ├─ Novel failure mode discovered                        │
  │  ├─ Process or tooling gap revealed                      │
  │  └─ Request from anyone on the team                      │
  │                                                          │
  │  Timeline:                                               │
  │  ├─ Draft within 24-48 hours while memory is fresh       │
  │  ├─ Review meeting within 1 week                         │
  │  └─ Action items assigned with owners and deadlines      │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Postmortem Template

```markdown
# Postmortem: [Incident Title]

## Summary
**Date:** YYYY-MM-DD
**Duration:** HH:MM (start) — HH:MM (end), total X hours Y minutes
**Severity:** SEV-X
**Impact:** [Who was affected and how]
**Authors:** [Name(s)]
**Reviewers:** [Name(s)]

## Status
| Metric | Value |
|--------|-------|
| Detected after | X minutes |
| Users affected | X% |
| Revenue impact | $X estimated |
| Error budget consumed | X% |
| Action items | X total, Y completed |

## Timeline (all times UTC)
| Time | Event |
|------|-------|
| 14:00 | Deployment of auth-service v2.3 begins |
| 14:05 | Canary shows elevated error rate |
| 14:15 | Full rollout completes |
| 14:18 | Alert: error rate > 5% |
| 14:20 | On-call acknowledges, begins investigation |
| 14:25 | IC declared, SEV-2 incident |
| 14:30 | Root cause identified: config regression |
| 14:32 | Rollback initiated |
| 14:35 | Error rate returning to baseline |
| 14:50 | Incident resolved |

## Root Cause
[Detailed technical explanation]

## Contributing Factors
- [Factor 1]
- [Factor 2]

## What Went Well
- [Positive aspect 1]
- [Positive aspect 2]

## What Went Poorly
- [Issue 1]
- [Issue 2]

## Where We Got Lucky
- [Lucky break 1]

## Action Items
| ID | Action | Owner | Priority | Due Date | Status |
|----|--------|-------|----------|----------|--------|
| 1  | [Action] | [Name] | P1 | YYYY-MM-DD | Open |
| 2  | [Action] | [Name] | P2 | YYYY-MM-DD | Open |

## Lessons Learned
[Key takeaways for the organization]
```

---

## 3. Root Cause Analysis Techniques

```
  ┌──────────────────────────────────────────────────────────┐
  │  TECHNIQUE          │ HOW IT WORKS                       │
  ├─────────────────────┼────────────────────────────────────┤
  │ 5 Whys              │ Ask "why?" repeatedly until you    │
  │                     │ reach the systemic root cause      │
  │                     │                                    │
  │ Fishbone / Ishikawa │ Categorize causes into People,     │
  │                     │ Process, Technology, Environment   │
  │                     │                                    │
  │ Fault Tree Analysis │ Boolean logic tree showing how     │
  │                     │ failures combine (AND/OR gates)    │
  │                     │                                    │
  │ Timeline Analysis   │ Map events chronologically to      │
  │                     │ find cause-effect chains           │
  │                     │                                    │
  │ Change Analysis     │ What changed recently? Compare     │
  │                     │ working vs broken state            │
  └─────────────────────┴────────────────────────────────────┘
```

---

## 4. The 5 Whys Technique

```
  ┌──────────────────────────────────────────────────────────┐
  │  EXAMPLE: Website returned 500 errors for 30 minutes     │
  │                                                          │
  │  Why 1: Why did the website return 500 errors?           │
  │  → The API server ran out of memory and crashed.         │
  │                                                          │
  │  Why 2: Why did it run out of memory?                    │
  │  → A memory leak in the new caching module.              │
  │                                                          │
  │  Why 3: Why wasn't the memory leak caught before prod?   │
  │  → Load testing doesn't run long enough to reveal leaks. │
  │                                                          │
  │  Why 4: Why doesn't load testing run longer?             │
  │  → Budget constraints limited CI/CD runtime to 10 min.  │
  │                                                          │
  │  Why 5: Why haven't we invested in better load testing?  │
  │  → No process to review test adequacy after incidents.   │
  │                                                          │
  │  ROOT CAUSE: No feedback loop from incidents to testing  │
  │  strategy improvements.                                  │
  │                                                          │
  │  [!] Common mistake: Stopping at "human error" (Why 1-2)│
  │      Keep asking until you reach a SYSTEMIC cause.       │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Fishbone Diagram

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  People          Process           Technology            │
  │    │                │                  │                  │
  │    ├─ Fatigue       ├─ No canary       ├─ No auto-scale  │
  │    ├─ New hire      ├─ Missing review  ├─ Single DB      │
  │    │                ├─ No runbook      ├─ No circuit brk │
  │    │                │                  │                  │
  │    └────────────────┼──────────────────┘                  │
  │                     │                                     │
  │                     ▼                                     │
  │            ┌─────────────────┐                            │
  │            │   30-min outage │                            │
  │            └─────────────────┘                            │
  │                     ▲                                     │
  │    ┌────────────────┼──────────────────┐                  │
  │    │                │                  │                  │
  │    ├─ Traffic spike ├─ Bad deploy      ├─ Alert noise     │
  │    ├─ Region issue  │  at peak hours   ├─ Missing SLO    │
  │    │                │                  │                  │
  │  Environment      Change            Monitoring           │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Postmortem Review Meeting

```
  ┌──────────────────────────────────────────────────────────┐
  │  POSTMORTEM REVIEW MEETING AGENDA (1 hour)               │
  │                                                          │
  │  00:00–05:00  Introduction & ground rules                │
  │               ├─ "This is blameless"                     │
  │               └─ Focus on systems, not people            │
  │                                                          │
  │  05:00–15:00  Timeline walkthrough                       │
  │               ├─ What happened, in order                 │
  │               └─ Fill in gaps from attendees             │
  │                                                          │
  │  15:00–25:00  Root cause discussion                      │
  │               ├─ Walk through 5 Whys                     │
  │               └─ Identify contributing factors           │
  │                                                          │
  │  25:00–35:00  What went well / poorly / lucky            │
  │               ├─ Celebrate what worked                   │
  │               └─ Honestly assess gaps                    │
  │                                                          │
  │  35:00–50:00  Action items                               │
  │               ├─ Brainstorm preventive actions           │
  │               ├─ Assign owners and priority              │
  │               └─ Set realistic due dates                 │
  │                                                          │
  │  50:00–60:00  Summary & distribution                     │
  │               ├─ Agree on final document                 │
  │               └─ Plan for sharing learnings              │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Action Item Best Practices

```
  ┌──────────────────────────────────────────────────────────┐
  │  GOOD vs BAD Action Items                                │
  │                                                          │
  │  BAD:                                                    │
  │  ✗ "Improve monitoring"         (too vague)              │
  │  ✗ "Be more careful"            (blames humans)          │
  │  ✗ "Fix the bug"                (already done)           │
  │  ✗ "Rewrite the service"        (too large)              │
  │                                                          │
  │  GOOD:                                                    │
  │  ✓ "Add alert for DB connection pool > 80%"              │
  │    Owner: @alice  | Priority: P1 | Due: Jan 31           │
  │                                                          │
  │  ✓ "Implement circuit breaker for payment gateway        │
  │    with 5s timeout and 50% error threshold"              │
  │    Owner: @bob   | Priority: P1 | Due: Feb 15            │
  │                                                          │
  │  ✓ "Add config validation to CI pipeline to              │
  │    reject invalid regex patterns"                        │
  │    Owner: @charlie | Priority: P2 | Due: Feb 28          │
  │                                                          │
  │  Properties of a good action item:                       │
  │  ├─ Specific and measurable                              │
  │  ├─ Has a single owner                                   │
  │  ├─ Has a priority (P1/P2/P3)                            │
  │  ├─ Has a due date                                       │
  │  └─ Tracked in issue tracker (not lost in a doc)         │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Tracking Postmortem Effectiveness

```
  ┌──────────────────────────────────────────────────────────┐
  │  METRICS TO TRACK                                        │
  │                                                          │
  │  Metric                    │ Target   │ Signal           │
  │  ──────────────────────────┼──────────┼──────────────────│
  │  Postmortem completion rate│ 100%     │ Are we learning? │
  │  (SEV-1 + SEV-2)          │          │                  │
  │                            │          │                  │
  │  Action item completion    │ > 80%    │ Are we fixing?   │
  │  within deadline           │          │                  │
  │                            │          │                  │
  │  Repeat incidents          │ < 10%    │ Are fixes working│
  │  (same root cause)         │          │                  │
  │                            │          │                  │
  │  Time to postmortem draft  │ < 48 hrs │ Are we timely?   │
  │                            │          │                  │
  │  Avg action items per      │ 3-5      │ Too few = under- │
  │  postmortem                │          │ investing; too   │
  │                            │          │ many = unfocused │
  └──────────────────────────────────────────────────────────┘
```

---

## 9. Real-World Scenario

### [SCENARIO] Postmortem for a Payment Processing Outage

```
  INCIDENT: Payment processing failed for 45 minutes
  
  5 Whys:
  1. Why did payments fail?
     → Payment service returned 503 for all requests.
  
  2. Why did it return 503?
     → Connection pool to payment DB was exhausted.
  
  3. Why was the connection pool exhausted?
     → A long-running analytics query held 50 connections.
  
  4. Why could an analytics query consume production connections?
     → Analytics and production share the same DB instance.
  
  5. Why do they share the same instance?
     → Never separated during initial architecture. No ticket
        was created during previous capacity review.
  
  ROOT CAUSE: Shared database between production and analytics
  workloads without connection isolation.
  
  Action Items:
  ┌─────────────────────────────────────────────────────────┐
  │ # │ Action                      │ Owner │ Due    │ Pri  │
  ├───┼─────────────────────────────┼───────┼────────┼──────┤
  │ 1 │ Set per-user connection     │ Alice │ Jan 20 │ P1   │
  │   │ limits on production DB     │       │        │      │
  │ 2 │ Migrate analytics to read   │ Bob   │ Feb 15 │ P1   │
  │   │ replica                     │       │        │      │
  │ 3 │ Add connection pool usage   │ Alice │ Jan 25 │ P1   │
  │   │ alert (threshold: 80%)     │       │        │      │
  │ 4 │ Document DB connection      │ Carol │ Feb 01 │ P2   │
  │   │ architecture in runbook     │       │        │      │
  └───┴─────────────────────────────┴───────┴────────┴──────┘
```

---

## 10. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Nobody writes postmortems" | Make it a policy: no incident closure without postmortem for SEV-1/2. Track completion rate. |
| "Postmortems become blame sessions" | Set ground rules explicitly. Facilitator redirects from "who" to "what system allowed this." |
| "Action items never get done" | Track in issue tracker with sprint planning. Report completion rate monthly. |
| "Same incidents keep recurring" | Check action item completion. Recurring incidents = action items not implemented or wrong root cause. |
| "Postmortems are too long/detailed" | Focus on timeline, root cause, and action items. 2-3 pages is ideal. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Postmortem** | Structured analysis after a significant incident |
| **5 Whys** | Repeated "why?" to reach systemic root cause |
| **Fishbone** | Categorizes causes by People/Process/Tech/Environment |
| **Blameless** | Focus on systems, not individuals |
| **Action Items** | Specific, owned, prioritized, time-bound fixes |
| **RCA** | Root Cause Analysis — the investigative technique |
| **Contributing Factors** | Conditions that enabled the failure (not root cause) |

---

## Quick Revision Questions

1. **When should a postmortem be written?**
   > Mandatory for SEV-1/2, data loss, security breach, error budget exhaustion, or outages > 30 minutes. Draft within 24-48 hours.

2. **What are the key sections of a postmortem document?**
   > Summary, timeline, root cause, contributing factors, what went well/poorly/lucky, action items with owners and deadlines, and lessons learned.

3. **Demonstrate the 5 Whys technique with an example.**
   > Website down → Server crashed (why?) → Out of memory (why?) → Memory leak (why?) → Not caught in testing (why?) → Load tests too short (why?) → No process to review test adequacy. Root cause: missing feedback loop.

4. **What makes a good vs bad action item?**
   > Good: specific, measurable, single owner, prioritized, has due date. Bad: vague ("improve monitoring"), blaming ("be more careful"), too large ("rewrite the service").

5. **How do you measure postmortem effectiveness?**
   > Track: postmortem completion rate (target 100%), action item completion within deadline (>80%), repeat incident rate (<10%), time to draft (<48 hours).

---

[← Previous: On-Call Management](03-on-call-management.md) | [Back to README](../README.md) | [Next: Blameless Culture →](05-blameless-culture.md)
