# Unit 2: CompTIA Security+ — Topic 4: Domain 3 — Architecture and Design

## Overview

**Domain 3: Security Architecture (18%)** tests your knowledge of secure design principles, infrastructure concepts, and how to architect systems that resist attacks. This domain focuses on building security in rather than bolting it on.

---

## 1. Security Design Principles

```
CORE DESIGN PRINCIPLES:

  → Defense in depth: Multiple security layers
  → Least privilege: Minimum access needed
  → Separation of duties: No single person controls
  → Need to know: Information access limited
  → Fail-safe/fail-secure: Secure default state
  → Keep it simple (KISS): Reduce complexity
  → Zero trust: Never trust, always verify
  → Privacy by design: Build privacy in

ZERO TRUST MODEL:
  → "Never trust, always verify"
  → Verify identity continuously
  → Least privilege access
  → Microsegmentation
  → Assume breach mentality
  → Context-based access decisions
  
  Control Plane:
  → Policy engine + Policy administrator
  → Subject/system → Policy enforcement point
  
  Data Plane:
  → Implicit trust zones eliminated
  → All traffic inspected and authorized

SECURE DEFAULTS:
  → Deny by default
  → Disable unnecessary features
  → Change default passwords
  → Minimal installation
  → Hardened configuration
```

---

## 2. Resilience and Recovery

```
HIGH AVAILABILITY:

  → Redundancy: Duplicate critical components
  → Failover: Automatic switch to backup
  → Clustering: Multiple servers as one
  → Load balancing: Distribute workload
  → RAID: Disk redundancy

  RAID LEVELS:
  RAID 0: Striping (performance, no redundancy)
  RAID 1: Mirroring (full redundancy)
  RAID 5: Striping + parity (1 drive failure)
  RAID 6: Striping + double parity (2 drives)
  RAID 10: Mirroring + striping (performance + redundancy)

BACKUP STRATEGIES:
  Type        | What's Backed Up    | Speed   | Storage
  Full        | Everything          | Slow    | High
  Incremental | Changes since last  | Fast    | Low
  Differential| Changes since full  | Medium  | Medium
  Snapshot    | Point-in-time copy  | Instant | Varies

  BACKUP RULE: 3-2-1
  → 3 copies of data
  → 2 different media types
  → 1 offsite copy

DISASTER RECOVERY:
  → RPO (Recovery Point Objective): Max data loss
  → RTO (Recovery Time Objective): Max downtime
  → MTBF (Mean Time Between Failures)
  → MTTR (Mean Time To Repair)

  Site Types:
  → Hot site: Ready immediately
  → Warm site: Hours to activate
  → Cold site: Days/weeks to activate
  → Cloud-based: On-demand recovery
```

---

## 3. Secure Network Architecture

```
NETWORK SECURITY DESIGN:

  ┌──────────────────────────────────────┐
  │  INTERNET                            │
  │      │                               │
  │  [Firewall] ← Edge/perimeter        │
  │      │                               │
  │  [DMZ/Screened Subnet]              │
  │      │  Web server, email relay      │
  │  [Firewall] ← Internal firewall     │
  │      │                               │
  │  [Internal Network]                  │
  │      │                               │
  │  [Segments/VLANs]                   │
  │  Users | Servers | IoT | Guest      │
  └──────────────────────────────────────┘

KEY CONCEPTS:
  → Screened subnet (DMZ): Buffer zone
  → Microsegmentation: Granular isolation
  → Network access control (NAC)
  → VPN for remote access
  → SD-WAN for branch security
  → East-west traffic monitoring
  → Honeynet/honeypot: Deception

SECURE DESIGN PATTERNS:
  → Bastion host/jump server
  → Proxy servers (forward/reverse)
  → API gateway security
  → Service mesh for microservices
  → Data diodes (one-way transfer)
```

---

## 4. Embedded and Specialized Systems

```
SPECIALIZED SYSTEMS:

IoT SECURITY:
  → Change default credentials
  → Firmware updates
  → Network segmentation
  → Disable unnecessary services
  → Encrypt communications
  → Monitor for anomalies

SCADA/ICS:
  → Supervisory Control and Data Acquisition
  → Industrial Control Systems
  → Air gap from IT network
  → Modbus, DNP3 protocols
  → Safety systems critical
  → Long lifecycle equipment

EMBEDDED SYSTEMS:
  → RTOS (Real-Time Operating System)
  → Limited computing resources
  → SoC (System on Chip)
  → FPGA (Field Programmable Gate Array)
  → Cannot easily patch/update
  → Physical security important

MOBILE DEVICES:
  → MDM (Mobile Device Management)
  → MAM (Mobile Application Management)
  → BYOD vs COPE vs CYOD policies
  → Containerization
  → Remote wipe capability
  → Geofencing/geolocation
```

---

## 5. Cryptographic Architecture

```
CRYPTOGRAPHIC CONCEPTS:

SYMMETRIC:
  → Same key for encrypt/decrypt
  → Fast, used for bulk data
  → AES (128/192/256-bit): Current standard
  → 3DES: Legacy
  → Key distribution challenge

ASYMMETRIC:
  → Public/private key pair
  → Slow, used for key exchange
  → RSA: Encryption + signatures
  → ECC: Smaller keys, mobile-friendly
  → Diffie-Hellman: Key exchange

HASHING:
  → One-way function
  → Fixed-length output
  → MD5: 128-bit (insecure)
  → SHA-1: 160-bit (deprecated)
  → SHA-256: 256-bit (current)
  → SHA-3: Latest standard
  → Used for integrity verification

PKI (Public Key Infrastructure):
  → Certificate Authority (CA)
  → Registration Authority (RA)
  → Digital certificates (X.509)
  → Certificate revocation (CRL, OCSP)
  → Certificate pinning
  → Key escrow
  → Certificate lifecycle

EXAM KEY POINTS:
  → AES = symmetric standard
  → RSA = asymmetric standard
  → SHA-256 = hashing standard
  → Know symmetric vs asymmetric use cases
  → Know PKI components
```

---

## Summary Table

| Topic | Key Concepts | Exam Tips |
|-------|-------------|----------|
| Design Principles | Zero trust, defense in depth | Know definitions |
| Resilience | RAID, backup, DR | Know RAID levels |
| Network Architecture | DMZ, segmentation | Know design patterns |
| Specialized Systems | IoT, SCADA, mobile | Security challenges |
| Cryptography | AES, RSA, SHA, PKI | Know algorithms + use |

---

## Revision Questions

1. What are the key principles of zero trust architecture?
2. What RAID level provides both performance and redundancy?
3. What is the difference between RPO and RTO?
4. What security measures apply to IoT devices?
5. When would you use symmetric vs asymmetric encryption?

---

*Previous: [03-domain-2-technologies-and-tools.md](03-domain-2-technologies-and-tools.md) | Next: [05-domain-4-identity-access-management.md](05-domain-4-identity-access-management.md)*

---

*[Back to README](../README.md)*
