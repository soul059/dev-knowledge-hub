# Threat Intelligence

## Unit 2 - Topic 4 | Threat Landscape

---

## Overview

**Threat Intelligence** (TI) is evidence-based knowledge about existing or emerging threats to an organization. It provides context — who is attacking, why, how, and what indicators to look for — enabling proactive, informed security decisions instead of reactive firefighting.

---

## 1. What is Threat Intelligence?

```
┌──────────────────────────────────────────────────────────────┐
│               THREAT INTELLIGENCE DEFINED                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  DATA ──► INFORMATION ──► INTELLIGENCE                       │
│                                                              │
│  Raw logs,    Processed,      Analyzed, contextualized,     │
│  indicators   correlated      actionable knowledge          │
│                                                              │
│  "IP 1.2.3.4   "IP 1.2.3.4    "APT28 is using IP 1.2.3.4  │
│   connected"    is malicious"   as C2 server targeting       │
│                                 defense sector via spear     │
│                                 phishing with CVE-XXXX"      │
│                                                              │
│  Volume: High   Volume: Medium  Volume: Low                  │
│  Value: Low     Value: Medium   Value: HIGH                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Types of Threat Intelligence

```
┌──────────────────────────────────────────────────────────────────┐
│                THREAT INTELLIGENCE PYRAMID                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│                          ╱╲                                      │
│                         ╱  ╲        STRATEGIC                    │
│                        ╱    ╲       "What threats are emerging?" │
│                       ╱      ╲      Audience: C-Suite, Board     │
│                      ╱────────╲                                  │
│                     ╱          ╲     OPERATIONAL                  │
│                    ╱            ╲    "Who, why, when?"            │
│                   ╱              ╲   Audience: Security Managers  │
│                  ╱────────────────╲                               │
│                 ╱                  ╲   TACTICAL                   │
│                ╱                    ╲  "How do they attack?"      │
│               ╱                      ╲ Audience: SOC/IR Teams    │
│              ╱────────────────────────╲                           │
│             ╱                          ╲  TECHNICAL               │
│            ╱        TECHNICAL           ╲ "What are the IoCs?"   │
│           ╱                              ╲ Audience: Systems     │
│          ╱────────────────────────────────╲                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

| Type | Description | Audience | Format | Lifespan |
|------|-------------|----------|--------|----------|
| **Strategic** | High-level trends, geopolitical context | Executives, board | Reports, briefings | Months-years |
| **Operational** | Specific campaigns, actor TTPs | Security managers | Analysis reports | Weeks-months |
| **Tactical** | Attack techniques, procedures | SOC, IR, threat hunters | MITRE ATT&CK mappings | Weeks |
| **Technical** | Specific IoCs (IPs, hashes, domains) | Security tools, SIEM | Feeds, STIX/TAXII | Hours-days |

---

## 3. Threat Intelligence Lifecycle

```
┌──────────────────────────────────────────────────────────────┐
│             THREAT INTELLIGENCE LIFECYCLE                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│        ┌──────────────┐                                      │
│        │ 1. PLANNING  │   Define requirements, priorities    │
│        │ & DIRECTION  │   What do we need to know?           │
│        └──────┬───────┘                                      │
│               │                                              │
│               ▼                                              │
│        ┌──────────────┐                                      │
│        │ 2. COLLECTION│   Gather data from sources           │
│        │              │   OSINT, feeds, dark web, partners   │
│        └──────┬───────┘                                      │
│               │                                              │
│               ▼                                              │
│        ┌──────────────┐                                      │
│        │ 3. PROCESSING│   Normalize, enrich, correlate       │
│        │              │   Remove duplicates, structure data  │
│        └──────┬───────┘                                      │
│               │                                              │
│               ▼                                              │
│        ┌──────────────┐                                      │
│        │ 4. ANALYSIS  │   Contextualize, assess, conclude    │
│        │              │   Turn information into intelligence │
│        └──────┬───────┘                                      │
│               │                                              │
│               ▼                                              │
│        ┌──────────────┐                                      │
│        │5.DISSEMINATION│  Share with stakeholders            │
│        │              │   Right intel to right people        │
│        └──────┬───────┘                                      │
│               │                                              │
│               ▼                                              │
│        ┌──────────────┐                                      │
│        │ 6. FEEDBACK  │   Evaluate effectiveness             │
│        │              │   Adjust requirements                │
│        └──────────────┘                                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Indicators of Compromise (IoCs)

| IoC Type | Examples | Detection Method |
|----------|---------|------------------|
| **IP Addresses** | Malicious C2 server IPs | Firewall logs, SIEM |
| **Domain Names** | Phishing domains, C2 domains | DNS logs, threat feeds |
| **File Hashes** | MD5/SHA-256 of malware samples | EDR, file integrity monitoring |
| **URLs** | Malicious download links | Web proxy, email gateway |
| **Email Addresses** | Phishing sender addresses | Email security gateway |
| **Registry Keys** | Persistence mechanisms | Endpoint monitoring |
| **Mutex Names** | Malware unique identifiers | Behavioral analysis |
| **YARA Rules** | Pattern matching for malware | File scanning |

---

## 5. Threat Intelligence Sources

| Source Type | Examples | Quality |
|-----------|---------|---------|
| **Open Source (OSINT)** | VirusTotal, AbuseIPDB, AlienVault OTX | Varies |
| **Commercial Feeds** | Recorded Future, CrowdStrike, Mandiant | High |
| **Government** | CISA, FBI IC3, NCSC advisories | High |
| **Industry ISACs** | FS-ISAC, H-ISAC (sector-specific) | High |
| **Dark Web Monitoring** | Underground forums, paste sites | Medium |
| **Internal** | Own SIEM logs, incident data, honeypots | Very High |
| **Community** | MISP, OTX, Twitter/X security researchers | Varies |

---

## 6. Sharing Standards

| Standard | Purpose |
|----------|---------|
| **STIX** (Structured Threat Information eXpression) | Standard format for threat intel data |
| **TAXII** (Trusted Automated eXchange of Indicator Information) | Transport protocol for sharing STIX data |
| **MISP** (Malware Information Sharing Platform) | Open-source platform for sharing IoCs |
| **OpenIOC** | Framework for describing IoCs |
| **CybOX** | Standard for cyber observables (merged into STIX) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Definition** | Contextualized, actionable knowledge about threats |
| **Types** | Strategic, Operational, Tactical, Technical |
| **Lifecycle** | Planning → Collection → Processing → Analysis → Dissemination → Feedback |
| **IoCs** | IPs, domains, hashes, URLs — signs of compromise |
| **Sources** | OSINT, commercial, government, ISACs, internal |
| **Sharing** | STIX/TAXII standards for automated exchange |

---

## Quick Revision Questions

1. **What is the difference between data, information, and intelligence?**
2. **Name the four types of threat intelligence and their audiences.**
3. **List 5 types of Indicators of Compromise (IoCs).**
4. **What are the six phases of the Threat Intelligence Lifecycle?**
5. **What are STIX and TAXII used for?**
6. **Why is internal threat intelligence often the most valuable?**

---

[← Previous: Attack Vectors](03-attack-vectors.md) | [Next: Common Attack Types →](05-common-attack-types.md)

---

*Unit 2 - Topic 4 of 6 | [Back to README](../README.md)*
