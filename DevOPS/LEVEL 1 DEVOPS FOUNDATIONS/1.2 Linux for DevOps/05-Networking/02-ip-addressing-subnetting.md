# Chapter 2: IP Addressing & Subnetting

[← Previous: Networking Fundamentals](01-networking-fundamentals.md) | [Back to README](../README.md) | [Next: Network Commands →](03-network-commands.md)

---

## Overview

Understanding IP addressing and subnetting is essential for configuring servers, VPCs, container networks, and Kubernetes pod CIDRs. This chapter covers IPv4, CIDR notation, subnet calculation, and private vs public address ranges.

---

## 2.1 IPv4 Address Structure

```
┌──────────────────────────────────────────────────────────────┐
│                  IPv4 ADDRESS FORMAT                          │
│                                                                │
│  Dotted decimal:  192.168.1.100                               │
│  Binary:          11000000.10101000.00000001.01100100         │
│                                                                │
│  4 octets × 8 bits = 32 bits total                            │
│  Each octet: 0 - 255                                          │
│                                                                │
│  Network Part        │  Host Part                              │
│  ┌───────────────────┼──────────────┐                         │
│  │ 192.168.1         │ .100         │                         │
│  └───────────────────┴──────────────┘                         │
│  (determined by subnet mask)                                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 Subnet Mask & CIDR Notation

```
┌──────────────────────────────────────────────────────────────┐
│              SUBNET MASK REFERENCE                            │
├──────────┬──────────────────┬────────┬──────────────────────┤
│ CIDR     │ Subnet Mask      │ Hosts  │ Common Use           │
├──────────┼──────────────────┼────────┼──────────────────────┤
│ /8       │ 255.0.0.0        │ 16.7M  │ Class A (10.0.0.0)  │
│ /16      │ 255.255.0.0      │ 65,534 │ Class B, VPC default │
│ /20      │ 255.255.240.0    │ 4,094  │ AWS subnet           │
│ /24      │ 255.255.255.0    │ 254    │ Most common LAN      │
│ /25      │ 255.255.255.128  │ 126    │ Split /24 in half    │
│ /26      │ 255.255.255.192  │ 62     │ Small subnet         │
│ /27      │ 255.255.255.224  │ 30     │ Small subnet         │
│ /28      │ 255.255.255.240  │ 14     │ Very small           │
│ /30      │ 255.255.255.252  │ 2      │ Point-to-point link  │
│ /32      │ 255.255.255.255  │ 1      │ Single host          │
└──────────┴──────────────────┴────────┴──────────────────────┘

  Formula: Usable hosts = 2^(32 - CIDR) - 2
  (subtract network address and broadcast address)
```

---

## 2.3 Private vs Public IP Ranges

```
┌──────────────────────────────────────────────────────────────┐
│            PRIVATE IP RANGES (RFC 1918)                       │
├───────────┬────────────────────────┬────────┬────────────────┤
│ Class     │ Range                  │ CIDR   │ Usage          │
├───────────┼────────────────────────┼────────┼────────────────┤
│ A         │ 10.0.0.0 - 10.255.    │ /8     │ Large corps,   │
│           │ 255.255                │        │ cloud VPCs     │
├───────────┼────────────────────────┼────────┼────────────────┤
│ B         │ 172.16.0.0 - 172.31.  │ /12    │ Docker default │
│           │ 255.255                │        │ (172.17.0.0/16)│
├───────────┼────────────────────────┼────────┼────────────────┤
│ C         │ 192.168.0.0 - 192.168.│ /16    │ Home networks, │
│           │ 255.255                │        │ small LANs     │
├───────────┼────────────────────────┼────────┼────────────────┤
│ Loopback  │ 127.0.0.0 - 127.255.  │ /8     │ localhost      │
│           │ 255.255                │        │                │
├───────────┼────────────────────────┼────────┼────────────────┤
│ Link-local│ 169.254.0.0 - 169.254.│ /16    │ APIPA (no DHCP)│
│           │ 255.255                │        │                │
└───────────┴────────────────────────┴────────┴────────────────┘
```

---

## 2.4 Subnetting Example

```
┌──────────────────────────────────────────────────────────────┐
│  Example: Subnet 192.168.1.0/26                               │
│                                                                │
│  Subnet mask: 255.255.255.192                                 │
│  Network bits: 26   Host bits: 6                              │
│  Hosts per subnet: 2^6 - 2 = 62                              │
│  Number of subnets from /24: 2^(26-24) = 4                   │
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │ Subnet 1: 192.168.1.0/26                                │  │
│  │   Network: 192.168.1.0    Broadcast: 192.168.1.63       │  │
│  │   Usable:  192.168.1.1  - 192.168.1.62                  │  │
│  ├─────────────────────────────────────────────────────────┤  │
│  │ Subnet 2: 192.168.1.64/26                               │  │
│  │   Network: 192.168.1.64   Broadcast: 192.168.1.127      │  │
│  │   Usable:  192.168.1.65 - 192.168.1.126                 │  │
│  ├─────────────────────────────────────────────────────────┤  │
│  │ Subnet 3: 192.168.1.128/26                              │  │
│  │   Network: 192.168.1.128  Broadcast: 192.168.1.191      │  │
│  │   Usable:  192.168.1.129- 192.168.1.190                 │  │
│  ├─────────────────────────────────────────────────────────┤  │
│  │ Subnet 4: 192.168.1.192/26                              │  │
│  │   Network: 192.168.1.192  Broadcast: 192.168.1.255      │  │
│  │   Usable:  192.168.1.193- 192.168.1.254                 │  │
│  └─────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.5 IP Configuration in Linux

```bash
# View current IP configuration
ip addr show
ip -4 addr show              # IPv4 only
ip addr show dev eth0        # Specific interface

# Assign IP address (temporary)
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0

# Set default gateway
sudo ip route add default via 192.168.1.1
ip route show                # View routing table

# Persistent configuration (Netplan — Ubuntu 18.04+)
# /etc/netplan/01-config.yaml
```

```yaml
# Ubuntu Netplan (Static IP)
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    ens33:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

```bash
# Apply Netplan
sudo netplan apply
sudo netplan try              # Apply with auto-rollback

# RHEL/CentOS (nmcli)
sudo nmcli con mod ens33 ipv4.addresses 192.168.1.100/24
sudo nmcli con mod ens33 ipv4.gateway 192.168.1.1
sudo nmcli con mod ens33 ipv4.dns "8.8.8.8 8.8.4.4"
sudo nmcli con mod ens33 ipv4.method manual
sudo nmcli con up ens33
```

---

## 2.6 DevOps Networking Context

```
┌──────────────────────────────────────────────────────────────┐
│            COMMON CIDR ALLOCATIONS IN DEVOPS                  │
│                                                                │
│  AWS VPC:          10.0.0.0/16      (65,534 IPs)             │
│  ├── Public  Sub:  10.0.1.0/24      (254 IPs)               │
│  ├── Private Sub:  10.0.2.0/24      (254 IPs)               │
│  └── DB Sub:       10.0.3.0/24      (254 IPs)               │
│                                                                │
│  Docker default:   172.17.0.0/16                              │
│  ├── Bridge:       172.17.0.1       (gateway)                │
│  └── Containers:   172.17.0.2+                                │
│                                                                │
│  Kubernetes:                                                   │
│  ├── Pod CIDR:     10.244.0.0/16    (Flannel default)        │
│  ├── Service CIDR: 10.96.0.0/12                              │
│  └── Node network: 192.168.1.0/24                            │
│                                                                │
│  Overlay networks: Separate address space per network         │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| No IP assigned | `ip addr show` — no inet | Check DHCP or configure static IP |
| Wrong subnet | Cannot reach other hosts | Verify subnet mask matches gateway |
| IP conflict | Intermittent connectivity | Check ARP: `arping -D IP` |
| Netplan error | `netplan apply` fails | Validate YAML indentation, run `netplan try` |
| Gateway unreachable | `ip route` shows no default | Add gateway: `ip route add default via GW` |
| Docker conflicts | Overlapping subnets | Change Docker's `bip` in `/etc/docker/daemon.json` |

---

## Summary Table

| Concept | Key Info |
|---------|----------|
| IPv4 Format | 4 octets, 32 bits (e.g., 192.168.1.100) |
| CIDR /24 | 255.255.255.0 = 254 hosts |
| Private ranges | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 |
| Netplan | Ubuntu persistent network config |
| nmcli | RHEL/CentOS network config |
| Docker default | 172.17.0.0/16 bridge network |

---

## Quick Revision Questions

1. **How many usable hosts are in a /24 subnet? What about /26?**
2. **What are the three RFC 1918 private IP ranges?**
3. **Given 10.0.0.0/20, what is the subnet mask and how many hosts?**
4. **How do you configure a static IP on Ubuntu using Netplan?**
5. **Why might Docker container subnets conflict with your corporate network?**
6. **What is the difference between `ip addr add` (temporary) and Netplan (persistent)?**

---

[← Previous: Networking Fundamentals](01-networking-fundamentals.md) | [Back to README](../README.md) | [Next: Network Commands →](03-network-commands.md)
