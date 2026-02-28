# Chapter 4: On-Call Management

[← Previous: PagerDuty & OpsGenie](03-pagerduty-opsgenie.md) | [Back to README](../README.md) | [Next: Incident Response →](05-incident-response.md)

---

## Overview

On-call management is the human side of alerting. It determines who responds to incidents, when, and how. Good on-call practices reduce burnout, improve response times, and create sustainable operations.

---

## 4.1 On-Call Rotation Models

```
┌──────────────────────────────────────────────────────────┐
│  ON-CALL ROTATION MODELS                                 │
│                                                          │
│  1. WEEKLY ROTATION (Most Common)                       │
│  ┌───┬───┬───┬───┬───┬───┬───┐                         │
│  │Mon│Tue│Wed│Thu│Fri│Sat│Sun│                         │
│  ├───┴───┴───┴───┴───┴───┴───┤                         │
│  │    Alice (Week 1)          │                         │
│  ├───────────────────────────┤                         │
│  │    Bob (Week 2)            │                         │
│  ├───────────────────────────┤                         │
│  │    Carol (Week 3)          │                         │
│  └───────────────────────────┘                         │
│                                                          │
│  2. FOLLOW-THE-SUN (Global Teams)                       │
│  ┌────────┬──────────┬────────┐                        │
│  │ APAC   │ EMEA     │ US     │  Each covers           │
│  │ 8am-4pm│ 8am-4pm  │8am-4pm│  business hours         │
│  │ +8 UTC │ +0 UTC   │-5 UTC │  in their timezone     │
│  └────────┴──────────┴────────┘                        │
│  → No one is woken up at night                          │
│                                                          │
│  3. PRIMARY + SECONDARY                                 │
│  Primary:   First responder, gets paged                │
│  Secondary: Backup if primary doesn't ack in 5 min     │
│  Shadow:    Training role, observes incidents           │
│                                                          │
│  4. SPLIT BY SERVICE                                    │
│  Team A: API + Frontend on-call                        │
│  Team B: Data pipeline on-call                         │
│  Team C: Infrastructure on-call                        │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 On-Call Best Practices

```
┌────────────────────────────────────────────────────────┐
│  ON-CALL ENGINEER RIGHTS & RESPONSIBILITIES            │
│                                                        │
│  Rights:                                               │
│  ✓ Compensated for on-call time                       │
│  ✓ Time off after incident-heavy rotation             │
│  ✓ Ability to swap shifts                             │
│  ✓ Clear escalation path (never alone)                │
│  ✓ Runbooks for every alert                           │
│  ✓ Authority to page others when needed               │
│                                                        │
│  Responsibilities:                                     │
│  ✓ Acknowledge alerts within 5 minutes                │
│  ✓ Have laptop and internet access                    │
│  ✓ Stay within phone/notification range               │
│  ✓ Follow runbook procedures                          │
│  ✓ Escalate when unsure (never guess)                 │
│  ✓ Write incident notes and postmortems               │
└────────────────────────────────────────────────────────┘
```

### Minimum Team Size for Sustainable On-Call

| Rotation Type | Min Engineers | Rationale |
|--------------|---------------|-----------|
| Weekly, 24/7 | 5-6 | On-call once every 5-6 weeks |
| Follow-the-sun | 3 per region | Coverage during business hours only |
| Primary + Secondary | 8+ | Two people always available |
| Weekend-only split | 4+ | Separate weekday/weekend rotations |

> **Rule of thumb**: No one should be on-call more than once every 4 weeks to prevent burnout.

---

## 4.3 On-Call Schedule Setup

### PagerDuty Schedule (Terraform)

```hcl
resource "pagerduty_schedule" "primary" {
  name      = "Platform Primary On-Call"
  time_zone = "America/New_York"

  layer {
    name                         = "Weekly Rotation"
    start                        = "2024-01-01T09:00:00-05:00"
    rotation_virtual_start       = "2024-01-01T09:00:00-05:00"
    rotation_turn_length_seconds = 604800  # 7 days

    users = [
      pagerduty_user.alice.id,
      pagerduty_user.bob.id,
      pagerduty_user.carol.id,
      pagerduty_user.dave.id,
      pagerduty_user.eve.id,
    ]

    restriction {
      type              = "daily_restriction"
      start_time_of_day = "09:00:00"  
      duration_seconds  = 86400       # 24 hours
    }
  }
}

resource "pagerduty_schedule" "secondary" {
  name      = "Platform Secondary On-Call"
  time_zone = "America/New_York"

  layer {
    name                         = "Weekly Rotation (Offset)"
    start                        = "2024-01-08T09:00:00-05:00"
    rotation_virtual_start       = "2024-01-08T09:00:00-05:00"
    rotation_turn_length_seconds = 604800

    # Offset by 1 from primary
    users = [
      pagerduty_user.bob.id,
      pagerduty_user.carol.id,
      pagerduty_user.dave.id,
      pagerduty_user.eve.id,
      pagerduty_user.alice.id,
    ]
  }
}
```

---

## 4.4 On-Call Handoff Process

```
┌──────────────────────────────────────────────────────────┐
│  ON-CALL HANDOFF CHECKLIST                               │
│                                                          │
│  OUTGOING ON-CALL:                                      │
│  □ Document any ongoing issues                          │
│  □ List recent incidents and their status               │
│  □ Note any temporary workarounds in place              │
│  □ Mention any upcoming maintenance windows             │
│  □ Flag any alerts that may need attention              │
│                                                          │
│  INCOMING ON-CALL:                                      │
│  □ Verify notification settings work                    │
│  □ Test phone/SMS/push delivery                         │
│  □ Review recent incident history                       │
│  □ Check current alert status                           │
│  □ Confirm access to all required systems               │
│  □ Review any new runbooks                              │
│                                                          │
│  HANDOFF MEETING (15 min):                              │
│  • Review week's incidents                              │
│  • Discuss any open issues                              │
│  • Transfer any context on ongoing problems             │
│  • Confirm contact information                          │
└──────────────────────────────────────────────────────────┘
```

---

## 4.5 On-Call Metrics

| Metric | Target | Why It Matters |
|--------|--------|----------------|
| MTTA (Mean Time to Acknowledge) | < 5 min | Ensures quick response |
| MTTR (Mean Time to Resolve) | < 1 hour (P1) | Measures effectiveness |
| Pages per shift | < 5 per week | Measures on-call load |
| Off-hours pages | < 2 per week | Measures sleep disruption |
| Escalation rate | < 10% | Measures first-responder effectiveness |
| False positive rate | < 20% | Measures alert quality |

```
┌──────────────────────────────────────────────────────────┐
│  ON-CALL HEALTH DASHBOARD                                │
│                                                          │
│  Pages This Week: ███████ 7 (target < 5)   ⚠           │
│  MTTA:           3.2 min (target < 5 min)   ✓          │
│  MTTR:           45 min  (target < 60 min)  ✓          │
│  Off-Hours:      ████ 4  (target < 2)       ❌          │
│  False Positive: 15%     (target < 20%)     ✓          │
│  Escalations:    1       (target < 10%)     ✓          │
│                                                          │
│  Action: Too many off-hours pages. Review alert         │
│  thresholds or add self-healing automation.              │
└──────────────────────────────────────────────────────────┘
```

---

## 4.6 Reducing On-Call Burden

| Strategy | Description | Impact |
|----------|-------------|--------|
| Improve runbooks | Clear step-by-step guides | Faster resolution, less stress |
| Auto-remediation | Scripts that fix known issues | Fewer pages |
| Alert tuning | Remove noisy alerts | Less fatigue |
| Better testing | Catch bugs before production | Fewer incidents |
| Chaos engineering | Find weaknesses proactively | Fewer surprises |
| Service ownership | Teams own their services | Focused, smaller on-call |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Burnout | Too frequent rotation | Add engineers, minimum 5-person rotation |
| Slow response | Notification settings wrong | Test notifications before shift starts |
| Knowledge gaps | Not all team members trained | Shadow rotation, pair on incidents |
| Alert fatigue | Too many non-actionable pages | Aggressive alert review and pruning |
| Schedule conflicts | No swap mechanism | Implement override/swap in PD/OpsGenie |

---

## Quick Revision Questions

1. **What is the recommended minimum team size for a sustainable on-call rotation?**
2. **What is follow-the-sun on-call and when should you use it?**
3. **What key metrics should you track for on-call health?**
4. **What should be included in an on-call handoff process?**
5. **How do you reduce on-call burden without compromising reliability?**

---

[← Previous: PagerDuty & OpsGenie](03-pagerduty-opsgenie.md) | [Back to README](../README.md) | [Next: Incident Response →](05-incident-response.md)
