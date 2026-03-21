# Physical Security Reconnaissance

## Unit 7 - Topic 5 | Social Engineering Reconnaissance

---

## Overview

**Physical security reconnaissance** gathers intelligence about a target's physical facilities — building layout, access controls, security personnel, surveillance systems, and entry/exit procedures. This intelligence supports physical penetration testing and tailgating attacks.

---

## 1. Physical Reconnaissance Targets

```
PHYSICAL SECURITY ASSESSMENT AREAS:

PERIMETER SECURITY:
├── Fencing type and height
├── Gates (vehicle and pedestrian)
├── Lighting (coverage, blind spots)
├── Landscaping (concealment options)
├── Signage (company name, warnings)
└── Perimeter cameras

BUILDING ACCESS:
├── Main entrance (reception, guard)
├── Side/rear entrances
├── Parking garage access
├── Loading dock / delivery area
├── Smoking area (social engineering spot)
├── Emergency exits (alarmed? propped open?)
└── Roof access

ACCESS CONTROLS:
├── Badge readers (HID, proximity, smart card)
├── PIN pads (observe shoulder surfing)
├── Biometric (fingerprint, facial)
├── Turnstiles / mantrap
├── Visitor management system
└── Tailgating prevention measures

SURVEILLANCE:
├── CCTV camera locations and coverage
├── Camera types (PTZ, fixed, dome)
├── Blind spots
├── Monitoring (live vs recorded only)
├── Alarm systems
└── Motion sensors
```

---

## 2. Reconnaissance Methods

```
OBSERVATION TECHNIQUES:

PASSIVE OBSERVATION:
├── Drive/walk by facility
├── Note guard schedules and shift changes
├── Observe employee behavior
│   ├── Do they badge in or tailgate?
│   ├── Do they challenge strangers?
│   └── Do they hold doors open?
├── Identify peak hours and quiet times
├── Note delivery schedules (vendors)
└── Photograph from public areas

OPEN SOURCE INTELLIGENCE:
├── Google Maps / Street View
│   └── Building layout, entrances, parking
├── Google Earth
│   └── Aerial view, perimeter, adjacent buildings
├── Building permits / floor plans
│   └── Public records at city hall
├── Real estate listings
│   └── Interior photos of similar buildings
├── Social media photos from employees
│   └── Office interior, badge designs
└── Yelp / Google reviews
    └── Visitor experience, security description

PRETEXTING:
├── Pose as delivery driver → observe access procedures
├── Pose as job applicant → tour the facility
├── Pose as vendor/contractor → assess security checks
├── Attend public events at facility
└── Call reception → learn visitor procedures
```

---

## 3. Badge Cloning Intelligence

```
BADGE/RFID RECONNAISSANCE:

BADGE TYPES:
├── 125 kHz (HID Prox, EM4100) → EASILY CLONABLE
├── 13.56 MHz (iCLASS, MIFARE) → Harder but possible
├── Smart cards (PIV, PKI) → Difficult to clone
└── Mobile credentials (phone) → App-dependent

RECONNAISSANCE FOR CLONING:
├── Identify badge type (visual inspection)
│   ├── HID iCLASS → black card with HID logo
│   ├── HID Prox → clamshell or ISO card
│   └── MIFARE → check card markings
├── Observe reader type
│   ├── HID reader model visible on unit
│   └── Identifies compatible card technology
├── Badge photo from social media
│   └── Employee posting their "first day" badge
├── Range for reading (Proxmark3)
│   └── Some readers have long read range
└── Facility number / card format
    └── Often printed on the badge

TOOLS:
├── Proxmark3 — read, clone, emulate cards
├── Flipper Zero — RFID/NFC reader
├── ACR122U — NFC reader/writer
└── Keysy — simple badge cloner
```

---

## 4. Physical Security Assessment Checklist

| Area | Check | Rating |
|------|-------|:------:|
| **Perimeter** | Fencing, lighting, cameras | ⬜ |
| **Entrances** | Access controls, guards | ⬜ |
| **Reception** | Visitor procedures, ID check | ⬜ |
| **Tailgating** | Do employees challenge? | ⬜ |
| **Doors** | Emergency exits propped open? | ⬜ |
| **Badges** | Type, clonable? | ⬜ |
| **Cameras** | Coverage, blind spots | ⬜ |
| **Dumpsters** | Locked? Shredding policy? | ⬜ |
| **Smoking area** | Outside badge perimeter? | ⬜ |
| **Loading dock** | Access during deliveries? | ⬜ |

---

## 5. Dumpster Diving Intelligence

```
DUMPSTER DIVING FINDINGS:

COMMON DISCARDED ITEMS:
├── Printed documents (not shredded)
├── Old employee lists / phone directories
├── Network diagrams / IT documentation
├── Hard drives / USB drives (not wiped)
├── Discarded badges / access cards
├── Post-it notes with passwords
├── Organizational charts
├── Meeting agendas / notes
├── Financial documents
└── Customer data

TECHNIQUE:
├── Identify dumpster location
├── Determine pickup schedule (go after hours before pickup)
├── Wear appropriate clothing (high-vis vest = invisible)
├── Sort through materials methodically
├── Photograph findings (leave originals)
└── Document chain of custody

LEGAL NOTE:
├── Generally legal once trash is on public property
├── Varies by jurisdiction
├── Must be within engagement scope
└── Get explicit authorization in Rules of Engagement
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Observation** | Map guards, cameras, access controls |
| **Google Maps** | Remote physical reconnaissance |
| **Badge cloning** | Identify card type for cloning feasibility |
| **Tailgating** | Follow employees through secured doors |
| **Dumpster diving** | Recover discarded sensitive documents |
| **Pretexting** | Pose as delivery/vendor for access |

---

## Quick Revision Questions

1. **What physical security elements should you observe?**
2. **How does Google Maps/Earth help physical reconnaissance?**
3. **What badge types are easily clonable?**
4. **What tools are used for RFID badge cloning?**
5. **What intelligence can dumpster diving provide?**

---

[← Previous: Behavioral Patterns](04-behavioral-patterns.md)

---

*Unit 7 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Active Reconnaissance →](../08-Active-Reconnaissance/01-port-scanning.md)*
