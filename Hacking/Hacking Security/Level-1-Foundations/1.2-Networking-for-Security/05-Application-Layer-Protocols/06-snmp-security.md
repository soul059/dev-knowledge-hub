# SNMP Security

## Unit 5 - Topic 6 | Application Layer Protocols

---

## Overview

**SNMP (Simple Network Management Protocol)** is used to monitor and manage network devices (routers, switches, servers, printers). SNMPv1 and v2c use **cleartext community strings** (essentially passwords), making them a major security risk. SNMPv3 adds authentication and encryption but is often not deployed.

---

## 1. How SNMP Works

```
┌──────────────────────────────────────────────────────────────────┐
│                  SNMP ARCHITECTURE                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐         ┌──────────────┐                      │
│  │ SNMP Manager │◄═══════►│ SNMP Agent   │                      │
│  │ (NMS/SIEM)   │  UDP    │ (on device)  │                      │
│  │ Port 162     │ 161     │              │                      │
│  └──────────────┘         └──────────────┘                      │
│                                                                  │
│  OPERATIONS:                                                     │
│  GET     → Manager reads data from agent                        │
│  SET     → Manager changes config on agent ← DANGEROUS!        │
│  TRAP    → Agent sends alert to manager                         │
│  WALK    → Manager reads entire data tree                       │
│                                                                  │
│  MIB (Management Information Base):                             │
│  Tree structure of all manageable data points                   │
│  OID example: 1.3.6.1.2.1.1.1.0 = System description          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. SNMP Versions

| Version | Authentication | Encryption | Security Level |
|---------|:-------------:|:----------:|:-:|
| **SNMPv1** | Community string (cleartext) | ❌ None | ❌ Terrible |
| **SNMPv2c** | Community string (cleartext) | ❌ None | ❌ Terrible |
| **SNMPv3** | Username + password (HMAC) | ✅ AES/DES | ✅ Secure |

```
SNMPv1/v2c — Community String Problem:
Community string is a CLEARTEXT "password" sent over the network.

Default strings:
  READ:  "public"     ← Reads all device info
  WRITE: "private"    ← Can CHANGE device config!

These are sent in PLAINTEXT over UDP.
If attacker captures traffic → instant device access.
```

---

## 3. SNMP Attacks

| Attack | Description | Impact |
|--------|-------------|--------|
| **Default Community Strings** | "public"/"private" left unchanged | Full device access |
| **Sniffing** | Capture community strings from network | Credential theft |
| **SNMP Walk** | Enumerate all MIB data | Network topology, configs, users |
| **SNMP SET Abuse** | Modify device configuration | Change routes, disable interfaces |
| **SNMP Amplification** | DDoS via spoofed SNMP requests | Service disruption |

### SNMP Enumeration

```bash
# Enumerate with default community string
snmpwalk -v2c -c public target.com

# Get specific OIDs
snmpget -v2c -c public target.com 1.3.6.1.2.1.1.1.0    # System desc
snmpwalk -v2c -c public target.com 1.3.6.1.2.1.25.4.2   # Running processes

# Nmap SNMP scripts
nmap -sU -p 161 --script snmp-info target.com
nmap -sU -p 161 --script snmp-brute target.com

# Brute force community strings
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt target.com

# What SNMP reveals:
# • System info (OS, hostname, uptime)
# • Network interfaces and IPs
# • Routing tables
# • Running processes
# • Installed software
# • User accounts
# • ARP table
```

---

## 4. SNMP Security Best Practices

| Practice | Description |
|----------|-------------|
| **Use SNMPv3** | Authentication + encryption |
| **Change community strings** | Never use "public" or "private" |
| **Restrict access** | ACLs limiting which IPs can query SNMP |
| **Disable SNMP SET** | Read-only unless write is truly needed |
| **Disable SNMPv1/v2c** | If using v3, disable older versions |
| **Use separate management VLAN** | SNMP traffic only on management network |
| **Monitor SNMP** | Alert on unauthorized SNMP access |

```cisco
! Cisco — Secure SNMP configuration
! Remove default community strings
no snmp-server community public
no snmp-server community private

! Create complex community string with ACL
access-list 10 permit 10.0.99.0 0.0.0.255
snmp-server community C0mpl3xStr1ng! RO 10

! Enable SNMPv3 (best)
snmp-server group SECGROUP v3 priv
snmp-server user admin SECGROUP v3 auth sha AuthPass123 priv aes 128 PrivPass123
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SNMP** | Network device monitoring/management protocol |
| **SNMPv1/v2c** | Cleartext community strings — critically insecure |
| **SNMPv3** | Authentication + encryption — always use this |
| **Default Strings** | "public" (read) / "private" (write) — change immediately |
| **Recon Value** | SNMP walk reveals network topology, configs, users |
| **Defense** | SNMPv3 + ACLs + management VLAN |

---

## Quick Revision Questions

1. **What is SNMP and what is it used for?**
2. **Why are SNMPv1/v2c community strings a security risk?**
3. **What information can an attacker obtain from an SNMP walk?**
4. **What makes SNMPv3 more secure?**
5. **What are the default community strings and why must they be changed?**
6. **How can SNMP be used for DDoS amplification?**

---

[← Previous: SSH](05-ssh.md)

---

*Unit 5 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Network Services Security →](../06-Network-Services-Security/01-dhcp-security.md)*
