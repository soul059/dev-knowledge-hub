# IPv6 Security Considerations

## Unit 3 - Topic 6 | Layer 3 Security

---

## Overview

**IPv6** is the next-generation internet protocol designed to replace IPv4. While IPv6 includes built-in improvements like **mandatory IPsec support** and a larger address space, it also introduces **new attack surfaces** that many organizations aren't prepared for. IPv6 is often enabled by default on modern systems, creating shadow attack vectors.

---

## 1. IPv6 vs IPv4 Security Comparison

| Feature | IPv4 | IPv6 |
|---------|------|------|
| **Address Size** | 32 bits (4.3 billion) | 128 bits (340 undecillion) |
| **IPsec** | Optional | Mandatory (spec), optional (practice) |
| **ARP** | Yes — ARP spoofing | No ARP — uses NDP (Neighbor Discovery) |
| **Broadcast** | Yes | No broadcast — uses multicast |
| **Auto-Config** | DHCP | SLAAC + DHCPv6 |
| **NAT** | Common | Generally not needed (enough addresses) |
| **Scanning** | Easy (/24 = 254 hosts) | Hard (/64 = 2⁶⁴ hosts!) |
| **Fragmentation** | Routers can fragment | Only source can fragment |

---

## 2. IPv6-Specific Threats

### NDP (Neighbor Discovery Protocol) Attacks

```
┌──────────────────────────────────────────────────────────────┐
│              NDP ATTACKS (IPv6's "ARP")                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  NDP replaces ARP in IPv6 but has SIMILAR vulnerabilities:   │
│                                                              │
│  • Neighbor Spoofing: Like ARP spoofing → MITM              │
│  • Router Advertisement Spoofing: Fake default gateway       │
│  • Redirect Attack: Redirect traffic via rogue router        │
│  • DAD DoS: Deny every Duplicate Address Detection           │
│                                                              │
│  TOOLS:                                                      │
│  • THC-IPv6 toolkit (parasite6, fake_router6, etc.)         │
│  • Scapy (craft custom IPv6/NDP packets)                     │
│  • SI6 Networks IPv6 toolkit                                 │
│                                                              │
│  DEFENSE:                                                    │
│  • RA Guard (Router Advertisement Guard)                     │
│  • SEND (Secure Neighbor Discovery) — rarely deployed        │
│  • IPv6 First Hop Security features                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Rogue Router Advertisement

```
Attacker sends fake Router Advertisement (RA) messages:
"I am the default gateway for this network"

All IPv6 hosts auto-configure to use attacker as gateway
→ Man-in-the-Middle for ALL IPv6 traffic

Tool: fake_router6 eth0
Defense: RA Guard on switch
```

### IPv6 as Shadow Channel

```
┌──────────────────────────────────────────────────────────────┐
│              IPv6 SHADOW CHANNEL                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PROBLEM:                                                    │
│  • Most OS have IPv6 ENABLED by default                      │
│  • Many organizations only monitor/filter IPv4               │
│  • IPv6 traffic bypasses IPv4 firewalls/IDS                  │
│                                                              │
│  ATTACK:                                                     │
│  Attacker uses IPv6 for C2 communication                     │
│  → IPv4-only monitoring misses everything                    │
│                                                              │
│  DEFENSE:                                                    │
│  • If not using IPv6: DISABLE it completely                  │
│  • If using IPv6: Monitor and filter BOTH protocols          │
│  • Apply same security policies to IPv6 as IPv4              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. IPv6 Extension Headers Abuse

```
IPv6 uses extension headers instead of IPv4 options.
These can be chained and abused:

┌──────────┬────────────┬────────────┬────────────┬─────────┐
│ IPv6 Hdr │ Hop-by-Hop │ Routing    │ Fragment   │ Payload │
│          │ Options    │ Header     │ Header     │         │
└──────────┴────────────┴────────────┴────────────┴─────────┘

ATTACKS:
• Chain many extension headers → firewall/IDS bypass
• Type 0 Routing Header → source routing (deprecated)
• Fragment header → fragmentation evasion (same as IPv4)
• Oversized header chains → DoS (processing overhead)
```

---

## 4. IPv6 Security Best Practices

| Practice | Description |
|----------|-------------|
| **Disable if unused** | If not using IPv6, disable on all interfaces |
| **RA Guard** | Block rogue Router Advertisements on switch |
| **DHCPv6 Guard** | Block rogue DHCPv6 servers |
| **Dual-stack security** | Apply same firewall/IDS rules to IPv4 AND IPv6 |
| **Monitor IPv6** | Include IPv6 in SIEM, IDS, traffic analysis |
| **Filter extension headers** | Block unnecessary extension header types |
| **SEND** | Secure NDP with cryptographic signatures (if feasible) |
| **IPv6 ACLs** | Apply access control lists to IPv6 traffic |

```cisco
! RA Guard (Cisco switch)
ipv6 nd raguard policy RAGUARD
  device-role host
interface GigabitEthernet0/1
  ipv6 nd raguard attach-policy RAGUARD

! Disable IPv6 (if not needed)
interface GigabitEthernet0/1
  no ipv6 enable
  no ipv6 address

# Linux — Disable IPv6
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
sysctl -p
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **IPv6 NDP** | Replaces ARP but has similar spoofing vulnerabilities |
| **RA Spoofing** | Fake router advertisements → MITM |
| **Shadow Channel** | IPv6 enabled by default → bypasses IPv4-only security |
| **Extension Headers** | Can be abused for evasion and DoS |
| **Key Defense** | Disable if unused, or apply dual-stack security |

---

## Quick Revision Questions

1. **How does NDP in IPv6 differ from ARP in IPv4?**
2. **What is a rogue Router Advertisement attack?**
3. **Why is IPv6 a "shadow channel" risk in many organizations?**
4. **How can IPv6 extension headers be abused?**
5. **What is RA Guard and where should it be deployed?**
6. **What should organizations do if they don't use IPv6?**

---

[← Previous: Fragmentation Attacks](05-fragmentation-attacks.md)

---

*Unit 3 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Layer 4 Security →](../04-Layer-4-Security/01-tcp-three-way-handshake.md)*
