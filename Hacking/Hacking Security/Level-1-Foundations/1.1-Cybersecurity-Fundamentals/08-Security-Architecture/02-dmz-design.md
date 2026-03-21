# DMZ Design

## Unit 8 - Topic 2 | Security Architecture

---

## Overview

A **DMZ (Demilitarized Zone)** is a perimeter network segment that sits between the untrusted internet and the trusted internal network. It hosts **public-facing services** (web servers, email gateways, DNS) while preventing direct access from the internet to internal resources. Proper DMZ design is critical for protecting the internal network from external attacks.

---

## 1. DMZ Architecture

### Single-Firewall DMZ (Three-Legged)

```
┌──────────────────────────────────────────────────────────────┐
│              SINGLE-FIREWALL DMZ (Three-Legged)               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                    ┌─────────────┐                           │
│                    │  INTERNET   │                           │
│                    └──────┬──────┘                           │
│                           │                                  │
│                    ┌──────┴──────┐                           │
│                    │  FIREWALL   │ ◄── 3 interfaces          │
│                    │ (3 legs)    │                           │
│                    └──┬─────┬───┘                           │
│                       │     │                               │
│              ┌────────┘     └────────┐                      │
│              │                       │                      │
│       ┌──────┴──────┐        ┌──────┴──────┐               │
│       │     DMZ     │        │  INTERNAL   │               │
│       │ Web Server  │        │  Network    │               │
│       │ Mail Server │        │  Users      │               │
│       │ DNS Server  │        │  Servers    │               │
│       └─────────────┘        └─────────────┘               │
│                                                              │
│  ✅ Simple, cost-effective                                   │
│  ❌ Single point of failure                                  │
│  ❌ If firewall compromised, everything exposed              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Dual-Firewall DMZ (Recommended)

```
┌──────────────────────────────────────────────────────────────┐
│              DUAL-FIREWALL DMZ (Recommended)                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                    ┌─────────────┐                           │
│                    │  INTERNET   │                           │
│                    └──────┬──────┘                           │
│                           │                                  │
│                    ┌──────┴──────┐                           │
│                    │ OUTER FW    │ ◄── External firewall     │
│                    │ (Vendor A)  │     (filters inbound)     │
│                    └──────┬──────┘                           │
│                           │                                  │
│                    ┌──────┴──────┐                           │
│                    │     DMZ     │                           │
│                    │ Web Server  │                           │
│                    │ Reverse     │                           │
│                    │ Proxy       │                           │
│                    └──────┬──────┘                           │
│                           │                                  │
│                    ┌──────┴──────┐                           │
│                    │ INNER FW    │ ◄── Internal firewall     │
│                    │ (Vendor B)  │     (protects internal)   │
│                    └──────┬──────┘                           │
│                           │                                  │
│                    ┌──────┴──────┐                           │
│                    │  INTERNAL   │                           │
│                    │  Network    │                           │
│                    └─────────────┘                           │
│                                                              │
│  ✅ Two layers of defense                                    │
│  ✅ Different vendors = different vulnerabilities             │
│  ✅ Compromised DMZ still blocked by inner firewall          │
│  ❌ More expensive and complex                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. DMZ Design Principles

| Principle | Description |
|-----------|-------------|
| **No Direct Path** | Internet → Internal MUST go through DMZ (never direct) |
| **Minimal Services** | Only public-facing services in the DMZ |
| **Hardened Servers** | DMZ servers hardened, patched, minimal OS |
| **No Sensitive Data** | Database servers NEVER in the DMZ |
| **Reverse Proxy** | Use reverse proxy in DMZ to front-end internal apps |
| **Logging** | All DMZ traffic logged to internal SIEM |
| **Different Vendors** | Use different firewall vendors for outer and inner (defense diversity) |

---

## 3. Services Commonly Placed in DMZ

| Service | Why in DMZ |
|---------|-----------|
| **Web Server** | Public website needs internet access |
| **Reverse Proxy / WAF** | Inspects HTTP traffic before forwarding to internal servers |
| **Email Gateway** | Filters incoming email before internal mail server |
| **DNS Server** | Resolves external DNS queries |
| **VPN Gateway** | Terminates remote access VPN connections |
| **FTP Server** | Public file sharing (if required) |

### Services That Should NOT Be in DMZ

| Service | Why NOT in DMZ |
|---------|---------------|
| **Database Server** | Contains sensitive data — internal zone only |
| **Domain Controller** | Active Directory — internal/management zone |
| **Internal File Server** | Employee data — internal zone |
| **SIEM / Monitoring** | Security data — management zone |

---

## 4. Traffic Flow Rules

```
┌──────────────────────────────────────────────────────────────┐
│              DMZ TRAFFIC RULES                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ALLOWED:                                                    │
│  ✅ Internet → DMZ (HTTP/HTTPS ports 80/443 only)            │
│  ✅ Internal → DMZ (management, updates)                     │
│  ✅ DMZ → Internal (specific ports to app/DB servers only)   │
│  ✅ Internal → Internet (via proxy)                          │
│                                                              │
│  DENIED:                                                     │
│  ❌ Internet → Internal (NEVER direct)                       │
│  ❌ DMZ → Internal (broad access — only specific rules)      │
│  ❌ DMZ → Management Zone                                    │
│                                                              │
│  CRITICAL RULE:                                              │
│  A compromised DMZ server should NOT be able to              │
│  initiate arbitrary connections to the internal network.     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **DMZ** | Perimeter zone between internet and internal network |
| **Purpose** | Host public services without exposing internal network |
| **Best Design** | Dual-firewall with different vendors |
| **Key Rule** | No direct internet → internal path |
| **DMZ Services** | Web server, mail gateway, DNS, VPN gateway |
| **Not in DMZ** | Databases, Domain Controllers, internal file servers |

---

## Quick Revision Questions

1. **What is a DMZ and what is its purpose?**
2. **Why is a dual-firewall DMZ design preferred over single-firewall?**
3. **Why should different firewall vendors be used for outer and inner firewalls?**
4. **Name 4 services that belong in the DMZ and 3 that do NOT.**
5. **What traffic rule prevents a compromised DMZ from attacking the internal network?**
6. **What role does a reverse proxy play in DMZ architecture?**

---

[← Previous: Network Security Zones](01-network-security-zones.md) | [Next: Firewall Placement →](03-firewall-placement.md)

---

*Unit 8 - Topic 2 of 6 | [Back to README](../README.md)*
