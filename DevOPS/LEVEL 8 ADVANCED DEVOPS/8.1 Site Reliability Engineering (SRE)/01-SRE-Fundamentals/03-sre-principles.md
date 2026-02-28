# Chapter 1.3: SRE Principles

[← Previous: SRE vs DevOps](02-sre-vs-devops.md) | [Back to README](../README.md) | [Next: Google's SRE Model →](04-googles-sre-model.md)

---

## Overview

SRE is built on a set of foundational principles that guide every decision — from how reliability targets are set, to how incidents are handled, to how engineers spend their time. These principles form the bedrock of a functioning SRE practice and distinguish SRE from traditional operations.

---

## 1. The Seven Core Principles of SRE

```
  ┌────────────────────────────────────────────────────────────┐
  │                  SRE CORE PRINCIPLES                       │
  ├────────────────────────────────────────────────────────────┤
  │                                                            │
  │  1. Embracing Risk                                         │
  │  2. Service Level Objectives (SLOs)                        │
  │  3. Eliminating Toil                                       │
  │  4. Monitoring & Observability                             │
  │  5. Automation                                             │
  │  6. Release Engineering                                    │
  │  7. Simplicity                                             │
  │                                                            │
  └────────────────────────────────────────────────────────────┘
```

---

## 2. Principle 1: Embracing Risk

### The Concept

100% reliability is the wrong target. Every incremental "nine" of availability costs exponentially more. SRE accepts that some level of failure is inevitable and desirable.

```
  Cost vs Reliability Curve
  
  Cost ($)
    │
    │                                          *
    │                                       *
    │                                    *
    │                                *
    │                           *
    │                      *
    │                *
    │          *
    │     *
    │  *
    │*
    └──────────────────────────────────────── Reliability (%)
     99%   99.9%  99.95% 99.99% 99.999%
     
   [!] Each additional "nine" roughly costs 10x more
```

### Error Budget Formula

```
Error Budget = 1 - SLO

Example:
  SLO = 99.9%
  Error Budget = 1 - 0.999 = 0.001 = 0.1%
  
  In a 30-day month (43,200 minutes):
  Allowed downtime = 43,200 × 0.001 = 43.2 minutes
```

### Decision Framework

```
  ┌────────────────────────────────┐
  │   Error Budget Remaining?      │
  ├────────────┬───────────────────┤
  │    YES     │       NO          │
  │            │                   │
  │  ┌────────┐│  ┌──────────────┐│
  │  │ Deploy ││  │ Freeze       ││
  │  │ freely ││  │ deployments  ││
  │  │        ││  │              ││
  │  │ Take   ││  │ Focus on     ││
  │  │ risks  ││  │ reliability  ││
  │  └────────┘│  └──────────────┘│
  └────────────┴───────────────────┘
```

---

## 3. Principle 2: Service Level Objectives (SLOs)

SLOs are the primary mechanism for making reliability decisions.

```
  ┌────────────────────────────────────────────────┐
  │         SLI → SLO → SLA Pipeline               │
  │                                                │
  │  SLI (Indicator)                               │
  │    "What do we measure?"                       │
  │    e.g., request latency, error rate           │
  │         │                                      │
  │         ▼                                      │
  │  SLO (Objective)                               │
  │    "What target do we set?"                    │
  │    e.g., 99.9% of requests < 200ms            │
  │         │                                      │
  │         ▼                                      │
  │  SLA (Agreement)                               │
  │    "What happens if we fail?"                  │
  │    e.g., Customer gets credits if SLO missed   │
  │                                                │
  └────────────────────────────────────────────────┘
```

### Key Rules for SLOs

1. **User-centric** — Measure what users experience, not server metrics
2. **Fewer is better** — 3-5 SLOs per service, max
3. **Aspirational but achievable** — Not so easy they're meaningless, not so hard they're ignored
4. **Drive action** — SLOs must trigger concrete responses when violated

---

## 4. Principle 3: Eliminating Toil

### What Qualifies as Toil?

Toil is work that is:

```
  ┌─────────────────────────────────────────────┐
  │              IS IT TOIL?                     │
  │                                             │
  │  ✓ Manual         - Human must do it        │
  │  ✓ Repetitive     - Done over and over      │
  │  ✓ Automatable    - Could be scripted       │
  │  ✓ Tactical       - Reactive, interrupt-    │
  │                     driven                  │
  │  ✓ No enduring    - Doesn't improve the     │
  │    value            system permanently      │
  │  ✓ Scales with    - Grows O(n) with service │
  │    service growth   size                    │
  └─────────────────────────────────────────────┘
```

### The 50% Rule

```
  SRE Time Budget:
  
  ┌─────────────────────────────────────────┐
  │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│░░░░░░░░░░░░░░░░░│
  │   Engineering ≥50%   │  Toil ≤50%       │
  │                      │                  │
  │ • Automation         │ • On-call        │
  │ • Tooling            │ • Manual deploys │
  │ • Architecture       │ • Ticket work    │
  │ • Reliability proj.  │ • Restarts       │
  └─────────────────────────────────────────┘
  
  [!] If toil exceeds 50%, escalate to management.
      This is a signal the team is understaffed or
      needs better automation.
```

---

## 5. Principle 4: Monitoring & Observability

### The Monitoring Hierarchy

```
  ┌──────────────────────────────────────────────────┐
  │           MONITORING HIERARCHY                    │
  │                                                  │
  │  Level 5:  Business Metrics                      │
  │            (revenue, sign-ups, conversions)       │
  │                    ▲                              │
  │  Level 4:  User Experience                       │
  │            (page load, error pages)              │
  │                    ▲                              │
  │  Level 3:  Application Metrics                   │
  │            (request rate, error rate, latency)    │
  │                    ▲                              │
  │  Level 2:  Service/Middleware                     │
  │            (queue depth, cache hit rate, DB)      │
  │                    ▲                              │
  │  Level 1:  Infrastructure                        │
  │            (CPU, memory, disk, network)           │
  │                                                  │
  └──────────────────────────────────────────────────┘
```

### Alerting Philosophy

Alerts should only fire when **human action is needed**. SRE categorizes signals into:

| Type | Action | Example |
|------|--------|---------|
| **Pages** | Wake someone up — immediate action needed | SLO burn rate critical |
| **Tickets** | Action needed within hours/days | Disk 80% full |
| **Logs** | Informational, no action unless pattern emerges | Occasional retry |

### The Four Golden Signals

```
  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐
  │  LATENCY   │  │  TRAFFIC   │  │   ERRORS   │  │ SATURATION │
  │            │  │            │  │            │  │            │
  │ How long   │  │ How much   │  │ Rate of    │  │ How "full" │
  │ requests   │  │ demand on  │  │ failed     │  │ the service│
  │ take       │  │ the system │  │ requests   │  │ is         │
  │            │  │            │  │            │  │            │
  │ p50, p95,  │  │ Requests/  │  │ HTTP 5xx,  │  │ CPU, memory│
  │ p99        │  │ second     │  │ exceptions │  │ queue depth│
  └────────────┘  └────────────┘  └────────────┘  └────────────┘
```

---

## 6. Principle 5: Automation

### Automation Value Hierarchy

```
  Value of Automation (lowest → highest):

  Level 0: No automation
            └── Human performs task manually every time

  Level 1: Documented procedure
            └── Written runbook, human follows steps

  Level 2: Scripted with human trigger
            └── Script exists but human must invoke it

  Level 3: Automated with human oversight
            └── System acts automatically, human monitors

  Level 4: Fully automated, self-healing
            └── System detects, diagnoses, and fixes
                without human involvement
```

### When to Automate

```
  Time to Automate vs Frequency
  
  If you do it...    And it takes...   Automate if it saves...
  ─────────────────  ────────────────  ──────────────────────
  Daily              5 minutes         30+ hours/year
  Weekly             30 minutes        26+ hours/year
  Monthly            2 hours           24+ hours/year
  Quarterly          4 hours           16+ hours/year
  Yearly             8 hours           Not worth it (usually)
```

---

## 7. Principle 6: Release Engineering

Safe, repeatable, and automated releases are critical to reliability.

```
  ┌──────────────────────────────────────────────┐
  │          RELEASE ENGINEERING IN SRE           │
  │                                              │
  │  Commit ──► Build ──► Test ──► Stage ──►     │
  │                                          │   │
  │    ┌────────────────────────────────────┐ │   │
  │    │     Progressive Rollout            │ │   │
  │    │                                    │ │   │
  │    │  1% ──► 5% ──► 25% ──► 50% ──► 100%   │
  │    │   │      │       │       │             │
  │    │   ▼      ▼       ▼       ▼             │
  │    │ Monitor Monitor Monitor Monitor        │
  │    │ SLIs    SLIs    SLIs    SLIs           │
  │    │                                    │   │
  │    │  ◄── Rollback at any stage ───────►│   │
  │    └────────────────────────────────────┘   │
  │                                              │
  └──────────────────────────────────────────────┘
```

### Key Practices

- **Hermetic builds** — identical output from identical input
- **Canary releases** — test with small traffic percentage first
- **Feature flags** — decouple deploy from release
- **Automated rollback** — revert automatically on SLO violation

---

## 8. Principle 7: Simplicity

> **"A system is reliable only to the extent that it can be understood."**

```
  ┌────────────────────────────────────────────┐
  │          SIMPLICITY IN SRE                 │
  │                                            │
  │  • Fewer moving parts = fewer failures     │
  │  • Every line of code is a liability       │
  │  • Remove dead code and features           │
  │  • Prefer boring technology                │
  │  • Minimize accidental complexity          │
  │                                            │
  │  Essential        vs      Accidental       │
  │  Complexity               Complexity       │
  │  (inherent to             (introduced by   │
  │   the problem)             poor design)    │
  │                                            │
  │  Goal: Minimize accidental complexity      │
  └────────────────────────────────────────────┘
```

### The "Boring Technology" Principle

- Choose well-understood, battle-tested technology
- Innovation tokens are limited — spend them on your core differentiator
- PostgreSQL > newest distributed DB (unless you truly need it)

---

## 9. Principles in Action — Real-World Scenario

### [SCENARIO] Applying SRE Principles to a Payment Service

```
  Problem: Payment service has frequent outages

  Principle Applied          Action Taken
  ─────────────────          ───────────────────────────────
  Embracing Risk        →    Set 99.95% availability SLO
                              (not 99.99% — too expensive)

  SLOs                  →    Measure: success rate, p99
                              latency, end-to-end duration

  Eliminating Toil      →    Automated cert rotation,
                              DB failover, log rotation

  Monitoring            →    Four golden signals dashboards,
                              SLO burn-rate alerts

  Automation            →    Self-healing: auto-restart on
                              OOM, auto-scale on load

  Release Engineering   →    Canary deploys (1% → 5% → 100%)
                              with automatic rollback

  Simplicity            →    Removed 3 unused middleware
                              components, simplified retry logic
```

---

## 10. Troubleshooting Tips

| Challenge | Guidance |
|-----------|----------|
| "We set SLOs but nobody looks at them" | Make SLOs visible: dashboards in common areas, weekly SLO reviews, error budget reports |
| "Engineers resist the 50% toil cap" | Track toil formally. When it exceeds 50%, management must either add headcount or reduce scope |
| "Too many alerts, alert fatigue" | If an alert doesn't require human action, it shouldn't be an alert. Ruthlessly prune |
| "Automation breaks things" | Automate incrementally. Start with human-triggered scripts, then add full automation |
| "Simplicity competes with features" | Every new feature adds operational load. Budget reliability work alongside feature work |

---

## Summary Table

| Principle | Core Idea | Key Metric/Practice |
|-----------|-----------|---------------------|
| **Embracing Risk** | 100% is wrong; accept quantified failure | Error budget = 1 − SLO |
| **SLOs** | Data-driven reliability targets | SLI → SLO → SLA |
| **Eliminating Toil** | Reduce manual, repetitive work | ≤ 50% toil cap |
| **Monitoring** | Observe what matters, alert on what's actionable | Four golden signals |
| **Automation** | Replace human toil with code | Automation value hierarchy |
| **Release Engineering** | Safe, repeatable, progressive deploys | Canary, rollback, feature flags |
| **Simplicity** | Fewer moving parts = more reliable | Remove accidental complexity |

---

## Quick Revision Questions

1. **Why does SRE say 100% reliability is the wrong target?**
   > Each additional nine costs exponentially more, and users can't distinguish 99.999% from 100%. The cost of preventing all failure exceeds the benefit.

2. **What is the error budget formula, and how does it influence deployments?**
   > Error Budget = 1 − SLO. While budget remains, teams deploy freely. When budget is exhausted, feature deployments freeze and focus shifts to reliability.

3. **List the five characteristics that define "toil."**
   > Manual, repetitive, automatable, tactical/reactive, no enduring value, and scales linearly with service size.

4. **What are the Four Golden Signals, and why are they preferred over low-level metrics?**
   > Latency, Traffic, Errors, Saturation. They directly reflect user experience, unlike CPU/memory which are causes, not symptoms.

5. **Describe the automation value hierarchy from Level 0 to Level 4.**
   > L0: No automation → L1: Documented procedure → L2: Script with human trigger → L3: Automated with oversight → L4: Fully autonomous self-healing.

6. **What does "boring technology" mean in SRE, and why is it preferred?**
   > Choose well-understood, battle-tested tools over cutting-edge ones. Innovation tokens are limited; spend them on your core business differentiator.

---

[← Previous: SRE vs DevOps](02-sre-vs-devops.md) | [Back to README](../README.md) | [Next: Google's SRE Model →](04-googles-sre-model.md)
