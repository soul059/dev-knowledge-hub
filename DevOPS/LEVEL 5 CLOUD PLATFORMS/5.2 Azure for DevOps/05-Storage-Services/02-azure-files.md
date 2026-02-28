# Chapter 2: Azure Files

## Overview
**Azure Files** provides fully managed file shares in the cloud accessible via **SMB** (Server Message Block) and **NFS** (Network File System) protocols. It replaces or supplements on-premises file servers with cloud-hosted shares.

---

## 2.1 Azure Files Architecture

```
┌──────────── AZURE FILES ARCHITECTURE ──────────┐
│                                                  │
│   ┌──────────────────────────────────────┐       │
│   │  Storage Account                     │       │
│   │  mydevopsstorage.file.core.windows.net│      │
│   │                                      │       │
│   │  ┌────────────────────────────┐      │       │
│   │  │  File Share: "configs"     │      │       │
│   │  │  Quota: 100 GiB            │      │       │
│   │  │                            │      │       │
│   │  │  /app-settings/            │      │       │
│   │  │    ├── dev.json            │      │       │
│   │  │    ├── staging.json        │      │       │
│   │  │    └── prod.json           │      │       │
│   │  │  /certificates/            │      │       │
│   │  │    └── ssl-cert.pfx        │      │       │
│   │  └────────────────────────────┘      │       │
│   └──────────────────────────────────────┘       │
│                                                  │
│   Access Methods:                                │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│   │ Windows  │  │  Linux   │  │  macOS   │     │
│   │ SMB 3.x  │  │ SMB/NFS  │  │  SMB 3.x │     │
│   └──────────┘  └──────────┘  └──────────┘     │
│         │              │             │           │
│         └──────────────┼─────────────┘           │
│                        ▼                         │
│        Mount as network drive / filesystem       │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 2.2 Azure Files vs Blob Storage

| Feature | Azure Files | Blob Storage |
|---------|------------|--------------|
| **Protocol** | SMB / NFS | REST API |
| **Access** | Mounted as drive | API / SDK / CLI |
| **Structure** | Directories and files | Flat (container/blob) |
| **Use Case** | Shared config, lift-and-shift | Unstructured data, media |
| **POSIX permissions** | ✅ (NFS) | ❌ |
| **Concurrent access** | Multiple VMs simultaneously | Via API |

---

## 2.3 Tiers

| Tier | Description | Use Case |
|------|-------------|----------|
| **Premium** | SSD-backed, low latency | Databases, high I/O workloads |
| **Transaction Optimized** | HDD, optimized for transactions | File shares with heavy read/write |
| **Hot** | HDD, optimized for general purpose | Team shares, web content |
| **Cool** | HDD, lower storage cost | Archival, infrequent access |

---

## 2.4 Creating and Mounting Azure File Shares

```bash
# Create file share
az storage share-rm create \
  --resource-group myRG \
  --storage-account mydevopsstorage \
  --name configs \
  --quota 100

# Upload file to share
az storage file upload \
  --account-name mydevopsstorage \
  --share-name configs \
  --source ./dev.json \
  --path app-settings/dev.json

# Create directory
az storage directory create \
  --account-name mydevopsstorage \
  --share-name configs \
  --name app-settings

# List files
az storage file list \
  --account-name mydevopsstorage \
  --share-name configs \
  --output table
```

### Mount on Windows

```powershell
# Mount Azure Files on Windows
$connectTestResult = Test-NetConnection -ComputerName mydevopsstorage.file.core.windows.net -Port 445
if ($connectTestResult.TcpTestSucceeded) {
    net use Z: \\mydevopsstorage.file.core.windows.net\configs /u:AZURE\mydevopsstorage <StorageAccountKey>
}
```

### Mount on Linux

```bash
# Install CIFS utils
sudo apt-get install cifs-utils

# Create mount point
sudo mkdir -p /mnt/configs

# Mount (using storage account key)
sudo mount -t cifs //mydevopsstorage.file.core.windows.net/configs /mnt/configs \
  -o vers=3.0,username=mydevopsstorage,password=<StorageAccountKey>,dir_mode=0777,file_mode=0777
```

---

## 2.5 Azure File Sync

```
┌──────────── AZURE FILE SYNC ───────────────────┐
│                                                  │
│   Synchronize on-premises file server with       │
│   Azure Files for hybrid access                  │
│                                                  │
│   ┌──────────────┐        ┌──────────────┐      │
│   │ On-Premises  │        │  Azure Files │      │
│   │ File Server  │◄══════►│  (Cloud)     │      │
│   │              │  Sync  │              │      │
│   │ ┌──────────┐ │        │ ┌──────────┐ │      │
│   │ │ D:\Share │ │        │ │ configs  │ │      │
│   │ └──────────┘ │        │ └──────────┘ │      │
│   └──────────────┘        └──────────────┘      │
│                                                  │
│   Features:                                      │
│   • Multi-site sync (hub-and-spoke)              │
│   • Cloud tiering (infrequent files in cloud)    │
│   • Fast disaster recovery                       │
│   • Direct cloud access via SMB/REST             │
│                                                  │
│   Components:                                    │
│   • Storage Sync Service (Azure resource)        │
│   • Sync Group (defines sync topology)           │
│   • Cloud Endpoint (Azure file share)            │
│   • Server Endpoint (on-prem folder path)        │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 2.6 Real-World DevOps Scenarios

| Scenario | Implementation |
|----------|---------------|
| **Shared Configuration** | Mount Azure Files to multiple VMs for shared config files |
| **Container Storage** | Mount file shares into ACI or AKS pods |
| **CI/CD Shared Storage** | Pipeline agents share build artifacts via mounted share |
| **Legacy App Migration** | Replace on-prem file server with Azure Files (SMB) |

```yaml
# AKS Pod mounting Azure Files
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: azure-files
      mountPath: /config
  volumes:
  - name: azure-files
    azureFile:
      secretName: azure-storage-secret
      shareName: configs
      readOnly: false
```

---

## 2.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot mount (port 445 blocked) | ISP/firewall blocking SMB | Use VPN, ExpressRoute, or Azure File Sync |
| Permission denied | Wrong storage key or RBAC | Verify credentials; use `Storage File Data SMB Share Contributor` role |
| Slow performance | Wrong tier or throttling | Use Premium tier for high I/O; check IOPS limits |
| File sync conflicts | Same file modified in two locations | Check sync status; conflicts create copies |
| NFS mount fails | Wrong storage account type | NFS requires Premium FileStorage account kind |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Protocols** | SMB 3.x and NFS |
| **Tiers** | Premium, Transaction Optimized, Hot, Cool |
| **Mount** | Network drive on Windows/Linux/macOS |
| **File Sync** | Hybrid sync between on-prem and cloud |
| **Cloud Tiering** | Keep hot files local, cool files in cloud |
| **Port** | Requires port 445 (SMB) — often blocked by ISPs |

---

## Quick Revision Questions

1. **What protocols does Azure Files support?**
   > SMB (Server Message Block) 3.x and NFS (Network File System).

2. **How is Azure Files different from Blob Storage?**
   > Azure Files is mounted as a network drive with directory structure; Blob Storage is accessed via REST API with flat namespace.

3. **What is Azure File Sync?**
   > A service that synchronizes on-premises file servers with Azure Files, providing cloud tiering and multi-site sync.

4. **What port does SMB use?**
   > Port 445. Many ISPs block this port, so VPN or ExpressRoute may be needed for non-Azure connections.

5. **What is cloud tiering?**
   > A File Sync feature that keeps frequently accessed files on-premises and tiers infrequently used files to Azure Files.

---

[⬅ Previous: Blob Storage](01-blob-storage.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Managed Disks ➡](03-managed-disks.md)
