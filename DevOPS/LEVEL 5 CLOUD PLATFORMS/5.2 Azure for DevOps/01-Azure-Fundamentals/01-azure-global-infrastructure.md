# Chapter 1: Azure Global Infrastructure

## Overview
Microsoft Azure's global infrastructure is the physical foundation upon which all Azure services run. Understanding this infrastructure is critical for DevOps engineers who need to design resilient, performant, and compliant cloud architectures.

---

## 1.1 What is Azure Global Infrastructure?

Azure's global infrastructure consists of **physical datacenters** organized into logical groupings that provide **redundancy, low latency, and compliance** with data residency requirements.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    AZURE GLOBAL INFRASTRUCTURE                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                    GEOGRAPHY                                 │       │
│   │   (e.g., United States, Europe, Asia Pacific)               │       │
│   │                                                              │       │
│   │   ┌───────────────────────┐  ┌───────────────────────┐      │       │
│   │   │      REGION           │  │      REGION           │      │       │
│   │   │   (e.g., East US)     │  │  (e.g., West US)      │      │       │
│   │   │                       │  │                       │      │       │
│   │   │  ┌─────┐ ┌─────┐     │  │  ┌─────┐ ┌─────┐     │      │       │
│   │   │  │ AZ1 │ │ AZ2 │     │  │  │ AZ1 │ │ AZ2 │     │      │       │
│   │   │  │     │ │     │     │  │  │     │ │     │     │      │       │
│   │   │  │ DC  │ │ DC  │     │  │  │ DC  │ │ DC  │     │      │       │
│   │   │  │ DC  │ │ DC  │     │  │  │ DC  │ │ DC  │     │      │       │
│   │   │  └─────┘ └─────┘     │  │  └─────┘ └─────┘     │      │       │
│   │   │          ┌─────┐     │  │          ┌─────┐     │      │       │
│   │   │          │ AZ3 │     │  │          │ AZ3 │     │      │       │
│   │   │          │ DC  │     │  │          │ DC  │     │      │       │
│   │   │          │ DC  │     │  │          │ DC  │     │      │       │
│   │   │          └─────┘     │  │          └─────┘     │      │       │
│   │   └───────────────────────┘  └───────────────────────┘      │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                         │
│   DC = Datacenter    AZ = Availability Zone                             │
└─────────────────────────────────────────────────────────────────────────┘
```

### Key Components

| Component | Description | Example |
|-----------|-------------|---------|
| **Geography** | A discrete market containing one or more regions | United States, Europe, Asia Pacific |
| **Region** | A set of datacenters within a latency-defined perimeter | East US, West Europe, Southeast Asia |
| **Availability Zone** | Physically separate locations within a region | Zone 1, Zone 2, Zone 3 |
| **Datacenter** | Physical facility with independent power/cooling/networking | Individual buildings |
| **Edge Location** | CDN/networking point of presence | Azure CDN PoPs worldwide |

---

## 1.2 Azure Datacenters

Each Azure datacenter is designed with:

- **Independent power supplies** — Multiple utility feeds, backup generators
- **Independent cooling systems** — Redundant HVAC systems
- **Independent networking** — Multiple fiber connections
- **Physical security** — Biometric access, 24/7 surveillance, mantraps

```
┌─────────────────────────────────────────┐
│           AZURE DATACENTER              │
├─────────────────────────────────────────┤
│                                         │
│   ┌──────────┐    ┌──────────────┐      │
│   │  Power   │    │   Cooling    │      │
│   │ ─ Grid A │    │ ─ HVAC A     │      │
│   │ ─ Grid B │    │ ─ HVAC B     │      │
│   │ ─ UPS    │    │ ─ Liquid     │      │
│   │ ─ Diesel │    │   cooling    │      │
│   └──────────┘    └──────────────┘      │
│                                         │
│   ┌──────────┐    ┌──────────────┐      │
│   │ Network  │    │  Security    │      │
│   │ ─ ISP A  │    │ ─ Biometric  │      │
│   │ ─ ISP B  │    │ ─ Cameras    │      │
│   │ ─ Azure  │    │ ─ Guards     │      │
│   │   backbone│    │ ─ Mantraps   │      │
│   └──────────┘    └──────────────┘      │
│                                         │
│   ┌─────────────────────────────┐       │
│   │      Server Racks          │       │
│   │   ╔═══╗ ╔═══╗ ╔═══╗ ╔═══╗ │       │
│   │   ║   ║ ║   ║ ║   ║ ║   ║ │       │
│   │   ║   ║ ║   ║ ║   ║ ║   ║ │       │
│   │   ╚═══╝ ╚═══╝ ╚═══╝ ╚═══╝ │       │
│   └─────────────────────────────┘       │
└─────────────────────────────────────────┘
```

---

## 1.3 Azure Global Network (Backbone)

Azure operates one of the **largest private networks** in the world:

- **Over 200,000 miles** of fiber optic cable
- **170+ edge sites** globally
- **60+ regions** in over 140 countries
- Traffic stays on Microsoft's private backbone as much as possible (**cold potato routing**)

```
                    AZURE GLOBAL BACKBONE
                    
    ┌────────┐         ┌────────┐         ┌────────┐
    │Americas│◄═══════►│ Europe │◄═══════►│  Asia  │
    │ Regions│         │ Regions│         │ Regions│
    └───┬────┘         └───┬────┘         └───┬────┘
        │                  │                  │
        ▼                  ▼                  ▼
    ┌────────┐         ┌────────┐         ┌────────┐
    │  Edge  │         │  Edge  │         │  Edge  │
    │  Sites │         │  Sites │         │  Sites │
    └────────┘         └────────┘         └────────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    ┌──────┴──────┐
                    │   Internet  │
                    │    Users    │
                    └─────────────┘
```

### Cold Potato vs. Hot Potato Routing

| Routing Type | Description | Azure Approach |
|-------------|-------------|----------------|
| **Cold Potato** | Traffic enters Microsoft's network early and stays on it | Default behavior — better performance |
| **Hot Potato** | Traffic stays on internet as long as possible | Cheaper but higher latency |

---

## 1.4 Azure Sovereign Clouds

Azure offers **separate, isolated** cloud instances for specific compliance needs:

| Cloud | Purpose | Regions |
|-------|---------|---------|
| **Azure Public** | General commercial use | 60+ regions worldwide |
| **Azure Government** | US government workloads (FedRAMP, ITAR) | US Gov Virginia, US Gov Arizona, etc. |
| **Azure China** | Operated by 21Vianet | China East, China North |

💡 **Best Practice:** Sovereign clouds are physically and logically isolated from Azure Public. Resources cannot be moved between them.

---

## 1.5 Real-World DevOps Scenarios

### Scenario 1: Multi-Region Deployment
A DevOps team deploys a web application across East US and West Europe for global users:

```
                    ┌──────────────┐
                    │ Azure Traffic │
                    │   Manager    │
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              ▼                         ▼
    ┌─────────────────┐      ┌─────────────────┐
    │    East US       │      │   West Europe    │
    │  ┌───────────┐   │      │  ┌───────────┐   │
    │  │ App Service│   │      │  │ App Service│   │
    │  └─────┬─────┘   │      │  └─────┬─────┘   │
    │        │          │      │        │          │
    │  ┌─────┴─────┐   │      │  ┌─────┴─────┐   │
    │  │  SQL DB   │   │      │  │  SQL DB    │   │
    │  │ (Primary) │   │      │  │ (Replica)  │   │
    │  └───────────┘   │      │  └───────────┘   │
    └─────────────────┘      └─────────────────┘
```

### Scenario 2: Disaster Recovery
Configure geo-redundant backups across paired regions:

```bash
# Check available regions
az account list-locations --output table

# Create resource groups in paired regions
az group create --name rg-primary --location eastus
az group create --name rg-dr --location westus

# List region pairs
az account list-locations --query "[].{Name:name, Paired:metadata.pairedRegion[0].name}" --output table
```

---

## 1.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Service unavailable in a region | Not all services are available in every region | Check [Azure Products by Region](https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/) |
| High latency | Resources deployed far from users | Use Azure Traffic Manager or Front Door for geographic routing |
| Compliance violation | Data stored in wrong geography | Use Azure Policy to restrict allowed regions |
| Region capacity limits | Popular regions may have capacity constraints | Use alternate regions or request quota increase |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Geographies** | Contain one or more regions; define data residency boundaries |
| **Regions** | 60+ worldwide; contain one or more datacenters |
| **Availability Zones** | 3+ per enabled region; physically separate facilities |
| **Datacenters** | Independent power, cooling, networking |
| **Global Backbone** | 200K+ miles of private fiber; cold potato routing |
| **Sovereign Clouds** | Isolated instances (Government, China) |
| **Region Pairs** | Automatic pairing for disaster recovery |

---

## Quick Revision Questions

1. **What is the difference between a Region and an Availability Zone in Azure?**
   > A Region is a geographic area containing one or more datacenters. An Availability Zone is a physically separate location within a region with independent power, cooling, and networking.

2. **What is cold potato routing and why does Azure use it?**
   > Cold potato routing means traffic enters Microsoft's private network as early as possible and stays on it. This reduces latency and improves security compared to routing through the public internet.

3. **Name three Azure Sovereign Clouds and their purposes.**
   > Azure Public (commercial), Azure Government (US government compliance), Azure China (operated by 21Vianet for Chinese regulations).

4. **Why are region pairs important for DevOps?**
   > Region pairs enable disaster recovery with prioritized recovery, sequential updates (to avoid simultaneous outages), and data residency compliance within the same geography.

5. **What CLI command lists available Azure regions?**
   > `az account list-locations --output table`

6. **How does Azure ensure datacenter physical security?**
   > Through biometric access controls, 24/7 surveillance cameras, security guards, mantraps, and multi-layered perimeter security.

---

[⬆ Back to Table of Contents](../README.md) | [Next: Regions and Availability Zones ➡](02-regions-and-availability-zones.md)
