# Chapter 6.4 — Team Topologies

[← Previous: Platform Engineer](03-platform-engineer.md) | [Next: Collaboration Models →](05-collaboration-models.md)

---

## Overview

**Team Topologies** is a framework by Matthew Skelton and Manuel Pais that provides a practical model for organizing teams for fast flow of software delivery. It defines **four fundamental team types** and **three interaction modes**, offering a clear vocabulary for designing team structures that minimize cognitive load and maximize delivery speed.

---

## The Four Team Types

```
┌──────────────────────────────────────────────────────────┐
│  FOUR FUNDAMENTAL TEAM TYPES                             │
│                                                          │
│  1. STREAM-ALIGNED TEAM                                  │
│  ┌────────────────────────────────┐                     │
│  │  Aligned to a flow of work     │  ← PRIMARY type     │
│  │  (feature, product, user       │     (most teams)    │
│  │   journey, or business domain) │                     │
│  │  End-to-end ownership          │                     │
│  └────────────────────────────────┘                     │
│                                                          │
│  2. PLATFORM TEAM                                        │
│  ┌────────────────────────────────┐                     │
│  │  Provides self-service APIs,   │  ← Enables stream   │
│  │  tools, and services that      │     teams to be     │
│  │  stream-aligned teams consume  │     autonomous      │
│  └────────────────────────────────┘                     │
│                                                          │
│  3. ENABLING TEAM                                        │
│  ┌────────────────────────────────┐                     │
│  │  Helps stream-aligned teams    │  ← Temporary        │
│  │  adopt new skills/technologies │     coaching role   │
│  │  Coaches, doesn't do the work  │                     │
│  └────────────────────────────────┘                     │
│                                                          │
│  4. COMPLICATED SUBSYSTEM TEAM                           │
│  ┌────────────────────────────────┐                     │
│  │  Owns complex component that   │  ← Rare, only when │
│  │  requires specialist knowledge │     truly needed    │
│  │  (ML model, video codec, etc.) │                     │
│  └────────────────────────────────┘                     │
└──────────────────────────────────────────────────────────┘
```

---

## Team Interaction Overview

```
┌──────────────────────────────────────────────────────────┐
│  HOW TEAMS INTERACT                                      │
│                                                          │
│          ENABLING TEAM                                   │
│          (e.g., DevOps coaching)                         │
│               │                                          │
│               │ facilitating                             │
│               ▼                                          │
│  ┌────────────────────────┐    x-as-a-service           │
│  │  STREAM-ALIGNED TEAM   │◄──────────────────┐        │
│  │  (e.g., Checkout team) │                    │        │
│  └────────────┬───────────┘                    │        │
│               │                          ┌─────┴──────┐ │
│               │ collaborating           │  PLATFORM   │ │
│               ▼                          │  TEAM       │ │
│  ┌────────────────────────┐             │  (e.g., K8s │ │
│  │  COMPLICATED SUBSYSTEM │             │   platform) │ │
│  │  (e.g., Payment engine)│             └────────────┘ │
│  └────────────────────────┘                             │
└──────────────────────────────────────────────────────────┘
```

---

## Three Interaction Modes

```
┌──────────────────────────────────────────────────────────┐
│  THREE TEAM INTERACTION MODES                            │
│                                                          │
│  1. COLLABORATION                                        │
│  ┌─────────┐  ┌─────────┐                               │
│  │ Team A  │◄►│ Team B  │  Two teams work closely       │
│  └─────────┘  └─────────┘  together on a shared goal.   │
│  Use: Discovery, innovation, new tech adoption.          │
│  Limit: Expensive — high communication overhead.         │
│  Duration: Temporary (weeks to months).                  │
│                                                          │
│  2. X-AS-A-SERVICE                                       │
│  ┌─────────┐     API     ┌──────────┐                   │
│  │ Team A  │────────────►│ Team B   │                   │
│  │(consumer)│             │(provider)│                   │
│  └─────────┘             └──────────┘                   │
│  Use: Clear boundary, well-defined service contract.     │
│  Benefit: Low coordination cost, high autonomy.          │
│  Duration: Long-term / ongoing.                          │
│                                                          │
│  3. FACILITATING                                         │
│  ┌──────────┐  coaching  ┌─────────┐                    │
│  │ Enabling │──────────►│ Team A  │                    │
│  │ Team     │            │         │                    │
│  └──────────┘            └─────────┘                    │
│  Use: Help teams learn new skills or technologies.       │
│  Key: Enabling team DOESN'T do the work for them.        │
│  Duration: Temporary (until team is self-sufficient).    │
└──────────────────────────────────────────────────────────┘
```

---

## Cognitive Load & Team Size

```
┌──────────────────────────────────────────────────────────┐
│  COGNITIVE LOAD — THE KEY CONSTRAINT                     │
│                                                          │
│  Three types of cognitive load:                          │
│                                                          │
│  INTRINSIC:    The core domain complexity                │
│                (business logic, algorithms)              │
│  ████████████ — Can't reduce, must manage               │
│                                                          │
│  EXTRANEOUS:   Unnecessary complexity                    │
│                (bad tools, manual processes, unclear docs)│
│  ████████████ — ELIMINATE with platforms & good tooling  │
│                                                          │
│  GERMANE:      Learning new but relevant skills          │
│                (new technology adoption)                 │
│  ████████████ — Manage with enabling teams               │
│                                                          │
│  TEAM SIZE GUIDANCE (Dunbar's Number applied):           │
│  ├── 5-9 people per team (ideal: 7±2)                   │
│  ├── Team owns what it can cognitively handle            │
│  ├── If team is overwhelmed → split domain or            │
│  │   offload to platform team                           │
│  └── Conway's Law: Architecture mirrors team structure   │
│                                                          │
│  "Team cognitive load is the human limitation that       │
│   determines how complex a system can be."               │
└──────────────────────────────────────────────────────────┘
```

---

## Conway's Law

```
┌──────────────────────────────────────────────────────────┐
│  CONWAY'S LAW & THE REVERSE CONWAY MANEUVER             │
│                                                          │
│  CONWAY'S LAW (1967):                                    │
│  "Organizations design systems that mirror their         │
│   communication structures."                             │
│                                                          │
│  TRADITIONAL ORG:              RESULTING ARCHITECTURE:   │
│  ┌──────────┐                  ┌────────────────┐       │
│  │ Frontend │                  │   Monolith     │       │
│  │   Team   │──┐               │ ┌────┬────┬──┐ │       │
│  ├──────────┤  │               │ │ FE │ BE │DB│ │       │
│  │ Backend  │──┼── tightly ──► │ └────┴────┴──┘ │       │
│  │   Team   │  │   coupled     │   (tightly     │       │
│  ├──────────┤  │               │    coupled)    │       │
│  │ Database │──┘               └────────────────┘       │
│  │   Team   │                                           │
│  └──────────┘                                           │
│                                                          │
│  REVERSE CONWAY MANEUVER:                                │
│  Design your team structure to get the architecture       │
│  you WANT, not the one that emerges accidentally.        │
│                                                          │
│  DESIRED ORG:                  RESULTING ARCHITECTURE:   │
│  ┌──────────────┐              ┌──────┐ ┌──────┐       │
│  │ Checkout Team│              │Orders│ │Payments│      │
│  │ (full-stack) │──────────►  │ API  │ │  API  │       │
│  ├──────────────┤              └──────┘ └──────┘       │
│  │ Payments Team│              ┌──────┐ ┌──────┐       │
│  │ (full-stack) │──────────►  │Search│ │ User │       │
│  ├──────────────┤              │ API  │ │  API │       │
│  │ Search Team  │              └──────┘ └──────┘       │
│  │ (full-stack) │               (loosely coupled        │
│  └──────────────┘                microservices)         │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Applying Team Topologies

```
SCENARIO: E-commerce company (150 engineers) migrating
from monolith to microservices

BEFORE (Traditional Structure):
├── Frontend Team (12 people)
├── Backend Team (20 people)
├── Database Team (5 people)
├── QA Team (10 people)
├── Ops Team (8 people)
└── Problems:
    ├── Handoffs everywhere → slow delivery
    ├── Ops bottleneck → 2-week deploy queue
    ├── QA bottleneck → 3-week testing cycles
    └── Everyone blocked by everyone else

AFTER (Team Topologies Applied):

STREAM-ALIGNED TEAMS (the core):
├── Checkout Team (7 people, full-stack)
│   Owns: Cart, payment, order confirmation
├── Search & Discovery Team (6 people)
│   Owns: Search, recommendations, catalog browse
├── User Account Team (5 people)
│   Owns: Auth, profile, preferences, notifications
└── Fulfillment Team (6 people)
    Owns: Inventory, shipping, tracking

PLATFORM TEAM (1):
└── Developer Experience Team (5 people)
    Provides: K8s platform, CI/CD templates,
    observability stack, service scaffolding
    Interaction: X-as-a-service

ENABLING TEAM (1):
└── Cloud Native Coaching Team (3 people)
    Helps: Teams adopt K8s, microservices patterns,
    distributed tracing
    Interaction: Facilitating (rotates between teams)

COMPLICATED SUBSYSTEM TEAM (1):
└── Payment Processing Team (4 people)
    Owns: PCI-compliant payment gateway integration
    (specialist knowledge required)
    Interaction: X-as-a-service to Checkout team

RESULT:
├── Lead time: 3 weeks → 2 days
├── Deploy frequency: Monthly → Multiple/day
├── Team autonomy: Each team deploys independently
└── Cognitive load: Manageable per team
```

---

## Team Topology Anti-Patterns

```
┌──────────────────────────────────────────────────────────┐
│  ANTI-PATTERNS TO AVOID                                  │
│                                                          │
│  ❌ "DevOps Team" as middleman                           │
│     Dev → DevOps → Ops (just added another silo)        │
│     ✅ Instead: Platform team providing self-service     │
│                                                          │
│  ❌ Shared services team that owns everything            │
│     One team owns auth, logging, search, config, ...    │
│     ✅ Instead: Stream-aligned teams own their domains   │
│                                                          │
│  ❌ Component teams instead of feature teams             │
│     Frontend team, API team, DB team → handoff hell     │
│     ✅ Instead: Cross-functional stream-aligned teams    │
│                                                          │
│  ❌ Enabling team that never leaves                      │
│     "Coaching" team that does the work forever           │
│     ✅ Instead: Time-boxed engagement, then move on      │
│                                                          │
│  ❌ Too many interaction modes at once                   │
│     Same two teams collaborating AND x-as-a-service     │
│     ✅ Instead: One clear interaction mode at a time     │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Stream team can't deploy independently | Shared database or tightly coupled services | Apply domain-driven design; decouple with APIs/events |
| Platform team is bottleneck | Not enough self-service | Build self-service APIs; Golden paths over ticket queues |
| Enabling team stays too long | No exit criteria | Time-box engagements (2-3 months); define success criteria |
| Teams are too large | Growing without splitting | Keep 5-9 people; split by domain when cognitive load is high |
| Architecture doesn't match teams | Conway's Law working against you | Apply Reverse Conway Maneuver: restructure teams first |
| Too many dependencies between teams | Wrong team boundaries | Redraw boundaries along domain lines (DDD bounded contexts) |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Team Topologies** | Framework by Skelton & Pais for organizing teams for fast flow |
| **Stream-Aligned Team** | Primary team type; aligned to a flow of business value |
| **Platform Team** | Provides self-service capabilities to stream-aligned teams |
| **Enabling Team** | Coaches teams on new skills; temporary engagement |
| **Complicated Subsystem** | Owns specialist components; rare and only when needed |
| **Collaboration Mode** | Two teams working closely; high cost, for discovery |
| **X-as-a-Service Mode** | Clear API boundary; low coordination cost |
| **Facilitating Mode** | Coaching relationship; enabling team helps others learn |
| **Conway's Law** | Architecture mirrors communication structure |
| **Cognitive Load** | The human constraint that limits team scope |

---

## Quick Revision Questions

1. **What are the four fundamental team types in Team Topologies?**
   <details><summary>Answer</summary>1) Stream-Aligned Team — the primary type, aligned to a flow of work (feature, product, user journey). 2) Platform Team — provides self-service APIs and tools for stream-aligned teams. 3) Enabling Team — helps teams adopt new skills/technologies through coaching (temporary). 4) Complicated Subsystem Team — owns complex components requiring specialist knowledge (rare). Most teams should be stream-aligned.</details>

2. **What are the three interaction modes and when should each be used?**
   <details><summary>Answer</summary>1) Collaboration — two teams work closely together; use for discovery, innovation, new tech adoption; expensive, keep temporary. 2) X-as-a-Service — clear API boundary between provider and consumer; use for well-defined services; low coordination cost, long-term. 3) Facilitating — enabling team coaches another team; use when teams need to learn new skills; temporary until team is self-sufficient.</details>

3. **What is Conway's Law and the Reverse Conway Maneuver?**
   <details><summary>Answer</summary>Conway's Law: "Organizations design systems that mirror their communication structures." If you have a frontend team, backend team, and DB team, you'll get a three-tier architecture whether you want it or not. The Reverse Conway Maneuver: deliberately design your team structure to get the architecture you want. Want microservices? Create cross-functional teams aligned to business domains.</details>

4. **Why is cognitive load important in team design?**
   <details><summary>Answer</summary>Cognitive load determines how much complexity a team can handle effectively. Three types: Intrinsic (core domain — can't reduce), Extraneous (unnecessary complexity — eliminate with platforms), Germane (learning new relevant skills — manage with enabling teams). If a team's cognitive load is too high, quality drops, velocity slows, and people burn out. Team Topologies uses cognitive load as the primary constraint for team sizing and scope.</details>

5. **What is a stream-aligned team and why is it the primary type?**
   <details><summary>Answer</summary>A stream-aligned team is aligned to a single flow of work — a product, feature set, user journey, or business domain. It's the primary type because: 1) It has end-to-end ownership (build, deploy, support). 2) It minimizes handoffs. 3) It aligns to business value. 4) All other team types exist to support stream-aligned teams. Most teams in an organization should be stream-aligned; platform, enabling, and complicated subsystem teams exist to reduce their cognitive load.</details>

6. **What are common Team Topologies anti-patterns?**
   <details><summary>Answer</summary>1) "DevOps team" as middleman (creates a new silo). 2) Shared services team owning too many unrelated things. 3) Component teams (frontend/backend/DB) instead of feature teams. 4) Enabling team that never disengages (becomes a crutch). 5) Multiple interaction modes between the same two teams simultaneously. 6) Teams too large (>9 people) without splitting. Each anti-pattern increases handoffs, dependencies, or cognitive load.</details>

---

[← Previous: Platform Engineer](03-platform-engineer.md) | [Next: Collaboration Models →](05-collaboration-models.md)
