# Chapter 1.1: AWS Global Infrastructure

## Overview

Amazon Web Services (AWS) operates one of the largest and most robust cloud infrastructures in the world. Understanding the global infrastructure is foundational — it affects **availability**, **latency**, **compliance**, and **disaster recovery** decisions for every workload you deploy.

---

## Key Concepts

### What Makes Up AWS Global Infrastructure?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      AWS GLOBAL INFRASTRUCTURE                         │
│                                                                        │
│  ┌────────────────────────────────────────────────────────────────┐    │
│  │  REGIONS  (33+ worldwide)                                      │    │
│  │  ┌──────────────────────────────────────────────────────────┐  │    │
│  │  │  AVAILABILITY ZONES (AZs)  (3-6 per region)              │  │    │
│  │  │  ┌───────────┐  ┌───────────┐  ┌───────────┐            │  │    │
│  │  │  │ Data Ctr  │  │ Data Ctr  │  │ Data Ctr  │  ...       │  │    │
│  │  │  │ Cluster 1 │  │ Cluster 2 │  │ Cluster 3 │            │  │    │
│  │  │  └───────────┘  └───────────┘  └───────────┘            │  │    │
│  │  │      Connected via low-latency private fiber              │  │    │
│  │  └──────────────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────────────┘    │
│                                                                        │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐     │
│  │  EDGE LOCATIONS  │  │  LOCAL ZONES      │  │  WAVELENGTH      │     │
│  │  (450+ PoPs)     │  │  (City-level)     │  │  ZONES (5G)      │     │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘     │
└─────────────────────────────────────────────────────────────────────────┘
```

### Component Breakdown

| Component | Count | Purpose |
|-----------|-------|---------|
| **Regions** | 33+ | Geographically isolated clusters of data centers |
| **Availability Zones** | 105+ | Physically separated data centers within a region |
| **Edge Locations** | 450+ | CloudFront CDN and Route 53 DNS endpoints |
| **Local Zones** | 30+ | Extensions of regions to metro areas for ultra-low latency |
| **Wavelength Zones** | 30+ | Embedded in 5G carrier networks |
| **Outposts** | Custom | AWS infrastructure on-premises |

---

## Regions

A **Region** is a geographical area containing multiple isolated data center groups (AZs). Each region is **completely independent** and **isolated** from other regions for fault tolerance and stability.

### Naming Convention

```
Region Code:     us-east-1        ap-south-1        eu-west-2
                 ──┬────┬──       ──┬─────┬──       ──┬────┬──
                   │    │           │     │           │    │
              Geography │      Geography │       Geography │
                    Direction         Direction        Direction
```

### Major Regions

| Region Code | Location | Common Use |
|-------------|----------|------------|
| `us-east-1` | N. Virginia | Default region, most services launch here first |
| `us-west-2` | Oregon | Secondary US, cost-effective |
| `eu-west-1` | Ireland | European workloads |
| `eu-central-1` | Frankfurt | GDPR compliance |
| `ap-south-1` | Mumbai | Indian subcontinent |
| `ap-southeast-1` | Singapore | Southeast Asia |
| `ap-northeast-1` | Tokyo | Japan workloads |

### Choosing a Region — Decision Factors

```
┌─────────────────────────────────────────────────────────┐
│            HOW TO CHOOSE AN AWS REGION                   │
│                                                         │
│  1. COMPLIANCE ─────────> Data must stay in country?    │
│     │                     (GDPR, HIPAA, etc.)           │
│     ▼                                                   │
│  2. LATENCY ────────────> Close to your users?          │
│     │                     (Use ping tests)              │
│     ▼                                                   │
│  3. SERVICE AVAILABILITY > Service available in region? │
│     │                     (Not all regions have all)    │
│     ▼                                                   │
│  4. PRICING ────────────> Compare regional pricing      │
│                           (Can differ by 10-20%)        │
└─────────────────────────────────────────────────────────┘
```

### CLI Commands for Regions

```bash
# List all available regions
aws ec2 describe-regions --output table

# List regions with a specific service (e.g., EKS)
aws ec2 describe-regions \
  --filters "Name=opt-in-status,Values=opt-in-not-required,opted-in" \
  --query "Regions[].RegionName" --output text

# Check which region your CLI is configured for
aws configure get region
```

---

## Availability Zones (AZs)

Each Region has **at minimum 3 AZs** (some have up to 6). AZs are the building blocks of high availability on AWS.

### Key Properties

- Each AZ = **one or more discrete data centers**
- Connected via **high-bandwidth, low-latency private fiber**
- Physically separated (distinct power, cooling, networking)
- Designed so a **failure in one AZ does not cascade** to others

```
              Region: us-east-1 (N. Virginia)
    ┌──────────────────────────────────────────────┐
    │                                              │
    │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
    │  │  AZ-1a   │  │  AZ-1b   │  │  AZ-1c   │  │
    │  │ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────┐ │  │
    │  │ │ DC 1 │ │  │ │ DC 4 │ │  │ │ DC 7 │ │  │
    │  │ │ DC 2 │ │  │ │ DC 5 │ │  │ │ DC 8 │ │  │
    │  │ │ DC 3 │ │  │ │ DC 6 │ │  │ │ DC 9 │ │  │
    │  │ └──────┘ │  │ └──────┘ │  │ └──────┘ │  │
    │  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
    │       │             │             │         │
    │       └─────────────┼─────────────┘         │
    │          Low-latency private links           │
    └──────────────────────────────────────────────┘
```

### Best Practice: Multi-AZ Deployment

**Always deploy critical workloads across at least 2 AZs.**

```
                 ┌──────────────────────┐
                 │   Elastic Load       │
                 │   Balancer (ALB)     │
                 └──────────┬───────────┘
                    ┌───────┴───────┐
                    ▼               ▼
             ┌──────────┐   ┌──────────┐
             │  AZ-1a   │   │  AZ-1b   │
             │ ┌──────┐ │   │ ┌──────┐ │
             │ │ EC2  │ │   │ │ EC2  │ │
             │ │ App  │ │   │ │ App  │ │
             │ └──┬───┘ │   │ └──┬───┘ │
             │    │     │   │    │     │
             │ ┌──▼───┐ │   │ ┌──▼───┐ │
             │ │ RDS  │ │   │ │ RDS  │ │
             │ │Primary│◄──────►Standby│ │
             │ └──────┘ │   │ └──────┘ │
             └──────────┘   └──────────┘
```

---

## Edge Locations

Edge Locations are AWS's content delivery endpoints. They are **not** full regions — they run a subset of services:

- **Amazon CloudFront** — CDN caching
- **Amazon Route 53** — DNS resolution
- **AWS Shield** — DDoS protection
- **AWS WAF** — Web Application Firewall
- **Lambda@Edge** — Run code at edge locations

```
    User in Sydney                          Origin Server
         │                                  (us-east-1)
         │                                       │
         ▼                                       │
    ┌─────────────┐     Cache Miss          ┌────▼────┐
    │ Edge Location│ ─────────────────────> │   S3    │
    │   Sydney     │ <───────────────────── │  Bucket │
    │  (Cached)    │     Content Response   └─────────┘
    └─────────────┘
         │
         ▼
    Response to User
    (Low Latency!)
```

---

## Local Zones and Wavelength Zones

### Local Zones
- Extend a **region to a specific city** (e.g., `us-east-1-bos-1a` for Boston)
- Use case: **real-time gaming**, **media rendering**, **ML inference**
- Latency: **single-digit milliseconds** to local users

### Wavelength Zones
- AWS infrastructure embedded in **5G telecom networks**
- Use case: **connected vehicles**, **AR/VR**, **IoT at the edge**
- Deployed with carriers like Verizon, Vodafone, KDDI

---

## AWS Outposts

- **Fully managed AWS infrastructure** deployed **on-premises**
- Same APIs, tools, and hardware as in AWS regions
- Use case: **data residency**, **local data processing**, **low-latency hybrid**

```
    ┌──────────────────────────┐        ┌──────────────┐
    │   YOUR DATA CENTER       │        │  AWS REGION   │
    │  ┌────────────────────┐  │        │              │
    │  │   AWS OUTPOST      │  │◄──────►│  us-east-1   │
    │  │  (EC2, EBS, S3,    │  │  VPN/  │              │
    │  │   ECS, RDS, etc.)  │  │  DX    │              │
    │  └────────────────────┘  │        └──────────────┘
    └──────────────────────────┘
```

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Service not available in region | Not all services launch in all regions | Check AWS Regional Services List |
| High latency for users | Users far from chosen region | Add CloudFront or pick a closer region |
| Compliance violation | Data stored outside required geography | Use region-lock with IAM policies (e.g., `aws:RequestedRegion`) |
| Single-AZ failure impacts app | Not deployed across multiple AZs | Re-architect for multi-AZ |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Region** | Geographically isolated area with 3+ AZs. Choose based on compliance, latency, price, and service availability. |
| **Availability Zone** | One or more data centers with independent power/network. Use multi-AZ for high availability. |
| **Edge Location** | CloudFront/Route 53 PoP for low-latency content delivery. 450+ worldwide. |
| **Local Zone** | Region extension to a specific city for single-digit ms latency. |
| **Wavelength Zone** | Embedded in 5G networks for ultra-low latency mobile/IoT apps. |
| **Outposts** | AWS infrastructure in your own data center for hybrid cloud. |

---

## Quick Revision Questions

1. **What is the minimum number of Availability Zones in an AWS Region, and why?**
   > Minimum 3 AZs. This ensures redundancy — if one AZ fails, workloads can failover to another while the third handles capacity.

2. **Name four factors to consider when selecting an AWS Region.**
   > Compliance, latency, service availability, and pricing.

3. **What is the difference between an Edge Location and a Local Zone?**
   > Edge Locations run CloudFront/Route53 for content caching. Local Zones extend full compute/storage services to a metro city for ultra-low latency workloads.

4. **How are Availability Zones within a region connected?**
   > Through high-bandwidth, low-latency private fiber links.

5. **What is AWS Outposts and when would you use it?**
   > Outposts is fully managed AWS hardware deployed in your data center. Used for data residency requirements or when workloads need consistent low-latency access to on-prem systems.

6. **Can data automatically move between AWS Regions?**
   > No. Data stays in a region unless you explicitly copy or replicate it to another region (e.g., S3 cross-region replication).

---

[Next Chapter: 1.2 Regions and Availability Zones →](02-regions-and-availability-zones.md)

[← Back to README](../README.md)
