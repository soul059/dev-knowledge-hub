# Chapter 4.5: AWS Storage Gateway

## Overview

AWS Storage Gateway is a **hybrid cloud storage** service that gives on-premises applications access to virtually unlimited cloud storage. It bridges the gap between on-premises environments and AWS, enabling backup, archiving, disaster recovery, and cloud data processing.

---

## Storage Gateway Types

```
┌──────────── AWS Storage Gateway ─────────────────────────────┐
│                                                               │
│  On-Premises                         AWS Cloud               │
│                                                               │
│  ┌─── S3 File Gateway ────────────────────────────────────┐  │
│  │                                                         │  │
│  │  NFS/SMB                S3 API                         │  │
│  │  ┌────────┐           ┌──────────────┐                 │  │
│  │  │ Server │◄── NFS ──►│ File Gateway │──► S3 Bucket   │  │
│  │  │        │   /SMB    │ (VM/Hardware)│                 │  │
│  │  └────────┘           └──────────────┘                 │  │
│  │                                                         │  │
│  │  Files → S3 objects    Local cache for low latency     │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌─── Volume Gateway ─────────────────────────────────────┐  │
│  │                                                         │  │
│  │  iSCSI                                                 │  │
│  │  ┌────────┐           ┌──────────────┐                 │  │
│  │  │ Server │◄─ iSCSI ─►│ Vol Gateway  │──► S3 + EBS   │  │
│  │  │        │           │              │     Snapshots   │  │
│  │  └────────┘           └──────────────┘                 │  │
│  │                                                         │  │
│  │  Cached: Primary data in S3, hot data cached locally   │  │
│  │  Stored: Primary data local, async backup to S3        │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌─── Tape Gateway ──────────────────────────────────────┐  │
│  │                                                         │  │
│  │  iSCSI-VTL                                             │  │
│  │  ┌────────┐           ┌──────────────┐                 │  │
│  │  │ Backup │◄─ iSCSI ─►│ Tape Gateway │──► S3 Glacier │  │
│  │  │ SW     │   VTL     │ (Virtual     │                │  │
│  │  └────────┘           │  Tape Lib)   │                │  │
│  │                       └──────────────┘                 │  │
│  │  Virtual tapes stored in S3, archived to Glacier       │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

---

## Gateway Types Comparison

| Feature | S3 File Gateway | Volume Gateway (Cached) | Volume Gateway (Stored) | Tape Gateway |
|---------|----------------|------------------------|------------------------|--------------|
| **Protocol** | NFS / SMB | iSCSI | iSCSI | iSCSI-VTL |
| **Backed by** | S3 | S3 + EBS snapshots | S3 + EBS snapshots | S3 + Glacier |
| **Primary data** | S3 | S3 (cache local) | Local (backup to S3) | Virtual tapes |
| **Use case** | File shares, data lakes | Extend storage to cloud | Low-latency local + backup | Replace physical tapes |
| **Access from AWS** | S3 API directly | EBS snapshot restore | EBS snapshot restore | S3 Glacier retrieval |

---

## S3 File Gateway Deep Dive

```
┌───── S3 File Gateway Architecture ──────────────────────┐
│                                                          │
│  On-Premises App Server                                  │
│  ┌──────────────────┐                                    │
│  │ mount -t nfs     │                                    │
│  │ gateway:/share   │                                    │
│  │ /mnt/cloud/      │                                    │
│  └────────┬─────────┘                                    │
│           │ NFS v3/v4.1                                  │
│           ▼                                              │
│  ┌───── File Gateway VM ──────────────────┐              │
│  │  Local disk cache (SSD recommended)    │              │
│  │  ┌──────────────────────────────────┐  │              │
│  │  │ Recently accessed files cached   │  │              │
│  │  │ Reads: from cache (low latency)  │  │              │
│  │  │ Writes: to cache → async to S3   │  │              │
│  │  └──────────────────────────────────┘  │              │
│  └────────────────┬───────────────────────┘              │
│                   │ HTTPS                                │
│                   ▼                                      │
│  ┌──── S3 Bucket ──────────────────────────┐            │
│  │  /share/file.txt → s3://bucket/file.txt │            │
│  │                                          │            │
│  │  • Lifecycle rules apply                 │            │
│  │  • S3 events trigger Lambda             │            │
│  │  • Cross-region replication             │            │
│  │  • S3 analytics & inventory             │            │
│  └──────────────────────────────────────────┘            │
└──────────────────────────────────────────────────────────┘

Flow: File write → Local cache → Async upload to S3
Flow: File read → Check cache → If miss, fetch from S3
```

---

## When to Use Each Storage Service

```
┌───────────────────────────────────────────────────────────┐
│                  Storage Decision Tree                     │
│                                                           │
│  Need shared file storage for cloud instances?            │
│  └── YES → EFS (NFS) or FSx (Windows/Lustre)            │
│                                                           │
│  Need block storage for a single instance?               │
│  └── YES → EBS                                           │
│                                                           │
│  Need object storage (API access)?                       │
│  └── YES → S3                                            │
│                                                           │
│  Need hybrid (on-prem ↔ cloud)?                          │
│  └── YES → Storage Gateway                               │
│      ├── File access (NFS/SMB) → S3 File Gateway        │
│      ├── Block access (iSCSI)  → Volume Gateway         │
│      └── Tape replacement      → Tape Gateway           │
│                                                           │
│  Need high-perf computing file system?                   │
│  └── YES → FSx for Lustre                               │
│                                                           │
│  Need Windows file shares?                               │
│  └── YES → FSx for Windows File Server                  │
└───────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Slow file access | Cache miss, fetching from S3 | Increase local cache disk size; use SSD |
| Gateway offline | VM/hardware connectivity issue | Check network, ensure HTTPS outbound to AWS |
| Files not appearing in S3 | Async upload backlog | Check gateway health; verify bandwidth |
| S3 changes not visible via NFS | Gateway cache stale | Use `RefreshCache` API or set TTL |
| High data transfer costs | Large amounts moving between on-prem and S3 | Consider AWS Direct Connect for cost + speed |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Storage Gateway** | Hybrid bridge: on-premises ↔ AWS cloud storage |
| **S3 File Gateway** | NFS/SMB to S3. Local cache. Best for file shares. |
| **Volume Gateway** | iSCSI block storage. Cached or Stored mode. EBS snapshots. |
| **Tape Gateway** | Virtual tape library. Replaces physical tapes. S3 Glacier. |
| **Local Cache** | SSD recommended. Reduces latency for frequently accessed data. |
| **Key Benefit** | Seamless migration path from on-premises to cloud storage. |

---

## Quick Revision Questions

1. **What protocols does S3 File Gateway support?**
   > NFS (v3 and v4.1) and SMB (Server Message Block).

2. **What is the difference between Cached and Stored Volume Gateway?**
   > Cached keeps primary data in S3 with a local cache for hot data. Stored keeps primary data locally with async backups to S3.

3. **How does Tape Gateway integrate with existing backup software?**
   > It presents virtual tapes via iSCSI-VTL, so existing backup applications (Veeam, Veritas, etc.) work without modification.

4. **Where are Volume Gateway snapshots stored?**
   > As EBS snapshots in S3, which can be restored to EBS volumes on EC2.

5. **When would you choose Storage Gateway over direct S3 uploads?**
   > When on-premises applications need file system (NFS/SMB) or block (iSCSI) access, rather than using the S3 API directly.

---

[← Previous: 4.4 EFS](04-efs-elastic-file-system.md)

[← Back to README](../README.md)
