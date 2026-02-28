# Unit 4: Combinational CMOS Logic

## 📋 Unit Overview

**Combinational CMOS logic circuits** produce outputs that depend only on the current input values—there is no memory or feedback. This unit covers the design of static CMOS gates, complementary logic structures, complex gate synthesis, and advanced logic styles like pseudo-NMOS and pass-transistor logic.

---

## 🎯 Unit Objectives

After completing this unit, you will be able to:
- Design static CMOS gates using pull-up and pull-down networks
- Apply complementary logic principles for arbitrary functions
- Synthesize complex gates from Boolean expressions
- Compare different CMOS logic styles and their trade-offs
- Analyze timing and power for combinational circuits

---

## 📚 Chapters in This Unit

| Chapter | Title | Key Topics |
|---------|-------|------------|
| 4.1 | [Static CMOS Gates](01-static-cmos-gates.md) | NAND, NOR, complex gates, PDN/PUN |
| 4.2 | [Complementary Logic Design](02-complementary-logic-design.md) | Duality, series-parallel networks |
| 4.3 | [Complex Gate Synthesis](03-complex-gate-synthesis.md) | AOI, OAI, compound functions |
| 4.4 | [Pseudo-NMOS Logic](04-pseudo-nmos-logic.md) | Ratioed logic, static power trade-off |
| 4.5 | [Pass Transistor Logic](05-pass-transistor-logic.md) | Transmission gates, CPL, threshold loss |

---

## 🔑 Key Concepts Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STATIC CMOS GATE STRUCTURE                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                         VDD                                          │
│                          │                                           │
│               ┌──────────┴──────────┐                                │
│               │                     │                                │
│               │  PULL-UP NETWORK    │                                │
│               │      (PUN)          │                                │
│               │   PMOS transistors  │                                │
│               │   Conducts for      │                                │
│               │   F = 1 (HIGH)      │                                │
│               │                     │                                │
│               └──────────┬──────────┘                                │
│                          │                                           │
│   Inputs ────────────────┼────────────────► Output (F)              │
│                          │                                           │
│               ┌──────────┴──────────┐                                │
│               │                     │                                │
│               │  PULL-DOWN NETWORK  │                                │
│               │      (PDN)          │                                │
│               │   NMOS transistors  │                                │
│               │   Conducts for      │                                │
│               │   F = 0 (LOW)       │                                │
│               │                     │                                │
│               └──────────┬──────────┘                                │
│                          │                                           │
│                         GND                                          │
│                                                                      │
│   Key Properties:                                                   │
│   • PUN and PDN are complementary (duals of each other)            │
│   • One network ON, other OFF (no static current)                  │
│   • Full rail-to-rail output swing                                 │
│   • Inverted output by default (NAND, NOR, NOT)                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📐 Logic Style Comparison

| Style | Speed | Power | Area | Noise Margin | Static Power |
|-------|-------|-------|------|--------------|--------------|
| Static CMOS | Good | Low | Large | Excellent | ~Zero |
| Pseudo-NMOS | Fast | High | Small | Reduced | Yes |
| Pass Transistor | Fast | Low | Small | Reduced | ~Zero |
| Transmission Gate | Good | Low | Medium | Good | ~Zero |

---

## 🔌 Duality Principle

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PDN ↔ PUN DUALITY                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   PDN (NMOS)                    PUN (PMOS)                          │
│   ──────────                    ──────────                          │
│                                                                      │
│   Series connection      ←→     Parallel connection                 │
│                                                                      │
│      A                              A ───┬─── B                     │
│      │                                   │                          │
│      B                                  ─┴─                         │
│      │                                                              │
│                                                                      │
│   Parallel connection    ←→     Series connection                   │
│                                                                      │
│   A ───┬─── B                       A                               │
│        │                            │                               │
│       ─┴─                           B                               │
│                                     │                               │
│                                                                      │
│   NMOS (input X)         ←→     PMOS (input X̄)                     │
│                                 (same input, complementary)         │
│                                                                      │
│   Example: 2-input NAND                                             │
│                                                                      │
│   PDN: A series B        ←→     PUN: A parallel B                   │
│   (both must be ON              (either can be ON                   │
│    to pull down)                 to pull up)                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Quick Reference: Common Gates

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STANDARD CMOS GATES                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   INVERTER       2-NAND          2-NOR                              │
│                                                                      │
│     VDD            VDD            VDD                               │
│      │              │              │                                │
│   ┌──┴──┐      ─┬───┴───┬─        │                                │
│   │PMOS │      │       │       ┌──┴──┐                              │
│   └──┬──┘    ┌─┴─┐   ┌─┴─┐     │     │                              │
│      │       │ P │   │ P │     │  P  │                              │
│      ├─out   └─┬─┘   └─┬─┘     └──┬──┘                              │
│      │         └───┬───┘          │                                 │
│   ┌──┴──┐          │           ┌──┴──┐                              │
│   │NMOS │          ├─out       │  P  │                              │
│   └──┬──┘          │           └──┬──┘                              │
│      │          ┌──┴──┐           │                                 │
│     GND         │  N  │           ├─out                             │
│                 └──┬──┘           │                                 │
│                    │           ─┬─┴─┬─                              │
│                 ┌──┴──┐        │   │                                │
│                 │  N  │      ┌─┴─┐ ┌─┴─┐                            │
│                 └──┬──┘      │ N │ │ N │                            │
│                    │         └─┬─┘ └─┬─┘                            │
│                   GND          └──┬──┘                              │
│                                   │                                 │
│                                  GND                                │
│                                                                      │
│   F = A'         F = (AB)'        F = (A+B)'                        │
│   1 NMOS         2 NMOS series    2 NMOS parallel                   │
│   1 PMOS         2 PMOS parallel  2 PMOS series                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔗 Navigation

| Previous Unit | Course Home | Next Unit |
|--------------|-------------|-----------|
| ← [Unit 3: CMOS Inverter](../03-CMOS-Inverter/README.md) | [Course Overview](../README.md) | [Unit 5: Sequential Circuits →](../05-Sequential-CMOS-Circuits/README.md) |
