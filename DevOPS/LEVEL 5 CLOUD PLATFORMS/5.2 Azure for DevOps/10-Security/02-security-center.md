# Chapter 2: Microsoft Defender for Cloud

## Overview
**Microsoft Defender for Cloud** (formerly Azure Security Center + Azure Defender) provides unified security management and threat protection across Azure, hybrid, and multi-cloud environments. It offers **security posture management (CSPM)**, **workload protection (CWP)**, and **compliance monitoring**.

---

## 2.1 Defender for Cloud Architecture

```
┌────────── MICROSOFT DEFENDER FOR CLOUD ──────────┐
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │  CLOUD SECURITY POSTURE MANAGEMENT (CSPM)    │  │
│  │                                              │  │
│  │  Secure Score: 72/100                        │  │
│  │  ┌──────────────────────────────────────┐    │  │
│  │  │ ████████████████████░░░░░░░░  72%    │    │  │
│  │  └──────────────────────────────────────┘    │  │
│  │                                              │  │
│  │  Recommendations:                            │  │
│  │  ├── 🔴 Enable MFA for accounts with owner  │  │
│  │  ├── 🔴 Encrypt storage accounts in transit  │  │
│  │  ├── 🟡 Enable DDoS protection              │  │
│  │  ├── 🟡 Restrict public IP addresses        │  │
│  │  └── 🟢 Enable diagnostic logs ✅           │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │  CLOUD WORKLOAD PROTECTION (CWP)             │  │
│  │                                              │  │
│  │  Defender Plans:                             │  │
│  │  ├── Defender for Servers (VM threats)        │  │
│  │  ├── Defender for App Service               │  │
│  │  ├── Defender for SQL (injection, anomalies) │  │
│  │  ├── Defender for Storage (malware)          │  │
│  │  ├── Defender for Containers (AKS, ACR)      │  │
│  │  ├── Defender for Key Vault                  │  │
│  │  ├── Defender for DNS                        │  │
│  │  └── Defender for Resource Manager           │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │  SECURITY ALERTS                             │  │
│  │  ├── 🔴 Brute force SSH detected (Sev High) │  │
│  │  ├── 🟡 Suspicious process on VM (Sev Med)  │  │
│  │  └── 🔵 Unusual Azure operation (Sev Low)   │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 2.2 Secure Score

```
┌────── SECURE SCORE ───────────────────────┐
│                                            │
│  Score = Points Achieved / Max Points      │
│                                            │
│  Categories:                               │
│  ┌──────────────────────────────────────┐  │
│  │  Identity & Access ─────── 12/15     │  │
│  │  ██████████████░░░░  80%             │  │
│  │                                     │  │
│  │  Network Security ──────── 8/12      │  │
│  │  ██████████░░░░░░░  67%              │  │
│  │                                     │  │
│  │  Data & Storage ────────── 10/10     │  │
│  │  ████████████████  100% ✅           │  │
│  │                                     │  │
│  │  Compute & Apps ────────── 6/10      │  │
│  │  ████████░░░░░░░  60%               │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Fix recommendations to increase score     │
│  Each fix shows max point increase         │
│                                            │
└────────────────────────────────────────────┘
```

---

## 2.3 Recommendations

| Category | Example Recommendations |
|----------|------------------------|
| **Identity** | Enable MFA, remove deprecated accounts, use managed identities |
| **Network** | Restrict management ports, enable NSG on subnets, use Private Endpoints |
| **Compute** | Apply system updates, enable endpoint protection, encrypt disks |
| **Data** | Enable TDE on SQL, use HTTPS-only storage, enable auditing |
| **Containers** | Use private ACR, enable Defender for Containers, scan images |

---

## 2.4 Defender Plans

| Plan | Protects | Key Features |
|------|----------|--------------|
| **Servers** | VMs | Vulnerability assessment, file integrity, JIT VM access |
| **App Service** | Web Apps | Detects attacks (SQL injection, command injection) |
| **SQL** | SQL DB & MI | SQL injection detection, anomaly detection, vulnerability assessment |
| **Storage** | Blobs, Files | Malware scanning, suspicious access patterns |
| **Containers** | AKS, ACR | Image scanning, runtime protection, admission control |
| **Key Vault** | Key Vault | Unusual access patterns, suspicious operations |
| **DNS** | DNS queries | Detects malware communication, data exfiltration |

---

## 2.5 Just-In-Time (JIT) VM Access

```
┌────── JIT VM ACCESS ──────────────────────┐
│                                            │
│  Problem: Management ports (22, 3389)      │
│  open 24/7 = attack surface               │
│                                            │
│  Solution: JIT locks ports by default      │
│                                            │
│  Flow:                                     │
│  ┌──────────────────────────────────────┐  │
│  │  1. NSG blocks all inbound 3389/22  │  │
│  │                                     │  │
│  │  2. Admin requests JIT access       │  │
│  │     (Port 22, 2 hours, from my IP)  │  │
│  │                                     │  │
│  │  3. Defender approves (auto/manual) │  │
│  │                                     │  │
│  │  4. NSG rule added: Allow 22 from   │  │
│  │     admin IP for 2 hours            │  │
│  │                                     │  │
│  │  5. After 2 hours → rule removed    │  │
│  │     Port locked again               │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

```bash
# Enable JIT VM access
az security jit-policy create \
  --resource-group myRG \
  --name "myVM" \
  --location uksouth \
  --virtual-machines '[{
    "id": "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM",
    "ports": [
      {"number": 22, "protocol": "TCP", "maxRequestAccessDuration": "PT3H"},
      {"number": 3389, "protocol": "TCP", "maxRequestAccessDuration": "PT3H"}
    ]
  }]'
```

---

## 2.6 Regulatory Compliance

```
┌────── COMPLIANCE DASHBOARD ───────────────┐
│                                            │
│  Standards:                                │
│  ┌──────────────────────────────────────┐  │
│  │  Azure Security Benchmark ── 89%     │  │
│  │  ███████████████████░░  89%          │  │
│  │                                     │  │
│  │  ISO 27001 ────────────── 82%        │  │
│  │  ████████████████░░░░  82%           │  │
│  │                                     │  │
│  │  PCI DSS ──────────────── 75%        │  │
│  │  ███████████████░░░░░  75%           │  │
│  │                                     │  │
│  │  GDPR ─────────────────── 91%        │  │
│  │  ██████████████████░░  91%           │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Each standard shows passing/failing       │
│  controls mapped to recommendations        │
│                                            │
└────────────────────────────────────────────┘
```

---

## 2.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Secure Score not updating | Takes up to 24 hours | Wait for next assessment cycle |
| Recommendations don't appear | Defender plan not enabled | Enable relevant Defender plan |
| JIT request denied | No JIT policy configured | Configure JIT for the VM first |
| Alerts not generating | Log collection not enabled | Enable diagnostic settings and agent |
| Compliance % seems wrong | Not all resources assessed yet | Wait for full assessment scan |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Defender for Cloud** | Unified security: CSPM + CWP across Azure/hybrid/multi-cloud |
| **Secure Score** | Percentage-based security posture rating |
| **Recommendations** | Actionable fixes to improve security posture |
| **Defender Plans** | Workload protection for Servers, SQL, Containers, KV, etc. |
| **JIT VM Access** | Lock management ports, open on-demand for limited time |
| **Compliance** | Map controls to standards (ISO, PCI, GDPR, ASB) |

---

## Quick Revision Questions

1. **What is Microsoft Defender for Cloud?**
   > A unified security management platform providing cloud security posture management (CSPM) and cloud workload protection (CWP) across Azure, hybrid, and multi-cloud.

2. **What is Secure Score?**
   > A percentage showing your security posture. Fix recommendations to increase the score. Higher score = better security.

3. **What is JIT VM Access?**
   > Just-In-Time access locks management ports (22, 3389) and opens them only when requested, for a specific IP and duration.

4. **Name three Defender plans.**
   > Defender for Servers (VM protection), Defender for SQL (injection/anomaly detection), Defender for Containers (AKS/ACR scanning and runtime protection).

5. **How does the compliance dashboard work?**
   > It maps Defender for Cloud recommendations to regulatory standards (ISO 27001, PCI DSS, GDPR), showing percentage compliance per standard.

---

[⬅ Previous: Azure Key Vault](01-azure-key-vault.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Network Security ➡](03-network-security.md)
