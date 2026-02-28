# Azure for DevOps - Comprehensive Study Notes

## Course Overview

This comprehensive guide covers **Azure for DevOps** — the essential knowledge and hands-on skills needed to leverage Microsoft Azure's cloud platform for modern DevOps workflows. From foundational infrastructure concepts to advanced container orchestration, CI/CD pipelines, monitoring, and security, these notes provide a complete reference for practitioners and exam candidates alike.

```
┌─────────────────────────────────────────────────────────────────────┐
│                     AZURE FOR DEVOPS                                │
│                   Learning Path Overview                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
│   │ Azure    │  │ Identity │  │ Compute  │  │Networking│          │
│   │Fundament.│─▶│ & Access │─▶│ Services │─▶│          │          │
│   └──────────┘  └──────────┘  └──────────┘  └──────────┘          │
│        │                                          │                 │
│        ▼                                          ▼                 │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
│   │ Storage  │  │ Database │  │Container │  │ Azure    │          │
│   │ Services │─▶│ Services │─▶│ Services │─▶│ DevOps   │          │
│   └──────────┘  └──────────┘  └──────────┘  └──────────┘          │
│        │                                          │                 │
│        ▼                                          ▼                 │
│   ┌──────────────────┐        ┌──────────────────┐                 │
│   │   Monitoring &   │───────▶│    Security      │                 │
│   │    Logging       │        │                  │                 │
│   └──────────────────┘        └──────────────────┘                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Complete Table of Contents

### Unit 1: Azure Fundamentals
| # | Chapter | File |
|---|---------|------|
| 1 | Azure Global Infrastructure | [01-azure-global-infrastructure.md](01-Azure-Fundamentals/01-azure-global-infrastructure.md) |
| 2 | Regions and Availability Zones | [02-regions-and-availability-zones.md](01-Azure-Fundamentals/02-regions-and-availability-zones.md) |
| 3 | Azure Portal, CLI, PowerShell | [03-azure-portal-cli-powershell.md](01-Azure-Fundamentals/03-azure-portal-cli-powershell.md) |
| 4 | Azure Resource Manager (ARM) | [04-azure-resource-manager.md](01-Azure-Fundamentals/04-azure-resource-manager.md) |
| 5 | Resource Groups and Subscriptions | [05-resource-groups-and-subscriptions.md](01-Azure-Fundamentals/05-resource-groups-and-subscriptions.md) |

### Unit 2: Identity and Access
| # | Chapter | File |
|---|---------|------|
| 1 | Azure Active Directory (Entra ID) | [01-azure-active-directory.md](02-Identity-and-Access/01-azure-active-directory.md) |
| 2 | Service Principals | [02-service-principals.md](02-Identity-and-Access/02-service-principals.md) |
| 3 | Managed Identities | [03-managed-identities.md](02-Identity-and-Access/03-managed-identities.md) |
| 4 | RBAC Roles | [04-rbac-roles.md](02-Identity-and-Access/04-rbac-roles.md) |
| 5 | Azure Policy | [05-azure-policy.md](02-Identity-and-Access/05-azure-policy.md) |

### Unit 3: Compute Services
| # | Chapter | File |
|---|---------|------|
| 1 | Virtual Machines | [01-virtual-machines.md](03-Compute-Services/01-virtual-machines.md) |
| 2 | VM Scale Sets | [02-vm-scale-sets.md](03-Compute-Services/02-vm-scale-sets.md) |
| 3 | Azure App Service | [03-azure-app-service.md](03-Compute-Services/03-azure-app-service.md) |
| 4 | Azure Functions | [04-azure-functions.md](03-Compute-Services/04-azure-functions.md) |
| 5 | Azure Container Instances | [05-azure-container-instances.md](03-Compute-Services/05-azure-container-instances.md) |

### Unit 4: Networking
| # | Chapter | File |
|---|---------|------|
| 1 | Virtual Networks (VNet) | [01-virtual-networks.md](04-Networking/01-virtual-networks.md) |
| 2 | Subnets and NSGs | [02-subnets-and-nsgs.md](04-Networking/02-subnets-and-nsgs.md) |
| 3 | Azure Load Balancer | [03-azure-load-balancer.md](04-Networking/03-azure-load-balancer.md) |
| 4 | Application Gateway | [04-application-gateway.md](04-Networking/04-application-gateway.md) |
| 5 | Azure DNS | [05-azure-dns.md](04-Networking/05-azure-dns.md) |
| 6 | VNet Peering | [06-vnet-peering.md](04-Networking/06-vnet-peering.md) |
| 7 | ExpressRoute and VPN | [07-expressroute-and-vpn.md](04-Networking/07-expressroute-and-vpn.md) |

### Unit 5: Storage Services
| # | Chapter | File |
|---|---------|------|
| 1 | Blob Storage | [01-blob-storage.md](05-Storage-Services/01-blob-storage.md) |
| 2 | Azure Files | [02-azure-files.md](05-Storage-Services/02-azure-files.md) |
| 3 | Managed Disks | [03-managed-disks.md](05-Storage-Services/03-managed-disks.md) |
| 4 | Storage Accounts | [04-storage-accounts.md](05-Storage-Services/04-storage-accounts.md) |
| 5 | Storage Tiers | [05-storage-tiers.md](05-Storage-Services/05-storage-tiers.md) |

### Unit 6: Database Services
| # | Chapter | File |
|---|---------|------|
| 1 | Azure SQL Database | [01-azure-sql-database.md](06-Database-Services/01-azure-sql-database.md) |
| 2 | Cosmos DB | [02-cosmos-db.md](06-Database-Services/02-cosmos-db.md) |
| 3 | Azure Database for MySQL/PostgreSQL | [03-azure-database-mysql-postgresql.md](06-Database-Services/03-azure-database-mysql-postgresql.md) |
| 4 | Azure Cache for Redis | [04-azure-cache-for-redis.md](06-Database-Services/04-azure-cache-for-redis.md) |

### Unit 7: Container Services
| # | Chapter | File |
|---|---------|------|
| 1 | Azure Container Registry (ACR) | [01-azure-container-registry.md](07-Container-Services/01-azure-container-registry.md) |
| 2 | Azure Kubernetes Service (AKS) | [02-azure-kubernetes-service.md](07-Container-Services/02-azure-kubernetes-service.md) |
| 3 | Azure Container Apps | [03-azure-container-apps.md](07-Container-Services/03-azure-container-apps.md) |
| 4 | Azure Container Instances | [04-azure-container-instances.md](07-Container-Services/04-azure-container-instances.md) |

### Unit 8: Azure DevOps Services
| # | Chapter | File |
|---|---------|------|
| 1 | Azure Repos | [01-azure-repos.md](08-Azure-DevOps-Services/01-azure-repos.md) |
| 2 | Azure Pipelines | [02-azure-pipelines.md](08-Azure-DevOps-Services/02-azure-pipelines.md) |
| 3 | Azure Boards | [03-azure-boards.md](08-Azure-DevOps-Services/03-azure-boards.md) |
| 4 | Azure Test Plans | [04-azure-test-plans.md](08-Azure-DevOps-Services/04-azure-test-plans.md) |
| 5 | Azure Artifacts | [05-azure-artifacts.md](08-Azure-DevOps-Services/05-azure-artifacts.md) |
| 6 | YAML Pipelines | [06-yaml-pipelines.md](08-Azure-DevOps-Services/06-yaml-pipelines.md) |

### Unit 9: Monitoring and Logging
| # | Chapter | File |
|---|---------|------|
| 1 | Azure Monitor | [01-azure-monitor.md](09-Monitoring-and-Logging/01-azure-monitor.md) |
| 2 | Log Analytics | [02-log-analytics.md](09-Monitoring-and-Logging/02-log-analytics.md) |
| 3 | Application Insights | [03-application-insights.md](09-Monitoring-and-Logging/03-application-insights.md) |
| 4 | Azure Alerts | [04-azure-alerts.md](09-Monitoring-and-Logging/04-azure-alerts.md) |
| 5 | Azure Dashboards | [05-azure-dashboards.md](09-Monitoring-and-Logging/05-azure-dashboards.md) |
| 6 | Azure Diagnostics | [06-azure-diagnostics.md](09-Monitoring-and-Logging/06-azure-diagnostics.md) |

### Unit 10: Security
| # | Chapter | File |
|---|---------|------|
| 1 | Azure Key Vault | [01-azure-key-vault.md](10-Security/01-azure-key-vault.md) |
| 2 | Azure Security Center (Defender) | [02-azure-security-center.md](10-Security/02-azure-security-center.md) |
| 3 | Network Security | [03-network-security.md](10-Security/03-network-security.md) |
| 4 | Encryption and Certificates | [04-encryption-and-certificates.md](10-Security/04-encryption-and-certificates.md) |
| 5 | Compliance and Governance | [05-compliance-and-governance.md](10-Security/05-compliance-and-governance.md) |

---

## How to Use These Notes

1. **Sequential Study** — Follow the units in order for a progressive learning path
2. **Quick Reference** — Jump to any specific topic using the table of contents links
3. **Exam Prep** — Use the revision questions at the end of each chapter
4. **Hands-On Practice** — Follow the CLI/PowerShell examples in each chapter
5. **Architecture Review** — Study the ASCII diagrams for visual understanding

## Prerequisites

- Basic understanding of cloud computing concepts
- Familiarity with command-line interfaces
- An Azure account (free tier is sufficient for most exercises)
- Basic networking knowledge (IP addresses, DNS, HTTP)

## Key Icons Used

| Icon | Meaning |
|------|---------|
| 💡 | Best Practice / Pro Tip |
| ⚠️ | Warning / Common Pitfall |
| 🔧 | Hands-on Configuration |
| 📋 | Summary / Reference |
| ❓ | Revision Question |

---

> **Last Updated:** February 2026  
> **Author:** Study Notes for Azure DevOps  
> **License:** For educational purposes
