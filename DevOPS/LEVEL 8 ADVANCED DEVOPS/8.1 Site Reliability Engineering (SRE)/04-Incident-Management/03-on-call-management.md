# Chapter 4.3: On-Call Management

[← Previous: Incident Response Process](02-incident-response-process.md) | [Back to README](../README.md) | [Next: Postmortems & RCAs →](04-postmortems-rcas.md)

---

## Overview

On-call is the practice of designating engineers to respond to production incidents outside normal work hours. Effective on-call management balances service reliability with engineer well-being. Poor on-call practices lead to burnout, attrition, and paradoxically, worse reliability as tired engineers make more mistakes.

---

## 1. On-Call Structure

```
  ┌──────────────────────────────────────────────────────────┐
  │            TYPICAL ON-CALL ROTATION                      │
  │                                                          │
  │  Primary On-Call ──────── First responder                │
  │       │                   Acknowledges all alerts        │
  │       │ escalate           Attempts initial mitigation   │
  │       ▼                                                  │
  │  Secondary On-Call ────── Backup if primary unavailable  │
  │       │                   Helps with complex issues      │
  │       │ escalate           Takes over if primary is      │
  │       ▼                   overwhelmed                    │
  │  Engineering Manager ──── Escalation for decisions       │
  │       │                   Resource allocation            │
  │       │                   Customer communication         │
  │       ▼                                                  │
  │  Senior Leadership ───── SEV-1 awareness                 │
  │                          Business decisions              │
  │                                                          │
  │  Rotation: Weekly (most common)                          │
  │  Team minimum: 6-8 engineers for sustainable rotation    │
  │  Max pages before escalation: culture-dependent          │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. On-Call Schedule Design

```
  ┌───────────────────────────────────────────────────────────┐
  │  WEEK  │ MON-FRI Business │ MON-FRI Off-hours │ WEEKEND  │
  │        │ (09:00-18:00)    │ (18:00-09:00)     │          │
  ├────────┼──────────────────┼───────────────────┼──────────┤
  │ Week 1 │ Alice (primary)  │ Alice (primary)   │ Alice    │
  │        │ Bob (secondary)  │ Bob (secondary)   │ Bob      │
  ├────────┼──────────────────┼───────────────────┼──────────┤
  │ Week 2 │ Charlie          │ Charlie           │ Charlie  │
  │        │ Diana            │ Diana             │ Diana    │
  ├────────┼──────────────────┼───────────────────┼──────────┤
  │ Week 3 │ Eve              │ Eve               │ Eve      │
  │        │ Frank            │ Frank             │ Frank    │
  ├────────┼──────────────────┼───────────────────┼──────────┤
  │ Week 4 │ Bob              │ Bob               │ Bob      │
  │        │ Alice            │ Alice             │ Alice    │
  └────────┴──────────────────┴───────────────────┴──────────┘
  
  Google's recommendation:
  ├─ Maximum 25% of time on-call (1 week in 4)
  ├─ No more than 2 incidents per 12-hour shift
  ├─ 6 hours minimum uninterrupted sleep
  └─ Compensatory time off after high-page shifts
```

---

## 3. On-Call Expectations and Compensation

```
  ┌──────────────────────────────────────────────────────────┐
  │  EXPECTATION           │ REQUIREMENT                     │
  ├────────────────────────┼─────────────────────────────────┤
  │ Response time          │ 5 min (SEV-1), 15 min (SEV-2)  │
  │ Availability           │ Laptop and internet at all times│
  │ Sobriety               │ No alcohol during on-call shift │
  │ Location               │ Within response time of laptop  │
  │ Handoff                │ Brief next on-call on active    │
  │                        │ issues at shift change          │
  │ Documentation          │ Update runbooks during shifts   │
  ├────────────────────────┼─────────────────────────────────┤
  │  COMPENSATION          │ OPTIONS                         │
  ├────────────────────────┼─────────────────────────────────┤
  │ On-call stipend        │ $X/day for carrying the pager   │
  │ Incident bonus         │ $X per incident responded to    │
  │ Compensatory time off  │ Day off after weekend on-call   │
  │ Reduced next-day load  │ Light work day after night page │
  └────────────────────────┴─────────────────────────────────┘
```

---

## 4. Alert Quality and On-Call Health

```
  ┌──────────────────────────────────────────────────────────┐
  │  ON-CALL HEALTH METRICS                                  │
  │                                                          │
  │  Metric                 │ Target  │ Red Flag             │
  │  ───────────────────────┼─────────┼──────────────────────│
  │  Pages per shift        │ < 2     │ > 5 per shift        │
  │  Actionable page rate   │ > 80%   │ < 50%                │
  │  Acknowledged in SLA    │ > 95%   │ < 80%                │
  │  Incidents requiring    │ < 10%   │ > 30%                │
  │  escalation             │         │                      │
  │  Time to acknowledge    │ < 5 min │ > 15 min             │
  │  Night pages            │ < 1/wk  │ > 3/wk               │
  │  Alert:Incident ratio   │ < 3:1   │ > 10:1               │
  │                                                          │
  │  [!] High page volume = signal that system reliability   │
  │      or alert quality needs investment, NOT that         │
  │      engineers should "deal with it"                     │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Alert Fatigue Prevention

```
  ┌──────────────────────────────────────────────────────────┐
  │  ALERT FATIGUE CYCLE:                                    │
  │                                                          │
  │  Too many  ──▶  Engineers  ──▶  Slow       ──▶  Real     │
  │  alerts        ignore them     response        incidents │
  │                                                 missed   │
  │        ▲                                          │      │
  │        └──────────────────── more alerts ─────────┘      │
  │                                                          │
  │  BREAKING THE CYCLE:                                     │
  │                                                          │
  │  1. Audit all alerts quarterly                           │
  │     ├─ Did it page? Was it actionable?                   │
  │     ├─ Delete or downgrade non-actionable alerts         │
  │     └─ Target: every page has a runbook entry            │
  │                                                          │
  │  2. Categorize alert outcomes                            │
  │     ├─ Actionable (issue found, action taken)            │
  │     ├─ Non-actionable (false positive, auto-resolved)    │
  │     └─ Noise (known issue, already tracked)              │
  │                                                          │
  │  3. Prevent alert storms                                 │
  │     ├─ Group related alerts (alert correlation)          │
  │     ├─ Suppress dependent alerts (if LB down, don't     │
  │     │  page for each backend separately)                 │
  │     └─ Rate-limit pages (max 1 per service per 15 min)  │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. On-Call Handoff Procedure

```
  ┌──────────────────────────────────────────────────────────┐
  │  HANDOFF CHECKLIST (outgoing → incoming)                 │
  │                                                          │
  │  □ Active incidents and their status                     │
  │  □ Recent deployments (last 24-48 hours)                 │
  │  □ Known flaky alerts to expect                          │
  │  □ Maintenance windows coming up                         │
  │  □ Action items from recent postmortems                  │
  │  □ Any temporary workarounds in place                    │
  │  □ Capacity concerns or upcoming traffic events          │
  │  □ Verify access (VPN, dashboards, PagerDuty)           │
  │                                                          │
  │  Format: 15-min handoff meeting or shared document       │
  │                                                          │
  │  Example handoff note:                                   │
  │  "Quiet week. One SEV-3 on Wednesday (DB slow query,     │
  │   auto-resolved). New deploy of auth-service v2.3 on     │
  │   Thursday — watch for elevated 401 errors. Redis        │
  │   memory alert is known noise — ticket INFRA-2341."     │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. PagerDuty Configuration Example

```yaml
# PagerDuty Escalation Policy
escalation_policy:
  name: "SRE Team - Production"
  repeat_enabled: true
  num_loops: 3
  
  escalation_rules:
    - escalation_delay_in_minutes: 5
      targets:
        - type: "schedule_reference"
          id: "primary-on-call-schedule"
    
    - escalation_delay_in_minutes: 10
      targets:
        - type: "schedule_reference"
          id: "secondary-on-call-schedule"
    
    - escalation_delay_in_minutes: 15
      targets:
        - type: "user_reference"
          id: "engineering-manager"

# On-Call Schedule
schedule:
  name: "SRE Primary On-Call"
  time_zone: "UTC"
  schedule_layers:
    - name: "Weekly Rotation"
      rotation_virtual_start: "2024-01-01T09:00:00Z"
      rotation_turn_length_seconds: 604800  # 7 days
      users:
        - id: "alice"
        - id: "bob"
        - id: "charlie"
        - id: "diana"
        - id: "eve"
        - id: "frank"
      restrictions:
        - type: "daily_restriction"
          start_time_of_day: "09:00:00"
          duration_seconds: 86400  # 24 hours
```

---

## 8. Real-World Scenario

### [SCENARIO] Improving On-Call for a Burned-Out Team

```
  BEFORE:
  ┌──────────────────────────────────────────────┐
  │ Team: 4 engineers                            │
  │ Rotation: 1 week on, 3 weeks off (25%)       │
  │ Pages per week: 15-25                        │
  │ Actionable rate: 40%                         │
  │ Night pages: 5-8 per week                    │
  │ Result: 2 engineers quit in 6 months         │
  └──────────────────────────────────────────────┘
  
  ACTIONS TAKEN:
  1. Alert audit: deleted 60% of alerts (non-actionable)
  2. Added auto-remediation for top 3 recurring alerts
  3. Set SLO-based alerting (symptom, not cause)
  4. Hired 2 more engineers (6-person rotation)
  5. Implemented compensatory time off
  6. Created runbooks for all remaining alerts
  
  AFTER (3 months later):
  ┌──────────────────────────────────────────────┐
  │ Team: 6 engineers                            │
  │ Rotation: 1 week on, 5 weeks off (17%)       │
  │ Pages per week: 3-5                          │
  │ Actionable rate: 90%                         │
  │ Night pages: 0-1 per week                    │
  │ Result: Zero attrition, team morale high     │
  └──────────────────────────────────────────────┘
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Engineers dread on-call" | Reduce page volume. Compensate properly. Ensure every page has a runbook. |
| "Night pages are too frequent" | Invest in auto-remediation. Batch non-urgent alerts to business hours. |
| "New team members are afraid of on-call" | Shadow rotations (2 weeks shadow before primary). Pair with experienced engineer. |
| "Handoffs are sloppy, context lost" | Mandatory 15-min handoff meeting with written notes. Template in team wiki. |
| "On-call creates hero culture" | Spread knowledge. Rotate systems. No single-person dependency. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Primary On-Call** | First responder for all alerts |
| **Secondary On-Call** | Backup, helps with complex incidents |
| **Rotation Length** | Typically 1 week; min 6-8 people |
| **Max On-Call Time** | 25% (Google recommendation) |
| **Page Target** | < 2 per 12-hour shift |
| **Actionable Rate** | > 80% of pages should require action |
| **Handoff** | Structured transfer of context between shifts |
| **Alert Fatigue** | Excessive non-actionable alerts causing numbness |

---

## Quick Revision Questions

1. **What is the minimum team size for sustainable on-call and why?**
   > 6-8 engineers. This ensures each person is on-call no more than 25% of the time (1 week in 4-6), preventing burnout and allowing time for project work.

2. **What metrics indicate unhealthy on-call?**
   > Pages > 5 per shift, actionable rate < 50%, night pages > 3/week, escalation rate > 30%, acknowledgment time > 15 min.

3. **How do you break the alert fatigue cycle?**
   > Audit alerts quarterly, delete non-actionable ones, add runbooks, implement alert grouping and suppression of dependent alerts, and auto-remediate recurring issues.

4. **What should an on-call handoff include?**
   > Active incidents, recent deployments, known flaky alerts, upcoming maintenance windows, pending postmortem actions, temporary workarounds, and capacity concerns.

5. **Why is compensation important for on-call?**
   > On-call restricts personal time and carries stress. Without compensation, it breeds resentment and increases attrition. Options include stipends, incident bonuses, and compensatory time off.

6. **What is Google's recommendation for maximum on-call incidents per shift?**
   > No more than 2 incidents per 12-hour shift. More than that indicates a reliability or alert quality problem that needs engineering investment.

---

[← Previous: Incident Response Process](02-incident-response-process.md) | [Back to README](../README.md) | [Next: Postmortems & RCAs →](04-postmortems-rcas.md)
