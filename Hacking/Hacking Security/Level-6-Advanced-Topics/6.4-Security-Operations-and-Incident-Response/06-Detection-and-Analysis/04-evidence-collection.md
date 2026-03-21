# Unit 6: Detection and Analysis — Topic 4: Evidence Collection

## Overview

**Evidence collection** during incident response preserves digital artifacts for investigation, remediation, and potential legal proceedings. Proper evidence collection ensures forensic integrity, maintains chain of custody, and provides the foundation for understanding what happened during an incident.

---

## 1. Evidence Collection Principles

```
EVIDENCE COLLECTION ORDER OF VOLATILITY:

  MOST VOLATILE (collect first):
  1. CPU registers, cache
  2. Memory (RAM)
  3. Network state (connections, ARP)
  4. Running processes
  5. Disk (file system)
  6. Remote logging data
  7. Physical configuration
  8. Archival media (backups)
  LEAST VOLATILE (collect last)

  Principle: Collect most volatile first because
  it changes or disappears quickly.

KEY PRINCIPLES:
  → Preserve original evidence
  → Work on forensic copies
  → Document everything
  → Maintain chain of custody
  → Use validated tools
  → Minimize changes to evidence
  → Hash everything
  → Timestamp all actions
```

---

## 2. Memory Collection

```
MEMORY ACQUISITION:

WHY COLLECT MEMORY:
  → Running processes (including hidden)
  → Network connections
  → Encryption keys
  → Malware in memory only
  → Command history
  → User credentials
  → Injected code
  → Evidence of anti-forensics

WINDOWS MEMORY COLLECTION:
  DumpIt:
  → Double-click to execute
  → Creates raw memory dump
  → Fast and simple
  → Output: hostname_date.raw

  WinPmem:
  → winpmem_mini_x64.exe output.raw
  → Supports multiple formats
  → Can include pagefile

  FTK Imager:
  → File > Capture Memory
  → GUI-based
  → Include pagefile option
  → Widely accepted forensically

LINUX MEMORY COLLECTION:
  LiME (Linux Memory Extractor):
  → insmod lime.ko "path=/evidence/mem.lime format=lime"
  → Kernel module based
  → Minimal system impact

REMOTE MEMORY COLLECTION:
  Velociraptor:
  → SELECT * FROM 
    Artifact.Windows.Memory.Acquisition()
  → Remote collection at scale
  → Centralized storage

  CrowdStrike:
  → RTR (Real Time Response)
  → memdump command
  → Collected to cloud

MEMORY COLLECTION CHECKLIST:
  [ ] Determine collection method
  [ ] Prepare storage media
  [ ] Record system time and date
  [ ] Execute memory dump tool
  [ ] Calculate hash of memory image
  [ ] Document collection details
  [ ] Secure evidence
  [ ] Complete chain of custody form
```

---

## 3. Disk and Artifact Collection

```
DISK EVIDENCE COLLECTION:

FULL DISK IMAGING:
  FTK Imager (GUI):
  → File > Create Disk Image
  → Select source drive
  → Choose format (E01 recommended)
  → Set compression and verify
  → Image splits for large drives

  dd (Linux CLI):
  → dd if=/dev/sda of=/evidence/disk.dd bs=4M
  → dc3dd: Enhanced dd with hashing
  → dc3dd if=/dev/sda of=/evidence/disk.dd hash=sha256

  WRITE BLOCKERS:
  → Hardware: Tableau, CRU
  → Software: Windows Registry forensic mode
  → CRITICAL: Always use when imaging
  → Prevents any writes to evidence drive

TARGETED ARTIFACT COLLECTION:
  KAPE (Kroll Artifact Parser):
  → Collects specific forensic artifacts
  → Much faster than full disk image
  → Pre-built target collections
  
  kape.exe --tsource C: --tdest D:\Evidence 
    --target KapeTriage --vhdx EVIDENCE

  KEY WINDOWS ARTIFACTS:
  → Event logs: C:\Windows\System32\winevt\Logs\
  → Registry hives: SYSTEM, SAM, SOFTWARE, SECURITY
  → Prefetch: C:\Windows\Prefetch\
  → Amcache: C:\Windows\AppCompat\Programs\
  → NTFS artifacts: $MFT, $UsnJrnl
  → Browser history and downloads
  → PowerShell history
  → Recent files (LNK files)
  → Scheduled tasks
  → Startup items

  KEY LINUX ARTIFACTS:
  → /var/log/ (all log files)
  → /etc/passwd, /etc/shadow
  → /home/*/.bash_history
  → /etc/crontab, /var/spool/cron/
  → /tmp/ (temporary files)
  → /root/.ssh/authorized_keys
  → /etc/hosts
```

---

## 4. Network Evidence

```
NETWORK EVIDENCE COLLECTION:

PACKET CAPTURE:
  tcpdump:
  → tcpdump -i eth0 -w /evidence/capture.pcap
  → tcpdump -i eth0 host 203.0.113.50 -w suspect.pcap

  Wireshark:
  → GUI-based capture
  → Real-time analysis
  → Filter during or after capture

  Full PCAP Solutions:
  → Arkime (formerly Moloch)
  → Security Onion
  → Network TAP/SPAN port

LOG-BASED EVIDENCE:
  → Firewall logs (connections, blocks)
  → Proxy logs (URLs, user agents)
  → DNS logs (queries, responses)
  → IDS/IPS alerts
  → VPN logs
  → DHCP logs (IP-to-MAC mapping)
  → NetFlow (traffic metadata)

SIEM EVIDENCE:
  → Export relevant search results
  → Save queries used
  → Document time ranges
  → Export raw events (JSON/CSV)
  → Screenshot dashboards

CLOUD EVIDENCE:
  → CloudTrail logs (AWS)
  → Activity logs (Azure)
  → Audit logs (GCP)
  → SaaS application logs (O365, GSuite)
  → Cloud storage access logs
  → IAM/identity logs

EVIDENCE PRESERVATION:
  → Export logs to immutable storage
  → Hash log exports
  → Document log source and time range
  → Verify log completeness
  → Store securely with access controls
```

---

## 5. Evidence Handling and Storage

```
CHAIN OF CUSTODY:

  ┌──────────────────────────────────────────────┐
  │ CHAIN OF CUSTODY FORM                        │
  │                                              │
  │ Case ID: INC-2024-001                       │
  │ Evidence ID: EVD-001                        │
  │ Description: Memory dump from JSMITH-PC     │
  │ Hash (SHA256): a1b2c3d4e5f6...             │
  │                                              │
  │ Collection:                                  │
  │ Date: 2024-01-15 11:30 UTC                  │
  │ Collected by: Jane Doe (Analyst)            │
  │ Location: Building A, Floor 3, Desk 42     │
  │ Tool: WinPmem v3.3                          │
  │ Method: Live memory acquisition             │
  │                                              │
  │ Transfer Log:                                │
  │ Date       | From      | To        | Purpose │
  │ 2024-01-15 | J. Doe    | Evidence  | Storage │
  │            |           | Locker A  |         │
  │ 2024-01-16 | Locker A  | J. Smith  | Analysis│
  │ 2024-01-16 | J. Smith  | Locker A  | Return  │
  └──────────────────────────────────────────────┘

STORAGE REQUIREMENTS:
  → Encrypted storage
  → Access controls (limited)
  → Tamper-evident containers
  → Environmentally controlled
  → Backup of all evidence
  → Retention per policy/legal requirement
  → Secure disposal when no longer needed

DIGITAL EVIDENCE INTEGRITY:
  → Hash original evidence (SHA256)
  → Hash forensic copies
  → Compare hashes to verify integrity
  → Document hash values
  → Re-verify before analysis
  → Use write protection always
```

---

## Summary Table

| Evidence Type | Tool | Priority | Volatility |
|--------------|------|----------|-----------|
| Memory | DumpIt, WinPmem | Critical | Very High |
| Running Processes | EDR, Velociraptor | Critical | High |
| Network Connections | netstat, EDR | High | High |
| Disk Image | FTK Imager, dd | High | Low |
| Targeted Artifacts | KAPE | High | Low-Medium |
| Network Capture | tcpdump, Wireshark | Medium | High |
| Logs | SIEM export | Medium | Medium |

---

## Revision Questions

1. What is the order of volatility and why does it matter?
2. What tools are used for memory acquisition on Windows?
3. What key Windows artifacts should be collected during IR?
4. What information must a chain of custody form include?
5. How is digital evidence integrity verified?

---

*Previous: [03-scope-determination.md](03-scope-determination.md) | Next: [05-timeline-development.md](05-timeline-development.md)*

---

*[Back to README](../README.md)*
