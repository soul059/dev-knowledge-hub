# Zero Trust Networking

## Unit 10 - Topic 2 | Network Segmentation

---

## Overview

**Zero Trust** is a security model based on the principle **"Never trust, always verify."** It assumes that threats exist both inside and outside the network, so no user, device, or connection should be automatically trusted — regardless of network location. Zero Trust is a paradigm shift from traditional perimeter-based security.

---

## 1. Traditional vs Zero Trust

```
┌──────────────────────────────────────────────────────────────┐
│              SECURITY MODEL COMPARISON                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TRADITIONAL (Castle-and-Moat):                              │
│  ┌─────────────────────────────┐                             │
│  │    TRUSTED ZONE             │                             │
│  │  Once inside, free access   │ ← Firewall at perimeter    │
│  │  User A → Any Server ✅     │                             │
│  │  Compromised → Moves freely │                             │
│  └─────────────────────────────┘                             │
│                                                              │
│  ZERO TRUST:                                                 │
│  ┌─────────────────────────────┐                             │
│  │    NO TRUSTED ZONE          │                             │
│  │  Every access is verified   │                             │
│  │  User A → Server X? VERIFY  │ ← Identity + Device +      │
│  │  User A → Server Y? VERIFY  │   Context checked EVERY    │
│  │  Even internal = untrusted  │   time                     │
│  └─────────────────────────────┘                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Zero Trust Principles

| Principle | Description |
|-----------|-------------|
| **Verify explicitly** | Always authenticate and authorize based on all data points |
| **Least privilege access** | Limit access to minimum needed, just-in-time |
| **Assume breach** | Design as if the network is already compromised |
| **Micro-segment** | Divide network into small, controlled segments |
| **Continuous validation** | Re-verify throughout the session, not just at login |
| **Encrypt everything** | All traffic encrypted, even internal |

---

## 3. Zero Trust Architecture Components

```
┌──────────────────────────────────────────────────────────────┐
│              ZERO TRUST ARCHITECTURE                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  USER ──► Identity Provider (MFA, SSO)                       │
│           │                                                  │
│           ▼                                                  │
│  DEVICE ──► Device Trust (compliance, health check)          │
│           │                                                  │
│           ▼                                                  │
│  POLICY ENGINE ──► Makes access decisions based on:          │
│           │        • User identity & role                    │
│           │        • Device health & compliance              │
│           │        • Location & time                         │
│           │        • Resource sensitivity                    │
│           │        • Behavior analytics                      │
│           ▼                                                  │
│  POLICY ENFORCEMENT POINT ──► Grants/denies access           │
│           │                                                  │
│           ▼                                                  │
│  RESOURCE (app, data, service)                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Key Technologies

| Technology | Role in Zero Trust |
|-----------|-------------------|
| **Identity Provider (IdP)** | Centralized authentication (Azure AD, Okta) |
| **MFA** | Multi-factor authentication for all access |
| **SSO** | Single Sign-On with continuous validation |
| **Micro-segmentation** | Network divided into secure zones |
| **SDP (Software-Defined Perimeter)** | Application-level access control |
| **ZTNA (Zero Trust Network Access)** | Replaces VPN with identity-based access |
| **SIEM/SOAR** | Monitor, detect, and respond to anomalies |
| **UEBA** | User and Entity Behavior Analytics |
| **EDR/XDR** | Endpoint detection and response |
| **DLP** | Data Loss Prevention |

---

## 5. Zero Trust vs VPN

| Feature | Traditional VPN | ZTNA |
|---------|:-:|:-:|
| **Trust model** | Trust after VPN connect | Never trust, always verify |
| **Network access** | Full network access | Per-application access |
| **Lateral movement** | Easy once connected | Blocked by design |
| **User experience** | VPN client, split tunnel issues | Seamless, transparent |
| **Scalability** | VPN concentrator bottleneck | Cloud-native, scalable |

---

## 6. NIST Zero Trust Framework (SP 800-207)

| Component | Description |
|-----------|-------------|
| **Policy Engine (PE)** | Decides whether to grant access |
| **Policy Administrator (PA)** | Executes the PE's decisions |
| **Policy Enforcement Point (PEP)** | Enforces access at the resource |
| **Data sources** | Identity, device, threat intel, logs |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Zero Trust** | Never trust, always verify |
| **Core principle** | No implicit trust based on network location |
| **Assume breach** | Design as if already compromised |
| **ZTNA** | Replaces VPN with per-app access |
| **Key tech** | IdP, MFA, micro-segmentation, SDP |
| **Framework** | NIST SP 800-207 |

---

## Quick Revision Questions

1. **What does "Never trust, always verify" mean?**
2. **How does Zero Trust differ from traditional perimeter security?**
3. **What are the three core principles of Zero Trust?**
4. **How does ZTNA replace traditional VPN?**
5. **What is the role of the Policy Engine in Zero Trust?**

---

[← Previous: Micro-Segmentation](01-micro-segmentation.md) | [Next: Software-Defined Networking →](03-software-defined-networking.md)

---

*Unit 10 - Topic 2 of 5 | [Back to README](../README.md)*
