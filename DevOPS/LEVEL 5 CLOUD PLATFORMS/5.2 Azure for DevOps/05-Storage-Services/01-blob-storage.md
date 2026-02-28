# Chapter 1: Blob Storage

## Overview
**Azure Blob Storage** is Microsoft's object storage solution for the cloud, optimized for storing massive amounts of unstructured data — text, binary, images, videos, logs, backups, and more.

---

## 1.1 Blob Storage Architecture

```
┌──────────── BLOB STORAGE HIERARCHY ──────────────┐
│                                                    │
│   ┌────────────────────────────────────────┐       │
│   │  Storage Account                       │       │
│   │  mydevopsstorage.blob.core.windows.net │       │
│   │                                        │       │
│   │  ┌──────────────────────────────────┐  │       │
│   │  │  Container: "images"             │  │       │
│   │  │  ┌──────┐ ┌──────┐ ┌──────┐     │  │       │
│   │  │  │logo. │ │hero. │ │icon. │     │  │       │
│   │  │  │png   │ │jpg   │ │svg   │     │  │       │
│   │  │  └──────┘ └──────┘ └──────┘     │  │       │
│   │  └──────────────────────────────────┘  │       │
│   │                                        │       │
│   │  ┌──────────────────────────────────┐  │       │
│   │  │  Container: "logs"               │  │       │
│   │  │  ┌──────────┐ ┌──────────┐      │  │       │
│   │  │  │app-2024- │ │app-2024- │      │  │       │
│   │  │  │01-15.log │ │01-16.log │      │  │       │
│   │  │  └──────────┘ └──────────┘      │  │       │
│   │  └──────────────────────────────────┘  │       │
│   │                                        │       │
│   │  ┌──────────────────────────────────┐  │       │
│   │  │  Container: "backups"            │  │       │
│   │  │  ┌──────────┐                    │  │       │
│   │  │  │db-backup │                    │  │       │
│   │  │  │.bak      │                    │  │       │
│   │  │  └──────────┘                    │  │       │
│   │  └──────────────────────────────────┘  │       │
│   └────────────────────────────────────────┘       │
│                                                    │
│   URL format:                                      │
│   https://<account>.blob.core.windows.net/         │
│          <container>/<blob>                         │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 1.2 Blob Types

| Blob Type | Description | Max Size | Use Case |
|-----------|-------------|----------|----------|
| **Block Blob** | Blocks of data uploaded individually | 190.7 TiB | Files, images, videos, logs |
| **Append Blob** | Optimized for append operations | 195 GiB | Log files, audit trails |
| **Page Blob** | Random read/write access | 8 TiB | VHD disks, databases |

---

## 1.3 Container Access Levels

| Level | Description |
|-------|-------------|
| **Private** (default) | No anonymous access; requires authentication |
| **Blob** | Anonymous read access to blobs only |
| **Container** | Anonymous read access to blobs + container listing |

> ⚠️ **Security Best Practice:** Always use **Private** access. Use SAS tokens or managed identities for controlled access.

---

## 1.4 Creating and Managing Blob Storage

```bash
# Create storage account
az storage account create \
  --name mydevopsstorage \
  --resource-group myRG \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2

# Create a container
az storage container create \
  --name images \
  --account-name mydevopsstorage \
  --public-access off

# Upload a blob
az storage blob upload \
  --account-name mydevopsstorage \
  --container-name images \
  --name logo.png \
  --file ./logo.png

# Upload entire directory
az storage blob upload-batch \
  --account-name mydevopsstorage \
  --destination images \
  --source ./local-images/

# List blobs in container
az storage blob list \
  --account-name mydevopsstorage \
  --container-name images \
  --output table

# Download a blob
az storage blob download \
  --account-name mydevopsstorage \
  --container-name images \
  --name logo.png \
  --file ./downloaded-logo.png

# Delete a blob
az storage blob delete \
  --account-name mydevopsstorage \
  --container-name images \
  --name logo.png
```

---

## 1.5 Shared Access Signatures (SAS)

```
┌────── SAS TOKEN TYPES ──────────────────────┐
│                                              │
│  1. Account SAS                              │
│     → Access to multiple services/resources  │
│                                              │
│  2. Service SAS                              │
│     → Access to specific service (blob, etc.)│
│                                              │
│  3. User Delegation SAS (RECOMMENDED)        │
│     → Signed with Azure AD credentials       │
│     → Most secure, no storage key exposure   │
│                                              │
│  SAS URL example:                            │
│  https://myaccount.blob.core.windows.net/    │
│  images/logo.png?sv=2021-06-08&st=2024-...   │
│  &se=2024-...&sr=b&sp=r&sig=XXXX            │
│                                              │
│  Parameters:                                 │
│  sv = Storage version                        │
│  st = Start time                             │
│  se = Expiry time                            │
│  sr = Resource (b=blob, c=container)         │
│  sp = Permissions (r=read, w=write)          │
│  sig = Signature                             │
│                                              │
└──────────────────────────────────────────────┘
```

```bash
# Generate SAS token for a blob (read-only, 1 hour)
END_DATE=$(date -u -d "1 hour" '+%Y-%m-%dT%H:%MZ')

az storage blob generate-sas \
  --account-name mydevopsstorage \
  --container-name images \
  --name logo.png \
  --permissions r \
  --expiry $END_DATE \
  --output tsv
```

---

## 1.6 Lifecycle Management

```json
{
  "rules": [
    {
      "name": "moveToCoolthenArchive",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        }
      }
    }
  ]
}
```

```bash
# Apply lifecycle policy
az storage account management-policy create \
  --account-name mydevopsstorage \
  --resource-group myRG \
  --policy @lifecycle-policy.json
```

---

## 1.7 Real-World DevOps Scenarios

| Scenario | How Blob Storage Helps |
|----------|----------------------|
| **CI/CD Artifacts** | Store build outputs, packages, test results |
| **Terraform State** | Remote state backend for infrastructure code |
| **Log Aggregation** | Centralized log storage from multiple services |
| **Static Website Hosting** | Host SPAs directly from blob storage |
| **Backup Storage** | Database backups, VM disk snapshots |

```bash
# Enable static website hosting
az storage blob service-properties update \
  --account-name mydevopsstorage \
  --static-website \
  --index-document index.html \
  --404-document 404.html

# Upload website files
az storage blob upload-batch \
  --account-name mydevopsstorage \
  --destination '$web' \
  --source ./dist/
```

---

## 1.8 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| 403 Forbidden | Missing permissions or expired SAS | Check RBAC role or regenerate SAS token |
| Blob not found | Wrong container/blob name | Verify container and blob name (case-sensitive) |
| Upload timeout | Large file, slow connection | Use `azcopy` for large transfers |
| High storage costs | Data in wrong tier | Enable lifecycle management policies |
| Anonymous access blocked | Public access disabled (org policy) | Use SAS or managed identity instead |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Blob Types** | Block (files), Append (logs), Page (VHDs) |
| **Containers** | Logical grouping of blobs within storage account |
| **Access Levels** | Private (default), Blob, Container |
| **SAS Tokens** | Time-limited, scoped access; User Delegation SAS preferred |
| **Lifecycle** | Auto-tier or delete blobs based on age |
| **Static Website** | Host HTML/CSS/JS directly from $web container |

---

## Quick Revision Questions

1. **What are the three blob types?**
   > Block blobs (files), Append blobs (logs), and Page blobs (VHDs/disks).

2. **What is the recommended SAS type?**
   > User Delegation SAS — signed with Azure AD credentials, most secure because no storage keys are exposed.

3. **What is lifecycle management?**
   > Automated policies that move blobs between tiers (Hot → Cool → Archive) or delete them based on age.

4. **How do you host a static website in Blob Storage?**
   > Enable static website hosting and upload files to the `$web` container.

5. **What is the URL format for accessing a blob?**
   > `https://<account>.blob.core.windows.net/<container>/<blob>`

---

[⬅ Previous: ExpressRoute and VPN](../04-Networking/07-expressroute-and-vpn.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Files ➡](02-azure-files.md)
