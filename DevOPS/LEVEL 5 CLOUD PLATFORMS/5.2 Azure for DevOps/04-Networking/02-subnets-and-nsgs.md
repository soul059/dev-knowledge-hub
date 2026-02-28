# Chapter 2: Subnets and NSGs

## Overview
**Subnets** segment VNets into smaller networks, and **Network Security Groups (NSGs)** act as virtual firewalls controlling inbound/outbound traffic. Together, they form the core of Azure network segmentation and security.

---

## 2.1 NSG Architecture

```
┌──────────────────── NSG CONCEPT ──────────────────────┐
│                                                        │
│   INTERNET                                             │
│      │                                                 │
│      ▼                                                 │
│   ┌──────────────────────────────────────┐            │
│   │  NSG (web-nsg)                       │            │
│   │  ┌────────────────────────────────┐  │            │
│   │  │ INBOUND RULES (priority order) │  │            │
│   │  │ 100: Allow HTTP (80)    ✅      │  │            │
│   │  │ 110: Allow HTTPS (443)  ✅      │  │            │
│   │  │ 120: Allow SSH (22)     ✅      │  │            │
│   │  │ 65000: Allow VNet       ✅      │  │            │
│   │  │ 65001: Allow LB         ✅      │  │            │
│   │  │ 65500: Deny All         ❌      │  │            │
│   │  └────────────────────────────────┘  │            │
│   │  ┌────────────────────────────────┐  │            │
│   │  │ OUTBOUND RULES                 │  │            │
│   │  │ 65000: Allow VNet       ✅      │  │            │
│   │  │ 65001: Allow Internet   ✅      │  │            │
│   │  │ 65500: Deny All         ❌      │  │            │
│   │  └────────────────────────────────┘  │            │
│   └──────────────────────────────────────┘            │
│      │                                                 │
│      ▼                                                 │
│   ┌──────────────────────────────────────┐            │
│   │  Subnet: web-subnet                  │            │
│   │  ┌──────┐  ┌──────┐  ┌──────┐       │            │
│   │  │ VM-1 │  │ VM-2 │  │ VM-3 │       │            │
│   │  └──────┘  └──────┘  └──────┘       │            │
│   └──────────────────────────────────────┘            │
│                                                        │
│   NSGs can be applied to:                              │
│   • Subnet (affects all resources in subnet)           │
│   • NIC (affects specific VM)                          │
│   • Both (both sets of rules are evaluated)            │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## 2.2 NSG Rule Properties

| Property | Description |
|----------|-------------|
| **Priority** | 100-4096; lower number = higher priority |
| **Source** | IP, Service Tag, ASG, or Any |
| **Destination** | IP, Service Tag, ASG, or Any |
| **Port** | Single, range (80-443), or * (all) |
| **Protocol** | TCP, UDP, ICMP, or Any |
| **Action** | Allow or Deny |
| **Direction** | Inbound or Outbound |

### Default Rules (Cannot be deleted)

| Priority | Name | Direction | Action |
|----------|------|-----------|--------|
| 65000 | AllowVnetInBound | Inbound | Allow VNet-to-VNet |
| 65001 | AllowAzureLoadBalancerInBound | Inbound | Allow LB health probes |
| 65500 | DenyAllInBound | Inbound | **Deny everything else** |
| 65000 | AllowVnetOutBound | Outbound | Allow VNet-to-VNet |
| 65001 | AllowInternetOutBound | Outbound | Allow to internet |
| 65500 | DenyAllOutBound | Outbound | **Deny everything else** |

---

## 2.3 Service Tags

Service tags represent groups of IP addresses for Azure services, eliminating the need to manage IPs manually.

| Service Tag | Description |
|-------------|-------------|
| `Internet` | Public internet IPs |
| `VirtualNetwork` | All VNet address spaces including peered VNets |
| `AzureLoadBalancer` | Azure's health probe source |
| `AzureCloud` | All Azure datacenter IPs |
| `AzureCloud.EastUS` | Azure IPs in East US |
| `Sql` | Azure SQL Database IPs |
| `Storage` | Azure Storage IPs |
| `AzureKeyVault` | Key Vault IPs |

---

## 2.4 Creating and Managing NSGs

```bash
# Create NSG
az network nsg create \
  --resource-group myRG \
  --name web-nsg \
  --location eastus

# Add inbound rules
az network nsg rule create \
  --resource-group myRG \
  --nsg-name web-nsg \
  --name AllowHTTP \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 80

az network nsg rule create \
  --resource-group myRG \
  --nsg-name web-nsg \
  --name AllowHTTPS \
  --priority 110 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 443

az network nsg rule create \
  --resource-group myRG \
  --nsg-name web-nsg \
  --name AllowSSH \
  --priority 120 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "203.0.113.0/24" \
  --destination-port-ranges 22

# Associate NSG with subnet
az network vnet subnet update \
  --resource-group myRG \
  --vnet-name myVNet \
  --name web-subnet \
  --network-security-group web-nsg

# List NSG rules
az network nsg rule list --resource-group myRG --nsg-name web-nsg --output table
```

---

## 2.5 NSG Flow Evaluation

```
┌────────── NSG RULE EVALUATION ORDER ──────────┐
│                                                 │
│   INBOUND TRAFFIC:                              │
│                                                 │
│   Packet Arrives                                │
│       │                                         │
│       ▼                                         │
│   ┌──────────────────┐                          │
│   │ Subnet-level NSG │  (if associated)         │
│   │ Rules evaluated  │                          │
│   │ lowest priority# │                          │
│   │ first (100→4096) │                          │
│   └────────┬─────────┘                          │
│            │ If ALLOWED                         │
│            ▼                                    │
│   ┌──────────────────┐                          │
│   │  NIC-level NSG   │  (if associated)         │
│   │ Rules evaluated  │                          │
│   │ lowest priority# │                          │
│   │ first (100→4096) │                          │
│   └────────┬─────────┘                          │
│            │ If ALLOWED                         │
│            ▼                                    │
│      DELIVERED TO VM                            │
│                                                 │
│   Traffic must pass BOTH NSGs (if both exist)   │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## 2.6 Application Security Groups (ASGs)

ASGs let you group VMs logically and apply NSG rules by group name instead of IP addresses.

```
┌──────── APPLICATION SECURITY GROUPS ────────┐
│                                              │
│   ASG: "web-servers"    ASG: "db-servers"   │
│   ┌──────┐ ┌──────┐    ┌──────┐            │
│   │ VM-1 │ │ VM-2 │    │ VM-3 │            │
│   └──────┘ └──────┘    └──────┘            │
│                                              │
│   NSG Rule:                                  │
│   "Allow port 3306 FROM web-servers          │
│    TO db-servers"                             │
│                                              │
│   ✅ No need to manage IP addresses!         │
│   ✅ VMs can join/leave ASGs dynamically     │
│                                              │
└──────────────────────────────────────────────┘
```

```bash
# Create ASGs
az network asg create --resource-group myRG --name web-servers
az network asg create --resource-group myRG --name db-servers

# NSG rule using ASGs
az network nsg rule create \
  --resource-group myRG \
  --nsg-name myNSG \
  --name AllowWebToDb \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-asgs web-servers \
  --destination-asgs db-servers \
  --destination-port-ranges 3306
```

---

## 2.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot SSH to VM | NSG blocking port 22 | Add inbound rule for port 22 |
| VMs can't communicate | NSG too restrictive | Check default VNet allow rule isn't overridden |
| Web app unreachable | HTTP/HTTPS ports not opened | Add rules for ports 80/443 |
| NSG changes not applying | Propagation delay | Wait ~1 minute; check effective rules |
| Rule conflicts | Multiple rules matching same traffic | Lower priority number wins |

💡 **Best Practice:** Apply NSGs at the subnet level for broad rules, and at the NIC level for VM-specific rules.

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **NSGs** | Virtual firewalls with prioritized allow/deny rules |
| **Priority** | 100-4096; lower = higher priority |
| **Default Rules** | VNet allow, LB allow, Deny All (cannot delete) |
| **Service Tags** | Named IP groups for Azure services |
| **ASGs** | Group VMs logically for NSG rules without IP management |
| **Evaluation** | Subnet NSG first, then NIC NSG; both must allow |

---

## Quick Revision Questions

1. **What is the default behavior of an NSG with no custom rules?**
   > It allows VNet-to-VNet and Load Balancer traffic, allows outbound internet, and denies all other inbound traffic.

2. **How are NSG rules evaluated?**
   > By priority (lowest number first, 100-4096). The first matching rule wins.

3. **What are Service Tags?**
   > Predefined groups of Azure IP addresses (e.g., AzureCloud, Storage, Sql) used in NSG rules instead of manual IPs.

4. **What is the benefit of Application Security Groups?**
   > They let you group VMs logically and reference them in NSG rules by name instead of IP addresses.

5. **If an NSG is applied to both the subnet and the NIC, how is traffic evaluated?**
   > Both NSGs are checked — inbound traffic must pass the subnet NSG first, then the NIC NSG.

---

[⬅ Previous: Virtual Networks](01-virtual-networks.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Load Balancer ➡](03-azure-load-balancer.md)
