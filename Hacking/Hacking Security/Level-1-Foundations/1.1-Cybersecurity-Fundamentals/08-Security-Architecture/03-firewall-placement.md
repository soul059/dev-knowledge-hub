# Firewall Placement

## Unit 8 - Topic 3 | Security Architecture

---

## Overview

**Firewall placement** is one of the most critical decisions in network security architecture. Where you position firewalls determines the level of traffic inspection, access control, and protection for different network segments. Proper placement creates layered defense and controls traffic flow between security zones.

---

## 1. Types of Firewalls

| Type | Layer | Description | Use Case |
|------|:-----:|-------------|----------|
| **Packet Filter** | 3-4 | Filters by IP, port, protocol | Basic perimeter defense |
| **Stateful Inspection** | 3-4 | Tracks connection state | Standard network firewall |
| **Application Layer (Proxy)** | 7 | Inspects application data (HTTP, FTP) | Deep traffic inspection |
| **Next-Generation (NGFW)** | 3-7 | Stateful + app awareness + IPS + DPI | Modern enterprise standard |
| **Web Application Firewall (WAF)** | 7 | Protects web applications (SQL injection, XSS) | In front of web servers |
| **Host-Based Firewall** | All | Runs on individual hosts | Endpoint protection |

---

## 2. Firewall Placement Strategy

```
┌──────────────────────────────────────────────────────────────────┐
│              ENTERPRISE FIREWALL PLACEMENT                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐                                                │
│  │  INTERNET   │                                                │
│  └──────┬──────┘                                                │
│         │                                                        │
│  ┌──────┴──────┐                                                │
│  │ EDGE FW     │ ◄── 1st LINE: Perimeter firewall (NGFW)       │
│  │ (Perimeter) │     Blocks known threats, DDoS, IP blocking    │
│  └──────┬──────┘                                                │
│         │                                                        │
│  ┌──────┴──────┐                                                │
│  │     DMZ     │     Public-facing services                     │
│  │  ┌───────┐  │                                                │
│  │  │  WAF  │  │ ◄── 2nd LINE: Web Application Firewall        │
│  │  └───┬───┘  │     Inspects HTTP/HTTPS traffic                │
│  │      │      │                                                │
│  │  ┌───┴───┐  │                                                │
│  │  │Web Srv│  │                                                │
│  │  └───────┘  │                                                │
│  └──────┬──────┘                                                │
│         │                                                        │
│  ┌──────┴──────┐                                                │
│  │ INTERNAL FW │ ◄── 3rd LINE: Internal segmentation firewall   │
│  │ (Core)      │     Controls east-west traffic                  │
│  └──────┬──────┘                                                │
│         │                                                        │
│    ┌────┼────┬────────┐                                         │
│    │    │    │        │                                         │
│  ┌─┴─┐┌─┴─┐┌─┴──┐ ┌──┴──┐                                    │
│  │Usr││Srv││ DB │ │Mgmt │  Internal zones                     │
│  └───┘└───┘└────┘ └─────┘                                      │
│                                                                  │
│  PLUS: Host-based firewalls on every endpoint                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Placement Positions

| Position | Type | Purpose |
|----------|------|---------|
| **Perimeter (Edge)** | NGFW | First line of defense, filters all inbound/outbound traffic |
| **DMZ Boundary** | NGFW/Stateful | Separates DMZ from internal network |
| **Between Zones** | NGFW/Stateful | Controls traffic between user, server, DB zones |
| **In Front of Web Apps** | WAF | Protects web applications from OWASP attacks |
| **Data Center Edge** | NGFW | Protects server infrastructure |
| **Cloud Perimeter** | Cloud NGFW / Security Groups | Protects cloud workloads |
| **On Every Host** | Host-based FW | Last line of defense (Windows Firewall, iptables) |

---

## 4. North-South vs East-West Traffic

```
┌──────────────────────────────────────────────────────────────┐
│          NORTH-SOUTH vs EAST-WEST                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  NORTH-SOUTH TRAFFIC:          EAST-WEST TRAFFIC:            │
│  (In/out of network)           (Within network, between      │
│                                 servers/zones)               │
│       ▲                                                      │
│       │ North                  ┌───┐    ┌───┐    ┌───┐      │
│  ┌────┼────┐                   │Srv│◄──►│Srv│◄──►│Srv│      │
│  │ Network │                   │ A │    │ B │    │ C │      │
│  └────┼────┘                   └───┘    └───┘    └───┘      │
│       │ South                    ◄── East-West ──►           │
│       ▼                                                      │
│                                                              │
│  Traditional firewalls focus on NORTH-SOUTH.                 │
│  Modern attacks move EAST-WEST (lateral movement).           │
│  → MUST have internal firewalls / microsegmentation          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Firewall Rules Best Practices

| Practice | Description |
|----------|-------------|
| **Default Deny** | Block everything, explicitly allow only needed traffic |
| **Least Privilege** | Allow only the minimum ports/protocols required |
| **Rule Order** | Most specific rules first, deny-all last |
| **Regular Review** | Audit rules quarterly — remove unused rules |
| **Logging** | Log all denied traffic and critical allowed traffic |
| **Change Management** | All rule changes go through approval process |
| **Documentation** | Document the purpose of every rule |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Perimeter FW** | First line — NGFW at network edge |
| **Internal FW** | Controls east-west (lateral) traffic |
| **WAF** | Protects web apps from application-layer attacks |
| **Host-based FW** | Last line of defense on every endpoint |
| **Default Deny** | Block everything by default, whitelist needed traffic |
| **Modern Focus** | East-west traffic inspection (lateral movement prevention) |

---

## Quick Revision Questions

1. **What are the key firewall placement points in enterprise architecture?**
2. **What is the difference between a NGFW and a traditional packet filter?**
3. **Why is east-west traffic monitoring as important as north-south?**
4. **Where should a WAF be placed and what does it protect against?**
5. **What does "default deny" mean in firewall policy?**
6. **Why is host-based firewall important even with network firewalls?**

---

[← Previous: DMZ Design](02-dmz-design.md) | [Next: Segmentation Strategies →](04-segmentation-strategies.md)

---

*Unit 8 - Topic 3 of 6 | [Back to README](../README.md)*
