# VPN Gateways

## Unit 8 - Topic 6 | Network Security Devices

---

## Overview

A **VPN (Virtual Private Network) Gateway** creates encrypted tunnels across untrusted networks, enabling secure remote access and site-to-site connectivity. VPNs protect data **confidentiality** and **integrity** in transit, making them essential for remote work, branch office connectivity, and secure communications.

---

## 1. VPN Types

```
┌──────────────────────────────────────────────────────────────┐
│              VPN ARCHITECTURES                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SITE-TO-SITE VPN:                                           │
│  ┌─────────┐      ╔═══encrypted═══╗      ┌─────────┐       │
│  │ Office A │──────║   Internet    ║──────│ Office B │       │
│  │ Gateway  │      ╚═══tunnel═════╝      │ Gateway  │       │
│  └─────────┘                              └─────────┘       │
│  • Connects two networks permanently                         │
│  • Always-on, transparent to users                           │
│                                                              │
│  REMOTE ACCESS VPN:                                          │
│  ┌────────┐       ╔═══encrypted═══╗      ┌──────────┐      │
│  │ Remote  │──────║   Internet    ║──────│ Corporate │      │
│  │ User    │      ╚═══tunnel═════╝      │ Gateway   │      │
│  └────────┘                              └──────────┘      │
│  • Individual user connects to corporate network             │
│  • On-demand, requires authentication                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. VPN Protocols

| Protocol | Layer | Encryption | Speed | Security |
|----------|:-----:|-----------|:-----:|:--------:|
| **IPSec** | 3 | AES, 3DES | ★★★★ | ★★★★★ |
| **OpenVPN** | 3-4 | OpenSSL | ★★★ | ★★★★★ |
| **WireGuard** | 3 | ChaCha20 | ★★★★★ | ★★★★★ |
| **SSL/TLS VPN** | 4-7 | TLS | ★★★ | ★★★★ |
| **L2TP/IPSec** | 2-3 | IPSec | ★★★ | ★★★★ |
| **PPTP** | 2 | MPPE | ★★★★ | ★ (broken!) |

> ⚠️ **PPTP is deprecated** — MSCHAPv2 authentication is easily cracked. Never use PPTP.

---

## 3. IPSec Components

```
IPSec Framework:
├── IKE (Internet Key Exchange)
│   ├── Phase 1: Establish secure channel (ISAKMP SA)
│   │   Authentication: PSK or Certificates
│   └── Phase 2: Negotiate IPSec tunnel (IPSec SA)
│
├── AH (Authentication Header)
│   └── Integrity + Authentication (no encryption)
│
├── ESP (Encapsulating Security Payload)
│   └── Confidentiality + Integrity + Authentication
│
└── Modes:
    ├── Transport Mode: Encrypts payload only (host-to-host)
    └── Tunnel Mode: Encrypts entire packet (gateway-to-gateway)
```

---

## 4. VPN Security Considerations

| Risk | Mitigation |
|------|-----------|
| **Split tunneling** | Only VPN traffic goes through tunnel; direct internet bypasses security | Use full tunnel mode |
| **Weak authentication** | Single-factor VPN access | Require MFA |
| **Compromised endpoint** | Infected device gets VPN access | Endpoint compliance checks |
| **VPN credential theft** | Phishing for VPN passwords | Certificate-based auth + MFA |
| **Outdated protocols** | PPTP, weak ciphers | Use modern protocols (WireGuard, OpenVPN) |

---

## 5. VPN Solutions

| Solution | Type | Notes |
|----------|------|-------|
| **OpenVPN** | Open source | SSL/TLS-based, very flexible |
| **WireGuard** | Open source | Modern, fast, minimal code |
| **Cisco AnyConnect** | Commercial | Enterprise remote access |
| **Palo Alto GlobalProtect** | Commercial | NGFW-integrated VPN |
| **strongSwan** | Open source | IPSec implementation |
| **AWS VPN** | Cloud | Site-to-site and client VPN |

---

## 6. VPN Attack Vectors

| Attack | Description |
|--------|-------------|
| **Credential stuffing** | Use stolen credentials to access VPN |
| **VPN vulnerability exploit** | CVEs in VPN software (e.g., Pulse Secure, Fortinet) |
| **Man-in-the-Middle** | Intercept VPN setup if improperly configured |
| **DNS leaks** | DNS queries bypass VPN tunnel |
| **IPv6 leaks** | IPv6 traffic bypasses IPv4 VPN |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **VPN** | Encrypted tunnel over untrusted networks |
| **Site-to-Site** | Connects two networks (always-on) |
| **Remote Access** | Individual user to corporate (on-demand) |
| **IPSec** | Standard site-to-site protocol |
| **WireGuard** | Modern, fast, recommended |
| **PPTP** | Broken — never use |

---

## Quick Revision Questions

1. **What is the difference between site-to-site and remote access VPN?**
2. **Why should PPTP never be used?**
3. **What are the two IPSec modes and when is each used?**
4. **What is split tunneling and why is it a security risk?**
5. **Name 3 modern VPN protocols.**

---

[← Previous: Proxy Servers](05-proxy-servers.md)

---

*Unit 8 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Wireless Networks →](../09-Wireless-Networks/01-802-11-standards.md)*
