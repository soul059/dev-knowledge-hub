# Chapter 5.5: Cost Optimization

[← Previous: Scaling Strategies](04-scaling-strategies.md) | [Back to README](../README.md) | [Next: What is Toil →](../06-Automation-and-Toil/01-what-is-toil.md)

---

## Overview

Cost optimization in SRE balances reliability with financial efficiency. Over-provisioning wastes money; under-provisioning causes outages. Effective cost management requires understanding workload patterns, right-sizing resources, leveraging pricing models, and continuously monitoring spend.

---

## 1. The Cost-Reliability Tradeoff

```
  ┌──────────────────────────────────────────────────────────┐
  │  Cost ($)                                                │
  │  │                                            ╱          │
  │  │                                          ╱            │
  │  │                                       ╱               │
  │  │                              ────────╱  (diminishing  │
  │  │                    ─────────╱           returns)       │
  │  │           ────────╱                                    │
  │  │  ────────╱                                             │
  │  │                                                        │
  │  └────────────────────────────────────────────▶           │
  │    99%   99.5%  99.9%  99.95% 99.99% 99.999%             │
  │                   Availability                            │
  │                                                          │
  │  KEY INSIGHT:                                             │
  │  Each additional "9" roughly 10x the cost               │
  │  99.9% → 99.99% costs far more than 99% → 99.9%        │
  │                                                          │
  │  SRE role: Find the optimal point where marginal cost   │
  │  of reliability equals marginal cost of downtime         │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Cloud Cost Anatomy

```
  ┌──────────────────────────────────────────────────────────┐
  │  TYPICAL CLOUD SPEND BREAKDOWN                           │
  │                                                          │
  │  Compute (EC2/VMs/GKE)  ████████████████  40-60%        │
  │  Storage (S3/EBS/GCS)   ████████          15-25%        │
  │  Database (RDS/Cloud SQL)██████           10-20%        │
  │  Network (egress/LB)    ████              5-15%         │
  │  Other (DNS/logs/misc)  ██                3-8%          │
  │                                                          │
  │  Top Cost Drivers:                                       │
  │  ├─ Over-provisioned instances (idle capacity)           │
  │  ├─ Unattached/orphaned resources                        │
  │  ├─ Cross-region data transfer                           │
  │  ├─ Premium storage tiers for cold data                  │
  │  └─ On-demand pricing for predictable workloads          │
  └──────────────────────────────────────────────────────────┘
```

---

## 3. Right-Sizing Resources

```yaml
# Right-sizing analysis workflow
right_sizing:
  # Step 1: Collect metrics (2-4 week window minimum)
  metrics:
    cpu:
      avg_utilization: 15%     # Way too low
      p95_utilization: 35%     # Even peak is low
      current_instance: "m5.2xlarge"   # 8 vCPU, 32GB
    memory:
      avg_utilization: 40%
      p95_utilization: 55%
    
  # Step 2: Recommendation
  recommendation:
    current: "m5.2xlarge"      # $0.384/hr = $280/mo
    recommended: "m5.large"     # $0.096/hr = $70/mo
    savings: "75%"              # $210/mo per instance
    risk: "low"                 # p95 CPU still under 70%
    action: "downsize"
    
  # Step 3: Validation
  validation:
    method: "canary_deployment"
    monitor_period: "72h"
    rollback_trigger: "p99_latency > 200ms"
```

**Right-sizing rules of thumb:**

| Metric | Action |
|--------|--------|
| CPU avg < 20%, p95 < 40% | Downsize by 50% |
| CPU avg 20-40%, p95 < 70% | Downsize by 25% |
| CPU avg 40-70%, p95 < 85% | Correct size |
| CPU avg > 70% or p95 > 85% | Upsize needed |
| Memory avg < 30% | Consider smaller tier |

---

## 4. Pricing Models

```
  ┌──────────────────────────────────────────────────────────┐
  │  MODEL            │ DISCOUNT │ COMMITMENT │ BEST FOR     │
  ├───────────────────┼──────────┼────────────┼──────────────┤
  │ On-Demand         │ 0%       │ None       │ Spiky/unknown│
  │                   │          │            │ workloads    │
  │                   │          │            │              │
  │ Reserved (1yr)    │ 30-40%   │ 1 year     │ Steady-state │
  │                   │          │            │ base load    │
  │                   │          │            │              │
  │ Reserved (3yr)    │ 50-60%   │ 3 years    │ Well-known   │
  │                   │          │            │ core infra   │
  │                   │          │            │              │
  │ Spot/Preemptible  │ 60-90%   │ None       │ Fault-       │
  │                   │          │ (can be    │ tolerant,    │
  │                   │          │ reclaimed) │ batch jobs   │
  │                   │          │            │              │
  │ Savings Plans     │ 20-50%   │ $/hr commit│ Flexible     │
  │                   │          │ 1 or 3 yr  │ compute use  │
  │                   │          │            │              │
  │ Sustained Use     │ 20-30%   │ Automatic  │ GCP VMs used │
  │ (GCP)             │          │            │ > 25% month  │
  └───────────────────┴──────────┴────────────┴──────────────┘
  
  OPTIMAL STRATEGY (layered approach):
  ┌──────────────────────────────────────────────────────────┐
  │  Capacity │                                              │
  │  ▲        │     ░░░░░░░░░░░░░░░░░ Spot/On-Demand        │
  │  │        │   ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ On-Demand (burst)    │
  │  │  ──────┤ ═══════════════════════ Reserved (growth)    │
  │  │  ══════│═══════════════════════ Reserved (base)       │
  │  │        │                                              │
  │  └────────┴──────────────────────────▶ Time              │
  │                                                          │
  │  Base load (60-70%) → Reserved instances                 │
  │  Growth buffer (15-20%) → Savings Plans                  │
  │  Burst capacity (10-20%) → On-Demand + Spot              │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Storage Cost Optimization

```
  ┌──────────────────────────────────────────────────────────┐
  │  DATA LIFECYCLE MANAGEMENT                               │
  │                                                          │
  │  Hot Data     ──(30d)──▶   Warm Data                     │
  │  (S3 Standard)           (S3 Infrequent Access)          │
  │  $0.023/GB               $0.0125/GB                      │
  │       │                       │                          │
  │       │                  (90d)│                           │
  │       │                       ▼                          │
  │       │               Cold Data                          │
  │       │               (S3 Glacier)                       │
  │       │               $0.004/GB                          │
  │       │                       │                          │
  │       │                  (365d)│                          │
  │       │                       ▼                          │
  │       │               Archive                            │
  │       │               (Glacier Deep Archive)             │
  │       │               $0.00099/GB                        │
  │       │                       │                          │
  │       │                  (7yr)│                           │
  │       │                       ▼                          │
  │       │                   DELETE                          │
  │                                                          │
  │  Savings: Moving 100TB from Standard → IA saves          │
  │  ~$1,050/month. To Glacier saves ~$1,900/month.         │
  └──────────────────────────────────────────────────────────┘
```

```json
// S3 Lifecycle Policy
{
  "Rules": [
    {
      "ID": "data-lifecycle",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ],
      "Expiration": {
        "Days": 2555
      }
    }
  ]
}
```

---

## 6. Cost Monitoring and FinOps Practices

```
  ┌──────────────────────────────────────────────────────────┐
  │  FinOps LIFECYCLE                                        │
  │                                                          │
  │  ┌──────────┐    ┌──────────┐    ┌──────────┐           │
  │  │ INFORM   │───▶│ OPTIMIZE │───▶│ OPERATE  │           │
  │  │          │    │          │    │          │            │
  │  │ • Tagging│    │ • Right- │    │ • Budget │           │
  │  │ • Alloc. │    │   sizing │    │   alerts │           │
  │  │ • Report │    │ • RI/SP  │    │ • Anomaly│           │
  │  │ • Showbk │    │ • Spot   │    │   detect │           │
  │  └──────────┘    │ • Tiering│    │ • Review │           │
  │       ▲          └──────────┘    └────┬─────┘           │
  │       │                               │                  │
  │       └───────────────────────────────┘                  │
  │                  (continuous)                             │
  └──────────────────────────────────────────────────────────┘
```

### Tagging Strategy for Cost Allocation

```yaml
# Mandatory tags for every cloud resource
required_tags:
  - key: "team"
    description: "Owning team"
    values: ["platform", "payments", "search", "ml"]
  
  - key: "environment"
    description: "Deployment environment"
    values: ["production", "staging", "development"]
  
  - key: "service"
    description: "Service name"
    values: ["api-gateway", "user-service", "order-service"]
  
  - key: "cost-center"
    description: "Financial cost center"
    values: ["engineering", "data", "infrastructure"]

# Budget alerts
budgets:
  - name: "platform-team-monthly"
    amount: 50000
    alerts:
      - threshold: 50%    # $25K
        action: "notify_slack"
      - threshold: 80%    # $40K
        action: "notify_slack + email_manager"
      - threshold: 100%   # $50K
        action: "notify_all + restrict_new_resources"
      - threshold: 120%   # $60K
        action: "escalate_to_vp"
```

---

## 7. Cost Optimization Checklist

```
  ┌──────────────────────────────────────────────────────────┐
  │  QUICK WINS (implement first, biggest ROI):              │
  │                                                          │
  │  □ Delete unused/orphaned resources                      │
  │    ├─ Unattached EBS volumes                             │
  │    ├─ Old snapshots and AMIs                             │
  │    ├─ Idle load balancers                                │
  │    └─ Unused Elastic IPs                                 │
  │                                                          │
  │  □ Right-size over-provisioned instances                 │
  │    ├─ Instances with <20% avg CPU                        │
  │    └─ Databases with <30% avg connections                │
  │                                                          │
  │  □ Purchase Reserved Instances for base load             │
  │    └─ Anything running 24/7 → RI or Savings Plan        │
  │                                                          │
  │  □ Implement storage lifecycle policies                  │
  │    └─ Move old data to cheaper tiers automatically       │
  │                                                          │
  │  □ Use Spot instances for fault-tolerant workloads       │
  │    ├─ CI/CD build agents                                 │
  │    ├─ Data processing / ETL                              │
  │    └─ Test environments                                  │
  │                                                          │
  │  □ Schedule non-production environments                  │
  │    └─ Dev/staging OFF outside business hours (save ~65%) │
  │                                                          │
  │  □ Optimize data transfer                                │
  │    ├─ Use VPC endpoints (avoid NAT Gateway costs)        │
  │    ├─ Compress API responses                             │
  │    └─ Keep traffic within same AZ when possible          │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] Reducing Cloud Spend by 40%

```
  SITUATION: SaaS company spending $200K/month on AWS
  TARGET: Reduce to <$120K/month without impacting reliability
  
  Phase 1 — Low-hanging fruit (Week 1-2):
  ├─ Deleted 47 orphaned EBS volumes → saved $2,800/mo
  ├─ Removed 12 idle dev environments → saved $8,500/mo
  ├─ Turned off staging overnight/weekends → saved $4,200/mo
  └─ Subtotal: $15,500/mo (8% reduction)
  
  Phase 2 — Right-sizing (Week 3-4):
  ├─ Downsized 85 over-provisioned instances → saved $22,000/mo
  ├─ Switched 6 RDS instances to smaller types → saved $8,400/mo
  ├─ Moved to Graviton (ARM) instances → saved $6,000/mo
  └─ Subtotal: $36,400/mo (18% reduction)
  
  Phase 3 — Pricing optimization (Week 5-8):
  ├─ Purchased 1yr RIs for base load → saved $18,000/mo
  ├─ Savings Plans for variable compute → saved $7,500/mo
  ├─ Spot instances for CI/CD → saved $5,200/mo
  └─ Subtotal: $30,700/mo (15% reduction)
  
  Phase 4 — Architecture (Week 9-12):
  ├─ S3 lifecycle policies → saved $3,800/mo
  ├─ CloudFront caching (reduced egress) → saved $2,100/mo
  ├─ VPC endpoints (no NAT Gateway) → saved $1,500/mo
  └─ Subtotal: $7,400/mo (4% reduction)
  
  TOTAL SAVINGS: $90,000/month (45% reduction!)
  Final spend: $110,000/month
  Reliability impact: None (SLO maintained at 99.95%)
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Costs spiked suddenly" | Check for untagged resources, run cost anomaly detection, review recent deployments. |
| "Reserved Instances are underutilized" | Review RI coverage reports. Consider exchangeable RIs or Savings Plans next cycle. |
| "Can't reduce costs without impacting performance" | Focus on waste first (orphaned resources). Right-size using p95 metrics, not averages. |
| "Teams overprovision 'just in case'" | Implement auto-scaling with proper HPA. Show teams their cost-per-request metrics. |
| "Storage costs keep growing" | Audit data retention policies. Implement lifecycle rules. Compress or deduplicate data. |

---

## Summary Table

| Technique | Typical Savings | Effort | Risk |
|-----------|----------------|--------|------|
| **Delete orphaned resources** | 5-10% | Low | None |
| **Right-sizing** | 15-30% | Medium | Low (monitor after) |
| **Reserved Instances** | 30-60% on compute | Medium | Commit risk |
| **Spot Instances** | 60-90% on compute | Medium | Interruption |
| **Storage tiering** | 40-70% on storage | Low | Access latency |
| **Schedule non-prod** | 60-70% on dev/staging | Low | None |
| **Graviton/ARM** | 20-30% | Medium | Compatibility |

---

## Quick Revision Questions

1. **What is the cost-reliability tradeoff in SRE?**
   > Each additional "nine" of availability roughly 10x the cost. SRE finds the optimal point where the marginal cost of more reliability exceeds the marginal cost of downtime.

2. **Describe the layered pricing strategy for cloud compute.**
   > Base load (60-70%) → Reserved Instances for maximum discount. Growth buffer (15-20%) → Savings Plans for flexibility. Burst capacity (10-20%) → On-Demand + Spot instances.

3. **What are the first things to check when trying to reduce cloud costs?**
   > Orphaned/unused resources (unattached volumes, idle instances), over-provisioned instances (CPU avg <20%), and non-production environments running 24/7.

4. **How does storage lifecycle management save money?**
   > Automatically transitions data from expensive hot tiers (Standard) to cheaper cold tiers (IA → Glacier → Deep Archive) as it ages and is accessed less frequently.

5. **What is right-sizing and how do you validate it?**
   > Analyzing actual resource utilization (CPU, memory, network) over 2-4 weeks and matching instance size to actual need. Validate using canary deployments and monitoring p99 latency during the transition.

6. **Why is tagging important for cost management?**
   > Tags enable cost allocation to specific teams/services/environments. Without tags, you can't determine who is spending what, making accountability and optimization impossible.

---

[← Previous: Scaling Strategies](04-scaling-strategies.md) | [Back to README](../README.md) | [Next: What is Toil →](../06-Automation-and-Toil/01-what-is-toil.md)
