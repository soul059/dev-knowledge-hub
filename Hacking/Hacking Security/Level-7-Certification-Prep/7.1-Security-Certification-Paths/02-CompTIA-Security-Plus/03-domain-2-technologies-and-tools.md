# Unit 2: CompTIA Security+ — Topic 3: Domain 2 — Technologies and Tools

## Overview

**Domain 3 (SY0-701): Security Architecture** covers the technologies, tools, and design principles used to secure infrastructure. This includes network security devices, secure protocols, cloud concepts, and infrastructure design patterns.

---

## 1. Network Security Devices

```
SECURITY APPLIANCES:

  Device          | Function                 | Layer
  Firewall        | Filter traffic by rules  | L3-L7
  IDS             | Detect intrusions        | L3-L7
  IPS             | Prevent intrusions       | L3-L7
  WAF             | Protect web applications | L7
  Proxy           | Intermediate web access  | L7
  Load Balancer   | Distribute traffic       | L4-L7
  VPN Concentrator| Manage VPN connections   | L3
  NAC             | Control network access   | L2-L3
  SIEM            | Aggregate and analyze    | All
  UTM             | Unified threat management| All

FIREWALL TYPES:
  → Packet filtering: IP/port rules
  → Stateful inspection: Connection tracking
  → Application-aware (NGFW): Deep inspection
  → Web application firewall (WAF): HTTP
  → Implicit deny: Default block all

IDS vs IPS:
  IDS: Detect and alert (passive)
  IPS: Detect and block (inline)
  NIDS/NIPS: Network-based
  HIDS/HIPS: Host-based
  Signature-based: Known patterns
  Anomaly-based: Deviation from baseline
  Behavior-based: Suspicious activity

EXAM TIP: Know the difference between
  → IDS (passive/alert) vs IPS (active/block)
  → Signature vs anomaly detection
  → Network-based vs host-based
```

---

## 2. Secure Protocols

```
SECURE vs INSECURE PROTOCOLS:

  Insecure    | Secure      | Port
  HTTP        | HTTPS       | 80 → 443
  FTP         | SFTP/FTPS   | 21 → 22/990
  Telnet      | SSH         | 23 → 22
  SNMP v1/v2  | SNMPv3      | 161
  SMTP        | SMTPS       | 25 → 587/465
  POP3        | POP3S       | 110 → 995
  IMAP        | IMAPS       | 143 → 993
  DNS         | DNSSEC/DoH  | 53 → 443
  LDAP        | LDAPS       | 389 → 636
  RDP         | RDP+TLS     | 3389

KEY PORTS TO MEMORIZE:
  Port  | Protocol
  22    | SSH, SFTP
  25    | SMTP
  53    | DNS
  80    | HTTP
  88    | Kerberos
  110   | POP3
  143   | IMAP
  389   | LDAP
  443   | HTTPS
  445   | SMB
  636   | LDAPS
  993   | IMAPS
  995   | POP3S
  1433  | MS SQL
  3306  | MySQL
  3389  | RDP
  8080  | HTTP alternate

VPN PROTOCOLS:
  → IPSec: Network layer VPN
  → SSL/TLS VPN: Application layer
  → WireGuard: Modern, lightweight
  → L2TP/IPSec: Combined protocol
  → Split tunnel vs full tunnel
```

---

## 3. Security Tools

```
SECURITY TOOLS TO KNOW:

NETWORK TOOLS:
  → Wireshark: Packet capture and analysis
  → tcpdump: CLI packet capture
  → Nmap: Port scanning
  → Netstat: Network connections
  → Traceroute: Network path
  → Nslookup/dig: DNS queries
  → Ping: Connectivity test
  → ARP: Address resolution

VULNERABILITY TOOLS:
  → Nessus: Vulnerability scanner
  → OpenVAS: Open-source scanner
  → Qualys: Cloud-based scanner
  → Nikto: Web server scanner
  → OWASP ZAP: Web app scanner

SIEM/LOG TOOLS:
  → Splunk: Log analysis
  → ELK Stack: Open-source SIEM
  → QRadar: IBM SIEM
  → Sentinel: Microsoft SIEM

FORENSIC TOOLS:
  → FTK Imager: Disk imaging
  → Autopsy: Forensic analysis
  → Volatility: Memory analysis
  → dd: Disk copying

COMMAND-LINE TOOLS (exam PBQs):
  → ipconfig/ifconfig: Network config
  → netstat -an: Active connections
  → nslookup: DNS lookup
  → ping: Connectivity
  → tracert/traceroute: Path trace
  → arp -a: ARP table
  → nmap -sV: Service scan
  → chmod: Linux permissions
  → openssl: Certificate operations
```

---

## 4. Cloud Security Concepts

```
CLOUD MODELS:

SERVICE MODELS:
  Model | You Manage          | Provider Manages
  IaaS  | OS, apps, data      | Hardware, network
  PaaS  | Apps, data          | OS, runtime, hardware
  SaaS  | Data (partial)      | Everything
  FaaS  | Code only           | Everything else

DEPLOYMENT MODELS:
  → Public: Shared infrastructure (AWS, Azure)
  → Private: Dedicated to one org
  → Hybrid: Mix of public and private
  → Community: Shared by similar orgs
  → Multi-cloud: Multiple providers

CLOUD SECURITY:
  → Shared responsibility model
  → Cloud access security broker (CASB)
  → Cloud security posture management (CSPM)
  → Cloud workload protection (CWPP)
  → Cloud-native security controls
  → Data sovereignty considerations
  → Right to audit clauses

EXAM TIP: Know the shared responsibility model:
  → IaaS: Customer secures OS and above
  → PaaS: Customer secures apps and data
  → SaaS: Customer secures data and access
  → Provider: Always secures infrastructure
```

---

## 5. Infrastructure Security

```
SECURE INFRASTRUCTURE:

NETWORK DESIGN:
  → Defense in depth (multiple layers)
  → DMZ (screened subnet)
  → Network segmentation
  → Microsegmentation
  → Zero trust architecture
  → Air gap (complete isolation)
  → Jump server/bastion host

VIRTUALIZATION SECURITY:
  → VM escape (critical threat)
  → VM sprawl management
  → Hypervisor security
  → Container security
  → Serverless security
  → Infrastructure as Code (IaC)

EMBEDDED/IoT SECURITY:
  → Default credential changes
  → Firmware updates
  → Network isolation
  → SCADA/ICS security
  → RTOS security
  → Constraint-based limitations

PHYSICAL SECURITY:
  → Bollards, fencing, gates
  → Access control vestibule (mantrap)
  → Video surveillance (CCTV)
  → Guards, visitor logs
  → Biometrics
  → Locks (electronic, physical)
  → Environmental controls (HVAC, fire)
  → Faraday cage (EMI shielding)
```

---

## Summary Table

| Topic | Key Concepts | Exam Focus |
|-------|-------------|-----------|
| Security Devices | Firewall, IDS/IPS, SIEM | Device functions |
| Secure Protocols | HTTPS, SSH, SFTP | Secure alternatives |
| Security Tools | Wireshark, Nmap, Nessus | Tool purposes |
| Cloud Security | Models, shared responsibility | Cloud concepts |
| Infrastructure | Defense in depth, zero trust | Design principles |

---

## Revision Questions

1. What is the difference between IDS and IPS?
2. What secure protocol replaces Telnet?
3. What is the shared responsibility model in cloud?
4. What command-line tools should you know for PBQs?
5. What is the purpose of a screened subnet (DMZ)?

---

*Previous: [02-domain-1-threats-attacks-vulnerabilities.md](02-domain-1-threats-attacks-vulnerabilities.md) | Next: [04-domain-3-architecture-and-design.md](04-domain-3-architecture-and-design.md)*

---

*[Back to README](../README.md)*
