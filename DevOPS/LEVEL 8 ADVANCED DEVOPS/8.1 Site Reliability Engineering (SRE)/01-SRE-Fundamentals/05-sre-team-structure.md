# Chapter 1.5: SRE Team Structure

[← Previous: Google's SRE Model](04-googles-sre-model.md) | [Back to README](../README.md) | [Next: SLIs →](../02-Service-Level-Objectives/01-slis.md)

---

## Overview

How you structure an SRE team has a direct impact on its effectiveness. There is no one-size-fits-all model — the right structure depends on organization size, service criticality, and engineering culture. This chapter covers common models, roles, responsibilities, and how to scale SRE teams.

---

## 1. SRE Team Models

### Model 1: Dedicated / Centralized SRE Team

```
  ┌──────────────────────────────────────────────────┐
  │                 Engineering Org                  │
  │                                                  │
  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐        │
  │  │Dev A │  │Dev B │  │Dev C │  │Dev D │        │
  │  └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘        │
  │     │         │         │         │              │
  │     └─────────┴────┬────┴─────────┘              │
  │                    │                             │
  │             ┌──────▼──────┐                      │
  │             │ Central SRE │                      │
  │             │    Team     │                      │
  │             │ (6-15 SREs) │                      │
  │             └─────────────┘                      │
  └──────────────────────────────────────────────────┘
  
  Pros: Consistent standards, shared tooling, career path
  Cons: Can become bottleneck, may lack service context
  Best for: Mid-size orgs (100-500 engineers)
```

### Model 2: Embedded SRE

```
  ┌──────────────────────────────────────────────────┐
  │                 Engineering Org                  │
  │                                                  │
  │  ┌─────────────┐  ┌─────────────┐               │
  │  │   Dev A      │  │   Dev B      │              │
  │  │   + SRE(1-2) │  │   + SRE(1-2) │              │
  │  │              │  │              │              │
  │  │ SRE sits     │  │ SRE sits     │              │
  │  │ inside the   │  │ inside the   │              │
  │  │ dev team     │  │ dev team     │              │
  │  └─────────────┘  └─────────────┘               │
  │                                                  │
  │  ┌─────────────┐  ┌─────────────┐               │
  │  │   Dev C      │  │   Dev D      │              │
  │  │   + SRE(1)   │  │   (no SRE)   │              │
  │  └─────────────┘  └─────────────┘               │
  └──────────────────────────────────────────────────┘
  
  Pros: Deep service knowledge, strong dev relationship
  Cons: Inconsistent practices, isolation, career path issues
  Best for: Large orgs with critical, complex services
```

### Model 3: Hybrid / Hub-and-Spoke

```
  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │          ┌──────────────────────┐                     │
  │          │  SRE Center of      │                     │
  │          │  Excellence (Hub)   │                     │
  │          │                     │                     │
  │          │  • Standards        │                     │
  │          │  • Tooling          │                     │
  │          │  • Training         │                     │
  │          │  • Career path      │                     │
  │          └─────────┬───────────┘                     │
  │            ┌───────┼──────────┐                      │
  │            │       │          │                      │
  │            ▼       ▼          ▼                      │
  │  ┌────────────┐ ┌─────────┐ ┌─────────┐            │
  │  │ Embedded   │ │Embedded │ │Embedded │  (Spokes)  │
  │  │ SRE in     │ │SRE in   │ │SRE in   │            │
  │  │ Team A     │ │Team B   │ │Team C   │            │
  │  └────────────┘ └─────────┘ └─────────┘            │
  │                                                      │
  └──────────────────────────────────────────────────────┘
  
  Pros: Best of both worlds — context + consistency
  Cons: Complex org structure, dual reporting
  Best for: Large orgs (500+ engineers)
```

### Model 4: DevOps with SRE Practices (No Dedicated SRE)

```
  ┌──────────────────────────────────────────┐
  │  Each dev team owns their reliability    │
  │                                          │
  │  ┌────────────┐  ┌────────────┐          │
  │  │  Team A    │  │  Team B    │          │
  │  │ (dev+ops)  │  │ (dev+ops)  │          │
  │  │ Uses SRE   │  │ Uses SRE   │          │
  │  │ practices  │  │ practices  │          │
  │  └────────────┘  └────────────┘          │
  │                                          │
  │  SRE practices adopted:                  │
  │  • SLOs & error budgets                  │
  │  • Blameless postmortems                 │
  │  • On-call rotation within team          │
  │                                          │
  └──────────────────────────────────────────┘
  
  Pros: Simple org structure, everyone owns reliability
  Cons: Inconsistent adoption, no SRE career path
  Best for: Small orgs (<100 engineers) or early startups
```

---

## 2. Comparison of Team Models

| Aspect | Centralized | Embedded | Hybrid | No Dedicated SRE |
|--------|-------------|----------|--------|-------------------|
| **Service context** | Low | High | High | High |
| **Consistency** | High | Low | High | Low |
| **Scalability** | Medium | High | High | Low |
| **Career path** | Clear | Unclear | Clear | None |
| **Bottleneck risk** | High | Low | Low | N/A |
| **Org size** | 100-500 | 500+ | 500+ | <100 |
| **Tooling** | Shared | Fragmented | Shared | Fragmented |

---

## 3. SRE Roles and Responsibilities

```
  ┌──────────────────────────────────────────────────────┐
  │                 SRE TEAM ROLES                       │
  │                                                      │
  │  ┌─────────────────┐  ┌─────────────────┐           │
  │  │  SRE Manager    │  │  Staff/Principal│           │
  │  │                 │  │  SRE             │           │
  │  │  • Team lead    │  │  • Technical     │           │
  │  │  • Stakeholder  │  │    leadership    │           │
  │  │    management   │  │  • Architecture  │           │
  │  │  • headcount    │  │    reviews       │           │
  │  │  • SLO reviews  │  │  • Cross-team    │           │
  │  └────────┬────────┘  │    initiatives   │           │
  │           │           └────────┬─────────┘           │
  │           │                    │                      │
  │           ▼                    ▼                      │
  │  ┌─────────────────────────────────────┐             │
  │  │         Senior SRE                  │             │
  │  │  • On-call lead                     │             │
  │  │  • Incident commander               │             │
  │  │  • Mentors junior SREs              │             │
  │  │  • Complex automation projects      │             │
  │  └────────────────┬────────────────────┘             │
  │                   │                                   │
  │                   ▼                                   │
  │  ┌─────────────────────────────────────┐             │
  │  │         SRE (Mid-level)             │             │
  │  │  • On-call rotation                 │             │
  │  │  • Monitors SLIs/SLOs               │             │
  │  │  • Writes automation                │             │
  │  │  • Contributes to postmortems       │             │
  │  └────────────────┬────────────────────┘             │
  │                   │                                   │
  │                   ▼                                   │
  │  ┌─────────────────────────────────────┐             │
  │  │         Junior SRE                  │             │
  │  │  • Shadow on-call                   │             │
  │  │  • Runbook updates                  │             │
  │  │  • Monitoring improvements          │             │
  │  │  • Toil reduction scripts           │             │
  │  └─────────────────────────────────────┘             │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

---

## 4. SRE Team Sizing

### Rule of Thumb

```
  Minimum viable SRE team: 6-8 people
  
  Why 6-8?
  ┌─────────────────────────────────────────────────┐
  │  On-call needs:                                 │
  │    • 24/7 coverage needs ≥2 people at any time  │
  │    • Sustainable rotation: 1-week-on / 5-off    │
  │    • Minimum: 6 people for one 24/7 rotation    │
  │                                                  │
  │  + 1-2 for engineering projects                  │
  │  + buffer for PTO / sick leave                   │
  │  ──────────────────────────                      │
  │  = 6-8 SREs minimum                              │
  └─────────────────────────────────────────────────┘
```

### Scaling Formula (Industry Guideline)

```
  Small org:    1 SRE per 10-20 developers
  Medium org:   1 SRE per 20-50 developers
  Large org:    1 SRE per 50-100 developers
  
  Service-based:
    Tier 1 (critical):  4-8 SREs dedicated
    Tier 2 (important): 2-4 SREs shared across services
    Tier 3 (standard):  Advisory/consulting only
```

---

## 5. SRE Hiring and Skills

### The T-Shaped SRE

```
  ┌────────────────── Breadth ──────────────────┐
  │  Linux  Networking  Cloud  Containers  CI/CD │
  │  ─────  ──────────  ─────  ──────────  ──── │
  │    │        │         │        │          │  │
  │    │        │         │        │          │  │
  │    │        │         │        │          │  │
  │    │        │         ▼        │          │  │
  │    │        │     ┌──────┐    │          │  │
  │    │        │     │Depth │    │          │  │
  │    │        │     │      │    │          │  │
  │    │        │     │ e.g. │    │          │  │
  │    │        │     │ K8s  │    │          │  │
  │    │        │     │expert│    │          │  │
  │    │        │     │      │    │          │  │
  │    │        │     └──────┘    │          │  │
  └─────────────────────────────────────────────┘
  
  Broad knowledge across systems + deep expertise in 1-2 areas
```

### Key Hiring Criteria

| Category | Must Have | Nice to Have |
|----------|-----------|-------------|
| **Coding** | Python/Go, scripting | System-level (C/Rust) |
| **Systems** | Linux admin, networking, DNS | Kernel internals |
| **Cloud** | One major cloud (AWS/GCP/Azure) | Multi-cloud |
| **Containers** | Docker, Kubernetes basics | Helm, service mesh |
| **Observability** | Prometheus, Grafana | Tracing, custom exporters |
| **Soft Skills** | Clear communication, calm under pressure | Technical writing |
| **Mindset** | Automate everything, data-driven | Chaos engineering |

---

## 6. Day in the Life of an SRE

```
  ┌──────────────────────────────────────────────────────┐
  │              TYPICAL SRE DAY                         │
  │                                                      │
  │  09:00  Check dashboards, review overnight alerts    │
  │         └── Any SLO violations? Error budget status? │
  │                                                      │
  │  09:30  Stand-up with SRE team                       │
  │         └── Incidents, action items, toil report     │
  │                                                      │
  │  10:00  Engineering work (2-3 hours)                 │
  │         └── Automation project: auto-remediation     │
  │             for disk full alerts                      │
  │                                                      │
  │  12:00  Lunch                                        │
  │                                                      │
  │  13:00  Design review with dev team                  │
  │         └── New microservice going to production     │
  │             Review: SLOs, monitoring, failure modes  │
  │                                                      │
  │  14:00  On-call handoff / incident review            │
  │         └── Review postmortem action items            │
  │                                                      │
  │  15:00  Engineering work (2 hours)                   │
  │         └── SLO dashboard improvements               │
  │                                                      │
  │  17:00  End of day / secondary on-call if applicable │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

---

## 7. Building an SRE Team from Scratch

### Step-by-Step Process

```
  Phase 1: Foundation (Month 1-3)
  ┌────────────────────────────────────────────┐
  │  • Hire 2-3 senior SREs                    │
  │  • Define SLIs/SLOs for top 3 services     │
  │  • Set up basic monitoring (Prometheus)     │
  │  • Create first runbooks                    │
  │  • Establish on-call rotation               │
  └────────────────────────────────────────────┘
           │
           ▼
  Phase 2: Growth (Month 3-6)
  ┌────────────────────────────────────────────┐
  │  • Grow team to 6-8                        │
  │  • Implement error budgets                 │
  │  • Start blameless postmortem practice     │
  │  • Automate top 5 manual tasks             │
  │  • Build SLO dashboards                    │
  └────────────────────────────────────────────┘
           │
           ▼
  Phase 3: Maturity (Month 6-12)
  ┌────────────────────────────────────────────┐
  │  • Production Readiness Reviews            │
  │  • Chaos engineering / game days           │
  │  • Capacity planning process               │
  │  • Cross-team reliability standards        │
  │  • SRE training program for devs           │
  └────────────────────────────────────────────┘
           │
           ▼
  Phase 4: Excellence (Month 12+)
  ┌────────────────────────────────────────────┐
  │  • Self-healing automation                 │
  │  • Reliability engineering as culture      │
  │  • Advanced observability (tracing, AIOps) │
  │  • SRE community of practice               │
  │  • Regular reliability reviews             │
  └────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] Choosing the Right Team Model

**Company:** FinTech startup, 150 engineers, 20 microservices

```
  Service Tier  │ Count │ SRE Model
  ──────────────│───────│────────────────────
  Tier 1        │   3   │ Dedicated SRE
  (Payments,    │       │ (4 SREs full-time)
   Auth, Core)  │       │
  ──────────────│───────│────────────────────
  Tier 2        │   7   │ Shared SRE
  (Notifications│       │ (3 SREs across
   Reports,     │       │  these services)
   Dashboard)   │       │
  ──────────────│───────│────────────────────
  Tier 3        │  10   │ Consulting
  (Internal     │       │ (SRE advises,
   tools, batch │       │  dev owns on-call)
   jobs)        │       │
  
  Total: 7 SREs + 1 SRE Manager = 8-person SRE team
  Ratio: 1 SRE per ~19 developers
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "SREs are burned out from on-call" | Minimum 6 people per rotation. Track on-call load. Add headcount if >2 pages per shift average |
| "Dev teams don't respect SRE input" | SRE must demonstrate value: show before/after reliability metrics. Executive sponsorship helps |
| "SREs don't have a career path" | Create SRE-specific ladder: Junior → SRE → Senior → Staff → Principal. Allow SWE ↔ SRE transfers |
| "We can't find SRE hires" | Grow from within: train sysadmins to code, train developers in systems. Internal rotation program |
| "SRE team is a bottleneck" | Move to embedded or hybrid model. Push more ownership to dev teams via SRE training |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Team Models** | Centralized, Embedded, Hybrid, No Dedicated SRE |
| **Minimum Team Size** | 6-8 for sustainable 24/7 on-call |
| **Sizing Ratio** | 1 SRE per 10-50 developers (varies by org maturity) |
| **Key Roles** | Manager, Staff/Principal, Senior, Mid-level, Junior SRE |
| **Hiring Profile** | T-shaped: broad systems knowledge + deep expertise in 1-2 areas |
| **Building from Scratch** | 4 phases: Foundation → Growth → Maturity → Excellence |
| **Service Tiering** | Tier 1 (dedicated SRE), Tier 2 (shared SRE), Tier 3 (advisory) |
| **Best Model** | Hybrid (hub-and-spoke) for large orgs; centralized for mid-size |

---

## Quick Revision Questions

1. **What are the four main SRE team models, and when is each appropriate?**
   > Centralized (mid-size orgs), Embedded (large orgs, critical services), Hybrid (large orgs, best of both), No Dedicated SRE (small orgs/startups).

2. **Why is 6-8 the minimum viable SRE team size?**
   > To sustain 24/7 on-call coverage with a 1-week-on/5-off rotation, plus buffer for engineering work and PTO.

3. **What does "T-shaped" mean in the context of SRE hiring?**
   > Broad knowledge across many systems domains (Linux, networking, cloud, containers) with deep expertise in one or two areas.

4. **How should services be tiered for SRE engagement?**
   > Tier 1 (business-critical) gets dedicated SRE, Tier 2 (important) gets shared SRE, Tier 3 (standard) gets advisory/consulting.

5. **Describe the four phases of building an SRE team from scratch.**
   > Foundation (hire core team, define SLOs, set up monitoring), Growth (expand team, implement error budgets), Maturity (PRRs, chaos engineering), Excellence (self-healing, reliability culture).

6. **What is the advantage of the Hybrid/Hub-and-Spoke model over purely centralized or embedded?**
   > It provides consistent standards, shared tooling, and career path (from the hub) while maintaining deep service context (from embedded spokes).

---

[← Previous: Google's SRE Model](04-googles-sre-model.md) | [Back to README](../README.md) | [Next: SLIs →](../02-Service-Level-Objectives/01-slis.md)
