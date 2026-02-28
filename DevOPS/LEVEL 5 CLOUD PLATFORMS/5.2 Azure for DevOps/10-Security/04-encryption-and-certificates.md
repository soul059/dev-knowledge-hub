# Chapter 4: Encryption and Certificates

## Overview
Azure provides **encryption at rest** and **in transit** across all services. This chapter covers encryption mechanisms (SSE, ADE, TDE, TLS), customer-managed keys (CMK), certificate management via Key Vault and App Service, and best practices for managing cryptographic assets in a DevOps workflow.

---

## 4.1 Encryption Overview

```
┌────────── ENCRYPTION IN AZURE ───────────────────┐
│                                                    │
│  AT REST (stored data):                            │
│  ┌──────────────────────────────────────────────┐  │
│  │  Storage Service Encryption (SSE)            │  │
│  │  ├── All storage accounts (auto, AES-256)    │  │
│  │  ├── Microsoft-managed keys (default)        │  │
│  │  └── Customer-managed keys (CMK in Key Vault)│  │
│  │                                              │  │
│  │  Azure Disk Encryption (ADE)                 │  │
│  │  ├── OS + Data disks on VMs                  │  │
│  │  ├── BitLocker (Windows) / DM-Crypt (Linux)  │  │
│  │  └── Keys stored in Key Vault                │  │
│  │                                              │  │
│  │  Transparent Data Encryption (TDE)           │  │
│  │  ├── SQL Database / SQL Managed Instance     │  │
│  │  ├── Encrypts database files at rest         │  │
│  │  └── Enabled by default (service-managed)    │  │
│  │                                              │  │
│  │  Cosmos DB Encryption                        │  │
│  │  └── Automatic, always on, AES-256           │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  IN TRANSIT (network data):                        │
│  ┌──────────────────────────────────────────────┐  │
│  │  TLS 1.2+ (HTTPS, AMQPS, SMTPS)             │  │
│  │  ├── Enforced on all Azure services          │  │
│  │  ├── App Service: HTTPS Only setting         │  │
│  │  ├── Storage: Secure transfer required       │  │
│  │  └── SQL: Encrypted connections by default   │  │
│  │                                              │  │
│  │  VPN / ExpressRoute Encryption               │  │
│  │  ├── IPsec for Site-to-Site VPN              │  │
│  │  └── MACsec for ExpressRoute Direct          │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 4.2 Key Management Options

```
┌────── KEY MANAGEMENT ─────────────────────┐
│                                            │
│  Microsoft-Managed Keys (MMK):             │
│  ┌──────────────────────────────────────┐  │
│  │  • Default for all services         │  │
│  │  • Microsoft creates, rotates, stores│  │
│  │  • No customer action required      │  │
│  │  • Sufficient for most workloads    │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Customer-Managed Keys (CMK):              │
│  ┌──────────────────────────────────────┐  │
│  │  • Keys stored in YOUR Key Vault    │  │
│  │  • You control rotation schedule    │  │
│  │  • Required for compliance (FIPS)   │  │
│  │  • Supports auto-rotation           │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Customer-Provided Keys (CPK – Blob only): │
│  ┌──────────────────────────────────────┐  │
│  │  • Key sent with each API request   │  │
│  │  • Never stored in Azure            │  │
│  │  • Maximum control                  │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

---

## 4.3 Configuring Encryption

```bash
# ── STORAGE ENCRYPTION (CMK) ──

# Create encryption key in Key Vault
az keyvault key create \
  --vault-name myapp-kv \
  --name storage-encryption-key \
  --kty RSA \
  --size 2048

# Enable CMK on storage account
az storage account update \
  --resource-group myRG \
  --name mystorageaccount \
  --encryption-key-vault "https://myapp-kv.vault.azure.net" \
  --encryption-key-name "storage-encryption-key" \
  --encryption-key-source Microsoft.Keyvault

# ── AZURE DISK ENCRYPTION ──

# Enable ADE on a Linux VM
az vm encryption enable \
  --resource-group myRG \
  --name myLinuxVM \
  --disk-encryption-keyvault myapp-kv \
  --volume-type All

# Check encryption status
az vm encryption show \
  --resource-group myRG \
  --name myLinuxVM

# ── ENFORCE HTTPS ──

# App Service: HTTPS Only
az webapp update \
  --resource-group myRG \
  --name myapp \
  --https-only true

# Storage: Require secure transfer
az storage account update \
  --resource-group myRG \
  --name mystorageaccount \
  --https-only true

# ── TDE (enabled by default on Azure SQL) ──
az sql db tde set \
  --resource-group myRG \
  --server myserver \
  --database mydb \
  --status Enabled
```

---

## 4.4 Certificate Management

```
┌────── CERTIFICATE LIFECYCLE ──────────────┐
│                                            │
│  1. Obtain Certificate                     │
│     ├── Key Vault self-signed (dev/test)   │
│     ├── Key Vault + CA integration         │
│     │   (DigiCert, GlobalSign)             │
│     ├── App Service Managed Certificate    │
│     │   (free, auto-renewed for custom     │
│     │    domains)                           │
│     └── External CA (import PFX)           │
│                                            │
│  2. Store Certificate                      │
│     └── Azure Key Vault                    │
│                                            │
│  3. Bind Certificate                       │
│     ├── App Service: SSL/TLS binding       │
│     ├── Application Gateway: HTTPS listener│
│     ├── Azure Front Door: custom domain    │
│     └── VM: install from Key Vault         │
│                                            │
│  4. Renew Certificate                      │
│     ├── Auto-renew (Key Vault + CA)        │
│     ├── App Service Managed (auto)         │
│     └── Manual re-import                   │
│                                            │
└────────────────────────────────────────────┘
```

```bash
# Create App Service Managed Certificate (free)
az webapp config ssl create \
  --resource-group myRG \
  --name myapp \
  --hostname "www.myapp.com"

# Bind certificate to App Service
az webapp config ssl bind \
  --resource-group myRG \
  --name myapp \
  --certificate-thumbprint "<thumbprint>" \
  --ssl-type SNI

# Import PFX certificate to Key Vault
az keyvault certificate import \
  --vault-name myapp-kv \
  --name myapp-cert \
  --file ./certificate.pfx \
  --password "pfxPassword"

# Use Key Vault certificate in App Service
az webapp config ssl import \
  --resource-group myRG \
  --name myapp \
  --key-vault myapp-kv \
  --key-vault-certificate-name myapp-cert
```

---

## 4.5 Encryption Comparison Table

| Service | At Rest | In Transit | Key Options |
|---------|---------|------------|-------------|
| **Storage** | SSE (AES-256) | TLS 1.2 (HTTPS) | MMK, CMK, CPK |
| **VM Disks** | SSE + optional ADE | N/A (disk-level) | MMK, CMK, ADE (KV) |
| **SQL Database** | TDE (AES-256) | TLS 1.2 | Service-managed, CMK |
| **Cosmos DB** | AES-256 (always on) | TLS 1.2 | Service-managed, CMK |
| **Key Vault** | HSM-protected | TLS 1.2 | HSM (Premium), Software |
| **App Service** | Platform encryption | TLS 1.2 (configurable) | Managed cert, KV cert |

---

## 4.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| CMK encryption fails | Key Vault access denied | Grant `wrap/unwrap` key permissions to service |
| ADE fails on VM | Key Vault not in same region | Use Key Vault in same region as VM |
| Certificate binding fails | Cert doesn't match custom domain | Ensure CN/SAN matches the hostname |
| TLS connection error | Client using TLS 1.0/1.1 | Enforce minimum TLS 1.2 on the service |
| Cert expired | Auto-renewal not configured | Enable auto-renewal in Key Vault policy |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Encryption at Rest** | SSE (storage), ADE (VM disks), TDE (SQL), Cosmos (auto) |
| **Encryption in Transit** | TLS 1.2 enforced across all Azure services |
| **Microsoft-Managed Keys** | Default, no action needed |
| **Customer-Managed Keys** | Your keys in Key Vault, you control rotation |
| **Certificates** | Key Vault, App Service Managed, or external CA import |
| **ADE** | BitLocker/DM-Crypt for VM OS and data disks |

---

## Quick Revision Questions

1. **What is the difference between SSE and ADE?**
   > SSE (Storage Service Encryption) encrypts storage data at the service level automatically. ADE (Azure Disk Encryption) uses BitLocker/DM-Crypt to encrypt VM disks with keys in Key Vault.

2. **What is TDE?**
   > Transparent Data Encryption encrypts SQL Database files at rest using AES-256, enabled by default with service-managed keys.

3. **What are customer-managed keys (CMK)?**
   > Encryption keys stored in your Key Vault that you control, including rotation schedule, instead of relying on Microsoft-managed keys.

4. **How do you get a free TLS certificate for App Service?**
   > Use App Service Managed Certificate — a free, auto-renewed certificate for custom domains bound to your web app.

5. **What TLS version should be enforced?**
   > TLS 1.2 minimum. TLS 1.0 and 1.1 are deprecated and should be disabled on all Azure services.

---

[⬅ Previous: Network Security](03-network-security.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Compliance and Governance ➡](05-compliance-and-governance.md)
