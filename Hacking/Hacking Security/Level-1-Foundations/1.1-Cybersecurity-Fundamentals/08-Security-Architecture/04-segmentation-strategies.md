# Segmentation Strategies

## Unit 8 - Topic 4 | Security Architecture

---

## Overview

**Network segmentation** divides a network into smaller, isolated segments to limit the scope of security breaches, control access, and improve performance. Modern approaches range from traditional VLANs to **microsegmentation** — a zero-trust approach that controls traffic between individual workloads.

---

## 1. Why Segmentation Matters

```
┌──────────────────────────────────────────────────────────────┐
│              WITHOUT SEGMENTATION                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FLAT NETWORK:                                               │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐               │
│  │ PC│─│ PC│─│Srv│─│ DB│─│POS│─│IoT│─│Dev│  ALL CONNECTED │
│  └───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘               │
│                                                              │
│  Attacker compromises ONE device → can reach EVERYTHING      │
│  Lateral movement is trivial → entire network exposed        │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│              WITH SEGMENTATION                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌──────────┐   │
│  │ Users   │   │ Servers │   │ Database│   │   IoT    │   │
│  │ ┌───┐   │   │ ┌───┐   │   │ ┌───┐   │   │ ┌───┐   │   │
│  │ │PC │   │   │ │Srv│   │   │ │DB │   │   │ │IoT│   │   │
│  │ └───┘   │   │ └───┘   │   │ └───┘   │   │ └───┘   │   │
│  └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘   │
│       │             │             │             │         │
│       └──────┬──────┴──────┬──────┘             │         │
│              │  FIREWALL   │                    │         │
│              └─────────────┴────────────────────┘         │
│                                                              │
│  Attacker compromises ONE segment → contained               │
│  Must breach firewall to reach other segments               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Segmentation Methods

| Method | How It Works | Granularity | Best For |
|--------|-------------|:-----------:|---------|
| **Physical Separation** | Separate physical networks/switches | Coarse | Air-gapped/critical systems |
| **VLANs** | Logical separation on same switch | Medium | Traditional enterprise |
| **Firewall Zones** | Firewall rules between segments | Medium | Zone-based security |
| **SDN (Software-Defined)** | Software-controlled network policies | Fine | Cloud/virtualized environments |
| **Microsegmentation** | Per-workload policies | Very Fine | Zero Trust architecture |

---

## 3. VLAN-Based Segmentation

```
┌──────────────────────────────────────────────────────────────┐
│              VLAN SEGMENTATION                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Single Physical Switch, Multiple Logical Networks:          │
│                                                              │
│  ┌──────────────────────────────────┐                        │
│  │          MANAGED SWITCH          │                        │
│  │                                  │                        │
│  │  VLAN 10     VLAN 20    VLAN 30  │                        │
│  │  (Users)    (Servers)   (Guest)  │                        │
│  │  ┌──┐┌──┐  ┌──┐┌──┐   ┌──┐     │                        │
│  │  │PC││PC│  │Sv││Sv│   │Gt│     │                        │
│  │  └──┘└──┘  └──┘└──┘   └──┘     │                        │
│  └──────────────┬───────────────────┘                        │
│                 │                                            │
│          ┌──────┴──────┐                                     │
│          │   ROUTER/   │  Inter-VLAN routing                 │
│          │  FIREWALL   │  with access control                │
│          └─────────────┘                                     │
│                                                              │
│  VLANs cannot communicate without a router/firewall          │
│  Firewall enforces rules between VLANs                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Microsegmentation

```
┌──────────────────────────────────────────────────────────────┐
│              MICROSEGMENTATION                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Traditional:  Segment by NETWORK (VLAN, subnet)             │
│  Micro:        Segment by WORKLOAD (individual VM/container) │
│                                                              │
│  ┌─────────────────────────────────────────┐                 │
│  │  Data Center / Cloud                    │                 │
│  │                                         │                 │
│  │  ┌─────┐    ┌─────┐    ┌─────┐         │                 │
│  │  │Web  │    │App  │    │ DB  │         │                 │
│  │  │VM   │───►│VM   │───►│VM   │         │                 │
│  │  └──┬──┘    └──┬──┘    └──┬──┘         │                 │
│  │     │          │          │             │                 │
│  │  ┌──┴──────────┴──────────┴──┐         │                 │
│  │  │  POLICY ENGINE            │         │                 │
│  │  │  Per-workload rules:      │         │                 │
│  │  │  Web→App: port 8080 only  │         │                 │
│  │  │  App→DB: port 3306 only   │         │                 │
│  │  │  Web→DB: DENY ❌          │         │                 │
│  │  │  DB→Web: DENY ❌          │         │                 │
│  │  └───────────────────────────┘         │                 │
│  └─────────────────────────────────────────┘                 │
│                                                              │
│  Each workload has its OWN security policy.                  │
│  Lateral movement between workloads is blocked by default.   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Segmentation Best Practices

| Practice | Description |
|----------|-------------|
| **Identify Assets** | Map all assets, data flows, and criticality |
| **Define Zones** | Group assets by function, sensitivity, and compliance |
| **Least Privilege** | Only allow traffic required for business function |
| **Default Deny** | Block all inter-segment traffic by default |
| **Monitor** | Log and alert on cross-segment traffic anomalies |
| **Start Simple** | Begin with coarse segmentation, refine to microsegmentation |
| **Compliance Alignment** | PCI DSS (CDE isolation), HIPAA (PHI protection) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Segmentation** | Dividing network into isolated segments |
| **Purpose** | Limit blast radius, control access, prevent lateral movement |
| **Methods** | Physical, VLANs, firewall zones, SDN, microsegmentation |
| **Microsegmentation** | Per-workload policies — Zero Trust approach |
| **Key Principle** | Default deny between segments |
| **Compliance** | Required by PCI DSS, recommended by NIST, HIPAA |

---

## Quick Revision Questions

1. **What is network segmentation and why is it important?**
2. **What is the risk of a flat network with no segmentation?**
3. **How do VLANs provide segmentation?**
4. **What is microsegmentation and how does it differ from VLANs?**
5. **What does "default deny" mean in segmentation?**
6. **Name 3 compliance frameworks that require or recommend segmentation.**

---

[← Previous: Firewall Placement](03-firewall-placement.md) | [Next: Cloud Security Basics →](05-cloud-security-basics.md)

---

*Unit 8 - Topic 4 of 6 | [Back to README](../README.md)*
