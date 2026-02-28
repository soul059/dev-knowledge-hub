# Chapter 7.1: Requirements and Specification

## Introduction

Requirements engineering is the foundation of successful embedded system design. Clear, complete, and testable requirements prevent costly errors discovered late in development. This chapter covers techniques for gathering, documenting, and managing requirements throughout the product lifecycle.

---

## Requirements Engineering Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                 REQUIREMENTS ENGINEERING WORKFLOW                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐     │
│  │  ELICIT     │───▶│  ANALYZE    │───▶│  DOCUMENT           │     │
│  │             │    │             │    │                     │     │
│  │ • Interview │    │ • Feasible? │    │ • Requirements Spec │     │
│  │ • Observe   │    │ • Complete? │    │ • Use Cases         │     │
│  │ • Survey    │    │ • Conflict? │    │ • Traceability      │     │
│  │ • Prototype │    │ • Testable? │    │ • Priorities        │     │
│  └─────────────┘    └─────────────┘    └──────────┬──────────┘     │
│         ▲                                         │                 │
│         │           ┌─────────────┐               │                 │
│         │           │  MANAGE     │◀──────────────┘                 │
│         │           │             │                                 │
│         │           │ • Version   │                                 │
│         └───────────│ • Changes   │                                 │
│                     │ • Track     │                                 │
│                     └─────────────┘                                 │
│                                                                      │
│  CONTINUOUS ITERATION THROUGHOUT PROJECT                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Types of Requirements

### Functional Requirements

**What the system must DO** - the features and behaviors.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FUNCTIONAL REQUIREMENTS                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Example: Smart Thermostat                                          │
│                                                                      │
│  SENSING:                                                            │
│  FR-001: Measure ambient temperature with ±0.5°C accuracy          │
│  FR-002: Measure humidity with ±3% RH accuracy                      │
│  FR-003: Sample temperature every 10 seconds                        │
│                                                                      │
│  CONTROL:                                                            │
│  FR-010: Control heating relay (ON/OFF)                             │
│  FR-011: Control cooling relay (ON/OFF)                             │
│  FR-012: Support heating/cooling deadband of 0.5-3°C               │
│                                                                      │
│  USER INTERFACE:                                                     │
│  FR-020: Display current temperature on LCD                         │
│  FR-021: Allow setpoint adjustment via buttons                      │
│  FR-022: Display heating/cooling status indicator                   │
│                                                                      │
│  COMMUNICATION:                                                      │
│  FR-030: Connect to WiFi network (2.4GHz 802.11b/g/n)              │
│  FR-031: Support HTTP REST API for remote control                   │
│  FR-032: Report data to cloud server every 60 seconds              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Non-Functional Requirements

**How well the system must perform** - quality attributes.

```
┌─────────────────────────────────────────────────────────────────────┐
│                NON-FUNCTIONAL REQUIREMENTS CATEGORIES                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CATEGORY          │ EXAMPLES                                       │
│  ──────────────────┼───────────────────────────────────────────────│
│  PERFORMANCE       │ Response time < 100ms for button press         │
│                    │ Boot time < 5 seconds                          │
│                    │ Control loop update rate 10Hz                  │
│  ──────────────────┼───────────────────────────────────────────────│
│  RELIABILITY       │ MTBF > 50,000 hours                           │
│                    │ Automatic recovery from power glitch           │
│                    │ Data retention during power loss               │
│  ──────────────────┼───────────────────────────────────────────────│
│  POWER             │ Average current < 50mA @ 5V                    │
│                    │ Sleep current < 1mA                            │
│                    │ Battery life > 6 months on 2xAA                │
│  ──────────────────┼───────────────────────────────────────────────│
│  ENVIRONMENTAL     │ Operating temp: -10°C to +50°C                │
│                    │ Storage temp: -20°C to +70°C                  │
│                    │ Humidity: 10-90% RH non-condensing            │
│  ──────────────────┼───────────────────────────────────────────────│
│  EMC               │ Immunity to ESD ±4kV contact                  │
│                    │ Radiated emissions meet FCC Class B            │
│  ──────────────────┼───────────────────────────────────────────────│
│  SECURITY          │ AES-128 encryption for communication           │
│                    │ Secure boot with signature verification        │
│                    │ No default passwords                           │
│  ──────────────────┼───────────────────────────────────────────────│
│  MAINTAINABILITY   │ Firmware update via OTA                        │
│                    │ Diagnostic logging capability                  │
│                    │ Modular software architecture                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Constraints

**Limitations that bound the design** - non-negotiable boundaries.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DESIGN CONSTRAINTS                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PHYSICAL:                                                           │
│  C-001: PCB size maximum 50mm × 80mm                                │
│  C-002: Enclosure height maximum 25mm                               │
│  C-003: Total weight < 100g including battery                       │
│                                                                      │
│  COST:                                                               │
│  C-010: BOM cost target < $15 @ 10,000 units                        │
│  C-011: Manufacturing cost target < $5/unit                         │
│                                                                      │
│  SCHEDULE:                                                           │
│  C-020: Prototype ready in 8 weeks                                  │
│  C-021: Production release in 6 months                              │
│                                                                      │
│  REGULATORY:                                                         │
│  C-030: Must comply with FCC Part 15                                │
│  C-031: Must comply with CE marking requirements                    │
│  C-032: RoHS compliant components only                              │
│                                                                      │
│  INTERFACE:                                                          │
│  C-040: Must use existing housing tooling                           │
│  C-041: Compatible with standard wall outlet mounting               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Writing Good Requirements

### SMART Requirements

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SMART REQUIREMENTS                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  S - SPECIFIC                                                        │
│      ✗ "System should be fast"                                      │
│      ✓ "System shall respond to button press within 100ms"          │
│                                                                      │
│  M - MEASURABLE                                                      │
│      ✗ "Temperature accuracy should be good"                        │
│      ✓ "Temperature measurement accuracy ±0.5°C from -20 to +60°C" │
│                                                                      │
│  A - ACHIEVABLE                                                      │
│      ✗ "System shall have infinite battery life"                    │
│      ✓ "System shall operate 12 months on 2×AA batteries"          │
│                                                                      │
│  R - RELEVANT                                                        │
│      ✗ "System shall have Bluetooth 5.0" (if WiFi-only product)    │
│      ✓ "System shall connect to 2.4GHz WiFi networks"              │
│                                                                      │
│  T - TESTABLE                                                        │
│      ✗ "System shall be user-friendly"                              │
│      ✓ "User shall complete setup in < 5 minutes without manual"   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Requirement Language

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REQUIREMENT KEYWORDS                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  KEYWORD     │ MEANING                        │ PRIORITY            │
│  ────────────┼────────────────────────────────┼────────────────────│
│  SHALL       │ Mandatory requirement          │ MUST HAVE           │
│  SHOULD      │ Recommended but not essential  │ SHOULD HAVE         │
│  MAY         │ Optional/permitted             │ NICE TO HAVE        │
│  WILL        │ Statement of intent/fact       │ INFORMATIONAL       │
│                                                                      │
│  EXAMPLE REQUIREMENTS:                                               │
│                                                                      │
│  FR-101: The system SHALL measure temperature every 10 seconds.     │
│          [Mandatory - core functionality]                           │
│                                                                      │
│  FR-102: The system SHOULD display temperature in both °C and °F.  │
│          [Recommended - good UX but not critical]                   │
│                                                                      │
│  FR-103: The system MAY support external temperature probes.        │
│          [Optional - future enhancement]                            │
│                                                                      │
│  NOTE: The system WILL use an ESP32 microcontroller.               │
│          [Statement of design decision, not requirement]            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Use Cases and User Stories

### Use Case Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    USE CASE DIAGRAM                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                     ┌─────────────────────────────────┐             │
│                     │      SMART THERMOSTAT           │             │
│                     │                                 │             │
│                     │  ┌───────────────────────┐     │             │
│   ┌───┐            │  │   View Temperature    │     │             │
│   │   │────────────┼──│                       │     │             │
│   │ U │            │  └───────────────────────┘     │             │
│   │ S │            │                                 │             │
│   │ E │            │  ┌───────────────────────┐     │             │
│   │ R │────────────┼──│   Adjust Setpoint     │     │             │
│   │   │            │  └───────────────────────┘     │             │
│   └───┘            │                                 │             │
│                     │  ┌───────────────────────┐     │             │
│                     │  │   View Schedule       │     │             │
│   ┌───┐            │  └───────────────────────┘     │             │
│   │ A │            │         │                       │             │
│   │ D │            │         │ <<includes>>          │             │
│   │ M │            │         ▼                       │             │
│   │ I │────────────┼──┌───────────────────────┐     │             │
│   │ N │            │  │   Program Schedule    │     │             │
│   └───┘            │  └───────────────────────┘     │   ┌───┐     │
│                     │                                 │   │ C │     │
│                     │  ┌───────────────────────┐     │───│ L │     │
│                     │  │   Remote Monitor      │     │   │ O │     │
│                     │  │   (via App/Web)       │─────┼───│ U │     │
│                     │  └───────────────────────┘     │   │ D │     │
│                     │                                 │   └───┘     │
│                     └─────────────────────────────────┘             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Use Case Description

```
┌─────────────────────────────────────────────────────────────────────┐
│                    USE CASE: ADJUST SETPOINT                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ID:           UC-002                                               │
│  Name:         Adjust Temperature Setpoint                          │
│  Actor:        User                                                 │
│  Priority:     High                                                 │
│  Precondition: System is powered and displaying current temp        │
│                                                                      │
│  MAIN FLOW:                                                          │
│  1. User presses UP or DOWN button                                  │
│  2. System displays setpoint adjustment mode                        │
│  3. System shows current setpoint value                             │
│  4. User presses UP/DOWN to adjust (+/- 0.5°C per press)           │
│  5. System displays new setpoint value                              │
│  6. If no button pressed for 5 seconds, system saves setpoint       │
│  7. System returns to normal display mode                           │
│                                                                      │
│  ALTERNATIVE FLOWS:                                                  │
│  4a. User presses both buttons simultaneously                       │
│      - System cancels adjustment, reverts to previous setpoint     │
│      - Go to step 7                                                 │
│                                                                      │
│  4b. Setpoint reaches maximum (35°C) or minimum (10°C)             │
│      - System beeps to indicate limit                               │
│      - Setpoint remains at limit                                    │
│                                                                      │
│  POSTCONDITION:                                                      │
│  - New setpoint stored in non-volatile memory                       │
│  - HVAC control uses new setpoint                                   │
│                                                                      │
│  NON-FUNCTIONAL:                                                     │
│  - Button response time < 100ms                                     │
│  - Display update time < 50ms                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### User Stories (Agile Format)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    USER STORIES                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  FORMAT: As a [role], I want [feature] so that [benefit]            │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  US-001: As a homeowner, I want to see the current temperature      │
│          so that I know if my home is comfortable.                  │
│                                                                      │
│  ACCEPTANCE CRITERIA:                                                │
│  • Temperature displayed in large digits on main screen             │
│  • Temperature updates at least every 30 seconds                    │
│  • Accuracy within ±1°F of reference thermometer                   │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  US-002: As a homeowner, I want to adjust temperature from my       │
│          phone so that I can control heating before arriving home.  │
│                                                                      │
│  ACCEPTANCE CRITERIA:                                                │
│  • Mobile app shows current temperature and setpoint                │
│  • Can adjust setpoint from app                                     │
│  • Changes reflected on device within 10 seconds                    │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  US-003: As an energy-conscious user, I want to see energy usage    │
│          history so that I can optimize my heating schedule.        │
│                                                                      │
│  ACCEPTANCE CRITERIA:                                                │
│  • Daily/weekly/monthly usage graphs in app                         │
│  • Compare usage to previous periods                                │
│  • Suggestions for energy savings                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Requirements Traceability

### Traceability Matrix

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TRACEABILITY MATRIX                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Purpose: Link requirements → design → implementation → tests      │
│                                                                      │
│  REQ ID   │ DESIGN        │ MODULE          │ TEST CASE             │
│  ─────────┼───────────────┼─────────────────┼───────────────────────│
│  FR-001   │ ARCH-003      │ temp_sensor.c   │ TC-001, TC-002        │
│  FR-002   │ ARCH-003      │ humidity.c      │ TC-003                │
│  FR-010   │ ARCH-005      │ relay_ctrl.c    │ TC-010, TC-011        │
│  FR-020   │ ARCH-007      │ display.c       │ TC-020                │
│  FR-030   │ ARCH-010      │ wifi_mgr.c      │ TC-030, TC-031        │
│  NFR-001  │ ARCH-001      │ system.c        │ TC-P01                │
│  NFR-010  │ ARCH-002      │ power_mgr.c     │ TC-P10                │
│                                                                      │
│  VISUALIZATION:                                                      │
│                                                                      │
│  Requirements      Design           Code              Tests          │
│       │               │               │                 │           │
│    FR-001 ────────▶ ARCH-003 ────▶ temp_sensor.c ──▶ TC-001       │
│       │               │               │                 │           │
│       │               │               │                 ├──▶ TC-002 │
│       │               │               │                 │           │
│    FR-010 ────────▶ ARCH-005 ────▶ relay_ctrl.c ───▶ TC-010       │
│       │               │               │                 │           │
│       └───────────────┴───────────────┴─────────────────┘           │
│                                                                      │
│  BENEFITS:                                                           │
│  • Verify all requirements are implemented                          │
│  • Verify all requirements are tested                               │
│  • Impact analysis when requirements change                         │
│  • Evidence for audits and certification                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Specification Document Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│           SYSTEM REQUIREMENTS SPECIFICATION (SRS)                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. INTRODUCTION                                                     │
│     1.1 Purpose                                                      │
│     1.2 Scope                                                        │
│     1.3 Definitions and Acronyms                                    │
│     1.4 References                                                   │
│     1.5 Overview                                                     │
│                                                                      │
│  2. OVERALL DESCRIPTION                                              │
│     2.1 Product Perspective                                          │
│     2.2 Product Functions (high-level)                              │
│     2.3 User Characteristics                                         │
│     2.4 Constraints                                                  │
│     2.5 Assumptions and Dependencies                                │
│                                                                      │
│  3. SPECIFIC REQUIREMENTS                                            │
│     3.1 External Interface Requirements                              │
│         3.1.1 User Interfaces                                        │
│         3.1.2 Hardware Interfaces                                    │
│         3.1.3 Software Interfaces                                    │
│         3.1.4 Communication Interfaces                               │
│     3.2 Functional Requirements                                      │
│     3.3 Non-Functional Requirements                                  │
│         3.3.1 Performance                                            │
│         3.3.2 Reliability                                            │
│         3.3.3 Security                                               │
│         3.3.4 Maintainability                                        │
│                                                                      │
│  4. APPENDICES                                                       │
│     A. Use Case Diagrams                                             │
│     B. State Diagrams                                                │
│     C. Traceability Matrix                                           │
│     D. Glossary                                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Requirements Management

### Change Control Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHANGE CONTROL PROCESS                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│              ┌───────────────┐                                      │
│              │ Change        │                                      │
│              │ Request       │                                      │
│              └───────┬───────┘                                      │
│                      │                                               │
│                      ▼                                               │
│              ┌───────────────┐                                      │
│              │ Log & Assign  │  Track all requests                  │
│              │ ID            │                                      │
│              └───────┬───────┘                                      │
│                      │                                               │
│                      ▼                                               │
│              ┌───────────────┐                                      │
│              │ Impact        │  Analyze: Cost, Schedule,            │
│              │ Analysis      │  Technical feasibility               │
│              └───────┬───────┘                                      │
│                      │                                               │
│                      ▼                                               │
│              ┌───────────────┐                                      │
│       ┌──────│ Review        │──────┐                               │
│       │      │ Board         │      │                               │
│       │      └───────────────┘      │                               │
│       │                             │                               │
│       ▼                             ▼                               │
│  ┌─────────┐                 ┌─────────────┐                       │
│  │ REJECT  │                 │ APPROVE     │                       │
│  └─────────┘                 └──────┬──────┘                       │
│                                     │                               │
│                                     ▼                               │
│                              ┌─────────────┐                       │
│                              │ Implement   │  Update all affected  │
│                              │ Change      │  documents             │
│                              └──────┬──────┘                       │
│                                     │                               │
│                                     ▼                               │
│                              ┌─────────────┐                       │
│                              │ Verify &    │                       │
│                              │ Close       │                       │
│                              └─────────────┘                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Requirements States

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REQUIREMENT LIFECYCLE STATES                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌────────┐     ┌──────────┐     ┌──────────┐     ┌─────────┐    │
│   │ DRAFT  │────▶│ APPROVED │────▶│ DESIGNED │────▶│IMPLEMENT│    │
│   └────────┘     └──────────┘     └──────────┘     └────┬────┘    │
│       │               │                                  │         │
│       │               │                                  ▼         │
│       │               │                            ┌──────────┐    │
│       │               │                            │ VERIFIED │    │
│       │               │                            └──────────┘    │
│       │               │                                             │
│       ▼               ▼                                             │
│   ┌────────┐     ┌──────────┐                                      │
│   │REJECTED│     │ DEFERRED │ ───▶ Future release                  │
│   └────────┘     └──────────┘                                      │
│                                                                      │
│   STATE TRACKING TABLE:                                             │
│                                                                      │
│   REQ ID   │ STATE      │ VERSION │ LAST CHANGED │ OWNER           │
│   ─────────┼────────────┼─────────┼──────────────┼─────────────────│
│   FR-001   │ Verified   │ 1.2     │ 2024-01-15   │ John            │
│   FR-002   │ Implement  │ 1.0     │ 2024-01-10   │ Mary            │
│   FR-010   │ Designed   │ 2.0     │ 2024-01-12   │ John            │
│   FR-015   │ Deferred   │ 1.0     │ 2024-01-05   │ Bob             │
│   FR-020   │ Draft      │ 0.1     │ 2024-01-14   │ Mary            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Requirements Elicitation Techniques

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ELICITATION TECHNIQUES                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TECHNIQUE        │ WHEN TO USE                    │ PROS/CONS      │
│  ─────────────────┼────────────────────────────────┼────────────────│
│  INTERVIEWS       │ Initial requirements,          │ + Rich detail  │
│                   │ complex domains                │ - Time consuming│
│  ─────────────────┼────────────────────────────────┼────────────────│
│  OBSERVATION      │ Understanding current          │ + Real behavior│
│                   │ workflows                      │ - Observer bias│
│  ─────────────────┼────────────────────────────────┼────────────────│
│  SURVEYS          │ Large user base,               │ + Scalable     │
│                   │ preference data                │ - Limited depth│
│  ─────────────────┼────────────────────────────────┼────────────────│
│  PROTOTYPING      │ UI requirements,               │ + Early feedback│
│                   │ unclear needs                  │ - May mislead  │
│  ─────────────────┼────────────────────────────────┼────────────────│
│  BRAINSTORMING    │ Generating ideas,              │ + Creative     │
│                   │ early phases                   │ - Unfocused    │
│  ─────────────────┼────────────────────────────────┼────────────────│
│  DOCUMENT         │ Existing systems,              │ + Baseline info│
│  ANALYSIS         │ regulations                    │ - May be dated │
│  ─────────────────┼────────────────────────────────┼────────────────│
│  COMPETITOR       │ Market expectations,           │ + Benchmarks   │
│  ANALYSIS         │ feature comparison             │ - No innovation│
│                                                                      │
│  EMBEDDED-SPECIFIC CONSIDERATIONS:                                   │
│  • Observe physical environment where device operates               │
│  • Interview technicians who maintain similar equipment             │
│  • Review applicable standards (safety, EMC, industry)              │
│  • Analyze failure modes of existing products                       │
│  • Test with target user demographics (age, tech-savviness)        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Functional Requirements** | What the system must do (features, behaviors) |
| **Non-Functional Requirements** | How well it must perform (quality attributes) |
| **Constraints** | Boundaries that limit design choices |
| **SMART** | Specific, Measurable, Achievable, Relevant, Testable |
| **SHALL/SHOULD/MAY** | Mandatory/Recommended/Optional requirement keywords |
| **Use Cases** | Scenarios describing user-system interaction |
| **User Stories** | Agile format: As a [role], I want [feature], so that [benefit] |
| **Traceability** | Linking requirements through design to test |
| **Change Control** | Formal process for modifying requirements |
| **SRS** | System Requirements Specification document |

---

## Quick Revision Questions

1. **What is the difference between functional and non-functional requirements? Give examples of each for an embedded system.**

2. **Explain the SMART criteria for writing requirements. Why is "testable" particularly important for embedded systems?**

3. **What is a requirements traceability matrix? How does it help during development and testing?**

4. **Compare the use case format with user story format. When would you use each?**

5. **Why is change control important in requirements management? What happens without it?**

6. **List three techniques for eliciting requirements. What are the specific considerations for embedded systems?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit 7 Overview](README.md) | [Unit 7: System Design](README.md) | [Chapter 7.2: Hardware-Software Co-Design](02-hardware-software-codesign.md) |

---
