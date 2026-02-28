# Chapter 6: Incident Management & Postmortems

[← Previous: Incident Response](05-incident-response.md) | [Back to README](../README.md) | [Next: Unit 9 - APM Concepts →](../09-APM/01-apm-concepts.md)

---

## Overview

Incident management extends beyond the immediate response — it includes postmortems (post-incident reviews), tracking action items, measuring reliability improvements, and building a learning culture. This chapter covers blameless postmortems, incident tracking, and continuous improvement.

---

## 6.1 Blameless Postmortems

```
┌──────────────────────────────────────────────────────────┐
│            BLAMELESS POSTMORTEM CULTURE                   │
│                                                          │
│  ✓ BLAMELESS (Focus on systems):                        │
│  "The deploy process allowed an untested config         │
│   to reach production without validation."              │
│                                                          │
│  ✗ BLAMEFUL (Focus on people):                          │
│  "Bob deployed a bad config without testing it."        │
│                                                          │
│  WHY BLAMELESS?                                         │
│  • People won't report issues if they fear punishment   │
│  • Systems should prevent human errors                  │
│  • The goal is to fix the PROCESS, not blame people     │
│  • Same person in same system = same outcome            │
│                                                          │
│  Core principle: "How did the system allow this to      │
│  happen, and how do we change the system?"              │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 Postmortem Template

```markdown
# Incident Postmortem: [Title]

## Metadata
- **Date**: 2024-01-15
- **Duration**: 55 minutes (14:05 - 15:00 UTC)
- **Severity**: SEV2
- **Incident Commander**: Alice
- **Author**: Bob
- **Status**: Complete

## Executive Summary
On January 15, the API experienced elevated error rates (15% of 
requests returning 500) for 55 minutes. The root cause was database 
connection pool exhaustion triggered by a traffic spike combined 
with a connection leak in the order-service v2.3.1 release.

## Impact
- **Duration**: 55 minutes
- **Users Affected**: ~12,000 users
- **Revenue Impact**: ~$15,000 in failed transactions
- **SLO Impact**: Consumed 40% of monthly error budget

## Timeline (UTC)
| Time  | Event |
|-------|-------|
| 14:00 | Traffic spike begins (Black Friday sale) |
| 14:05 | Alert fires: APIHighErrorRate |
| 14:07 | On-call (Alice) acknowledges |
| 14:10 | SEV2 declared, war room opened |
| 14:12 | Checked dashboards — DB connection pool at 100% |
| 14:15 | Root cause identified: connection leak in order-service |
| 14:20 | Mitigation: increased connection pool from 50 to 200 |
| 14:25 | Error rate dropping but still elevated |
| 14:30 | Rolled back order-service to v2.3.0 |
| 14:35 | Error rate < 1% |
| 14:45 | Error rate < 0.1%, monitoring |
| 15:00 | Incident resolved |

## Root Cause
The order-service v2.3.1 (deployed Jan 14) introduced a bug where 
database connections were not returned to the pool on timeout errors. 
Under normal load, connections leaked slowly (1-2 per minute). 
During the traffic spike, connection exhaustion accelerated, causing 
cascading failures.

## Contributing Factors
1. Missing connection pool metrics in order-service dashboard
2. No integration test for connection cleanup on timeout
3. Connection leak was not caught in staging (lower traffic)
4. No circuit breaker between API gateway and order-service

## Resolution
1. Rolled back order-service to v2.3.0
2. Increased connection pool temporarily
3. Fixed connection leak bug in v2.3.2

## Action Items
| ID | Action | Owner | Priority | Due Date | Status |
|----|--------|-------|----------|----------|--------|
| 1 | Fix connection leak bug | Bob | P1 | Jan 16 | Done |
| 2 | Add DB connection pool metric to dashboard | Carol | P2 | Jan 22 | Done |
| 3 | Add integration test for connection cleanup | Bob | P2 | Jan 29 | In Progress |
| 4 | Implement circuit breaker for order-service | Dave | P3 | Feb 15 | Not Started |
| 5 | Add load test to staging CI pipeline | Eve | P3 | Feb 28 | Not Started |
| 6 | Alert on connection pool > 80% | Carol | P1 | Jan 18 | Done |

## Lessons Learned
- **What went well**: Fast detection (5 min), clear incident process
- **What could improve**: Need connection pool monitoring, need load testing
- **Where we got lucky**: Traffic spike happened during business hours
```

---

## 6.3 Five Whys Analysis

```
┌──────────────────────────────────────────────────────────┐
│  FIVE WHYS - ROOT CAUSE ANALYSIS                         │
│                                                          │
│  Problem: Users saw 500 errors on checkout               │
│                                                          │
│  Why 1: API gateway returned 500 to users               │
│    └─ Why 2: Order-service was returning errors          │
│       └─ Why 3: Database connections were exhausted      │
│          └─ Why 4: Connections were not being returned   │
│             └─ Why 5: The connection cleanup code had    │
│                       a bug that skipped cleanup on      │
│                       timeout exceptions                 │
│                                                          │
│  ROOT CAUSE: Missing error handling for timeout          │
│  exceptions in database connection cleanup path.         │
│                                                          │
│  SYSTEMIC CAUSE: No integration test covering            │
│  connection cleanup under error conditions.              │
│                                                          │
│  Keep asking "why" until you reach a systemic cause     │
│  you can fix with process/tooling changes.              │
└──────────────────────────────────────────────────────────┘
```

---

## 6.4 Incident Tracking Metrics

```
┌──────────────────────────────────────────────────────────┐
│  INCIDENT TRACKING DASHBOARD                             │
│                                                          │
│  Monthly Incident Summary                                │
│  ┌────────────────────────────────────────────┐          │
│  │ SEV1: ██ 2            (target: 0)          │          │
│  │ SEV2: █████ 5         (target: < 3)        │          │
│  │ SEV3: ████████ 8      (target: < 10)       │          │
│  │ SEV4: ████████████ 12 (target: < 20)       │          │
│  └────────────────────────────────────────────┘          │
│                                                          │
│  Key Metrics:                                            │
│  • MTTD (Mean Time to Detect):    5 min                 │
│  • MTTA (Mean Time to Acknowledge): 3 min               │
│  • MTTR (Mean Time to Resolve):   45 min                │
│  • MTBF (Mean Time Between Failures): 8 days            │
│  • Change Failure Rate:           12%                    │
│  • Postmortem Completion Rate:    90%                    │
│  • Action Item Completion Rate:   75%                    │
│                                                          │
│  Trends (Month over Month):                             │
│  MTTR: 60min → 45min  ↓ improving                      │
│  SEV1: 3 → 2          ↓ improving                      │
│  Action completion: 60% → 75%  ↑ improving             │
└──────────────────────────────────────────────────────────┘
```

### DORA Metrics Connection

| DORA Metric | Relates To | Measured By |
|-------------|-----------|-------------|
| Deployment Frequency | How often we deploy | CI/CD pipeline data |
| Lead Time for Changes | Code-to-production time | Git + deployment data |
| Change Failure Rate | % of deploys causing incidents | Incidents / deployments |
| Mean Time to Restore | How fast we recover | Incident MTTR |

---

## 6.5 Post-Incident Review Process

```
┌──────────────────────────────────────────────────────────┐
│  POST-INCIDENT REVIEW TIMELINE                           │
│                                                          │
│  Day 0: Incident occurs and is resolved                 │
│         └─ IC creates postmortem doc (draft)             │
│                                                          │
│  Day 1-2: Author fills in timeline, root cause          │
│           └─ Review with involved team members           │
│                                                          │
│  Day 3-5: Postmortem meeting (30-60 min)                │
│           └─ Walk through timeline                       │
│           └─ Discuss root cause and contributing factors │
│           └─ Identify action items with owners           │
│           └─ Assign priorities and due dates             │
│                                                          │
│  Day 5+: Publish postmortem to team/org                 │
│          └─ Track action items in Jira/GitHub            │
│                                                          │
│  Week 4: Review action item progress                    │
│          └─ Ensure systemic fixes are implemented        │
│                                                          │
│  Quarterly: Incident trends review                      │
│             └─ Are we getting better?                    │
│             └─ Common failure patterns                   │
└──────────────────────────────────────────────────────────┘
```

---

## 6.6 Learning from Incidents

### Incident Categories

| Category | Example | Systemic Fix |
|----------|---------|-------------|
| Deployment | Bad config pushed | Canary deployments, config validation |
| Capacity | Traffic spike overwhelms | Auto-scaling, load testing |
| Dependency | External API down | Circuit breakers, fallbacks |
| Data | Corrupt data causing crashes | Input validation, schema checks |
| Infrastructure | Node failure | Multi-AZ, redundancy |
| Security | DDoS attack | WAF, rate limiting |
| Human Error | Wrong command in prod | Automation, access controls |

### Building a Learning Culture

```
┌──────────────────────────────────────────────┐
│  INCIDENT LEARNING CYCLE                     │
│                                              │
│       ┌──────────┐                          │
│       │ Incident │                          │
│       │ Occurs   │                          │
│       └────┬─────┘                          │
│            ▼                                 │
│       ┌──────────┐                          │
│       │Postmortem│                          │
│       │ Review   │                          │
│       └────┬─────┘                          │
│            ▼                                 │
│       ┌──────────┐                          │
│       │ Action   │                          │
│       │ Items    │                          │
│       └────┬─────┘                          │
│            ▼                                 │
│       ┌──────────┐                          │
│       │ System   │                          │
│       │ Improved │                          │
│       └────┬─────┘                          │
│            ▼                                 │
│       ┌──────────┐                          │
│       │ Future   │──── Fewer / smaller     │
│       │ Incident │     incidents over time  │
│       └──────────┘                          │
└──────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Postmortems not written | No accountability | Mandate within 48 hours, assign owner |
| Action items not completed | No tracking | Track in Jira, review weekly |
| Same incidents repeat | Root cause not addressed | Focus on systemic fixes, not Band-Aids |
| Blame culture | Fear of consequences | Leadership must model blamelessness |
| Postmortems too shallow | Rushing through | Use Five Whys, require contributing factors |

---

## Summary Table

| Concept | Details |
|---------|---------|
| Blameless Postmortem | Focus on systems, not people |
| Five Whys | Iterative root cause analysis |
| Key Metrics | MTTD, MTTA, MTTR, MTBF, Change Failure Rate |
| DORA Connection | Deployment frequency, lead time, CFR, MTTR |
| Action Items | Owned, prioritized, tracked, reviewed |
| Learning Culture | Incidents → Postmortems → Improvements → Fewer incidents |

---

## Quick Revision Questions

1. **What is a blameless postmortem and why is it important?**
2. **What sections should a postmortem document include?**
3. **How does the Five Whys technique help find root causes?**
4. **What are the DORA metrics and how do they relate to incident management?**
5. **How do you ensure action items from postmortems are actually completed?**
6. **What is the difference between root cause and contributing factors?**

---

[← Previous: Incident Response](05-incident-response.md) | [Back to README](../README.md) | [Next: Unit 9 - APM Concepts →](../09-APM/01-apm-concepts.md)
