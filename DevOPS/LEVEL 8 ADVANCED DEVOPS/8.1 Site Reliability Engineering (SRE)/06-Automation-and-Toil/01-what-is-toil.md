# Chapter 6.1: What is Toil

[← Previous: Cost Optimization](../05-Capacity-Planning/05-cost-optimization.md) | [Back to README](../README.md) | [Next: Measuring Toil →](02-measuring-toil.md)

---

## Overview

Toil is a central concept in SRE. Google defines toil as the kind of work tied to running a production service that tends to be manual, repetitive, automatable, tactical, devoid of enduring value, and that scales linearly as a service grows. Understanding and managing toil is critical because unchecked toil crowds out engineering work, degrades team morale, and prevents reliability improvements.

---

## 1. Defining Toil

```
  ┌──────────────────────────────────────────────────────────┐
  │                    IS IT TOIL?                           │
  │                                                          │
  │  All SIX characteristics must be present:                │
  │                                                          │
  │  ┌────────────────┐  Work done by a human that a         │
  │  │  1. MANUAL     │  machine could do. Running a         │
  │  │                │  script manually = toil.              │
  │  └────────────────┘                                      │
  │  ┌────────────────┐  Done over and over, not a one-time  │
  │  │  2. REPETITIVE │  task. Resizing disks monthly,       │
  │  │                │  restarting pods weekly.              │
  │  └────────────────┘                                      │
  │  ┌────────────────┐  Could be solved by software.        │
  │  │ 3. AUTOMATABLE │  If no automation is possible,       │
  │  │                │  it's not toil — it's just hard.     │
  │  └────────────────┘                                      │
  │  ┌────────────────┐  Interrupt-driven, reactive work.    │
  │  │  4. TACTICAL   │  Not strategic. Firefighting,        │
  │  │                │  not building.                       │
  │  └────────────────┘                                      │
  │  ┌────────────────┐  Doesn't move the service forward.   │
  │  │ 5. NO ENDURING │  After the work is done, nothing     │
  │  │    VALUE       │  is permanently improved.            │
  │  └────────────────┘                                      │
  │  ┌────────────────┐  Work grows proportionally to        │
  │  │ 6. O(n) WITH   │  service size/traffic. More users   │
  │  │    SERVICE     │  = more toil (without automation).   │
  │  │    GROWTH      │                                      │
  │  └────────────────┘                                      │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Toil vs Overhead vs Engineering

```
  ┌──────────────────────────────────────────────────────────┐
  │  TYPE          │ EXAMPLES             │ VALUE            │
  ├────────────────┼──────────────────────┼──────────────────┤
  │ TOIL           │ • Manual deployments │ No lasting value │
  │ (eliminate)    │ • Restarting pods    │ Scales linearly  │
  │                │ • Cert renewals      │ Could automate   │
  │                │ • Ticket processing  │                  │
  │                │ • Manual scaling     │                  │
  │                │ • Password resets    │                  │
  │                │                      │                  │
  │ OVERHEAD       │ • Meetings           │ Necessary but    │
  │ (minimize)    │ • Email              │ not productive.  │
  │                │ • HR tasks           │ Not automatable  │
  │                │ • Team admin         │ in the same way  │
  │                │ • Travel/expenses    │                  │
  │                │                      │                  │
  │ ENGINEERING    │ • Writing automation │ Enduring value.  │
  │ (maximize)    │ • Designing systems  │ Reduces future   │
  │                │ • Improving reliability│ toil. Scales    │
  │                │ • Performance tuning │ sublinearly      │
  │                │ • Building tools     │                  │
  └────────────────┴──────────────────────┴──────────────────┘
```

---

## 3. Google's 50% Rule

```
  ┌──────────────────────────────────────────────────────────┐
  │  SRE TIME ALLOCATION (Google's Target)                   │
  │                                                          │
  │  ┌──────────────────────────┬──────────────────────────┐ │
  │  │                          │                          │ │
  │  │    ENGINEERING WORK      │        TOIL + OPS        │ │
  │  │       ≥ 50%              │         ≤ 50%            │ │
  │  │                          │                          │ │
  │  │  • Feature work          │  • On-call responses     │ │
  │  │  • Automation            │  • Manual deployments    │ │
  │  │  • System design         │  • Ticket handling       │ │
  │  │  • Reliability projects  │  • Monitoring tweaks     │ │
  │  │  • Tool building         │  • Data entry            │ │
  │  │                          │                          │ │
  │  └──────────────────────────┴──────────────────────────┘ │
  │                                                          │
  │  If toil exceeds 50%:                                    │
  │  ├─ Team cannot invest in reliability improvements       │
  │  ├─ Toil will continue to grow (positive feedback loop)  │
  │  ├─ Engineers burn out and leave                         │
  │  └─ Service reliability stagnates or degrades            │
  │                                                          │
  │  The Toil Death Spiral:                                  │
  │  More toil → Less automation → More toil → Burnout      │
  └──────────────────────────────────────────────────────────┘
```

---

## 4. Common Sources of Toil

```
  ┌──────────────────────────────────────────────────────────┐
  │  CATEGORY        │ TOIL EXAMPLES                         │
  ├──────────────────┼───────────────────────────────────────┤
  │ Deployment       │ • Manual deploy steps                 │
  │                  │ • Config file editing by hand          │
  │                  │ • Build artifact management            │
  │                  │                                        │
  │ Scaling          │ • Manually adding servers              │
  │                  │ • Manually resizing databases           │
  │                  │ • Capacity request tickets              │
  │                  │                                        │
  │ Monitoring       │ • Manually checking dashboards          │
  │                  │ • Non-actionable alert noise            │
  │                  │ • Manually correlating logs             │
  │                  │                                        │
  │ Access Control   │ • Manual user provisioning              │
  │                  │ • Password resets                       │
  │                  │ • Certificate renewals                  │
  │                  │                                        │
  │ Data Management  │ • Manual database migrations            │
  │                  │ • Data cleanup scripts run by hand      │
  │                  │ • Manual backup verification            │
  │                  │                                        │
  │ Incident Mgmt   │ • Copy-paste runbook steps              │
  │                  │ • Manual status page updates            │
  │                  │ • Repetitive incident remediation       │
  └──────────────────┴───────────────────────────────────────┘
```

---

## 5. The Impact of Toil

```
  ┌──────────────────────────────────────────────────────────┐
  │  IMPACT DIMENSION   │ EFFECT                             │
  ├─────────────────────┼────────────────────────────────────┤
  │ Team morale         │ Engineers didn't sign up for        │
  │                     │ repetitive manual work. High toil   │
  │                     │ → low satisfaction → attrition.     │
  │                     │                                     │
  │ Career growth       │ Time spent on toil isn't spent      │
  │                     │ learning or building skills.        │
  │                     │                                     │
  │ Reliability         │ Less time for proactive work means  │
  │                     │ bugs don't get fixed, tech debt     │
  │                     │ accumulates, outages increase.      │
  │                     │                                     │
  │ Scalability         │ If ops work grows linearly with     │
  │                     │ service size, you need proportional │
  │                     │ headcount (unsustainable).          │
  │                     │                                     │
  │ Cost                │ Human time = most expensive         │
  │                     │ resource. Automating saves money.   │
  │                     │                                     │
  │ Error rate          │ Manual work → human errors.         │
  │                     │ Automation is more consistent.      │
  └─────────────────────┴────────────────────────────────────┘
```

---

## 6. Toil vs Not-Toil Decision Guide

```
  ┌──────────────────────────────────────────────────────────┐
  │  ACTIVITY                │ TOIL? │ WHY                    │
  ├──────────────────────────┼───────┼────────────────────────┤
  │ Manually deploying v1.5  │ YES   │ Repetitive, automatable│
  │ Restarting crashed pods  │ YES   │ Tactical, no lasting   │
  │                          │       │ value                  │
  │ Manually rotating certs  │ YES   │ Repetitive, automatable│
  │ Acknowledging non-       │ YES   │ Alert noise, no value  │
  │ actionable alerts        │       │                        │
  │                          │       │                        │
  │ Writing deployment       │ NO    │ Creates lasting        │
  │ automation               │       │ automation (eng work)  │
  │ Designing a new service  │ NO    │ Strategic, enduring    │
  │ On-call (novel incident) │ NO    │ Requires human         │
  │                          │       │ judgment, not repetitive│
  │ Writing a postmortem     │ NO    │ Creates lasting        │
  │                          │       │ knowledge              │
  │ Team meeting             │ NO    │ Overhead, not toil     │
  │ Migrating to new DB      │ NO    │ One-time project       │
  │ (one-time)               │       │ (not repetitive)       │
  └──────────────────────────┴───────┴────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] The Toil Takeover

```
  SITUATION: Platform SRE team of 6 engineers
  
  Initial time allocation (measured over 4 weeks):
  ┌──────────────────────────────────────────────────────────┐
  │  Manual deploys (12/week)           ████████  22%        │
  │  Cert renewals (8/month)            ██        5%         │
  │  Capacity tickets                   ████      10%        │
  │  Alert noise investigation          ██████    15%        │
  │  Runbook execution                  ████      10%        │
  │  Password/access requests           ███       8%         │
  │  ─────────────────────────────────────────────           │
  │  TOTAL TOIL                         ████████  70% ← RED │
  │  Engineering time remaining                   30%        │
  └──────────────────────────────────────────────────────────┘
  
  Action plan (prioritized by effort vs impact):
  ├─ Week 1-2: Automate cert renewals (cert-manager)  →  -5%
  ├─ Week 3-4: CI/CD pipeline for deploys             → -18%
  ├─ Week 5-6: Auto-scaling policies                   → -8%
  ├─ Week 7-8: Alert tuning (remove noise)             → -10%
  ├─ Week 9-10: Self-service access (SSO/RBAC)         → -6%
  └─ Result: Toil reduced from 70% → 23%
  
  After 10 weeks:
  ├─ 4 engineers freed up for engineering projects
  ├─ Built self-healing for top 3 incident types
  ├─ Reduced MTTR by 40%
  └─ Team satisfaction score improved from 3.2 → 4.5/5
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Team says they don't have toil" | They may not recognize it. Track all tasks for 2 weeks. Measure against the 6 toil criteria. |
| "We don't have time to automate" | You can't afford not to. Start with highest-frequency toil. Even partial automation helps. |
| "Management doesn't see toil as a problem" | Show the math: engineer hours × hourly cost × frequency. Compare to automation cost. |
| "Some toil feels necessary" | Distinguish between temporary toil (acceptable during incidents) and chronic toil (must eliminate). |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Toil** | Manual, repetitive, automatable, tactical, no enduring value, scales linearly |
| **50% Rule** | SRE teams should spend ≤50% of time on toil, ≥50% on engineering |
| **Overhead** | Administrative work (meetings, email) — minimize but different from toil |
| **Engineering** | Strategic work that creates lasting value — maximize this |
| **Toil Death Spiral** | More toil → less automation → even more toil → burnout |
| **Key Impact** | Morale, reliability, scalability, cost, error rate |

---

## Quick Revision Questions

1. **What are the six characteristics of toil?**
   > Manual, repetitive, automatable, tactical, no enduring value, and scales linearly with service growth. All six must apply for work to be classified as toil.

2. **What is Google's 50% rule for SRE teams?**
   > SRE teams should spend no more than 50% of their time on operational work (toil). At least 50% should be engineering work that creates lasting improvements.

3. **How is toil different from overhead?**
   > Toil is automatable operational work tied to running services. Overhead is administrative work (meetings, email, HR tasks) that isn't automatable in the same way. Both reduce engineering time, but toil is the priority to eliminate.

4. **What is the "toil death spiral"?**
   > When toil exceeds capacity, there's no time for automation, so toil grows further, leading to more toil, burnout, attrition, and eventually, service degradation.

5. **Is a novel on-call incident classified as toil?**
   > No. A novel incident requires human judgment and isn't repetitive. However, if the same incident keeps recurring and the response becomes formulaic, then the remediation becomes toil that should be automated.

6. **Why does toil threaten scalability?**
   > If operational work grows linearly with service size (O(n)), you need proportional headcount growth. This is unsustainable. Automation makes operations scale sublinearly.

---

[← Previous: Cost Optimization](../05-Capacity-Planning/05-cost-optimization.md) | [Back to README](../README.md) | [Next: Measuring Toil →](02-measuring-toil.md)
