# Unit 5: Sequential CMOS Circuits

## 📋 Unit Overview

This unit covers **sequential logic elements** in CMOS technology—circuits with memory that store state information. We study latches, flip-flops, timing constraints, and clock distribution strategies essential for building synchronous digital systems.

---

## 🎯 Unit Objectives

After completing this unit, you will be able to:
- Distinguish between combinational and sequential logic
- Design and analyze CMOS latches and flip-flops
- Understand setup time, hold time, and timing constraints
- Explain clock distribution and skew management
- Compare static and dynamic storage elements

---

## 📐 Sequential Circuit Fundamentals

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMBINATIONAL vs SEQUENTIAL                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   COMBINATIONAL LOGIC            SEQUENTIAL LOGIC                   │
│                                                                      │
│    Inputs ──┬──► Logic ──► Output    Inputs ──┬──► Logic ──► Output │
│             │                                 │      ▲              │
│             │                                 │      │              │
│             │                            ┌────┴──────┴────┐         │
│             │                            │    Memory      │         │
│             │                            │   (State)      │         │
│             │                            └────────────────┘         │
│                                                                      │
│   • Output depends ONLY               • Output depends on           │
│     on current inputs                   inputs AND stored state     │
│   • No memory                         • Has memory                  │
│   • Examples: AND, OR,                • Examples: Flip-flops,       │
│     adders, MUX                         counters, registers         │
│                                                                      │
│   Types of Sequential Circuits:                                     │
│                                                                      │
│   ┌─────────────────┬─────────────────────────────────────────┐    │
│   │ Synchronous     │ State changes only on clock edges      │    │
│   │                 │ Predictable timing, easier to design   │    │
│   ├─────────────────┼─────────────────────────────────────────┤    │
│   │ Asynchronous    │ State changes immediately on inputs    │    │
│   │                 │ Faster but prone to race conditions    │    │
│   └─────────────────┴─────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Storage Element Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STORAGE ELEMENTS                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   1. LATCH (Level-sensitive)                                        │
│      ─────────────────────────                                      │
│      • Transparent when enable is active                           │
│      • Output follows input during active period                   │
│      • Holds value when enable is inactive                         │
│                                                                      │
│      CLK                                                            │
│        │  ┌───────────────┐                                         │
│       ─┤  │  Transparent  │   Opaque    Transparent                 │
│        │  │   D → Q       │   Q held    D → Q                       │
│                                                                      │
│   2. FLIP-FLOP (Edge-sensitive)                                     │
│      ─────────────────────────                                      │
│      • Samples input only at clock edge                            │
│      • Output changes only at edge transitions                     │
│      • More timing-predictable than latches                        │
│                                                                      │
│      CLK                                                            │
│        │    ┌─────────────┐                                         │
│       ─┤    ↑             ↑   Samples only at rising edges         │
│        │    │             │                                         │
│                                                                      │
│   ┌───────────────────┬─────────────────────────────────────────┐   │
│   │ Element           │ Key Characteristic                      │   │
│   ├───────────────────┼─────────────────────────────────────────┤   │
│   │ SR Latch          │ Set-Reset, async, prone to hazards     │   │
│   │ D Latch           │ Data latch, level-sensitive            │   │
│   │ D Flip-Flop       │ Edge-triggered, most common            │   │
│   │ Master-Slave FF   │ Two latches, edge-triggered behavior   │   │
│   └───────────────────┴─────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Chapters in This Unit

| Chapter | Topic | Description |
|---------|-------|-------------|
| 5.1 | [SR and D Latches](01-sr-d-latches.md) | Basic storage elements, operation, CMOS implementation |
| 5.2 | [D Flip-Flop Design](02-d-flip-flop-design.md) | Master-slave configuration, edge triggering |
| 5.3 | [Timing Constraints](03-timing-constraints.md) | Setup time, hold time, clock-to-Q delay |
| 5.4 | [Static vs Dynamic Storage](04-static-dynamic-storage.md) | Charge storage, refresh, dynamic latches |
| 5.5 | [Clock Distribution](05-clock-distribution.md) | Clock skew, buffering, clock trees |

---

## 🔑 Key Equations Preview

| Parameter | Formula | Description |
|-----------|---------|-------------|
| Max Frequency | $f_{max} = \frac{1}{t_{cq} + t_{logic} + t_{setup}}$ | Clock period constraint |
| Hold Margin | $t_{hold} < t_{cq} + t_{logic}$ | Hold time requirement |
| Clock Skew | $\Delta t_{skew} = t_{clk,late} - t_{clk,early}$ | Arrival time difference |
| Cycle Time | $T_{cycle} > t_{cq} + t_{pd} + t_{setup} + t_{skew}$ | Complete timing |

---

## 🔗 Navigation

| Previous Unit | Course Home | Next Unit |
|--------------|-------------|-----------|
| ← [Unit 4: Combinational CMOS Logic](../04-Combinational-CMOS-Logic/README.md) | [VLSI Design Notes](../README.md) | [Unit 6: CMOS Layout →](../06-CMOS-Layout/README.md) |
