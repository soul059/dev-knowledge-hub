# Chapter 4: Application Gateway

## Overview
**Azure Application Gateway** is a Layer 7 (HTTP/HTTPS) load balancer that provides URL-based routing, SSL termination, Web Application Firewall (WAF), and session affinity. It understands HTTP headers, unlike the Layer 4 Load Balancer.

---

## 4.1 Application Gateway Architecture

```
┌──────────── APPLICATION GATEWAY ────────────────┐
│                                                   │
│   Client Request                                  │
│   GET /api/users HTTP/1.1                         │
│       │                                           │
│       ▼                                           │
│   ┌───────────────────────────────────┐           │
│   │  Frontend (Listener)              │           │
│   │  • Public IP / Private IP         │           │
│   │  • Port: 443 (HTTPS)              │           │
│   │  • SSL Certificate loaded         │           │
│   └───────────────┬───────────────────┘           │
│                   │                                │
│   ┌───────────────▼───────────────────┐           │
│   │  WAF (Optional)                   │           │
│   │  • OWASP Rule Sets               │           │
│   │  • SQL Injection protection       │           │
│   │  • XSS protection                │           │
│   └───────────────┬───────────────────┘           │
│                   │                                │
│   ┌───────────────▼───────────────────┐           │
│   │  Routing Rules                    │           │
│   │  /api/*    → API Backend Pool     │           │
│   │  /images/* → Static Backend Pool  │           │
│   │  /*        → Web Backend Pool     │           │
│   └───┬───────────┬──────────┬────────┘           │
│       │           │          │                     │
│       ▼           ▼          ▼                     │
│   ┌───────┐ ┌─────────┐ ┌────────┐               │
│   │  API  │ │ Static  │ │  Web   │               │
│   │ Pool  │ │  Pool   │ │  Pool  │               │
│   │VM1 VM2│ │Blob Stor│ │VM3 VM4 │               │
│   └───────┘ └─────────┘ └────────┘               │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 4.2 Load Balancer vs Application Gateway

| Feature | Load Balancer | Application Gateway |
|---------|---------------|---------------------|
| **OSI Layer** | Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
| **Routing** | Port-based only | URL path, host header, query string |
| **SSL Termination** | ❌ | ✅ |
| **WAF** | ❌ | ✅ |
| **Websockets** | Pass-through | ✅ Native support |
| **Session Affinity** | Source IP | Cookie-based |
| **Autoscaling** | ❌ | ✅ (v2 SKU) |
| **URL Rewrite** | ❌ | ✅ |
| **Health Probes** | TCP/HTTP | HTTP/HTTPS with custom paths |

---

## 4.3 SKU Comparison

| Feature | Standard v1 | Standard v2 | WAF v1 | WAF v2 |
|---------|-------------|-------------|--------|--------|
| Autoscaling | ❌ | ✅ | ❌ | ✅ |
| Zone redundancy | ❌ | ✅ | ❌ | ✅ |
| Static VIP | ❌ | ✅ | ❌ | ✅ |
| WAF | ❌ | ❌ | ✅ | ✅ |
| Performance | Lower | Higher | Lower | Higher |

> ⚠️ **Always use v2 SKU** for production. v1 is being deprecated.

---

## 4.4 Key Components

```
┌────── APPLICATION GATEWAY COMPONENTS ──────┐
│                                             │
│  1. Frontend IP Config                      │
│     └─ Public IP and/or Private IP          │
│                                             │
│  2. Listeners                               │
│     ├─ Basic: Single site                   │
│     └─ Multi-site: Multiple domains         │
│                                             │
│  3. Routing Rules                           │
│     ├─ Basic: All traffic → one pool        │
│     └─ Path-based: URL path → specific pool │
│                                             │
│  4. Backend Pools                           │
│     ├─ VMs / VMSS                           │
│     ├─ App Services                         │
│     ├─ IP addresses / FQDNs                 │
│     └─ Even other clouds!                   │
│                                             │
│  5. HTTP Settings                           │
│     ├─ Port, Protocol, Cookie affinity      │
│     ├─ Connection draining                  │
│     └─ Custom probe settings                │
│                                             │
│  6. Health Probes                            │
│     ├─ Custom path (e.g., /health)          │
│     └─ HTTP status code matching            │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 4.5 SSL Termination and End-to-End SSL

```
┌────── SSL TERMINATION OPTIONS ──────────────┐
│                                              │
│  OPTION 1: SSL OFFLOADING                    │
│  Client ──HTTPS──▶ AppGW ──HTTP──▶ Backend  │
│  (SSL terminated at gateway, plain HTTP      │
│   to backend — less CPU on backends)         │
│                                              │
│  OPTION 2: END-TO-END SSL                    │
│  Client ──HTTPS──▶ AppGW ──HTTPS──▶ Backend │
│  (Re-encrypted to backend — full encryption  │
│   but more CPU usage)                        │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 4.6 Creating an Application Gateway

```bash
# Create a subnet for Application Gateway (dedicated)
az network vnet subnet create \
  --resource-group myRG \
  --vnet-name myVNet \
  --name AppGwSubnet \
  --address-prefix 10.0.2.0/24

# Create public IP
az network public-ip create \
  --resource-group myRG \
  --name appGwPublicIP \
  --sku Standard \
  --allocation-method Static

# Create Application Gateway
az network application-gateway create \
  --resource-group myRG \
  --name myAppGateway \
  --location eastus \
  --sku Standard_v2 \
  --capacity 2 \
  --vnet-name myVNet \
  --subnet AppGwSubnet \
  --public-ip-address appGwPublicIP \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --frontend-port 80 \
  --servers 10.0.1.4 10.0.1.5

# Add path-based routing
az network application-gateway url-path-map create \
  --resource-group myRG \
  --gateway-name myAppGateway \
  --name urlPathMap \
  --paths "/api/*" \
  --address-pool apiBackendPool \
  --http-settings myHTTPSettings \
  --default-address-pool webBackendPool \
  --default-http-settings myHTTPSettings

# Add a health probe
az network application-gateway probe create \
  --resource-group myRG \
  --gateway-name myAppGateway \
  --name myHealthProbe \
  --protocol Http \
  --host-name-from-http-settings true \
  --path /health \
  --interval 30 \
  --timeout 30 \
  --threshold 3
```

---

## 4.7 Web Application Firewall (WAF)

```
┌────── WAF PROTECTION ──────────────────────┐
│                                             │
│  WAF protects against:                      │
│  • SQL Injection attacks                    │
│  • Cross-Site Scripting (XSS)               │
│  • Command injection                        │
│  • HTTP request smuggling                   │
│  • Remote file inclusion                    │
│  • Protocol violations                      │
│                                             │
│  MODES:                                     │
│  ┌──────────────┐  ┌──────────────┐        │
│  │  DETECTION   │  │  PREVENTION  │        │
│  │  Logs only   │  │  Blocks +    │        │
│  │  No blocking │  │  Logs        │        │
│  └──────────────┘  └──────────────┘        │
│                                             │
│  Rule Sets: OWASP 3.2, 3.1, 3.0            │
│  Custom Rules: Rate limiting, geo-filtering │
│                                             │
└─────────────────────────────────────────────┘
```

```bash
# Create WAF policy
az network application-gateway waf-policy create \
  --resource-group myRG \
  --name myWAFPolicy

# Enable WAF with OWASP 3.2
az network application-gateway waf-config set \
  --resource-group myRG \
  --gateway-name myAppGateway \
  --enabled true \
  --firewall-mode Prevention \
  --rule-set-version 3.2
```

---

## 4.8 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| 502 Bad Gateway | Backend unhealthy or unreachable | Check backend health, NSG rules, probe config |
| SSL errors | Certificate mismatch/expired | Upload correct cert; check cert chain |
| Slow performance | Wrong SKU or undersized | Use v2 with autoscaling |
| WAF blocking legit traffic | Rule too strict | Switch to Detection mode, review logs, add exclusions |
| Subnet too small | Not enough IPs for instances | Use /24 minimum for AppGW subnet |

> 💡 **Tip:** Application Gateway requires its own **dedicated subnet**. Don't place other resources in it.

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Layer** | Layer 7 — understands HTTP/HTTPS |
| **Routing** | URL path-based, multi-site, query string |
| **SSL** | Termination or end-to-end encryption |
| **WAF** | OWASP protection (Detection or Prevention mode) |
| **SKU** | Use v2 for autoscaling and zone-redundancy |
| **Subnet** | Requires a dedicated subnet (/24 recommended) |
| **Backend** | VMs, VMSS, App Service, IPs, FQDNs |

---

## Quick Revision Questions

1. **What layer does Application Gateway operate at?**
   > Layer 7 (Application layer) — it inspects HTTP/HTTPS headers for routing decisions.

2. **What is URL path-based routing?**
   > Routing requests to different backend pools based on the URL path (e.g., /api/* to API servers, /images/* to storage).

3. **What is SSL termination?**
   > Decrypting HTTPS at the Application Gateway and forwarding plain HTTP to backends, offloading SSL processing.

4. **What is the purpose of WAF?**
   > Web Application Firewall protects against common web attacks like SQL injection and XSS using OWASP rule sets.

5. **Why does Application Gateway need a dedicated subnet?**
   > It deploys multiple instances for high availability — the subnet must have enough IPs for scaling.

---

[⬅ Previous: Azure Load Balancer](03-azure-load-balancer.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure DNS ➡](05-azure-dns.md)
