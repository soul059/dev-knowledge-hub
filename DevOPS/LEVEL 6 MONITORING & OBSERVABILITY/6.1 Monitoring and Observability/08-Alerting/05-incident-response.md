# Chapter 5: Incident Response

[← Previous: On-Call Management](04-on-call-management.md) | [Back to README](../README.md) | [Next: Incident Management →](06-incident-management.md)

---

## Overview

Incident response is the structured process of detecting, triaging, communicating, and resolving production incidents. A well-defined incident response process reduces downtime, minimizes customer impact, and prevents panic.

---

## 5.1 Incident Lifecycle

```
┌──────────────────────────────────────────────────────────┐
│              INCIDENT LIFECYCLE                          │
│                                                          │
│  ┌─────────┐   ┌─────────┐   ┌──────────┐              │
│  │ DETECT  │──►│ TRIAGE  │──►│ RESPOND  │              │
│  │         │   │         │   │          │              │
│  │ Alert   │   │ Assess  │   │ Diagnose │              │
│  │ fires   │   │ severity│   │ & Fix    │              │
│  └─────────┘   └─────────┘   └────┬─────┘              │
│                                    │                     │
│                                    ▼                     │
│                              ┌──────────┐               │
│                              │ RECOVER  │               │
│                              │          │               │
│                              │ Verify   │               │
│                              │ & Stable │               │
│                              └────┬─────┘               │
│                                   │                      │
│                                   ▼                      │
│                            ┌────────────┐               │
│                            │  LEARN     │               │
│                            │            │               │
│                            │ Postmortem │               │
│                            │ & Improve  │               │
│                            └────────────┘               │
│                                                          │
│  Key Metrics:                                           │
│  TTD (Time to Detect) + TTT (Time to Triage)           │
│  + TTR (Time to Resolve) = Total Incident Duration     │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 Incident Severity Classification

| Severity | Impact | Examples | Response |
|----------|--------|----------|----------|
| SEV1 / P1 | Full outage, all users affected | Site down, data loss, security breach | All hands, war room, exec comms |
| SEV2 / P2 | Major degradation, many users | Payment failures, slow > 10x normal | On-call + team lead, customer comms |
| SEV3 / P3 | Minor degradation, some users | Feature partially broken, intermittent errors | On-call investigates, business hours |
| SEV4 / P4 | Minimal impact, cosmetic | UI glitch, non-critical feature | Ticket, fix in next sprint |

---

## 5.3 Incident Response Roles

```
┌──────────────────────────────────────────────────────────┐
│  INCIDENT RESPONSE ROLES (ICS-inspired)                  │
│                                                          │
│  ┌──────────────────────────────────┐                   │
│  │  INCIDENT COMMANDER (IC)         │                   │
│  │  • Owns the incident             │                   │
│  │  • Coordinates response          │                   │
│  │  • Makes decisions               │                   │
│  │  • Delegates tasks               │                   │
│  │  • NOT debugging (managing)      │                   │
│  └────────────┬─────────────────────┘                   │
│               │                                          │
│  ┌────────────┼───────────┬─────────────┐               │
│  ▼            ▼           ▼             ▼               │
│  ┌──────┐ ┌────────┐ ┌────────┐ ┌──────────┐          │
│  │Comms │ │ Tech   │ │Subject │ │Scribe    │          │
│  │Lead  │ │ Lead   │ │Matter  │ │          │          │
│  │      │ │        │ │Experts │ │          │          │
│  │Update│ │Debug & │ │        │ │Document  │          │
│  │status│ │fix the │ │Called  │ │timeline, │          │
│  │page, │ │issue   │ │in as   │ │actions,  │          │
│  │Slack,│ │        │ │needed  │ │decisions │          │
│  │execs │ │        │ │        │ │          │          │
│  └──────┘ └────────┘ └────────┘ └──────────┘          │
└──────────────────────────────────────────────────────────┘
```

---

## 5.4 Incident Response Playbook

### Step-by-Step Process

```
┌──────────────────────────────────────────────────────────┐
│  INCIDENT RESPONSE PLAYBOOK                              │
│                                                          │
│  1. ALERT RECEIVED                                      │
│     □ Acknowledge alert within 5 minutes                │
│     □ Read alert summary and annotations                │
│     □ Open runbook link from alert                      │
│                                                          │
│  2. ASSESS SEVERITY                                     │
│     □ Check: Is service completely down or degraded?    │
│     □ Check: How many users affected?                   │
│     □ Check: Is data being lost?                        │
│     □ Assign SEV level (1-4)                            │
│                                                          │
│  3. COMMUNICATE                                         │
│     □ Post in #incidents Slack channel                  │
│     □ If SEV1/2: Start war room / video call            │
│     □ If SEV1: Notify stakeholders                      │
│     □ Update status page if customer-facing             │
│                                                          │
│  4. DIAGNOSE                                            │
│     □ Check dashboards (Grafana)                        │
│     □ Check recent deployments                          │
│     □ Check logs (Loki/ELK)                             │
│     □ Check traces for errors (Tempo/Jaeger)            │
│     □ Check infrastructure (K8s, cloud)                 │
│                                                          │
│  5. MITIGATE                                            │
│     □ Rollback if caused by deployment                  │
│     □ Scale up if capacity issue                        │
│     □ Failover if infrastructure issue                  │
│     □ Toggle feature flag if feature is causing issue   │
│     □ Apply hotfix if quick fix available               │
│                                                          │
│  6. VERIFY RECOVERY                                     │
│     □ Confirm metrics return to normal                  │
│     □ Check error rate drops                            │
│     □ Verify customer-facing functionality              │
│     □ Monitor for 15+ minutes for stability             │
│                                                          │
│  7. CLOSE INCIDENT                                      │
│     □ Resolve alert in PagerDuty/OpsGenie               │
│     □ Update status page to "Resolved"                  │
│     □ Post summary in #incidents channel                │
│     □ Schedule postmortem within 48 hours               │
└──────────────────────────────────────────────────────────┘
```

---

## 5.5 Communication Templates

### Status Page Update

```
[Investigating] 14:05 UTC
We are investigating reports of elevated error rates
on the API. Some users may experience 500 errors.

[Identified] 14:15 UTC
The issue has been identified as a database connection
pool exhaustion following a traffic spike. We are
scaling the connection pool and database replicas.

[Monitoring] 14:35 UTC
A fix has been deployed. Error rates are returning to
normal. We are monitoring the situation.

[Resolved] 15:00 UTC
The issue has been fully resolved. API error rates
are back to normal levels. A postmortem will follow.
Total impact duration: 55 minutes.
```

### Slack Incident Channel Format

```
┌──────────────────────────────────────────────────────────┐
│  #incident-2024-01-15-api-errors                        │
│                                                          │
│  📋 INCIDENT HEADER                                     │
│  ─────────────────────                                   │
│  Severity: SEV2                                          │
│  IC: @alice                                              │
│  Tech Lead: @bob                                         │
│  Comms Lead: @carol                                      │
│  Start Time: 2024-01-15 14:05 UTC                       │
│  Status: Active                                          │
│  Impact: ~15% of API requests returning 500              │
│  Status Page: Updated                                    │
│                                                          │
│  🕐 TIMELINE                                            │
│  14:05 - Alert fired: APIHighErrorRate                  │
│  14:07 - @alice acknowledged, assessing                 │
│  14:10 - Declared SEV2, war room started                │
│  14:15 - Root cause: DB connection pool exhausted       │
│  14:20 - Scaling DB connections from 50 → 200           │
│  14:30 - Error rate dropping                            │
│  14:45 - Error rate < 0.1%, monitoring                  │
│  15:00 - Incident resolved                              │
└──────────────────────────────────────────────────────────┘
```

---

## 5.6 Common Diagnostic Steps

| Check | Tools | What to Look For |
|-------|-------|------------------|
| Recent deployments | Git, CI/CD, ArgoCD | Deploy correlating with incident start |
| Error rate metrics | Grafana, Prometheus | Spike timing, affected services |
| Logs | Loki, ELK, kubectl logs | Error messages, stack traces |
| Traces | Tempo, Jaeger | Slow spans, error spans, root cause |
| Infrastructure | kubectl, cloud console | Pod restarts, node issues, OOM |
| Dependencies | Service map, status pages | External service outage |
| Config changes | Terraform, ConfigMaps | Recent config rollout |

---

## 5.7 Mitigation Strategies

```
┌──────────────────────────────────────────────────────────┐
│  MITIGATION DECISION TREE                                │
│                                                          │
│  Was there a recent deployment?                         │
│  ├─ YES → ROLLBACK deployment                          │
│  └─ NO ──► Is it a capacity issue?                      │
│            ├─ YES → SCALE UP (replicas, resources)      │
│            └─ NO ──► Is a dependency down?               │
│                      ├─ YES → FAILOVER or circuit break │
│                      └─ NO ──► Is it a feature bug?      │
│                                ├─ YES → FEATURE FLAG off│
│                                └─ NO ──► Is it infra?    │
│                                          ├─ YES → REBOOT│
│                                          └─ NO → HOTFIX │
│                                                          │
│  Priority: Restore service first, investigate later.    │
│  "Stop the bleeding, then do surgery."                  │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Slow response | No clear process | Document and drill incident playbook |
| Chaos in war room | No incident commander | Always assign IC first |
| Customers unaware | No status page updates | Assign comms lead role |
| Repeat incidents | No postmortem follow-up | Mandate postmortem + action items |
| Can't diagnose | Missing observability | Improve instrumentation post-incident |

---

## Quick Revision Questions

1. **What are the five phases of the incident lifecycle?**
2. **What are the key roles in incident response and what does each do?**
3. **How do you decide the severity of an incident?**
4. **What is the first priority during an incident: finding root cause or restoring service?**
5. **What communication should happen during a SEV1 incident?**
6. **What is a mitigation decision tree and how does it help during incidents?**

---

[← Previous: On-Call Management](04-on-call-management.md) | [Back to README](../README.md) | [Next: Incident Management →](06-incident-management.md)
