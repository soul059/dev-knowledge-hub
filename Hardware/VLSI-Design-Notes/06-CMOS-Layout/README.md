# Unit 6: CMOS Layout

## 📋 Unit Overview

This unit covers the **physical design** of CMOS circuits—translating transistor-level schematics into manufacturable layouts. We study design rules, stick diagrams, standard cell layout, and optimization techniques that determine area, performance, and yield.

---

## 🎯 Unit Objectives

After completing this unit, you will be able to:
- Interpret and apply CMOS design rules
- Create stick diagrams and layouts for basic gates
- Design standard cells with consistent height
- Understand lambda-based and micron-based design rules
- Optimize layouts for area, speed, and manufacturability

---

## 📐 Layout Fundamentals

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FROM SCHEMATIC TO SILICON                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Design Flow:                                                      │
│                                                                      │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐          │
│   │  Schematic  │     │    Stick    │     │   Layout    │          │
│   │             │────►│   Diagram   │────►│             │          │
│   │  (Logic)    │     │  (Topology) │     │  (Geometry) │          │
│   └─────────────┘     └─────────────┘     └─────────────┘          │
│         │                                        │                  │
│         │                                        ▼                  │
│         │                                  ┌─────────────┐          │
│         │                                  │    DRC      │          │
│         └────────────────────────────────►│    LVS      │          │
│                                            │  Extraction │          │
│                                            └─────────────┘          │
│                                                                      │
│   Key Concepts:                                                     │
│                                                                      │
│   ┌───────────────────────────────────────────────────────────────┐ │
│   │ Concept        │ Description                                 │ │
│   ├───────────────────────────────────────────────────────────────┤ │
│   │ Mask layers    │ Physical layers (poly, metal, diffusion)   │ │
│   │ Design rules   │ Minimum dimensions and spacings            │ │
│   │ DRC           │ Design Rule Check (geometry verification)   │ │
│   │ LVS           │ Layout vs Schematic (connectivity check)    │ │
│   │ Extraction    │ Extract parasitics from layout             │ │
│   └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│   Mask Layers for CMOS:                                            │
│                                                                      │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │ Layer          │ Purpose                                    │  │
│   ├──────────────────────────────────────────────────────────────┤  │
│   │ N-well         │ Creates region for PMOS transistors       │  │
│   │ Active (Diff)  │ Defines transistor source/drain regions   │  │
│   │ Polysilicon    │ Transistor gates and local interconnect   │  │
│   │ N+ implant     │ Dopes active region for NMOS              │  │
│   │ P+ implant     │ Dopes active region for PMOS              │  │
│   │ Contact        │ Connects diffusion/poly to Metal1         │  │
│   │ Metal1         │ First metal interconnect layer            │  │
│   │ Via1           │ Connects Metal1 to Metal2                 │  │
│   │ Metal2+        │ Higher metal layers                       │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Cross-Section View

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CMOS CROSS-SECTION                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                    PMOS Region          NMOS Region                 │
│                    (in N-well)          (in P-substrate)            │
│                                                                      │
│         Metal2    ════════════════════════════════════              │
│                         │ Via1                                      │
│         Metal1    ══════╪════════════════════════════               │
│                    │Contact│                │Contact│               │
│                    │     │ │                │     │ │               │
│     ┌──────────────┴─────┴─┴────────────────┴─────┴─┴───────┐       │
│     │                      │                                │       │
│     │   P+      │Gate│  P+ │    N+      │Gate│   N+        │       │
│     │ (Source)  │Poly│(Drain)  (Source) │Poly│ (Drain)     │       │
│     │           │    │     │            │    │             │       │
│     │     ╔═════╧════╧═════╣      ╔═════╧════╧═════╗       │       │
│     │     ║    N-Well      ║      ║   P-substrate  ║       │       │
│     │     ║                ║      ║                ║       │       │
│     │     ╚════════════════╝      ╚════════════════╝       │       │
│     │                                                       │       │
│     │                    P-substrate                        │       │
│     └───────────────────────────────────────────────────────┘       │
│                                                                      │
│   Key observations:                                                 │
│   • PMOS in N-well, NMOS in P-substrate                            │
│   • Polysilicon crosses active to form transistor                  │
│   • Contacts connect active/poly to metal                          │
│   • Multiple metal layers for routing                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Chapters in This Unit

| Chapter | Topic | Description |
|---------|-------|-------------|
| 6.1 | [Design Rules](01-design-rules.md) | Minimum widths, spacings, lambda rules |
| 6.2 | [Stick Diagrams](02-stick-diagrams.md) | Simplified layout representation |
| 6.3 | [Inverter Layout](03-inverter-layout.md) | Complete inverter physical design |
| 6.4 | [Standard Cell Design](04-standard-cell-design.md) | Fixed-height cells, power rails |
| 6.5 | [Layout Optimization](05-layout-optimization.md) | Area, speed, and DFM techniques |

---

## 🔑 Key Concepts Preview

| Concept | Description |
|---------|-------------|
| Lambda (λ) | Half of minimum feature size, scalable unit |
| Minimum width | Smallest allowed dimension for a layer |
| Minimum spacing | Smallest allowed gap between features |
| Overlap | Required extension of one layer over another |
| Standard cell | Fixed-height layout block for automated P&R |

---

## 🔗 Navigation

| Previous Unit | Course Home | Next Unit |
|--------------|-------------|-----------|
| ← [Unit 5: Sequential CMOS Circuits](../05-Sequential-CMOS-Circuits/README.md) | [VLSI Design Notes](../README.md) | [Unit 7: Timing Analysis →](../07-Timing-Analysis/README.md) |
