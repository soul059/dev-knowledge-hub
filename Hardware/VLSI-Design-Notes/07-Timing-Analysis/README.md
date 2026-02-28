# Unit 7: Timing Analysis

## 📋 Unit Overview

**Timing analysis** is the cornerstone of digital circuit verification, ensuring that circuits operate correctly at the desired clock frequency. This unit covers the fundamentals of delay modeling, static timing analysis, and techniques for achieving timing closure.

---

## 🎯 Unit Objectives

After completing this unit, you will be able to:
- Model gate and interconnect delays
- Perform static timing analysis (STA)
- Identify and optimize critical paths
- Understand timing constraints and margins
- Apply timing closure techniques

---

## 📊 Timing in Digital Circuits

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WHY TIMING MATTERS                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Digital circuits must meet timing requirements:                   │
│                                                                      │
│                          Clock Period = T                           │
│            ←───────────────────────────────────────────→            │
│                                                                      │
│   CLK  ────┐     ┌─────────────────────────┐     ┌─────            │
│            │     │                         │     │                  │
│            └─────┘                         └─────┘                  │
│            ↑                               ↑                        │
│     Launch edge                      Capture edge                   │
│                                                                      │
│   Data ────────┬───────────────────────────┬───────────             │
│        Valid   │    Propagation time       │  Valid                 │
│        data    │    through logic          │  data                  │
│                └───────────────────────────┘                        │
│                     ←── Must be < T ──→                            │
│                                                                      │
│   Timing constraints:                                               │
│   • Setup time: Data must be stable BEFORE clock edge              │
│   • Hold time: Data must remain stable AFTER clock edge            │
│   • Clock-to-Q: Output valid after clock edge                      │
│                                                                      │
│   If timing violated:                                               │
│   ├─ Setup violation → Wrong data captured                         │
│   ├─ Hold violation → Data corruption                              │
│   └─ Both → Unpredictable behavior                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Chapter Index

| Chapter | Title | Description |
|---------|-------|-------------|
| 7.1 | [Delay Models](01-delay-models.md) | RC delay, Elmore delay, gate delay modeling |
| 7.2 | [Static Timing Analysis](02-static-timing-analysis.md) | STA fundamentals, timing paths, constraints |
| 7.3 | [Critical Path Analysis](03-critical-path-analysis.md) | Finding and optimizing critical paths |
| 7.4 | [Wire Delay Models](04-wire-delay-models.md) | Interconnect RC modeling, Elmore delay |
| 7.5 | [Timing Closure](05-timing-closure.md) | Techniques to meet timing requirements |

---

## 🔑 Key Formulas

| Parameter | Formula |
|-----------|---------|
| Maximum Frequency | $f_{max} = \frac{1}{T_{min}} = \frac{1}{t_{cq} + t_{logic} + t_{setup}}$ |
| Setup Constraint | $t_{cq} + t_{logic} + t_{setup} \leq T$ |
| Hold Constraint | $t_{cq} + t_{logic} \geq t_{hold}$ |
| Gate Delay | $t_p \approx 0.7 \times R_{eq} \times C_{load}$ |
| Wire Delay (Elmore) | $t_d = 0.7 \times R_{total} \times C_{total}$ |
| Timing Slack | $Slack = T - (t_{cq} + t_{logic} + t_{setup})$ |

---

## 🔗 Navigation

| Previous | Home | Next |
|----------|------|------|
| ← [Unit 6: CMOS Layout](../06-CMOS-Layout/README.md) | [Course Home](../README.md) | [Delay Models →](01-delay-models.md) |
