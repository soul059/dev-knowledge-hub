# Unit 5: Physical Social Engineering — Topic 3: Dumpster Diving

## Overview

**Dumpster diving** is the practice of searching through an organization's trash and recycling to find sensitive information that was improperly disposed of. Despite being one of the oldest information gathering techniques, dumpster diving remains surprisingly effective because many organizations still fail to properly destroy sensitive documents, storage media, and other materials containing valuable data.

---

## 1. What Can Be Found

```
VALUABLE DUMPSTER FINDS:

DOCUMENTS:
  → Organization charts (who's who)
  → Internal memos and emails (printed)
  → Phone directories (names, extensions)
  → Financial reports and budgets
  → Customer lists and records
  → Meeting notes and agendas
  → Policy documents and procedures
  → Printouts with login credentials
  → Sticky notes with passwords

TECHNICAL INFORMATION:
  → Network diagrams
  → System configuration printouts
  → IP address lists
  → Server names and details
  → Software license keys
  → Source code printouts
  → Database schemas
  → API documentation

PERSONAL INFORMATION:
  → Employee records (HR documents)
  → Social Security numbers
  → Medical records
  → Pay stubs and tax forms
  → Performance reviews
  → Resumes of applicants

PHYSICAL ITEMS:
  → Old access badges / key cards
  → USB drives and CDs
  → Hard drives (not wiped)
  → Old phones and laptops
  → Letterhead and forms (for forgery)
  → Company-branded items
  → Expired certificates/licenses

SOCIAL ENGINEERING FUEL:
  → Project names for pretexting
  → Vendor names and contacts
  → Internal terminology and jargon
  → Email formats and addresses
  → Calendars and schedules
  → Travel itineraries
```

---

## 2. Dumpster Diving Methodology

```
DUMPSTER DIVING PROCESS:

PHASE 1: RECONNAISSANCE
  → Locate dumpsters and recycling bins
  → Identify trash collection schedule
  → Determine security around waste areas
  → Note if shredders are used
  → Check for separate paper recycling bins

PHASE 2: TIMING
  → Best time: just before pickup
    (maximum content, one visit)
  → After hours: less observation
  → During rain: fewer people outside
  → Avoid: during business hours
  → Friday afternoon: week's worth of trash

PHASE 3: EXECUTION
  → Wear gloves and appropriate clothing
  → Bring flashlight and bags
  → Focus on paper recycling (highest value)
  → Look for shredded paper (can be reassembled)
  → Check executive/IT area bins specifically
  → Take everything, sort later
  → Photograph items if too much to carry

PHASE 4: ANALYSIS
  → Sort materials by category
  → Identify actionable intelligence
  → Cross-reference findings
  → Build organizational picture
  → Identify pretext material
  → Document for report (if authorized)

LEGALITY NOTE:
  → In the US: generally legal once trash is
    placed at curb or in public dumpster
  → California v. Greenwood (1988): No reasonable
    expectation of privacy in trash
  → May vary by jurisdiction
  → Trespassing to access dumpsters IS illegal
  → Private dumpsters may be different
  → Always verify local laws
  → For pentesting: ensure in scope
```

---

## 3. Shredded Document Recovery

```
SHREDDED PAPER ANALYSIS:

SHREDDER TYPES AND SECURITY:

STRIP-CUT:
  ████████████████  ← Easy to reassemble
  ████████████████    30-40 strips per page
  ████████████████    Can be taped back together
  SECURITY: LOW       Software can assist

CROSS-CUT:
  ██ ██ ██ ██ ██   ← Moderate difficulty
  ██ ██ ██ ██ ██     300-400 pieces per page
  ██ ██ ██ ██ ██     Time-consuming but possible
  SECURITY: MEDIUM    Software assistance helps

MICRO-CUT:
  ▪▪▪▪▪▪▪▪▪▪▪▪▪   ← Very difficult
  ▪▪▪▪▪▪▪▪▪▪▪▪▪     3,000+ pieces per page
  ▪▪▪▪▪▪▪▪▪▪▪▪▪     Practically impossible
  SECURITY: HIGH      to reassemble

REASSEMBLY METHODS:
  → Manual: tape and patience
  → Software-assisted: scan and match algorithms
  → DARPA "Shredder Challenge": demonstrated
    automated reassembly
  → Strip-cut can be reassembled in minutes
  → Cross-cut: hours to days per page
```

---

## 4. Digital Dumpster Diving

```
ELECTRONIC WASTE:

HARD DRIVES:
  → Often discarded without wiping
  → Quick format ≠ data destruction
  → Data recovery tools: Recuva, PhotoRec
  → Professional recovery from damaged drives
  → Full disk encryption prevents recovery

USB DRIVES:
  → Left in recycled keyboards/docks
  → Found in desk drawers of disposed furniture
  → Often contain sensitive files
  → Quick format doesn't erase data

OLD PHONES/TABLETS:
  → Factory reset may not fully wipe
  → Contact lists, emails, photos
  → App data with credentials
  → Call logs and messages

PRINTERS/COPIERS:
  → Internal hard drives store copies
  → Print queues contain documents
  → Fax memory retains received faxes
  → Often sold/donated without wiping

CDs/DVDs:
  → Backup discs
  → Training materials
  → Software installers with license keys
  → Database exports
```

---

## 5. Defending Against Dumpster Diving

```
PREVENTION MEASURES:

DOCUMENT DESTRUCTION:
  → Cross-cut or micro-cut shredders
  → Shred-all policy (not just "sensitive" docs)
  → Locked shred bins throughout facility
  → Contracted destruction services
  → Certificate of destruction for sensitive materials

ELECTRONIC MEDIA:
  → NIST 800-88 media sanitization guidelines
  → Degaussing for magnetic media
  → Physical destruction (crushing, shredding)
  → Encrypted drives (unreadable without key)
  → Asset tracking for all storage media

PHYSICAL SECURITY:
  → Locked dumpsters and recycling bins
  → Dumpsters in fenced/secured areas
  → CCTV on waste disposal areas
  → Key/badge required for waste areas
  → Shred bins emptied by cleared personnel

POLICY:
  → Clear desk policy (nothing left out)
  → Data classification and handling procedures
  → Secure disposal training for all employees
  → Regular audits of disposal practices
  → Electronic-only documents where possible
```

---

## Summary Table

| Item Found | Value for Attacker | Prevention |
|-----------|-------------------|------------|
| Org charts | Map target structure | Shred |
| Passwords on paper | Direct access | Shred + policy |
| Network diagrams | Plan network attacks | Shred + classify |
| Old hard drives | Data recovery | Degauss/destroy |
| Letterhead | Forge documents | Shred |
| Phone directories | Vishing campaigns | Shred + digital only |

---

## Revision Questions

1. What types of information can be recovered through dumpster diving?
2. Is dumpster diving legal? What are the considerations?
3. How do different shredder types affect document security?
4. What electronic waste poses the greatest security risk?
5. What comprehensive defense prevents dumpster diving success?

---

*Previous: [02-impersonation.md](02-impersonation.md) | Next: [04-shoulder-surfing.md](04-shoulder-surfing.md)*

---

*[Back to README](../README.md)*
