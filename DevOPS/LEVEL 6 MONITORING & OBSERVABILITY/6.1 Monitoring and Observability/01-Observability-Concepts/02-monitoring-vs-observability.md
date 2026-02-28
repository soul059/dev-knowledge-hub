# Chapter 2: Monitoring vs Observability

[← Previous: What is Observability?](01-what-is-observability.md) | [Back to README](../README.md) | [Next: Three Pillars →](03-three-pillars.md)

---

## Overview

Monitoring and observability are often used interchangeably, but they represent fundamentally different philosophies. Monitoring is a **subset** of observability. This chapter clarifies the distinction and shows when each approach is appropriate.

---

## 2.1 Monitoring Defined

> **Monitoring** = Collecting, processing, aggregating, and displaying real-time quantitative data about a system — CPU, memory, disk, request counts, error rates, etc.

Monitoring answers: **"Is something wrong?"**

```
┌─────────────────────────────────────────────┐
│               MONITORING                     │
│                                              │
│   "I know what can go wrong,                │
│    so I set up checks for those things."    │
│                                              │
│   ┌──────────┐    ┌──────────┐              │
│   │  Define  │───►│  Check   │              │
│   │  Checks  │    │  Status  │              │
│   └──────────┘    └────┬─────┘              │
│                        │                     │
│                   ┌────▼─────┐              │
│                   │  Alert   │              │
│                   │  if Bad  │              │
│                   └──────────┘              │
│                                              │
│   Examples:                                  │
│   • CPU > 90%  → Alert                      │
│   • Disk > 85% → Alert                      │
│   • HTTP 5xx > 1% → Alert                  │
└─────────────────────────────────────────────┘
```

---

## 2.2 Observability Defined

> **Observability** = The ability to understand the internal state of a system by examining its external outputs, enabling you to ask **new, unanticipated questions**.

Observability answers: **"Why is something wrong?"** and **"What else is affected?"**

```
┌─────────────────────────────────────────────┐
│             OBSERVABILITY                    │
│                                              │
│   "I don't know what will go wrong,         │
│    so I instrument everything to ask         │
│    any question later."                      │
│                                              │
│   ┌──────────┐    ┌──────────┐              │
│   │Instrument│───►│ Collect  │              │
│   │Everything│    │  All     │              │
│   └──────────┘    │ Signals  │              │
│                   └────┬─────┘              │
│                        │                     │
│                   ┌────▼─────┐              │
│                   │  Explore │              │
│                   │  Query   │              │
│                   │  Analyze │              │
│                   └──────────┘              │
│                                              │
│   Examples:                                  │
│   • Why is p99 latency high for EU users?   │
│   • Which deployment caused the regression? │
│   • What's the blast radius of this failure?│
└─────────────────────────────────────────────┘
```

---

## 2.3 Key Differences

```
┌────────────────────┬────────────────────────┬────────────────────────┐
│     Aspect         │     MONITORING         │    OBSERVABILITY       │
├────────────────────┼────────────────────────┼────────────────────────┤
│ Philosophy         │ Check known failure    │ Explore unknown        │
│                    │ modes                  │ failure modes          │
├────────────────────┼────────────────────────┼────────────────────────┤
│ Questions          │ Pre-defined            │ Ad-hoc, arbitrary      │
│                    │ (known-unknowns)       │ (unknown-unknowns)     │
├────────────────────┼────────────────────────┼────────────────────────┤
│ Data               │ Aggregated metrics     │ High-cardinality,      │
│                    │                        │ high-dimensionality    │
├────────────────────┼────────────────────────┼────────────────────────┤
│ Approach           │ Dashboards & alerts    │ Exploration & querying │
├────────────────────┼────────────────────────┼────────────────────────┤
│ Tells you          │ WHAT is broken         │ WHY it is broken       │
├────────────────────┼────────────────────────┼────────────────────────┤
│ Setup              │ Easier, well-defined   │ Requires investment in │
│                    │                        │ instrumentation        │
├────────────────────┼────────────────────────┼────────────────────────┤
│ Scalability        │ Works for simple       │ Essential for complex, │
│                    │ systems                │ distributed systems    │
├────────────────────┼────────────────────────┼────────────────────────┤
│ Relationship       │ Subset of              │ Superset of            │
│                    │ observability          │ monitoring             │
└────────────────────┴────────────────────────┴────────────────────────┘
```

---

## 2.4 The Spectrum

Monitoring and observability exist on a spectrum, not as binary choices:

```
Pure Monitoring ◄──────────────────────────────────► Full Observability

  "Is it up?"      "How fast?"      "Why slow?"      "What changed?"
       │                │                │                  │
  ┌────▼────┐     ┌─────▼────┐    ┌─────▼─────┐    ┌──────▼──────┐
  │ Health  │     │ Metrics  │    │ Metrics + │    │ Metrics +   │
  │ Checks  │     │   +      │    │ Logs +    │    │ Logs +      │
  │ Ping    │     │ Alerts   │    │ Basic     │    │ Traces +    │
  │         │     │          │    │ Traces    │    │ Profiling + │
  │         │     │          │    │           │    │ Ad-hoc      │
  │         │     │          │    │           │    │ Querying    │
  └─────────┘     └──────────┘    └───────────┘    └─────────────┘

  Level 0          Level 1          Level 2           Level 3
  Reactive         Proactive        Investigative     Explorative
```

---

## 2.5 Known-Unknowns vs Unknown-Unknowns

### Known-Unknowns (Monitoring)
Things you **know** can go wrong, but don't know **when**:
- Server runs out of disk
- Database connections exhausted
- Memory leak crashes service
- Certificate expires

### Unknown-Unknowns (Observability)
Things you **didn't predict** would go wrong:
- Specific user in specific region on specific browser sees errors
- Cascading failure from service C's retry storm
- Race condition that only appears under exact load pattern
- Performance regression caused by a dependency's dependency

```
              ┌───────────────────────────┐
              │      What you know        │
              │     ┌─────────────┐       │
              │     │   Known-    │       │
              │     │   Knowns    │       │  ◄── Dashboards
              │     │(we monitor) │       │
              │     └─────────────┘       │
              │                           │
              │     ┌─────────────┐       │
              │     │   Known-    │       │
              │     │  Unknowns   │       │  ◄── Alerts (Monitoring)
              │     │ (we alert)  │       │
              │     └─────────────┘       │
              │                           │
              │     ┌─────────────┐       │
              │     │  Unknown-   │       │
              │     │  Unknowns   │       │  ◄── Observability
              │     │ (we explore)│       │
              │     └─────────────┘       │
              └───────────────────────────┘
```

---

## 2.6 When Monitoring is Enough

Monitoring alone can work when:

| Condition | Example |
|-----------|---------|
| Simple architecture | Single monolith with 1 database |
| Predictable failures | Disk fills up, OOM kills |
| Low traffic variance | Internal tools, batch processing |
| Stable codebase | Releases are infrequent |
| Small team | 1-2 people manage everything |

---

## 2.7 When You Need Observability

Observability becomes essential when:

| Condition | Example |
|-----------|---------|
| Microservices | 50+ services communicating via APIs/messages |
| Frequent deployments | Multiple deploys per day |
| Multi-cloud/hybrid | Services across AWS, GCP, on-prem |
| High traffic variance | Flash sales, viral content |
| Complex failures | Cascading, intermittent, partial failures |
| Large teams | Many teams owning different services |

---

## 2.8 Complementary, Not Competing

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│   You need BOTH monitoring AND observability.          │
│                                                        │
│   Monitoring = the ALERTING layer                     │
│   "Wake me up at 3 AM when the error rate > 5%"      │
│                                                        │
│   Observability = the INVESTIGATION layer             │
│   "OK I'm awake, now WHY is the error rate > 5%?"    │
│                                                        │
│   ┌──────────┐                                        │
│   │ Monitor  │──► Alert fires                         │
│   └──────────┘         │                              │
│                        ▼                              │
│                   ┌──────────┐                        │
│                   │ Observe  │──► Investigate          │
│                   └──────────┘         │              │
│                                        ▼              │
│                                   ┌──────────┐        │
│                                   │   Fix    │        │
│                                   └──────────┘        │
└────────────────────────────────────────────────────────┘
```

---

## 2.9 Real-World Scenario

🌍 **Scenario: API Latency Spike**

```
3:00 AM — Monitoring alert: "API p95 latency > 500ms"

With Only Monitoring:
├── Dashboard shows latency spike on api-gateway
├── All servers healthy: CPU 40%, Memory 60%
├── DB metrics look fine
├── Try restarting api-gateway... no improvement
├── Try scaling api-gateway... no improvement
├── Call more engineers...
├── Someone notices payment-svc logs show "connection pool exhausted"
├── But WHY is the connection pool exhausted?
├── Hours of log searching...
└── Resolution: 3 hours

With Observability:
├── Alert links to dashboard showing p95 latency spike
├── Distributed trace shows: api-gateway → auth-svc → payment-svc (SLOW)
├── payment-svc trace shows: each request opens 5 DB connections (was 1)
├── Correlated log: "Using new ORM version after deploy at 2:45 AM"
├── Git diff: ORM upgrade changed connection pooling defaults
├── Fix: Revert ORM config, add connection pool settings
└── Resolution: 15 minutes
```

---

## 2.10 Practical Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TOOL MAPPING                                      │
├──────────────────┬───────────────────┬──────────────────────────────┤
│    Activity      │    Monitoring     │    Observability             │
├──────────────────┼───────────────────┼──────────────────────────────┤
│ Infrastructure   │ Nagios, Zabbix,   │ Same tools +                │
│ health           │ CloudWatch        │ deeper analysis             │
├──────────────────┼───────────────────┼──────────────────────────────┤
│ Application      │ Uptime checks,    │ Distributed tracing         │
│ behavior         │ error rate alerts │ (Jaeger, Zipkin)            │
├──────────────────┼───────────────────┼──────────────────────────────┤
│ User experience  │ Synthetic tests   │ Real User Monitoring        │
│                  │                   │ (RUM), session replay       │
├──────────────────┼───────────────────┼──────────────────────────────┤
│ Root cause       │ Runbooks with     │ Ad-hoc queries across       │
│ analysis         │ known solutions   │ all telemetry data          │
├──────────────────┼───────────────────┼──────────────────────────────┤
│ Capacity         │ Threshold alerts  │ Predictive analytics,       │
│ planning         │ on resource usage │ trend analysis              │
└──────────────────┴───────────────────┴──────────────────────────────┘
```

---

## Summary Table

| Aspect | Monitoring | Observability |
|--------|-----------|---------------|
| **Purpose** | Detect known problems | Understand any system behavior |
| **Questions** | Pre-defined | Ad-hoc, exploratory |
| **Data** | Aggregated metrics | Rich, high-cardinality telemetry |
| **Tells you** | WHAT is broken | WHY it's broken |
| **Approach** | Dashboards + alerts | Exploration + correlation |
| **Best for** | Simple, predictable systems | Complex, distributed systems |
| **Relationship** | Subset of observability | Superset of monitoring |

---

## Quick Revision Questions

1. **Explain the relationship between monitoring and observability. Why is monitoring considered a subset of observability?**

2. **What are "known-unknowns" and "unknown-unknowns"? Which does monitoring address, and which does observability address?**

3. **Give a real-world example where monitoring alone would be insufficient to resolve an incident.**

4. **At what point should an organization invest in observability beyond basic monitoring? List three indicators.**

5. **How do monitoring and observability complement each other during an incident? Describe the typical workflow.**

6. **Compare the data requirements of monitoring vs. observability. Why does observability need high-cardinality data?**

---

[← Previous: What is Observability?](01-what-is-observability.md) | [Back to README](../README.md) | [Next: Three Pillars →](03-three-pillars.md)
