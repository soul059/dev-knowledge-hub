# Unit 9: Digital Forensics Basics — Topic 4: Disk Forensics Basics

## Overview

**Disk forensics** examines storage media (hard drives, SSDs, USB drives) to recover, analyze, and interpret data relevant to an investigation. This includes examining file systems, recovering deleted files, analyzing artifacts, and building a comprehensive picture of user activity on the system.

---

## 1. File System Fundamentals

```
FILE SYSTEM TYPES:

  File System | OS          | Key Features
  NTFS        | Windows     | Journaling, ACLs, ADS, MFT
  FAT32       | Cross-platform| No journaling, 4GB file limit
  exFAT       | Cross-platform| Large files, flash storage
  ext4        | Linux       | Journaling, inodes
  APFS        | macOS/iOS   | Encryption, snapshots
  HFS+        | macOS (old) | Resource forks

NTFS KEY STRUCTURES:
  $MFT (Master File Table):
  → Record for every file and directory
  → File metadata (timestamps, size, attributes)
  → Data runs (location on disk)
  → Resident data (small files stored in MFT)

  $LogFile (NTFS Journal):
  → Records file system transactions
  → Used for recovery after crash
  → Contains recent file operations

  $UsnJrnl (Update Sequence Number Journal):
  → Records changes to files
  → File created, modified, deleted
  → Excellent for timeline analysis
  → Can contain millions of entries

  $Bitmap:
  → Tracks allocated/unallocated clusters
  → Shows where deleted data may exist

FORENSIC SIGNIFICANCE:
  → Deleted files may still exist on disk
  → Metadata persists in MFT after deletion
  → Journal records track recent activity
  → Slack space may contain old data
  → Alternate data streams can hide data
```

---

## 2. Key Windows Artifacts

```
WINDOWS FORENSIC ARTIFACTS:

FILE SYSTEM ARTIFACTS:
  Artifact       | Location                    | Evidence
  Prefetch       | C:\Windows\Prefetch\        | Program execution
  Amcache        | C:\Windows\AppCompat\       | Program execution
                 | Programs\Amcache.hve        |
  ShimCache      | SYSTEM registry hive        | Program execution
  LNK Files      | C:\Users\*\Recent\          | File access
  Jump Lists     | C:\Users\*\AppData\         | File/app access
                 | Roaming\Microsoft\Windows\  |
                 | Recent\AutomaticDestinations\|
  Recycle Bin    | C:\$Recycle.Bin\            | Deleted files
  Thumbs/Thumbcache| C:\Users\*\AppData\      | Image previews
                    | Local\Microsoft\Windows\ |
                    | Explorer\                |

REGISTRY ARTIFACTS:
  Artifact       | Key                         | Evidence
  Run/RunOnce    | HKLM/HKCU\...\Run          | Persistence
  UserAssist     | HKCU\...\UserAssist         | Program execution
  MRU Lists      | HKCU\...\RecentDocs         | Recently opened
  Typed URLs     | HKCU\...\TypedURLs          | Browser URLs
  USB History    | HKLM\SYSTEM\...\Enum\USB    | USB devices
  BAM            | HKLM\SYSTEM\...\bam         | Program execution
  Network History| HKLM\SOFTWARE\...\          | WiFi networks
                 | NetworkList                  |

EVENT LOGS:
  Log              | Key Events
  Security.evtx    | Logons (4624), Logoffs (4634)
                   | Failed logons (4625)
                   | Account changes (4720-4738)
  System.evtx      | Service installs (7045)
                   | Shutdowns (1074, 6008)
  PowerShell.evtx  | Script execution (4104)
  Sysmon.evtx      | Process create (1)
                   | Network connect (3)
                   | File create (11)

BROWSER ARTIFACTS:
  Browser  | History Location
  Chrome   | AppData\Local\Google\Chrome\
           | User Data\Default\History (SQLite)
  Firefox  | AppData\Roaming\Mozilla\Firefox\
           | Profiles\*\places.sqlite
  Edge     | AppData\Local\Microsoft\Edge\
           | User Data\Default\History
```

---

## 3. Forensic Analysis Tools

```
DISK FORENSIC TOOLS:

AUTOPSY (Open Source):
  → Full-featured forensic platform
  → File system analysis
  → Keyword search
  → Hash analysis
  → Timeline generation
  → Web artifact analysis
  → Email analysis
  → Data carving

  Workflow:
  1. Create new case
  2. Add data source (disk image)
  3. Run ingest modules (select all relevant)
  4. Wait for processing
  5. Browse results by category
  6. Generate reports

FTK IMAGER (Free):
  → Disk imaging
  → Evidence preview
  → Export files
  → View file system structure
  → Access deleted files
  → Create forensic images

ERIC ZIMMERMAN TOOLS (Free):
  Tool          | Purpose
  MFTECmd       | Parse $MFT
  PECmd         | Parse Prefetch files
  LECmd         | Parse LNK files
  JLECmd        | Parse Jump Lists
  AmcacheParser | Parse Amcache
  ShellBagsExplorer| Parse ShellBags
  Registry Explorer| Browse registry hives
  Timeline Explorer| View timelines
  EvtxECmd      | Parse event logs

  # Parse MFT
  MFTECmd.exe -f "$MFT" --csv C:\output --csvf mft.csv

  # Parse Prefetch
  PECmd.exe -d "C:\Windows\Prefetch" 
    --csv C:\output --csvf prefetch.csv

  # Parse event logs
  EvtxECmd.exe -d "C:\Windows\System32\winevt\Logs"
    --csv C:\output --csvf eventlogs.csv

KAPE (Free):
  → Automated artifact collection and processing
  → Targets: What to collect
  → Modules: How to process
  
  # Collect and process common artifacts
  kape.exe --tsource C: --tdest D:\Evidence 
    --target KapeTriage 
    --msource D:\Evidence --mdest D:\Processed 
    --module !EZParser
```

---

## 4. Deleted File Recovery

```
DELETED FILE RECOVERY:

HOW DELETION WORKS:
  → File marked as deleted in MFT
  → Clusters marked as unallocated
  → Data remains until overwritten
  → Metadata may persist in MFT
  → Recovery possible until overwritten

  NTFS:
  ┌──────────┐     ┌──────────┐
  │ File.docx│     │ MFT Entry│
  │ (Active) │────▶│ Allocated│
  └──────────┘     └──────────┘
       │ DELETE
  ┌──────────┐     ┌──────────┐
  │ Data on  │     │ MFT Entry│
  │ disk     │────▶│ Deleted  │
  │ (still!) │     │ marker   │
  └──────────┘     └──────────┘
       │ OVERWRITE
  ┌──────────┐     ┌──────────┐
  │ New data │     │ MFT Entry│
  │ written  │     │ Reused   │
  └──────────┘     └──────────┘

RECOVERY METHODS:
  File System Based:
  → Parse MFT for deleted entries
  → Recover using file system metadata
  → Maintains file names and timestamps

  Data Carving:
  → Search unallocated space for file signatures
  → Header/footer matching
  → Works even without file system metadata
  → May not recover file names

FILE SIGNATURES (Magic Bytes):
  Type  | Header (Hex)        | Footer
  JPEG  | FF D8 FF             | FF D9
  PNG   | 89 50 4E 47          | 
  PDF   | 25 50 44 46          | %%EOF
  ZIP   | 50 4B 03 04          | 
  DOCX  | 50 4B 03 04          | (ZIP-based)
  EXE   | 4D 5A                |
  SQLITE| 53 51 4C 69 74 65    |

CARVING TOOLS:
  → Scalpel: Fast file carver
  → Foremost: File recovery
  → PhotoRec: Multi-format recovery
  → Autopsy: Built-in carving module

SSD CONSIDERATIONS:
  → TRIM command erases deleted data
  → Recovery much harder on SSDs
  → Over-provisioning may retain data
  → Wear leveling complicates analysis
  → Time-critical: image ASAP
```

---

## 5. Timeline Analysis from Disk

```
SUPER TIMELINE FROM DISK:

USING PLASO (log2timeline):
  → Parses 100+ artifact types
  → Creates unified timeline
  → Millions of events possible

  # Create timeline from disk image
  log2timeline.py /evidence/timeline.plaso 
    /evidence/disk.E01

  # Filter and output
  psort.py -o l2tcsv /evidence/timeline.plaso 
    "date > '2024-01-10' AND date < '2024-01-20'"
    -w /evidence/filtered_timeline.csv

TIMELINE SOURCES FROM DISK:
  Source              | Events
  $MFT timestamps    | File create, modify, access
  $UsnJrnl           | File changes
  Prefetch           | Program execution times
  Event logs         | System events
  Registry timestamps| Key modification times
  Browser history    | Web browsing times
  LNK files          | File access times
  Jump Lists         | Application/file access

ANALYSIS APPROACH:
  1. Create super timeline
  2. Filter to timeframe of interest
  3. Identify key events (execution, access)
  4. Build narrative of user activity
  5. Correlate with other evidence sources
  6. Document findings with evidence references
```

---

## Summary Table

| Area | Key Artifacts | Tools | Evidence |
|------|-------------|-------|---------|
| File System | MFT, Journal, Bitmap | MFTECmd, Autopsy | File activity |
| Program Execution | Prefetch, Amcache | PECmd, AmcacheParser | What ran |
| File Access | LNK, Jump Lists | LECmd, JLECmd | What was opened |
| User Activity | Registry, Browser | RegRipper, Hindsight | User behavior |
| Deleted Data | Unallocated space | Autopsy, PhotoRec | Recovered files |
| Timeline | All artifacts | Plaso, Timeline Explorer | Event sequence |

---

## Revision Questions

1. What key structures in NTFS are forensically significant?
2. What Windows artifacts indicate program execution?
3. How can deleted files be recovered from NTFS?
4. What challenges does SSD TRIM present for forensic recovery?
5. How is a super timeline created from disk artifacts?

---

*Previous: [03-chain-of-custody.md](03-chain-of-custody.md) | Next: [05-memory-forensics-basics.md](05-memory-forensics-basics.md)*

---

*[Back to README](../README.md)*
