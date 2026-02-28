# Chapter 3: Managed Disks

## Overview
**Azure Managed Disks** are block-level storage volumes managed by Azure for use with Azure VMs. Azure handles the storage account management, so you only need to specify the disk type and size.

---

## 3.1 Managed vs Unmanaged Disks

```
┌────── MANAGED vs UNMANAGED DISKS ──────────┐
│                                             │
│  UNMANAGED (Legacy):                        │
│  ┌──────────┐                               │
│  │ VM       │                               │
│  └────┬─────┘                               │
│       │                                     │
│  ┌────▼──────────────────────┐              │
│  │ Storage Account (yours)   │              │
│  │ YOU manage:               │              │
│  │ • Storage account limits  │              │
│  │ • VHD file placement      │              │
│  │ • IOPS calculations       │              │
│  └───────────────────────────┘              │
│                                             │
│  MANAGED (Recommended):                     │
│  ┌──────────┐                               │
│  │ VM       │                               │
│  └────┬─────┘                               │
│       │                                     │
│  ┌────▼──────────────────────┐              │
│  │ Managed Disk              │              │
│  │ AZURE manages:            │              │
│  │ • Storage account (hidden)│              │
│  │ • Replication              │              │
│  │ • Scaling                  │              │
│  │ • Encryption               │              │
│  └───────────────────────────┘              │
│                                             │
│  ✅ Always use Managed Disks               │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 3.2 Disk Types

| Disk Type | IOPS (max) | Throughput | Latency | Use Case |
|-----------|------------|------------|---------|----------|
| **Ultra Disk** | 160,000 | 4,000 MB/s | Sub-ms | SAP HANA, top-tier DBs |
| **Premium SSD v2** | 80,000 | 1,200 MB/s | Sub-ms | Production DBs, high perf |
| **Premium SSD** | 20,000 | 900 MB/s | Low | Production VMs, databases |
| **Standard SSD** | 6,000 | 750 MB/s | Medium | Web servers, dev/test |
| **Standard HDD** | 2,000 | 500 MB/s | Higher | Backups, infrequent access |

---

## 3.3 Disk Roles

```
┌────── VM DISK ROLES ──────────────────────┐
│                                            │
│   ┌──────────────────────────────────┐     │
│   │  Azure VM                        │     │
│   │                                  │     │
│   │  ┌────────────────────┐          │     │
│   │  │ OS Disk (C: / /)   │ Required │     │
│   │  │ Contains OS         │          │     │
│   │  │ Max: 4 TiB          │          │     │
│   │  └────────────────────┘          │     │
│   │                                  │     │
│   │  ┌────────────────────┐          │     │
│   │  │ Temp Disk (D: /tmp)│ Ephemeral│     │
│   │  │ Non-persistent!     │          │     │
│   │  │ Lost on reboot/move │          │     │
│   │  └────────────────────┘          │     │
│   │                                  │     │
│   │  ┌────────────────────┐          │     │
│   │  │ Data Disk (E:,F:)  │ Optional │     │
│   │  │ App data, databases │          │     │
│   │  │ Max: 32 TiB         │          │     │
│   │  │ Multiple per VM     │          │     │
│   │  └────────────────────┘          │     │
│   │                                  │     │
│   └──────────────────────────────────┘     │
│                                            │
│   ⚠️ NEVER store important data on        │
│      the temp disk — it's ephemeral!       │
│                                            │
└────────────────────────────────────────────┘
```

---

## 3.4 Creating and Managing Disks

```bash
# Create a managed disk
az disk create \
  --resource-group myRG \
  --name myDataDisk \
  --size-gb 128 \
  --sku Premium_LRS \
  --location eastus

# Attach disk to VM
az vm disk attach \
  --resource-group myRG \
  --vm-name myVM \
  --name myDataDisk

# Detach disk from VM
az vm disk detach \
  --resource-group myRG \
  --vm-name myVM \
  --name myDataDisk

# Resize a disk (VM must be deallocated)
az disk update \
  --resource-group myRG \
  --name myDataDisk \
  --size-gb 256

# Create snapshot
az snapshot create \
  --resource-group myRG \
  --name myDiskSnapshot \
  --source myDataDisk

# Create disk from snapshot (for recovery)
az disk create \
  --resource-group myRG \
  --name restoredDisk \
  --source myDiskSnapshot \
  --sku Premium_LRS

# List disks
az disk list --resource-group myRG --output table
```

---

## 3.5 Disk Encryption

| Encryption Type | Description |
|-----------------|-------------|
| **SSE (Server-Side Encryption)** | Default — all managed disks encrypted at rest with platform-managed keys |
| **SSE with CMK** | Customer-managed keys via Azure Key Vault |
| **Azure Disk Encryption (ADE)** | OS-level encryption using BitLocker (Windows) or DM-Crypt (Linux) |
| **Encryption at host** | Data encrypted before reaching storage service |

```bash
# Enable Azure Disk Encryption on a VM
az vm encryption enable \
  --resource-group myRG \
  --name myVM \
  --disk-encryption-keyvault myKeyVault

# Check encryption status
az vm encryption show \
  --resource-group myRG \
  --name myVM
```

---

## 3.6 Shared Disks

```
┌────── SHARED DISKS ──────────────────────┐
│                                           │
│   ┌──────┐    ┌──────┐                   │
│   │ VM-1 │    │ VM-2 │                   │
│   └───┬──┘    └──┬───┘                   │
│       │          │                        │
│       └────┬─────┘                        │
│            │                              │
│   ┌────────▼─────────┐                   │
│   │  Shared Managed  │                   │
│   │  Disk            │                   │
│   │  (Premium/Ultra) │                   │
│   └──────────────────┘                   │
│                                           │
│   Use case: Clustered applications        │
│   (Windows Failover Cluster, SQL FCI)     │
│   Requires cluster-aware filesystem       │
│                                           │
└───────────────────────────────────────────┘
```

---

## 3.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot attach disk | VM and disk in different regions | Ensure same region |
| Disk not showing in OS | Disk not initialized/formatted | Initialize and format in OS |
| Cannot resize disk | VM is running | Deallocate VM first, then resize |
| Cannot delete disk | Disk attached to VM | Detach disk first, then delete |
| High latency | Using Standard HDD for production | Upgrade to Premium SSD |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Managed Disks** | Azure-managed block storage — always preferred |
| **Disk Types** | Ultra, Premium SSD v2, Premium SSD, Standard SSD, Standard HDD |
| **Disk Roles** | OS Disk, Temp Disk (ephemeral!), Data Disk |
| **Snapshots** | Point-in-time copy for backup/recovery |
| **Encryption** | SSE (default), CMK, ADE (BitLocker/DM-Crypt) |
| **Shared Disks** | Multi-VM attach for clustered applications |

---

## Quick Revision Questions

1. **What is the difference between managed and unmanaged disks?**
   > Managed disks are handled by Azure (storage, replication, encryption). Unmanaged disks require you to manage your own storage accounts.

2. **Why should you not store data on the temp disk?**
   > The temp disk is ephemeral — data is lost when the VM is deallocated, resized, or experiences a host failure.

3. **What disk type provides the highest IOPS?**
   > Ultra Disk — up to 160,000 IOPS.

4. **What is Azure Disk Encryption?**
   > OS-level encryption using BitLocker (Windows) or DM-Crypt (Linux) with keys stored in Azure Key Vault.

5. **How do you create a backup of a managed disk?**
   > Create a snapshot using `az snapshot create`, then create a new disk from the snapshot if recovery is needed.

---

[⬅ Previous: Azure Files](02-azure-files.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Storage Accounts ➡](04-storage-accounts.md)
