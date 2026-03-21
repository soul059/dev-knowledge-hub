# Zero Trust Model

## Unit 3 - Topic 5 | Security Principles

---

## Overview

**Zero Trust** is a security model based on the principle: **"Never trust, always verify."** Unlike traditional perimeter-based security that trusts everything inside the network, Zero Trust assumes that threats can exist both inside and outside the network. Every access request must be authenticated, authorized, and continuously validated.

---

## 1. Zero Trust vs Traditional Security

```
┌──────────────────────────────────────────────────────────────────┐
│           TRADITIONAL (Castle-and-Moat) vs ZERO TRUST            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TRADITIONAL MODEL ❌                                             │
│  ─────────────────────                                           │
│         Outside = UNTRUSTED                                      │
│              │                                                   │
│         [Firewall/VPN]                                           │
│              │                                                   │
│         Inside = TRUSTED  ← Everything inside is trusted         │
│         (Free movement)      Compromised insider = game over     │
│                                                                  │
│                                                                  │
│  ZERO TRUST MODEL ✅                                              │
│  ─────────────────────                                           │
│         NOTHING is trusted by default                            │
│              │                                                   │
│         Every request verified:                                  │
│         ├── Who are you? (Identity)                              │
│         ├── What device? (Device health)                         │
│         ├── Where from? (Location/network)                       │
│         ├── What access? (Least privilege)                       │
│         └── Is this normal? (Behavior analysis)                  │
│                                                                  │
│         Compromised insider = limited blast radius               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Principles of Zero Trust

```
┌──────────────────────────────────────────────────────────────────┐
│                  ZERO TRUST PILLARS                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. VERIFY EXPLICITLY                                            │
│     Always authenticate and authorize based on ALL available     │
│     data: identity, location, device, service, data, anomalies  │
│                                                                  │
│  2. USE LEAST PRIVILEGE ACCESS                                   │
│     Limit access with JIT/JEA (Just-In-Time/Just-Enough-Access) │
│     Risk-based adaptive policies, data protection                │
│                                                                  │
│  3. ASSUME BREACH                                                │
│     Minimize blast radius and segment access                     │
│     Verify end-to-end encryption, use analytics for detection    │
│     Improve defenses continuously                                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Zero Trust Architecture Components

```
┌──────────────────────────────────────────────────────────────────────┐
│                    ZERO TRUST ARCHITECTURE                           │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  User/Device ──► Policy Engine ──► Access Decision                   │
│       │              │                  │                             │
│       │              ▼                  ▼                             │
│       │         ┌──────────────────────────────┐                     │
│       │         │     POLICY DECISION POINT     │                     │
│       │         │  ┌────────┐  ┌────────────┐  │                     │
│       │         │  │Identity│  │  Device    │  │                     │
│       │         │  │ Check  │  │  Health    │  │                     │
│       │         │  └────────┘  └────────────┘  │                     │
│       │         │  ┌────────┐  ┌────────────┐  │                     │
│       │         │  │Location│  │  Behavior  │  │                     │
│       │         │  │ Check  │  │  Analysis  │  │                     │
│       │         │  └────────┘  └────────────┘  │                     │
│       │         └──────────────┬───────────────┘                     │
│       │                       │                                      │
│       │                       ▼                                      │
│       │              ┌──────────────┐                                │
│       └─────────────►│  RESOURCE    │ Access granted per-session     │
│                      │  (App/Data)  │ Continuously monitored         │
│                      └──────────────┘                                │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Key Components

| Component | Description |
|-----------|-------------|
| **Identity Provider** | Verify user identity (Azure AD, Okta) |
| **Device Trust** | Check device health, compliance, patching |
| **Policy Engine** | Evaluate all signals, make access decisions |
| **Micro-segmentation** | Granular network segments, not flat networks |
| **Encryption** | All traffic encrypted, even internal |
| **Continuous Monitoring** | Real-time analytics, UEBA, SIEM |
| **MFA Everywhere** | Multi-factor for all access, not just VPN |

---

## 4. Zero Trust Implementation Steps

```
Step 1: IDENTIFY
   ├── Map all assets, data, users, and applications
   ├── Classify data sensitivity
   └── Identify access patterns ("transaction flows")

Step 2: PROTECT
   ├── Implement strong identity verification (MFA)
   ├── Micro-segment the network
   ├── Encrypt all communications
   └── Apply least privilege access

Step 3: DETECT
   ├── Monitor all traffic and access requests
   ├── Implement behavioral analytics (UEBA)
   ├── Correlate signals across identity, device, network
   └── Alert on anomalies

Step 4: RESPOND
   ├── Automate response to detected threats
   ├── Revoke access in real-time
   ├── Investigate and remediate
   └── Update policies based on findings

Step 5: RECOVER
   ├── Restore from verified backups
   ├── Re-validate all access
   ├── Conduct lessons learned
   └── Improve policies
```

---

## 5. Zero Trust Maturity Model

| Stage | Description | Characteristics |
|-------|-------------|----------------|
| **Traditional** | Perimeter-based | VPN, castle-and-moat, flat network |
| **Initial** | Starting ZT | Some identity verification, basic segmentation |
| **Advanced** | Significant ZT | MFA everywhere, micro-segmentation, analytics |
| **Optimal** | Full ZT | Continuous verification, automated response, AI-driven |

---

## 6. NIST Zero Trust Architecture (SP 800-207)

| NIST Principle | Description |
|---------------|-------------|
| All data and services are resources | Not just servers — everything |
| All communication is secured | Regardless of network location |
| Access is granted per-session | Each request evaluated independently |
| Access is determined by dynamic policy | Context-aware, risk-based |
| Monitor all assets | Continuous, not periodic |
| Authentication is dynamic and strict | MFA, risk-based step-up |
| Collect information for improvement | Use data to refine policies |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Core Motto** | "Never trust, always verify" |
| **vs Traditional** | No trusted zones — verify everything |
| **Three Pillars** | Verify explicitly, Least privilege, Assume breach |
| **Key Technologies** | MFA, micro-segmentation, UEBA, encryption |
| **NIST Standard** | SP 800-207 |
| **Implementation** | Identify → Protect → Detect → Respond → Recover |

---

## Quick Revision Questions

1. **What is the core principle of Zero Trust in one sentence?**
2. **How does Zero Trust differ from traditional perimeter security?**
3. **What are the three pillars of Zero Trust?**
4. **What is micro-segmentation and why is it important for Zero Trust?**
5. **Why must "assume breach" be part of Zero Trust thinking?**
6. **What NIST publication defines Zero Trust Architecture?**

---

[← Previous: Defense in Depth](04-defense-in-depth.md) | [Next: Security by Design →](06-security-by-design.md)

---

*Unit 3 - Topic 5 of 6 | [Back to README](../README.md)*
