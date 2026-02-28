# Chapter 4.3: EBS — Elastic Block Store

## Overview

Amazon EBS provides **persistent block storage volumes** for EC2 instances. Think of EBS as a virtual hard drive that persists independently of the instance lifecycle — critical for databases, applications, and boot volumes.

---

## EBS Architecture

```
┌──────────── Availability Zone: us-east-1a ──────────────────┐
│                                                              │
│  ┌──── EC2 Instance ──────┐     ┌──── EBS Volume ────────┐ │
│  │  i-xxx                  │     │  vol-aaa (gp3)         │ │
│  │                         │◄───►│  Root: /dev/xvda       │ │
│  │  OS + App               │     │  100 GB, 3000 IOPS     │ │
│  │                         │     └────────────────────────┘ │
│  │                         │                                 │
│  │                         │     ┌──── EBS Volume ────────┐ │
│  │                         │◄───►│  vol-bbb (io2)         │ │
│  │  Database data          │     │  Data: /dev/xvdf       │ │
│  │                         │     │  500 GB, 64000 IOPS    │ │
│  └─────────────────────────┘     └────────────────────────┘ │
│                                                              │
│  ┌──── Unattached Volume ────────────────────────────────┐  │
│  │  vol-ccc (gp3)   500 GB   Available   (snapshot for DR)│  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  Key Rules:                                                  │
│  • EBS volumes are AZ-specific (can't attach cross-AZ)     │
│  • One volume → one instance (except io1/io2 Multi-Attach) │
│  • One instance → multiple volumes                          │
│  • Persists after instance termination (if configured)      │
└──────────────────────────────────────────────────────────────┘
```

---

## EBS Volume Types

```
┌─────────────── EBS Volume Types ──────────────────────────────┐
│                                                                │
│  SSD-backed (random I/O):                                     │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  gp3 (General Purpose)                                   │ │
│  │  • 3,000 IOPS baseline (free), up to 16,000             │ │
│  │  • 125 MiB/s baseline, up to 1,000 MiB/s               │ │
│  │  • IOPS and throughput independently configurable        │ │
│  │  • Best for: Boot volumes, dev/test, small DBs           │ │
│  │  • ★ DEFAULT CHOICE for most workloads                   │ │
│  └──────────────────────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  gp2 (Previous Generation)                                │ │
│  │  • 3 IOPS/GB (burst up to 3,000 for < 1 TB)             │ │
│  │  • Max 16,000 IOPS at 5,334 GB                          │ │
│  │  • IOPS tied to volume size (can't configure separately) │ │
│  └──────────────────────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  io2 Block Express / io1 (Provisioned IOPS)              │ │
│  │  • Up to 256,000 IOPS (io2 BE) / 64,000 IOPS (io1)     │ │
│  │  • Sub-millisecond latency                               │ │
│  │  • Multi-Attach supported                                │ │
│  │  • Best for: Mission-critical DBs, high-perf apps       │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                │
│  HDD-backed (sequential I/O):                                 │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  st1 (Throughput Optimized)                               │ │
│  │  • Max 500 IOPS, 500 MiB/s throughput                   │ │
│  │  • Best for: Big data, data warehouses, log processing   │ │
│  │  • Cannot be boot volume                                 │ │
│  └──────────────────────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  sc1 (Cold HDD)                                          │ │
│  │  • Max 250 IOPS, 250 MiB/s throughput                   │ │
│  │  • Cheapest EBS option                                   │ │
│  │  • Best for: Infrequent access, archival                 │ │
│  │  • Cannot be boot volume                                 │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### Quick Selection Guide

| Workload | Volume Type | Why |
|----------|-------------|-----|
| Boot volume | gp3 | Cost-effective, good baseline IOPS |
| General app | gp3 | Configurable IOPS/throughput |
| Production database | io2 | High IOPS, low latency, Multi-Attach |
| Data warehouse (sequential) | st1 | High throughput, low cost |
| Archive / cold storage | sc1 | Cheapest, infrequent access |

---

## EBS Snapshots

```
┌──────────── EBS Snapshot Workflow ──────────────────────────┐
│                                                              │
│  EBS Volume (AZ: us-east-1a)                                │
│  ┌────────────────────┐                                     │
│  │  vol-xxx  100 GB   │──── Snapshot ────►  ┌────────────┐ │
│  │  Data blocks:      │   (incremental)     │ snap-aaa   │ │
│  │  [A][B][C][D]      │                     │ Stored in  │ │
│  └────────────────────┘                     │ S3 (region)│ │
│                                              └─────┬──────┘ │
│  After changes: [A][B'][C][D'][E]                  │        │
│                                                     │        │
│  Second snapshot (only changed blocks):            │        │
│  ┌────────────┐                                    │        │
│  │ snap-bbb   │  Contains: [B'][D'][E] only       │        │
│  │ Incremental│  References snap-aaa for [A][C]    │        │
│  └────────────┘                                    │        │
│                                                     ▼        │
│  Restore to:    ┌──── New volume ────────────┐             │
│  • Same AZ      │  vol-yyy (any AZ in region) │             │
│  • Different AZ  │  or different region        │             │
│  • Different     │  or different volume type   │             │
│    region (copy) └─────────────────────────────┘             │
└──────────────────────────────────────────────────────────────┘
```

### Snapshot CLI

```bash
# Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-xxx \
  --description "DB backup 2024-01-15" \
  --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=db-backup-20240115}]'

# Create volume from snapshot (in different AZ)
aws ec2 create-volume \
  --snapshot-id snap-xxx \
  --availability-zone us-east-1b \
  --volume-type gp3 \
  --iops 3000 \
  --throughput 125

# Copy snapshot to another region (DR)
aws ec2 copy-snapshot \
  --source-region us-east-1 \
  --source-snapshot-id snap-xxx \
  --destination-region eu-west-1 \
  --description "DR copy"

# Create snapshot lifecycle policy (automated backups)
aws dlm create-lifecycle-policy \
  --description "Daily EBS snapshots" \
  --state ENABLED \
  --execution-role-arn arn:aws:iam::role/AWSDataLifecycleManagerRole \
  --policy-details '{
    "PolicyType": "EBS_SNAPSHOT_MANAGEMENT",
    "ResourceTypes": ["VOLUME"],
    "TargetTags": [{"Key": "Backup", "Value": "daily"}],
    "Schedules": [{
      "Name": "DailySnapshots",
      "CreateRule": {"Interval": 24, "IntervalUnit": "HOURS", "Times": ["03:00"]},
      "RetainRule": {"Count": 7}
    }]
  }'

# List snapshots
aws ec2 describe-snapshots \
  --owner-ids self \
  --query 'Snapshots[*].[SnapshotId,VolumeId,StartTime,State]' \
  --output table
```

---

## EBS Volume Operations

```bash
# Create a gp3 volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --volume-type gp3 \
  --size 100 \
  --iops 3000 \
  --throughput 125 \
  --encrypted \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=app-data}]'

# Attach volume to instance
aws ec2 attach-volume \
  --volume-id vol-xxx \
  --instance-id i-xxx \
  --device /dev/xvdf

# Modify volume (resize or change type — no downtime!)
aws ec2 modify-volume \
  --volume-id vol-xxx \
  --volume-type io2 \
  --size 200 \
  --iops 10000

# Detach volume
aws ec2 detach-volume --volume-id vol-xxx

# After attaching, format and mount (on EC2 instance):
# sudo mkfs -t xfs /dev/xvdf
# sudo mkdir /data
# sudo mount /dev/xvdf /data
# echo '/dev/xvdf /data xfs defaults,nofail 0 2' | sudo tee -a /etc/fstab
```

---

## EBS Encryption

```
┌────────── EBS Encryption ──────────────────────────────────┐
│                                                             │
│  ┌─ Encrypted Volume ──────────────────────────────────┐   │
│  │  • Data at rest encrypted (AES-256)                  │   │
│  │  • Data in transit (instance ↔ volume) encrypted     │   │
│  │  • Snapshots automatically encrypted                 │   │
│  │  • Volumes from encrypted snapshots = encrypted      │   │
│  │  • Uses AWS KMS (CMK or aws/ebs default key)        │   │
│  │  • Minimal performance impact                        │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  To encrypt an existing unencrypted volume:                 │
│  1. Create snapshot of unencrypted volume                   │
│  2. Copy snapshot with encryption enabled                   │
│  3. Create new encrypted volume from encrypted snapshot     │
│  4. Swap volumes on instance                               │
│                                                             │
│  Enable default encryption for all new volumes:             │
│  aws ec2 enable-ebs-encryption-by-default                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Can't attach volume | Volume and instance in different AZs | Create volume in same AZ or restore from snapshot |
| Volume stuck "attaching" | Device name conflict | Use different device name (e.g., /dev/xvdg) |
| gp2 performance drop | Burst credits exhausted | Switch to gp3 (baseline 3000 IOPS regardless of size) |
| Snapshot taking too long | Large volume, first snapshot | First snap copies all blocks; subsequent are incremental |
| "Volume in use" on delete | Still attached to instance | Detach first, then delete |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **gp3** | Default SSD. 3000 IOPS baseline. IOPS/throughput configurable independently. |
| **io2** | High-perf SSD. Up to 256K IOPS. Sub-ms latency. Multi-Attach. |
| **st1** | Throughput HDD. 500 MiB/s. Big data, sequential workloads. |
| **sc1** | Cold HDD. Cheapest. Infrequent access. |
| **Snapshot** | Incremental backup to S3. Cross-AZ/region restore. |
| **Encryption** | AES-256 via KMS. Enable by default for compliance. |
| **AZ-bound** | Volumes can only attach to instances in the same AZ. |

---

## Quick Revision Questions

1. **What is the default IOPS for a gp3 volume?**
   > 3,000 IOPS baseline (included free), configurable up to 16,000.

2. **Can you attach an EBS volume to an instance in a different AZ?**
   > No. EBS volumes are AZ-specific. Use snapshots to move data across AZs.

3. **How are EBS snapshots stored?**
   > Incrementally in Amazon S3 (managed by AWS; you can't see them in your S3 console).

4. **How do you encrypt an existing unencrypted volume?**
   > Snapshot → copy snapshot with encryption → create volume from encrypted snapshot → swap.

5. **Which volume types can be used as boot volumes?**
   > Only SSD types: gp2, gp3, io1, io2. HDD types (st1, sc1) cannot be boot volumes.

6. **What is EBS Multi-Attach?**
   > io1/io2 volumes can be attached to multiple instances in the same AZ simultaneously, useful for clustered applications.

---

[← Previous: 4.2 S3 Storage Classes](02-s3-storage-classes-lifecycle.md) | [Next: 4.4 EFS →](04-efs-elastic-file-system.md)

[← Back to README](../README.md)
