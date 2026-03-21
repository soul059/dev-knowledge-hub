# Unit 3: Azure Security — Topic 1: Azure Security Architecture

## Overview

**Microsoft Azure** provides a comprehensive security architecture built on defense-in-depth principles. Azure's security spans physical data centers, network infrastructure, identity services, and application-level controls. Understanding Azure's security architecture is essential for securing workloads, conducting assessments, and leveraging Azure's native security capabilities effectively.

---

## 1. Azure Infrastructure

```
AZURE GLOBAL INFRASTRUCTURE:

  ┌──────────────────────────────────────────────┐
  │              AZURE GLOBAL                    │
  │                                              │
  │  ┌────────────────────────────────────────┐  │
  │  │    GEOGRAPHY (e.g., United States)     │  │
  │  │                                        │  │
  │  │  ┌────────────────┐ ┌────────────────┐ │  │
  │  │  │ REGION         │ │ REGION         │ │  │
  │  │  │ (East US)      │ │ (West US)      │ │  │
  │  │  │                │ │                │ │  │
  │  │  │ ┌────────────┐ │ │ ┌────────────┐ │ │  │
  │  │  │ │ AZ 1       │ │ │ │ AZ 1       │ │ │  │
  │  │  │ │ AZ 2       │ │ │ │ AZ 2       │ │ │  │
  │  │  │ │ AZ 3       │ │ │ │ AZ 3       │ │ │  │
  │  │  │ └────────────┘ │ │ └────────────┘ │ │  │
  │  │  └────────────────┘ └────────────────┘ │  │
  │  └────────────────────────────────────────┘  │
  │                                              │
  │  60+ Regions | 200+ Edge Locations           │
  └──────────────────────────────────────────────┘

MANAGEMENT HIERARCHY:
  Management Groups (top-level governance)
  └── Subscriptions (billing and access boundary)
      └── Resource Groups (logical containers)
          └── Resources (VMs, Storage, Databases)

  ┌──────────────────────────────────┐
  │     Root Management Group        │
  ├─────────────┬────────────────────┤
  │ MG: Prod    │ MG: Non-Prod      │
  ├─────────────┼────────────────────┤
  │ Sub: Prod1  │ Sub: Dev1         │
  │ Sub: Prod2  │ Sub: Test1        │
  ├─────────────┼────────────────────┤
  │ RG: Web     │ RG: DevApp        │
  │ RG: Data    │ RG: DevDB         │
  │ RG: Network │ RG: DevNet        │
  └─────────────┴────────────────────┘

AZURE POLICIES:
  → Applied at management group, subscription, or RG level
  → Enforce standards and compliance
  → Audit or deny non-compliant resources
  → Built-in and custom policies
  → Policy initiatives (groups of policies)
  
  Example:
  → Deny VMs without encryption
  → Require specific tags on resources
  → Restrict allowed regions
  → Enforce network security settings
```

---

## 2. Defense in Depth

```
AZURE SECURITY LAYERS:

  ┌─────────────────────────────────────────┐
  │ PHYSICAL SECURITY                       │
  │ Data center access, surveillance, guards│
  ├─────────────────────────────────────────┤
  │ IDENTITY & ACCESS                       │
  │ Azure AD, MFA, Conditional Access       │
  ├─────────────────────────────────────────┤
  │ PERIMETER                               │
  │ DDoS Protection, Azure Firewall, WAF    │
  ├─────────────────────────────────────────┤
  │ NETWORK                                 │
  │ NSG, VNet, Private Link, ExpressRoute   │
  ├─────────────────────────────────────────┤
  │ COMPUTE                                 │
  │ VM hardening, patching, endpoint protect│
  ├─────────────────────────────────────────┤
  │ APPLICATION                             │
  │ Secure code, SAST/DAST, API management  │
  ├─────────────────────────────────────────┤
  │ DATA                                    │
  │ Encryption, classification, rights mgmt │
  └─────────────────────────────────────────┘

KEY SERVICES PER LAYER:
  Identity:    Azure AD, PIM, Conditional Access
  Perimeter:   DDoS Protection, Firewall, WAF
  Network:     NSG, ASG, VNet, Private Link
  Compute:     Defender for Servers, Update Mgmt
  Application: App Service security, Key Vault
  Data:        Storage encryption, SQL TDE, AIP
  Monitoring:  Sentinel, Defender for Cloud, Monitor
```

---

## 3. Azure Security Services

```
CORE SECURITY SERVICES:

MICROSOFT DEFENDER FOR CLOUD:
  → Unified security management
  → Security posture assessment (Secure Score)
  → Threat protection for Azure resources
  → Compliance dashboard
  → Recommendations engine
  → Integration with Sentinel

MICROSOFT SENTINEL:
  → Cloud-native SIEM + SOAR
  → Log collection and analysis
  → Threat detection (analytics rules)
  → Automated response (playbooks)
  → Threat hunting
  → Workbooks (dashboards)

AZURE KEY VAULT:
  → Secret management
  → Key management
  → Certificate management
  → HSM-backed keys
  → Access policies + RBAC
  → Audit logging

AZURE POLICY:
  → Governance and compliance
  → Enforce organizational standards
  → Built-in policy library
  → Custom policies (JSON)
  → Initiative assignments
  → Compliance reporting

AZURE MONITOR:
  → Metrics and logs
  → Log Analytics workspace
  → Alerts and action groups
  → Application Insights
  → Diagnostic settings
  → Activity Log

MICROSOFT ENTRA (Azure AD):
  → Identity management
  → Authentication (MFA, passwordless)
  → Conditional Access
  → Privileged Identity Management
  → Identity Protection
  → Application registration
```

---

## 4. Azure CLI Security Commands

```
SECURITY ASSESSMENT:

# Login
az login

# List subscriptions
az account list --output table

# Set subscription
az account set --subscription "Prod"

# List resources
az resource list --output table

# Check VM encryption
az vm encryption show --resource-group myRG --name myVM

# List NSG rules
az network nsg rule list --resource-group myRG --nsg-name myNSG

# List storage accounts
az storage account list --output table

# Check storage encryption
az storage account show --name mystorageaccount \
  --query "encryption"

# List Key Vault secrets
az keyvault secret list --vault-name myVault

# Azure Policy assignments
az policy assignment list --output table

# Defender for Cloud secure score
az security secure-score list --output table

# Activity log (last 24h)
az monitor activity-log list \
  --start-time $(date -d '24 hours ago' --iso-8601) \
  --output table

SECURITY ASSESSMENT TOOLS:
  → ScoutSuite: Multi-cloud security auditing
  → Azucar: Azure security assessment
  → MicroBurst: Azure pentest toolkit
  → PowerZure: Azure offensive tooling
  → ROADtools: Azure AD assessment
  → AADInternals: Azure AD exploitation
```

---

## 5. Best Practices

```
AZURE SECURITY BEST PRACTICES:

IDENTITY:
  [ ] Enable MFA for all users
  [ ] Use Conditional Access policies
  [ ] Implement PIM for privileged roles
  [ ] Regular access reviews
  [ ] Disable legacy authentication
  [ ] Use managed identities for Azure resources

NETWORK:
  [ ] NSGs on all subnets and NICs
  [ ] Azure Firewall for centralized filtering
  [ ] Private Link for PaaS services
  [ ] DDoS Protection Standard for public resources
  [ ] Network Watcher for monitoring
  [ ] ExpressRoute or VPN for hybrid connectivity

DATA:
  [ ] Enable encryption at rest (default)
  [ ] Enforce HTTPS/TLS
  [ ] Use Key Vault for secrets
  [ ] Enable soft delete and purge protection
  [ ] Classify data with Information Protection
  [ ] Enable diagnostic logging

MONITORING:
  [ ] Enable Defender for Cloud (all plans)
  [ ] Deploy Sentinel for SIEM
  [ ] Activity Log forwarded to Log Analytics
  [ ] Diagnostic settings on all resources
  [ ] Alert rules for critical events
  [ ] Regular security posture reviews
```

---

## Summary Table

| Service | Category | Purpose |
|---------|----------|---------|
| Defender for Cloud | Posture Management | Security scoring and recommendations |
| Sentinel | SIEM/SOAR | Threat detection and response |
| Azure AD / Entra | Identity | Authentication and authorization |
| Key Vault | Secrets | Key and secret management |
| Azure Policy | Governance | Compliance enforcement |
| Azure Monitor | Monitoring | Metrics, logs, alerts |

---

## Revision Questions

1. How is Azure's management hierarchy organized for security?
2. What are the layers of Azure's defense-in-depth model?
3. What is the role of Microsoft Defender for Cloud?
4. How do Azure Policies enforce security standards?
5. What CLI commands are useful for Azure security assessment?

---

*Previous: None (First topic in this unit) | Next: [02-azure-ad.md](02-azure-ad.md)*

---

*[Back to README](../README.md)*
