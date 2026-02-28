# Chapter 3.1: Availability Calculations

[← Previous: Error Budgets](../02-Service-Level-Objectives/06-error-budgets.md) | [Back to README](../README.md) | [Next: Measuring Reliability →](02-measuring-reliability.md)

---

## Overview

Availability is the most fundamental reliability metric. It quantifies the proportion of time a service is operational and serving users correctly. Understanding how to calculate availability — especially for complex systems with dependencies — is essential for setting realistic SLOs and designing resilient architectures.

---

## 1. Basic Availability Formula

```
                     Uptime
  Availability = ──────────────── × 100%
                  Uptime + Downtime

  OR equivalently:

                  Total Time − Downtime
  Availability = ────────────────────── × 100%
                       Total Time

  Example:
    In a 30-day month (43,200 min), service was down for 43.2 min:
    Availability = (43,200 - 43.2) / 43,200 = 99.9%
```

---

## 2. The Nines Table (Complete Reference)

```
  ┌───────────┬──────────────────┬──────────────────┬─────────────────┐
  │  Avail.   │ Downtime/Year    │ Downtime/Month   │ Downtime/Week   │
  ├───────────┼──────────────────┼──────────────────┼─────────────────┤
  │  90%      │ 36.5 days        │ 3 days           │ 16.8 hours      │
  │  95%      │ 18.25 days       │ 1.5 days         │ 8.4 hours       │
  │  99%      │ 3.65 days        │ 7.2 hours        │ 1.68 hours      │
  │  99.5%    │ 1.83 days        │ 3.6 hours        │ 50.4 min        │
  │  99.9%    │ 8.76 hours       │ 43.2 min         │ 10.1 min        │
  │  99.95%   │ 4.38 hours       │ 21.6 min         │ 5.04 min        │
  │  99.99%   │ 52.6 min         │ 4.32 min         │ 1.01 min        │
  │  99.999%  │ 5.26 min         │ 25.9 sec         │ 6.05 sec        │
  │  99.9999% │ 31.5 sec         │ 2.59 sec         │ 0.605 sec       │
  └───────────┴──────────────────┴──────────────────┴─────────────────┘
```

---

## 3. MTBF, MTTR, and MTTF

```
  ┌──────────────────────────────────────────────────────────────┐
  │          RELIABILITY TIME METRICS                            │
  │                                                              │
  │  Timeline:                                                   │
  │  ──────────────────────────────────────────────────────      │
  │  │ UP │ DOWN │    UP    │DOWN│      UP      │ DOWN │ UP     │
  │  ──────────────────────────────────────────────────────      │
  │       ▲       ▲          ▲    ▲               ▲      ▲      │
  │       │       │          │    │               │      │      │
  │       │  MTTR │   MTBF   │MTTR│    MTBF       │ MTTR │      │
  │       │       │          │    │               │      │      │
  │                                                              │
  │  MTBF = Mean Time Between Failures                           │
  │       = Total Uptime / Number of Failures                    │
  │                                                              │
  │  MTTR = Mean Time To Repair/Recover                          │
  │       = Total Downtime / Number of Failures                  │
  │                                                              │
  │  MTTF = Mean Time To Failure (for non-repairable items)      │
  │       = Total Operating Time / Number of Failures            │
  │                                                              │
  │  Availability = MTBF / (MTBF + MTTR)                        │
  │                                                              │
  │  Example:                                                    │
  │    MTBF = 720 hours (30 days)                                │
  │    MTTR = 0.72 hours (43.2 min)                              │
  │    Availability = 720 / (720 + 0.72) = 99.9%                │
  └──────────────────────────────────────────────────────────────┘
```

### Key Insight

```
  To improve availability, you can EITHER:
  
  1. Increase MTBF (fail less often)
     → Better testing, robust architecture, chaos engineering
     → Expensive, diminishing returns
  
  2. Decrease MTTR (recover faster)
     → Better monitoring, automation, runbooks, rollback
     → Often more cost-effective
  
  ┌────────────────────────────────────────────┐
  │  [TIP] Reducing MTTR is usually the       │
  │  best ROI for improving availability.      │
  │                                            │
  │  Going from 2-hour MTTR to 15-minute MTTR │
  │  is often easier than preventing failures. │
  └────────────────────────────────────────────┘
```

---

## 4. Composite Availability — Series Systems

When components are in **series** (all must work), multiply availability:

```
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │  Web App │────►│  API GW  │────►│ Database │
  │  99.99%  │     │  99.9%   │     │  99.99%  │
  └──────────┘     └──────────┘     └──────────┘
  
  Combined Availability = 0.9999 × 0.999 × 0.9999
                        = 0.9988
                        ≈ 99.88%
  
  [!] The weakest component dominates!
      Web (99.99%) × API GW (99.9%) × DB (99.99%) = 99.88%
      You cannot achieve 99.99% with a 99.9% dependency.
```

### Example Calculation

```
  E-Commerce Architecture:
  
  CDN (99.95%) ──► Load Balancer (99.99%) ──► App Server (99.99%)
       ──► Cache (99.99%) ──► Database (99.95%)
  
  Combined = 0.9995 × 0.9999 × 0.9999 × 0.9999 × 0.9995
           = 0.9987
           ≈ 99.87%
  
  Even though most components are 99.99%, two at 99.95%
  pull the whole system down to 99.87%.
```

---

## 5. Composite Availability — Parallel Systems (Redundancy)

When components are in **parallel** (any one suffices), availability improves:

```
  ┌──────────┐
  │ Server A │──┐
  │  99.9%   │  │
  └──────────┘  │
                ├──► Result (either works)
  ┌──────────┐  │
  │ Server B │──┘
  │  99.9%   │
  └──────────┘
  
  P(both fail) = (1 - 0.999) × (1 - 0.999) = 0.001 × 0.001 = 0.000001
  Combined Availability = 1 - 0.000001 = 99.9999%
  
  Formula: A_combined = 1 - ∏(1 - A_i)
  
  Two 99.9% servers in parallel = 99.9999% (six nines!)
```

### N-way Redundancy

```
  ┌──────────────────────────────────────────────────┐
  │  Parallel Redundancy Results (each node 99.9%)   │
  │                                                  │
  │  Nodes │ Combined Availability │ Nines           │
  │  ──────│───────────────────────│──────           │
  │    1   │ 99.9%                 │ Three           │
  │    2   │ 99.9999%              │ Six             │
  │    3   │ 99.9999999%           │ Nine            │
  │                                                  │
  │  [!] But this assumes INDEPENDENT failures.      │
  │      Correlated failures (e.g., same rack,       │
  │      same region, same bug) reduce this benefit. │
  └──────────────────────────────────────────────────┘
```

---

## 6. Mixed Series-Parallel Systems

Real architectures combine both patterns:

```
  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │       ┌─────────┐                                    │
  │   ┌──►│  App A  │──┐     ┌──────────┐               │
  │   │   │  99.9%  │  │     │          │               │
  │   │   └─────────┘  ├────►│  DB      │               │
  │   │                │     │  Primary │               │
  │   │   ┌─────────┐  │     │  99.99%  │               │
  │   ├──►│  App B  │──┘     │          │               │
  │   │   │  99.9%  │        └────┬─────┘               │
  │   │   └─────────┘             │ replication          │
  │   │                      ┌────▼─────┐               │
  │   │                      │  DB      │               │
  │   │   (parallel)         │  Replica │               │
  │   │                      │  99.99%  │               │
  │   │                      └──────────┘               │
  │                                                      │
  │  Step 1: App layer (parallel)                        │
  │    A_app = 1 - (1-0.999)(1-0.999) = 99.9999%        │
  │                                                      │
  │  Step 2: DB layer (parallel with failover)           │
  │    A_db = 1 - (1-0.9999)(1-0.9999) = 99.999999%     │
  │                                                      │
  │  Step 3: Combined (series: app then db)              │
  │    A_total = 0.999999 × 0.99999999 ≈ 99.999899%     │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

---

## 7. Request-Based vs Time-Based Availability

```
  ┌──────────────────────────────────────────────────────────┐
  │  TIME-BASED AVAILABILITY                                 │
  │  ───────────────────────                                 │
  │  A = uptime / (uptime + downtime)                        │
  │                                                          │
  │  Simple but coarse. A 1-minute blip counts the same      │
  │  whether 10 users or 10 million users were affected.     │
  │                                                          │
  │  REQUEST-BASED AVAILABILITY (preferred for SLOs)         │
  │  ──────────────────────────────────────────────          │
  │  A = successful requests / total requests                │
  │                                                          │
  │  More nuanced. A partial failure affecting 1% of         │
  │  users shows as ~99% availability, not 0%.               │
  │                                                          │
  │  Example:                                                │
  │    1 million requests, 500 failed                        │
  │    Request-based: 999,500/1,000,000 = 99.95%             │
  │                                                          │
  │    System was "up" the whole time (no full outage)        │
  │    Time-based: 100% — misses the partial failure!        │
  │                                                          │
  │  [!] SRE strongly prefers REQUEST-BASED availability     │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] Calculating Availability for a Microservices Architecture

```
  ┌──────────────────────────────────────────────────────────┐
  │  Architecture: Online Booking System                     │
  │                                                          │
  │  User Request Flow:                                      │
  │                                                          │
  │  CDN ──► LB ──► API GW ──► Auth ──► Booking ──► DB      │
  │  99.95    99.99   99.9    99.9    99.95    99.99         │
  │                                                          │
  │  Series calculation:                                     │
  │  A = 0.9995 × 0.9999 × 0.999 × 0.999 × 0.9995 × 0.9999│
  │  A = 0.9968                                              │
  │  A ≈ 99.68%                                              │
  │                                                          │
  │  That's ~2.3 hours of downtime per month!                │
  │                                                          │
  │  Improvement: Add redundancy to weakest links            │
  │                                                          │
  │  • Add second Auth server (parallel):                    │
  │    Auth: 1-(1-0.999)² = 99.9999%                         │
  │                                                          │
  │  • Result with redundant Auth + Booking:                 │
  │    A = 0.9995 × 0.9999 × 0.999 × 0.999999 ×            │
  │        0.999999 × 0.9999                                 │
  │    A ≈ 99.88%                                            │
  │                                                          │
  │  Still limited by CDN (99.95%) and API GW (99.9%)        │
  └──────────────────────────────────────────────────────────┘
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Our calculated availability doesn't match reality" | Ensure you're measuring request-based, not just time-based. Include partial failures. |
| "Adding redundancy didn't improve availability" | Check for correlated failures. Same rack/region/software version means failures aren't independent. |
| "Dependency with low SLO is limiting us" | Either improve the dependency, add redundancy/caching in front of it, or accept the ceiling. |
| "We achieved 99.99% but users still complain" | Availability alone isn't enough. Check latency and correctness SLIs. |

---

## Summary Table

| Concept | Formula / Description |
|---------|----------------------|
| **Basic Availability** | Uptime / (Uptime + Downtime) |
| **MTBF-based** | MTBF / (MTBF + MTTR) |
| **Series (all must work)** | A₁ × A₂ × ... × Aₙ |
| **Parallel (any works)** | 1 − (1−A₁)(1−A₂)...(1−Aₙ) |
| **Request-based** | Successful requests / Total requests (preferred) |
| **MTTR vs MTBF** | Reducing MTTR is usually better ROI than increasing MTBF |
| **Key Rule** | System availability ≤ weakest serial dependency |

---

## Quick Revision Questions

1. **Calculate the availability of a system with MTBF=500h and MTTR=0.5h.**
   > A = 500 / (500 + 0.5) = 99.9%.

2. **Three services at 99.9% are in series. What is the combined availability?**
   > 0.999³ = 0.997 = 99.7%.

3. **Two servers at 99.9% are in parallel. What is the combined availability?**
   > 1 − (0.001)² = 1 − 0.000001 = 99.9999%.

4. **Why does SRE prefer request-based over time-based availability?**
   > Request-based captures partial failures that time-based misses. A service can be "up" but returning errors to some users.

5. **Why is reducing MTTR often better ROI than increasing MTBF?**
   > Preventing all failures is infinitely expensive. Faster recovery (monitoring, automation, runbooks) is typically achievable and more cost-effective.

---

[← Previous: Error Budgets](../02-Service-Level-Objectives/06-error-budgets.md) | [Back to README](../README.md) | [Next: Measuring Reliability →](02-measuring-reliability.md)
