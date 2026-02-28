# Chapter 3: Network Security

## Overview
Azure provides **multiple layers of network security** to protect resources — NSGs, Azure Firewall, DDoS Protection, Private Endpoints, Service Endpoints, Web Application Firewall (WAF), and Azure Bastion. This chapter covers how these work together in a defence-in-depth strategy.

---

## 3.1 Defence-in-Depth Network Security

```
┌────────── DEFENCE IN DEPTH ──────────────────────┐
│                                                    │
│  INTERNET                                          │
│     │                                              │
│     ▼                                              │
│  ┌──────────────────────────────────────────────┐  │
│  │  Layer 1: DDoS Protection                    │  │
│  │  (Volumetric attack mitigation)              │  │
│  └──────────────────────────────────────────────┘  │
│     │                                              │
│     ▼                                              │
│  ┌──────────────────────────────────────────────┐  │
│  │  Layer 2: Azure Firewall / WAF               │  │
│  │  (Application & network filtering)           │  │
│  └──────────────────────────────────────────────┘  │
│     │                                              │
│     ▼                                              │
│  ┌──────────────────────────────────────────────┐  │
│  │  Layer 3: NSG (Network Security Groups)      │  │
│  │  (Subnet & NIC-level ACLs)                   │  │
│  └──────────────────────────────────────────────┘  │
│     │                                              │
│     ▼                                              │
│  ┌──────────────────────────────────────────────┐  │
│  │  Layer 4: Private Endpoints                  │  │
│  │  (PaaS services on private network)          │  │
│  └──────────────────────────────────────────────┘  │
│     │                                              │
│     ▼                                              │
│  ┌──────────────────────────────────────────────┐  │
│  │  Layer 5: Application-level security         │  │
│  │  (TLS, authentication, authorization)        │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 3.2 Azure Firewall

```
┌────── AZURE FIREWALL ─────────────────────┐
│                                            │
│  Managed, cloud-native firewall            │
│  (Layer 3-7 filtering)                     │
│                                            │
│  ┌──────────────────────────────────────┐  │
│  │  Rule Types:                        │  │
│  │                                     │  │
│  │  NAT Rules (DNAT)                   │  │
│  │  ├── Inbound traffic translation    │  │
│  │  └── Public IP → private VM         │  │
│  │                                     │  │
│  │  Network Rules (Layer 4)            │  │
│  │  ├── IP, port, protocol filtering   │  │
│  │  └── Allow 10.0.0.0/24 → SQL:1433  │  │
│  │                                     │  │
│  │  Application Rules (Layer 7)        │  │
│  │  ├── FQDN-based filtering           │  │
│  │  └── Allow *.microsoft.com:443      │  │
│  │                                     │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Tiers:                                    │
│  ├── Standard: Network + App rules         │
│  └── Premium: + TLS inspection, IDPS, URL  │
│       filtering, Web categories            │
│                                            │
└────────────────────────────────────────────┘
```

```bash
# Create Azure Firewall
az network firewall create \
  --resource-group myRG \
  --name myFirewall \
  --location uksouth \
  --vnet-name hub-vnet

# Add application rule (allow outbound HTTPS to Microsoft)
az network firewall application-rule create \
  --resource-group myRG \
  --firewall-name myFirewall \
  --collection-name "AllowMicrosoft" \
  --priority 100 \
  --action Allow \
  --name "allow-ms" \
  --protocols Https=443 \
  --source-addresses "10.0.0.0/16" \
  --fqdn-tags "MicrosoftActiveDirectory" "AzureCloud"

# Add network rule (allow SQL traffic)
az network firewall network-rule create \
  --resource-group myRG \
  --firewall-name myFirewall \
  --collection-name "AllowSQL" \
  --priority 200 \
  --action Allow \
  --name "allow-sql" \
  --protocols TCP \
  --source-addresses "10.0.1.0/24" \
  --destination-addresses "10.0.2.0/24" \
  --destination-ports 1433
```

---

## 3.3 Private Endpoints vs Service Endpoints

```
┌────── PRIVATE ENDPOINT ───────────────────┐
│                                            │
│  VNet: 10.0.0.0/16                         │
│  ┌──────────────────────────────────────┐  │
│  │  Subnet: app-subnet                 │  │
│  │  ┌─────┐     ┌─────────────────┐    │  │
│  │  │ VM  │────►│ Private Endpoint │   │  │
│  │  │     │     │ 10.0.1.5        │    │  │
│  │  └─────┘     └────────┬────────┘    │  │
│  │                       │ Private Link │  │
│  └───────────────────────┼─────────────┘  │
│                          ▼                 │
│  ┌──────────────────────────────────────┐  │
│  │  Azure SQL Database                 │  │
│  │  (No public endpoint needed)        │  │
│  │  Traffic stays on Microsoft network │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘

┌────── SERVICE ENDPOINT ───────────────────┐
│                                            │
│  VNet: 10.0.0.0/16                         │
│  ┌──────────────────────────────────────┐  │
│  │  Subnet: app-subnet                 │  │
│  │  Service Endpoint: Microsoft.Sql    │  │
│  │  ┌─────┐                            │  │
│  │  │ VM  │──── Optimised route ───►   │  │
│  │  └─────┘    (Azure backbone)        │  │
│  └──────────────────────────────────────┘  │
│                          ▼                 │
│  ┌──────────────────────────────────────┐  │
│  │  Azure SQL Database                 │  │
│  │  (Public endpoint still exists)     │  │
│  │  Firewall allows VNet traffic       │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

| Feature | Private Endpoint | Service Endpoint |
|---------|-----------------|------------------|
| **Traffic path** | Private IP in your VNet | Optimised route via Azure backbone |
| **Public endpoint** | Can be disabled | Still exists (firewall restricts) |
| **On-premises access** | Yes (via VPN/ExpressRoute) | No (VNet only) |
| **Cost** | Per-hour + data processing | Free |
| **DNS** | Private DNS zone required | No DNS changes |
| **Cross-region** | Yes | Same region only |

```bash
# Create Private Endpoint for SQL
az network private-endpoint create \
  --resource-group myRG \
  --name sql-private-endpoint \
  --vnet-name myVNet \
  --subnet app-subnet \
  --private-connection-resource-id "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Sql/servers/myserver" \
  --group-id sqlServer \
  --connection-name sql-connection

# Create Service Endpoint
az network vnet subnet update \
  --resource-group myRG \
  --vnet-name myVNet \
  --name app-subnet \
  --service-endpoints Microsoft.Sql Microsoft.Storage
```

---

## 3.4 Azure Bastion

```
┌────── AZURE BASTION ──────────────────────┐
│                                            │
│  Problem: Exposing RDP/SSH ports to        │
│  internet is a security risk               │
│                                            │
│  Solution: Bastion provides secure         │
│  RDP/SSH via the Azure Portal (HTTPS)      │
│                                            │
│  ┌──────────────────────────────────────┐  │
│  │  User → Azure Portal (HTTPS 443)    │  │
│  │       → Azure Bastion               │  │
│  │       → VM (RDP 3389 / SSH 22)      │  │
│  │                                     │  │
│  │  VM needs NO public IP              │  │
│  │  NO NSG rule for 3389/22 needed     │  │
│  │  Session in browser                 │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Requires: AzureBastionSubnet (/26 min)    │
│                                            │
└────────────────────────────────────────────┘
```

```bash
# Create Bastion
az network bastion create \
  --resource-group myRG \
  --name myBastion \
  --vnet-name myVNet \
  --public-ip-address bastion-pip \
  --location uksouth
```

---

## 3.5 DDoS Protection

| Plan | Features | Cost |
|------|----------|------|
| **Basic** | Automatic, free, always-on for all Azure resources | Free |
| **Standard** | Adaptive tuning, metrics, logging, cost protection, rapid response | ~$2,944/month |

```bash
# Create DDoS Protection Plan
az network ddos-protection create \
  --resource-group myRG \
  --name myDDoSPlan \
  --location uksouth

# Associate with VNet
az network vnet update \
  --resource-group myRG \
  --name myVNet \
  --ddos-protection-plan myDDoSPlan \
  --ddos-protection true
```

---

## 3.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Can't reach PaaS service | Private Endpoint DNS not resolving | Configure Private DNS zone and VNet link |
| Firewall blocking legitimate traffic | Rule priority misconfigured | Check rule order (lower number = higher priority) |
| Bastion connection fails | AzureBastionSubnet not /26 or larger | Resize subnet to at least /26 |
| Service Endpoint not working | Not enabled on both subnet and service | Enable on subnet AND add VNet rule on service |
| DDoS false positives | Legitimate traffic spike | Tune DDoS policy or contact support |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Defence-in-Depth** | DDoS → Firewall → NSG → Private Endpoint → App Security |
| **Azure Firewall** | Managed L3-L7 firewall with NAT, Network, App rules |
| **Private Endpoints** | Private IP for PaaS services, no public exposure |
| **Service Endpoints** | Optimised routing to PaaS (free, same-region) |
| **Azure Bastion** | Secure RDP/SSH via browser, no public IP on VMs |
| **DDoS Protection** | Basic (free) or Standard (adaptive, metrics, cost protection) |

---

## Quick Revision Questions

1. **What is the difference between Private Endpoints and Service Endpoints?**
   > Private Endpoints assign a private IP in your VNet (can disable public endpoint). Service Endpoints optimise routing to PaaS but the public endpoint remains.

2. **What is Azure Bastion?**
   > A managed service providing secure RDP/SSH access to VMs through the Azure Portal over HTTPS, eliminating the need for public IPs on VMs.

3. **What are the three Azure Firewall rule types?**
   > NAT rules (inbound DNAT), Network rules (L4 IP/port filtering), and Application rules (L7 FQDN-based filtering).

4. **What is defence-in-depth?**
   > A layered security strategy where multiple controls (DDoS, firewall, NSG, encryption, authentication) protect resources at different levels.

5. **When would you use DDoS Protection Standard over Basic?**
   > When you need adaptive tuning, real-time metrics/alerts, cost protection (credits during attacks), and rapid response support.

---

[⬅ Previous: Microsoft Defender for Cloud](02-security-center.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Encryption and Certificates ➡](04-encryption-and-certificates.md)
