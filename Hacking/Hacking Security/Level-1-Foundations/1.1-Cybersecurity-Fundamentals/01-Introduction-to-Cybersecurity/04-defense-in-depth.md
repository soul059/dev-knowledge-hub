# Defense in Depth

## Unit 1 - Topic 4 | Introduction to Cybersecurity

---

## Overview

**Defense in Depth** (DiD) is a cybersecurity strategy that employs multiple layers of security controls throughout an information system. If one layer fails, the next layer provides protection. It's inspired by the military strategy of the same name — making it progressively harder for an attacker to reach the target.

---

## 1. The Castle Analogy

```
┌─────────────────────────────────────────────────────────────────────────┐
│  THE CASTLE ANALOGY                                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Medieval Castle                    Modern Cybersecurity                │
│  ────────────────                   ──────────────────────              │
│  Moat                         ──►   Perimeter Firewall                  │
│  Castle Walls                 ──►   Network Security (IDS/IPS)          │
│  Guards at Gates              ──►   Access Controls (Authentication)    │
│  Locked Doors                 ──►   Endpoint Security (AV/EDR)          │
│  Safe / Vault                 ──►   Data Encryption                     │
│  Royal Guard (personal)       ──►   Application Security                │
│  Watchmen (lookouts)          ──►   Security Monitoring (SIEM)          │
│                                                                         │
│  If the moat is crossed, walls still protect. If walls are breached,   │
│  guards still defend. SAME concept in cybersecurity.                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Defense in Depth Model

```
┌────────────────────────────────────────────────────────────────────────┐
│  LAYER 1: POLICIES, PROCEDURES & AWARENESS                            │
│  ┌────────────────────────────────────────────────────────────────────┐│
│  │  LAYER 2: PHYSICAL SECURITY                                        ││
│  │  ┌────────────────────────────────────────────────────────────────┐││
│  │  │  LAYER 3: PERIMETER SECURITY                                    │││
│  │  │  ┌────────────────────────────────────────────────────────────┐│││
│  │  │  │  LAYER 4: NETWORK SECURITY                                  ││││
│  │  │  │  ┌────────────────────────────────────────────────────────┐││││
│  │  │  │  │  LAYER 5: HOST SECURITY                                │││││
│  │  │  │  │  ┌────────────────────────────────────────────────────┐│││││
│  │  │  │  │  │  LAYER 6: APPLICATION SECURITY                     ││││││
│  │  │  │  │  │  ┌────────────────────────────────────────────────┐││││││
│  │  │  │  │  │  │  LAYER 7: DATA SECURITY                        │││││││
│  │  │  │  │  │  │                                                │││││││
│  │  │  │  │  │  │          🏆 CROWN JEWELS (DATA)                │││││││
│  │  │  │  │  │  │                                                │││││││
│  │  │  │  │  │  └────────────────────────────────────────────────┘││││││
│  │  │  │  │  └────────────────────────────────────────────────────┘│││││
│  │  │  │  └────────────────────────────────────────────────────────┘││││
│  │  │  └────────────────────────────────────────────────────────────┘│││
│  │  └────────────────────────────────────────────────────────────────┘││
│  └────────────────────────────────────────────────────────────────────┘│
└────────────────────────────────────────────────────────────────────────┘
```

---

## 3. The Seven Layers Explained

### Layer 1: Policies, Procedures & Awareness

| Component | Description |
|-----------|-------------|
| **Security Policies** | Written rules governing security behavior |
| **Procedures** | Step-by-step instructions for security tasks |
| **Awareness Training** | Educating employees about threats and best practices |
| **Acceptable Use Policy** | Rules for using company resources |
| **Incident Response Plan** | Predefined steps for handling security incidents |

### Layer 2: Physical Security

| Component | Description |
|-----------|-------------|
| **Access Controls** | Badge readers, biometrics, man-traps |
| **Surveillance** | CCTV cameras, security guards |
| **Environmental** | Fire suppression, HVAC, power protection |
| **Secure Facilities** | Locked server rooms, data center cages |
| **Device Security** | Cable locks, secure disposal |

### Layer 3: Perimeter Security

| Component | Description |
|-----------|-------------|
| **Firewalls** | Filter traffic between networks (stateful, NGFW) |
| **DMZ** | Isolated zone for public-facing services |
| **VPN** | Encrypted tunnels for remote access |
| **Email Gateway** | Filter spam, phishing, malicious attachments |
| **Web Proxy** | Control and monitor web access |

### Layer 4: Network Security

| Component | Description |
|-----------|-------------|
| **IDS/IPS** | Detect and prevent network intrusions |
| **Network Segmentation** | Divide network into isolated zones |
| **VLANs** | Logical separation of network traffic |
| **NAC** | Network Access Control — verify before granting access |
| **Encryption** | TLS/SSL for data in transit |

### Layer 5: Host Security

| Component | Description |
|-----------|-------------|
| **Antivirus/EDR** | Detect and respond to malware on endpoints |
| **Patch Management** | Keep systems updated with security patches |
| **Host Firewall** | OS-level firewall (Windows Firewall, iptables) |
| **Hardening** | Remove unnecessary services, apply baselines |
| **HIDS** | Host-based Intrusion Detection System |

### Layer 6: Application Security

| Component | Description |
|-----------|-------------|
| **Secure Coding** | Follow secure development practices (OWASP) |
| **Input Validation** | Sanitize all user inputs |
| **WAF** | Web Application Firewall |
| **Code Review** | Peer review and automated scanning (SAST/DAST) |
| **Authentication** | Strong auth mechanisms in applications |

### Layer 7: Data Security

| Component | Description |
|-----------|-------------|
| **Encryption at Rest** | AES-256 for stored data |
| **Encryption in Transit** | TLS 1.3 for data moving across networks |
| **DLP** | Data Loss Prevention tools |
| **Backup & Recovery** | Regular backups with tested restoration |
| **Data Classification** | Label data by sensitivity level |
| **Access Controls** | Granular permissions to data resources |

---

## 4. Attack Scenario — Without vs With Defense in Depth

### Without Defense in Depth (Single Layer)

```
Attacker ──► [Firewall] ──► 💀 Direct access to data!

If the firewall is bypassed (misconfiguration, zero-day),
there is NOTHING else protecting the assets.
```

### With Defense in Depth (Multiple Layers)

```
Attacker
   │
   ├──► [Firewall] ──► Bypassed via misconfigured rule
   │         │
   │         ├──► [IDS/IPS] ──► Alert triggered, but attacker evades
   │         │         │
   │         │         ├──► [Network Segmentation] ──► Blocked from sensitive VLAN
   │         │         │              │
   │         │         │              ├──► [Host EDR] ──► Malware detected & quarantined ✅
   │         │         │              │
   │         │         │              └──► ATTACK STOPPED at Layer 5
   │         │         │
   │         │         └── Even if host is compromised:
   │         │              └──► [Data Encryption] ──► Data unreadable without keys 🔒
   │         │
   │         └── SIEM correlates events ──► SOC team alerted 🚨
   │
   └── Multiple layers = Multiple chances to DETECT and STOP
```

---

## 5. Defense in Depth — Control Types

```
┌──────────────────────────────────────────────────────────────┐
│                    CONTROL CATEGORIES                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PREVENTIVE ──► Stop attacks before they happen              │
│  (Firewall, MFA, encryption, access controls)                │
│                                                              │
│  DETECTIVE  ──► Identify attacks in progress                 │
│  (IDS, SIEM, log monitoring, anomaly detection)              │
│                                                              │
│  CORRECTIVE ──► Fix issues after detection                   │
│  (Patch management, incident response, backup restore)       │
│                                                              │
│  DETERRENT  ──► Discourage attackers                         │
│  (Warning banners, security cameras, penalties)              │
│                                                              │
│  COMPENSATING ──► Alternative when primary control fails     │
│  (Manual review when automated system is down)               │
│                                                              │
│  RECOVERY   ──► Restore normal operations                    │
│  (Disaster recovery, backup restoration, failover)           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Control Implementation Matrix

| Layer | Preventive | Detective | Corrective |
|-------|-----------|-----------|------------|
| **Physical** | Locks, fences | Cameras, alarms | Repair, replace |
| **Network** | Firewalls, ACLs | IDS/IPS, NetFlow | Reconfig, block IPs |
| **Host** | Hardening, AV | EDR, HIDS | Patch, reimage |
| **Application** | Input validation | WAF logs, DAST | Code fix, deploy patch |
| **Data** | Encryption, DLP | Audit logs, FIM | Restore from backup |
| **People** | Training, policies | Phishing tests | Disciplinary action |

---

## 6. Real-World Design Example

### Corporate Network Defense in Depth

```
                        INTERNET
                           │
                    ┌──────┴──────┐
                    │  Edge       │ ← Layer 3: Perimeter
                    │  Firewall   │   (Stateful + NGFW)
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │    DMZ      │ ← Public-facing servers
                    │  Web/Mail   │   (Isolated zone)
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │  Internal   │ ← Layer 4: Network
                    │  Firewall   │   (Segmentation)
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
        ┌─────┴─────┐ ┌───┴───┐ ┌─────┴─────┐
        │ User VLAN │ │Server │ │  Admin     │ ← Segmented
        │ (10.1.x)  │ │ VLAN  │ │  VLAN      │   VLANs
        └─────┬─────┘ │(10.2) │ │ (10.3)     │
              │        └───┬───┘ └─────┬─────┘
              │            │           │
        ┌─────┴─────┐ ┌───┴───┐ ┌─────┴─────┐
        │ Endpoints │ │  DB   │ │  DC/SIEM   │ ← Layer 5: Host
        │  w/ EDR   │ │Server │ │  Servers   │   (EDR, AV)
        └───────────┘ │w/Encr │ └───────────┘
                      └───────┘
                      ← Layer 7: Data (Encrypted at rest)
```

---

## Summary Table

| Layer | What It Protects | Key Controls |
|-------|-----------------|-------------|
| **Policies & Awareness** | Human behavior | Training, policies, procedures |
| **Physical** | Facilities & hardware | Locks, cameras, guards |
| **Perimeter** | Network boundary | Firewalls, DMZ, VPN |
| **Network** | Internal network | IDS/IPS, segmentation, NAC |
| **Host** | Individual systems | AV/EDR, patching, hardening |
| **Application** | Software | Secure coding, WAF, code review |
| **Data** | Information assets | Encryption, DLP, backups, classification |

---

## Quick Revision Questions

1. **What is the core idea behind Defense in Depth?**
2. **List all seven layers of Defense in Depth with one example control each.**
3. **Why is a single firewall not sufficient security?**
4. **What's the difference between preventive, detective, and corrective controls?**
5. **Draw a basic Defense in Depth diagram for a small business network.**
6. **How does network segmentation contribute to Defense in Depth?**

---

[← Previous: AAA Framework](03-aaa-framework.md) | [Next: Security vs Convenience →](05-security-vs-convenience.md)

---

*Unit 1 - Topic 4 of 6 | [Back to README](../README.md)*
