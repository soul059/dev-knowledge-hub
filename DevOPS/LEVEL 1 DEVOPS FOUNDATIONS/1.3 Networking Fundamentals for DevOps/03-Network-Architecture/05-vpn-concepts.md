# Chapter 5: VPN Concepts

## Overview

A **Virtual Private Network (VPN)** creates an encrypted tunnel between two network endpoints over the public internet, enabling secure communication as if both sides were on the same private network. In DevOps, VPNs connect on-premise data centers to cloud VPCs, link developers securely to private resources, and enable multi-cloud connectivity.

---

## 5.1 VPN Types

```
┌──────────────────────────────────────────────────────────────┐
│                    VPN TYPES                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. SITE-TO-SITE VPN                                         │
│  ┌──────────────┐      Encrypted Tunnel     ┌─────────────┐ │
│  │ On-Premise   │ ══════════════════════════ │  AWS VPC    │ │
│  │ Data Center  │      via IPSec             │  10.0.0.0/16│ │
│  │ 192.168.0/16 │                            │             │ │
│  └──────────────┘                            └─────────────┘ │
│  - Connects two networks permanently                         │
│  - Always-on connection                                      │
│  - Uses IPSec protocol                                       │
│                                                              │
│  2. CLIENT-TO-SITE (Remote Access) VPN                       │
│  ┌──────────┐         Encrypted Tunnel      ┌─────────────┐ │
│  │ Developer│ ══════════════════════════════ │  Corporate  │ │
│  │ Laptop   │     via OpenVPN / WireGuard    │  Network    │ │
│  └──────────┘                                └─────────────┘ │
│  - Individual device connects to a network                   │
│  - On-demand connection                                      │
│  - Uses SSL/TLS or IPSec                                     │
│                                                              │
│  3. SITE-TO-SITE (MULTI-CLOUD)                               │
│  ┌──────────┐     Encrypted      ┌──────────┐               │
│  │ AWS VPC  │ ════════════════════│ Azure    │               │
│  │          │     Tunnel          │  VNet    │               │
│  └──────────┘                     └──────────┘               │
│  - Connects different cloud providers                        │
│  - Uses VPN gateways on both ends                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 VPN Protocols

| Protocol | Port | Encryption | Speed | Use Case |
|----------|------|------------|-------|----------|
| IPSec | UDP 500, 4500 | AES-256 | Good | Site-to-Site standard |
| OpenVPN | TCP 443 / UDP 1194 | TLS + AES | Good | Remote access |
| WireGuard | UDP 51820 | ChaCha20 | Excellent | Modern alternative |
| L2TP/IPSec | UDP 1701 | IPSec | Moderate | Legacy systems |
| SSTP | TCP 443 | SSL/TLS | Moderate | Windows environments |
| IKEv2/IPSec | UDP 500, 4500 | AES-256 | Good | Mobile-friendly |

---

## 5.3 AWS Site-to-Site VPN

```
┌──────────────────────────────────────────────────────────────┐
│              AWS SITE-TO-SITE VPN ARCHITECTURE               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  On-Premise                          AWS Cloud               │
│  ┌─────────────┐                     ┌────────────────────┐  │
│  │             │                     │  VPC 10.0.0.0/16   │  │
│  │  Corporate  │    Tunnel 1 (active)│  ┌──────────────┐  │  │
│  │  Network    │ ═════════════════════▶│ Virtual       │  │  │
│  │             │    Tunnel 2 (standby)│ │ Private      │  │  │
│  │ 192.168.0/16│ ═════════════════════▶│ Gateway (VGW)│  │  │
│  │             │                     │  └──────┬───────┘  │  │
│  │  ┌────────┐ │                     │         │          │  │
│  │  │Customer│ │                     │  ┌──────▼───────┐  │  │
│  │  │Gateway │ │                     │  │ Route Table  │  │  │
│  │  │(Router)│ │                     │  │ 192.168.0/16 │  │  │
│  │  └────────┘ │                     │  │  → VGW       │  │  │
│  └─────────────┘                     │  └──────────────┘  │  │
│                                      └────────────────────┘  │
│  Key Components:                                             │
│  - Customer Gateway: Your on-prem router                     │
│  - VPN Gateway (VGW): AWS-side endpoint                      │
│  - 2 tunnels for high availability                           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Terraform: AWS Site-to-Site VPN

```hcl
# Customer Gateway (your on-prem router)
resource "aws_customer_gateway" "main" {
  bgp_asn    = 65000
  ip_address = "203.0.113.1"    # Your public IP
  type       = "ipsec.1"
  tags       = { Name = "on-prem-gateway" }
}

# Virtual Private Gateway (AWS side)
resource "aws_vpn_gateway" "main" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "vpn-gateway" }
}

# VPN Connection
resource "aws_vpn_connection" "main" {
  vpn_gateway_id      = aws_vpn_gateway.main.id
  customer_gateway_id = aws_customer_gateway.main.id
  type                = "ipsec.1"
  static_routes_only  = true
  tags                = { Name = "on-prem-to-aws" }
}

# Static route to on-prem network
resource "aws_vpn_connection_route" "on_prem" {
  vpn_connection_id      = aws_vpn_connection.main.id
  destination_cidr_block = "192.168.0.0/16"
}

# Route table entry pointing to VGW
resource "aws_route" "to_onprem" {
  route_table_id         = aws_route_table.private.id
  destination_cidr_block = "192.168.0.0/16"
  gateway_id             = aws_vpn_gateway.main.id
}
```

---

## 5.4 AWS Client VPN

```
┌──────────────────────────────────────────────────────────────┐
│             AWS CLIENT VPN                                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐        TLS Tunnel       ┌────────────────────┐ │
│  │ Developer│ ════════════════════════▶│ Client VPN         │ │
│  │ Laptop   │   (OpenVPN protocol)    │ Endpoint           │ │
│  │          │                         │ ┌────────────────┐ │ │
│  │ OpenVPN  │                         │ │ ENI in Subnet  │ │ │
│  │ Client   │                         │ │ 10.0.1.0/24    │ │ │
│  └──────────┘                         │ └───────┬────────┘ │ │
│                                       │         │          │ │
│       Assigned IP:                    │    ┌────▼────┐     │ │
│       10.100.0.x (from VPN CIDR)      │    │  VPC   │     │ │
│                                       │    │Resources│     │ │
│                                       │    └─────────┘     │ │
│                                       └────────────────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.5 VPN vs Direct Connect vs Transit Gateway

| Feature | VPN | Direct Connect | Transit Gateway |
|---------|-----|---------------|-----------------|
| Connection | Over internet | Dedicated fiber | Hub-and-spoke |
| Encryption | Yes (IPSec) | Not by default | VPN attachments |
| Bandwidth | Up to 1.25 Gbps | Up to 100 Gbps | Aggregates connections |
| Setup time | Minutes | Weeks/months | Hours |
| Cost | Low | High | Medium |
| Reliability | Internet-dependent | SLA-backed | High availability |
| Best for | Dev/test, backup | Production, large data | Multi-VPC connectivity |

---

## 5.6 WireGuard Quick Setup (DevOps Favorite)

```bash
# Install WireGuard
sudo apt install wireguard

# Generate keys
wg genkey | tee privatekey | wg pubkey > publickey

# Server config (/etc/wireguard/wg0.conf)
[Interface]
PrivateKey = <server-private-key>
Address = 10.100.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.100.0.2/32

# Client config
[Interface]
PrivateKey = <client-private-key>
Address = 10.100.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <server-public-key>
Endpoint = vpn.example.com:51820
AllowedIPs = 10.0.0.0/8
PersistentKeepalive = 25

# Start VPN
sudo wg-quick up wg0
sudo wg show     # Check status
```

---

## 5.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| VPN tunnel not establishing | Mismatched pre-shared keys | Verify keys on both ends |
| Tunnel up but no traffic | Missing route table entries | Add routes to VPN gateway |
| Intermittent disconnects | NAT traversal issues | Enable NAT-T (UDP 4500) |
| Slow VPN performance | Internet bandwidth limitation | Consider Direct Connect |
| Can't reach specific subnet | Split tunneling misconfigured | Check AllowedIPs / routing |
| Phase 2 negotiation failure | Mismatched encryption settings | Align IKE/IPSec parameters |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Site-to-Site VPN | Connects two networks permanently over IPSec |
| Client VPN | Individual device connects to a network |
| IPSec | Standard VPN protocol, UDP 500/4500 |
| WireGuard | Modern, fast, simple VPN protocol |
| AWS VGW | Virtual Private Gateway — AWS VPN endpoint |
| Customer Gateway | On-premise router endpoint for VPN |
| 2 Tunnels | AWS provides 2 tunnels for HA |
| Transit Gateway | Hub-and-spoke for connecting multiple VPCs + VPN |

---

## Quick Revision Questions

1. **What are the two main types of VPN and when would you use each?**
2. **Why does AWS create two VPN tunnels for a site-to-site connection?**
3. **Compare WireGuard and OpenVPN. When would you choose each?**
4. **What AWS components are needed to set up a site-to-site VPN?**
5. **When would you choose Direct Connect over VPN?**
6. **A VPN tunnel is established but traffic isn't flowing. What's the most likely cause?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Firewalls & Security Groups](04-firewalls-security-groups.md) | [README](../README.md) | [Proxy Servers →](06-proxy-servers.md) |
