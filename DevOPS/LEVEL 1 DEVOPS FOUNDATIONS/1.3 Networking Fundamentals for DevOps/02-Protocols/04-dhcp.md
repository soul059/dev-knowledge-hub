# Chapter 4: DHCP

## Overview

DHCP (Dynamic Host Configuration Protocol) automatically assigns IP addresses and network configuration to devices on a network. Without DHCP, every device would need manual IP configuration. In cloud environments, DHCP runs silently behind the scenes — when an EC2 instance boots, it gets its IP from the VPC's built-in DHCP service.

---

## 4.1 What DHCP Assigns

```
┌──────────────────────────────────────────────────────────────┐
│              DHCP CONFIGURATION PARAMETERS                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  When a device gets a DHCP lease, it receives:               │
│                                                              │
│  ┌──────────────────────────────────────────┐                │
│  │  IP Address:       10.0.1.50             │                │
│  │  Subnet Mask:      255.255.255.0 (/24)   │                │
│  │  Default Gateway:  10.0.1.1              │                │
│  │  DNS Server(s):    10.0.0.2, 8.8.8.8    │                │
│  │  Lease Duration:   86400 seconds (24h)   │                │
│  │  Domain Name:      internal.company.com  │                │
│  │  NTP Server:       10.0.0.10             │                │
│  └──────────────────────────────────────────┘                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 DHCP DORA Process

```
┌──────────────────────────────────────────────────────────────┐
│              DHCP DORA PROCESS                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client                                 DHCP Server          │
│  (no IP yet)                            (10.0.1.1)           │
│    │                                       │                 │
│    │── D: DISCOVER (broadcast) ──────────▶│  "Any DHCP      │
│    │   src: 0.0.0.0                        │   server out    │
│    │   dst: 255.255.255.255                │   there?"       │
│    │   UDP 67/68                           │                 │
│    │                                       │                 │
│    │◀─ O: OFFER ──────────────────────────│  "Here, use     │
│    │   "IP: 10.0.1.50                      │   this IP"      │
│    │    Mask: 255.255.255.0                │                 │
│    │    GW: 10.0.1.1                       │                 │
│    │    DNS: 10.0.0.2"                     │                 │
│    │                                       │                 │
│    │── R: REQUEST (broadcast) ───────────▶│  "I'll take     │
│    │   "I accept 10.0.1.50"               │   that one"     │
│    │                                       │                 │
│    │◀─ A: ACK ────────────────────────────│  "Confirmed,    │
│    │   "10.0.1.50 is yours for 24h"        │   it's yours"   │
│    │                                       │                 │
│    │   Client configures interface         │                 │
│    │   with received settings              │                 │
│                                                              │
│  DORA = Discover → Offer → Request → Acknowledge            │
│  Protocol: UDP ports 67 (server) and 68 (client)             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.3 DHCP Lease Lifecycle

```
┌──────────────────────────────────────────────────────────────┐
│              DHCP LEASE LIFECYCLE                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ├─────────────── Lease Duration (e.g., 24 hours) ────────┤  │
│  │                                                        │  │
│  │ T1 (50% = 12h)              T2 (87.5% = 21h)          │  │
│  │      │                           │                     │  │
│  ▼      ▼                           ▼                     ▼  │
│  ┌──────┬───────────────────────────┬─────────────────┬────┐ │
│  │ NEW  │    BOUND (using IP)       │    REBINDING    │EXP │ │
│  │LEASE │                           │                 │    │ │
│  └──────┴───────────────────────────┴─────────────────┴────┘ │
│         │                           │                        │
│         At T1: Client tries         At T2: Client             │
│         to RENEW with same          broadcasts to ANY         │
│         DHCP server (unicast)       DHCP server               │
│                                                              │
│  If renewal fails and lease expires → IP released            │
│  Device gets 169.254.x.x (APIPA / link-local)               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.4 DHCP in Cloud Environments

### AWS VPC DHCP

```
┌──────────────────────────────────────────────────────────────┐
│              AWS DHCP OPTIONS SET                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  AWS runs a DHCP service automatically in every VPC.         │
│  Instances receive their private IP via DHCP at launch.      │
│                                                              │
│  Default DHCP Options:                                       │
│  ├── domain-name: ec2.internal (us-east-1)                   │
│  │                region.compute.internal (other regions)    │
│  ├── domain-name-servers: AmazonProvidedDNS (10.0.0.2)      │
│  └── ntp-servers: Amazon Time Sync (169.254.169.123)         │
│                                                              │
│  Custom DHCP Options Set:                                    │
│  ├── domain-name: mycompany.internal                         │
│  ├── domain-name-servers: 10.0.0.53, 8.8.8.8               │
│  └── ntp-servers: 10.0.0.100                                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Create custom DHCP options set
aws ec2 create-dhcp-options --dhcp-configurations \
  "Key=domain-name,Values=mycompany.internal" \
  "Key=domain-name-servers,Values=10.0.0.53,8.8.8.8"

# Associate with VPC
aws ec2 associate-dhcp-options \
  --dhcp-options-id dopt-12345678 \
  --vpc-id vpc-abcdef12
```

### Terraform Example

```hcl
resource "aws_vpc_dhcp_options" "custom" {
  domain_name         = "mycompany.internal"
  domain_name_servers = ["10.0.0.53", "8.8.8.8"]
  ntp_servers         = ["169.254.169.123"]

  tags = {
    Name = "custom-dhcp-options"
  }
}

resource "aws_vpc_dhcp_options_association" "main" {
  vpc_id          = aws_vpc.main.id
  dhcp_options_id = aws_vpc_dhcp_options.custom.id
}
```

---

## 4.5 DHCP vs Static IP

```
┌────────────────────┬───────────────────┬───────────────────┐
│  Feature           │  DHCP             │  Static IP        │
├────────────────────┼───────────────────┼───────────────────┤
│  Configuration     │  Automatic        │  Manual           │
│  Management        │  Easy at scale    │  Labor-intensive  │
│  IP conflicts      │  Avoided          │  Possible         │
│  Tracking          │  Lease database   │  Manual tracking  │
│  Suitable for      │  Workstations,    │  Servers, routers,│
│                    │  mobile, VMs      │  printers, DNS    │
│  Cloud instances   │  Default method   │  Not typical      │
│  Reliability       │  Depends on DHCP  │  Always available │
│                    │  server           │                   │
└────────────────────┴───────────────────┴───────────────────┘

Best Practice: Use DHCP with reservations for servers
(assigns same IP based on MAC address but via DHCP)
```

---

## 4.6 DHCP Relay

When the DHCP server is on a different subnet:

```
┌──────────────────────────────────────────────────────────────┐
│              DHCP RELAY AGENT                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Subnet A (10.0.1.0/24)         Subnet B (10.0.2.0/24)      │
│  ┌──────────────┐               ┌──────────────┐            │
│  │ New Device   │               │  DHCP Server │            │
│  │  (no IP)     │               │  10.0.2.10   │            │
│  └──────┬───────┘               └──────▲───────┘            │
│         │ Broadcast                     │ Unicast            │
│         ▼                              │                    │
│  ┌──────────────┐                      │                    │
│  │ Router with  │──────────────────────┘                    │
│  │ DHCP Relay   │                                           │
│  │ 10.0.1.1     │                                           │
│  └──────────────┘                                           │
│                                                              │
│  Relay converts broadcast → unicast to reach DHCP server    │
│  across subnets (broadcasts don't cross router boundaries)  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Device gets 169.254.x.x | DHCP server unreachable | Check DHCP service, network connectivity |
| IP address conflict | Duplicate assignments | Check DHCP scope, remove static conflicts |
| Wrong DNS servers | Bad DHCP options | Update DHCP options set, renew leases |
| Instance can't resolve hostnames | DHCP didn't provide DNS | Check DHCP options, /etc/resolv.conf |
| Lease expires, app disconnects | Short lease + DHCP down | Increase lease time, add redundant DHCP |

```bash
# Renew DHCP lease
# Linux
sudo dhclient -r && sudo dhclient

# Windows
ipconfig /release
ipconfig /renew

# Check current DHCP lease info
# Linux
cat /var/lib/dhcp/dhclient.leases
nmcli device show

# Windows
ipconfig /all | findstr "DHCP"
```

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| DHCP | Automatically assigns IP, mask, gateway, DNS to devices |
| DORA | Discover → Offer → Request → Acknowledge |
| Ports | UDP 67 (server), UDP 68 (client) |
| Lease | Temporary IP assignment with renewal at 50% and 87.5% |
| AWS DHCP | Built into VPC; customizable via DHCP Options Sets |
| APIPA | 169.254.x.x — indicates DHCP failure |
| Relay | Forwards DHCP broadcasts across subnets |

---

## Quick Revision Questions

1. **What does DORA stand for in DHCP, and what happens at each step?**
2. **Why does DHCP use broadcast? What protocol and ports does it use?**
3. **What happens when a DHCP lease expires and the server is unavailable?**
4. **In AWS, how do you customize DNS servers for instances in a VPC?**
5. **What is a DHCP relay agent, and when is it needed?**
6. **A newly launched EC2 instance gets a 169.254.x.x IP. What went wrong?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← DNS](03-dns.md) | [README](../README.md) | [SSH →](05-ssh.md) |
