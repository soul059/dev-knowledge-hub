# Unit 3: CMOS Inverter

## 📋 Unit Overview

The **CMOS inverter** is the fundamental building block of all digital CMOS circuits. Understanding its behavior—from static characteristics to dynamic performance—is essential for designing any CMOS logic circuit. This unit provides a comprehensive analysis of the CMOS inverter's operation, performance metrics, and design considerations.

---

## 🎯 Unit Objectives

After completing this unit, you will be able to:
- Analyze voltage transfer characteristics and define noise margins
- Determine switching threshold and its dependence on sizing
- Evaluate dynamic behavior including propagation delay
- Calculate power dissipation (static and dynamic)
- Optimize inverter design for speed and power

---

## 📚 Chapters in This Unit

| Chapter | Title | Key Topics |
|---------|-------|------------|
| 3.1 | [Static Characteristics](01-static-characteristics.md) | VTC, operating regions, voltage levels |
| 3.2 | [Switching Threshold](02-switching-threshold.md) | VM calculation, sizing for threshold |
| 3.3 | [Noise Margins](03-noise-margins.md) | NMH, NML, robustness criteria |
| 3.4 | [Dynamic Characteristics](04-dynamic-characteristics.md) | Rise/fall times, propagation delay |
| 3.5 | [Power Dissipation](05-power-dissipation.md) | Static, dynamic, short-circuit power |
| 3.6 | [Delay Analysis](06-delay-analysis.md) | RC delay, logical effort |

---

## 🔑 Key Concepts Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CMOS INVERTER STRUCTURE                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                    VDD                                               │
│                     │                                                │
│              ┌──────┴──────┐                                         │
│              │    PMOS     │ Pull-up network                        │
│              │  (ON when   │                                         │
│              │   Vin=LOW)  │                                         │
│              └──────┬──────┘                                         │
│                     │                                                │
│   V_in ─────────────┼─────────────► V_out                           │
│                     │                                                │
│              ┌──────┴──────┐                                         │
│              │    NMOS     │ Pull-down network                      │
│              │  (ON when   │                                         │
│              │   Vin=HIGH) │                                         │
│              └──────┬──────┘                                         │
│                     │                                                │
│                    GND                                               │
│                                                                      │
│   Key Characteristics:                                              │
│   • Complementary operation: One ON, other OFF                      │
│   • Full rail-to-rail output swing                                  │
│   • Very low static power (ideally zero)                            │
│   • Inverting logic function: Vout = NOT(Vin)                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📐 Essential Equations

| Parameter | Equation | Description |
|-----------|----------|-------------|
| Switching Threshold | $V_M = \frac{V_{DD} + V_{tp} + V_{tn}\sqrt{\frac{\beta_n}{\beta_p}}}{1 + \sqrt{\frac{\beta_n}{\beta_p}}}$ | Input voltage where V_in = V_out |
| Noise Margin High | $NM_H = V_{OH} - V_{IH}$ | Tolerance for HIGH input |
| Noise Margin Low | $NM_L = V_{IL} - V_{OL}$ | Tolerance for LOW input |
| Propagation Delay | $t_p = \frac{C_L \cdot V_{DD}}{2 \cdot I_{avg}}$ | Average delay |
| Dynamic Power | $P_{dyn} = \alpha \cdot C_L \cdot V_{DD}^2 \cdot f$ | Switching power |

---

## 📊 Voltage Transfer Characteristic (Preview)

```
    V_out
      │
  VDD ┤●●●●●●●●●●────┐
      │              │
      │              │
VOH   ┤              │
      │              ●
      │               ●
      │                ●
      │                 ●    Transition region
      │                  ●
      │                   ●
VOL   ┤                    ●
      │                    │
    0 ┼────────────────────┴●●●●●●●●●●●●
      0   VIL   VM   VIH           VDD
                  V_in
```

---

## 🔗 Navigation

| Previous Unit | Course Home | Next Unit |
|--------------|-------------|-----------|
| ← [Unit 2: MOS Transistors](../02-MOS-Transistors/README.md) | [Course Overview](../README.md) | [Unit 4: Combinational Logic →](../04-Combinational-CMOS-Logic/README.md) |
