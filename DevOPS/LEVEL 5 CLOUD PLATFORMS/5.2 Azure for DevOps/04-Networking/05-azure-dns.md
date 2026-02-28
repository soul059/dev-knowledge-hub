# Chapter 5: Azure DNS

## Overview
**Azure DNS** hosts DNS zones on Azure infrastructure, providing name resolution using Microsoft's global network of DNS servers. It supports both **public DNS** (internet-facing) and **private DNS** (VNet-internal) zones.

---

## 5.1 DNS Architecture

```
┌──────────────── AZURE DNS ──────────────────┐
│                                              │
│   User types: app.contoso.com                │
│       │                                      │
│       ▼                                      │
│   ┌──────────────────────────┐               │
│   │  DNS Resolver            │               │
│   │  "Where is contoso.com?" │               │
│   └──────────┬───────────────┘               │
│              │                                │
│              ▼                                │
│   ┌──────────────────────────┐               │
│   │  Azure DNS Zone:         │               │
│   │  contoso.com             │               │
│   │                          │               │
│   │  Records:                │               │
│   │  ┌─────────────────────┐ │               │
│   │  │ A    app    52.1.2.3│ │               │
│   │  │ A    api    52.1.2.4│ │               │
│   │  │ CNAME www   app     │ │               │
│   │  │ MX   @     mail.co  │ │               │
│   │  │ TXT  @     "v=spf1" │ │               │
│   │  └─────────────────────┘ │               │
│   └──────────────────────────┘               │
│                                              │
│   Name Servers (auto-assigned):              │
│   ns1-01.azure-dns.com                       │
│   ns2-01.azure-dns.net                       │
│   ns3-01.azure-dns.org                       │
│   ns4-01.azure-dns.info                      │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 5.2 DNS Record Types

| Record Type | Purpose | Example |
|-------------|---------|---------|
| **A** | Maps name to IPv4 address | `app.contoso.com → 52.168.1.100` |
| **AAAA** | Maps name to IPv6 address | `app.contoso.com → 2001:db8::1` |
| **CNAME** | Alias to another DNS name | `www.contoso.com → app.contoso.com` |
| **MX** | Mail exchange server | `contoso.com → mail.contoso.com` |
| **TXT** | Text records (SPF, verification) | `contoso.com → "v=spf1 include:..."` |
| **NS** | Name server delegation | Auto-created by Azure |
| **SOA** | Start of Authority | Auto-created by Azure |
| **SRV** | Service locator | `_sip._tcp.contoso.com` |
| **CAA** | Certificate Authority Authorization | Which CAs can issue certs |
| **Alias** | Azure-specific: points to Azure resource | Points to LB, AppGW, CDN, etc. |

---

## 5.3 Alias Records (Azure-Specific)

```
┌────── ALIAS RECORDS vs CNAME ──────────────┐
│                                             │
│  CNAME Limitations:                         │
│  • Cannot be at zone apex (contoso.com)     │
│  • Extra DNS lookup required                │
│                                             │
│  Alias Record Advantages:                   │
│  ✅ Works at zone apex                      │
│  ✅ Points directly to Azure resource       │
│  ✅ Auto-updates when resource IP changes   │
│  ✅ No dangling DNS (deleted resource)      │
│                                             │
│  Supported targets:                         │
│  • Public IP address                        │
│  • Azure Traffic Manager profile            │
│  • Azure CDN endpoint                       │
│  • Another DNS record in same zone          │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 5.4 Creating a Public DNS Zone

```bash
# Create DNS zone
az network dns zone create \
  --resource-group myRG \
  --name contoso.com

# Add A record
az network dns record-set a add-record \
  --resource-group myRG \
  --zone-name contoso.com \
  --record-set-name app \
  --ipv4-address 52.168.1.100

# Add CNAME record
az network dns record-set cname set-record \
  --resource-group myRG \
  --zone-name contoso.com \
  --record-set-name www \
  --cname app.contoso.com

# Add an Alias record (zone apex pointing to public IP)
az network dns record-set a create \
  --resource-group myRG \
  --zone-name contoso.com \
  --name "@" \
  --target-resource "/subscriptions/{sub}/resourceGroups/myRG/providers/Microsoft.Network/publicIPAddresses/myPublicIP"

# List all records
az network dns record-set list \
  --resource-group myRG \
  --zone-name contoso.com \
  --output table

# Get name servers (update at your registrar)
az network dns zone show \
  --resource-group myRG \
  --name contoso.com \
  --query nameServers \
  --output tsv
```

---

## 5.5 Private DNS Zones

```
┌───────── PRIVATE DNS ZONES ─────────────────┐
│                                              │
│   For VNet-internal name resolution          │
│                                              │
│   ┌──────────────────────────────────┐       │
│   │  Private Zone: internal.contoso  │       │
│   │                                  │       │
│   │  Records:                        │       │
│   │  db.internal.contoso → 10.0.2.4  │       │
│   │  api.internal.contoso → 10.0.1.5 │       │
│   └──────────────┬───────────────────┘       │
│                  │ VNet Link                  │
│   ┌──────────────▼───────────────────┐       │
│   │  VNet: myVNet                    │       │
│   │  ┌──────┐    ┌──────┐           │       │
│   │  │ VM-1 │───▶│ VM-2 │           │       │
│   │  │      │    │ db.  │           │       │
│   │  │ resolves   │internal│          │       │
│   │  │ name  │    │.contoso│          │       │
│   │  └──────┘    └──────┘           │       │
│   └──────────────────────────────────┘       │
│                                              │
│   Features:                                  │
│   • Auto-registration of VM names            │
│   • Cross-VNet resolution via VNet links     │
│   • No internet exposure                     │
│                                              │
└──────────────────────────────────────────────┘
```

```bash
# Create private DNS zone
az network private-dns zone create \
  --resource-group myRG \
  --name internal.contoso.com

# Link private DNS zone to VNet (with auto-registration)
az network private-dns link vnet create \
  --resource-group myRG \
  --zone-name internal.contoso.com \
  --name myVNetLink \
  --virtual-network myVNet \
  --registration-enabled true

# Add a record
az network private-dns record-set a add-record \
  --resource-group myRG \
  --zone-name internal.contoso.com \
  --record-set-name db \
  --ipv4-address 10.0.2.4
```

---

## 5.6 Real-World Scenario: Custom Domain for App Service

```bash
# Step 1: Add CNAME record in Azure DNS
az network dns record-set cname set-record \
  --resource-group myRG \
  --zone-name contoso.com \
  --record-set-name www \
  --cname myapp.azurewebsites.net

# Step 2: Add TXT verification record
az network dns record-set txt add-record \
  --resource-group myRG \
  --zone-name contoso.com \
  --record-set-name asuid.www \
  --value "VERIFICATION_ID_FROM_PORTAL"

# Step 3: Add custom domain to App Service
az webapp config hostname add \
  --resource-group myRG \
  --webapp-name myapp \
  --hostname www.contoso.com
```

---

## 5.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Domain not resolving | NS records not updated at registrar | Update registrar with Azure nameservers |
| Propagation delay | DNS TTL not expired | Wait for TTL; reduce TTL before changes |
| Private DNS not working | VNet link missing | Create VNet link to private zone |
| VM names not auto-registering | Registration not enabled | Enable auto-registration on VNet link |
| Alias record not updating | Resource deleted | Check target resource still exists |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Public DNS** | Internet-facing zones hosted on Azure nameservers |
| **Private DNS** | VNet-internal resolution, no internet exposure |
| **Alias Records** | Zone-apex support, auto-updating, Azure-specific |
| **Auto-registration** | VMs automatically get DNS records in private zones |
| **Record Types** | A, AAAA, CNAME, MX, TXT, SRV, CAA, NS, SOA |
| **VNet Links** | Connect private zones to VNets for resolution |

---

## Quick Revision Questions

1. **What is the difference between public and private DNS zones?**
   > Public zones resolve names on the internet; private zones resolve names only within linked VNets.

2. **What are Alias records?**
   > Azure-specific records that point directly to Azure resources (Public IP, Traffic Manager, CDN), work at zone apex, and auto-update.

3. **What is auto-registration in private DNS?**
   > When enabled on a VNet link, VMs in that VNet automatically get A records created in the private zone.

4. **Why can't you use a CNAME at the zone apex?**
   > DNS standard prohibits CNAME at zone apex (e.g., contoso.com). Use an Alias record instead.

5. **How do you delegate a domain to Azure DNS?**
   > Create the zone in Azure DNS, then update the NS records at your domain registrar to point to Azure's nameservers.

---

[⬅ Previous: Application Gateway](04-application-gateway.md) | [⬆ Back to Table of Contents](../README.md) | [Next: VNet Peering ➡](06-vnet-peering.md)
