# Unit 9: Digital Forensics Basics — Topic 3: Chain of Custody

## Overview

**Chain of custody** is the documented, unbroken record of who handled evidence, when, and why. It tracks evidence from the moment of collection through analysis, storage, and potential court presentation. A broken chain of custody can render evidence inadmissible, undermining the entire investigation.

---

## 1. Chain of Custody Fundamentals

```
CHAIN OF CUSTODY PRINCIPLE:

  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │COLLECTION│─▶│ TRANSFER │─▶│ STORAGE  │
  └──────────┘  └──────────┘  └──────────┘
       │              │              │
  Who collected  Who received   Where stored
  When           When           Access log
  Where          Why            Secure?
  How            Condition      Tamper-proof?
       │              │              │
  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │ ANALYSIS │─▶│ RETURN   │─▶│DISPOSAL/ │
  │          │  │          │  │COURT     │
  └──────────┘  └──────────┘  └──────────┘
       │              │              │
  Who analyzed   Returned to   Final disposition
  When           storage       Documented
  What tools     Condition     Authorized

KEY REQUIREMENTS:
  → Every transfer documented
  → No gaps in the record
  → Evidence condition noted at each transfer
  → Hash values verified at each transfer
  → Secure storage between transfers
  → Limited number of handlers
  → Tamper-evident packaging
```

---

## 2. Chain of Custody Documentation

```
CHAIN OF CUSTODY FORM:

  ┌──────────────────────────────────────────────────┐
  │ CHAIN OF CUSTODY FORM                            │
  │                                                  │
  │ CASE INFORMATION:                                │
  │ Case Number: INC-2024-001                       │
  │ Case Name: Data Breach Investigation             │
  │ Investigating Agency: Company SOC               │
  │ Lead Investigator: Jane Doe                      │
  │                                                  │
  │ EVIDENCE INFORMATION:                            │
  │ Evidence ID: EVD-001                            │
  │ Description: Dell Latitude 5520 Laptop          │
  │ Serial Number: DELL-SN-12345                    │
  │ Model: Latitude 5520                            │
  │ Condition: Good, powered off                     │
  │                                                  │
  │ ASSOCIATED DIGITAL EVIDENCE:                     │
  │ Evidence ID: EVD-001-IMG                        │
  │ Description: Forensic disk image (E01)          │
  │ File: JSMITH-PC_20240115.E01                    │
  │ MD5: d41d8cd98f00b204e9800998ecf8427e          │
  │ SHA256: e3b0c44298fc1c149afbf4c8996fb924...     │
  │ Size: 256 GB                                    │
  │ Tool: FTK Imager v4.7.1                        │
  │                                                  │
  │ CUSTODY LOG:                                     │
  │ # | Date/Time   | From    | To      | Purpose   │
  │ 1 | 01/15 11:30 | Scene   | J. Doe  | Collection│
  │ 2 | 01/15 12:00 | J. Doe  | Locker  | Storage   │
  │ 3 | 01/16 09:00 | Locker  | M. Smith| Analysis  │
  │ 4 | 01/16 17:00 | M. Smith| Locker  | Return    │
  │ 5 | 01/20 10:00 | Locker  | J. Doe  | Review    │
  │ 6 | 01/20 15:00 | J. Doe  | Locker  | Return    │
  │                                                  │
  │ SIGNATURES:                                      │
  │ Each transfer requires signatures from            │
  │ both releasing and receiving parties.             │
  │                                                  │
  │ Transfer 1: _______ (From) _______ (To)         │
  │ Transfer 2: _______ (From) _______ (To)         │
  │ [continues for each transfer]                    │
  └──────────────────────────────────────────────────┘
```

---

## 3. Evidence Collection Documentation

```
COLLECTION DOCUMENTATION:

AT TIME OF COLLECTION, RECORD:
  → Date and time (UTC)
  → Location (building, room, desk)
  → Who is present
  → System state (on/off/locked)
  → What was collected
  → How it was collected
  → Tool(s) used
  → Hash values computed
  → Photographs taken
  → Serial numbers recorded
  → Condition of item

PHOTOGRAPH REQUIREMENTS:
  → Overall scene / workspace
  → Evidence in original location
  → Close-up of evidence (serial numbers)
  → Screen state (if powered on)
  → Connected cables and devices
  → Any labels or identifying marks
  → Include date/time reference

COLLECTION NOTES EXAMPLE:
  ┌──────────────────────────────────────────────┐
  │ COLLECTION NOTES                             │
  │                                              │
  │ Date: 2024-01-15 11:30 UTC                  │
  │ Location: Building A, Floor 3, Desk 42      │
  │ Collector: Jane Doe, Security Analyst        │
  │ Witness: John Williams, IT Manager           │
  │                                              │
  │ Scene Description:                           │
  │ Dell laptop found powered on at employee     │
  │ desk. Screen locked. Connected to docking    │
  │ station with monitor, keyboard, mouse.       │
  │ External USB drive connected to dock.        │
  │                                              │
  │ Actions Taken:                               │
  │ 1. Photographed scene (6 photos)            │
  │ 2. Noted system state: powered on, locked   │
  │ 3. Collected RAM using DumpIt               │
  │ 4. Performed clean shutdown                 │
  │ 5. Disconnected from dock                   │
  │ 6. Recorded serial number                   │
  │ 7. Placed in anti-static bag               │
  │ 8. Sealed with evidence tape (signed)       │
  │ 9. Also seized USB drive (Kingston 32GB)    │
  │ 10. Transported to evidence room            │
  │                                              │
  │ Hash values computed in evidence room:       │
  │ Disk image: [hash recorded separately]       │
  │ Memory dump: [hash recorded separately]      │
  └──────────────────────────────────────────────┘
```

---

## 4. Maintaining Chain of Custody

```
MAINTAINING INTEGRITY:

PHYSICAL EVIDENCE:
  → Use tamper-evident bags
  → Seal with evidence tape
  → Sign across tape/seal
  → Label with case info
  → Store in locked evidence room
  → Log all access
  → Re-verify seals when accessed

DIGITAL EVIDENCE:
  → Hash upon collection
  → Hash after each copy/transfer
  → Store on encrypted media
  → Access controls on evidence shares
  → Audit logging for access
  → Write-protect when possible
  → Verify hash before analysis

COMMON CHAIN OF CUSTODY BREAKS:
  ✗ Unsigned transfers
  ✗ Gaps in timeline (unaccounted periods)
  ✗ Unlocked storage
  ✗ No access logs
  ✗ Missing hash verification
  ✗ Evidence left unattended
  ✗ Multiple people handling without docs
  ✗ Evidence accessed without logging

HOW DEFENSE ATTORNEYS ATTACK CoC:
  → "Who had access to this evidence?"
  → "Can you account for every moment?"
  → "How do you know it wasn't altered?"
  → "Where are the hash verification records?"
  → "Who else had access to the storage room?"
  → "Is there surveillance of the evidence room?"

DIGITAL CHAIN OF CUSTODY TOOLS:
  → Evidence management systems
  → Digital signature on evidence
  → Blockchain-based tracking (emerging)
  → Automated hash verification
  → Access-controlled evidence repositories
  → Audit trail for all evidence access
```

---

## 5. Chain of Custody Best Practices

```
BEST PRACTICES:

  [ ] Use standardized forms
  [ ] Fill forms completely (no blank fields)
  [ ] Include both physical and digital evidence
  [ ] Get signatures for every transfer
  [ ] Minimize the number of handlers
  [ ] Store in secure, access-controlled location
  [ ] Verify hashes at every transfer point
  [ ] Photograph evidence at collection
  [ ] Use tamper-evident packaging
  [ ] Maintain continuous documentation
  [ ] Train all evidence handlers
  [ ] Regular audits of evidence storage
  [ ] Backup evidence management records
  [ ] Retain CoC records per policy

FOR INCIDENT RESPONSE (vs law enforcement):
  → May be less formal but still important
  → Maintain basic chain of custody
  → Document who accessed evidence
  → Hash all evidence
  → Secure storage
  → May need to escalate to formal forensics
  → Be prepared for legal proceedings

QUICK REFERENCE — EVERY TRANSFER:
  □ Date and time noted
  □ Releasing party signed
  □ Receiving party signed
  □ Condition noted
  □ Purpose documented
  □ Hash verified (digital)
  □ Seal intact (physical)
```

---

## Summary Table

| Element | Requirement | Purpose |
|---------|------------|---------|
| Collection docs | Who, what, when, where, how | Establish provenance |
| Transfer log | Every handoff documented | Prove continuity |
| Hash values | MD5 + SHA256 at each point | Prove integrity |
| Signatures | Both parties at each transfer | Accountability |
| Secure storage | Locked, logged, controlled | Prevent tampering |
| Photographs | Scene, evidence, labels | Visual documentation |

---

## Revision Questions

1. What information must a chain of custody form contain?
2. What documentation should be created at the time of evidence collection?
3. How can the chain of custody be broken, and what are the consequences?
4. How do digital and physical chain of custody requirements differ?
5. What best practices ensure a defensible chain of custody?

---

*Previous: [02-evidence-handling.md](02-evidence-handling.md) | Next: [04-disk-forensics-basics.md](04-disk-forensics-basics.md)*

---

*[Back to README](../README.md)*
