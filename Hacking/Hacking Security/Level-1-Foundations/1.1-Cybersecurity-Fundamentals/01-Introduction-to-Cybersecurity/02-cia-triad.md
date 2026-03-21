# CIA Triad

## Unit 1 - Topic 2 | Introduction to Cybersecurity

---

## Overview

The **CIA Triad** is the most fundamental model in information security. It defines three core objectives that every security program must address: **Confidentiality**, **Integrity**, and **Availability**. Every security control, policy, and decision can be mapped back to one or more of these three pillars.

---

## 1. The CIA Triad Model

```
                         CONFIDENTIALITY
                              ▲
                             ╱ ╲
                            ╱   ╲
                           ╱     ╲
                          ╱       ╲
                         ╱  CIA    ╲
                        ╱  TRIAD    ╲
                       ╱             ╲
                      ╱               ╲
                     ╱                 ╲
                    ▼                   ▼
              INTEGRITY ◄──────────► AVAILABILITY

    All three must be balanced — weakening one affects others
```

---

## 2. Confidentiality

### Definition

> **Confidentiality** ensures that information is accessible only to those authorized to have access. It protects data from unauthorized disclosure.

### Key Concepts

```
┌────────────────────────────────────────────────────────────┐
│                    CONFIDENTIALITY                          │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  WHO can see this data?                                    │
│                                                            │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐     │
│  │Encryption│    │  Access   │    │  Data            │     │
│  │          │    │  Controls │    │  Classification  │     │
│  └──────────┘    └──────────┘    └──────────────────┘     │
│                                                            │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐     │
│  │  MFA     │    │  VPN     │    │  Data Masking    │     │
│  │          │    │          │    │                  │     │
│  └──────────┘    └──────────┘    └──────────────────┘     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Controls for Confidentiality

| Control Type | Examples |
|-------------|----------|
| **Technical** | Encryption (AES-256, TLS), access control lists, firewalls |
| **Administrative** | Security policies, data classification, background checks |
| **Physical** | Locked server rooms, biometric access, security cameras |

### Threats to Confidentiality

| Threat | Description | Example |
|--------|-------------|---------|
| **Eavesdropping** | Intercepting data in transit | Packet sniffing on unsecured Wi-Fi |
| **Data Breach** | Unauthorized access to stored data | SQL injection exposing customer database |
| **Social Engineering** | Tricking people into revealing info | Phishing email stealing credentials |
| **Shoulder Surfing** | Observing someone's screen/keyboard | Watching PIN entry at ATM |
| **Dumpster Diving** | Searching discarded materials | Finding unshredded documents in trash |

### Real-World Application

```
SCENARIO: Online Banking

  Customer ──────[HTTPS/TLS]──────► Bank Server
     │                                   │
     │  • Password + MFA                 │  • Encrypted database
     │  • Session encryption             │  • Access controls
     │  • Timeout after idle             │  • Audit logging
     │                                   │
     └── Confidentiality Controls ───────┘
```

---

## 3. Integrity

### Definition

> **Integrity** ensures that data is accurate, consistent, and has not been altered by unauthorized parties. It guarantees trustworthiness of information.

### Key Concepts

```
┌────────────────────────────────────────────────────────────┐
│                      INTEGRITY                              │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Has the data been TAMPERED with?                          │
│                                                            │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐     │
│  │  Hashing │    │  Digital │    │  Version         │     │
│  │  (SHA)   │    │  Sigs    │    │  Control         │     │
│  └──────────┘    └──────────┘    └──────────────────┘     │
│                                                            │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐     │
│  │  Input   │    │  Audit   │    │  Checksums       │     │
│  │  Valid.  │    │  Trails  │    │                  │     │
│  └──────────┘    └──────────┘    └──────────────────┘     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Types of Integrity

| Type | Description | Example |
|------|-------------|---------|
| **Data Integrity** | Accuracy of stored data | Database records unchanged by unauthorized users |
| **System Integrity** | System operates as intended | OS files not modified by malware |
| **Source Integrity** | Data comes from trusted origin | Verified email sender via DKIM |

### Controls for Integrity

| Control | How It Works |
|---------|-------------|
| **Hashing** | SHA-256 hash verifies file hasn't changed |
| **Digital Signatures** | Proves sender identity + data hasn't been modified |
| **Checksums** | Quick verification of data integrity during transfer |
| **Access Controls** | Prevent unauthorized modification (write permissions) |
| **Version Control** | Track all changes with history (Git) |
| **Input Validation** | Ensure only valid data enters the system |
| **Database Constraints** | Enforce data rules at the database level |

### Integrity Check Example

```
File Integrity Monitoring (FIM):

  Original File ──► SHA-256 Hash ──► Store Hash in DB
                    a3f2b8c9d1...

  Later Check:
  Current File  ──► SHA-256 Hash ──► Compare with DB
                    a3f2b8c9d1...     ✅ Match = Integrity OK
                    OR
                    e7d4a1f3b2...     ❌ Mismatch = TAMPERED!
```

---

## 4. Availability

### Definition

> **Availability** ensures that systems, applications, and data are accessible to authorized users when needed. It guarantees uptime and reliability.

### Key Concepts

```
┌────────────────────────────────────────────────────────────┐
│                     AVAILABILITY                            │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Can authorized users ACCESS the system when needed?       │
│                                                            │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐     │
│  │Redundancy│    │  Load    │    │  Disaster        │     │
│  │          │    │  Balance │    │  Recovery        │     │
│  └──────────┘    └──────────┘    └──────────────────┘     │
│                                                            │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐     │
│  │  Backups │    │  Failover│    │  DDoS            │     │
│  │          │    │          │    │  Protection      │     │
│  └──────────┘    └──────────┘    └──────────────────┘     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Availability Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| **Uptime %** | Percentage of time system is operational | 99.99% ("four nines") |
| **RTO** | Recovery Time Objective — max downtime tolerated | Varies by criticality |
| **RPO** | Recovery Point Objective — max data loss tolerated | Minutes to hours |
| **MTTR** | Mean Time To Repair | As low as possible |
| **MTBF** | Mean Time Between Failures | As high as possible |

### Uptime Table (The "Nines")

| Availability | Downtime/Year | Downtime/Month | Downtime/Week |
|-------------|---------------|----------------|---------------|
| 99% | 3.65 days | 7.31 hours | 1.68 hours |
| 99.9% | 8.77 hours | 43.83 min | 10.08 min |
| 99.99% | 52.60 min | 4.38 min | 1.01 min |
| 99.999% | 5.26 min | 26.30 sec | 6.05 sec |

### Threats to Availability

| Threat | Description |
|--------|-------------|
| **DDoS Attacks** | Overwhelming systems with traffic |
| **Hardware Failure** | Disk crashes, power supply failure |
| **Natural Disasters** | Floods, earthquakes, fires |
| **Ransomware** | Encrypts data making it inaccessible |
| **Human Error** | Accidental deletion, misconfiguration |
| **Power Outage** | Loss of electricity |

---

## 5. Balancing the Triad

### The Tradeoff Challenge

```
High Confidentiality + High Integrity = Lower Availability
  (Too many security checks slow down access)

High Availability + High Confidentiality = Lower Integrity
  (Fast replication may not verify data consistency)

High Availability + High Integrity = Lower Confidentiality
  (Open systems with audit trails but less access control)

                    THE GOAL:
         ┌─────────────────────────┐
         │   Balance all three     │
         │   based on business     │
         │   requirements and      │
         │   risk tolerance        │
         └─────────────────────────┘
```

### CIA Applied to Different Systems

| System | Priority Order | Reasoning |
|--------|---------------|-----------|
| **Military Intelligence** | C > I > A | Secrets must stay secret |
| **Banking Transactions** | I > C > A | Money amounts must be accurate |
| **Emergency Services (911)** | A > I > C | Systems must always be available |
| **E-Commerce Website** | A > C > I | Uptime = revenue |
| **Medical Records** | I > A > C | Patient data must be accurate |
| **Social Media** | A > I > C | Users expect constant access |

---

## 6. Extended Models

### The Parkerian Hexad (Extended CIA)

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│Confidentiality│  │  Integrity   │  │ Availability │
│              │  │              │  │              │
└──────────────┘  └──────────────┘  └──────────────┘
        +                 +                 +
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Possession  │  │ Authenticity │  │   Utility    │
│  (Control)   │  │  (Genuine)   │  │ (Usefulness) │
└──────────────┘  └──────────────┘  └──────────────┘
```

| Property | Description |
|----------|-------------|
| **Possession** | Physical control of the data medium |
| **Authenticity** | Data is genuine and from a verified source |
| **Utility** | Data is in a usable format |

---

## Summary Table

| CIA Element | Question | Key Controls | Primary Threat |
|-------------|----------|-------------|----------------|
| **Confidentiality** | Who can see it? | Encryption, access controls, MFA | Data breach, eavesdropping |
| **Integrity** | Has it been changed? | Hashing, digital signatures, audit logs | Tampering, MITM attacks |
| **Availability** | Can I access it? | Redundancy, backups, DDoS protection | DDoS, ransomware, failure |

---

## Quick Revision Questions

1. **What are the three pillars of the CIA Triad and what does each protect?**
2. **Give an example where Availability takes priority over Confidentiality.**
3. **How does hashing help ensure Integrity?**
4. **What is the difference between "four nines" and "five nines" availability?**
5. **Explain the Parkerian Hexad and what it adds to the CIA Triad.**
6. **For a hospital's patient records system, how would you prioritize C, I, and A?**

---

[← Previous: What is Cybersecurity?](01-what-is-cybersecurity.md) | [Next: AAA Framework →](03-aaa-framework.md)

---

*Unit 1 - Topic 2 of 6 | [Back to README](../README.md)*
