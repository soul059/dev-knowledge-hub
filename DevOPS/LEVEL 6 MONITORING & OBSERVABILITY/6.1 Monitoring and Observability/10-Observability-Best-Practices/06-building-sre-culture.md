# Chapter 6: Building SRE Culture

[← Previous: Observability as Code](05-observability-as-code.md) | [Back to README](../README.md)

---

## Overview

Tools and technology are only part of observability. Building an SRE (Site Reliability Engineering) culture means embedding reliability practices into how teams work — defining SLOs, running blameless postmortems, reducing toil, and continuously improving. This chapter covers the organizational and cultural practices that make observability effective.

---

## 6.1 SRE Foundations

```
┌──────────────────────────────────────────────────────────┐
│  SRE CULTURE PILLARS                                     │
│                                                          │
│  ┌───────────────┐  ┌───────────────┐  ┌─────────────┐ │
│  │  RELIABILITY   │  │ OBSERVABILITY │  │   CULTURE   │ │
│  │  AS FEATURE    │  │  AS PRACTICE  │  │  AS HABIT   │ │
│  │               │  │               │  │             │ │
│  │ • SLOs define │  │ • Instrument  │  │ • Blameless │ │
│  │   "reliable   │  │   everything  │  │ • Share     │ │
│  │   enough"     │  │ • Dashboards  │  │   knowledge │ │
│  │ • Error       │  │   for every   │  │ • Reduce    │ │
│  │   budgets     │  │   service     │  │   toil      │ │
│  │   drive       │  │ • Alerts tied │  │ • Learn     │ │
│  │   decisions   │  │   to SLOs     │  │   from      │ │
│  │               │  │               │  │   failure   │ │
│  └───────┬───────┘  └───────┬───────┘  └──────┬──────┘ │
│          │                  │                  │         │
│          └──────────────────┼──────────────────┘         │
│                             ▼                            │
│              ┌─────────────────────────┐                │
│              │  CONTINUOUS IMPROVEMENT  │                │
│              │  Measure → Learn → Act  │                │
│              └─────────────────────────┘                │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 SLO-Driven Development

### Defining SLOs for a Service

```
┌──────────────────────────────────────────────────────────┐
│  SLO DEFINITION WORKFLOW                                 │
│                                                          │
│  Step 1: Identify User Journeys                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │ "As a user, I can complete checkout in <2s"     │    │
│  │ "As a user, I can search products successfully" │    │
│  │ "As a user, I can view my order history"        │    │
│  └─────────────────────────────────────────────────┘    │
│                    ▼                                     │
│  Step 2: Define SLIs (measurable indicators)            │
│  ┌─────────────────────────────────────────────────┐    │
│  │ Availability: % of successful requests          │    │
│  │ Latency: % of requests < threshold              │    │
│  │ Correctness: % of responses with valid data     │    │
│  └─────────────────────────────────────────────────┘    │
│                    ▼                                     │
│  Step 3: Set Targets + Error Budgets                    │
│  ┌─────────────────────────────────────────────────┐    │
│  │ Availability SLO: 99.9% (30d rolling)           │    │
│  │ Error Budget: 0.1% × 30d = 43.2 minutes        │    │
│  │ Latency SLO: 95% of requests < 500ms           │    │
│  └─────────────────────────────────────────────────┘    │
│                    ▼                                     │
│  Step 4: Implement Monitoring + Alerts                  │
│  ┌─────────────────────────────────────────────────┐    │
│  │ Burn rate alerts (multi-window)                  │    │
│  │ SLO dashboard per service                        │    │
│  │ Error budget tracking                            │    │
│  └─────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

### SLO Document Template

```yaml
# slo/checkout-service.yaml
service: checkout-service
owner: team-payments
review_cadence: monthly

slos:
  - name: Checkout Availability
    description: "Users can successfully complete checkout"
    sli:
      type: availability
      metric: |
        sum(rate(http_requests_total{service="checkout",status_code!~"5.."}[30d]))
        / sum(rate(http_requests_total{service="checkout"}[30d]))
    target: 99.9
    window: 30d
    error_budget_minutes: 43.2
    consequences:
      budget_exhausted:
        - "Freeze feature releases"
        - "Dedicate sprint to reliability"
      budget_healthy:
        - "Normal feature velocity"

  - name: Checkout Latency
    description: "Checkout completes within acceptable time"
    sli:
      type: latency
      metric: |
        histogram_quantile(0.95,
          sum(rate(http_request_duration_seconds_bucket{service="checkout"}[30d]))
          by (le)
        )
      good_threshold: "< 2s" 
    target: 95.0
    window: 30d
```

---

## 6.3 Error Budget Policy

```
┌──────────────────────────────────────────────────────────┐
│  ERROR BUDGET DECISION FRAMEWORK                         │
│                                                          │
│  Budget Remaining     │  Actions                         │
│  ─────────────────────┼────────────────────────────────  │
│                       │                                  │
│  > 50% remaining      │  ✅ Ship features normally       │
│                       │  ✅ Try risky deployments        │
│                       │  ✅ Experiment with new infra    │
│                       │                                  │
│  25-50% remaining     │  ⚠️  Increased caution           │
│                       │  ⚠️  Prioritize reliability work │
│                       │  ⚠️  Review recent incidents     │
│                       │                                  │
│  < 25% remaining      │  🛑 Feature freeze               │
│                       │  🛑 All hands on reliability     │
│                       │  🛑 Mandatory postmortem review  │
│                       │                                  │
│  0% (exhausted)       │  🚫 FULL STOP on changes         │
│                       │  🚫 Rollback recent changes      │
│                       │  🚫 Exec review required         │
└──────────────────────────────────────────────────────────┘
```

---

## 6.4 Blameless Postmortem Practice

### Postmortem Process

```
┌──────────────────────────────────────────────────────────┐
│  POSTMORTEM LIFECYCLE                                    │
│                                                          │
│  INCIDENT RESOLVED                                       │
│       │                                                  │
│       ▼  (within 48 hours)                              │
│  ┌──────────────────────┐                               │
│  │ Draft Postmortem     │  Owner: Incident Commander    │
│  │ • Timeline           │                               │
│  │ • Impact metrics     │                               │
│  │ • Root cause(s)      │                               │
│  └──────────┬───────────┘                               │
│             ▼  (within 5 business days)                  │
│  ┌──────────────────────┐                               │
│  │ Review Meeting       │  Attendees: involved team +   │
│  │ • Walk through       │  stakeholders                 │
│  │ • Identify actions   │  Tone: blameless, curious     │
│  │ • Assign owners      │                               │
│  └──────────┬───────────┘                               │
│             ▼                                            │
│  ┌──────────────────────┐                               │
│  │ Action Items         │  Track in issue tracker       │
│  │ • P1: fix within 1wk│  (JIRA, Linear, GitHub)       │
│  │ • P2: fix within 1mo│                               │
│  └──────────┬───────────┘                               │
│             ▼  (monthly)                                 │
│  ┌──────────────────────┐                               │
│  │ Follow-up Review     │  Are action items done?       │
│  │ • Track completion   │  Did the fix work?            │
│  │ • Share learnings    │  Publish to team/org          │
│  └──────────────────────┘                               │
└──────────────────────────────────────────────────────────┘
```

### Key Principles

```
┌──────────────────────────────────────────────────────────┐
│  BLAMELESS CULTURE PRINCIPLES                            │
│                                                          │
│  DO:                               DON'T:               │
│  ─────────────────────             ─────────────────     │
│  "The deploy process allowed       "John broke prod"    │
│   this change to reach prod"                             │
│                                                          │
│  "Our canary testing didn't        "QA should have      │
│   cover this scenario"              caught this"         │
│                                                          │
│  "The runbook was outdated"        "The on-call didn't  │
│                                     follow the runbook"  │
│                                                          │
│  Focus on SYSTEMS and PROCESSES,                        │
│  not PEOPLE.                                             │
│                                                          │
│  Ask "How can we make the system    Not "Who is          │
│  safer?" instead of                 responsible?"        │
└──────────────────────────────────────────────────────────┘
```

---

## 6.5 Reducing Toil

```
┌──────────────────────────────────────────────────────────┐
│  TOIL IDENTIFICATION AND REDUCTION                       │
│                                                          │
│  Toil = manual, repetitive, automatable work that       │
│  scales linearly with service growth and has no          │
│  lasting value.                                          │
│                                                          │
│  COMMON TOIL IN OBSERVABILITY:                          │
│                                                          │
│  Toil Activity         │ Automation                     │
│  ──────────────────────┼──────────────────────────────  │
│  Create dashboards     │ Templates + provisioning       │
│   for new services     │ (auto-generate from SLO def)   │
│                        │                                 │
│  Manually silence      │ Maintenance window API         │
│   alerts for deploys   │ integrated with CI/CD          │
│                        │                                 │
│  Investigate false     │ Tune alerts, add context,      │
│   alerts               │ link to runbooks               │
│                        │                                 │
│  Rotate log files      │ Centralized log pipeline       │
│                        │                                 │
│  Check service health  │ Synthetic monitoring +         │
│   manually             │ SLO dashboards                 │
│                        │                                 │
│  Manually scale        │ Autoscaling based on metrics   │
│   infrastructure       │                                 │
│                        │                                 │
│  TARGET: <50% of SRE time on toil                       │
│  GOAL:   Automate away repeating operational tasks      │
└──────────────────────────────────────────────────────────┘
```

---

## 6.6 Team Structure & Ownership

```
┌──────────────────────────────────────────────────────────┐
│  OBSERVABILITY OWNERSHIP MODEL                           │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │  PLATFORM / SRE TEAM                             │   │
│  │  Owns:                                           │   │
│  │  • Observability platform (Prometheus, Grafana,  │   │
│  │    Loki, Tempo, OTel Collector)                  │   │
│  │  • Shared dashboards/templates                   │   │
│  │  • Pipeline infrastructure                       │   │
│  │  • Standards and guidelines                      │   │
│  │  • Training and enablement                       │   │
│  └──────────────────────────────────────────────────┘   │
│                      ▲                                   │
│                      │ supports                         │
│                      ▼                                   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐      │
│  │  Team A     │ │  Team B     │ │  Team C     │      │
│  │             │ │             │ │             │      │
│  │ Owns:       │ │ Owns:       │ │ Owns:       │      │
│  │ • Service   │ │ • Service   │ │ • Service   │      │
│  │   SLOs      │ │   SLOs      │ │   SLOs      │      │
│  │ • Service   │ │ • Service   │ │ • Service   │      │
│  │   dashboards│ │   dashboards│ │   dashboards│      │
│  │ • Service   │ │ • Service   │ │ • Service   │      │
│  │   alerts    │ │   alerts    │ │   alerts    │      │
│  │ • On-call   │ │ • On-call   │ │ • On-call   │      │
│  └─────────────┘ └─────────────┘ └─────────────┘      │
│                                                          │
│  Principle: "You build it, you run it, you observe it" │
└──────────────────────────────────────────────────────────┘
```

---

## 6.7 Maturity Roadmap

```
┌──────────────────────────────────────────────────────────────┐
│  OBSERVABILITY MATURITY JOURNEY                              │
│                                                              │
│  Phase 1: REACTIVE (Months 0-3)                             │
│  ├── Basic metrics + uptime monitoring                      │
│  ├── Centralized logging                                    │
│  ├── Manual incident response                               │
│  └── Goal: "We know when things are down"                   │
│                                                              │
│  Phase 2: PROACTIVE (Months 3-6)                            │
│  ├── SLOs defined for critical services                     │
│  ├── Alerting tied to SLOs (burn rate)                      │
│  ├── Distributed tracing for key flows                      │
│  ├── Structured logging                                     │
│  └── Goal: "We catch issues before users do"                │
│                                                              │
│  Phase 3: SYSTEMATIC (Months 6-12)                          │
│  ├── Observability as Code (all config in Git)              │
│  ├── Automated incident response                            │
│  ├── Full correlation (metrics↔logs↔traces)                │
│  ├── Blameless postmortems as standard practice             │
│  └── Goal: "We learn from every incident"                   │
│                                                              │
│  Phase 4: OPTIMIZED (Months 12+)                            │
│  ├── Cost-optimized pipelines                               │
│  ├── Self-service observability for dev teams               │
│  ├── Predictive alerting (ML-assisted)                      │
│  ├── Chaos engineering informed by observability            │
│  └── Goal: "Observability drives engineering decisions"     │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.8 DORA Metrics & Continuous Improvement

```
┌──────────────────────────────────────────────────────────┐
│  DORA METRICS (measure engineering effectiveness)        │
│                                                          │
│  Metric                    │ Elite       │ Low           │
│  ──────────────────────────┼─────────────┼─────────────  │
│  Deployment Frequency      │ Multi/day   │ 1/month      │
│  Lead Time for Changes     │ < 1 hour    │ > 6 months   │
│  Change Failure Rate       │ < 5%        │ > 45%        │
│  Mean Time to Recovery     │ < 1 hour    │ > 1 week     │
│                                                          │
│  HOW OBSERVABILITY IMPROVES DORA:                       │
│                                                          │
│  • Faster MTTR: good dashboards + runbooks              │
│  • Lower failure rate: canary deploys + SLO gates       │
│  • Faster lead time: confidence to ship (SLO budget)    │
│  • Higher frequency: safe to deploy (good rollback)     │
└──────────────────────────────────────────────────────────┘
```

### Tracking DORA with Observability

```yaml
# Prometheus metrics from CI/CD pipeline
# Deployment frequency
deployment_total{service="api", environment="production"}

# Lead time (seconds from commit to production)
deployment_lead_time_seconds{service="api"}

# Change failure rate
deployment_rollback_total{service="api"} / deployment_total{service="api"}

# MTTR (from incident tracker integration)
incident_resolution_duration_seconds{service="api", severity="critical"}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Teams don't own SLOs | No clear ownership model | Assign SLO per service owner in SLO doc |
| Postmortems are blame-heavy | Cultural resistance | Train on blameless principles, leadership models behavior |
| Toil keeps growing | No time budgeted for automation | Enforce 50% cap — track toil hours explicitly |
| Alert fatigue | Alerts not tied to SLOs | Audit all alerts, delete symptom-only alerts without SLO link |
| Slow incident response | No runbooks, no training | Create runbook per alert, regular game days |

---

## Summary Table

| Practice | Key Metric |
|----------|------------|
| SLO-Driven | Error budget remaining (%) |
| Blameless Postmortems | Action items completed (%) |
| Toil Reduction | % time on toil (target <50%) |
| Team Ownership | "You build it, you run it, you observe it" |
| DORA Metrics | Deployment frequency, MTTR, CFR, lead time |
| Maturity Roadmap | Reactive → Proactive → Systematic → Optimized |

---

## Quick Revision Questions

1. **What is an error budget and how does it influence engineering decisions?**
2. **What makes a postmortem "blameless" and why does it matter?**
3. **What is toil, and what is the SRE target for toil budget?**
4. **Describe the "you build it, you run it" ownership model for observability.**
5. **What are DORA metrics and how does observability improve them?**
6. **Outline a 12-month observability maturity roadmap for an organization.**

---

[← Previous: Observability as Code](05-observability-as-code.md) | [Back to README](../README.md)
