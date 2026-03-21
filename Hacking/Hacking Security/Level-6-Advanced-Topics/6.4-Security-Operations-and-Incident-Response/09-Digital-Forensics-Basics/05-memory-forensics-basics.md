# Unit 9: Digital Forensics Basics — Topic 5: Memory Forensics Basics

## Overview

**Memory forensics** analyzes the contents of a system's RAM (Random Access Memory) to uncover evidence that exists only in volatile memory. This includes running processes, network connections, encryption keys, injected code, and malware that never touches disk. Memory analysis is essential for detecting sophisticated threats that operate entirely in memory.

---

## 1. Why Memory Forensics

```
EVIDENCE FOUND ONLY IN MEMORY:

  → Running processes (including hidden)
  → Open network connections
  → Encryption keys (BitLocker, TrueCrypt)
  → Decrypted data
  → User passwords/credentials
  → Injected malicious code
  → Command history
  → Clipboard contents
  → Chat/messaging content
  → Fileless malware
  → Process hollowing artifacts
  → DLL injection evidence
  → Rootkit hooks

WHEN TO USE MEMORY FORENSICS:
  → Suspected fileless malware
  → Process injection detected
  → Rootkit suspected
  → Encryption key recovery needed
  → Live system investigation
  → Incident response (first step)
  → Volatile evidence preservation
  → Anti-forensics suspected on disk

MEMORY vs DISK:
  Aspect          | Memory        | Disk
  Volatility      | Very high     | Low
  Evidence type   | Runtime state | Persistent data
  Malware         | Fileless, injected| Files on disk
  Credentials     | Plaintext     | Hashed/encrypted
  Network state   | Active conns  | Historical logs
  Recovery        | Must capture live| Survives reboot
```

---

## 2. Memory Acquisition

```
MEMORY ACQUISITION TOOLS:

WINDOWS:
  DumpIt:
  → Simple double-click execution
  → Creates raw memory dump
  → Minimal footprint
  → Output: hostname_date.raw

  WinPmem:
  → winpmem_mini_x64.exe memdump.raw
  → Supports multiple output formats
  → Can include pagefile
  → Open source

  FTK Imager:
  → File > Capture Memory
  → GUI-based, easy to use
  → Include pagefile option
  → Industry standard

  Belkasoft RAM Capturer:
  → Lightweight, free
  → Supports Windows 10/11
  → Minimal artifact

LINUX:
  LiME (Linux Memory Extractor):
  → Kernel module
  → insmod lime.ko "path=/evidence/mem.lime format=lime"
  → Minimal footprint
  → Supports raw and lime formats

  /proc/kcore:
  → dd if=/proc/kcore of=memdump.raw
  → Not always complete

REMOTE ACQUISITION:
  Velociraptor:
  → Remote memory acquisition
  → Enterprise scale
  → Centralized collection

  CrowdStrike RTR:
  → memdump command
  → Collected through agent
  → Cloud-stored

ACQUISITION BEST PRACTICES:
  [ ] Run tool from external media (USB)
  [ ] Minimize running processes
  [ ] Note system date/time
  [ ] Record tool version
  [ ] Hash memory dump immediately
  [ ] Document acquisition method
  [ ] Include pagefile if possible
  [ ] Acquire BEFORE shutting down
```

---

## 3. Volatility Framework

```
VOLATILITY — PRIMARY MEMORY ANALYSIS TOOL:

VOLATILITY 2 (Python 2):
  → Legacy but well-documented
  → Large plugin library
  → Extensive community support

VOLATILITY 3 (Python 3):
  → Current version
  → Improved performance
  → Auto profile detection
  → Modern architecture

BASIC USAGE (Volatility 3):
  # List processes
  vol.py -f memdump.raw windows.pslist

  # Process tree
  vol.py -f memdump.raw windows.pstree

  # Hidden processes (compare pslist vs psscan)
  vol.py -f memdump.raw windows.psscan

  # Network connections
  vol.py -f memdump.raw windows.netscan

  # DLLs loaded by process
  vol.py -f memdump.raw windows.dlllist --pid 1234

  # Command line arguments
  vol.py -f memdump.raw windows.cmdline

  # Environment variables
  vol.py -f memdump.raw windows.envars

  # Registry hives
  vol.py -f memdump.raw windows.registry.hivelist

  # File handles
  vol.py -f memdump.raw windows.handles --pid 1234

VOLATILITY 2 COMMANDS (legacy):
  # Identify profile
  vol.py -f memdump.raw imageinfo
  
  # List processes
  vol.py -f memdump.raw --profile=Win10x64 pslist
  
  # Network connections
  vol.py -f memdump.raw --profile=Win10x64 netscan
  
  # Malware detection
  vol.py -f memdump.raw --profile=Win10x64 malfind
```

---

## 4. Memory Analysis Techniques

```
ANALYSIS TECHNIQUES:

PROCESS ANALYSIS:
  Goal: Identify suspicious processes

  # Compare pslist vs psscan
  # pslist: Walks active process list (can be hidden)
  # psscan: Scans all memory for process structures
  
  Suspicious indicators:
  → Process not in pslist but in psscan (hidden)
  → Unexpected parent-child relationships
  → Misspelled system processes (svchost → svch0st)
  → System processes from wrong path
  → Unusual command line arguments
  → Process running from temp/user directories
  → Multiple instances of unique processes

  Normal Process Hierarchy:
  System (4)
  └── smss.exe
      └── csrss.exe
      └── wininit.exe
          └── services.exe
              └── svchost.exe (multiple)
              └── spoolsv.exe
          └── lsass.exe
      └── winlogon.exe
          └── explorer.exe
              └── user applications

CODE INJECTION DETECTION:
  # Find injected code
  vol.py -f memdump.raw windows.malfind

  malfind looks for:
  → Memory regions with execute permission
  → Not backed by a file on disk
  → Contains executable code (MZ header, shellcode)
  → Common in process injection attacks

  Injection Types:
  → Classic DLL injection
  → Process hollowing
  → Reflective DLL loading
  → AtomBombing
  → Process Doppelganging
  → Thread execution hijacking

CREDENTIAL EXTRACTION:
  # Dump password hashes
  vol.py -f memdump.raw windows.hashdump

  # LSA secrets
  vol.py -f memdump.raw windows.lsadump

  # Mimikatz-style (using mimikatz plugin)
  # Can extract plaintext passwords from memory

NETWORK ANALYSIS:
  # Active and closed connections
  vol.py -f memdump.raw windows.netscan

  Look for:
  → Connections to known C2 IPs
  → Unusual ports
  → Connections from suspicious processes
  → Listening ports on unusual numbers
  → Connections to external IPs from services
```

---

## 5. Memory Forensics Workflow

```
MEMORY ANALYSIS WORKFLOW:

  ┌────────────────────────────────────────────┐
  │     MEMORY FORENSICS WORKFLOW              │
  │                                            │
  │  1. ACQUIRE MEMORY                         │
  │     DumpIt/WinPmem → memdump.raw          │
  │     Hash the dump                          │
  │                                            │
  │  2. IDENTIFY OS PROFILE                    │
  │     Volatility auto-detects (v3)           │
  │     Or: vol.py imageinfo (v2)              │
  │                                            │
  │  3. PROCESS ANALYSIS                       │
  │     pslist → pstree → psscan               │
  │     Compare: hidden processes?             │
  │     Check names, paths, parents            │
  │                                            │
  │  4. NETWORK ANALYSIS                       │
  │     netscan → connections                  │
  │     Identify C2, unusual connections       │
  │                                            │
  │  5. MALWARE DETECTION                      │
  │     malfind → injected code                │
  │     Dump suspicious processes              │
  │     Submit to VT or sandbox                │
  │                                            │
  │  6. CREDENTIAL ANALYSIS                    │
  │     hashdump → password hashes             │
  │     Check for credential theft evidence    │
  │                                            │
  │  7. ADDITIONAL ANALYSIS                    │
  │     Registry, handles, DLLs               │
  │     Strings, YARA scanning                 │
  │                                            │
  │  8. REPORT FINDINGS                        │
  │     Document methodology                   │
  │     List evidence found                    │
  │     Include IOCs discovered                │
  └────────────────────────────────────────────┘

YARA SCANNING IN MEMORY:
  # Scan memory with YARA rules
  vol.py -f memdump.raw windows.vadyarascan 
    --yara-file malware_rules.yar

  # Dump suspicious process for analysis
  vol.py -f memdump.raw windows.dumpfiles 
    --pid 1234

STRINGS ANALYSIS:
  # Extract strings from memory dump
  strings -el memdump.raw > strings_unicode.txt
  strings memdump.raw > strings_ascii.txt
  
  # Search for interesting patterns
  grep -i "password" strings_unicode.txt
  grep -i "http://" strings_ascii.txt
  grep -E "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" strings_ascii.txt
```

---

## Summary Table

| Analysis Area | Volatility Plugin | What It Finds |
|--------------|------------------|--------------|
| Processes | pslist, pstree, psscan | Running/hidden processes |
| Network | netscan | Active connections |
| Injection | malfind | Injected code |
| Credentials | hashdump, lsadump | Password hashes |
| DLLs | dlllist | Loaded libraries |
| Registry | hivelist, printkey | Registry contents |
| Files | dumpfiles, filescan | Files in memory |

---

## Revision Questions

1. What evidence exists only in memory and cannot be found on disk?
2. What tools are used for Windows memory acquisition?
3. How does Volatility's pslist differ from psscan for detecting hidden processes?
4. What does the malfind plugin detect?
5. What is the recommended workflow for memory forensics analysis?

---

*Previous: [04-disk-forensics-basics.md](04-disk-forensics-basics.md) | Next: [06-network-forensics-basics.md](06-network-forensics-basics.md)*

---

*[Back to README](../README.md)*
