# Network Configuration Files

## Unit 6 - Topic 1 | Linux Network Security

---

## Overview

Linux network configuration is managed through a set of **configuration files** and **command-line tools**. Understanding these files is essential for security professionals because misconfigurations can expose services, enable attacks, or break network isolation. These files are also forensic artifacts during incident response.

---

## 1. Key Network Configuration Files

| File | Purpose | Security Relevance |
|------|---------|-------------------|
| `/etc/hostname` | System hostname | Info disclosure |
| `/etc/hosts` | Static hostname-to-IP mapping | Local DNS poisoning |
| `/etc/resolv.conf` | DNS resolver configuration | DNS hijacking |
| `/etc/nsswitch.conf` | Name service lookup order | Lookup manipulation |
| `/etc/network/interfaces` | Network interfaces (Debian) | Interface config |
| `/etc/netplan/*.yaml` | Network config (Ubuntu 18+) | Interface config |
| `/etc/sysconfig/network-scripts/` | Network config (RHEL) | Interface config |
| `/etc/hosts.allow` | TCP wrappers allow | Access control |
| `/etc/hosts.deny` | TCP wrappers deny | Access control |

---

## 2. Important Files in Detail

```bash
# /etc/hosts — Static DNS entries
# Attackers may modify to redirect traffic
127.0.0.1   localhost
10.0.0.5    webserver.internal
# ⚠️ ATTACK: Adding "10.0.0.99  bank.com" redirects bank traffic!
# DEFENSE: chattr +i /etc/hosts

# /etc/resolv.conf — DNS server configuration
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com
# ⚠️ ATTACK: Changing nameserver to attacker's DNS
# DEFENSE: chattr +i /etc/resolv.conf or use systemd-resolved

# /etc/nsswitch.conf — Name resolution order
hosts: files dns mDNS4_minimal
# files = /etc/hosts first, then dns
# ⚠️ If "files" is first, /etc/hosts overrides DNS

# /etc/hostname — System hostname
webserver01
# Used for identification, logging, prompt
```

---

## 3. Network Management Commands

```bash
# === VIEW CONFIGURATION ===
ip addr show                         # IP addresses
ip route show                        # Routing table
ip link show                         # Network interfaces
ip neigh show                        # ARP table (neighbors)

# === LEGACY COMMANDS (still common) ===
ifconfig                             # Interface config
route -n                             # Routing table
arp -a                               # ARP cache
netstat -tlnp                        # Listening ports

# === DNS ===
cat /etc/resolv.conf                 # DNS servers
systemd-resolve --status             # systemd-resolved status
dig example.com                      # DNS lookup
nslookup example.com                 # DNS lookup (simpler)
host example.com                     # DNS lookup

# === CONNECTIONS ===
ss -tlnp                             # TCP listening (with process)
ss -ulnp                             # UDP listening
ss -anp                              # All connections
ss -s                                # Connection statistics

# === WIRELESS ===
iwconfig                             # Wireless interface config
iw dev wlan0 info                    # Wireless details
```

---

## 4. Security Audit of Network Config

```bash
# CHECK 1: What's listening?
ss -tlnp                             # Only expected services?

# CHECK 2: DNS configuration
cat /etc/resolv.conf                 # Trusted DNS servers?

# CHECK 3: Hosts file
cat /etc/hosts                       # Any suspicious entries?

# CHECK 4: Routing table
ip route show                        # Any unexpected routes?

# CHECK 5: ARP table
ip neigh show                        # Any duplicate IPs (ARP spoof)?

# CHECK 6: Network interfaces
ip addr show                         # Any unexpected interfaces? (tun, tap)

# CHECK 7: Active connections
ss -anp | grep ESTABLISHED           # Who is connected?
```

---

## 5. NetworkManager vs systemd-networkd

| Feature | NetworkManager | systemd-networkd |
|---------|:---:|:---:|
| **Use case** | Desktop / laptop | Server |
| **GUI** | ✅ | ❌ |
| **Wi-Fi** | ✅ | Limited |
| **Config tool** | `nmcli`, `nmtui` | `.network` files |
| **VPN** | Built-in support | Manual |

```bash
# nmcli examples
nmcli device status                  # Show devices
nmcli connection show                # Show connections
nmcli connection modify eth0 ipv4.dns "8.8.8.8 8.8.4.4"
```

---

## Summary Table

| File | Purpose |
|------|---------|
| `/etc/hosts` | Static DNS — protect with immutable flag |
| `/etc/resolv.conf` | DNS servers — verify trusted |
| `/etc/nsswitch.conf` | Name resolution order |
| `ss -tlnp` | Show all listening services |
| `ip addr` | Show interface configuration |
| `ip route` | Show routing table |

---

## Quick Revision Questions

1. **What is `/etc/resolv.conf` and why is it a security concern?**
2. **How can an attacker abuse `/etc/hosts`?**
3. **What command shows all listening network services?**
4. **What is the difference between NetworkManager and systemd-networkd?**
5. **How do you view the ARP table on Linux?**

---

[Next: Netfilter and iptables →](02-netfilter-and-iptables.md)

---

*Unit 6 - Topic 1 of 5 | [Back to README](../README.md)*
