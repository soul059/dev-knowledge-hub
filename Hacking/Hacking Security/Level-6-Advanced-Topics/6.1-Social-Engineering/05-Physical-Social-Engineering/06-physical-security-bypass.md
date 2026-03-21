# Unit 5: Physical Social Engineering — Topic 6: Physical Security Bypass

## Overview

**Physical security bypass** encompasses techniques used to defeat physical security controls — locks, doors, fences, alarms, cameras, and access control systems — to gain unauthorized entry to facilities. In penetration testing, physical security assessments reveal how well an organization's physical controls protect sensitive areas like server rooms, executive offices, and data centers.

---

## 1. Physical Security Layers

```
DEFENSE IN DEPTH — PHYSICAL:

LAYER 1: PERIMETER
  ┌──────────────────────────────────────┐
  │ Fences, gates, barriers, bollards    │
  │ Perimeter cameras and motion sensors │
  │ Guard posts, vehicle checkpoints     │
  │ Lighting and landscaping             │
  └──────────────────────────────────────┘
              │
LAYER 2: BUILDING EXTERIOR
  ┌──────────────────────────────────────┐
  │ Doors, windows, loading docks        │
  │ Locks and access control systems     │
  │ Alarm systems (intrusion detection)  │
  │ Exterior cameras                     │
  └──────────────────────────────────────┘
              │
LAYER 3: INTERIOR
  ┌──────────────────────────────────────┐
  │ Reception/lobby security             │
  │ Badge readers on internal doors      │
  │ Mantraps for sensitive areas         │
  │ Interior cameras and guards          │
  └──────────────────────────────────────┘
              │
LAYER 4: SENSITIVE AREAS
  ┌──────────────────────────────────────┐
  │ Server room / data center            │
  │ Executive offices                    │
  │ Biometric access controls            │
  │ Cabinet locks, safe rooms            │
  │ Environmental controls               │
  └──────────────────────────────────────┘
```

---

## 2. Lock Bypass Techniques

```
LOCK PICKING:

PIN TUMBLER LOCKS:
  → Most common lock type (doors, cabinets)
  → Use tension wrench + pick
  → Apply slight rotational pressure
  → Lift each pin to shear line
  → Lock opens when all pins set

TOOLS:
  → Lock pick set (hooks, rakes, diamonds)
  → Tension wrenches (various sizes)
  → Electric pick guns
  → Bump keys
  → Snap guns
  → Bypass tools (specific to lock type)

BUMPING:
  → Use specially cut "bump key"
  → Insert and strike with tool
  → Pins jump to shear line
  → Quick and requires less skill
  → Works on most pin tumbler locks

SHIMMING:
  → Bypass padlocks with shim
  → Thin metal slid into shackle
  → Releases locking mechanism
  → Works on many padlocks
  → Very quick technique

BYPASS TECHNIQUES:
  → Credit card latch bypass (spring latches)
  → Under-door tool (lever handles)
  → Through-the-door tools
  → Hinge pin removal (outward-opening doors)
  → Destructive entry (drill, pry — last resort)
```

---

## 3. Electronic Access Control Bypass

```
BADGE/CARD SYSTEMS:

RFID CLONING (125 kHz — Legacy):
  → HID Prox, EM4100 cards
  → Read with Proxmark3 or similar
  → Clone to blank card
  → No encryption — trivially cloned
  → Read range: up to several feet

iCLASS / SEOS (13.56 MHz — Modern):
  → Encrypted communication
  → Mutual authentication
  → Harder but not impossible to clone
  → Known vulnerabilities in older versions
  → Requires specialized equipment

NFC ATTACKS:
  → Use NFC-enabled phone to read cards
  → Proxmark3 for advanced attacks
  → Replay attacks on some systems
  → Relay attacks (extend read range)

KEYPAD BYPASS:
  → Look for worn keys (reveal digits used)
  → UV light reveals fingerprints on keys
  → Thermal imaging shows recently pressed keys
  → Social engineering the code
  → Default codes (1234, 0000, building address)

DOOR CONTROLLER ATTACKS:
  → Wiegand protocol interception
  → Tap into reader wiring
  → Inject valid credentials
  → Controller board tampering
  → Network-connected controllers can be hacked remotely
```

---

## 4. Alarm and Camera Bypass

```
ALARM SYSTEM BYPASS:

MOTION SENSORS (PIR):
  → Move extremely slowly
  → Use materials to block IR signature
  → Crawl below sensor detection zone
  → Know sensor blind spots
  → Sensor placement analysis via recon

DOOR/WINDOW SENSORS:
  → Magnetic reed switches
  → Place magnet near sensor to maintain circuit
  → Some sensors have tamper alarms
  → Bypass wiring (if accessible)

ALARM PANEL:
  → Default codes (installer codes)
  → Technical bypass of communication
  → Cut phone/network line (detected by monitoring)
  → Jam cellular backup signal
  → Social engineer alarm company

CAMERA EVASION:
  → Map camera locations and blind spots
  → IR LED array to blind cameras (at night)
  → Laser pointer to saturate sensor
  → Physical obstruction (spray, cover)
  → Timing (when guards aren't monitoring)
  → Disguise (hat, glasses, mask)
  → Approach from camera blind spots
```

---

## 5. Common Physical Pentest Targets

```
TARGET AREAS:

SERVER ROOM / DATA CENTER:
  Typical Security:
    → Badge + PIN or biometric
    → Mantrap / airlock
    → CCTV with recording
    → Environmental monitoring
  Bypass Methods:
    → Badge cloning + shoulder surf PIN
    → Tailgate through mantrap
    → Vendor impersonation
    → After-hours cleaning crew access

EXECUTIVE OFFICES:
  Typical Security:
    → Key lock or badge reader
    → Often unlocked during business hours
    → May have separate alarm
  Bypass Methods:
    → Visit during business hours
    → Lock picking after hours
    → Cleaning crew impersonation
    → "Meeting with [executive]" pretext

WIRING CLOSETS / MDF/IDF:
  Typical Security:
    → Key lock (often master-keyed)
    → Sometimes unlocked
    → Rarely monitored
  Bypass Methods:
    → Lock picking (simple locks)
    → Master key bypass
    → Telecom technician pretext
    → Often overlooked in security plans

MAILROOM / SHIPPING:
  Typical Security:
    → Minimal access control
    → Delivery personnel regularly enter
  Bypass Methods:
    → Delivery person impersonation
    → Walk in during deliveries
    → Packages as trojan horse for devices
```

---

## 6. Physical Pentest Reporting

```
REPORT ELEMENTS:

FINDINGS FORMAT:
  → Location tested
  → Security controls observed
  → Bypass method used
  → Time to bypass
  → Evidence (photos, videos — authorized only)
  → Risk rating
  → Remediation recommendations

EVIDENCE COLLECTION:
  → Photographs of accessed areas
  → Video of bypass techniques
  → Collected items (documents, badges)
  → Planted "proof" items (business cards)
  → Timestamps for all activities
  → GPS coordinates if relevant
```

---

## Summary Table

| Target | Common Controls | Bypass Method | Difficulty |
|--------|----------------|---------------|-----------|
| Perimeter | Fence, gate | Climb, follow vehicle | Medium |
| Building door | Badge reader | Clone, tailgate | Medium |
| Internal door | Pin tumbler lock | Pick, bump | Low-Medium |
| Server room | Badge + biometric | Social engineering | High |
| Alarm system | PIR, door contacts | Slow movement, magnets | Medium-High |
| Cameras | CCTV | Blind spots, IR flood | Medium |

---

## Revision Questions

1. What are the four layers of physical security defense in depth?
2. What lock bypass techniques are most commonly used in pentesting?
3. How can RFID badge systems be compromised?
4. What methods can bypass alarm systems and cameras?
5. What are the most common physical pentest targets and their bypass methods?

---

*Previous: [05-baiting.md](05-baiting.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
