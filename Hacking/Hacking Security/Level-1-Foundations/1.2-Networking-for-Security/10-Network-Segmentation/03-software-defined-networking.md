# Software-Defined Networking (SDN)

## Unit 10 - Topic 3 | Network Segmentation

---

## Overview

**Software-Defined Networking (SDN)** separates the network's **control plane** (decision-making) from the **data plane** (packet forwarding), enabling centralized, programmable network management. SDN enables **dynamic segmentation**, **automated security policies**, and **rapid response** to threats — fundamentally changing how networks are secured.

---

## 1. Traditional vs SDN Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              TRADITIONAL vs SDN                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TRADITIONAL:                                                │
│  Each device has its own control + data plane                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │ Control  │  │ Control  │  │ Control  │                  │
│  │──────────│  │──────────│  │──────────│                  │
│  │  Data    │  │  Data    │  │  Data    │                  │
│  │ Switch 1 │  │ Switch 2 │  │ Switch 3 │                  │
│  └──────────┘  └──────────┘  └──────────┘                  │
│  Config each device individually (CLI)                       │
│                                                              │
│  SDN:                                                        │
│  Centralized controller manages all devices                  │
│  ┌──────────────────────────────────┐                       │
│  │       SDN CONTROLLER            │                       │
│  │    (Centralized Control Plane)   │                       │
│  └──────┬──────────┬──────────┬────┘                       │
│         │          │          │   OpenFlow                   │
│    ┌────┴───┐ ┌────┴───┐ ┌───┴────┐                        │
│    │ Data   │ │ Data   │ │ Data   │                        │
│    │Switch 1│ │Switch 2│ │Switch 3│                        │
│    └────────┘ └────────┘ └────────┘                        │
│  Program all from single controller                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. SDN Components

| Component | Description |
|-----------|-------------|
| **Application Layer** | Security apps, monitoring, policies |
| **Control Layer** | SDN controller (the "brain") |
| **Infrastructure Layer** | Physical/virtual switches and routers |
| **Northbound API** | Controller ↔ Applications communication |
| **Southbound API** | Controller ↔ Network devices (OpenFlow) |

---

## 3. Security Benefits of SDN

| Benefit | Description |
|---------|-------------|
| **Dynamic segmentation** | Create/modify segments in real-time via API |
| **Automated response** | Detect threat → automatically isolate host |
| **Centralized visibility** | Single view of all network traffic and policies |
| **Consistent policies** | Apply uniform rules across entire network |
| **Rapid quarantine** | Isolate compromised device in seconds |
| **Granular control** | Per-flow policies (not just per-port/VLAN) |

---

## 4. SDN Security Use Cases

```
AUTOMATED THREAT RESPONSE:
1. IDS detects malware on Host X
2. IDS alerts SDN controller via API
3. Controller pushes new flow rules to all switches
4. Host X is immediately isolated (quarantined)
5. All happens in seconds — no manual intervention

DYNAMIC SEGMENTATION:
1. New compliance requirement: PCI data must be isolated
2. Admin defines policy in SDN controller
3. Controller automatically creates micro-segments
4. Only authorized traffic flows between PCI systems
5. Policy enforced across all switches simultaneously
```

---

## 5. SDN Security Risks

| Risk | Description | Mitigation |
|------|-------------|-----------|
| **Controller compromise** | Single point of failure — if controller hacked, entire network compromised | Controller hardening, redundancy, MFA |
| **API abuse** | Unauthorized API calls can reconfigure network | API authentication, rate limiting |
| **DoS on controller** | Flood controller with requests | Controller redundancy, flow limits |
| **Man-in-the-middle** | Intercept controller-switch communication | TLS for all SDN communications |
| **Rogue applications** | Malicious app on northbound API | App authentication, code signing |

---

## 6. SDN Solutions

| Solution | Type | Notes |
|----------|------|-------|
| **OpenDaylight** | Open source | Java-based SDN controller |
| **ONOS** | Open source | Carrier-grade SDN platform |
| **VMware NSX** | Commercial | Network virtualization + micro-segmentation |
| **Cisco ACI** | Commercial | Application-centric infrastructure |
| **OpenFlow** | Protocol | Standard southbound API for SDN |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SDN** | Separates control plane from data plane |
| **Controller** | Centralized brain — manages all devices |
| **Security Benefit** | Dynamic segmentation, automated quarantine |
| **Key Risk** | Controller is single point of failure |
| **OpenFlow** | Standard protocol for SDN communication |
| **Use Case** | Automated threat response in seconds |

---

## Quick Revision Questions

1. **What does SDN separate and why?**
2. **What are the three layers of SDN architecture?**
3. **How can SDN automate threat response?**
4. **What is the biggest security risk of SDN?**
5. **What is OpenFlow?**

---

[← Previous: Zero Trust Networking](02-zero-trust-networking.md) | [Next: Network Access Control →](04-network-access-control.md)

---

*Unit 10 - Topic 3 of 5 | [Back to README](../README.md)*
