# Chapter 4.1: Incident Classification

[← Previous: Redundancy and Replication](../03-Reliability/06-redundancy-and-replication.md) | [Back to README](../README.md) | [Next: Incident Response Process →](02-incident-response-process.md)

---

## Overview

Incident classification is the systematic categorization of operational events by their severity, scope, and impact. Proper classification ensures that the right people respond with appropriate urgency, communication channels are activated, and organizational expectations are set correctly. Without it, teams either under-respond to critical issues or burn out from treating everything as an emergency.

---

## 1. What is an Incident?

```
  ┌──────────────────────────────────────────────────────────┐
  │  EVENT vs ALERT vs INCIDENT                              │
  │                                                          │
  │  EVENT                                                   │
  │  ├─ Any observable occurrence in the system              │
  │  ├─ Most events are informational / normal               │
  │  │                                                       │
  │  ▼                                                       │
  │  ALERT                                                   │
  │  ├─ An event that crosses a predefined threshold         │
  │  ├─ Requires investigation but may not be an incident    │
  │  │                                                       │
  │  ▼                                                       │
  │  INCIDENT                                                │
  │  ├─ An unplanned disruption or degradation to service    │
  │  ├─ Impacts users or violates SLOs                       │
  │  └─ Requires coordinated response and remediation        │
  │                                                          │
  │  Ratio: 10,000 events → 100 alerts → 5 incidents        │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Severity Levels

```
  ┌──────────────────────────────────────────────────────────────────┐
  │  SEV  │ NAME        │ IMPACT           │ RESPONSE               │
  ├───────┼─────────────┼──────────────────┼────────────────────────┤
  │ SEV-1 │ Critical    │ Complete service  │ All hands on deck.     │
  │       │             │ outage. Revenue   │ Exec notification.     │
  │       │             │ loss. Data loss   │ War room. Comms every  │
  │       │             │ risk. Security    │ 15 min. Postmortem     │
  │       │             │ breach.           │ mandatory.             │
  │       │             │                  │ Response: < 5 min      │
  │       │             │                  │                        │
  │ SEV-2 │ Major       │ Significant      │ On-call team + backup. │
  │       │             │ degradation.     │ Manager notified.      │
  │       │             │ Core features    │ Comms every 30 min.    │
  │       │             │ impacted. SLO    │ Postmortem required.   │
  │       │             │ burning fast.    │ Response: < 15 min     │
  │       │             │                  │                        │
  │ SEV-3 │ Minor       │ Partial impact.  │ On-call engineer.      │
  │       │             │ Workaround       │ Normal work hours OK.  │
  │       │             │ available. Non-  │ Track in ticket.       │
  │       │             │ critical feature.│ Response: < 4 hours    │
  │       │             │                  │                        │
  │ SEV-4 │ Low         │ Cosmetic or      │ Regular backlog.       │
  │       │             │ informational.   │ Fix in normal sprint.  │
  │       │             │ No user impact.  │ Response: best effort  │
  └───────┴─────────────┴──────────────────┴────────────────────────┘
```

---

## 3. Classification Criteria

```
  ┌──────────────────────────────────────────────────────────┐
  │  CLASSIFICATION DECISION TREE                           │
  │                                                          │
  │  Is the service completely down?                         │
  │  ├─ YES ──▶ SEV-1                                       │
  │  └─ NO                                                   │
  │      Is core functionality degraded?                     │
  │      ├─ YES                                              │
  │      │   Are >10% of users affected?                     │
  │      │   ├─ YES ──▶ SEV-2                                │
  │      │   └─ NO  ──▶ SEV-3                                │
  │      └─ NO                                               │
  │          Is there any user-visible impact?               │
  │          ├─ YES ──▶ SEV-3                                │
  │          └─ NO  ──▶ SEV-4                                │
  │                                                          │
  │  Auto-escalation rules:                                  │
  │  ├─ SEV-3 unresolved for 4h → escalate to SEV-2         │
  │  ├─ SEV-2 unresolved for 2h → escalate to SEV-1         │
  │  └─ Any data loss or security concern → SEV-1            │
  └──────────────────────────────────────────────────────────┘
```

---

## 4. Impact Dimensions

```
  ┌──────────────────────────────────────────────────────────┐
  │  DIMENSION        │ QUESTIONS TO ASK                     │
  ├───────────────────┼──────────────────────────────────────┤
  │ User Impact       │ How many users affected?             │
  │                   │ Which user segments?                 │
  │                   │ Can they use a workaround?           │
  │                   │                                      │
  │ Revenue Impact    │ Is checkout/payment affected?        │
  │                   │ Estimated $/minute loss?             │
  │                   │ Contract or SLA penalties?           │
  │                   │                                      │
  │ Data Impact       │ Is data being lost or corrupted?     │
  │                   │ Is data integrity compromised?       │
  │                   │ Can it be recovered?                 │
  │                   │                                      │
  │ Reputational      │ Is this public-facing?                │
  │ Impact            │ Media/social media attention?        │
  │                   │ Regulatory implications?             │
  │                   │                                      │
  │ Blast Radius      │ Single service or multi-service?     │
  │                   │ Single region or global?             │
  │                   │ Expanding or contained?              │
  └───────────────────┴──────────────────────────────────────┘
```

---

## 5. Incident Priority vs Severity

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  SEVERITY = How bad is the impact right now?             │
  │  URGENCY  = How quickly must we respond?                 │
  │  PRIORITY = Severity × Urgency (what we work on first)  │
  │                                                          │
  │          │ High Urgency  │ Medium       │ Low Urgency    │
  │  ────────┼───────────────┼──────────────┼────────────────│
  │  High    │ P1 CRITICAL   │ P2 HIGH      │ P3 MEDIUM      │
  │  Severity│               │              │                │
  │  ────────┼───────────────┼──────────────┼────────────────│
  │  Medium  │ P2 HIGH       │ P3 MEDIUM    │ P4 LOW         │
  │  Severity│               │              │                │
  │  ────────┼───────────────┼──────────────┼────────────────│
  │  Low     │ P3 MEDIUM     │ P4 LOW       │ P5 PLANNED     │
  │  Severity│               │              │                │
  │                                                          │
  │  Example:                                                │
  │  CEO's email is slow → High urgency, low severity = P3   │
  │  Backup failure at 2 AM → Low urgency, high severity = P3│
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Incident Classification Configuration

```yaml
# PagerDuty / Incident Management Configuration
incident_classification:
  severities:
    SEV1:
      name: "Critical"
      description: "Complete service outage or data breach"
      response_time_minutes: 5
      notification:
        channels: ["pagerduty", "slack-incidents", "email-exec"]
        escalation_policy: "all-hands"
      communication:
        frequency_minutes: 15
        stakeholders: ["engineering", "product", "support", "exec"]
      postmortem: required
      error_budget_impact: "SLO suspension"
      
    SEV2:
      name: "Major"
      description: "Significant degradation affecting >10% users"
      response_time_minutes: 15
      notification:
        channels: ["pagerduty", "slack-incidents"]
        escalation_policy: "on-call-plus-backup"
      communication:
        frequency_minutes: 30
        stakeholders: ["engineering", "product", "support"]
      postmortem: required
      
    SEV3:
      name: "Minor"
      description: "Partial impact with workaround available"
      response_time_minutes: 240
      notification:
        channels: ["slack-incidents"]
        escalation_policy: "on-call-only"
      communication:
        frequency_minutes: 120
        stakeholders: ["engineering"]
      postmortem: optional
      
    SEV4:
      name: "Low"
      description: "No user impact, cosmetic or informational"
      response_time_minutes: 1440  # 24 hours
      notification:
        channels: ["slack-ops"]
        escalation_policy: "none"
      postmortem: not_required
```

---

## 7. Real-World Scenario

### [SCENARIO] Classifying a Production Database Incident

```
  Timeline:
  ──────────────────────────────────────────────────────
  09:00  Alert: Database connection pool at 90% capacity
         → Classification: SEV-4 (approaching limit)
         → Action: On-call notes it, monitors
  
  09:15  Alert: Connection pool at 100%, new connections rejected
         → Reclassify: SEV-3 (some requests failing)
         → Action: On-call investigates
  
  09:25  Alert: Error rate at 40%, orders failing
         → Reclassify: SEV-2 (revenue impact, >10% users)
         → Action: Backup on-call paged, status page updated
  
  09:30  Discovery: Runaway query holding locks
         → Kill query, pool drains
  
  09:35  Error rate back to 0.1%
         → Reclassify: SEV-4 (monitoring for recurrence)
  
  09:45  Incident resolved, total duration: 45 min
         → Postmortem scheduled (was SEV-2)
  ──────────────────────────────────────────────────────
  
  Key learning: Severity is DYNAMIC — adjust as impact evolves
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Everything is classified as SEV-1" | Enforce decision tree. Review last 30 days: if > 2 SEV-1/week, classification is broken. |
| "Team argues over severity during incidents" | Pre-define clear criteria. Use objective thresholds (error rate %, user count). |
| "Under-classification leads to slow response" | Add auto-escalation: if SEV-3 isn't acknowledged in 4 hours, promote to SEV-2. |
| "Business and engineering disagree on impact" | Include revenue impact in severity criteria. Map SLOs to business metrics. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Event** | Any observable system occurrence |
| **Alert** | Event crossing a threshold, needs investigation |
| **Incident** | Unplanned disruption requiring coordinated response |
| **Severity** | Measure of current impact (SEV-1 to SEV-4) |
| **Urgency** | How quickly response is needed |
| **Priority** | Severity × Urgency — determines response order |
| **Blast Radius** | Scope of impact (users, services, regions) |

---

## Quick Revision Questions

1. **What is the difference between an alert and an incident?**
   > An alert is an event that crosses a threshold and requires investigation. An incident is a confirmed unplanned disruption impacting users or SLOs that requires coordinated response.

2. **Describe the four severity levels and their response expectations.**
   > SEV-1: Full outage, 5-min response, all hands. SEV-2: Major degradation, 15-min response, on-call + backup. SEV-3: Minor impact, 4-hour response, on-call only. SEV-4: No user impact, 24-hour response, backlog.

3. **How does priority differ from severity?**
   > Severity measures how bad the impact is. Priority = Severity × Urgency — it determines what to work on first. A high-severity low-urgency issue may have the same priority as a low-severity high-urgency one.

4. **Why should severity be treated as dynamic during an incident?**
   > Impact evolves. An alert may start as SEV-4 but escalate as more users are affected. Conversely, mitigation may reduce severity before full resolution.

5. **What are the auto-escalation rules?**
   > SEV-3 unresolved for 4 hours → SEV-2. SEV-2 unresolved for 2 hours → SEV-1. Any data loss or security breach → immediate SEV-1.

---

[← Previous: Redundancy and Replication](../03-Reliability/06-redundancy-and-replication.md) | [Back to README](../README.md) | [Next: Incident Response Process →](02-incident-response-process.md)
