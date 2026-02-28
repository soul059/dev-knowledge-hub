# Unit 7: System Design

## Overview

This unit covers the methodology and practices for designing complete embedded systems. From initial requirements to final testing, these concepts guide the development of reliable, maintainable, and efficient embedded products.

---

## Unit Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EMBEDDED SYSTEM DESIGN FLOW                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  1. REQUIREMENTS ANALYSIS                                   │     │
│  │     • Functional requirements (what it does)                │     │
│  │     • Non-functional requirements (how well)                │     │
│  │     • Constraints (cost, size, power)                       │     │
│  └────────────────────────┬───────────────────────────────────┘     │
│                           ▼                                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  2. SYSTEM ARCHITECTURE                                     │     │
│  │     • Hardware/software partitioning                        │     │
│  │     • Component selection                                   │     │
│  │     • Block diagrams                                        │     │
│  └────────────────────────┬───────────────────────────────────┘     │
│                           ▼                                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  3. DETAILED DESIGN                                         │     │
│  │     • Schematic design                                      │     │
│  │     • Software architecture                                 │     │
│  │     • Interface definitions                                 │     │
│  └────────────────────────┬───────────────────────────────────┘     │
│                           ▼                                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  4. IMPLEMENTATION                                          │     │
│  │     • PCB layout                                            │     │
│  │     • Firmware development                                  │     │
│  │     • Unit testing                                          │     │
│  └────────────────────────┬───────────────────────────────────┘     │
│                           ▼                                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  5. INTEGRATION & TEST                                      │     │
│  │     • Hardware-software integration                         │     │
│  │     • System testing                                        │     │
│  │     • Validation against requirements                       │     │
│  └────────────────────────┬───────────────────────────────────┘     │
│                           ▼                                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  6. DEPLOYMENT & MAINTENANCE                                │     │
│  │     • Production setup                                      │     │
│  │     • Field updates                                         │     │
│  │     • Support and documentation                             │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Chapter Overview

### Chapter 7.1: Requirements and Specification
- Gathering and documenting requirements
- Functional vs non-functional requirements
- Use cases and user stories
- Specification documents
- Traceability

### Chapter 7.2: Hardware-Software Co-Design
- Partitioning decisions
- Trade-offs and optimization
- Hardware abstraction layers
- Resource allocation
- Timing analysis

### Chapter 7.3: Prototyping and Development
- Development boards and tools
- Rapid prototyping methods
- Iterative development
- Version control for embedded
- Documentation practices

### Chapter 7.4: Testing and Validation
- Testing strategies and levels
- Unit testing in embedded
- Hardware-in-the-loop testing
- Test fixtures and automation
- Debugging techniques

### Chapter 7.5: Power Management
- Power budget analysis
- Low-power design techniques
- Sleep modes and wake sources
- Battery considerations
- Power supply design

---

## Key Design Considerations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DESIGN TRADE-OFFS                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  THE IRON TRIANGLE OF EMBEDDED SYSTEMS:                             │
│                                                                      │
│                    PERFORMANCE                                       │
│                        ╱╲                                           │
│                       ╱  ╲                                          │
│                      ╱    ╲                                         │
│                     ╱      ╲                                        │
│                    ╱        ╲                                       │
│                   ╱          ╲                                      │
│                  ╱            ╲                                     │
│                 ╱   BALANCE    ╲                                    │
│                ╱________________╲                                   │
│            COST              POWER                                  │
│                                                                      │
│  Improving one often impacts others:                                │
│  • Higher performance → More power, higher cost                    │
│  • Lower power → Slower performance, may need special parts        │
│  • Lower cost → Fewer features, less performance                   │
│                                                                      │
│  ADDITIONAL CONSTRAINTS:                                             │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ Constraint       │ Impact                                │       │
│  ├──────────────────────────────────────────────────────────┤       │
│  │ Size/Weight      │ Component choice, PCB layers          │       │
│  │ Time-to-market   │ Platform choice, reuse                │       │
│  │ Reliability      │ Component quality, redundancy         │       │
│  │ Certifications   │ Design rules, testing requirements    │       │
│  │ Temperature      │ Component ratings, thermal design     │       │
│  │ Volume           │ Manufacturing process, component $    │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Design Methodologies

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DEVELOPMENT APPROACHES                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  WATERFALL (Traditional):           V-MODEL:                        │
│                                                                      │
│  Requirements ──▶                   Req ─────────────────── VAL    │
│       │                               ╲                    ╱        │
│       ▼                                ╲                  ╱         │
│  Design ────────▶                   Design ─────────── IntTest     │
│       │                                  ╲            ╱             │
│       ▼                                   ╲          ╱              │
│  Implementation ─▶                     Detail ──── UnitTest        │
│       │                                      ╲    ╱                 │
│       ▼                                       ╲  ╱                  │
│  Testing ───────▶                            CODE                   │
│       │                                                              │
│       ▼                          Each design stage has             │
│  Deployment                      corresponding test stage          │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  AGILE/ITERATIVE:                   SPIRAL:                        │
│                                                                      │
│  ┌─────────────────────┐            ┌───────────────────┐          │
│  │ Sprint 1            │            │     Prototype 1   │──┐       │
│  │ Plan→Dev→Test→Demo  │──┐         └─────────┬─────────┘  │       │
│  └─────────────────────┘  │                   │           │        │
│                           │         ┌─────────▼─────────┐  │       │
│  ┌─────────────────────┐  │         │     Prototype 2   │──┤       │
│  │ Sprint 2            │◀─┤         └─────────┬─────────┘  │       │
│  │ Improved version    │──┤                   │           │        │
│  └─────────────────────┘  │         ┌─────────▼─────────┐  │       │
│                           │         │     Production    │◀─┘       │
│  ┌─────────────────────┐  │         └───────────────────┘          │
│  │ Sprint N            │◀─┘                                        │
│  │ Final product       │           Risk-driven, prototype          │
│  └─────────────────────┘           at each level                   │
│                                                                      │
│  Short cycles, continuous feedback                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## System Design Checklist

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PRE-DESIGN CHECKLIST                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  REQUIREMENTS:                                                       │
│  □ Functional requirements documented                               │
│  □ Performance requirements quantified                              │
│  □ Environmental requirements specified                             │
│  □ Safety requirements identified                                   │
│  □ Regulatory requirements known                                    │
│                                                                      │
│  HARDWARE:                                                           │
│  □ MCU selection criteria defined                                   │
│  □ Peripheral requirements listed                                   │
│  □ Power budget estimated                                           │
│  □ Memory requirements estimated                                    │
│  □ I/O count verified                                               │
│                                                                      │
│  SOFTWARE:                                                           │
│  □ RTOS vs bare-metal decision                                      │
│  □ Communication protocols identified                               │
│  □ Update mechanism planned                                         │
│  □ Error handling strategy                                          │
│  □ Testing strategy defined                                         │
│                                                                      │
│  DEVELOPMENT:                                                        │
│  □ Development tools available                                      │
│  □ Debug access planned                                             │
│  □ Version control set up                                           │
│  □ CI/CD pipeline considered                                        │
│  □ Documentation templates ready                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit 6: Interfacing](../06-Interfacing/README.md) | [Course Index](../README.md) | [Chapter 7.1: Requirements and Specification](01-requirements-specification.md) |

---
