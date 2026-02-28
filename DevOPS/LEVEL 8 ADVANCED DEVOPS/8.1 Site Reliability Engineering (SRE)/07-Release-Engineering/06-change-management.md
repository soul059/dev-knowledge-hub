# Chapter 7.6: Change Management

[← Previous: Rollback Strategies](05-rollback-strategies.md) | [Back to README](../README.md) | [Next: Scalability Patterns →](../08-System-Design/01-scalability-patterns.md)

---

## Overview

Change management in SRE is the structured process of planning, reviewing, approving, and executing changes to production systems while minimizing risk. Google's data shows that **~70% of outages are caused by changes to a running system**. Effective change management is about making changes safe, fast, and reversible — not about slowing things down with bureaucracy.

---

## 1. Why Changes Cause Outages

```
  ┌──────────────────────────────────────────────────────────┐
  │  CHANGE-RELATED OUTAGE CATEGORIES                        │
  │                                                          │
  │  ┌─────────────────┐                                     │
  │  │ Config changes   │──────── 35% of incidents           │
  │  │ (flags, params)  │                                     │
  │  └─────────────────┘                                     │
  │  ┌─────────────────┐                                     │
  │  │ Code deploys     │──────── 25% of incidents           │
  │  │ (new features)   │                                     │
  │  └─────────────────┘                                     │
  │  ┌─────────────────┐                                     │
  │  │ Infrastructure   │──────── 20% of incidents           │
  │  │ (scaling, certs) │                                     │
  │  └─────────────────┘                                     │
  │  ┌─────────────────┐                                     │
  │  │ Dependency       │──────── 15% of incidents           │
  │  │ (library, API)   │                                     │
  │  └─────────────────┘                                     │
  │  ┌─────────────────┐                                     │
  │  │ Data / schema    │──────── 5% of incidents            │
  │  │ (migrations)     │                                     │
  │  └─────────────────┘                                     │
  │                                                          │
  │  KEY INSIGHT: Config changes cause more outages than     │
  │  code deploys because they often bypass review and       │
  │  testing processes.                                      │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Change Risk Classification

```
  ┌───────────┬──────────────────────┬──────────────────────────┐
  │ Risk Tier │ Characteristics      │ Process Required         │
  ├───────────┼──────────────────────┼──────────────────────────┤
  │           │ - Well-tested        │ - Automated deployment   │
  │ LOW       │ - Feature flagged    │ - Peer code review       │
  │           │ - Gradual rollout    │ - Automated rollback     │
  │           │ - Non-critical path  │ - Post-deploy monitoring │
  │           │                      │                          │
  ├───────────┼──────────────────────┼──────────────────────────┤
  │           │ - DB schema changes  │ - All LOW requirements + │
  │ MEDIUM    │ - Auth/security      │ - SRE review required    │
  │           │ - Cross-service API  │ - Staged rollout (canary)│
  │           │ - Payment flows      │ - 1-hour bake time       │
  │           │                      │                          │
  ├───────────┼──────────────────────┼──────────────────────────┤
  │           │ - Infra migration    │ - All MEDIUM reqs +      │
  │ HIGH      │ - Data model change  │ - Change advisory board  │
  │           │ - Networking/DNS     │ - Maintenance window     │
  │           │ - Core crypto/TLS    │ - Backup verification    │
  │           │                      │ - Dedicated IC on-call   │
  ├───────────┼──────────────────────┼──────────────────────────┤
  │           │ - Multi-region       │ - All HIGH reqs +        │
  │ CRITICAL  │ - Auth provider swap │ - Exec approval          │
  │           │ - Primary database   │ - Region-by-region       │
  │           │ - DNS root changes   │ - Active war room        │
  │           │                      │ - Rollback rehearsal     │
  └───────────┴──────────────────────┴──────────────────────────┘
```

---

## 3. SRE-Friendly Change Process

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐│
  │  │ PLAN   │─▶│ REVIEW │─▶│ TEST   │─▶│ DEPLOY │─▶│ VERIFY ││
  │  └────────┘  └────────┘  └────────┘  └────────┘  └────────┘│
  │                                                              │
  │  PLAN:                                                       │
  │  ├─ What exactly is changing?                                │
  │  ├─ What could go wrong? (pre-mortem)                       │
  │  ├─ How do we rollback?                                      │
  │  └─ What's the blast radius?                                │
  │                                                              │
  │  REVIEW:                                                     │
  │  ├─ Peer review (code + config)                             │
  │  ├─ SRE review (for medium+ risk)                           │
  │  └─ Auto-checks (lint, schema compat, security scan)        │
  │                                                              │
  │  TEST:                                                       │
  │  ├─ Unit + integration tests (CI)                           │
  │  ├─ Staging deployment + smoke tests                        │
  │  └─ Rollback test in staging                                │
  │                                                              │
  │  DEPLOY:                                                     │
  │  ├─ Progressive rollout (canary → %)                        │
  │  ├─ Automated health checks between stages                  │
  │  └─ Human gate for high-risk changes                        │
  │                                                              │
  │  VERIFY:                                                     │
  │  ├─ Post-deploy monitoring (30 min min)                     │
  │  ├─ Key metric dashboards reviewed                          │
  │  └─ Sign-off: "Change successful"                           │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

---

## 4. Change Request Template

```yaml
# Change Request (CR) Template
change_request:
  id: "CR-2024-0892"
  title: "Migrate user sessions from Redis to DynamoDB"
  
  classification:
    risk_tier: "HIGH"
    change_type: "infrastructure"
    services_affected:
      - auth-service
      - session-manager
      - api-gateway
    blast_radius: "All authenticated users (~2M active)"

  schedule:
    proposed_window: "2024-03-15 02:00-06:00 UTC"  # Low traffic
    estimated_duration: "2 hours"
    freeze_periods_check: "No freeze (checked)"

  plan:
    description: |
      Migrate session storage from self-managed Redis cluster 
      to AWS DynamoDB for improved availability and reduced 
      operational burden.
    steps:
      - "Deploy dual-write code (write to both Redis and DynamoDB)"
      - "Enable read-from-DynamoDB for 5% of traffic"
      - "Monitor error rates and latency for 30 minutes"
      - "Gradually increase to 100% over 2 hours"
      - "Disable Redis reads after 24-hour bake period"
    
  rollback:
    plan: |
      Revert traffic routing to Redis. DynamoDB writes 
      continue but reads revert to Redis. Feature flag 
      controls this: session.storage.backend=redis
    estimated_rollback_time: "< 30 seconds (feature flag)"
    rollback_tested: true
    rollback_test_date: "2024-03-12"
    
  risk_assessment:
    what_could_go_wrong:
      - "DynamoDB latency higher than Redis (p99)"
      - "Session data format incompatibility"
      - "DynamoDB throttling under peak load"
    mitigations:
      - "DynamoDB DAX cache layer provisioned"
      - "Extensive integration testing completed"
      - "Auto-scaling configured, load tested to 3x peak"
    
  approvals:
    - role: "Service Owner"
      name: "Sarah Chen"
      status: "approved"
    - role: "SRE"
      name: "Marcus Johnson"
      status: "approved"
    - role: "Security"
      name: "Priya Patel"
      status: "approved"
```

---

## 5. Change Freezes and Safe Deployment Windows

```
  ┌──────────────────────────────────────────────────────────┐
  │  DEPLOYMENT SAFETY CALENDAR                              │
  │                                                          │
  │  SAFE WINDOWS:                                           │
  │  ┌──────────────────────────────────────────────┐        │
  │  │ Mon  Tue  Wed  Thu  Fri  Sat  Sun           │        │
  │  │  ✓    ✓    ✓    ✓    ✗    ✗    ✗            │        │
  │  │                                              │        │
  │  │ Deploy Mon-Thu, 9am-2pm local time           │        │
  │  │ Emergency only: Fri-Sun                      │        │
  │  └──────────────────────────────────────────────┘        │
  │                                                          │
  │  FREEZE PERIODS:                                         │
  │  ├─ Major holidays (Dec 15 - Jan 5)                     │
  │  ├─ Major sales events (Black Friday week)              │
  │  ├─ End of financial quarter (last 3 days)              │
  │  └─ During active SEV-1/SEV-2 incidents                 │
  │                                                          │
  │  FREEZE EXCEPTIONS:                                      │
  │  ├─ Security patches (CVE critical/high)                │
  │  ├─ Active incident mitigations                         │
  │  └─ Requires VP+ approval with SRE sign-off            │
  │                                                          │
  │  CULTURE NOTE:                                           │
  │  High-performing teams deploy ANY day because their     │
  │  CI/CD pipeline and rollback processes are so reliable   │
  │  that day-of-week doesn't affect safety. The goal is    │
  │  to reach this maturity level, not to rely on freezes.  │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Configuration as Code (CaC)

```
  ┌──────────────────────────────────────────────────────────┐
  │  Config changes cause 35% of outages because they       │
  │  bypass normal code review. Treat config like code!      │
  │                                                          │
  │  BEFORE: Config in UI / manual changes                   │
  │  ┌──────────┐    ┌──────────┐    ┌──────────┐           │
  │  │ Engineer │──▶ │ Admin UI │──▶ │ PROD     │           │
  │  │ clicks   │    │ (no rev) │    │ (broken) │           │
  │  └──────────┘    └──────────┘    └──────────┘           │
  │  No review. No audit trail. No rollback.                │
  │                                                          │
  │  AFTER: Configuration as Code                            │
  │  ┌──────────┐    ┌──────────┐    ┌──────────┐           │
  │  │ Engineer │──▶ │ Git PR   │──▶ │ CI/CD    │           │
  │  │ edits    │    │ (review) │    │ pipeline │           │
  │  │ YAML     │    │ (tests)  │    │ (staged) │           │
  │  └──────────┘    └──────────┘    └──────────┘           │
  │  Full review. Audit trail. Git revert for rollback.     │
  └──────────────────────────────────────────────────────────┘
```

```yaml
# Example: Rate limiting config managed as code
# File: config/rate-limits.yaml (in Git repo)
rate_limits:
  api_global:
    requests_per_second: 10000
    burst_size: 15000
    
  per_user:
    authenticated:
      requests_per_minute: 600
      burst_size: 100
    anonymous:
      requests_per_minute: 60
      burst_size: 20

  per_endpoint:
    /api/v1/search:
      requests_per_minute: 100
      burst_size: 30
    /api/v1/upload:
      requests_per_minute: 10
      burst_size: 5

# Validation in CI pipeline:
# - Schema validation (are all fields valid?)
# - Sanity checks (rate > 0, burst >= rate)
# - Diff review (what exactly changed?)
# - Canary deploy (apply to 1 pod first)
```

---

## 7. DORA-Based Change Metrics

```
  ┌──────────────────────────────────────────────────────────┐
  │  MEASURING CHANGE MANAGEMENT EFFECTIVENESS               │
  │                                                          │
  │  ┌──────────────────────┬───────────┬──────────────────┐ │
  │  │ Metric               │ Target    │ Current         │ │
  │  ├──────────────────────┼───────────┼──────────────────┤ │
  │  │ Deploy Frequency     │ Daily+    │ 4x/day          │ │
  │  │ Lead Time for Change │ < 1 day   │ 6 hours         │ │
  │  │ Change Failure Rate  │ < 5%      │ 3.2%            │ │
  │  │ MTTR (from change)   │ < 1 hour  │ 12 min          │ │
  │  │ Rollback Success %   │ > 99%     │ 99.7%           │ │
  │  │ Review Cycle Time    │ < 4 hours │ 2.5 hours       │ │
  │  │ Time-to-production   │ < 24 hrs  │ 8 hours         │ │
  │  └──────────────────────┴───────────┴──────────────────┘ │
  │                                                          │
  │  HIGH-PERFORMING TEAMS:                                  │
  │  ├─ Deploy 100x more frequently                         │
  │  ├─ Have 3x lower change failure rate                   │
  │  ├─ Recover 2600x faster                                │
  │  └─ Achieve this through AUTOMATION, not bureaucracy    │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] Config Change Outage and Process Improvement

```
  INCIDENT: Global rate limiter misconfiguration
  
  WHAT HAPPENED:
  ├─ Engineer changed rate limit from 10,000 rps to 1,000 rps
  │   (intended to change per-user limit, changed global limit)
  ├─ Change made via admin UI (no code review)
  ├─ Applied to all regions simultaneously
  └─ 60% of legitimate traffic blocked for 23 minutes
  
  TIMELINE:
  14:02 — Config changed via admin UI
  14:05 — Alerts: 429 error rate spike (3 min detection delay)
  14:08 — On-call pages SRE
  14:12 — SRE identifies config change as cause
  14:15 — Struggle to find previous value (no version history!)
  14:19 — Guessed previous value, applied via admin UI
  14:22 — Tried 3 more values before finding correct one
  14:25 — Correct value restored, traffic recovers
  14:25 — Total outage: 23 minutes, ~$340K revenue impact
  
  ┌──────────────────────────────────────────────────────────┐
  │  PROCESS IMPROVEMENTS IMPLEMENTED                        │
  │                                                          │
  │  1. CONFIG AS CODE                                       │
  │     All config moved to Git repo with PR review          │
  │     Admin UI made read-only for production               │
  │                                                          │
  │  2. VALIDATION PIPELINE                                  │
  │     ├─ Schema validation (correct field types)          │
  │     ├─ Range checks (rate > 1000 for global)            │
  │     ├─ Diff display (shows exactly what changed)        │
  │     └─ Canary apply (1 region first, 15 min bake)       │
  │                                                          │
  │  3. AUDIT LOGGING                                        │
  │     Every config change logged: who, what, when, why    │
  │     Git provides built-in audit trail and rollback       │
  │                                                          │
  │  RESULT: Zero config-related outages in next 8 months   │
  └──────────────────────────────────────────────────────────┘
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Too many changes deploying at once, hard to isolate issues" | Enforce small, atomic changes. One logical change per deploy. Use feature flags to decouple deploy from release. |
| "Change review process too slow, bottlenecking velocity" | Automate low-risk approvals. Only require human review for medium+ risk. Pre-approved change templates for common operations. |
| "Config changes keep causing outages" | Move ALL config to version-controlled repos. Add validation CI. Treat config changes with same rigor as code changes. |
| "We keep deploying during incidents" | Implement automated freeze during active incidents. CI/CD pipeline checks incident status before allowing deploys. |
| "Friday deploys cause weekend pages" | Enforce deploy windows (Mon-Thu). Track day-of-week incident correlation. Aim for reliable-enough pipelines to remove this restriction eventually. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Change Classification** | LOW/MEDIUM/HIGH/CRITICAL risk tiers with escalating process requirements |
| **70% Rule** | ~70% of outages caused by changes — making changes safe is critical |
| **Config as Code** | Treat config with same rigor as code: Git, PR review, CI validation |
| **Change Freezes** | Restrict deploys during high-risk periods (holidays, sales events) |
| **DORA Metrics** | Deploy frequency, lead time, change failure rate, MTTR |
| **Blast Radius** | Number of users/services affected if change goes wrong |

---

## Quick Revision Questions

1. **Why are configuration changes particularly dangerous?**
   > Config changes cause ~35% of outages because they often bypass normal review and testing processes. They can also take effect immediately across all instances without gradual rollout.

2. **What are the four risk tiers and how do they differ?**
   > LOW (automated approval, peer review), MEDIUM (SRE review, canary required), HIGH (change board, maintenance window, backup verification), CRITICAL (exec approval, region-by-region, active war room, rollback rehearsal).

3. **What is Configuration as Code (CaC) and why does SRE advocate for it?**
   > Storing all configuration in version-controlled repositories (Git) with PR review, CI validation, and automated deployment — instead of manual UI changes. Provides audit trail, rollback capability, and peer review.

4. **What are the key DORA metrics for change management?**
   > Deploy frequency (how often), lead time for changes (how fast from commit to production), change failure rate (% of deploys causing incidents), and MTTR (how quickly you recover from failures).

5. **When should change freezes be implemented?**
   > During major holidays, sales events, end of financial quarters, and active SEV-1/SEV-2 incidents. High-maturity teams aim to eliminate the need for freezes through reliable CI/CD and rollback processes.

6. **What should a change request include?**
   > Description of change, risk classification, affected services, blast radius, step-by-step plan, rollback plan (tested!), risk assessment (what can go wrong + mitigations), and required approvals.

---

[← Previous: Rollback Strategies](05-rollback-strategies.md) | [Back to README](../README.md) | [Next: Scalability Patterns →](../08-System-Design/01-scalability-patterns.md)
