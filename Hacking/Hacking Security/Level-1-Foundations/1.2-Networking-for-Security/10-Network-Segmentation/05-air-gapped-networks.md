# Air-Gapped Networks

## Unit 10 - Topic 5 | Network Segmentation

---

## Overview

An **air-gapped network** is a network that is physically isolated from all other networks, including the Internet. It has **no wired or wireless connections** to external systems. Air gaps represent the **highest level of network segmentation** and are used for the most sensitive systems — military, critical infrastructure, classified data, and industrial control systems.

---

## 1. What Is an Air Gap?

```
┌──────────────────────────────────────────────────────────────┐
│              AIR-GAPPED NETWORK                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CONNECTED NETWORK:          AIR-GAPPED NETWORK:             │
│  ┌──────────────┐           ┌──────────────┐                │
│  │  Corporate   │           │  Classified  │                │
│  │  Network     │           │  Network     │                │
│  │     │        │           │              │                │
│  │     │        │   ═══╪═══ │  NO physical │                │
│  │  Internet    │  AIR GAP  │  connection  │                │
│  │  Connected   │           │  to anything │                │
│  └──────────────┘           └──────────────┘                │
│                                                              │
│  The "air gap" = physical separation                         │
│  No cables, no Wi-Fi, no Bluetooth, no RF                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Where Air Gaps Are Used

| Sector | Use Case |
|--------|----------|
| **Military/Intelligence** | Classified networks (SIPRNet, JWICS) |
| **Nuclear Facilities** | SCADA/ICS controlling nuclear reactors |
| **Critical Infrastructure** | Power grid, water treatment control systems |
| **Financial** | High-frequency trading systems, key management |
| **Healthcare** | Medical device networks in some facilities |
| **Cryptocurrency** | Cold wallets / offline key storage |
| **Voting Systems** | Election tabulation systems |

---

## 3. Air Gap Attack Techniques

Despite physical isolation, air gaps have been breached using creative techniques:

| Attack | Method | Notable Example |
|--------|--------|----------------|
| **USB/Removable Media** | Infected USB drive carried across air gap | **Stuxnet** (Iran nuclear centrifuges) |
| **Supply Chain** | Pre-infected hardware/software | Various APT campaigns |
| **Insider Threat** | Trusted person smuggles data | Snowden / Manning |
| **Acoustic** | Data encoded in ultrasonic sound | Research demos (Fansmitter) |
| **Electromagnetic** | Data leaked via EM emissions (TEMPEST) | TEMPEST standards exist for this |
| **Optical** | Data transmitted via LED blinking | Research demos (LED-it-GO) |
| **Thermal** | Temperature changes between computers | Research demo (BitWhisper) |
| **RF** | Radio signals from compromised hardware | Research demos |

```
STUXNET ATTACK FLOW:
1. Malware created by nation-state attackers
2. Spread via infected USB drives
3. USB carried into air-gapped Iranian nuclear facility
4. Malware targeted Siemens PLCs
5. Modified centrifuge speeds while showing normal readings
6. Destroyed ~1,000 uranium enrichment centrifuges
7. First known cyber weapon against air-gapped ICS
```

---

## 4. Air Gap Security Controls

| Control | Description |
|---------|-------------|
| **USB port disable** | Physically or logically disable USB ports |
| **Removable media policy** | Strict controls on any media entering facility |
| **Media scanning stations** | Scan all media before introducing to air-gapped network |
| **TEMPEST shielding** | Faraday cage / EM shielding for sensitive areas |
| **Physical security** | Mantrap, guards, cameras, no phones in secure area |
| **Supply chain verification** | Verify hardware/software integrity |
| **Data diodes** | One-way data transfer devices (in only, or out only) |
| **Personnel vetting** | Background checks for all personnel with access |
| **Acoustic isolation** | Sound-dampening for ultra-sensitive environments |

---

## 5. Data Diodes

```
┌──────────────────────────────────────────────────────────────┐
│              DATA DIODE (One-Way Transfer)                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐               │
│  │ External │───►│  DATA    │───►│ Air-Gap  │               │
│  │ Network  │    │  DIODE   │    │ Network  │               │
│  └──────────┘    └──────────┘    └──────────┘               │
│                  (one-way only)                               │
│                                                              │
│  • Hardware device — physically enforces one direction       │
│  • Cannot be bypassed by software                            │
│  • Used to send updates IN but no data can flow OUT          │
│  • Or send logs OUT but nothing can flow IN                  │
│                                                              │
│  Vendors: Waterfall Security, Owl Cyber Defense              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Air Gap** | Physical isolation from all other networks |
| **Use Cases** | Military, nuclear, critical infrastructure |
| **Not invincible** | Stuxnet proved air gaps can be breached |
| **Main threat** | Removable media (USB), insider threats |
| **Data Diodes** | Hardware-enforced one-way data transfer |
| **TEMPEST** | Protection against electromagnetic emanation |

---

## Quick Revision Questions

1. **What is an air-gapped network?**
2. **How did Stuxnet breach an air-gapped network?**
3. **Name 3 exotic techniques for attacking air-gapped networks.**
4. **What is a data diode and how does it work?**
5. **What physical security controls protect air-gapped networks?**

---

[← Previous: Network Access Control](04-network-access-control.md)

---

*Unit 10 - Topic 5 of 5 | [Back to README](../README.md)*

---

### 🎉 Congratulations! You have completed Subject 1.2: Networking for Security Professionals!
