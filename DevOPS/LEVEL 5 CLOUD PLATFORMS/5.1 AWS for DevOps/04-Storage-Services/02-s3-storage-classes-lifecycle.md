# Chapter 4.2: S3 Storage Classes & Lifecycle Policies

## Overview

S3 offers multiple **storage classes** optimized for different access patterns and costs. **Lifecycle policies** automate transitioning objects between classes and expiring old data, which is critical for cost optimization in DevOps.

---

## Storage Classes Comparison

```
                        Cost vs Access Speed
   
   Cost $/GB/mo                 Access Latency
   ──────────                   ──────────────
   $0.023  Standard             Milliseconds   ◄── Hot data
   $0.0125 Standard-IA          Milliseconds   ◄── Infrequent
   $0.01   One Zone-IA          Milliseconds   ◄── Single AZ
   $0.0036 Glacier IR           Milliseconds   ◄── Quarterly
   $0.004  Glacier Flexible     Minutes-Hours  ◄── Archive
   $0.00099 Glacier Deep        12-48 Hours    ◄── Cold archive
   
   Intelligent-Tiering: Auto-moves between tiers ($0.0025/1000 obj monitoring)
   
   ┌─────────────────────────────────────────────────┐
   │  Rule of thumb:                                  │
   │  • Accessed daily      → Standard                │
   │  • Accessed monthly    → Standard-IA             │
   │  • Accessed quarterly  → Glacier Instant Retr.   │
   │  • Accessed yearly     → Glacier Flexible        │
   │  • Compliance archive  → Glacier Deep Archive    │
   │  • Unknown pattern     → Intelligent-Tiering     │
   └─────────────────────────────────────────────────┘
```

### Detailed Comparison

| Storage Class | Durability | AZs | Min Duration | Retrieval Fee | Use Case |
|--------------|-----------|-----|-------------|---------------|----------|
| **Standard** | 11 nines | ≥3 | None | None | Active data, websites |
| **Intelligent-Tiering** | 11 nines | ≥3 | None | None | Unknown access patterns |
| **Standard-IA** | 11 nines | ≥3 | 30 days | Per GB | Backups, DR |
| **One Zone-IA** | 11 nines | 1 | 30 days | Per GB | Re-creatable data |
| **Glacier Instant** | 11 nines | ≥3 | 90 days | Per GB | Quarterly access archives |
| **Glacier Flexible** | 11 nines | ≥3 | 90 days | Per GB + request | Yearly access archives |
| **Glacier Deep** | 11 nines | ≥3 | 180 days | Per GB + request | 7-10 year compliance |

> **Note:** "Min Duration" means you are charged for at least that many days even if you delete earlier.

---

## Intelligent-Tiering (Auto-Optimization)

```
┌─────── S3 Intelligent-Tiering ──────────────────────────┐
│                                                          │
│   Object uploaded                                        │
│       │                                                  │
│       ▼                                                  │
│   ┌────────────────┐                                     │
│   │ Frequent Access │  ◄── Default tier (Standard price) │
│   │ Tier            │                                    │
│   └───────┬────────┘                                     │
│           │ Not accessed for 30 days                     │
│           ▼                                              │
│   ┌────────────────┐                                     │
│   │ Infrequent     │  ◄── ~40% cheaper                  │
│   │ Access Tier    │                                     │
│   └───────┬────────┘                                     │
│           │ Not accessed for 90 days (opt-in)            │
│           ▼                                              │
│   ┌────────────────┐                                     │
│   │ Archive Instant│  ◄── ~68% cheaper                  │
│   │ Access Tier    │                                     │
│   └───────┬────────┘                                     │
│           │ Not accessed for 180 days (opt-in)           │
│           ▼                                              │
│   ┌────────────────┐                                     │
│   │ Deep Archive   │  ◄── ~95% cheaper                  │
│   │ Access Tier    │                                     │
│   └────────────────┘                                     │
│                                                          │
│   ⚠ No retrieval fees   ⚠ Auto-moves back on access    │
│   ⚠ Small monitoring fee ($0.0025/1000 objects/mo)      │
└──────────────────────────────────────────────────────────┘
```

---

## Lifecycle Policies

```
┌─── Lifecycle Rule: artifact-lifecycle ───────────────────┐
│                                                           │
│  Scope: Prefix "builds/"                                  │
│                                                           │
│  Day 0    ──► Standard           (active builds)          │
│               │                                           │
│  Day 30   ──► Standard-IA       (older builds)            │
│               │                                           │
│  Day 90   ──► Glacier Flexible   (archive)                │
│               │                                           │
│  Day 365  ──► Glacier Deep       (compliance)             │
│               │                                           │
│  Day 730  ──► DELETE             (2 years, expire)        │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐ │
│  │ Separately manage previous versions:                │ │
│  │  Day 0   → Previous version created                 │ │
│  │  Day 30  → Transition to Glacier Flexible           │ │
│  │  Day 90  → DELETE previous versions                 │ │
│  └─────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────┘
```

### Lifecycle Policy CLI

```bash
# Create lifecycle policy
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-devops-artifacts-2024 \
  --lifecycle-configuration '{
    "Rules": [
      {
        "ID": "ArchiveBuildArtifacts",
        "Status": "Enabled",
        "Filter": {
          "Prefix": "builds/"
        },
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
          "Days": 730
        },
        "NoncurrentVersionTransitions": [
          {
            "NoncurrentDays": 30,
            "StorageClass": "GLACIER"
          }
        ],
        "NoncurrentVersionExpiration": {
          "NoncurrentDays": 90
        }
      },
      {
        "ID": "CleanupIncompleteUploads",
        "Status": "Enabled",
        "Filter": {
          "Prefix": ""
        },
        "AbortIncompleteMultipartUpload": {
          "DaysAfterInitiation": 7
        }
      }
    ]
  }'

# View current lifecycle rules
aws s3api get-bucket-lifecycle-configuration \
  --bucket my-devops-artifacts-2024
```

---

## S3 Replication

```
┌──── Cross-Region Replication (CRR) ──────────────────────┐
│                                                           │
│  Source Bucket (us-east-1)     Dest Bucket (eu-west-1)   │
│  ┌─────────────────────┐     ┌─────────────────────┐    │
│  │ builds/v1/app.zip   │────►│ builds/v1/app.zip   │    │
│  │ logs/2024/01/        │────►│ logs/2024/01/        │    │
│  │                      │     │                      │    │
│  │ Versioning: ENABLED  │     │ Versioning: ENABLED  │    │
│  └─────────────────────┘     └─────────────────────┘    │
│                                                           │
│  Requirements:                                            │
│  ✓ Versioning enabled on BOTH buckets                    │
│  ✓ IAM role with replication permissions                 │
│  ✓ Can replicate to different account                    │
│  ✓ Can change storage class in destination               │
│                                                           │
│  Use cases: DR, compliance, latency reduction            │
└───────────────────────────────────────────────────────────┘

Same-Region Replication (SRR):
  • Log aggregation across accounts
  • Live replication between prod/test
  • Data sovereignty compliance
```

---

## Cost Optimization Decision Matrix

```
┌─────────────────────────────────────────────────────────────┐
│                    S3 Cost Factors                           │
│                                                             │
│  1. Storage cost/GB/month  (varies by class)                │
│  2. Request cost           (PUT/GET/LIST per 1,000)         │
│  3. Data transfer          (egress = costs, ingress = free) │
│  4. Retrieval cost         (IA and Glacier classes)         │
│  5. Monitoring cost        (Intelligent-Tiering only)       │
│                                                             │
│  Optimization Strategies:                                   │
│  ├── Use lifecycle rules for predictable patterns           │
│  ├── Use Intelligent-Tiering for unknown patterns           │
│  ├── Clean up incomplete multipart uploads                  │
│  ├── Delete old versions with lifecycle rules               │
│  ├── Use S3 Storage Lens for usage analytics                │
│  └── Use VPC endpoints to avoid data transfer costs         │
└─────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Lifecycle transition not working | Rule status "Disabled" or wrong prefix | Check rule is Enabled; verify prefix matches objects |
| Unexpected charges after delete | Min duration charge (30/90/180 days) | Only transition if objects will stay long enough |
| Can't transition to One Zone-IA | Coming from Glacier class | Can't move from Glacier back to One Zone-IA; only downward transitions |
| Replication not working | Versioning not enabled | Enable versioning on both source and destination |
| High retrieval costs | Glacier Flexible for frequent access | Use Glacier Instant Retrieval or Intelligent-Tiering instead |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Standard** | Default. No minimum duration. Best for frequently accessed data. |
| **Standard-IA** | 30-day minimum. Cheaper storage, retrieval fee. Infrequent access. |
| **One Zone-IA** | Single AZ. Cheapest IA. Only for re-creatable data. |
| **Glacier Instant** | 90-day min. Millisecond retrieval. Quarterly access. |
| **Glacier Flexible** | 90-day min. Minutes to hours retrieval. Yearly access. |
| **Glacier Deep** | 180-day min. 12-48 hour retrieval. Compliance archives. |
| **Intelligent-Tiering** | Auto-moves data. No retrieval fees. Small monitoring fee. |
| **Lifecycle** | Automate transitions and expiration. Clean up old versions. |
| **Replication** | CRR (cross-region) or SRR (same-region). Requires versioning. |

---

## Quick Revision Questions

1. **What is the minimum storage duration charge for Standard-IA?**
   > 30 days. If you delete an object before 30 days, you're still charged for 30 days.

2. **When should you use Intelligent-Tiering vs lifecycle policies?**
   > Intelligent-Tiering when access patterns are unpredictable. Lifecycle policies when you know the access pattern upfront.

3. **What is required for S3 replication?**
   > Versioning must be enabled on both source and destination buckets.

4. **What is the cheapest S3 storage class?**
   > Glacier Deep Archive (~$0.00099/GB/month), but retrieval takes 12-48 hours.

5. **How do you clean up incomplete multipart uploads?**
   > Add a lifecycle rule with `AbortIncompleteMultipartUpload` action.

---

[← Previous: 4.1 S3 Overview](01-s3-simple-storage-service.md) | [Next: 4.3 EBS Volumes →](03-ebs-elastic-block-store.md)

[← Back to README](../README.md)
