# Load Balancers

## Unit 8 - Topic 4 | Network Security Devices

---

## Overview

A **load balancer** distributes incoming network traffic across multiple servers to ensure **availability**, **reliability**, and **performance**. From a security perspective, load balancers play a critical role in **DDoS mitigation**, **SSL/TLS termination**, **health monitoring**, and **hiding backend infrastructure**.

---

## 1. How Load Balancers Work

```
┌──────────────────────────────────────────────────────────────┐
│              LOAD BALANCER ARCHITECTURE                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client ──► Load Balancer (VIP: 203.0.113.10)               │
│                    │                                         │
│              ┌─────┼─────┐                                   │
│              ▼     ▼     ▼                                   │
│          ┌─────┐┌─────┐┌─────┐                              │
│          │Srv 1││Srv 2││Srv 3│                              │
│          │.101 ││.102 ││.103 │                              │
│          └─────┘└─────┘└─────┘                              │
│                                                              │
│  VIP = Virtual IP (single entry point)                       │
│  Backend servers hidden from clients                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Load Balancing Algorithms

| Algorithm | How It Works | Best For |
|-----------|-------------|----------|
| **Round Robin** | Distributes sequentially (1→2→3→1→...) | Equal-capacity servers |
| **Weighted Round Robin** | Assigns weight to servers (powerful = more requests) | Mixed-capacity servers |
| **Least Connections** | Sends to server with fewest active connections | Varying request durations |
| **IP Hash** | Same client IP → same server (sticky) | Session persistence |
| **Least Response Time** | Sends to fastest-responding server | Performance optimization |
| **Random** | Random server selection | Simple, even distribution |

---

## 3. Security Benefits

| Benefit | Description |
|---------|-------------|
| **DDoS Mitigation** | Distributes attack traffic across servers, absorbs volume |
| **SSL/TLS Termination** | Offloads encryption from backend servers |
| **Backend Hiding** | Clients only see VIP, not real server IPs |
| **Health Monitoring** | Removes compromised/failed servers from pool |
| **Rate Limiting** | Can throttle requests per client |
| **WAF Integration** | Many load balancers include WAF features |
| **Access Control** | Can restrict access by IP, geography, headers |

---

## 4. Load Balancer Types

| Type | Layer | Inspection | Use Case |
|------|:-----:|-----------|----------|
| **Layer 4 (Transport)** | TCP/UDP | IP, port only | Fast, simple routing |
| **Layer 7 (Application)** | HTTP/S | URL, headers, cookies | Content-based routing |

```
Layer 7 Routing Examples:
• /api/*     → API server pool
• /images/*  → CDN/static server pool
• /admin/*   → Admin server pool (restricted access)
```

---

## 5. Popular Load Balancer Solutions

| Solution | Type | Notes |
|----------|------|-------|
| **HAProxy** | Open source | High-performance L4/L7 LB |
| **Nginx** | Open source | Web server + reverse proxy + LB |
| **F5 BIG-IP** | Commercial | Enterprise ADC + LB |
| **AWS ALB/NLB** | Cloud | Application (L7) and Network (L4) |
| **Azure LB** | Cloud | Microsoft cloud load balancer |
| **Cloudflare** | Cloud | CDN + LB + DDoS protection |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Load Balancer** | Distributes traffic across multiple servers |
| **VIP** | Single entry point hiding backend servers |
| **L4 vs L7** | Transport (fast, simple) vs Application (content-aware) |
| **Security Role** | DDoS mitigation, SSL offload, backend hiding |
| **Health Checks** | Automatically removes failed servers |
| **Key Tools** | HAProxy, Nginx, F5, AWS ALB |

---

## Quick Revision Questions

1. **What is the primary purpose of a load balancer?**
2. **How do load balancers help with DDoS attacks?**
3. **What is the difference between L4 and L7 load balancing?**
4. **Why is hiding backend servers a security benefit?**
5. **Name 3 load balancing algorithms.**

---

[← Previous: Web Application Firewalls](03-waf.md) | [Next: Proxy Servers →](05-proxy-servers.md)

---

*Unit 8 - Topic 4 of 6 | [Back to README](../README.md)*
