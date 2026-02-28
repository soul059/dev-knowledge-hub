# Chapter 4: Storage Accounts

## Overview
A **Storage Account** is the top-level Azure resource that provides a unique namespace for all Azure Storage data — blobs, files, queues, tables, and disks. All storage settings (replication, performance, access tier) are configured at this level.

---

## 4.1 Storage Account Architecture

```
┌──────────── STORAGE ACCOUNT ───────────────────┐
│                                                  │
│   Storage Account: mydevopsstorage               │
│   Namespace: mydevopsstorage.<service>.core...   │
│                                                  │
│   ┌──────────────────────────────────────────┐   │
│   │  SERVICES:                               │   │
│   │                                          │   │
│   │  ┌──────────┐  ┌──────────┐              │   │
│   │  │  Blob    │  │  Files   │              │   │
│   │  │  Storage │  │  (SMB)   │              │   │
│   │  └──────────┘  └──────────┘              │   │
│   │                                          │   │
│   │  ┌──────────┐  ┌──────────┐              │   │
│   │  │  Queue   │  │  Table   │              │   │
│   │  │  Storage │  │  Storage │              │   │
│   │  └──────────┘  └──────────┘              │   │
│   │                                          │   │
│   └──────────────────────────────────────────┘   │
│                                                  │
│   Endpoints:                                      │
│   • Blob:  mydevopsstorage.blob.core.windows.net │
│   • Files: mydevopsstorage.file.core.windows.net │
│   • Queue: mydevopsstorage.queue.core.windows.net│
│   • Table: mydevopsstorage.table.core.windows.net│
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 4.2 Storage Account Types

| Account Type | Supported Services | Performance | Use Case |
|-------------|-------------------|-------------|----------|
| **StorageV2** (general-purpose v2) | Blob, File, Queue, Table | Standard / Premium | **Recommended for most scenarios** |
| **BlobStorage** | Blob only | Standard | Legacy, use StorageV2 instead |
| **BlockBlobStorage** | Block & Append Blobs | Premium | High transaction workloads |
| **FileStorage** | Files only | Premium | High-performance file shares (NFS) |

> ✅ **Always use StorageV2** unless you specifically need Premium Block Blobs or Premium Files.

---

## 4.3 Replication Options

```
┌────── REPLICATION OPTIONS ──────────────────────┐
│                                                   │
│  LRS (Locally Redundant Storage)                   │
│  ┌───────────────────────┐                        │
│  │  Datacenter            │                        │
│  │  Copy 1  Copy 2  Copy 3│  3 copies same DC     │
│  └───────────────────────┘  99.999999999% (11 9s) │
│                                                   │
│  ZRS (Zone-Redundant Storage)                      │
│  ┌────────┐ ┌────────┐ ┌────────┐                 │
│  │ Zone 1 │ │ Zone 2 │ │ Zone 3 │                 │
│  │ Copy 1 │ │ Copy 2 │ │ Copy 3 │  3 zones       │
│  └────────┘ └────────┘ └────────┘  12 9s          │
│                                                   │
│  GRS (Geo-Redundant Storage)                       │
│  ┌────────────┐        ┌────────────┐             │
│  │ Primary    │        │ Secondary  │             │
│  │ Region     │───────►│ Region     │             │
│  │ 3 copies   │ Async  │ 3 copies   │ 6 copies   │
│  └────────────┘        └────────────┘  16 9s      │
│                                                   │
│  RA-GRS (Read-Access Geo-Redundant)                │
│  Same as GRS but secondary is READABLE             │
│  Endpoint: myaccount-secondary.blob.core...        │
│                                                   │
│  GZRS (Geo-Zone-Redundant Storage)                 │
│  ┌───────────────┐     ┌────────────┐             │
│  │ Primary Region│     │ Secondary  │             │
│  │ ZRS (3 zones) │────►│ LRS        │             │
│  └───────────────┘     └────────────┘  16 9s      │
│                                                   │
│  RA-GZRS = GZRS + readable secondary              │
│                                                   │
└───────────────────────────────────────────────────┘
```

| Replication | Durability | Regions | Read from Secondary |
|-------------|-----------|---------|---------------------|
| **LRS** | 11 nines | 1 DC | ❌ |
| **ZRS** | 12 nines | 1 region (3 zones) | ❌ |
| **GRS** | 16 nines | 2 regions | ❌ |
| **RA-GRS** | 16 nines | 2 regions | ✅ |
| **GZRS** | 16 nines | 2 regions (primary = ZRS) | ❌ |
| **RA-GZRS** | 16 nines | 2 regions (primary = ZRS) | ✅ |

---

## 4.4 Creating a Storage Account

```bash
# Create StorageV2 with LRS
az storage account create \
  --name mydevopsstorage \
  --resource-group myRG \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot \
  --min-tls-version TLS1_2 \
  --allow-blob-public-access false

# Create with geo-redundancy
az storage account create \
  --name myprodstore \
  --resource-group myRG \
  --location eastus \
  --sku Standard_GZRS \
  --kind StorageV2

# List storage accounts
az storage account list --resource-group myRG --output table

# Get connection string
az storage account show-connection-string \
  --name mydevopsstorage \
  --resource-group myRG \
  --output tsv

# Get access keys
az storage account keys list \
  --account-name mydevopsstorage \
  --resource-group myRG \
  --output table

# Rotate access key
az storage account keys renew \
  --account-name mydevopsstorage \
  --resource-group myRG \
  --key primary
```

---

## 4.5 Network Security

```
┌────── STORAGE ACCOUNT NETWORK RULES ────────┐
│                                              │
│  Options:                                    │
│  1. Public access from all networks          │
│     (default — NOT recommended for prod)     │
│                                              │
│  2. Selected networks only                   │
│     • Specific VNet subnets                  │
│     • Specific IP ranges                     │
│     • Private endpoints                      │
│                                              │
│  3. Disabled (private endpoints only)        │
│     (Most secure)                            │
│                                              │
└──────────────────────────────────────────────┘
```

```bash
# Set default action to deny
az storage account update \
  --name mydevopsstorage \
  --resource-group myRG \
  --default-action Deny

# Allow specific VNet/subnet
az storage account network-rule add \
  --account-name mydevopsstorage \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet web-subnet

# Allow specific IP
az storage account network-rule add \
  --account-name mydevopsstorage \
  --resource-group myRG \
  --ip-address 203.0.113.5

# Create private endpoint
az network private-endpoint create \
  --resource-group myRG \
  --name myStoragePE \
  --vnet-name myVNet \
  --subnet data-subnet \
  --private-connection-resource-id $(az storage account show --name mydevopsstorage --resource-group myRG --query id -o tsv) \
  --group-id blob \
  --connection-name myBlobConnection
```

---

## 4.6 Storage Account Security Best Practices

| Practice | CLI/Setting |
|----------|-------------|
| Disable public blob access | `--allow-blob-public-access false` |
| Require TLS 1.2 | `--min-tls-version TLS1_2` |
| Disable shared key access | `--allow-shared-key-access false` |
| Use managed identity | Assign RBAC role instead of keys |
| Enable soft delete | Recover accidentally deleted blobs |
| Use private endpoints | No public internet exposure |

---

## 4.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| 403: AuthorizationFailure | Firewall blocking access | Add IP/VNet to network rules |
| Storage name unavailable | Name globally taken | Choose a different unique name (3-24 chars, lowercase/numbers) |
| Cannot change replication | Incompatible conversion | Some conversions require creating new account |
| Access key compromised | Key leaked in code | Rotate key immediately; use managed identity |
| TLS error | Old client TLS version | Ensure client supports TLS 1.2+ |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Account Types** | StorageV2 (recommended), BlockBlobStorage, FileStorage |
| **Services** | Blob, Files, Queue, Table |
| **Replication** | LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS |
| **Access Keys** | Two keys for rotation; prefer managed identity |
| **Network Rules** | Lock down with firewall, VNet rules, private endpoints |
| **Naming** | Globally unique, 3-24 chars, lowercase + numbers only |

---

## Quick Revision Questions

1. **What is the recommended storage account type?**
   > StorageV2 (general-purpose v2) — supports all storage services and features.

2. **What is the difference between GRS and RA-GRS?**
   > Both replicate to a secondary region; RA-GRS additionally allows read access from the secondary endpoint.

3. **How many copies does GZRS maintain?**
   > 6 copies — 3 across availability zones in the primary region + 3 in the secondary region (LRS).

4. **What are the storage account naming rules?**
   > 3-24 characters, lowercase letters and numbers only, globally unique.

5. **How should you secure a storage account for production?**
   > Disable public access, require TLS 1.2, use private endpoints, use managed identity instead of access keys.

---

[⬅ Previous: Managed Disks](03-managed-disks.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Storage Tiers ➡](05-storage-tiers.md)
