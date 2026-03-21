# Unit 3: Azure Security — Topic 4: Network Security Groups

## Overview

**Network Security Groups (NSGs)** are Azure's primary network-level security control. NSGs filter network traffic to and from Azure resources within a Virtual Network (VNet). They contain inbound and outbound security rules that allow or deny traffic based on source, destination, port, and protocol. Proper NSG configuration is essential for network segmentation and defense in depth.

---

## 1. NSG Fundamentals

```
NSG CHARACTERISTICS:
  → Stateful (return traffic automatically allowed)
  → Applied to subnets or individual NICs
  → Contains prioritized allow/deny rules
  → Processed in priority order (lowest number first)
  → Default rules cannot be deleted (can be overridden)
  → Free service (no cost)

DEFAULT RULES (Cannot Delete):

INBOUND:
  Priority | Name                    | Source         | Dest   | Action
  65000    | AllowVnetInBound        | VirtualNetwork | VNet   | Allow
  65001    | AllowAzureLoadBalancer  | AzureLB        | Any    | Allow
  65500    | DenyAllInBound          | Any            | Any    | Deny

OUTBOUND:
  Priority | Name                    | Source | Dest            | Action
  65000    | AllowVnetOutBound       | VNet   | VirtualNetwork  | Allow
  65001    | AllowInternetOutBound   | Any    | Internet        | Allow
  65500    | DenyAllOutBound         | Any    | Any             | Deny

RULE COMPONENTS:
  → Priority: 100-4096 (lower = higher priority)
  → Name: Descriptive identifier
  → Direction: Inbound or Outbound
  → Source: IP, CIDR, Service Tag, ASG
  → Source port: * or specific
  → Destination: IP, CIDR, Service Tag, ASG
  → Destination port: Specific or range
  → Protocol: TCP, UDP, ICMP, Any
  → Action: Allow or Deny
```

---

## 2. NSG Configuration

```
EXAMPLE NSG RULES:

WEB SERVER NSG:
  Inbound:
  Priority | Name          | Source      | Port | Protocol | Action
  100      | AllowHTTPS    | Any         | 443  | TCP      | Allow
  110      | AllowHTTP     | Any         | 80   | TCP      | Allow
  120      | AllowSSH      | 10.0.0.0/8 | 22   | TCP      | Allow
  200      | DenyAll       | Any         | Any  | Any      | Deny

APP SERVER NSG:
  Inbound:
  Priority | Name          | Source        | Port | Protocol | Action
  100      | AllowFromWeb  | WebSubnet     | 8080 | TCP      | Allow
  110      | AllowSSH      | MgmtSubnet   | 22   | TCP      | Allow
  200      | DenyAll       | Any           | Any  | Any      | Deny

DB SERVER NSG:
  Inbound:
  Priority | Name          | Source        | Port | Protocol | Action
  100      | AllowFromApp  | AppSubnet     | 3306 | TCP      | Allow
  200      | DenyAll       | Any           | Any  | Any      | Deny

SERVICE TAGS:
  → Internet: All public IPs
  → VirtualNetwork: VNet address space + peered VNets
  → AzureLoadBalancer: Azure LB health probes
  → AzureCloud: All Azure datacenter IPs
  → Storage: Azure Storage service IPs
  → Sql: Azure SQL service IPs
  → AzureActiveDirectory: Azure AD IPs
  
  # Use service tags instead of IPs
  Source: AzureCloud.EastUS
  Destination: Storage.WestUS

APPLICATION SECURITY GROUPS (ASG):
  → Group VMs logically (not by subnet)
  → Use ASGs in NSG rules
  → More flexible than subnet-based rules
  → VMs can belong to multiple ASGs
  
  Example:
  ASG: WebServers (VM1, VM2, VM3)
  ASG: AppServers (VM4, VM5)
  ASG: DBServers (VM6, VM7)
  
  Rule: Allow WebServers → AppServers on port 8080
  Rule: Allow AppServers → DBServers on port 3306

CLI COMMANDS:
  # Create NSG
  az network nsg create -g myRG -n myNSG
  
  # Add rule
  az network nsg rule create \
    -g myRG --nsg-name myNSG \
    -n AllowHTTPS \
    --priority 100 \
    --direction Inbound \
    --access Allow \
    --protocol TCP \
    --destination-port-ranges 443 \
    --source-address-prefixes '*'
  
  # Associate with subnet
  az network vnet subnet update \
    -g myRG --vnet-name myVNet \
    -n mySubnet --network-security-group myNSG
  
  # List effective rules
  az network nic list-effective-nsg -g myRG -n myNIC
```

---

## 3. NSG Flow Logs

```
NSG FLOW LOGS:

WHAT THEY CAPTURE:
  → Source and destination IP
  → Source and destination port
  → Protocol (TCP/UDP)
  → Traffic direction
  → Traffic decision (allow/deny)
  → Throughput data (bytes/packets)

ENABLE:
  az network watcher flow-log create \
    --resource-group myRG \
    --nsg myNSG \
    --storage-account flowlogsstorage \
    --enabled true \
    --retention 90 \
    --log-version 2 \
    --traffic-analytics true \
    --workspace myLogAnalytics

TRAFFIC ANALYTICS:
  → Visualization of network traffic
  → Flow patterns and anomalies
  → Top talkers
  → Open ports analysis
  → Geographic traffic analysis
  → Integration with Log Analytics
  → Powered by NSG Flow Logs

USE CASES:
  → Identify unauthorized traffic flows
  → Detect port scanning attempts
  → Monitor denied traffic patterns
  → Compliance evidence
  → Network troubleshooting
  → Capacity planning
```

---

## 4. Azure Firewall

```
AZURE FIREWALL (vs NSG):

NSG:
  → Layer 3/4 filtering
  → Subnet and NIC level
  → Free
  → Basic allow/deny rules
  → No logging built-in (need flow logs)

AZURE FIREWALL:
  → Layer 3/4/7 filtering
  → Centralized, managed service
  → FQDN filtering (domain-based)
  → Threat intelligence integration
  → TLS inspection
  → Built-in HA
  → Centralized logging
  → Costs ~$1.25/hour + data processing

AZURE FIREWALL FEATURES:
  → Application rules (FQDN-based)
  → Network rules (IP/port-based)
  → NAT rules (DNAT)
  → Threat intelligence filtering
  → DNS proxy
  → Web categories filtering
  → URL filtering (Premium)
  → IDPS (Premium — Intrusion Detection)

WHEN TO USE:
  NSG Only:
  → Simple environments
  → Basic subnet isolation
  → Cost-sensitive deployments
  
  NSG + Azure Firewall:
  → Centralized traffic control
  → FQDN/domain filtering needed
  → Compliance requirements
  → Advanced threat protection
  → Logging and analytics needed
```

---

## 5. Security Assessment

```
NSG SECURITY ASSESSMENT:

CHECK FOR OPEN RULES:
  # Find NSGs allowing any source on management ports
  az network nsg list --query "[].{Name:name, RG:resourceGroup}" -o table
  
  # Check specific NSG rules
  az network nsg rule list -g myRG --nsg-name myNSG -o table
  
  # Find rules with source = Any (*) on sensitive ports
  # SSH (22), RDP (3389), database ports

COMMON MISCONFIGURATIONS:
  → SSH (22) open to Any (0.0.0.0/0 or *)
  → RDP (3389) open to Any
  → Database ports open to internet
  → All ports open in source or destination
  → Missing deny-all catch rule (relying on default)
  → Overly permissive outbound rules
  → NSG not associated with any subnet/NIC

SECURITY CHECKLIST:
  [ ] No management ports (22, 3389) open to internet
  [ ] Database ports restricted to app subnets
  [ ] Explicit deny rules where needed
  [ ] Service tags used instead of IP addresses
  [ ] ASGs used for logical grouping
  [ ] NSG flow logs enabled
  [ ] Traffic analytics enabled
  [ ] Regular rule review process
  [ ] Unused NSGs cleaned up
  [ ] Rules documented with description
```

---

## Summary Table

| Feature | NSG | Azure Firewall |
|---------|-----|---------------|
| Layer | 3/4 | 3/4/7 |
| Scope | Subnet/NIC | VNet (centralized) |
| Cost | Free | ~$900+/month |
| FQDN filtering | No | Yes |
| Threat Intel | No | Yes |
| TLS Inspection | No | Yes (Premium) |
| Best for | Basic segmentation | Advanced filtering |

---

## Revision Questions

1. How are NSG rules prioritized and evaluated?
2. What are Service Tags and Application Security Groups?
3. How do NSG flow logs support security monitoring?
4. When should Azure Firewall be used in addition to NSGs?
5. What NSG misconfigurations commonly lead to security issues?

---

*Previous: [03-azure-rbac.md](03-azure-rbac.md) | Next: [05-azure-security-center.md](05-azure-security-center.md)*

---

*[Back to README](../README.md)*
