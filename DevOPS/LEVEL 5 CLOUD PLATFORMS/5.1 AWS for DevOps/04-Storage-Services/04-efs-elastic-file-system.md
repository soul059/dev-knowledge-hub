# Chapter 4.4: EFS — Elastic File System

## Overview

Amazon EFS is a **managed NFS file system** that can be mounted by multiple EC2 instances simultaneously across AZs. Unlike EBS (one instance, one AZ), EFS provides **shared, scalable file storage** — essential for multi-instance application architectures.

---

## EFS vs EBS vs S3

```
┌─────────────────── Storage Comparison ──────────────────────┐
│                                                              │
│  S3 (Object)          EBS (Block)          EFS (File)       │
│  ┌──────────┐        ┌──────────┐        ┌──────────┐      │
│  │ Bucket   │        │ Volume   │        │ File     │      │
│  │ Objects  │        │ Blocks   │        │ System   │      │
│  │   🌐     │        │  📦 → 1  │        │  📂 → N  │      │
│  └──────────┘        └──────────┘        └──────────┘      │
│                                                              │
│  • Regional          • AZ-specific       • Regional (multi- │
│  • Unlimited         • Fixed size          AZ)              │
│  • HTTP API          • Attach to 1 EC2   • Mount on N EC2s │
│  • Cheapest $/GB     • Lowest latency    • Shared access   │
│  • Not mountable     • Boot volume OK    • POSIX filesystem │
│  • Web, backups      • Databases         • Content mgmt,   │
│                      • Boot volumes        CMS, shared home │
└──────────────────────────────────────────────────────────────┘
```

---

## EFS Architecture

```
┌──────────────────── VPC ────────────────────────────────────┐
│                                                              │
│  ┌─── AZ-1a ──────────────────────────────────────────────┐ │
│  │                                                         │ │
│  │  ┌── EC2-A ──┐     ┌── Mount Target ──┐               │ │
│  │  │ App Server│◄───►│ eni-xxx          │◄──┐            │ │
│  │  │ /mnt/efs  │ NFS │ 10.0.1.100       │   │            │ │
│  │  └───────────┘     └──────────────────┘   │            │ │
│  │                                            │            │ │
│  │  ┌── EC2-B ──┐                             │            │ │
│  │  │ App Server│◄────────────────────────────┘            │ │
│  │  │ /mnt/efs  │                                          │ │
│  │  └───────────┘                                          │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌─── AZ-1b ──────────────────────────────────────────────┐ │
│  │                                                         │ │
│  │  ┌── EC2-C ──┐     ┌── Mount Target ──┐               │ │
│  │  │ App Server│◄───►│ eni-yyy          │◄──────────┐   │ │
│  │  │ /mnt/efs  │ NFS │ 10.0.2.100       │           │   │ │
│  │  └───────────┘     └──────────────────┘           │   │ │
│  │                                                    │   │ │
│  │  ┌── ECS ────┐                                    │   │ │
│  │  │ Fargate   │◄───────────────────────────────────┘   │ │
│  │  │ /mnt/efs  │                                        │ │
│  │  └───────────┘                                        │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌──────────── EFS File System ────────────────────────────┐ │
│  │  fs-xxx                                                  │ │
│  │  Size: Auto-scales (petabytes)                          │ │
│  │  Performance: General Purpose / Max I/O                 │ │
│  │  Throughput: Bursting / Provisioned / Elastic           │ │
│  │  Mount Targets: One per AZ                              │ │
│  └──────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

---

## Creating and Mounting EFS

```bash
# Create EFS file system
EFS_ID=$(aws efs create-file-system \
  --performance-mode generalPurpose \
  --throughput-mode elastic \
  --encrypted \
  --tags Key=Name,Value=app-shared-fs \
  --query 'FileSystemId' --output text)

# Create mount targets (one per AZ)
aws efs create-mount-target \
  --file-system-id $EFS_ID \
  --subnet-id subnet-1a \
  --security-groups sg-efs

aws efs create-mount-target \
  --file-system-id $EFS_ID \
  --subnet-id subnet-1b \
  --security-groups sg-efs

# On EC2 instance — install EFS mount helper and mount:
# sudo yum install -y amazon-efs-utils
# sudo mkdir /mnt/efs
# sudo mount -t efs -o tls fs-xxx:/ /mnt/efs

# Or add to /etc/fstab for persistent mount:
# fs-xxx:/ /mnt/efs efs _netdev,tls 0 0
```

### EFS Security Group

```bash
# Create SG for EFS mount targets
aws ec2 create-security-group \
  --group-name efs-sg \
  --description "Allow NFS from app instances" \
  --vpc-id vpc-xxx

# Allow NFS (port 2049) from app security group
aws ec2 authorize-security-group-ingress \
  --group-id sg-efs \
  --protocol tcp \
  --port 2049 \
  --source-group sg-app
```

---

## EFS Performance Modes

| Mode | Max IOPS | Latency | Use Case |
|------|---------|---------|----------|
| **General Purpose** | ~35,000 | Lowest | Web serving, CMS, home dirs |
| **Max I/O** | 500,000+ | Higher | Big data, media processing |

## EFS Throughput Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| **Bursting** | Scales with storage size; burst credits | Spiky workloads |
| **Provisioned** | Fixed throughput regardless of size | Consistent high throughput |
| **Elastic** | Auto-scales throughput up/down | Unpredictable workloads (recommended) |

---

## EFS Storage Classes & Lifecycle

```
┌──── EFS Lifecycle Management ──────────────────────────┐
│                                                         │
│  Standard Storage                                       │
│  ┌───────────────────────┐                              │
│  │ Frequently accessed   │                              │
│  │ files                 │                              │
│  └──────────┬────────────┘                              │
│             │ Not accessed for 30 days                  │
│             ▼                                           │
│  Infrequent Access (IA)                                │
│  ┌───────────────────────┐                              │
│  │ ~92% cheaper storage  │                              │
│  │ Per-access charge     │                              │
│  └──────────┬────────────┘                              │
│             │ Accessed again → moves back to Standard   │
│             ▼                                           │
│  Archive (newest tier)                                  │
│  ┌───────────────────────┐                              │
│  │ ~95% cheaper storage  │                              │
│  │ Rarely accessed       │                              │
│  └───────────────────────┘                              │
│                                                         │
│  One Zone variants: Same tiers but single AZ (cheaper) │
└─────────────────────────────────────────────────────────┘
```

```bash
# Enable lifecycle management
aws efs put-lifecycle-configuration \
  --file-system-id fs-xxx \
  --lifecycle-policies \
    TransitionToIA=AFTER_30_DAYS \
    TransitionToArchive=AFTER_90_DAYS \
    TransitionToPrimaryStorageClass=AFTER_1_ACCESS
```

---

## EFS Access Points

```
┌──── EFS Access Points ─────────────────────────────────┐
│                                                         │
│  EFS File System (fs-xxx)                              │
│  │                                                     │
│  ├── Access Point: /app1  (uid: 1001, gid: 1001)     │
│  │   Used by: ECS Service A                            │
│  │   Permissions: 755                                  │
│  │                                                     │
│  ├── Access Point: /app2  (uid: 1002, gid: 1002)     │
│  │   Used by: ECS Service B                            │
│  │   Permissions: 755                                  │
│  │                                                     │
│  └── Access Point: /shared (uid: 1000, gid: 1000)    │
│      Used by: Lambda Function                          │
│      Permissions: 755                                  │
│                                                         │
│  Benefits:                                              │
│  • Enforce user/group per application                  │
│  • Root directory isolation per app                     │
│  • Simplify IAM-based access control                   │
└─────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Mount timeout | SG not allowing port 2049 | Add NFS (2049) rule from app SG to EFS SG |
| Slow performance | Using bursting mode with small FS | Switch to Elastic throughput mode |
| "Too many files" | General Purpose IOPS limit | Switch to Max I/O mode |
| High EFS costs | All files in Standard storage | Enable lifecycle to IA (30 days) |
| Lambda can't mount | No mount target in Lambda's subnet | Create mount targets in Lambda's VPC subnets |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **EFS** | Managed NFS, multi-AZ, multi-instance shared storage |
| **Mount Target** | One per AZ, ENI with IP, needs SG allowing port 2049 |
| **Performance** | General Purpose (low latency) or Max I/O (high throughput) |
| **Throughput** | Elastic (recommended), Bursting, or Provisioned |
| **Lifecycle** | Standard → IA (30 days) → Archive (90 days) for cost savings |
| **Access Points** | Per-app root directory + user/group enforcement |
| **vs EBS** | Multi-instance, auto-scaling size, higher latency, higher cost/GB |

---

## Quick Revision Questions

1. **What is the main advantage of EFS over EBS?**
   > EFS can be mounted by multiple instances across multiple AZs simultaneously, while EBS is limited to one instance in one AZ.

2. **What port does EFS use?**
   > NFS port 2049. Security Groups must allow this.

3. **How does EFS billing work?**
   > Pay per GB stored (no pre-provisioning), with lower rates for IA and Archive tiers.

4. **What is an EFS Access Point?**
   > An application-specific entry point that enforces a root directory, user ID, and group ID, providing isolation between applications sharing the same file system.

5. **Which throughput mode is recommended for most workloads?**
   > Elastic throughput mode — it automatically scales up and down based on demand.

---

[← Previous: 4.3 EBS Volumes](03-ebs-elastic-block-store.md) | [Next: 4.5 Storage Gateway →](05-storage-gateway.md)

[← Back to README](../README.md)
