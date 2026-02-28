# Chapter 2.2: EC2 Instance Types and Sizing

## Overview

Choosing the right instance type is one of the most impactful decisions for **cost** and **performance**. AWS offers hundreds of instance types optimized for different workloads. This chapter covers the naming convention, families, and how to right-size.

---

## Instance Type Naming Convention

```
    m  5  .  2xlarge
    │  │     │
    │  │     └── Size (nano → metal)
    │  │
    │  └── Generation (higher = newer, better price/performance)
    │
    └── Family (m = general purpose)

    Optional suffixes:
    m5a  → AMD processors (a = AMD, cheaper)
    m5g  → Graviton/ARM processors (g = Graviton, best price/perf)
    m5d  → Instance store (d = local NVMe SSD)
    m5n  → Enhanced networking (n = network optimized)
    m5zn → High frequency Intel (z = high frequency)
```

---

## Instance Families

```
┌──────────────────────────────────────────────────────────────────┐
│                    EC2 INSTANCE FAMILIES                         │
│                                                                  │
│  GENERAL PURPOSE (Balanced)                                      │
│  ┌──────┬──────┬──────┐                                         │
│  │  T3  │  M5  │  M6g │  Equal CPU:Memory ratio                │
│  │Burst │Steady│Gravtn│  Web servers, dev/test, small DBs       │
│  └──────┴──────┴──────┘                                         │
│                                                                  │
│  COMPUTE OPTIMIZED (CPU-heavy)                                   │
│  ┌──────┬──────┐                                                │
│  │  C5  │  C6g │  High CPU:Memory ratio                        │
│  │Intel │Gravtn│  Batch, ML inference, gaming, HPC             │
│  └──────┴──────┘                                                │
│                                                                  │
│  MEMORY OPTIMIZED (RAM-heavy)                                    │
│  ┌──────┬──────┬──────┐                                         │
│  │  R5  │  X1  │  z1d │  High Memory:CPU ratio                 │
│  │ RAM  │  TB  │ Fast │  In-memory DBs, caching, analytics     │
│  └──────┴──────┴──────┘                                         │
│                                                                  │
│  STORAGE OPTIMIZED (Disk-heavy)                                  │
│  ┌──────┬──────┬──────┐                                         │
│  │  I3  │  D2  │  H1  │  High sequential/random I/O            │
│  │NVMe  │ HDD  │ HDD  │  Data warehousing, Hadoop, log proc    │
│  └──────┴──────┴──────┘                                         │
│                                                                  │
│  ACCELERATED COMPUTING (GPU/FPGA)                                │
│  ┌──────┬──────┬──────┐                                         │
│  │  P4  │  G5  │  Inf1│  GPU/custom hardware                   │
│  │Train │Render│Infer │  ML training, video, graphics          │
│  └──────┴──────┴──────┘                                         │
└──────────────────────────────────────────────────────────────────┘
```

### Instance Family Reference Table

| Family | Prefix | Optimized For | Example Use Cases |
|--------|--------|---------------|-------------------|
| **General Purpose** | T, M, A | Balanced CPU/Memory | Web apps, dev/test, microservices |
| **Compute Optimized** | C | High-performance CPU | Batch processing, gaming, scientific |
| **Memory Optimized** | R, X, z | Large memory footprint | Redis, SAP HANA, in-memory DBs |
| **Storage Optimized** | I, D, H | High I/O throughput | Data warehousing, HDFS, search |
| **Accelerated** | P, G, Inf, DL | GPU/AI hardware | ML training, video encoding, 3D |

---

## Instance Sizes

```
┌───────────────────────────────────────────────────────────┐
│            INSTANCE SIZES (t3 family example)             │
│                                                           │
│  Size         vCPU   Memory   Network      Storage       │
│  ──────────── ────── ──────── ──────────── ────────────  │
│  t3.nano       2     0.5 GiB  Up to 5 Gbps  EBS only   │
│  t3.micro      2     1 GiB    Up to 5 Gbps  EBS only   │
│  t3.small      2     2 GiB    Up to 5 Gbps  EBS only   │
│  t3.medium     2     4 GiB    Up to 5 Gbps  EBS only   │
│  t3.large      2     8 GiB    Up to 5 Gbps  EBS only   │
│  t3.xlarge     4     16 GiB   Up to 5 Gbps  EBS only   │
│  t3.2xlarge    8     32 GiB   Up to 5 Gbps  EBS only   │
│                                                           │
│  Each step up = DOUBLE the previous size's resources     │
│  Price roughly doubles with each size increase           │
└───────────────────────────────────────────────────────────┘
```

---

## T-Series: Burstable Instances

T-series instances are unique — they use a **CPU credit system**.

```
┌────────────────────────────────────────────────────────────────┐
│               T3 BURSTABLE CPU CREDITS                        │
│                                                                │
│  CPU Usage %                                                   │
│  100% ─ ─ ─ ─ ─ ─┐    ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─            │
│                    │    │                                       │
│                    │    │   ← Burst (uses credits)             │
│                    │    │                                       │
│  Baseline ────────┼────┼──────────────────────── ──── 20%     │
│  (e.g., 20%)      │    │                                       │
│                    └────┘                                       │
│                                                                │
│  Below baseline → Earn credits (banked)                       │
│  Above baseline → Spend credits (burst)                       │
│  Credits = 0    → Throttled to baseline                       │
│                                                                │
│  T3 Baselines:                                                 │
│  t3.nano    = 5%     t3.micro  = 10%                          │
│  t3.small   = 20%    t3.medium = 20%                          │
│  t3.large   = 30%    t3.xlarge = 40%                          │
│                                                                │
│  UNLIMITED MODE (default for t3):                             │
│  Can burst beyond credits — charged per vCPU-hour extra       │
└────────────────────────────────────────────────────────────────┘
```

```bash
# Check CPU credit balance
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUCreditBalance \
  --dimensions Name=InstanceId,Value=i-xxx \
  --start-time 2026-02-27T00:00:00Z \
  --end-time 2026-02-27T23:59:59Z \
  --period 3600 \
  --statistics Average
```

---

## Graviton Instances (ARM-based)

AWS Graviton processors offer **up to 40% better price/performance** vs x86.

```
┌──────────────────────────────────────────────────────┐
│              GRAVITON vs INTEL/AMD                    │
│                                                      │
│  x86 (Intel/AMD)        ARM (Graviton)              │
│  ────────────────       ──────────────               │
│  m5.xlarge              m6g.xlarge                   │
│  c5.2xlarge             c6g.2xlarge                  │
│  r5.large               r6g.large                   │
│                                                      │
│  Graviton Benefits:                                  │
│  ✓ ~20% cheaper than equivalent x86                 │
│  ✓ ~40% better price/performance                    │
│  ✓ Lower power consumption                          │
│                                                      │
│  Graviton Requirements:                              │
│  • Application must support ARM/aarch64              │
│  • Most Linux software works out of the box         │
│  • Docker: build multi-arch images                  │
│  • Some Windows workloads NOT supported             │
└──────────────────────────────────────────────────────┘
```

---

## Right-Sizing Guide

```
┌─────────────────────────────────────────────────────────────┐
│              RIGHT-SIZING DECISION FLOW                     │
│                                                             │
│  Avg CPU < 10% ──────► Downsize or switch to Lambda        │
│                                                             │
│  Avg CPU 10-40% ─────► Right size. Consider T-series.      │
│                                                             │
│  Avg CPU 40-70% ─────► Well-sized. Monitor peaks.          │
│                                                             │
│  Avg CPU > 80% ──────► Upsize or add more instances        │
│                                                             │
│  Memory > 80% used ──► Switch to R-series (memory opt)     │
│                                                             │
│  Network constrained ► Upgrade to larger size or           │
│                        n-suffix (enhanced networking)       │
│                                                             │
│  TOOLS:                                                     │
│  • AWS Cost Explorer → Right-sizing recommendations        │
│  • AWS Compute Optimizer → ML-based suggestions            │
│  • CloudWatch → CPU, memory, network metrics               │
└─────────────────────────────────────────────────────────────┘
```

### CLI Commands for Right-Sizing

```bash
# Get CPU utilization over last 7 days
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-xxx \
  --start-time $(date -d '7 days ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date +%Y-%m-%dT%H:%M:%S) \
  --period 3600 \
  --statistics Average Maximum

# Get Compute Optimizer recommendations
aws compute-optimizer get-ec2-instance-recommendations \
  --instance-arns arn:aws:ec2:us-east-1:123456789012:instance/i-xxx
```

---

## Instance Type Selection Guide

| Workload | Recommended Family | Why |
|----------|-------------------|-----|
| Web server (low-traffic) | t3.small / t3.medium | Burstable, cost-effective |
| Web server (steady) | m6g.large | Graviton = best price/perf |
| API backend | m6g.xlarge | Balanced CPU/memory |
| CI/CD build agents | c6g.xlarge | CPU-intensive builds |
| Docker host | m5.xlarge | Good balance for containers |
| Redis/Memcached | r6g.large | Memory-optimized |
| PostgreSQL/MySQL | r5.xlarge | Memory for DB buffer pool |
| Machine Learning training | p4d.24xlarge | GPU-accelerated |
| Video encoding | c5.4xlarge | CPU-intensive |
| Data warehouse / Hadoop | i3.xlarge | Storage-optimized (NVMe) |
| Dev/test environment | t3.micro | Cheap, burstable |

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| App slow with T-series | CPU credits exhausted | Enable unlimited mode or upgrade to M-series |
| `InsufficientInstanceCapacity` | AZ can't fulfill instance type | Try different AZ, size, or use capacity reservations |
| High CPU but low utilization | Wrong instance family | Consider compute-optimized (C-series) |
| Application OOM killed | Not enough memory | Upgrade size or switch to R-series |
| Network bottleneck | Instance network bandwidth limit | Use larger size or enhanced networking |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Naming** | `[Family][Generation].[Size]` — e.g., m5.xlarge |
| **General Purpose** | T (burst), M (steady) — balanced CPU/memory |
| **Compute Optimized** | C — high CPU for batch, build, gaming |
| **Memory Optimized** | R, X — large memory for DBs, caching |
| **Storage Optimized** | I, D — high I/O for data-heavy workloads |
| **Graviton** | g suffix — ARM-based, 40% better price/performance |
| **T-series** | Burstable with CPU credits. Below baseline = earn, above = spend |
| **Right-sizing** | Use CloudWatch + Compute Optimizer. Target 40-70% CPU average |

---

## Quick Revision Questions

1. **What does `c6g.2xlarge` mean?**
   > c = Compute Optimized, 6 = 6th generation, g = Graviton (ARM), 2xlarge = size with 8 vCPU/16 GiB.

2. **When would you choose a T-series vs M-series instance?**
   > T-series when CPU usage is bursty (low average with occasional spikes). M-series when CPU usage is steady/sustained.

3. **What happens when a T3 instance runs out of CPU credits in unlimited mode?**
   > It continues to burst but you're charged extra per vCPU-hour at a surcharge rate.

4. **Why are Graviton instances recommended for most workloads?**
   > They offer up to 40% better price/performance than x86. Most Linux applications and Docker containers work without changes.

5. **What AWS tool provides ML-based right-sizing recommendations?**
   > AWS Compute Optimizer uses machine learning to analyze CloudWatch metrics and recommend optimal instance types.

6. **Which instance family would you pick for a Redis caching cluster?**
   > R-series (Memory Optimized) — e.g., r6g.large — since Redis is an in-memory data store.

---

[← Previous: 2.1 EC2 Instances](01-ec2-instances.md) | [Next: 2.3 AMIs and Launch Templates →](03-amis-and-launch-templates.md)

[← Back to README](../README.md)
