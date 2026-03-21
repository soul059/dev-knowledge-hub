# Unit 8: Post-Incident Activities — Topic 1: Lessons Learned

## Overview

**Lessons learned** is a structured post-incident review that captures what happened, what worked, what didn't, and what should change. This blameless retrospective transforms incidents from costly disruptions into valuable opportunities for organizational learning and security improvement.

---

## 1. Lessons Learned Process

```
LESSONS LEARNED FRAMEWORK:

  ┌────────────────────────────────────────────┐
  │        LESSONS LEARNED PROCESS             │
  │                                            │
  │  1. SCHEDULE MEETING                       │
  │     1-2 weeks after incident closure       │
  │     All involved parties invited           │
  │     Neutral facilitator assigned           │
  │                                            │
  │  2. PREPARE                                │
  │     Distribute incident summary            │
  │     Distribute timeline                    │
  │     Pre-meeting survey (optional)          │
  │     Prepare discussion questions           │
  │                                            │
  │  3. CONDUCT MEETING                        │
  │     Review what happened                   │
  │     Discuss what went well                 │
  │     Discuss what didn't go well            │
  │     Identify improvements                  │
  │     Assign action items                    │
  │                                            │
  │  4. DOCUMENT FINDINGS                      │
  │     Lessons learned report                 │
  │     Action items with owners               │
  │     Distribute to stakeholders             │
  │                                            │
  │  5. FOLLOW UP                              │
  │     Track action items                     │
  │     Verify implementations                 │
  │     Review effectiveness                   │
  └────────────────────────────────────────────┘

TIMING:
  → Too early: Team still fatigued, memories raw
  → Too late: Details forgotten, momentum lost
  → Ideal: 1-2 weeks after closure
  → Complex incidents: May need multiple sessions
```

---

## 2. Meeting Structure

```
LESSONS LEARNED MEETING AGENDA:

  Duration: 1-2 hours
  Facilitator: Neutral party (not IC)
  Attendees: IR team, IT, management, others

  GROUND RULES (establish first):
  → Blameless culture — focus on process
  → Psychological safety — honest discussion
  → All perspectives valued
  → Constructive focus — solutions not blame
  → What's said stays in the room
  → Document observations, not individuals

AGENDA:
  1. INCIDENT SUMMARY (10 min)
     → Facilitator presents timeline
     → Scope and impact summary
     → Response overview

  2. WHAT WENT WELL (20 min)
     → Detection: How was it found?
     → Communication: Was it effective?
     → Containment: Was it timely?
     → Tools: Were they adequate?
     → Team: Did roles work?
     → Processes: Did playbooks help?

  3. WHAT DIDN'T GO WELL (30 min)
     → Detection gaps: What was missed?
     → Communication breakdowns
     → Resource constraints
     → Tool limitations
     → Process gaps
     → Knowledge gaps
     → Timing issues

  4. WHAT SHOULD CHANGE (20 min)
     → Process improvements
     → Tool enhancements
     → Training needs
     → Policy updates
     → Detection improvements
     → Communication improvements

  5. ACTION ITEMS (10 min)
     → Prioritize improvements
     → Assign owners
     → Set deadlines
     → Define success criteria
```

---

## 3. Key Questions

```
DISCUSSION QUESTIONS BY PHASE:

PREPARATION:
  → Were we adequately prepared?
  → Was the IR plan current and useful?
  → Were playbooks available and helpful?
  → Did we have the right tools?
  → Was the team trained for this scenario?
  → Were contacts and escalation paths current?

DETECTION:
  → How was the incident detected?
  → How long was the dwell time?
  → Could we have detected it sooner?
  → Were there missed alerts or indicators?
  → Were detection rules adequate?
  → Did threat intelligence help?

ANALYSIS:
  → Was triage effective and timely?
  → Did we have sufficient data for analysis?
  → Were investigation tools adequate?
  → How long did scoping take?
  → Was the scope accurately determined?
  → Were there analysis bottlenecks?

CONTAINMENT:
  → Was containment timely and effective?
  → Did we contain without alerting the attacker?
  → Was business impact managed well?
  → Were containment decisions documented?
  → Were pre-authorized actions sufficient?

ERADICATION & RECOVERY:
  → Was eradication thorough?
  → Were systems recovered successfully?
  → Was data loss minimized?
  → Were hardening improvements applied?
  → How long did recovery take?

COMMUNICATION:
  → Was internal communication effective?
  → Were the right people notified?
  → Was external communication timely?
  → Were stakeholders kept informed?
  → Did the communication plan work?

OVERALL:
  → What was our biggest success?
  → What was our biggest challenge?
  → What one thing would we change?
  → Do we need additional resources?
  → Are there systemic issues to address?
```

---

## 4. Documenting Lessons

```
LESSONS LEARNED REPORT TEMPLATE:

  ┌──────────────────────────────────────────────┐
  │ LESSONS LEARNED REPORT                       │
  │                                              │
  │ Incident: INC-2024-001                      │
  │ Date of Meeting: 2024-02-01                 │
  │ Facilitator: John Williams                   │
  │ Attendees: [list]                            │
  │                                              │
  │ INCIDENT SUMMARY:                            │
  │ Phishing email led to account compromise,    │
  │ lateral movement, and data exfiltration       │
  │ affecting 5 systems over 3 days.             │
  │                                              │
  │ WHAT WENT WELL:                              │
  │ + SOC detected beaconing within 3 days       │
  │ + Containment was coordinated and effective  │
  │ + IR team communication was excellent        │
  │ + EDR enabled remote investigation           │
  │ + Recovery completed within 5 days           │
  │                                              │
  │ WHAT DIDN'T GO WELL:                         │
  │ - Phishing email bypassed email filter       │
  │ - No MFA on VPN allowed lateral movement     │
  │ - 3-day dwell time before detection          │
  │ - Backup testing revealed 30% failure rate   │
  │ - After-hours response delayed by 2 hours    │
  │ - IR plan was outdated (18 months old)       │
  │                                              │
  │ ROOT CAUSE:                                  │
  │ Email filtering did not block credential     │
  │ harvesting URL. Lack of MFA on VPN enabled   │
  │ lateral movement with stolen credentials.    │
  │                                              │
  │ ACTION ITEMS: [See action item table]        │
  └──────────────────────────────────────────────┘

ACTION ITEMS TABLE:
  ID   | Action               | Owner    | Due    | Priority
  LL-01| Deploy advanced email | SecEng   | 30 days| Critical
       | filtering with URL   |          |        |
       | sandboxing           |          |        |
  LL-02| Implement MFA for    | IAM Team | 45 days| Critical
       | VPN access           |          |        |
  LL-03| Improve beaconing    | SOC      | 14 days| High
       | detection rules      |          |        |
  LL-04| Fix backup testing   | IT Ops   | 30 days| High
       | process              |          |        |
  LL-05| Establish on-call    | IR Mgr   | 14 days| High
       | rotation             |          |        |
  LL-06| Update IR plan       | IR Mgr   | 30 days| Medium
  LL-07| Phishing awareness   | Security | 30 days| Medium
       | training refresh     |          |        |
```

---

## 5. Follow-Up and Tracking

```
ACTION ITEM TRACKING:

TRACKING PROCESS:
  → Weekly status updates on action items
  → Monthly review with IR manager
  → Quarterly review with CISO
  → Close items when implemented AND verified
  → Escalate overdue items

STATUS REPORTING:
  Report Date: 2024-03-01

  ID   | Description          | Status      | Notes
  LL-01| Email filtering      | ✓ Complete  | Proofpoint deployed
  LL-02| MFA for VPN          | In Progress | 80% deployed
  LL-03| Beaconing detection  | ✓ Complete  | 3 new rules
  LL-04| Backup testing       | In Progress | Process created
  LL-05| On-call rotation     | ✓ Complete  | Active since Feb
  LL-06| Update IR plan       | In Progress | Draft review
  LL-07| Phishing training    | Not Started | Scheduled March

  Overall: 3/7 complete, 3/7 in progress, 1/7 not started

EFFECTIVENESS VALIDATION:
  → Did the change actually address the issue?
  → Test with purple team exercise
  → Run tabletop scenario
  → Measure improvement metrics
  → Gather feedback from team
```

---

## Summary Table

| Phase | Activities | Timing | Output |
|-------|-----------|--------|--------|
| Schedule | Plan meeting, invite | Week 1 post-close | Calendar event |
| Prepare | Distribute materials | Week 1-2 | Pre-read packet |
| Conduct | Facilitated discussion | Week 2 | Raw notes |
| Document | Report and action items | Week 2-3 | LL Report |
| Follow-up | Track action items | Ongoing | Status reports |

---

## Revision Questions

1. Why is a blameless culture essential for lessons learned?
2. What should a lessons learned meeting agenda include?
3. What key questions should be discussed for each IR phase?
4. What should a lessons learned report document?
5. How should action items be tracked and validated?

---

*Previous: None (First topic in this unit) | Next: [02-report-writing.md](02-report-writing.md)*

---

*[Back to README](../README.md)*
