# Chapter 2: Regions and Availability Zones

## Overview
Azure Regions and Availability Zones are the building blocks for designing highly available and disaster-resilient applications. DevOps engineers must understand how to leverage these to achieve SLA targets and minimize downtime.

---

## 2.1 Azure Regions

An Azure **Region** is a set of datacenters deployed within a latency-defined perimeter and connected through a dedicated regional low-latency network.

### Region Categories

| Category | Description | Example |
|----------|-------------|---------|
| **Recommended** | Most services available; support Availability Zones | East US, West Europe |
| **Alternate** | Extend Azure's footprint; may not support AZs | South India, West US |
| **Sovereign** | Isolated for government/compliance | US Gov Virginia, China East |

### Popular Regions for DevOps Workloads

```
    ┌──────────────────────────────────────────────────┐
    │              POPULAR AZURE REGIONS                │
    ├──────────────────────────────────────────────────┤
    │                                                  │
    │   AMERICAS          EUROPE          ASIA-PACIFIC │
    │   ─────────         ──────          ──────────── │
    │   East US           West Europe     Southeast Asia│
    │   East US 2         North Europe    East Asia     │
    │   West US 2         UK South        Australia East│
    │   Central US        France Central  Japan East    │
    │   Canada Central    Germany WC      Korea Central │
    │   Brazil South      Switzerland N   Central India │
    │                                                  │
    └──────────────────────────────────────────────────┘
```

---

## 2.2 Availability Zones (AZs)

Availability Zones are **physically separate locations** within an Azure region. Each zone is made up of one or more datacenters equipped with independent power, cooling, and networking.

```
┌────────────────────── AZURE REGION (e.g., East US 2) ──────────────────────┐
│                                                                             │
│   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐           │
│   │  ZONE 1         │  │  ZONE 2         │  │  ZONE 3         │           │
│   │                 │  │                 │  │                 │           │
│   │ ┌────────────┐  │  │ ┌────────────┐  │  │ ┌────────────┐  │           │
│   │ │Datacenter A│  │  │ │Datacenter C│  │  │ │Datacenter E│  │           │
│   │ └────────────┘  │  │ └────────────┘  │  │ └────────────┘  │           │
│   │ ┌────────────┐  │  │ ┌────────────┐  │  │ ┌────────────┐  │           │
│   │ │Datacenter B│  │  │ │Datacenter D│  │  │ │Datacenter F│  │           │
│   │ └────────────┘  │  │ └────────────┘  │  │ └────────────┘  │           │
│   │                 │  │                 │  │                 │           │
│   │ Independent:    │  │ Independent:    │  │ Independent:    │           │
│   │ • Power         │  │ • Power         │  │ • Power         │           │
│   │ • Cooling       │  │ • Cooling       │  │ • Cooling       │           │
│   │ • Networking    │  │ • Networking    │  │ • Networking    │           │
│   └─────────────────┘  └─────────────────┘  └─────────────────┘           │
│              │                  │                    │                      │
│              └──────────────────┼────────────────────┘                      │
│                      High-speed │fiber (< 2ms latency)                     │
│                                 │                                           │
└─────────────────────────────────────────────────────────────────────────────┘
```

### AZ Key Facts

- **Minimum 3 zones** per enabled region
- **< 2ms round-trip latency** between zones
- Each zone is an **independent failure domain**
- Zone numbers are **logically mapped per subscription** (Zone 1 in your subscription may map to a different physical zone than Zone 1 in another subscription)

---

## 2.3 Service Categories by AZ Support

| Category | Description | Examples |
|----------|-------------|---------|
| **Zonal Services** | Pinned to a specific zone | VMs, Managed Disks, Public IPs |
| **Zone-Redundant Services** | Automatically replicated across zones | Zone-redundant Storage, SQL Database |
| **Non-Regional Services** | No region dependency | Azure AD, Traffic Manager, Azure Front Door |

```
┌──────────────────── SERVICE ZONE CATEGORIES ────────────────────┐
│                                                                  │
│  ZONAL                ZONE-REDUNDANT        NON-REGIONAL        │
│  ──────               ──────────────        ────────────        │
│  ┌──────┐             ┌──────────┐          ┌──────────┐        │
│  │ VM   │──▶ Zone 1   │   ZRS    │──▶ All   │ Azure AD │        │
│  └──────┘             │ Storage  │   Zones  │ Traffic  │        │
│  ┌──────┐             └──────────┘          │ Manager  │        │
│  │ Disk │──▶ Zone 2   ┌──────────┐          └──────────┘        │
│  └──────┘             │ SQL DB   │                              │
│  ┌──────┐             │ (zone-   │                              │
│  │Public│──▶ Zone 3   │redundant)│                              │
│  │  IP  │             └──────────┘                              │
│  └──────┘                                                        │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2.4 Region Pairs

Each Azure region is paired with another region within the same geography to form a **region pair**. Pair relationships are fixed by Microsoft.

### Benefits of Region Pairs

| Benefit | Description |
|---------|-------------|
| **Sequential Updates** | Planned updates are rolled out to one region at a time |
| **Prioritized Recovery** | In a broad outage, one region from each pair is prioritized |
| **Data Residency** | Both regions stay within the same geography |
| **Physical Isolation** | Pairs are at least 300 miles apart |

### Common Region Pairs

```
    Region A                    Region B
    ────────                    ────────
    East US         ◄═══════►  West US
    East US 2       ◄═══════►  Central US
    North Europe    ◄═══════►  West Europe
    UK South        ◄═══════►  UK West
    Southeast Asia  ◄═══════►  East Asia
    Australia East  ◄═══════►  Australia SE
    Canada Central  ◄═══════►  Canada East
    Japan East      ◄═══════►  Japan West
```

### Checking Region Pairs with CLI

```bash
# List all regions with their paired region
az account list-locations \
  --query "[].{Region:name, DisplayName:displayName, Paired:metadata.pairedRegion[0].name}" \
  --output table

# Output example:
# Region          DisplayName       Paired
# ----------      ---------------   ----------
# eastus          East US           westus
# westeurope      West Europe       northeurope
```

---

## 2.5 Designing for High Availability

### SLA Comparison with AZ Architecture

| Deployment Strategy | SLA | Description |
|--------------------|-----|-------------|
| Single VM (Premium SSD) | 99.9% | ~8.76 hours downtime/year |
| Availability Set | 99.95% | ~4.38 hours downtime/year |
| Availability Zones | 99.99% | ~52.56 minutes downtime/year |
| Multi-Region | 99.999%+ | ~5.26 minutes downtime/year |

### HA Architecture Pattern

```
                         ┌───────────────┐
                         │  Azure Front  │
                         │     Door      │
                         └───────┬───────┘
                                 │
                    ┌────────────┼────────────┐
                    │                         │
           ┌────────┴────────┐      ┌────────┴────────┐
           │   REGION 1      │      │   REGION 2      │
           │  (Primary)      │      │  (Secondary)    │
           │                 │      │                 │
           │ Zone1  Zone2    │      │ Zone1  Zone2    │
           │ ┌──┐   ┌──┐    │      │ ┌──┐   ┌──┐    │
           │ │VM│   │VM│    │      │ │VM│   │VM│    │
           │ └──┘   └──┘    │      │ └──┘   └──┘    │
           │       │         │      │       │         │
           │ ┌─────┴──────┐  │      │ ┌─────┴──────┐  │
           │ │  Zone-Red.  │  │      │ │  Zone-Red.  │  │
           │ │  SQL DB     │──┼──────┼─│  SQL DB     │  │
           │ │             │  │  Geo │ │  (replica)  │  │
           │ └─────────────┘  │  Rep │ └─────────────┘  │
           └──────────────────┘      └──────────────────┘
```

---

## 2.6 Hands-On: Working with Regions and Zones

### Azure CLI Commands

```bash
# List all available regions for your subscription
az account list-locations --output table

# Check which services are available in a specific region
az provider show --namespace Microsoft.Compute \
  --query "resourceTypes[?resourceType=='virtualMachines'].locations" \
  --output table

# Create a VM in a specific availability zone
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --location eastus2 \
  --zone 1 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys

# Create zone-redundant storage
az storage account create \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --location eastus2 \
  --sku Standard_ZRS
```

### Azure PowerShell Commands

```powershell
# List regions
Get-AzLocation | Select-Object DisplayName, Location | Format-Table

# Create VM in specific zone
New-AzVM -ResourceGroupName "myRG" `
  -Location "eastus2" `
  -Zone 1 `
  -Name "myVM" `
  -Image "Ubuntu2204" `
  -Size "Standard_B2s"

# Check zone support for a region
Get-AzComputeResourceSku | Where-Object {
  $_.Locations -contains "eastus2" -and 
  $_.ResourceType -eq "virtualMachines" -and 
  $_.LocationInfo.Zones
} | Select-Object Name -First 10
```

---

## 2.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot create resource in specific zone | VM size not available in that zone | Use `az vm list-skus --location <region> --zone` to check |
| Availability Zone not supported | Region does not support AZs | Choose a region with AZ support |
| Cross-zone latency higher than expected | Network misconfiguration | Ensure resources use Azure backbone, not public internet |
| Cannot pair custom regions | Region pairs are fixed by Microsoft | Design DR around Microsoft's defined pairs |
| Zone mapping confusion | Zone numbers are per-subscription | Use `az rest` API to check physical zone mapping |

⚠️ **Common Pitfall:** Don't assume Zone 1 in your subscription is the same physical zone as Zone 1 in another subscription. Azure randomizes mapping for load balancing.

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Regions** | 60+ locations; choose based on latency, compliance, and service availability |
| **Availability Zones** | 3+ per enabled region; independent failure domains; < 2ms latency between zones |
| **Zonal Services** | Deployed to specific zone (VMs, Disks) |
| **Zone-Redundant** | Auto-replicated across zones (ZRS, SQL DB) |
| **Region Pairs** | Fixed pairings for DR; sequential updates; 300+ miles apart |
| **SLA Impact** | Single VM 99.9% → AZ 99.99% → Multi-region 99.999%+ |
| **Zone Mapping** | Logical-to-physical mapping is per subscription |

---

## Quick Revision Questions

1. **How many Availability Zones does an AZ-enabled region have at minimum?**
   > At least 3 Availability Zones, each with independent power, cooling, and networking.

2. **What is the difference between a Zonal service and a Zone-Redundant service?**
   > A Zonal service is pinned to a specific zone (e.g., a VM in Zone 1), while a Zone-Redundant service is automatically replicated across all zones (e.g., ZRS storage).

3. **Why are region pairs important for disaster recovery?**
   > They ensure sequential updates, prioritized recovery during outages, and data residency within the same geography, with at least 300 miles of physical separation.

4. **What SLA does an Availability Zone deployment provide for VMs?**
   > 99.99% uptime SLA (~52.56 minutes of downtime per year).

5. **Can you change Azure's region pair assignments?**
   > No, region pairs are fixed by Microsoft and cannot be customized.

6. **What CLI command creates a VM in a specific Availability Zone?**
   > `az vm create --zone 1 --location eastus2 ...` — the `--zone` parameter specifies the AZ.

---

[⬅ Previous: Azure Global Infrastructure](01-azure-global-infrastructure.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Portal, CLI, PowerShell ➡](03-azure-portal-cli-powershell.md)
