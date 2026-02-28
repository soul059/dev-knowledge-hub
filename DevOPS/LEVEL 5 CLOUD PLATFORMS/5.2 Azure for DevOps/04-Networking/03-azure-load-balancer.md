# Chapter 3: Azure Load Balancer

## Overview
**Azure Load Balancer** is a Layer 4 (TCP/UDP) load balancer that distributes incoming traffic across healthy backend resources. It provides high availability and network performance for applications.

---

## 3.1 Load Balancer Architecture

```
┌────────────── AZURE LOAD BALANCER ──────────────┐
│                                                   │
│   INTERNET / Internal                             │
│       │                                           │
│       ▼                                           │
│   ┌──────────────────────────────────┐            │
│   │  Frontend IP Configuration       │            │
│   │  • Public IP: 52.168.1.100       │            │
│   │    (or Internal: 10.0.1.100)     │            │
│   └──────────────┬───────────────────┘            │
│                  │                                 │
│   ┌──────────────▼───────────────────┐            │
│   │  Load Balancing Rules             │            │
│   │  • Port 80  → Backend Pool       │            │
│   │  • Port 443 → Backend Pool       │            │
│   └──────────────┬───────────────────┘            │
│                  │                                 │
│   ┌──────────────▼───────────────────┐            │
│   │  Health Probes                    │            │
│   │  • HTTP /health every 15s         │            │
│   │  • TCP port 443 every 15s         │            │
│   └──────────────┬───────────────────┘            │
│                  │                                 │
│   ┌──────────────▼───────────────────┐            │
│   │  Backend Pool                     │            │
│   │  ┌──────┐  ┌──────┐  ┌──────┐   │            │
│   │  │ VM-1 │  │ VM-2 │  │ VM-3 │   │            │
│   │  │  ✅  │  │  ✅  │  │  ❌  │   │            │
│   │  └──────┘  └──────┘  └──────┘   │            │
│   │  (unhealthy VMs removed)         │            │
│   └──────────────────────────────────┘            │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 3.2 Public vs Internal Load Balancer

| Feature | Public LB | Internal LB |
|---------|-----------|-------------|
| **Frontend IP** | Public IP address | Private IP from VNet |
| **Access** | Internet-facing | Internal traffic only |
| **Use Case** | Web servers, public APIs | Database tier, internal APIs |
| **Backend** | VMs, VMSS, public IPs | VMs, VMSS in same VNet |

```
┌────── MULTI-TIER WITH BOTH LB TYPES ──────┐
│                                             │
│   Internet                                  │
│      │                                      │
│      ▼                                      │
│   ┌──────────────┐                          │
│   │  PUBLIC LB   │  ← Internet-facing       │
│   └──────┬───────┘                          │
│          │                                   │
│   ┌──────▼──────────────────┐               │
│   │  WEB TIER (Subnet A)    │               │
│   │  VM-1  VM-2  VM-3       │               │
│   └──────┬──────────────────┘               │
│          │                                   │
│   ┌──────▼───────┐                          │
│   │ INTERNAL LB  │  ← Private only          │
│   └──────┬───────┘                          │
│          │                                   │
│   ┌──────▼──────────────────┐               │
│   │  DB TIER (Subnet B)     │               │
│   │  SQL-1  SQL-2            │               │
│   └─────────────────────────┘               │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 3.3 SKU Comparison

| Feature | Basic | Standard |
|---------|-------|----------|
| **Backend pool size** | Up to 300 | Up to 1,000 |
| **Health probes** | HTTP, TCP | HTTP, HTTPS, TCP |
| **Availability Zones** | ❌ Not supported | ✅ Zone-redundant |
| **SLA** | Not guaranteed | 99.99% |
| **Security** | Open by default | **Closed by default** (needs NSG) |
| **Outbound rules** | ❌ | ✅ |
| **Multiple frontends** | ❌ | ✅ |
| **Cost** | Free | Charged |

> ⚠️ **Note:** Basic SKU is being retired. Always use **Standard** for production.

---

## 3.4 Health Probes

```bash
# Health probe options:
# ─────────────────────
# HTTP probe:  GET /health → expects 200 OK
# HTTPS probe: GET /health → expects 200 OK (Standard SKU only)
# TCP probe:   Successful TCP connection to port
#
# Parameters:
# • Interval: How often to probe (default 15 seconds)
# • Unhealthy threshold: Consecutive failures (default 2)
```

---

## 3.5 Distribution Modes

| Mode | Hash Based On | Use Case |
|------|---------------|----------|
| **Default (5-tuple)** | Source IP, Source Port, Dest IP, Dest Port, Protocol | General load balancing |
| **Source IP (2-tuple)** | Source IP, Dest IP | Sticky sessions by IP |
| **Source IP + Protocol (3-tuple)** | Source IP, Dest IP, Protocol | Session affinity |

---

## 3.6 Creating a Load Balancer

```bash
# Create public IP
az network public-ip create \
  --resource-group myRG \
  --name myPublicIP \
  --sku Standard \
  --zone 1 2 3

# Create Load Balancer
az network lb create \
  --resource-group myRG \
  --name myLoadBalancer \
  --sku Standard \
  --public-ip-address myPublicIP \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool

# Create health probe
az network lb probe create \
  --resource-group myRG \
  --lb-name myLoadBalancer \
  --name myHealthProbe \
  --protocol Http \
  --port 80 \
  --path /health \
  --interval 15 \
  --threshold 2

# Create load balancing rule
az network lb rule create \
  --resource-group myRG \
  --lb-name myLoadBalancer \
  --name myHTTPRule \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool \
  --probe-name myHealthProbe \
  --idle-timeout 4 \
  --enable-tcp-reset true

# Add VMs to backend pool
az network nic ip-config address-pool add \
  --resource-group myRG \
  --nic-name vm1-nic \
  --ip-config-name ipconfig1 \
  --lb-name myLoadBalancer \
  --address-pool myBackEndPool
```

---

## 3.7 Outbound Rules (Standard SKU)

```bash
# Create outbound rule for SNAT
az network lb outbound-rule create \
  --resource-group myRG \
  --lb-name myLoadBalancer \
  --name myOutboundRule \
  --frontend-ip-configs myFrontEnd \
  --address-pool myBackEndPool \
  --protocol All \
  --idle-timeout 4 \
  --outbound-ports 10000
```

---

## 3.8 Real-World Scenario: DevOps Pipeline with LB

```
┌────── CI/CD DEPLOYMENT WITH LOAD BALANCER ──────┐
│                                                   │
│   1. Pipeline builds new VM image                 │
│   2. New VMs added to backend pool                │
│   3. Health probe verifies new VMs                │
│   4. Traffic starts flowing to new VMs            │
│   5. Old VMs drained and removed                  │
│                                                   │
│   Pipeline YAML:                                  │
│   ─────────────────────────────────               │
│   - task: AzureCLI@2                              │
│     inputs:                                       │
│       azureSubscription: 'my-conn'                │
│       scriptType: 'bash'                          │
│       scriptLocation: 'inlineScript'              │
│       inlineScript: |                             │
│         az network nic ip-config \                │
│           address-pool add \                      │
│           --nic-name $(newVmNic) \                │
│           --ip-config-name ipconfig1 \            │
│           --lb-name myLB \                        │
│           --address-pool myPool                   │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 3.9 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| VMs not receiving traffic | Health probe failing | Check probe port/path, verify app is running |
| Uneven distribution | Session persistence enabled | Change distribution mode or disable affinity |
| SNAT port exhaustion | Too many outbound connections | Use outbound rules, add more frontend IPs |
| Standard LB no connectivity | NSG missing | NSGs required for Standard SKU — add allow rules |
| Backend pool empty | NICs not associated | Associate NIC IP configs with backend pool |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Layer** | Layer 4 (TCP/UDP) — no HTTP header inspection |
| **Types** | Public (internet-facing) and Internal (private) |
| **SKUs** | Basic (being retired) and Standard (recommended) |
| **Health Probes** | HTTP, HTTPS, TCP — removes unhealthy backends |
| **Distribution** | 5-tuple (default), Source IP, Source IP + Protocol |
| **Standard SKU** | Zone-redundant, closed by default, 99.99% SLA |

---

## Quick Revision Questions

1. **What layer does Azure Load Balancer operate at?**
   > Layer 4 (Transport) — it works with TCP and UDP packets, not HTTP headers.

2. **What is the difference between Public and Internal Load Balancer?**
   > Public LB has a public IP for internet traffic; Internal LB uses a private IP for VNet-internal traffic.

3. **Why is Standard SKU recommended over Basic?**
   > Standard supports Availability Zones, has 99.99% SLA, supports HTTPS probes, outbound rules, and more backends.

4. **What happens when a health probe fails?**
   > The backend VM is marked unhealthy and removed from the rotation until the probe succeeds again.

5. **Why do you need NSGs with Standard Load Balancer?**
   > Standard SKU is secure by default (denies all) — you must explicitly allow traffic with NSG rules.

6. **What is SNAT port exhaustion and how do you fix it?**
   > When backend VMs run out of outbound ports. Fix by configuring outbound rules or adding more frontend IPs.

---

[⬅ Previous: Subnets and NSGs](02-subnets-and-nsgs.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Application Gateway ➡](04-application-gateway.md)
