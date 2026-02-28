# Chapter 1.4: Google's SRE Model

[← Previous: SRE Principles](03-sre-principles.md) | [Back to README](../README.md) | [Next: SRE Team Structure →](05-sre-team-structure.md)

---

## Overview

Google invented SRE in 2003 and has been refining the model for over two decades. Google's SRE organization manages some of the largest services on the planet — Search, Gmail, YouTube, Cloud — with remarkably small teams relative to system scale. Their model has become the gold standard for the industry.

---

## 1. Origin Story

```
  Timeline of SRE at Google
  
  2003 ──► Ben Treynor Sloss asked to run production team
           └── "What if we treat ops as a software problem?"
           
  2004-2010 ──► SRE grows across Google's major services
                └── Gmail, Search, Ads, YouTube
                
  2014 ──► First SREcon conference
  
  2016 ──► "Site Reliability Engineering" book published (the "SRE Bible")
  
  2018 ──► "The Site Reliability Workbook" published
  
  2020+ ──► SRE adopted industry-wide: Netflix, Spotify, LinkedIn, etc.
```

---

## 2. Google's SRE Organizational Model

```
  ┌─────────────────────────────────────────────────────────┐
  │                Google Engineering Org                     │
  │                                                         │
  │  ┌──────────────────┐       ┌──────────────────────┐   │
  │  │  Product Dev      │       │  SRE Organization    │   │
  │  │  (SWE Teams)      │       │                      │   │
  │  │                   │       │  VP: Ben Treynor     │   │
  │  │  • Build features │ ◄───► │                      │   │
  │  │  • Write code     │       │  • Reliability       │   │
  │  │  • Design systems │       │  • On-call           │   │
  │  │                   │       │  • Automation         │   │
  │  └──────────────────┘       │  • Capacity planning  │   │
  │                              │                      │   │
  │                              │  ~2000+ SREs         │   │
  │                              │  (as of 2020s)       │   │
  │                              └──────────────────────┘   │
  │                                                         │
  └─────────────────────────────────────────────────────────┘
```

### Key Organizational Facts

| Aspect | Detail |
|--------|--------|
| SRE as a separate org | SRE sits alongside product development, not under it |
| Reporting | SRE VP reports to SVP of Engineering |
| Hiring bar | SREs are hired to the same standard as software engineers |
| Transfer policy | SREs can transfer to SWE roles and vice versa |
| Team size | Typically 6-8 SREs per service (varies by criticality) |

---

## 3. The Social Contract: SRE ↔ Product Dev

Google's SRE model works because of a clear agreement between SRE and the product development teams:

```
  ┌──────────────────────────┐     ┌──────────────────────────┐
  │    Product Dev Agrees    │     │      SRE Agrees           │
  ├──────────────────────────┤     ├──────────────────────────┤
  │                          │     │                          │
  │  • Consult SRE on design │     │  • Own production for    │
  │  • Share on-call when    │     │    the service           │
  │    needed                │     │                          │
  │  • Accept SLO-driven     │     │  • Provide on-call       │
  │    release freezes       │     │    coverage              │
  │  • Fix reliability bugs  │     │  • Automate operational  │
  │    flagged by SRE        │     │    work                  │
  │  • Participate in        │     │  • Conduct and share     │
  │    postmortems           │     │    postmortems           │
  │                          │     │                          │
  └──────────────────────────┘     └──────────────────────────┘
  
  [!] CRITICAL: SRE can "give back the pager" if the product 
  team doesn't uphold their end of the contract.
```

### "Giving Back the Pager"

If a service consistently burns through error budgets and the product team doesn't prioritize reliability fixes:

1. SRE escalates to management
2. If unresolved, SRE can **hand production responsibility back** to the dev team
3. This forces the dev team to handle their own on-call
4. Typically motivates rapid reliability improvements

---

## 4. Google's Engagement Models

Google uses three distinct models for how SRE teams engage with services:

```
  ┌───────────────────────────────────────────────────────────┐
  │            THREE SRE ENGAGEMENT MODELS                    │
  ├───────────────────────────────────────────────────────────┤
  │                                                           │
  │  MODEL 1: Kitchen Sink / Embedded SRE                     │
  │  ┌─────────────────────────────────────────────────┐      │
  │  │  SRE team owns a set of related services        │      │
  │  │  Works closely alongside dev team               │      │
  │  │  Full production ownership                      │      │
  │  │  Best for: Critical, complex services           │      │
  │  └─────────────────────────────────────────────────┘      │
  │                                                           │
  │  MODEL 2: Infrastructure SRE                              │
  │  ┌─────────────────────────────────────────────────┐      │
  │  │  SRE team owns shared infrastructure            │      │
  │  │  (storage, networking, compute platforms)        │      │
  │  │  Used by many product teams                     │      │
  │  │  Best for: Foundational platform services       │      │
  │  └─────────────────────────────────────────────────┘      │
  │                                                           │
  │  MODEL 3: Consulting / Advisory SRE                       │
  │  ┌─────────────────────────────────────────────────┐      │
  │  │  SRE provides guidance but doesn't own prod     │      │
  │  │  Reviews designs, sets SLOs, trains dev teams   │      │
  │  │  Dev team retains on-call responsibility        │      │
  │  │  Best for: Less critical or newer services      │      │
  │  └─────────────────────────────────────────────────┘      │
  │                                                           │
  └───────────────────────────────────────────────────────────┘
```

---

## 5. The Production Readiness Review (PRR)

Before SRE takes on a new service, Google conducts a Production Readiness Review:

```
  ┌──────────────────────────────────────────────────────┐
  │           PRODUCTION READINESS REVIEW                 │
  │                                                      │
  │  Checklist:                                          │
  │                                                      │
  │  □ SLIs and SLOs defined                             │
  │  □ Monitoring and alerting configured                │
  │  □ Runbooks / playbooks written                      │
  │  □ Capacity planning done                            │
  │  □ Load tested to 2x expected peak                   │
  │  □ Failure modes documented                          │
  │  □ Rollback procedures tested                        │
  │  □ Data backup and recovery verified                 │
  │  □ Security review completed                         │
  │  □ Dependency map documented                         │
  │  □ On-call rotation set up                           │
  │  □ Postmortem process defined                        │
  │                                                      │
  │  Result: PASS → SRE takes on-call                    │
  │          FAIL → Dev team fixes gaps, re-reviews      │
  └──────────────────────────────────────────────────────┘
```

---

## 6. Google's Error Budget Policy

```
  ┌──────────────────────────────────────────────────────────┐
  │              ERROR BUDGET POLICY (Google-style)           │
  ├──────────────────────────────────────────────────────────┤
  │                                                          │
  │  Budget Status          │  Action                        │
  │  ──────────────         │  ──────────────────────────    │
  │  Budget > 50% remaining │  Normal development pace       │
  │                         │  Deploy at will                │
  │  ──────────────         │  ──────────────────────────    │
  │  Budget 20-50%          │  Caution mode                  │
  │                         │  More rigorous testing         │
  │                         │  Review risky changes          │
  │  ──────────────         │  ──────────────────────────    │
  │  Budget < 20%           │  Reduced velocity              │
  │                         │  Only reliability fixes        │
  │                         │  Extra review for all changes  │
  │  ──────────────         │  ──────────────────────────    │
  │  Budget Exhausted       │  FREEZE                        │
  │                         │  No feature launches           │
  │                         │  All hands on reliability      │
  │                         │  Postmortem required           │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Google's On-Call Model

```
  ┌──────────────────────────────────────────────────┐
  │          GOOGLE SRE ON-CALL MODEL                │
  │                                                  │
  │  Rotation: 1 week on, typically 1-2 weeks off    │
  │  Coverage: 12-hour shifts (follow-the-sun)       │
  │                                                  │
  │  ┌──────────────────────────────────────────┐   │
  │  │ Primary             Secondary             │   │
  │  │ On-Call  ──────────► On-Call              │   │
  │  │ (handles all        (backup,              │   │
  │  │  pages)             escalation)           │   │
  │  └──────────────────────────────────────────┘   │
  │                                                  │
  │  Rules:                                          │
  │  • Max 2 incidents per 12-hour shift             │
  │  • Time to acknowledge: 5 minutes               │
  │  • Post-incident: write postmortem within 48h    │
  │  • Compensated for on-call time                  │
  │  • At least 50% of on-call time is quiet         │
  │  • If consistently too busy → add headcount      │
  │                                                  │
  └──────────────────────────────────────────────────┘
```

---

## 8. Postmortem Culture at Google

Google's postmortem process is **blameless** and focused on systemic improvement:

```
  Postmortem Document Structure (Google Template):
  
  ┌─────────────────────────────────────────────┐
  │  1. Title & Date                            │
  │  2. Summary (1-2 paragraphs)                │
  │  3. Impact (users affected, duration, $)    │
  │  4. Timeline (minute-by-minute)             │
  │  5. Root Cause(s)                           │
  │  6. Contributing Factors                    │
  │  7. Resolution                              │
  │  8. Lessons Learned                         │
  │     • What went well                        │
  │     • What went wrong                       │
  │     • Where we got lucky                    │
  │  9. Action Items (with owners & due dates)  │
  │  10. Supporting data / graphs               │
  └─────────────────────────────────────────────┘
  
  [!] "Where we got lucky" is unique to Google's template
      and exposes hidden risks that didn't cause harm THIS time.
```

---

## 9. Key Google SRE Practices

### Practice Matrix

| Practice | Description | Tool/Method |
|----------|-------------|-------------|
| **Borg → Kubernetes** | Container orchestration | Large-scale cluster management |
| **Borgmon → Prometheus** | Time-series monitoring | SLI measurement & alerting |
| **Monarch** | Global monitoring at scale | Distributed metrics collection |
| **Spanner** | Globally consistent DB | TrueTime API for strong consistency |
| **Colossus/GFS** | Distributed file system | Foundation for BigTable, Spanner |
| **Chubby** | Distributed locking | Leader election, config storage |
| **Load Shedding** | Graceful degradation | Drop low-priority traffic under load |
| **CRE** | Customer Reliability Engineering | SRE for cloud customers |

### Google's Internal Tools → Open Source Equivalents

```
  Google Internal          Open Source Equivalent
  ──────────────           ──────────────────────
  Borg              ──►    Kubernetes
  Borgmon            ──►    Prometheus
  Stubby             ──►    gRPC
  Dapper             ──►    Jaeger / Zipkin
  GFS / Colossus     ──►    HDFS / Ceph
  MapReduce          ──►    Apache Spark
  Dremel             ──►    Apache Drill / Presto
  Chubby             ──►    Apache ZooKeeper / etcd
```

---

## 10. Adopting Google's Model Outside Google

### What to Adopt

```
  ┌──────────────────────────────────────────────────────┐
  │  UNIVERSALLY APPLICABLE:                             │
  │                                                      │
  │  ✓ SLIs / SLOs / Error Budgets                      │
  │  ✓ Blameless postmortems                            │
  │  ✓ 50% toil cap                                     │
  │  ✓ On-call best practices                           │
  │  ✓ Production readiness reviews                      │
  │  ✓ Automation-first mindset                          │
  │                                                      │
  │  ADAPT TO YOUR SCALE:                                │
  │                                                      │
  │  ~ Team structure (you may not need a separate org)  │
  │  ~ Engagement model (embedded may work better)       │
  │  ~ Tooling (use what fits, not what Google uses)     │
  │                                                      │
  │  PROBABLY DON'T NEED:                                │
  │                                                      │
  │  ✗ Building your own Borg/Spanner                   │
  │  ✗ Follow-the-sun (unless truly global)             │
  │  ✗ Thousands of SREs                                │
  └──────────────────────────────────────────────────────┘
```

---

## 11. Troubleshooting Tips

| Challenge | Guidance |
|-----------|----------|
| "We're not Google-scale" | You don't need to be. SLOs, error budgets, and blameless postmortems work at any scale |
| "We can't hire to Google standards" | Start with training existing staff. SRE mindset matters more than pedigree |
| "Our dev teams won't follow the social contract" | Start with voluntary engagement. Success stories sell better than mandates |
| "We don't have budget for separate SRE org" | Use embedded model — assign SRE responsibilities to senior engineers within dev teams |

---

## Summary Table

| Google SRE Element | Description | Adaptability |
|--------------------|-------------|-------------|
| **Separate SRE org** | SRE as peer to product dev | Adapt: embed or hybrid models work |
| **Social contract** | Mutual obligations between SRE and dev | Adopt as-is |
| **PRR** | Readiness gate before SRE takes over | Adopt as-is |
| **Error budget policy** | Graduated response to budget consumption | Adopt as-is |
| **On-call model** | 12h shifts, max 2 events/shift, compensated | Adapt to team size |
| **Blameless postmortems** | Focus on systems, not people | Adopt as-is |
| **"Give back the pager"** | Ultimate escalation if dev team doesn't cooperate | Adopt as-is |
| **CRE** | SRE for customer-facing reliability | Useful for SaaS companies |

---

## Quick Revision Questions

1. **What is the "social contract" between SRE and product development at Google?**
   > SRE agrees to own production and provide on-call; dev agrees to consult SRE on design, accept SLO-driven freezes, and fix reliability bugs. Either side can break the contract.

2. **What happens when SRE "gives back the pager"?**
   > The dev team takes over on-call and production operations, motivating them to investment in reliability to get SRE support back.

3. **Describe Google's three SRE engagement models.**
   > Embedded (full ownership of specific services), Infrastructure (owns shared platforms), Consulting (advises but doesn't own production).

4. **What is a Production Readiness Review (PRR)?**
   > A checklist-based gate that a service must pass before SRE takes on-call responsibility — covering SLOs, monitoring, runbooks, capacity, and more.

5. **What is unique about the "Where we got lucky" section in Google's postmortem template?**
   > It exposes hidden risks that didn't cause harm this time but could in the future — proactively addressing near-misses.

6. **Name three Google-internal tools and their open-source equivalents.**
   > Borg → Kubernetes, Borgmon → Prometheus, Stubby → gRPC.

---

[← Previous: SRE Principles](03-sre-principles.md) | [Back to README](../README.md) | [Next: SRE Team Structure →](05-sre-team-structure.md)
