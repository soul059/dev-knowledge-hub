# Networking Fundamentals for DevOps

## Course Overview

Networking is the backbone of every modern application, cloud deployment, and DevOps pipeline. This course provides a comprehensive understanding of networking concepts essential for any DevOps engineer — from foundational models like OSI and TCP/IP, through protocols and cloud networking, to security and troubleshooting.

Whether you're configuring VPCs in AWS, debugging DNS resolution failures, or setting up load balancers for microservices, these notes will serve as both a learning guide and a quick-reference handbook.

```
┌─────────────────────────────────────────────────────────────────┐
│                  NETWORKING FOR DEVOPS                          │
│                                                                 │
│   ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│   │ Basics  │───▶│Protocols │───▶│  Cloud   │───▶│ Security │  │
│   └─────────┘    └──────────┘    └──────────┘    └──────────┘  │
│        │              │               │               │         │
│        ▼              ▼               ▼               ▼         │
│   OSI/TCP-IP     TCP/UDP/DNS     VPC/Subnets     TLS/Certs     │
│   IP/Subnets     HTTP/SSH        Gateways        Zero Trust    │
│   NAT/CIDR       DHCP/FTP       Peering          Monitoring   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- Basic understanding of Linux/Windows command line
- Familiarity with cloud platforms (AWS/Azure/GCP) is helpful but not required
- Willingness to practice with hands-on labs

---

## Complete Table of Contents

### Unit 1: Networking Basics
| # | Chapter | File |
|---|---------|------|
| 1 | OSI and TCP/IP Models | [01-osi-tcp-ip-models.md](01-Networking-Basics/01-osi-tcp-ip-models.md) |
| 2 | IP Addressing and Subnetting | [02-ip-addressing-subnetting.md](01-Networking-Basics/02-ip-addressing-subnetting.md) |
| 3 | CIDR Notation | [03-cidr-notation.md](01-Networking-Basics/03-cidr-notation.md) |
| 4 | Public vs Private IPs | [04-public-vs-private-ips.md](01-Networking-Basics/04-public-vs-private-ips.md) |
| 5 | NAT and PAT | [05-nat-and-pat.md](01-Networking-Basics/05-nat-and-pat.md) |

### Unit 2: Protocols
| # | Chapter | File |
|---|---------|------|
| 1 | TCP vs UDP | [01-tcp-vs-udp.md](02-Protocols/01-tcp-vs-udp.md) |
| 2 | HTTP/HTTPS | [02-http-https.md](02-Protocols/02-http-https.md) |
| 3 | DNS | [03-dns.md](02-Protocols/03-dns.md) |
| 4 | DHCP | [04-dhcp.md](02-Protocols/04-dhcp.md) |
| 5 | SSH | [05-ssh.md](02-Protocols/05-ssh.md) |
| 6 | FTP/SFTP | [06-ftp-sftp.md](02-Protocols/06-ftp-sftp.md) |

### Unit 3: Network Architecture
| # | Chapter | File |
|---|---------|------|
| 1 | VLANs | [01-vlans.md](03-Network-Architecture/01-vlans.md) |
| 2 | Routing Basics | [02-routing-basics.md](03-Network-Architecture/02-routing-basics.md) |
| 3 | Load Balancing Concepts | [03-load-balancing.md](03-Network-Architecture/03-load-balancing.md) |
| 4 | Firewalls and Security Groups | [04-firewalls-security-groups.md](03-Network-Architecture/04-firewalls-security-groups.md) |
| 5 | VPN Concepts | [05-vpn-concepts.md](03-Network-Architecture/05-vpn-concepts.md) |
| 6 | Proxy Servers | [06-proxy-servers.md](03-Network-Architecture/06-proxy-servers.md) |

### Unit 4: Cloud Networking
| # | Chapter | File |
|---|---------|------|
| 1 | Virtual Private Cloud (VPC) | [01-vpc.md](04-Cloud-Networking/01-vpc.md) |
| 2 | Subnets and Route Tables | [02-subnets-route-tables.md](04-Cloud-Networking/02-subnets-route-tables.md) |
| 3 | Internet and NAT Gateways | [03-internet-nat-gateways.md](04-Cloud-Networking/03-internet-nat-gateways.md) |
| 4 | Peering and Transit Gateways | [04-peering-transit-gateways.md](04-Cloud-Networking/04-peering-transit-gateways.md) |
| 5 | Network ACLs vs Security Groups | [05-nacls-vs-security-groups.md](04-Cloud-Networking/05-nacls-vs-security-groups.md) |
| 6 | Service Endpoints | [06-service-endpoints.md](04-Cloud-Networking/06-service-endpoints.md) |

### Unit 5: DNS Deep Dive
| # | Chapter | File |
|---|---------|------|
| 1 | DNS Hierarchy | [01-dns-hierarchy.md](05-DNS-Deep-Dive/01-dns-hierarchy.md) |
| 2 | Record Types (A, AAAA, CNAME, MX, TXT, NS) | [02-dns-record-types.md](05-DNS-Deep-Dive/02-dns-record-types.md) |
| 3 | DNS Resolution Process | [03-dns-resolution-process.md](05-DNS-Deep-Dive/03-dns-resolution-process.md) |
| 4 | Public vs Private DNS | [04-public-vs-private-dns.md](05-DNS-Deep-Dive/04-public-vs-private-dns.md) |
| 5 | Route 53 / Cloud DNS Basics | [05-route53-cloud-dns.md](05-DNS-Deep-Dive/05-route53-cloud-dns.md) |

### Unit 6: Network Security
| # | Chapter | File |
|---|---------|------|
| 1 | SSL/TLS Certificates | [01-ssl-tls-certificates.md](06-Network-Security/01-ssl-tls-certificates.md) |
| 2 | Certificate Authorities | [02-certificate-authorities.md](06-Network-Security/02-certificate-authorities.md) |
| 3 | HTTPS and Encryption | [03-https-encryption.md](06-Network-Security/03-https-encryption.md) |
| 4 | Network Segmentation | [04-network-segmentation.md](06-Network-Security/04-network-segmentation.md) |
| 5 | Zero Trust Networking | [05-zero-trust-networking.md](06-Network-Security/05-zero-trust-networking.md) |
| 6 | Common Vulnerabilities | [06-common-vulnerabilities.md](06-Network-Security/06-common-vulnerabilities.md) |

### Unit 7: Network Troubleshooting
| # | Chapter | File |
|---|---------|------|
| 1 | Diagnostic Tools (ping, traceroute, dig, nslookup) | [01-diagnostic-tools.md](07-Network-Troubleshooting/01-diagnostic-tools.md) |
| 2 | Packet Capture (tcpdump, Wireshark) | [02-packet-capture.md](07-Network-Troubleshooting/02-packet-capture.md) |
| 3 | Network Monitoring | [03-network-monitoring.md](07-Network-Troubleshooting/03-network-monitoring.md) |
| 4 | Common Issues and Solutions | [04-common-issues-solutions.md](07-Network-Troubleshooting/04-common-issues-solutions.md) |

---

## How to Use These Notes

1. **Sequential Learning**: Follow units in order for a structured learning path
2. **Quick Reference**: Jump to any specific topic using the table of contents
3. **Hands-On Practice**: Try every command and configuration example in a lab environment
4. **Revision**: Use the summary tables and quick revision questions at the end of each chapter
5. **Navigation**: Use Previous/Next links at the bottom of each chapter to move between topics

---

## Key Icons Used

| Icon | Meaning |
|------|---------|
| `[!]` | Important concept |
| `[>]` | Best practice |
| `[#]` | Command / Configuration |
| `[?]` | Troubleshooting tip |
| `[~]` | Real-world scenario |

---

## Recommended Lab Setup

```
┌──────────────────────────────────────────────┐
│              Recommended Lab Setup            │
├──────────────────────────────────────────────┤
│                                              │
│  Option 1: Cloud Free Tier                   │
│  ├── AWS Free Tier Account                   │
│  ├── Azure Free Account                      │
│  └── GCP Free Tier                           │
│                                              │
│  Option 2: Local Setup                       │
│  ├── VirtualBox / VMware                     │
│  ├── 2-3 Linux VMs (Ubuntu/CentOS)           │
│  ├── Docker Desktop                          │
│  └── Wireshark installed                     │
│                                              │
│  Option 3: Online Labs                       │
│  ├── KodeKloud                               │
│  ├── A Cloud Guru                            │
│  └── Katacoda (archived)                     │
│                                              │
└──────────────────────────────────────────────┘
```

---

> **Start Learning**: [Unit 1 — OSI and TCP/IP Models →](01-Networking-Basics/01-osi-tcp-ip-models.md)
