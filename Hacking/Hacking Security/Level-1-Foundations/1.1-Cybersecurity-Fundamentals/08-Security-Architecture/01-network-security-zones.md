# Network Security Zones

## Unit 8 - Topic 1 | Security Architecture

---

## Overview

**Network security zones** are logical or physical segments of a network, each with a defined security level and access control policies. By dividing a network into zones, organizations can control traffic flow, limit the blast radius of attacks, and enforce the principle of **least privilege** at the network level.

---

## 1. What Are Security Zones?

```
┌──────────────────────────────────────────────────────────────────┐
│                  NETWORK SECURITY ZONES                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    ┌─────────────┐                               │
│                    │  INTERNET   │  Untrusted Zone               │
│                    │  (Public)   │  Trust Level: 0               │
│                    └──────┬──────┘                               │
│                           │                                      │
│                    ┌──────┴──────┐                               │
│                    │  FIREWALL   │                               │
│                    └──────┬──────┘                               │
│                           │                                      │
│              ┌────────────┼────────────┐                        │
│              │            │            │                        │
│       ┌──────┴──────┐ ┌──┴───┐ ┌──────┴──────┐                │
│       │    DMZ      │ │ MGT  │ │  INTERNAL   │                │
│       │ (Public     │ │ Zone │ │  (Private)  │                │
│       │  Services)  │ │      │ │             │                │
│       │ Trust: 50   │ │ T:80 │ │  Trust: 100 │                │
│       └─────────────┘ └──────┘ └──────┬──────┘                │
│                                       │                        │
│                              ┌────────┼────────┐               │
│                              │        │        │               │
│                        ┌─────┴──┐ ┌───┴──┐ ┌──┴──────┐       │
│                        │ Users  │ │Server│ │Database │       │
│                        │ Zone   │ │ Zone │ │  Zone   │       │
│                        └────────┘ └──────┘ └─────────┘       │
│                                                                  │
│  Higher trust level = more sensitive data                        │
│  Traffic between zones is inspected by firewalls                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Common Security Zones

| Zone | Trust Level | Contains | Traffic Rules |
|------|:----------:|----------|---------------|
| **Internet (Untrusted)** | 0 | External world | No trust — all traffic filtered |
| **DMZ** | 50 | Web servers, mail servers, DNS | Limited access from internet, limited to internal |
| **Internal (Trusted)** | 100 | Employee workstations, internal apps | Trusted, can access most resources |
| **Server Zone** | 80 | Application/file servers | Only accepts connections from authorized zones |
| **Database Zone** | 90 | Database servers | Most restricted — only app servers can connect |
| **Management Zone** | 80 | Admin tools, SIEM, monitoring | Restricted to IT administrators |
| **Guest Zone** | 10 | Guest Wi-Fi, visitor devices | Internet access only, no internal access |

---

## 3. Zone-Based Traffic Rules

```
┌──────────────────────────────────────────────────────────────┐
│              TRAFFIC FLOW RULES                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  GENERAL PRINCIPLE:                                          │
│  • Traffic from LOW trust → HIGH trust = DENY (default)      │
│  • Traffic from HIGH trust → LOW trust = ALLOW (default)     │
│  • Same zone = ALLOW (usually)                               │
│  • Exceptions defined by explicit firewall rules             │
│                                                              │
│  EXAMPLE RULES:                                              │
│  Internet → DMZ:    ALLOW (HTTP/HTTPS only)                  │
│  Internet → Internal: DENY ❌                                │
│  DMZ → Internal:    DENY ❌ (prevents compromised DMZ        │
│                          from reaching internal)             │
│  DMZ → Database:    ALLOW (specific ports only)              │
│  Internal → DMZ:    ALLOW                                    │
│  Internal → Internet: ALLOW (via proxy)                      │
│  Guest → Internal:  DENY ❌                                  │
│  Guest → Internet:  ALLOW                                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Benefits of Network Zoning

| Benefit | Description |
|---------|-------------|
| **Blast Radius Reduction** | Compromised zone doesn't immediately expose entire network |
| **Access Control** | Enforce least-privilege at network level |
| **Regulatory Compliance** | PCI DSS requires network segmentation |
| **Monitoring** | Traffic between zones is logged and inspected |
| **Defense in Depth** | Multiple layers an attacker must cross |

---

## 5. Real-World Example: Corporate Network

```
Internet
    │
    ▼
┌─────────┐
│ Edge FW  │──── DMZ: Web Servers, Email Gateway, VPN Gateway
└────┬────┘
     │
┌────┴────┐
│ Core FW  │──── Server Zone: App Servers, File Shares
└────┬────┘
     │
     ├──── User Zone: Employee Workstations
     ├──── Dev Zone: Development Servers
     ├──── DB Zone: Database Servers (most restricted)
     └──── Mgmt Zone: SIEM, Admin Consoles
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Security Zone** | Network segment with defined trust level and access policies |
| **Key Zones** | Internet, DMZ, Internal, Server, Database, Guest |
| **Trust Principle** | Low→High = Deny; High→Low = Allow (by default) |
| **Purpose** | Limit blast radius, enforce least privilege, support compliance |
| **Enforcement** | Firewalls between zones inspect and filter all traffic |

---

## Quick Revision Questions

1. **What is a network security zone?**
2. **Why should the DMZ not have direct access to the internal network?**
3. **What is the default traffic rule between zones with different trust levels?**
4. **Name 5 common security zones and their trust levels.**
5. **How does network zoning support defense in depth?**
6. **Why does PCI DSS require network segmentation?**

---

[Next: DMZ Design →](02-dmz-design.md)

---

*Unit 8 - Topic 1 of 6 | [Back to README](../README.md)*
