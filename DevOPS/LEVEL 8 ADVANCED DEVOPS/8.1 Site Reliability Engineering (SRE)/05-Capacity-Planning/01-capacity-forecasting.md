# Chapter 5.1: Capacity Forecasting

[← Previous: Learning from Failures](../04-Incident-Management/06-learning-from-failures.md) | [Back to README](../README.md) | [Next: Load Testing →](02-load-testing.md)

---

## Overview

Capacity forecasting predicts future resource needs based on historical trends, growth projections, and planned changes. Accurate forecasting prevents both under-provisioning (outages) and over-provisioning (waste). In SRE, capacity planning is a proactive discipline that ensures systems can handle expected demand while maintaining SLO targets.

---

## 1. Capacity Planning Process

```
  ┌──────────────────────────────────────────────────────────┐
  │           CAPACITY PLANNING LIFECYCLE                    │
  │                                                          │
  │  1. MEASURE  ──▶ Current resource utilization            │
  │       │          (CPU, memory, disk, network, DB)        │
  │       ▼                                                  │
  │  2. FORECAST ──▶ Project future demand                   │
  │       │          (growth rate, seasonal patterns)        │
  │       ▼                                                  │
  │  3. PLAN     ──▶ Determine when/what to add              │
  │       │          (lead time, buffer, cost)               │
  │       ▼                                                  │
  │  4. EXECUTE  ──▶ Provision and deploy capacity           │
  │       │          (auto-scale, manual scaling)            │
  │       ▼                                                  │
  │  5. VALIDATE ──▶ Load test to verify capacity            │
  │       │          (meets SLO under projected load)        │
  │       ▼                                                  │
  │  6. MONITOR  ──▶ Track actual vs forecast, adjust         │
  │                                                          │
  │  Cadence: Monthly review, quarterly deep planning        │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Key Capacity Metrics

```
  ┌──────────────────────────────────────────────────────────┐
  │  RESOURCE       │ METRICS              │ ALARM THRESHOLD │
  ├─────────────────┼──────────────────────┼─────────────────┤
  │ CPU             │ Avg utilization      │ > 70% sustained │
  │                 │ P95 utilization      │ > 85%           │
  │                 │                      │                 │
  │ Memory          │ RSS / heap usage     │ > 80%           │
  │                 │ OOM events           │ > 0             │
  │                 │                      │                 │
  │ Disk            │ Utilization %        │ > 80%           │
  │                 │ IOPS                 │ > 80% of limit  │
  │                 │ Growth rate (GB/day) │ Days until full │
  │                 │                      │                 │
  │ Network         │ Bandwidth (Gbps)     │ > 70% of NIC   │
  │                 │ Packet drop rate     │ > 0.1%          │
  │                 │                      │                 │
  │ Database        │ Connection pool util │ > 80%           │
  │                 │ Query latency p99    │ Increasing trend│
  │                 │ Storage growth       │ Days until full │
  │                 │                      │                 │
  │ Application     │ Requests/sec (RPS)   │ > 70% of tested │
  │                 │ Latency p99          │ Increasing trend│
  │                 │ Error rate           │ > baseline      │
  │                 │ Thread pool usage    │ > 80%           │
  └─────────────────┴──────────────────────┴─────────────────┘
```

---

## 3. Forecasting Methods

```
  ┌──────────────────────────────────────────────────────────┐
  │  METHOD                │ BEST FOR              │ ACCURACY│
  ├────────────────────────┼───────────────────────┼─────────┤
  │ Linear extrapolation   │ Steady growth         │ Medium  │
  │ y = mx + b             │ services              │         │
  │                        │                       │         │
  │ Exponential growth     │ Fast-growing          │ Medium  │
  │ y = a × e^(rt)         │ startups              │         │
  │                        │                       │         │
  │ Seasonal decomposition │ E-commerce, media     │ High    │
  │ trend + seasonal +     │ services with         │         │
  │ residual               │ periodic patterns     │         │
  │                        │                       │         │
  │ Regression analysis    │ Multi-variable        │ High    │
  │ y = b₀ + b₁x₁ + b₂x₂ │ dependencies          │         │
  │                        │                       │         │
  │ Business-driven        │ Known upcoming events │ Highest │
  │ (product roadmap,      │ (launches, campaigns, │         │
  │ marketing calendar)    │ seasonal sales)       │         │
  └────────────────────────┴───────────────────────┴─────────┘
```

### Forecasting Example

```
  Current: 10,000 RPS  (January)
  Growth:  15% month-over-month
  
  ┌──────────────────────────────────────────────────────────┐
  │   RPS                                                    │
  │   ▲                                                      │
  │   │                                    ╱ 52,000 capacity │
  │ 50K│                          ╱──────╱  headroom         │
  │   │                     ╱                                │
  │ 40K│                ╱                                    │
  │   │           ╱  ← Provision point                      │
  │ 30K│      ╱       (need 90 days lead time)              │
  │   │  ╱                                                   │
  │ 20K│╱                                                    │
  │   │                                                      │
  │ 10K│● current                                            │
  │   │                                                      │
  │   └──────────────────────────────────────────────▶       │
  │   Jan  Mar  May  Jul  Sep  Nov                  Month    │
  │                                                          │
  │  Month    Projected RPS   Capacity needed (30% buffer)   │
  │  Jan      10,000          13,000                         │
  │  Apr      15,200          19,760                         │
  │  Jul      23,100          30,030                         │
  │  Oct      35,100          45,630                         │
  └──────────────────────────────────────────────────────────┘
```

---

## 4. Capacity Planning Buffer Strategy

```
  ┌──────────────────────────────────────────────────────────┐
  │  BUFFER = Headroom between current usage and capacity    │
  │                                                          │
  │  Why buffers:                                            │
  │  ├─ Traffic spikes (viral content, flash sales)          │
  │  ├─ Forecasting errors (growth faster than expected)     │
  │  ├─ Failure absorption (N+1 requires spare capacity)     │
  │  └─ Lead time coverage (provisioning isn't instant)      │
  │                                                          │
  │  Recommended buffers:                                    │
  │  ┌──────────────────┬────────┬───────────────────────┐   │
  │  │ Category         │ Buffer │ Reason                │   │
  │  ├──────────────────┼────────┼───────────────────────┤   │
  │  │ Auto-scaling     │ 20-30% │ Scale-up latency      │   │
  │  │ cloud workloads  │        │ (minutes)             │   │
  │  │                  │        │                       │   │
  │  │ Stateful services│ 40-50% │ Cannot scale quickly  │   │
  │  │ (databases)      │        │ (hours to days)       │   │
  │  │                  │        │                       │   │
  │  │ On-prem hardware │ 50-60% │ Procurement lead time │   │
  │  │                  │        │ (weeks to months)     │   │
  │  │                  │        │                       │   │
  │  │ Network/CDN      │ 30-40% │ Burst capacity needed │   │
  │  └──────────────────┴────────┴───────────────────────┘   │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Prometheus Capacity Queries

```promql
# CPU: Days until 100% at current growth rate
predict_linear(
  avg(rate(node_cpu_seconds_total{mode!="idle"}[1h]))[30d:1h],
  86400 * 30  # 30 days projection
)

# Disk: Days until 90% full
(
  node_filesystem_avail_bytes{mountpoint="/"}
  /
  deriv(node_filesystem_avail_bytes{mountpoint="/"}[7d])
) / -86400
# Result in days (negative deriv means shrinking space)

# RPS trend: 30-day projection
predict_linear(
  sum(rate(http_requests_total[5m]))[30d:5m],
  86400 * 90  # 90 days projection
)

# Memory utilization trend
predict_linear(
  (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes)
  / node_memory_MemTotal_bytes
  [30d:1h],
  86400 * 30
)
```

---

## 6. Capacity Planning Document Template

```yaml
# Quarterly Capacity Plan
service: "payment-service"
period: "Q2 2024"
author: "SRE Team"
last_updated: "2024-03-15"

current_state:
  instances: 6
  instance_type: "m5.xlarge"
  avg_cpu_util: 45%
  peak_cpu_util: 72%
  avg_rps: 5000
  peak_rps: 8000
  max_tested_rps: 12000  # From load test
  
growth_projection:
  monthly_growth_rate: 12%
  known_events:
    - event: "Summer sale"
      date: "2024-07-15"
      estimated_multiplier: 3x
    - event: "New market launch"
      date: "2024-06-01"
      estimated_increase: 20%

forecast:
  - quarter: "Q2 2024"
    projected_peak_rps: 11200
    required_instances: 8
    buffer_instances: 2  # N+2 for 2 AZ failure
    total_instances: 10
    
  - quarter: "Q3 2024"
    projected_peak_rps: 18000  # Includes summer sale
    required_instances: 12
    buffer_instances: 3
    total_instances: 15

actions:
  - action: "Scale to 10 instances by May 1"
    owner: "Alice"
    cost_impact: "+$2,400/month"
    
  - action: "Load test at 18K RPS before June 15"
    owner: "Bob"
    
  - action: "Evaluate instance type upgrade (m5.2xlarge)"
    owner: "Charlie"
    deadline: "2024-04-15"
```

---

## 7. Real-World Scenario

### [SCENARIO] Black Friday Capacity Planning

```
  E-commerce platform preparing for Black Friday
  
  Historical data:
  ├─ Normal daily peak: 10,000 RPS
  ├─ Last Black Friday peak: 45,000 RPS (4.5x)
  ├─ Growth since last year: 30%
  ├─ Expected peak: 45,000 × 1.3 = 58,500 RPS
  ├─ With 30% buffer: 76,000 RPS required capacity
  
  Capacity plan:
  ┌──────────────────────────────────────────────────┐
  │ Layer          │ Normal │ Black Friday │ Action   │
  ├────────────────┼────────┼──────────────┼──────────┤
  │ Web servers    │ 10     │ 40           │ Auto-scale│
  │ API servers    │ 8      │ 32           │ Pre-scale│
  │ DB read replica│ 2      │ 6            │ Add 4    │
  │ Cache (Redis)  │ 3-node │ 6-node       │ Scale    │
  │ CDN            │ 50 Gbps│ 200 Gbps     │ Reserve  │
  └────────────────┴────────┴──────────────┴──────────┘
  
  Timeline:
  T-8 weeks: Load test at 76K RPS  → found DB bottleneck
  T-6 weeks: DB optimization + replica scaling
  T-4 weeks: Re-test at 76K RPS   → passed
  T-2 weeks: Pre-scale API servers and cache
  T-1 week:  Final validation, war room schedule set
  T-0:       Black Friday → peak at 52K RPS (within plan)
  
  Lesson: Always plan for more than you think you need.
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Growth is unpredictable" | Use multiple methods (linear + business-driven). Plan for 2x the optimistic forecast. |
| "We always over-provision" | Track forecast accuracy. Use cloud auto-scaling for variable workloads. |
| "Database is the bottleneck" | Databases can't auto-scale easily. Plan 6-month ahead with 40-50% buffer. |
| "We don't know our system's capacity" | Load test! You can't plan capacity without knowing current limits. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Capacity Forecasting** | Predicting future resource needs |
| **Linear Extrapolation** | Simple growth projection (y = mx + b) |
| **Seasonal Decomposition** | Accounts for periodic patterns |
| **Buffer** | Headroom above forecast (20-60% depending on type) |
| **Lead Time** | Time to provision new capacity |
| **predict_linear** | Prometheus function for trend projection |
| **N+1 / N+2** | Spare capacity for failure absorption |

---

## Quick Revision Questions

1. **What are the six steps of capacity planning?**
   > Measure (current utilization), Forecast (project demand), Plan (determine what to add), Execute (provision), Validate (load test), Monitor (track actual vs forecast).

2. **Why do databases need larger capacity buffers than web servers?**
   > Databases are stateful and can't auto-scale quickly. Adding replicas takes hours to days for data sync. Web servers are stateless and can scale in minutes.

3. **What is the purpose of `predict_linear` in Prometheus?**
   > It projects a time series into the future based on its recent trend. Used to predict when resources will be exhausted (e.g., disk full in X days).

4. **How do you calculate capacity needs for a known traffic spike?**
   > Normal peak × expected multiplier × (1 + growth since last event) × (1 + buffer). Example: 10K × 4.5 × 1.3 × 1.3 = 76K RPS.

5. **What's the difference between auto-scaling and capacity planning?**
   > Auto-scaling handles short-term demand changes (minutes). Capacity planning is strategic (months) — ensuring infrastructure, databases, and budgets can handle projected growth.

---

[← Previous: Learning from Failures](../04-Incident-Management/06-learning-from-failures.md) | [Back to README](../README.md) | [Next: Load Testing →](02-load-testing.md)
