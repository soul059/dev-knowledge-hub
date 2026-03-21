# Micro-Segmentation

## Unit 10 - Topic 1 | Network Segmentation

---

## Overview

**Micro-segmentation** is a security technique that divides the network into small, isolated segments — down to the individual workload or application level. Unlike traditional segmentation (VLANs/subnets), micro-segmentation controls **east-west traffic** (lateral movement between servers) rather than just **north-south traffic** (in/out of the network).

---

## 1. Traditional vs Micro-Segmentation

```
┌──────────────────────────────────────────────────────────────┐
│              SEGMENTATION COMPARISON                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TRADITIONAL SEGMENTATION (VLANs):                           │
│  ┌───────────────────────┐                                   │
│  │      VLAN 10          │  Firewall between VLANs           │
│  │  Srv1  Srv2  Srv3     │  but Srv1/Srv2/Srv3 can           │
│  │  (all can talk freely)│  freely communicate               │
│  └───────────────────────┘                                   │
│                                                              │
│  MICRO-SEGMENTATION:                                         │
│  ┌───────────────────────┐                                   │
│  │  ┌─────┐ ┌─────┐ ┌─────┐                                │
│  │  │Srv1 │ │Srv2 │ │Srv3 │  Each server has its            │
│  │  │ FW  │ │ FW  │ │ FW  │  own firewall policy             │
│  │  └──┬──┘ └──┬──┘ └──┬──┘                                 │
│  │     │       │       │    Only allowed connections          │
│  │     └───────┼───────┘    between specific services        │
│  └───────────────────────┘                                   │
│                                                              │
│  Srv1:Web → Srv2:API (port 8080 only) ✅                     │
│  Srv1:Web → Srv3:DB (blocked) ❌                             │
│  Srv2:API → Srv3:DB (port 5432 only) ✅                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Why Micro-Segmentation Matters

| Without Micro-Segmentation | With Micro-Segmentation |
|---------------------------|------------------------|
| Attacker compromises one server → moves freely to all servers in segment | Attacker compromises one server → blocked from reaching others |
| Flat network inside VLANs | Granular controls between every workload |
| Perimeter-only security | Defense in depth at every layer |
| Hard to detect lateral movement | Unusual connections immediately flagged |

---

## 3. Implementation Approaches

| Approach | Description | Examples |
|----------|-------------|---------|
| **Host-based firewalls** | Firewall rules on each server | iptables, Windows Firewall |
| **Hypervisor-based** | Control traffic at the virtual switch level | VMware NSX, Nutanix Flow |
| **Agent-based** | Software agent on each workload | Illumio, Guardicore |
| **Cloud-native** | Security groups, network policies | AWS Security Groups, K8s NetworkPolicy |
| **SDN-based** | Software-defined networking controls | Cisco ACI, VMware NSX |

---

## 4. Kubernetes Network Policies (Example)

```yaml
# Only allow web pod to talk to API pod on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-to-api
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web
    ports:
    - port: 8080
      protocol: TCP
```

---

## 5. Benefits for Security

| Benefit | Description |
|---------|-------------|
| **Limits blast radius** | Compromise of one workload doesn't spread |
| **Prevents lateral movement** | Attackers can't pivot between segments |
| **Visibility** | Maps all application communication flows |
| **Compliance** | Meet PCI DSS, HIPAA segmentation requirements |
| **Zero Trust alignment** | Never trust, always verify — even inside the network |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Micro-segmentation** | Per-workload security policies |
| **East-West traffic** | Server-to-server (lateral) traffic controlled |
| **Implementation** | Host firewalls, hypervisor, agents, SDN |
| **Key benefit** | Stops lateral movement after compromise |
| **Tools** | VMware NSX, Illumio, AWS Security Groups, K8s NetworkPolicy |

---

## Quick Revision Questions

1. **How does micro-segmentation differ from traditional VLAN segmentation?**
2. **What is east-west vs north-south traffic?**
3. **Name 3 implementation approaches for micro-segmentation.**
4. **How does micro-segmentation prevent lateral movement?**
5. **How does micro-segmentation align with Zero Trust?**

---

[Next: Zero Trust Networking →](02-zero-trust-networking.md)

---

*Unit 10 - Topic 1 of 5 | [Back to README](../README.md)*
