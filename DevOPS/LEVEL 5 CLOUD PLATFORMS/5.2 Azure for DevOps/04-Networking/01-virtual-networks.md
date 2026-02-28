# Chapter 1: Virtual Networks (VNet)

## Overview
An **Azure Virtual Network (VNet)** is the fundamental building block for private networking in Azure. VNets enable Azure resources to securely communicate with each other, the internet, and on-premises networks.

---

## 1.1 VNet Architecture

```
┌──────────────────── AZURE VIRTUAL NETWORK ──────────────────────┐
│                                                                   │
│   VNet: 10.0.0.0/16                                              │
│   ┌───────────────────────────────────────────────────────┐      │
│   │                                                       │      │
│   │   Subnet: web-subnet (10.0.1.0/24)                   │      │
│   │   ┌──────┐  ┌──────┐  ┌──────┐                      │      │
│   │   │ VM-1 │  │ VM-2 │  │ VM-3 │                      │      │
│   │   │.4    │  │.5    │  │.6    │                      │      │
│   │   └──────┘  └──────┘  └──────┘                      │      │
│   │                                                       │      │
│   │   Subnet: app-subnet (10.0.2.0/24)                   │      │
│   │   ┌──────┐  ┌──────┐                                 │      │
│   │   │App   │  │App   │                                 │      │
│   │   │Svc-1 │  │Svc-2 │                                 │      │
│   │   └──────┘  └──────┘                                 │      │
│   │                                                       │      │
│   │   Subnet: db-subnet (10.0.3.0/24)                    │      │
│   │   ┌──────┐                                            │      │
│   │   │SQL DB│                                            │      │
│   │   │.4    │                                            │      │
│   │   └──────┘                                            │      │
│   │                                                       │      │
│   └───────────────────────────────────────────────────────┘      │
│                                                                   │
│   Reserved IPs per subnet:                                       │
│   .0 = Network address    .1 = Default gateway                   │
│   .2-.3 = Azure DNS       .255 = Broadcast                      │
│   (5 IPs reserved, so /24 = 251 usable addresses)               │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

---

## 1.2 Creating VNets

```bash
# Create VNet
az network vnet create \
  --resource-group myRG \
  --name myVNet \
  --address-prefixes 10.0.0.0/16 \
  --location eastus

# Add subnets
az network vnet subnet create \
  --resource-group myRG \
  --vnet-name myVNet \
  --name web-subnet \
  --address-prefixes 10.0.1.0/24

az network vnet subnet create \
  --resource-group myRG \
  --vnet-name myVNet \
  --name app-subnet \
  --address-prefixes 10.0.2.0/24

az network vnet subnet create \
  --resource-group myRG \
  --vnet-name myVNet \
  --name db-subnet \
  --address-prefixes 10.0.3.0/24

# List VNets
az network vnet list --resource-group myRG --output table

# List subnets
az network vnet subnet list --resource-group myRG --vnet-name myVNet --output table
```

---

## 1.3 IP Addressing

| Type | Description | Use Case |
|------|-------------|----------|
| **Private IP** | Internal address within VNet (10.x, 172.16.x, 192.168.x) | VM-to-VM communication |
| **Public IP (Dynamic)** | Changes on VM deallocation | Dev/test |
| **Public IP (Static)** | Persists across restarts | Production, DNS |
| **Standard Public IP** | Zone-redundant, secure by default | Production workloads |
| **Basic Public IP** | No zone redundancy, open by default | Being retired |

---

## 1.4 VNet Key Rules

- A VNet is **scoped to a single region** and a **single subscription**
- VNets in different regions connect via **VNet Peering** or **VPN Gateway**
- Subnets **cannot span** multiple VNets
- **Resources within the same VNet** can communicate by default
- **Resources in different VNets** are isolated by default
- Azure reserves **5 IPs per subnet**

---

## 1.5 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| VMs can't communicate | Different VNets without peering | Set up VNet peering |
| IP address conflict | Overlapping address spaces | Plan CIDR blocks carefully |
| Subnet full | Not enough IPs in subnet | Create a new subnet with larger prefix |
| Can't delete VNet | Resources still attached | Delete all resources in VNet first |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **VNet** | Private network in Azure; region-scoped |
| **Subnets** | Subdivisions of VNet; 5 IPs reserved by Azure |
| **Address Space** | Use private ranges (10.x, 172.16.x, 192.168.x) |
| **Communication** | Same VNet = automatic; Different VNets = peering needed |

---

## Quick Revision Questions

1. **How many IP addresses does Azure reserve per subnet?**
   > 5 addresses: .0 (network), .1 (gateway), .2-.3 (DNS), .255 (broadcast).

2. **Can a VNet span multiple Azure regions?**
   > No, a VNet is scoped to a single region. Use VNet Peering for cross-region connectivity.

3. **What is the default communication behavior between subnets in the same VNet?**
   > Subnets within the same VNet can communicate freely by default.

4. **How many usable IP addresses does a /24 subnet have in Azure?**
   > 251 (256 minus 5 reserved addresses).

5. **What CLI command creates a new subnet in an existing VNet?**
   > `az network vnet subnet create --vnet-name myVNet --name mySubnet --address-prefixes 10.0.x.0/24`.

---

[⬅ Previous: Azure Container Instances](../03-Compute-Services/05-azure-container-instances.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Subnets and NSGs ➡](02-subnets-and-nsgs.md)
