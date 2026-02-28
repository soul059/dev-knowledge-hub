# Chapter 1: VLANs

## Overview

A VLAN (Virtual Local Area Network) is a logical grouping of network devices that can communicate as if they were on the same physical LAN, regardless of their physical location. VLANs are fundamental to network segmentation, security isolation, and efficient traffic management. In DevOps, understanding VLANs helps you design cloud subnets, container networks, and on-premises infrastructure.

---

## 1.1 What Is a VLAN?

```
┌──────────────────────────────────────────────────────────────┐
│              WITHOUT VLANs (One Big Network)                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────── Single Broadcast Domain ──────────────┐│
│  │                                                          ││
│  │  [Web]  [App]  [DB]  [HR]  [Finance]  [Dev]  [Printer]  ││
│  │                                                          ││
│  │  All devices see ALL broadcast traffic                   ││
│  │  No isolation = Security risk + Performance issues       ││
│  └──────────────────────────────────────────────────────────┘│
│                                                              │
├──────────────────────────────────────────────────────────────┤
│              WITH VLANs (Segmented)                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐         │
│  │  VLAN 10    │  │   VLAN 20    │  │  VLAN 30    │         │
│  │  Production │  │   Development│  │  Management │         │
│  │             │  │              │  │             │         │
│  │ [Web] [App] │  │ [Dev] [Test] │  │ [HR] [Fin]  │         │
│  │ [DB]        │  │ [Staging]    │  │ [Printer]   │         │
│  └─────────────┘  └──────────────┘  └─────────────┘         │
│                                                              │
│  Each VLAN = separate broadcast domain                       │
│  VLANs cannot communicate without a router (Layer 3)         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.2 VLAN Benefits

| Benefit | Description |
|---------|-------------|
| **Security** | Isolate sensitive systems (DB, management) from general network |
| **Performance** | Reduce broadcast domain size → less unnecessary traffic |
| **Flexibility** | Group devices logically regardless of physical location |
| **Compliance** | Separate PCI-DSS, HIPAA data onto isolated network segments |
| **Cost** | Use one physical switch for multiple logical networks |

---

## 1.3 VLAN Tagging (802.1Q)

```
┌──────────────────────────────────────────────────────────────┐
│              802.1Q VLAN TAGGING                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Normal Ethernet Frame:                                      │
│  ┌──────┬─────┬──────┬────────────┬─────┐                   │
│  │ Dest │ Src │ Type │   Payload  │ FCS │                   │
│  │ MAC  │ MAC │      │            │     │                   │
│  └──────┴─────┴──────┴────────────┴─────┘                   │
│                                                              │
│  802.1Q Tagged Frame (adds 4 bytes):                         │
│  ┌──────┬─────┬────────┬──────┬────────────┬─────┐          │
│  │ Dest │ Src │802.1Q  │ Type │   Payload  │ FCS │          │
│  │ MAC  │ MAC │VLAN Tag│      │            │     │          │
│  └──────┴─────┴───┬────┴──────┴────────────┴─────┘          │
│                    │                                         │
│              ┌─────┴──────┐                                  │
│              │ TPID: 0x8100│ (Tag Protocol Identifier)       │
│              │ PRI: 3 bits │ (Priority / QoS)                │
│              │ CFI: 1 bit  │ (Canonical Format)              │
│              │ VID: 12 bits│ (VLAN ID: 0-4095)               │
│              └─────────────┘                                 │
│                                                              │
│  Port Types:                                                 │
│  ├── Access Port: Carries ONE VLAN (untagged to end device)  │
│  └── Trunk Port:  Carries MULTIPLE VLANs (tagged)            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.4 VLAN in Cloud Context

Cloud subnets are the equivalent of VLANs in traditional networking:

```
┌──────────────────────────────────────────────────────────────┐
│         TRADITIONAL VLANs vs CLOUD SUBNETS                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  On-Premises:                  Cloud (AWS):                  │
│  ┌──────────────┐              ┌──────────────┐             │
│  │ VLAN 10      │              │ Public Subnet│             │
│  │ 10.0.10.0/24 │       ≈      │ 10.0.1.0/24 │             │
│  │ Web servers  │              │ Web servers  │             │
│  └──────────────┘              └──────────────┘             │
│  ┌──────────────┐              ┌──────────────┐             │
│  │ VLAN 20      │              │ Private Sub  │             │
│  │ 10.0.20.0/24 │       ≈      │ 10.0.2.0/24 │             │
│  │ App servers  │              │ App servers  │             │
│  └──────────────┘              └──────────────┘             │
│  ┌──────────────┐              ┌──────────────┐             │
│  │ VLAN 30      │              │ Private Sub  │             │
│  │ 10.0.30.0/24 │       ≈      │ 10.0.3.0/24 │             │
│  │ Databases    │              │ Databases    │             │
│  └──────────────┘              └──────────────┘             │
│                                                              │
│  Same concept: Network segmentation for isolation            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.5 Docker and VLANs

```bash
# Docker bridge networks act like VLANs
# Each bridge = isolated network segment

# Create custom bridge networks
docker network create --subnet=172.18.0.0/16 frontend
docker network create --subnet=172.19.0.0/16 backend

# Containers on different networks can't communicate
docker run -d --network frontend --name web nginx
docker run -d --network backend --name api node-app

# Connect container to multiple networks (like VLAN trunking)
docker network connect backend web
```

---

## 1.6 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Devices on same VLAN can't talk | Port misconfiguration | Verify VLAN assignment on switch ports |
| Cross-VLAN traffic blocked | No Layer 3 routing | Configure inter-VLAN routing on router |
| Trunk link not passing VLANs | VLAN not allowed on trunk | Add VLAN to trunk allowed list |
| Docker containers isolated | Different bridge networks | Connect containers to same network |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| VLAN | Logical network segmentation on Layer 2 |
| 802.1Q | Standard for VLAN tagging (4-byte tag in frame) |
| Access Port | Belongs to one VLAN (connects end devices) |
| Trunk Port | Carries multiple VLANs between switches |
| VLAN ID | 12-bit field (0-4095), usable: 1-4094 |
| Cloud equivalent | Subnets within a VPC |
| Docker equivalent | Bridge networks |

---

## Quick Revision Questions

1. **What problem do VLANs solve that a flat network cannot?**
2. **What is the difference between an access port and a trunk port?**
3. **How does 802.1Q tagging work? What information does the tag contain?**
4. **How do cloud subnets relate to traditional VLANs?**
5. **In Docker, how do you create isolated network segments similar to VLANs?**
6. **Can two devices on different VLANs communicate? If so, how?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← FTP/SFTP](../02-Protocols/06-ftp-sftp.md) | [README](../README.md) | [Routing Basics →](02-routing-basics.md) |
