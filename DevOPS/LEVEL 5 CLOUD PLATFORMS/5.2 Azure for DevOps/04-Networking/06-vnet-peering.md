# Chapter 6: VNet Peering

## Overview
**VNet Peering** connects two Azure Virtual Networks so resources in each VNet can communicate with each other using private IP addresses. Traffic travels over the Microsoft backbone — not the public internet.

---

## 6.1 VNet Peering Architecture

```
┌──────────────── VNET PEERING ──────────────────┐
│                                                  │
│   ┌──────────────────┐    ┌──────────────────┐  │
│   │  VNet-A           │    │  VNet-B           │  │
│   │  10.1.0.0/16      │    │  10.2.0.0/16      │  │
│   │                   │    │                   │  │
│   │  ┌──────┐        │    │        ┌──────┐  │  │
│   │  │ VM-1 │        │◄──►│        │ VM-2 │  │  │
│   │  │10.1. │        │    │        │10.2. │  │  │
│   │  │0.4   │        │    │        │0.4   │  │  │
│   │  └──────┘        │    │        └──────┘  │  │
│   └──────────────────┘    └──────────────────┘  │
│           ▲                       ▲               │
│           │   Microsoft Backbone  │               │
│           └───────────────────────┘               │
│                                                  │
│   ✅ Low latency, high bandwidth                 │
│   ✅ Private IP communication                    │
│   ❌ NO transitive peering by default            │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 6.2 Types of Peering

| Type | Description | Latency |
|------|-------------|---------|
| **Regional Peering** | VNets in the same Azure region | Same as within VNet |
| **Global Peering** | VNets in different Azure regions | Cross-region (slightly higher) |

---

## 6.3 Key Properties

| Property | Detail |
|----------|--------|
| **Non-transitive** | If A↔B and B↔C, A cannot reach C (unless A↔C peering exists) |
| **No overlapping IPs** | Address spaces must not overlap |
| **Bidirectional setup** | Must create peering from both VNets |
| **No gateway required** | Direct connection, no VPN gateway needed |
| **Cross-subscription** | Peering works across subscriptions |
| **Cross-tenant** | Peering works across Azure AD tenants |

---

## 6.4 Transitive Peering Problem

```
┌────── NON-TRANSITIVE PEERING ──────────────┐
│                                             │
│   VNet-A ◄──peered──► VNet-B ◄──peered──►VNet-C│
│                                             │
│   VNet-A CANNOT reach VNet-C!               │
│                                             │
│   SOLUTIONS:                                │
│                                             │
│   Option 1: Direct peering                  │
│   VNet-A ◄──peered──► VNet-C                │
│                                             │
│   Option 2: Hub-Spoke topology              │
│        ┌─────────┐                          │
│        │  HUB    │                          │
│        │  VNet   │                          │
│        │ (NVA/FW)│                          │
│        └─┬───┬───┘                          │
│          │   │                               │
│     ┌────┘   └────┐                         │
│     ▼              ▼                         │
│  ┌──────┐    ┌──────┐                       │
│  │Spoke │    │Spoke │                       │
│  │VNet-A│    │VNet-B│                       │
│  └──────┘    └──────┘                       │
│  Traffic routes through Hub NVA              │
│                                             │
│   Option 3: Azure Virtual WAN               │
│   (Managed hub with auto-routing)            │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 6.5 Hub-Spoke Topology

```
┌──────────── HUB-SPOKE ARCHITECTURE ────────────┐
│                                                  │
│              ┌──────────────┐                    │
│              │   HUB VNET   │                    │
│              │  10.0.0.0/16 │                    │
│              │              │                    │
│              │ • Azure FW   │                    │
│              │ • VPN Gateway│                    │
│              │ • Bastion    │                    │
│              │ • DNS        │                    │
│              └──┬───┬───┬──┘                    │
│                 │   │   │                        │
│          ┌──────┘   │   └──────┐                │
│          │          │          │                  │
│     ┌────▼────┐ ┌───▼───┐ ┌───▼────┐           │
│     │ SPOKE 1 │ │SPOKE 2│ │SPOKE 3 │           │
│     │  Web    │ │  API  │ │  Data  │           │
│     │10.1./16│ │10.2/16│ │10.3/16 │           │
│     └─────────┘ └───────┘ └────────┘           │
│                                                  │
│   Spokes peer with Hub only                      │
│   Spokes communicate THROUGH Hub                 │
│   Hub centralizes shared services                │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 6.6 Peering Settings

| Setting | Description |
|---------|-------------|
| **Allow forwarded traffic** | Accept traffic that didn't originate from the peered VNet (needed for NVA routing) |
| **Allow gateway transit** | Let the peered VNet use your VPN gateway |
| **Use remote gateways** | Use the peered VNet's VPN gateway for on-premises connectivity |

```
┌────── GATEWAY TRANSIT ──────────────────────┐
│                                              │
│   On-Premises ◄──VPN──► Hub VNet             │
│                          │  (VPN Gateway)    │
│                          │  Allow gateway    │
│                          │  transit = true   │
│                          │                   │
│                    Peered│with               │
│                          │  Use remote       │
│                          │  gateway = true   │
│                          │                   │
│                        Spoke VNet             │
│                        (No gateway needed)    │
│                                              │
│   Spoke VMs can reach on-prem through        │
│   Hub's VPN gateway!                         │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 6.7 Creating VNet Peering

```bash
# Get VNet IDs
VNETA_ID=$(az network vnet show \
  --resource-group rgA \
  --name vnet-a \
  --query id --output tsv)

VNETB_ID=$(az network vnet show \
  --resource-group rgB \
  --name vnet-b \
  --query id --output tsv)

# Create peering: VNet-A → VNet-B
az network vnet peering create \
  --resource-group rgA \
  --name vnetA-to-vnetB \
  --vnet-name vnet-a \
  --remote-vnet $VNETB_ID \
  --allow-vnet-access \
  --allow-forwarded-traffic

# Create peering: VNet-B → VNet-A (MUST do both directions)
az network vnet peering create \
  --resource-group rgB \
  --name vnetB-to-vnetA \
  --vnet-name vnet-b \
  --remote-vnet $VNETA_ID \
  --allow-vnet-access \
  --allow-forwarded-traffic

# Verify peering status
az network vnet peering show \
  --resource-group rgA \
  --name vnetA-to-vnetB \
  --vnet-name vnet-a \
  --query peeringState \
  --output tsv
# Expected output: Connected

# List all peerings
az network vnet peering list \
  --resource-group rgA \
  --vnet-name vnet-a \
  --output table
```

---

## 6.8 Peering States

| State | Meaning |
|-------|---------|
| **Initiated** | Only one side peering created |
| **Connected** | Both sides peering established — traffic flows |
| **Disconnected** | One side deleted — traffic stops |

> ⚠️ You must create peering from **both** VNets. A single-side peering stays in `Initiated` state and traffic won't flow.

---

## 6.9 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Peering state "Initiated" | Only one side created | Create peering from both VNets |
| Overlapping address spaces | VNet CIDRs overlap | Re-address one VNet (requires recreation) |
| Cannot reach peered VMs | NSG blocking traffic | Check NSG rules allow VNet-to-VNet |
| Transitive routing not working | Peering is non-transitive | Use Hub-Spoke with NVA, or add direct peering |
| Gateway transit fails | Settings mismatched | Enable "allow gateway transit" on hub, "use remote gateway" on spoke |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Regional Peering** | Same-region VNets, lowest latency |
| **Global Peering** | Cross-region VNets, slightly higher latency |
| **Non-transitive** | A↔B and B↔C doesn't mean A↔C |
| **Setup** | Must create peering from BOTH sides |
| **Hub-Spoke** | Hub centralizes shared services; spokes peer with hub only |
| **Gateway Transit** | Spokes can use hub's VPN gateway for on-prem connectivity |
| **No overlapping IPs** | Address spaces must be unique across peered VNets |

---

## Quick Revision Questions

1. **Is VNet peering transitive?**
   > No. If VNet-A peers with VNet-B, and VNet-B peers with VNet-C, VNet-A cannot reach VNet-C unless directly peered.

2. **What is the difference between regional and global peering?**
   > Regional peering connects VNets in the same region; global peering connects VNets across different regions.

3. **What happens if you only create peering from one side?**
   > The peering stays in "Initiated" state — traffic won't flow until both sides are configured.

4. **What is hub-spoke topology?**
   > A design where a central hub VNet hosts shared services (firewall, VPN), and spoke VNets peer with the hub for centralized control.

5. **What is gateway transit?**
   > A feature allowing a spoke VNet to use the hub VNet's VPN gateway to reach on-premises networks.

---

[⬅ Previous: Azure DNS](05-azure-dns.md) | [⬆ Back to Table of Contents](../README.md) | [Next: ExpressRoute and VPN ➡](07-expressroute-and-vpn.md)
