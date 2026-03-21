# Network Devices (Router, Switch, Firewall)

## Unit 1 - Topic 2 | Network Fundamentals Review

---

## Overview

Network devices form the infrastructure of every organization. Understanding how **routers**, **switches**, **firewalls**, and other devices work — and their security implications — is fundamental for both defending networks and identifying attack surfaces during penetration testing.

---

## 1. Network Device Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                  NETWORK DEVICE HIERARCHY                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│            ┌─────────────┐                                      │
│            │  INTERNET   │                                      │
│            └──────┬──────┘                                      │
│                   │                                              │
│            ┌──────┴──────┐                                      │
│            │   ROUTER    │  Layer 3 — Routes between networks   │
│            └──────┬──────┘                                      │
│                   │                                              │
│            ┌──────┴──────┐                                      │
│            │  FIREWALL   │  Layer 3-7 — Filters traffic         │
│            └──────┬──────┘                                      │
│                   │                                              │
│            ┌──────┴──────┐                                      │
│            │   SWITCH    │  Layer 2 — Forwards frames           │
│            └──────┬──────┘                                      │
│                   │                                              │
│         ┌────────┼────────┐                                     │
│         │        │        │                                     │
│       ┌─┴─┐   ┌─┴─┐   ┌─┴─┐                                  │
│       │PC │   │PC │   │Srv│  Endpoints                         │
│       └───┘   └───┘   └───┘                                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Switches (Layer 2)

| Feature | Description |
|---------|-------------|
| **OSI Layer** | Layer 2 (Data Link) |
| **Function** | Forwards Ethernet frames based on MAC addresses |
| **Key Table** | CAM/MAC address table |
| **Intelligence** | Learns which MAC is on which port |

### Switch Security Concerns

| Attack | Description | Defense |
|--------|-------------|---------|
| **MAC Flooding** | Overflows CAM table → switch acts as hub | Port security (limit MACs per port) |
| **ARP Spoofing** | Poisoning ARP cache through switch | Dynamic ARP Inspection (DAI) |
| **VLAN Hopping** | Escaping VLAN isolation | Disable DTP, use access ports |
| **DHCP Starvation** | Exhausting DHCP pool | DHCP snooping |
| **Default Credentials** | Factory admin passwords | Change immediately |

### Switch Hardening Commands (Cisco)

```
! Enable port security
switchport port-security maximum 2
switchport port-security violation shutdown
switchport port-security mac-address sticky

! Disable unused ports
interface range gi0/10-24
  shutdown

! Disable DTP (prevents VLAN hopping)
switchport mode access
switchport nonegotiate
```

---

## 3. Routers (Layer 3)

| Feature | Description |
|---------|-------------|
| **OSI Layer** | Layer 3 (Network) |
| **Function** | Routes packets between different networks/subnets |
| **Key Table** | Routing table |
| **Protocols** | OSPF, BGP, EIGRP, RIP, static routes |

### Router Security Concerns

| Attack | Description | Defense |
|--------|-------------|---------|
| **Route Poisoning** | Injecting false routing info | Route authentication (MD5/SHA) |
| **Default Credentials** | Factory admin passwords | Strong passwords, SSH only |
| **Unauthorized Access** | Telnet/HTTP management | Use SSH, disable Telnet |
| **DoS** | Flooding router interfaces | ACLs, rate limiting |
| **IP Spoofing** | Forged source IPs | uRPF (Unicast Reverse Path Forwarding) |

### Router Hardening Commands (Cisco)

```
! Disable unnecessary services
no ip http server
no ip http secure-server
no cdp run
no ip source-route

! Use SSH instead of Telnet
line vty 0 4
  transport input ssh
  login local

! Access control list
access-list 100 deny ip any any log

! Enable logging
logging buffered 64000
logging trap informational
```

---

## 4. Firewalls

| Feature | Description |
|---------|-------------|
| **OSI Layer** | Layer 3-7 (depending on type) |
| **Function** | Filters traffic based on rules, policies, state |
| **Types** | Packet filter, stateful, proxy, NGFW |

### Firewall Types Comparison

| Type | Layer | Capabilities | Performance |
|------|:-----:|-------------|:-----------:|
| **Packet Filter** | 3-4 | IP/port filtering only | ⚡ Fast |
| **Stateful** | 3-4 | Tracks connection state | ⚡ Fast |
| **Application Proxy** | 7 | Deep packet inspection | 🐌 Slower |
| **NGFW** | 3-7 | All above + IPS + App ID + TLS inspection | ⚡ Fast (hardware) |

### Firewall Rule Best Practices

```
RULE ORDER MATTERS:
1. Most specific rules FIRST
2. More general rules after
3. Default DENY ALL at the bottom

EXAMPLE:
┌────┬──────────┬──────────┬──────┬────────┬────────┐
│ #  │  Source   │   Dest   │ Port │Protocol│ Action │
├────┼──────────┼──────────┼──────┼────────┼────────┤
│ 1  │ 10.0.1.5 │ WebServer│ 443  │ TCP    │ ALLOW  │
│ 2  │ 10.0.1.0 │ Any      │ 80   │ TCP    │ ALLOW  │
│ 3  │ 10.0.1.0 │ Any      │ 443  │ TCP    │ ALLOW  │
│ 4  │ Any      │ Any      │ Any  │ Any    │ DENY ❌│
└────┴──────────┴──────────┴──────┴────────┴────────┘
```

---

## 5. Other Important Devices

| Device | Layer | Function | Security Role |
|--------|:-----:|----------|---------------|
| **Hub** | 1 | Broadcasts all traffic to all ports | ❌ No security — replaced by switches |
| **Access Point** | 1-2 | Wireless network access | WPA3, WIDS/WIPS |
| **Load Balancer** | 4-7 | Distributes traffic across servers | SSL offloading, DDoS mitigation |
| **IDS/IPS** | 3-7 | Detects/prevents intrusions | Signature + anomaly-based |
| **Proxy Server** | 7 | Intermediary for requests | Content filtering, anonymization |
| **VPN Gateway** | 3 | Encrypted tunnel for remote access | IPsec, SSL/TLS VPN |

---

## Summary Table

| Device | Layer | Key Security Action |
|--------|:-----:|-------------------|
| **Switch** | 2 | Port security, DAI, DHCP snooping, disable DTP |
| **Router** | 3 | ACLs, SSH only, route authentication, uRPF |
| **Firewall** | 3-7 | Default deny, least privilege rules, log everything |
| **All Devices** | — | Change default credentials, disable unused ports/services, enable logging |

---

## Quick Revision Questions

1. **At what OSI layer do switches, routers, and firewalls operate?**
2. **What is MAC flooding and how does port security prevent it?**
3. **Why should Telnet be disabled on routers and SSH used instead?**
4. **What is the difference between a stateful firewall and an NGFW?**
5. **What is the first rule of firewall policy? (Hint: default action)**
6. **Name 3 hardening steps common to ALL network devices.**

---

[← Previous: OSI & TCP/IP Models](01-osi-tcp-ip-models.md) | [Next: IP Addressing & Subnetting →](03-ip-addressing-subnetting.md)

---

*Unit 1 - Topic 2 of 5 | [Back to README](../README.md)*
