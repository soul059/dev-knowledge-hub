# Chapter 2.3: SLAs (Service Level Agreements)

[← Previous: SLOs](02-slos.md) | [Back to README](../README.md) | [Next: Choosing SLIs →](04-choosing-slis.md)

---

## Overview

A Service Level Agreement (SLA) is a **contractual commitment** between a service provider and its customers. It defines the expected level of service and the consequences (usually financial) when that level is not met. SLAs are built on top of SLOs but add legal and business teeth.

> **SLAs answer: "What happens if we fail to meet our commitments?"**

---

## 1. SLI → SLO → SLA Relationship

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │   SLI (Indicator)     → What we measure                  │
  │       │                  "99.7% of requests succeeded"    │
  │       ▼                                                  │
  │   SLO (Objective)     → What we aim for (internal)       │
  │       │                  "Target: 99.9% availability"     │
  │       ▼                                                  │
  │   SLA (Agreement)     → What we promise (external)       │
  │                          "If below 99.5%, customer        │
  │                           gets 10% credit"               │
  │                                                          │
  │  ┌─────────────────────────────────────────────┐         │
  │  │                                             │         │
  │  │  SLO Target (internal)    ──────  99.9%     │         │
  │  │  SLA Commitment (external) ─────  99.5%     │         │
  │  │                                             │         │
  │  │  [!] SLA is ALWAYS looser than SLO          │         │
  │  │      Buffer protects against payouts         │         │
  │  └─────────────────────────────────────────────┘         │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

### Why the Buffer?

```
  SLO: 99.9%  ──────────────────────────── internal target
                        ↕ buffer zone
  SLA: 99.5%  ──────────────────────────── contractual commitment

  This buffer means:
  • If you're at 99.8%, you've missed your SLO (fix things!)
  • But you haven't breached your SLA (no financial penalty)
  • Buffer gives you time to respond before it costs money
```

---

## 2. SLA Components

```
  ┌────────────────────────────────────────────────────┐
  │              SLA DOCUMENT STRUCTURE                │
  │                                                    │
  │  1. Service Description                            │
  │     What service is covered?                       │
  │                                                    │
  │  2. Performance Metrics                            │
  │     SLIs that will be measured                     │
  │                                                    │
  │  3. Service Levels                                 │
  │     Target values (SLO targets)                    │
  │                                                    │
  │  4. Measurement Method                             │
  │     How metrics are collected and reported         │
  │                                                    │
  │  5. Reporting Cadence                              │
  │     Monthly, quarterly reports                     │
  │                                                    │
  │  6. Remedies / Credits                             │
  │     What happens on SLA breach                     │
  │                                                    │
  │  7. Exclusions                                     │
  │     Planned maintenance, force majeure, etc.       │
  │                                                    │
  │  8. Term and Renewal                               │
  │     Duration and review process                    │
  │                                                    │
  └────────────────────────────────────────────────────┘
```

---

## 3. Real-World SLA Examples

### AWS EC2 SLA (Simplified)

```
  ┌─────────────────────────────────────────────────┐
  │  AWS EC2 Compute SLA                            │
  │                                                 │
  │  Monthly Uptime %   │  Service Credit %         │
  │  ──────────────────│──────────────────          │
  │  ≥ 99.99%          │  No credit                │
  │  99.0% - 99.99%    │  10% credit               │
  │  95.0% - 99.0%     │  30% credit               │
  │  < 95.0%           │  100% credit              │
  │                                                 │
  │  Exclusions:                                    │
  │  • Scheduled maintenance                        │
  │  • Customer-caused outages                      │
  │  • Force majeure                                │
  └─────────────────────────────────────────────────┘
```

### Google Cloud SLA (Simplified)

```
  ┌─────────────────────────────────────────────────┐
  │  Google Cloud Compute Engine SLA                 │
  │                                                 │
  │  Monthly Uptime %   │  Financial Credit %       │
  │  ──────────────────│──────────────────          │
  │  99.0% - 99.99%    │  10%                      │
  │  95.0% - 99.0%     │  25%                      │
  │  < 95.0%           │  50%                      │
  │                                                 │
  │  Multi-zone deployment: 99.99% SLA              │
  │  Single-zone:          99.5% SLA                │
  └─────────────────────────────────────────────────┘
```

---

## 4. SLA Credit Tiers — Designing Your Own

```yaml
# Example SLA for a SaaS product
sla:
  service: "Acme Analytics Platform"
  version: "2.0"
  effective_date: "2026-01-01"
  
  commitments:
    - metric: "Monthly Availability"
      target: 99.9%
      measurement: "1 - (minutes_of_downtime / total_minutes_in_month)"
      
      credit_tiers:
        - range: "99.5% - 99.9%"
          credit: "10% of monthly fee"
        - range: "99.0% - 99.5%"
          credit: "25% of monthly fee"
        - range: "95.0% - 99.0%"
          credit: "50% of monthly fee"
        - range: "< 95.0%"
          credit: "100% of monthly fee"
          
    - metric: "API Latency (p95)"
      target: "< 500ms"
      measurement: "95th percentile response time from load balancer logs"
      credit: "Not subject to financial credits (SLO only)"
      
  exclusions:
    - "Scheduled maintenance (notified 72h in advance)"
    - "Customer misuse or misconfiguration"
    - "Beta or preview features"
    - "Force majeure events"
    
  claim_process:
    - "Submit credit request within 30 days of incident"
    - "Include affected time period and evidence"
    - "Credits applied to next billing cycle"
    - "Maximum credit: 100% of monthly fee"
```

---

## 5. SLA vs SLO — Key Differences

| Aspect | SLO | SLA |
|--------|-----|-----|
| **Audience** | Internal (engineering teams) | External (customers, legal) |
| **Consequences** | Feature freeze, reliability focus | Financial credits, legal liability |
| **Strictness** | Tighter (e.g., 99.9%) | Looser (e.g., 99.5%) |
| **Flexibility** | Can be adjusted by team | Requires contract renegotiation |
| **Number** | 3-5 per service | 1-2 per service (key metrics) |
| **Latency SLIs** | Usually included | Often excluded (hard to guarantee) |
| **Who sets it** | SRE + Dev team | Business + Legal + SRE |

---

## 6. When NOT to Have an SLA

```
  ┌─────────────────────────────────────────────────┐
  │  SKIP SLAs WHEN:                                │
  │                                                 │
  │  • Free-tier services (no revenue to credit)    │
  │  • Internal services (use SLOs instead)         │
  │  • Beta / preview features                      │
  │  • Services where uptime isn't the key value    │
  │                                                 │
  │  ALWAYS HAVE SLAs WHEN:                         │
  │                                                 │
  │  • Enterprise contracts (customers expect them) │
  │  • Revenue-critical services                    │
  │  • Regulatory requirements                      │
  │  • Multi-tenant platforms                       │
  │                                                 │
  │  [!] Even without SLAs, ALWAYS have SLOs.       │
  │      SLOs drive internal reliability decisions.  │
  └─────────────────────────────────────────────────┘
```

---

## 7. The SLA Anti-Patterns

```
  ┌─────────────────────────────────────────────────────┐
  │              SLA ANTI-PATTERNS                      │
  │                                                     │
  │  ✗ SLA = SLO (no buffer)                            │
  │    → Every SLO miss triggers financial penalties    │
  │                                                     │
  │  ✗ SLA stricter than SLO                            │
  │    → Impossible to meet commitments                 │
  │                                                     │
  │  ✗ SLA without measurement infrastructure           │
  │    → Can't prove compliance or violations           │
  │                                                     │
  │  ✗ SLA with vague terms ("best effort")             │
  │    → Unenforceable, creates customer distrust       │
  │                                                     │
  │  ✗ SLA without exclusion clauses                    │
  │    → Liable for events beyond your control          │
  │                                                     │
  │  ✗ SLA on metrics you can't control                 │
  │    → e.g., guaranteeing client-side performance     │
  └─────────────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] SLA Negotiation for Enterprise Customer

```
  Company: CloudAPI Inc. (API gateway SaaS)
  Customer: MegaBank (enterprise, $500K/year contract)

  ┌──────────────────────────────────────────────────────┐
  │  Negotiation Process:                                │
  │                                                      │
  │  1. Customer requests: 99.99% availability SLA       │
  │                                                      │
  │  2. SRE team analysis:                              │
  │     • Current measured availability: 99.95%          │
  │     • Database dependency SLO: 99.99%                │
  │     • CDN dependency SLA: 99.95%                     │
  │     • Max achievable: ~99.95%                        │
  │                                                      │
  │  3. SRE recommendation:                              │
  │     • Internal SLO: 99.95%                           │
  │     • External SLA: 99.9% (with buffer)              │
  │     • Cannot offer 99.99% due to CDN dependency      │
  │                                                      │
  │  4. Business negotiation:                            │
  │     • SLA: 99.9% (standard tier)                     │
  │     • Premium SLA: 99.95% (+20% contract value)      │
  │       requires multi-region deployment               │
  │                                                      │
  │  5. Final agreement:                                 │
  │     • 99.9% SLA with 10%/25%/50% credit tiers       │
  │     • Scheduled maintenance excluded                  │
  │     • 30-day claim window                            │
  └──────────────────────────────────────────────────────┘
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Customer demanding SLA we can't meet" | Show dependency analysis. Offer premium tier at higher price with architectural changes. |
| "We breached SLA but shouldn't pay credits" | Ensure exclusion clauses cover the scenario. Document everything. |
| "SLA credits are too expensive" | Set buffer between SLO and SLA. Invest in reliability to stay within SLO. |
| "No one tracks SLA compliance" | Automate SLA reporting. Include in monthly business reviews. |
| "Different customers want different SLAs" | Create tiered SLA offerings (Standard, Premium, Enterprise). |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **SLA Definition** | Contractual commitment with financial consequences |
| **SLA vs SLO** | SLA is external and looser; SLO is internal and tighter |
| **Buffer** | SLO target > SLA commitment (e.g., 99.9% SLO, 99.5% SLA) |
| **Credit Tiers** | Graduated penalties based on severity of breach |
| **Exclusions** | Planned maintenance, customer fault, force majeure |
| **Claim Process** | Customer-initiated within a defined window |
| **Key Rule** | Always have SLOs. Only add SLAs when business requires them. |

---

## Quick Revision Questions

1. **What is the relationship between SLIs, SLOs, and SLAs?**
   > SLIs are what you measure, SLOs are your internal targets, SLAs are external contractual commitments with financial consequences.

2. **Why should the SLA always be less strict than the SLO?**
   > The buffer between SLO and SLA gives you time to detect and fix issues before incurring financial penalties.

3. **Name three common SLA exclusions.**
   > Scheduled maintenance (with advance notice), customer-caused issues, and force majeure events.

4. **When should you NOT have an SLA?**
   > For free-tier services, internal services, beta features, or when uptime isn't the primary value proposition.

5. **How do cloud providers like AWS structure their SLA credit tiers?**
   > Graduated credits based on severity: e.g., 10% credit for 99-99.99% uptime, 30% for 95-99%, 100% for <95%.

6. **What is the main anti-pattern when setting an SLA?**
   > Setting SLA = SLO (no buffer). Every SLO miss becomes a financial penalty, creating constant payouts.

---

[← Previous: SLOs](02-slos.md) | [Back to README](../README.md) | [Next: Choosing SLIs →](04-choosing-slis.md)
