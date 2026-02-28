# Unit 8: Power Optimization

## 📋 Unit Overview

**Power consumption** is a critical design constraint in modern VLSI, affecting battery life, thermal management, and operating costs. This unit covers the sources of power dissipation and techniques to minimize power while maintaining performance.

---

## 🎯 Unit Objectives

After completing this unit, you will be able to:
- Identify and quantify power dissipation sources
- Apply circuit and architecture-level power reduction techniques
- Understand dynamic and static power trade-offs
- Design power-efficient CMOS circuits

---

## ⚡ Power Components in CMOS

```
┌─────────────────────────────────────────────────────────────────────┐
│                    POWER DISSIPATION SOURCES                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Total power = Dynamic power + Static power                       │
│                                                                      │
│   $P_{total} = P_{dynamic} + P_{static}$                           │
│                                                                      │
│   $P_{total} = (P_{switching} + P_{short-circuit}) + P_{leakage}$  │
│                                                                      │
│                                                                      │
│   Power breakdown in modern technologies:                           │
│                                                                      │
│       ┌──────────────────────────────────────────────────────────┐  │
│       │                                                          │  │
│       │  Old technology         Modern technology (≤14nm)       │  │
│       │  (180nm+)                                                │  │
│       │                                                          │  │
│       │  ┌────────────────┐    ┌────────────────────────────┐   │  │
│       │  │████████████████│    │████████████│░░░░░░░░░░░░░░│   │  │
│       │  │████ Dynamic ███│    │██ Dynamic █│░░ Leakage ░░░│   │  │
│       │  │████  (95%)  ███│    │██  (50%)  █│░░  (50%)  ░░░│   │  │
│       │  │████████████████│    │████████████│░░░░░░░░░░░░░░│   │  │
│       │  └────────────────┘    └────────────────────────────┘   │  │
│       │                                                          │  │
│       │  Leakage was negligible   Leakage now major concern     │  │
│       │                                                          │  │
│       └──────────────────────────────────────────────────────────┘  │
│                                                                      │
│   Why power matters:                                                │
│   • Mobile devices: Battery life                                   │
│   • Data centers: Electricity cost, cooling                       │
│   • Embedded: Thermal limits, reliability                         │
│   • IoT: Energy harvesting constraints                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Chapter Index

| Chapter | Title | Description |
|---------|-------|-------------|
| 8.1 | [Dynamic Power](01-dynamic-power.md) | Switching and short-circuit power |
| 8.2 | [Static Power](02-static-power.md) | Leakage mechanisms and modeling |
| 8.3 | [Power Reduction Techniques](03-power-reduction-techniques.md) | Circuit and logic-level techniques |
| 8.4 | [Voltage Scaling](04-voltage-scaling.md) | DVFS, multi-Vdd design |
| 8.5 | [Power Gating](05-power-gating.md) | Sleep modes and power switches |

---

## 🔑 Key Formulas

| Parameter | Formula |
|-----------|---------|
| Dynamic Power | $P_{dyn} = \alpha C_L V_{DD}^2 f$ |
| Short-Circuit Power | $P_{sc} = I_{sc} \cdot V_{DD} \cdot t_{sc} \cdot f$ |
| Subthreshold Leakage | $I_{sub} = I_0 e^{(V_{GS}-V_t)/nV_T}$ |
| Total Leakage Power | $P_{leak} = I_{leak} \cdot V_{DD}$ |
| Energy per Operation | $E = C_L V_{DD}^2$ |
| Power-Delay Product | $PDP = P \cdot t_p$ |

---

## 🔗 Navigation

| Previous | Home | Next |
|----------|------|------|
| ← [Unit 7: Timing Analysis](../07-Timing-Analysis/README.md) | [Course Home](../README.md) | [Dynamic Power →](01-dynamic-power.md) |
