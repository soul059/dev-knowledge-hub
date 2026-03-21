# Unit 9: Digital Forensics Basics — Topic 2: Evidence Handling

## Overview

**Evidence handling** encompasses the procedures for acquiring, processing, storing, and managing digital evidence to maintain its integrity and admissibility. Proper evidence handling ensures that findings can withstand legal scrutiny and that original evidence is never altered during investigation.

---

## 1. Evidence Acquisition

```
ACQUISITION METHODS:

STATIC ACQUISITION (Dead Box):
  → System powered off
  → Drive removed or booted from forensic media
  → Bit-for-bit copy of entire drive
  → Highest integrity
  → No volatile data captured

  Tools:
  → FTK Imager (GUI, free)
  → dd / dc3dd (Linux CLI)
  → Guymager (Linux GUI)
  → Forensic duplicators (hardware)

LIVE ACQUISITION:
  → System powered on
  → Volatile data captured (RAM, processes)
  → Must minimize changes to system
  → Run tools from external media
  → Document all actions

  Tools:
  → DumpIt (memory)
  → WinPmem (memory)
  → FTK Imager Lite (disk, portable)
  → Velociraptor (remote, targeted)
  → KAPE (targeted artifact collection)

REMOTE ACQUISITION:
  → Network-based collection
  → Agent-based (EDR, Velociraptor)
  → Useful for cloud and distributed systems
  → Must verify integrity

IMAGE FORMATS:
  Format | Description          | Features
  RAW    | Bit-for-bit copy     | Universal, large
  E01    | EnCase format        | Compression, metadata
  AFF    | Advanced Forensic    | Open format
  VMDK   | Virtual machine disk | For VM forensics
  VHD    | Virtual hard disk    | Microsoft format

ACQUISITION COMMANDS:
  # dd (Linux)
  dd if=/dev/sda of=/evidence/disk.raw bs=4M 
    status=progress

  # dc3dd (enhanced dd with hashing)
  dc3dd if=/dev/sda of=/evidence/disk.raw 
    hash=sha256 log=/evidence/acquisition.log

  # FTK Imager CLI
  ftkimager \\.\PhysicalDrive0 
    /evidence/disk --e01 --compress 6 
    --verify

  # Guymager
  # GUI: Select drive → Acquire → E01 format
  # Auto-verifies hash
```

---

## 2. Hash Verification

```
HASH INTEGRITY VERIFICATION:

PURPOSE:
  → Prove evidence has not been altered
  → Verify forensic copy matches original
  → Maintain admissibility
  → Detect corruption or tampering

HASH ALGORITHMS:
  Algorithm | Length   | Status
  MD5       | 128 bit | Legacy (still used in forensics)
  SHA-1     | 160 bit | Deprecated for security
  SHA-256   | 256 bit | Current standard
  SHA-512   | 512 bit | Maximum security

BEST PRACTICE: Use at least two hash algorithms
  → MD5 (backward compatibility)
  → SHA-256 (current standard)

HASH VERIFICATION WORKFLOW:
  Step 1: Hash original drive
  → sha256sum /dev/sda > original_hash.txt
  
  Step 2: Create forensic copy
  → dc3dd if=/dev/sda of=/evidence/disk.raw 
      hash=sha256
  
  Step 3: Hash forensic copy
  → sha256sum /evidence/disk.raw > copy_hash.txt
  
  Step 4: Compare hashes
  → diff original_hash.txt copy_hash.txt
  → Must match EXACTLY

  Step 5: Document
  → Record both hashes
  → Note date, time, tool
  → Include in chain of custody

HASH TOOLS:
  # Windows
  Get-FileHash -Path C:\evidence\disk.raw `
    -Algorithm SHA256
  certutil -hashfile C:\evidence\disk.raw SHA256

  # Linux
  sha256sum /evidence/disk.raw
  md5sum /evidence/disk.raw

  # Multiple hashes
  md5deep -r /evidence/
  sha256deep -r /evidence/
```

---

## 3. Evidence Storage

```
EVIDENCE STORAGE REQUIREMENTS:

PHYSICAL STORAGE:
  → Evidence locker/safe
  → Access controls (key/combination)
  → Temperature and humidity controlled
  → Fire protection
  → Tamper-evident bags/containers
  → Labeled with case information
  → Logged access

DIGITAL STORAGE:
  → Encrypted drives (AES-256)
  → Access controls (role-based)
  → Audit logging
  → Redundant storage (RAID)
  → Offsite backup
  → Integrity monitoring
  → Adequate capacity

STORAGE MEDIA:
  Media            | Use Case          | Lifespan
  External HDD     | Case evidence     | 3-5 years
  NAS/SAN          | Active cases      | 5-10 years
  LTO Tape         | Long-term archive | 15-30 years
  Cloud (encrypted)| Remote storage    | As subscribed
  Optical (BD-R)   | Archive           | 25-50 years

LABELING:
  ┌──────────────────────────────────┐
  │ EVIDENCE LABEL                   │
  │                                  │
  │ Case: INC-2024-001              │
  │ Evidence ID: EVD-003            │
  │ Description: Disk image -       │
  │   JSMITH-PC                     │
  │ Format: E01                     │
  │ Size: 256 GB                    │
  │ SHA256: a1b2c3d4...            │
  │ Collected by: Jane Doe         │
  │ Date: 2024-01-15               │
  │ Storage: Evidence Locker B-3   │
  └──────────────────────────────────┘
```

---

## 4. Evidence Processing

```
EVIDENCE PROCESSING WORKFLOW:

INTAKE:
  → Receive evidence with chain of custody
  → Verify hash values
  → Log into evidence tracking system
  → Store in evidence locker
  → Create working copy for examination

CREATING WORKING COPIES:
  → Hash original evidence
  → Create copy for examination
  → Hash working copy
  → Verify hashes match
  → Work ONLY on the copy
  → Never modify original

  # Create working copy
  cp /evidence/originals/disk.raw \
     /evidence/working/disk_working.raw
  
  # Verify
  sha256sum /evidence/originals/disk.raw
  sha256sum /evidence/working/disk_working.raw

MOUNTING EVIDENCE:
  → Always mount as READ-ONLY
  → Use forensic tools designed for this

  # Linux: Mount read-only
  mount -o ro,noexec,loop /evidence/disk.raw /mnt/evidence
  
  # Autopsy: Import as data source
  # → Case > Add Data Source > Disk Image
  
  # FTK: Add evidence item
  # → File > Add Evidence Item > Image

PROCESSING PIPELINE:
  1. Import into forensic tool
  2. File system analysis
  3. File recovery (deleted files)
  4. Hash all files
  5. Search for known bad (hash sets)
  6. Keyword search
  7. Artifact extraction
  8. Timeline generation
  9. Report generation
```

---

## 5. Evidence Retention and Disposal

```
RETENTION POLICY:

RETENTION PERIODS:
  Case Type         | Minimum Retention
  Active incident   | Until case closure + 3 years
  Criminal          | As directed by law enforcement
  Civil litigation  | Until litigation resolved + 7 years
  Regulatory        | Per regulatory requirement
  Employee          | Per HR policy (typically 3-7 years)
  General IR        | 1-3 years post-closure

RETENTION CONSIDERATIONS:
  → Legal hold requirements
  → Regulatory mandates
  → Insurance requirements
  → Statute of limitations
  → Organizational policy
  → Storage costs
  → Privacy regulations (GDPR right to erasure)

SECURE DISPOSAL:
  When retention period expires:
  → Verify no active legal holds
  → Obtain authorization for disposal
  → Use secure deletion methods
  → Document disposal

  Disposal Methods:
  Method           | Data Type    | Standard
  Secure erase     | HDD/SSD     | NIST SP 800-88
  Degaussing       | HDD/tape    | NIST SP 800-88
  Physical destroy | All media   | DoD 5220.22-M
  Crypto erase     | Encrypted   | Destroy keys

  Documentation:
  → Evidence ID disposed
  → Method of disposal
  → Date of disposal
  → Authorized by
  → Performed by
  → Witness (if required)
```

---

## Summary Table

| Phase | Activities | Key Requirement |
|-------|-----------|----------------|
| Acquisition | Create forensic image | Write blocker, proper tool |
| Verification | Hash and compare | Dual hash (MD5 + SHA256) |
| Storage | Secure, labeled, logged | Encryption, access control |
| Processing | Work on copies only | Read-only mount |
| Retention | Follow policy, legal holds | Documentation |
| Disposal | Secure, authorized, documented | NIST SP 800-88 |

---

## Revision Questions

1. What is the difference between static and live acquisition?
2. Why should dual hash algorithms be used for verification?
3. What are the requirements for evidence storage?
4. Why must examiners always work on copies, never originals?
5. What factors determine evidence retention periods?

---

*Previous: [01-forensic-methodology.md](01-forensic-methodology.md) | Next: [03-chain-of-custody.md](03-chain-of-custody.md)*

---

*[Back to README](../README.md)*
