# Unit 9: Digital Forensics Basics — Topic 1: Forensic Methodology

## Overview

**Digital forensics methodology** provides a systematic, repeatable approach to investigating digital evidence. Following a structured methodology ensures evidence integrity, supports legal proceedings, and produces reliable findings. This topic covers the standard forensic process from identification through reporting.

---

## 1. Forensic Process Model

```
DIGITAL FORENSIC PROCESS:

  ┌──────────────┐
  │ IDENTIFICATION│ What needs to be investigated?
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │ PRESERVATION │ Protect evidence integrity
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │ COLLECTION   │ Acquire evidence properly
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │ EXAMINATION  │ Process and search evidence
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │ ANALYSIS     │ Interpret findings
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │ REPORTING    │ Document and present findings
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │ PRESENTATION │ Expert testimony if needed
  └──────────────┘

NIST SP 800-86 FRAMEWORK:
  Phase 1: Collection
  → Identify and acquire data
  → Preserve evidence integrity
  → Document all actions

  Phase 2: Examination
  → Process collected data
  → Extract relevant information
  → Use validated tools

  Phase 3: Analysis
  → Interpret extracted data
  → Correlate findings
  → Draw conclusions

  Phase 4: Reporting
  → Document methods and findings
  → Present to stakeholders
  → Maintain for legal proceedings
```

---

## 2. Forensic Principles

```
CORE FORENSIC PRINCIPLES:

  1. EVIDENCE INTEGRITY
     → Never modify original evidence
     → Work on forensic copies
     → Verify with hash values
     → Use write blockers
     → Document all tools and methods

  2. CHAIN OF CUSTODY
     → Track evidence from collection to court
     → Document every transfer
     → Secure storage
     → Limited access
     → Complete documentation

  3. REPRODUCIBILITY
     → Another examiner should reach same results
     → Document every step
     → Use validated tools
     → Record all parameters

  4. DOCUMENTATION
     → Record every action taken
     → Include timestamps
     → Note tools and versions
     → Document findings
     → Maintain examination notes

  5. COMPETENCY
     → Use validated and accepted tools
     → Maintain training and certifications
     → Stay current with technology
     → Understand legal requirements
     → Know tool limitations

FORENSIC READINESS:
  → Have forensic tools ready and updated
  → Maintain trained personnel
  → Have acquisition media prepared
  → Know legal authority and requirements
  → Have forensic workstation available
  → Maintain write blockers
  → Have storage capacity available
```

---

## 3. Forensic Lab and Workstation

```
FORENSIC WORKSTATION:

HARDWARE REQUIREMENTS:
  → High-performance CPU (multi-core)
  → 64+ GB RAM (for memory analysis)
  → Large storage (multiple TB)
  → Write blockers (hardware)
  → Forensic duplicator
  → Multiple drive interfaces
  → Network isolated capability
  → UPS (uninterruptible power)

SOFTWARE TOOLKIT:
  Category          | Tools
  Disk Forensics    | FTK Imager, EnCase, Autopsy
  Memory Forensics  | Volatility, Rekall
  Network Forensics | Wireshark, NetworkMiner
  Timeline          | Plaso/log2timeline
  Registry          | RegRipper, Registry Explorer
  File Analysis     | ExifTool, PE tools
  Hashing           | HashCalc, md5deep
  Artifact Parse    | KAPE, Eric Zimmerman tools
  Mobile            | Cellebrite, Oxygen
  Malware Analysis  | IDA, Ghidra, Any.Run

FORENSIC DISTRIBUTIONS:
  → SIFT Workstation (SANS)
  → CAINE (Computer Aided INvestigative Environment)
  → Kali Linux (penetration testing)
  → Tsurugi Linux (forensics focused)
  → Paladin (forensic boot environment)

LAB REQUIREMENTS:
  → Physical security (locked room)
  → Access controls (limited personnel)
  → Evidence storage (secure cabinets)
  → Network isolation capability
  → Documented procedures
  → Tool validation records
  → Calibration records
```

---

## 4. Types of Forensic Investigations

```
INVESTIGATION TYPES:

  Type              | Focus                | Trigger
  Incident Response | Active breach        | Security alert
  Criminal          | Law violation        | Law enforcement
  Civil Litigation  | Legal dispute        | Court order
  Employee          | Policy violation     | HR/management
  Compliance        | Regulatory           | Audit finding
  Intelligence      | Threat research      | Proactive

INVESTIGATION SCOPE:
  Type              | Evidence Types       | Authority
  IR Forensics      | Logs, memory, disk   | Company policy
  Criminal          | Full system, mobile  | Warrant/consent
  Civil             | Specific documents   | Court order
  Employee          | Company systems      | Acceptable use
  eDiscovery        | Email, documents     | Legal hold

LEGAL CONSIDERATIONS:
  → Understand legal authority
  → Obtain proper authorization
  → Know privacy laws
  → Preserve attorney-client privilege
  → Follow rules of evidence
  → Maintain admissibility standards
  → Know jurisdiction requirements
```

---

## 5. Forensic Workflow Example

```
INVESTIGATION WORKFLOW:

SCENARIO: Suspected data theft by departing employee

  DAY 1: IDENTIFICATION & PRESERVATION
  → Receive investigation request from HR
  → Review scope: Employee laptop + email
  → Issue legal hold for email
  → Preserve laptop (do not return to employee)
  → Document initial request

  DAY 2: COLLECTION
  → Image laptop hard drive (FTK Imager)
  → Collect email archive
  → Calculate hashes of images
  → Complete chain of custody forms
  → Store original evidence securely

  DAY 3-5: EXAMINATION
  → Load disk image into Autopsy
  → Extract browser history
  → Extract USB device history
  → Extract file access history
  → Extract deleted files
  → Extract email attachments
  → Parse Windows artifacts (Prefetch, LNK, etc.)
  → Create super timeline

  DAY 5-7: ANALYSIS
  → Analyze timeline for suspicious activity
  → Identify files copied to USB
  → Identify cloud uploads
  → Identify email forwards to personal account
  → Correlate with business data classification
  → Determine what was taken and when

  DAY 7-10: REPORTING
  → Write forensic report
  → Include evidence summary
  → Document methodology
  → Present findings to legal/HR
  → Preserve evidence for potential litigation

DOCUMENTATION AT EACH STEP:
  → Date and time of action
  → Who performed the action
  → Tool used and version
  → Results obtained
  → Hash values verified
  → Photos of physical evidence
```

---

## Summary Table

| Phase | Activities | Key Requirement |
|-------|-----------|----------------|
| Identification | Define scope, identify evidence | Authorization |
| Preservation | Protect evidence integrity | Write blockers, hashing |
| Collection | Acquire forensic copies | Chain of custody |
| Examination | Process and extract data | Validated tools |
| Analysis | Interpret findings | Expertise, methodology |
| Reporting | Document and present | Clear, factual writing |

---

## Revision Questions

1. What are the phases of the digital forensic process?
2. What core principles must guide forensic investigations?
3. What hardware and software does a forensic workstation need?
4. What types of forensic investigations exist?
5. Why is documentation critical at every step of the process?

---

*Previous: None (First topic in this unit) | Next: [02-evidence-handling.md](02-evidence-handling.md)*

---

*[Back to README](../README.md)*
