# Chapter 5: Storage Tiers and Access Tiers

## Overview
Azure Storage **access tiers** optimize costs by categorizing data based on access frequency. Data that is accessed rarely can be stored at a much lower cost per GB, while frequently accessed data is optimized for fast retrieval.

---

## 5.1 Access Tiers Overview

```
┌────────── STORAGE ACCESS TIERS ──────────────────┐
│                                                    │
│   ┌────────────────────────────────────────────┐   │
│   │           COST vs ACCESS PATTERN           │   │
│   │                                            │   │
│   │  Storage Cost ◄──────────────► Access Cost │   │
│   │                                            │   │
│   │  HOT     ████░░░░░░  Lowest access cost    │   │
│   │          Highest storage cost              │   │
│   │                                            │   │
│   │  COOL    ███████░░░  Medium                │   │
│   │          30-day minimum                    │   │
│   │                                            │   │
│   │  COLD    ████████░░  Lower storage cost    │   │
│   │          90-day minimum                    │   │
│   │                                            │   │
│   │  ARCHIVE ██████████  Lowest storage cost   │   │
│   │          Highest access cost               │   │
│   │          180-day minimum                   │   │
│   │          Hours to rehydrate                │   │
│   │                                            │   │
│   └────────────────────────────────────────────┘   │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 5.2 Tier Comparison

| Feature | Hot | Cool | Cold | Archive |
|---------|-----|------|------|---------|
| **Storage cost** | Highest | Lower | Lower still | Lowest |
| **Access cost** | Lowest | Higher | Higher | Highest |
| **Min retention** | None | 30 days | 90 days | 180 days |
| **Availability** | 99.9% | 99% | 99% | Offline |
| **Availability (RA-GRS)** | 99.99% | 99.9% | 99.9% | Offline |
| **Latency** | Milliseconds | Milliseconds | Milliseconds | Hours |
| **Set at** | Account or blob | Account or blob | Blob only | Blob only |

> ⚠️ **Early deletion fees** apply if data is deleted or moved before the minimum retention period.

---

## 5.3 Setting Access Tiers

```bash
# Set default tier at account level (Hot or Cool only)
az storage account create \
  --name mydevopsstorage \
  --resource-group myRG \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Cool

# Change account-level default tier
az storage account update \
  --name mydevopsstorage \
  --resource-group myRG \
  --access-tier Hot

# Set tier on individual blob
az storage blob set-tier \
  --account-name mydevopsstorage \
  --container-name logs \
  --name app-2023-01-15.log \
  --tier Cool

# Move blob to Archive
az storage blob set-tier \
  --account-name mydevopsstorage \
  --container-name backups \
  --name old-backup.bak \
  --tier Archive

# Rehydrate from Archive (takes hours)
az storage blob set-tier \
  --account-name mydevopsstorage \
  --container-name backups \
  --name old-backup.bak \
  --tier Hot \
  --rehydrate-priority Standard    # Standard (up to 15h) or High (under 1h)
```

---

## 5.4 Archive Tier Details

```
┌────── ARCHIVE REHYDRATION ──────────────────┐
│                                              │
│  Archive Tier:                               │
│  • Blob is OFFLINE — cannot be read directly │
│  • Must REHYDRATE to Hot or Cool first       │
│  • Metadata remains accessible               │
│                                              │
│  Rehydration Options:                        │
│  ┌──────────────────────────────────┐        │
│  │ Standard Priority               │        │
│  │ • Up to 15 hours                │        │
│  │ • Lower cost                    │        │
│  └──────────────────────────────────┘        │
│  ┌──────────────────────────────────┐        │
│  │ High Priority                   │        │
│  │ • Under 1 hour (for <10 GB)     │        │
│  │ • Higher cost                   │        │
│  └──────────────────────────────────┘        │
│                                              │
│  Alternative: Copy to a new blob in          │
│  Hot/Cool tier (keeps original archived)     │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 5.5 Lifecycle Management Policies

Automate tier transitions based on data age:

```json
{
  "rules": [
    {
      "name": "ageBasedTiering",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/", "telemetry/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToCold": {
              "daysAfterModificationGreaterThan": 90
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 180
            },
            "delete": {
              "daysAfterModificationGreaterThan": 730
            }
          },
          "snapshot": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          }
        }
      }
    }
  ]
}
```

```
┌────── LIFECYCLE TIMELINE EXAMPLE ──────────┐
│                                             │
│   Day 0         Day 30       Day 90         │
│   ─────────────────────────────────────►    │
│   │             │            │              │
│   ▼             ▼            ▼              │
│   HOT ────────► COOL ──────► COLD           │
│                                             │
│   Day 180       Day 730                     │
│   ──────────────────────►                   │
│   │             │                           │
│   ▼             ▼                           │
│   ARCHIVE ────► DELETE                      │
│                                             │
└─────────────────────────────────────────────┘
```

```bash
# Apply lifecycle management policy
az storage account management-policy create \
  --account-name mydevopsstorage \
  --resource-group myRG \
  --policy @lifecycle-policy.json

# View current policy
az storage account management-policy show \
  --account-name mydevopsstorage \
  --resource-group myRG
```

---

## 5.6 Cost Optimization Strategy

| Data Type | Recommended Tier | Reason |
|-----------|-----------------|--------|
| Active application data | Hot | Frequent access, low latency needed |
| Monthly reports | Cool | Accessed occasionally |
| Quarterly audit logs | Cold | Rarely accessed |
| Compliance archives (7yr) | Archive | Legal retention, rarely read |
| CI/CD build artifacts (recent) | Hot | Active development |
| CI/CD build artifacts (old) | Cool → Archive | Infrequent reference |

---

## 5.7 Real-World DevOps Scenario

```
┌────── LOG MANAGEMENT PIPELINE ──────────────┐
│                                              │
│  1. App writes logs → Hot tier blob          │
│  2. Lifecycle policy:                        │
│     • 7 days  → stays Hot (active debugging) │
│     • 30 days → Cool (occasional review)     │
│     • 90 days → Cold (rare access)           │
│     • 365 days → Archive (compliance)        │
│     • 730 days → Delete                      │
│                                              │
│  Cost savings: ~80% compared to keeping      │
│  everything in Hot tier                      │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 5.8 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot read Archive blob | Blob is offline | Rehydrate to Hot or Cool first |
| Early deletion fee charged | Moved/deleted before min retention | Wait for minimum retention period |
| Lifecycle policy not running | Policy takes up to 24h to execute | Wait; check policy is enabled |
| Wrong tier set | Default account tier applies | Set tier explicitly on the blob |
| High storage costs | All data in Hot tier | Apply lifecycle policies for tiering |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Hot** | Frequent access, highest storage cost, lowest access cost |
| **Cool** | Infrequent (30-day min), lower storage, higher access cost |
| **Cold** | Rare access (90-day min), even lower storage cost |
| **Archive** | Offline (180-day min), cheapest storage, hours to rehydrate |
| **Lifecycle Policies** | Automate tiering and deletion based on data age |
| **Early Deletion** | Fees if data removed before minimum retention period |

---

## Quick Revision Questions

1. **What are the four blob access tiers?**
   > Hot, Cool, Cold, and Archive — each optimized for different access frequencies.

2. **Can you read a blob in the Archive tier directly?**
   > No. Archive blobs are offline and must be rehydrated to Hot or Cool tier first, which takes hours.

3. **What is the minimum retention period for Cool tier?**
   > 30 days. Early delete/move incurs charges.

4. **What are lifecycle management policies?**
   > Automated rules that transition blobs between tiers or delete them based on age (days since last modification).

5. **How do you save costs on log storage?**
   > Use lifecycle policies: keep recent logs Hot, move older logs to Cool/Cold, archive for compliance, delete after retention period.

---

[⬅ Previous: Storage Accounts](04-storage-accounts.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Unit 6 - Azure SQL Database ➡](../06-Database-Services/01-azure-sql-database.md)
