# Chapter 7: ExpressRoute and VPN

## Overview
**Azure VPN Gateway** and **Azure ExpressRoute** provide hybrid connectivity between on-premises networks and Azure. VPN uses encrypted tunnels over the internet, while ExpressRoute uses a dedicated private connection via a connectivity provider.

---

## 7.1 Connectivity Options Comparison

```
┌────────── HYBRID CONNECTIVITY OPTIONS ──────────┐
│                                                   │
│  OPTION 1: SITE-TO-SITE VPN                       │
│  ┌────────────┐   Encrypted    ┌──────────────┐  │
│  │ On-Premises │───IPsec/IKE──►│  Azure VNet   │  │
│  │ VPN Device  │   (Internet)  │  VPN Gateway  │  │
│  └────────────┘                └──────────────┘  │
│  • Up to 10 Gbps (VpnGw5)                        │
│  • Over public internet (encrypted)               │
│  • Quick to set up                                 │
│                                                   │
│  OPTION 2: POINT-TO-SITE VPN                       │
│  ┌────────────┐   Encrypted    ┌──────────────┐  │
│  │ Laptop /   │───VPN Client──►│  Azure VNet   │  │
│  │ Workstation│   (Internet)   │  VPN Gateway  │  │
│  └────────────┘                └──────────────┘  │
│  • Individual device to Azure                      │
│  • Remote workers / developers                     │
│                                                   │
│  OPTION 3: EXPRESSROUTE                            │
│  ┌────────────┐   Private      ┌──────────────┐  │
│  │ On-Premises │───Dedicated───►│  Azure        │  │
│  │ Datacenter  │  Connection   │  (MSEE)       │  │
│  └────────────┘                └──────────────┘  │
│  • Up to 100 Gbps                                  │
│  • NOT over the internet                           │
│  • Higher reliability & lower latency              │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 7.2 VPN Gateway SKUs

| SKU | S2S Tunnels | P2S Connections | Throughput |
|-----|-------------|-----------------|------------|
| **VpnGw1** | 30 | 250 | 650 Mbps |
| **VpnGw2** | 30 | 500 | 1 Gbps |
| **VpnGw3** | 30 | 1,000 | 1.25 Gbps |
| **VpnGw4** | 100 | 5,000 | 5 Gbps |
| **VpnGw5** | 100 | 10,000 | 10 Gbps |
| **VpnGw1AZ** | 30 | 250 | 650 Mbps (Zone-redundant) |

> 💡 Use AZ SKUs (VpnGw1AZ, etc.) for zone-redundant deployments.

---

## 7.3 VPN Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Route-based** | Uses IP routing tables / traffic selectors | Most scenarios, P2S, multi-site, VNet-to-VNet |
| **Policy-based** | Uses IPsec policies with address prefixes | Legacy devices, single S2S tunnel |

> ⚠️ **Route-based** is recommended for almost all scenarios.

---

## 7.4 Site-to-Site VPN Architecture

```
┌────── SITE-TO-SITE VPN ARCHITECTURE ────────┐
│                                              │
│   ON-PREMISES                  AZURE         │
│   ┌──────────┐                ┌─────────┐   │
│   │Corporate │                │ VNet     │   │
│   │Network   │                │10.0.0.0 │   │
│   │192.168.  │    IPsec       │ /16     │   │
│   │1.0/24    │    Tunnel      │         │   │
│   │          │ ◄════════════► │         │   │
│   │ ┌──────┐ │                │┌──────┐ │   │
│   │ │ VPN  │ │                ││ VPN  │ │   │
│   │ │Device│ │                ││ GW   │ │   │
│   │ │      │ │                ││      │ │   │
│   │ └──────┘ │                │└──────┘ │   │
│   └──────────┘                └─────────┘   │
│                                              │
│   Required Components:                       │
│   • GatewaySubnet in Azure VNet              │
│   • VPN Gateway (Azure side)                 │
│   • Local Network Gateway (defines on-prem)  │
│   • Connection object (links them)           │
│   • On-prem VPN device (customer side)       │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 7.5 Creating Site-to-Site VPN

```bash
# Step 1: Create GatewaySubnet (MUST be named exactly "GatewaySubnet")
az network vnet subnet create \
  --resource-group myRG \
  --vnet-name myVNet \
  --name GatewaySubnet \
  --address-prefix 10.0.255.0/27

# Step 2: Create public IP for VPN Gateway
az network public-ip create \
  --resource-group myRG \
  --name vpnGatewayIP \
  --sku Standard \
  --allocation-method Static

# Step 3: Create VPN Gateway (takes 30-45 minutes!)
az network vnet-gateway create \
  --resource-group myRG \
  --name myVpnGateway \
  --vnet myVNet \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw2 \
  --public-ip-address vpnGatewayIP \
  --no-wait

# Step 4: Create Local Network Gateway (represents on-prem)
az network local-gateway create \
  --resource-group myRG \
  --name onPremGateway \
  --gateway-ip-address 203.0.113.1 \
  --local-address-prefixes 192.168.1.0/24

# Step 5: Create connection
az network vpn-connection create \
  --resource-group myRG \
  --name myS2Sconnection \
  --vnet-gateway1 myVpnGateway \
  --local-gateway2 onPremGateway \
  --shared-key "MyPreSharedKey123!" \
  --connection-protocol IKEv2

# Verify connection status
az network vpn-connection show \
  --resource-group myRG \
  --name myS2Sconnection \
  --query connectionStatus \
  --output tsv
# Expected: Connected
```

---

## 7.6 ExpressRoute Architecture

```
┌──────────── EXPRESSROUTE ARCHITECTURE ──────────┐
│                                                   │
│   ON-PREMISES         PROVIDER          AZURE     │
│   ┌──────────┐    ┌────────────┐    ┌─────────┐ │
│   │Corporate │    │ ExpressRoute│    │ Microsoft│ │
│   │Datacenter│◄──►│ Provider   │◄──►│ Edge     │ │
│   │          │    │ (Equinix,  │    │ (MSEE)   │ │
│   │ ┌──────┐ │    │  Megaport, │    │          │ │
│   │ │Router│ │    │  etc.)     │    │ ┌──────┐ │ │
│   │ └──────┘ │    └────────────┘    │ │ VNet │ │ │
│   └──────────┘                      │ │      │ │ │
│                                     │ └──────┘ │ │
│   Peering Types:                    └─────────┘ │
│   ┌─────────────────────────────────────────┐   │
│   │ • Azure Private Peering                 │   │
│   │   → Access VMs, databases in VNets      │   │
│   │ • Microsoft Peering                     │   │
│   │   → Access Microsoft 365, Dynamics 365  │   │
│   │   → Access Azure public services        │   │
│   └─────────────────────────────────────────┘   │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 7.7 ExpressRoute SKUs

| Feature | Local | Standard | Premium |
|---------|-------|----------|---------|
| **Region access** | Local region only | Same geopolitical region | Global |
| **VNet connections** | ∞ (local) | 10 | 100 |
| **Route prefixes** | Local routes | 4,000 (private) | 10,000 (private) |
| **Bandwidth** | 1-10 Gbps | 50 Mbps - 10 Gbps | 50 Mbps - 100 Gbps |
| **Cost** | Lowest | Medium | Highest |

---

## 7.8 ExpressRoute vs VPN Comparison

| Feature | VPN Gateway | ExpressRoute |
|---------|-------------|--------------|
| **Connection** | Over public internet | Private dedicated link |
| **Encryption** | IPsec/IKE | Optional (MACsec) |
| **Bandwidth** | Up to 10 Gbps | Up to 100 Gbps |
| **Latency** | Variable | Low, predictable |
| **Setup time** | Minutes | Weeks (provider provisioning) |
| **SLA** | 99.95% (active-active) | 99.95% (standard) |
| **Cost** | Lower | Higher |
| **Use case** | Dev/test, small workloads | Production, large data, compliance |

---

## 7.9 Active-Active VPN Gateway

```
┌────── ACTIVE-ACTIVE VPN ──────────────────┐
│                                            │
│   On-Premises                   Azure      │
│   ┌──────────┐              ┌─────────┐   │
│   │ VPN      │  Tunnel 1    │ GW      │   │
│   │ Device 1 │◄════════════►│ Inst. 1 │   │
│   │          │              │ PIP-1   │   │
│   └──────────┘              │         │   │
│   ┌──────────┐  Tunnel 2   │ GW      │   │
│   │ VPN      │◄════════════►│ Inst. 2 │   │
│   │ Device 2 │              │ PIP-2   │   │
│   │          │              │         │   │
│   └──────────┘              └─────────┘   │
│                                            │
│   ✅ Both tunnels active simultaneously    │
│   ✅ Automatic failover                    │
│   ✅ Higher throughput                     │
│                                            │
└────────────────────────────────────────────┘
```

```bash
# Create active-active VPN gateway
az network vnet-gateway create \
  --resource-group myRG \
  --name myVpnGateway \
  --vnet myVNet \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw2 \
  --public-ip-address vpnPIP1 vpnPIP2 \
  --enable-active-active
```

---

## 7.10 ExpressRoute + VPN Coexistence

```
┌────── COEXISTENCE ARCHITECTURE ────────────┐
│                                             │
│   On-Premises                               │
│       │                                     │
│       ├── ExpressRoute (primary)            │
│       │   Private, high-bandwidth           │
│       │                                     │
│       └── S2S VPN (failover)                │
│           Encrypted backup path             │
│                                             │
│   Both connect to the same Azure VNet       │
│   ExpressRoute preferred (higher metric)    │
│   VPN activates if ExpressRoute fails       │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 7.11 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| VPN not connecting | Shared key mismatch | Verify pre-shared key on both sides |
| Intermittent VPN drops | IKE/IPsec parameter mismatch | Align encryption, DH group, SA lifetime |
| GatewaySubnet missing | Forgot to create | Create subnet named exactly "GatewaySubnet" |
| Gateway creation slow | Normal behavior | VPN Gateway creation takes 30-45 minutes |
| ExpressRoute not routing | BGP misconfiguration | Verify BGP ASN and advertised routes |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **S2S VPN** | IPsec tunnel over internet, up to 10 Gbps |
| **P2S VPN** | Individual device to Azure, remote workers |
| **ExpressRoute** | Private dedicated connection, up to 100 Gbps |
| **GatewaySubnet** | Required subnet (exact name) for VPN/ER gateways |
| **Active-Active** | Two gateway instances, two tunnels, automatic failover |
| **Coexistence** | ExpressRoute + VPN together for redundancy |
| **Route-based** | Recommended VPN type for most scenarios |

---

## Quick Revision Questions

1. **What is the main difference between VPN Gateway and ExpressRoute?**
   > VPN uses encrypted tunnels over the public internet; ExpressRoute uses a private dedicated connection through a provider.

2. **What is the GatewaySubnet?**
   > A special subnet (must be named "GatewaySubnet") reserved for VPN and ExpressRoute gateway resources.

3. **What are the two ExpressRoute peering types?**
   > Azure Private Peering (access VNet resources) and Microsoft Peering (access Microsoft 365 and Azure public services).

4. **What is active-active VPN gateway?**
   > A configuration with two gateway instances and two tunnels, both active simultaneously for higher availability and throughput.

5. **When would you use VPN + ExpressRoute coexistence?**
   > When you want ExpressRoute as the primary high-performance path and S2S VPN as a backup failover connection.

6. **How long does VPN Gateway creation take?**
   > Approximately 30-45 minutes to provision.

---

[⬅ Previous: VNet Peering](06-vnet-peering.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Unit 5 - Blob Storage ➡](../05-Storage-Services/01-blob-storage.md)
