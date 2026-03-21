# Routing Protocol Security

## Unit 3 - Topic 3 | Layer 3 Security

---

## Overview

**Routing protocols** (OSPF, BGP, RIP, EIGRP) determine how routers discover network paths and forward traffic. If an attacker can inject **false routing information**, they can redirect traffic for interception (MITM), cause network outages (DoS), or create routing loops. Securing routing protocols is critical for network infrastructure protection.

---

## 1. Common Routing Protocols

| Protocol | Type | Scope | Security | Notes |
|----------|------|-------|:--------:|-------|
| **RIP** | Distance Vector | Small networks | ❌ No built-in auth | v2 supports MD5 auth |
| **OSPF** | Link State | Enterprise | ✅ MD5/SHA auth | Area-based, widely used |
| **EIGRP** | Hybrid | Enterprise (Cisco) | ✅ MD5/SHA auth | Cisco proprietary |
| **BGP** | Path Vector | Internet | ⚠️ Optional auth | Routes between ISPs |
| **IS-IS** | Link State | ISP/Large enterprise | ✅ Auth support | Common in SP networks |

---

## 2. Routing Protocol Attacks

```
┌──────────────────────────────────────────────────────────────────┐
│                  ROUTING ATTACKS                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ROUTE INJECTION / POISONING:                                    │
│  Attacker injects false routes into routing tables               │
│                                                                  │
│  Normal:                        Attack:                          │
│  R1 ──── R2 ──── R3             R1 ──── R2 ──── R3              │
│                ↓                           ↓                    │
│             Server                    ATTACKER                   │
│                                    (fake route to                │
│                                     10.0.0.0/8)                 │
│                                                                  │
│  Result: Traffic for 10.0.0.0/8 goes to ATTACKER                │
│  → MITM, blackhole, or traffic analysis                         │
│                                                                  │
│  ATTACK TYPES:                                                   │
│  • Route injection — advertise fake networks                    │
│  • Route poisoning — corrupt existing routes                    │
│  • Route hijacking — claim to be a better path                  │
│  • Rogue router — add unauthorized router to network            │
│  • BGP hijacking — advertise another org's IP prefixes          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. BGP Hijacking (Internet-Scale)

```
┌──────────────────────────────────────────────────────────────┐
│              BGP HIJACKING                                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  BGP is the routing protocol of the INTERNET.                │
│  ISPs use BGP to exchange routing information.               │
│                                                              │
│  ATTACK: Malicious ASN announces routes for IP ranges        │
│  it does NOT own → internet traffic redirected               │
│                                                              │
│  Real-world examples:                                        │
│  • 2018: BGP hijack redirected Amazon DNS traffic            │
│  • 2022: Russian BGP hijack of Twitter, Facebook IPs        │
│  • Regular accidental hijacks cause outages                  │
│                                                              │
│  DEFENSE:                                                    │
│  • RPKI (Resource Public Key Infrastructure)                 │
│  • BGP Route Origin Validation                               │
│  • IRR (Internet Routing Registry) filtering                 │
│  • BGP monitoring services                                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Securing Routing Protocols

### OSPF Authentication

```cisco
! MD5 Authentication (recommended)
router ospf 1
  area 0 authentication message-digest

interface GigabitEthernet0/0
  ip ospf message-digest-key 1 md5 MySecretKey123

! SHA Authentication (more secure)
key chain OSPF-KEY
  key 1
    key-string MySecretKey123
    cryptographic-algorithm hmac-sha-256

interface GigabitEthernet0/0
  ip ospf authentication key-chain OSPF-KEY
```

### BGP Authentication

```cisco
! MD5 Authentication between BGP peers
router bgp 65001
  neighbor 10.0.0.2 remote-as 65002
  neighbor 10.0.0.2 password MyBGPSecret
```

### RIPv2 Authentication

```cisco
! Enable MD5 authentication
key chain RIP-KEY
  key 1
    key-string MyRIPSecret

interface GigabitEthernet0/0
  ip rip authentication mode md5
  ip rip authentication key-chain RIP-KEY
```

---

## 5. Routing Security Best Practices

| Practice | Description |
|----------|-------------|
| **Enable Authentication** | MD5/SHA on all routing protocols |
| **Filter Routes** | Distribute-lists to control which routes are accepted |
| **Passive Interfaces** | Don't send routing updates on user-facing interfaces |
| **Max Prefix Limits** | Limit number of routes accepted from a peer |
| **Route Filtering** | Prefix-lists to block invalid routes |
| **Logging** | Log all routing changes and neighbor events |
| **Monitoring** | BGP monitoring tools (RIPE RIS, BGPStream) |
| **RPKI** | Validate BGP route origins cryptographically |

```cisco
! Passive interface — stop routing updates on user VLANs
router ospf 1
  passive-interface default
  no passive-interface GigabitEthernet0/0   ! Only allow on uplinks
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Routing Attacks** | Inject false routes → MITM, DoS, traffic hijacking |
| **BGP Hijacking** | Internet-scale route theft — major real-world threat |
| **Authentication** | MD5/SHA authentication on all routing protocols |
| **Passive Interfaces** | Don't send routing info on user-facing ports |
| **RPKI** | Cryptographic route origin validation for BGP |

---

## Quick Revision Questions

1. **What is route injection and what impact can it have?**
2. **What is BGP hijacking and name a real-world example.**
3. **How does OSPF authentication prevent rogue router attacks?**
4. **What is a passive interface and why is it important?**
5. **What is RPKI and how does it help secure BGP?**

---

[← Previous: ICMP Abuse](02-icmp-abuse.md) | [Next: IP Spoofing →](04-ip-spoofing.md)

---

*Unit 3 - Topic 3 of 6 | [Back to README](../README.md)*
