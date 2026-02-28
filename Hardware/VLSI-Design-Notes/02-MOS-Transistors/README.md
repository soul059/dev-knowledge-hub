# Unit 2: MOS Transistors

## 📋 Unit Overview

The **Metal-Oxide-Semiconductor Field-Effect Transistor (MOSFET)** is the fundamental building block of all modern digital integrated circuits. This unit provides a comprehensive understanding of MOS transistor operation, characteristics, and design considerations essential for VLSI design.

---

## 🎯 Learning Objectives

After completing this unit, you will be able to:

1. **Explain the structure and operation** of NMOS and PMOS transistors
2. **Analyze I-V characteristics** and understand operating regions
3. **Calculate threshold voltage** and understand its dependencies
4. **Apply body effect concepts** in circuit design
5. **Account for channel length modulation** in current calculations
6. **Perform transistor sizing** for specific design requirements

---

## 📑 Chapter Contents

| Chapter | Topic | Description |
|---------|-------|-------------|
| 2.1 | [NMOS and PMOS Transistors](01-nmos-pmos-transistors.md) | Structure, operation, symbols |
| 2.2 | [I-V Characteristics](02-iv-characteristics.md) | Operating regions, current equations |
| 2.3 | [Threshold Voltage](03-threshold-voltage.md) | V_th components, dependencies |
| 2.4 | [Body Effect](04-body-effect.md) | V_SB influence, modeling |
| 2.5 | [Channel Length Modulation](05-channel-length-modulation.md) | λ parameter, output resistance |
| 2.6 | [Transistor Sizing](06-transistor-sizing.md) | W/L ratio, drive strength |

---

## 🔑 Key Concepts

### MOS Transistor as a Switch

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MOS TRANSISTOR: THE IDEAL SWITCH                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   NMOS TRANSISTOR                    PMOS TRANSISTOR                │
│                                                                      │
│   Gate (G)                           Gate (G)                        │
│      │                                  │                            │
│      │                                  │                            │
│      ▼                                  ▼                            │
│   ┌──┴──┐                            ┌──┴──┐                         │
│   │     │                            │  ○  │  ← Bubble (inversion)  │
│ ──┤     ├──                        ──┤     ├──                       │
│ S │     │ D                        S │     │ D                       │
│   └──┬──┘                            └──┬──┘                         │
│      │                                  │                            │
│      │ B (Body)                         │ B (Body)                   │
│                                                                      │
│   Switch Behavior:                   Switch Behavior:                │
│   ┌──────────┬─────────┐            ┌──────────┬─────────┐          │
│   │ V_GS     │ State   │            │ V_GS     │ State   │          │
│   ├──────────┼─────────┤            ├──────────┼─────────┤          │
│   │ < V_th   │  OFF    │            │ > V_th   │  OFF    │          │
│   │ > V_th   │  ON     │            │ < V_th   │  ON     │          │
│   └──────────┴─────────┘            └──────────┴─────────┘          │
│                                                                      │
│   (V_th positive for NMOS)          (V_th negative for PMOS)        │
│   (Typically ~0.3-0.5V)             (Typically ~-0.3 to -0.5V)      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### CMOS Complementary Operation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMPLEMENTARY MOS (CMOS) CONCEPT                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                          VDD                                         │
│                           │                                          │
│                      ┌────┴────┐                                     │
│                      │  PMOS   │  ← Pull-up network                 │
│              ┌───────┤         │                                     │
│              │       └────┬────┘                                     │
│              │            │                                          │
│   Input ─────┤            ├─────────► Output                        │
│              │            │                                          │
│              │       ┌────┴────┐                                     │
│              └───────┤  NMOS   │  ← Pull-down network               │
│                      │         │                                     │
│                      └────┬────┘                                     │
│                           │                                          │
│                          GND                                         │
│                                                                      │
│   KEY PRINCIPLE:                                                    │
│   • PMOS and NMOS are COMPLEMENTARY                                 │
│   • When PMOS is ON, NMOS is OFF (and vice versa)                  │
│   • Output always connected to VDD or GND                           │
│   • No static current path (low power!)                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📐 Fundamental Equations

### MOS Current Equations Summary

| Region | Condition | Current Equation |
|--------|-----------|------------------|
| Cutoff | $V_{GS} < V_{th}$ | $I_D = 0$ |
| Linear (Triode) | $V_{GS} > V_{th}$, $V_{DS} < V_{GS} - V_{th}$ | $I_D = \mu_n C_{ox}\frac{W}{L}\left[(V_{GS}-V_{th})V_{DS} - \frac{V_{DS}^2}{2}\right]$ |
| Saturation | $V_{GS} > V_{th}$, $V_{DS} \geq V_{GS} - V_{th}$ | $I_D = \frac{1}{2}\mu_n C_{ox}\frac{W}{L}(V_{GS}-V_{th})^2$ |

### Key Parameters

| Parameter | Symbol | Typical Value (65nm) | Description |
|-----------|--------|---------------------|-------------|
| Threshold Voltage | $V_{th}$ | 0.3-0.4 V | Turn-on voltage |
| Electron Mobility | $\mu_n$ | 300 cm²/V·s | Carrier mobility (NMOS) |
| Hole Mobility | $\mu_p$ | 100 cm²/V·s | Carrier mobility (PMOS) |
| Oxide Capacitance | $C_{ox}$ | 10 fF/μm² | Gate oxide capacitance |
| Process Transconductance | $k'_n = \mu_n C_{ox}$ | 200 μA/V² | NMOS process parameter |
| Process Transconductance | $k'_p = \mu_p C_{ox}$ | 70 μA/V² | PMOS process parameter |

---

## 📊 Unit Summary Table

| Topic | Key Points |
|-------|------------|
| NMOS | n-channel, turns ON when V_GS > V_th (positive) |
| PMOS | p-channel, turns ON when V_GS < V_th (negative) |
| I-V Regions | Cutoff, Linear (Triode), Saturation |
| Threshold | Depends on oxide, doping, work function |
| Body Effect | V_SB increases effective threshold |
| CLM | Finite output resistance in saturation |
| Sizing | W/L ratio controls current drive |
| μ_n/μ_p | ~2-3, PMOS slower than NMOS |

---

## 🧮 Quick Reference Formulas

$$I_{D,sat} = \frac{1}{2}\mu C_{ox}\frac{W}{L}(V_{GS}-V_{th})^2(1+\lambda V_{DS})$$

$$V_{th} = V_{th0} + \gamma(\sqrt{|2\phi_F + V_{SB}|} - \sqrt{|2\phi_F|})$$

$$g_m = \frac{\partial I_D}{\partial V_{GS}} = \mu C_{ox}\frac{W}{L}(V_{GS}-V_{th}) = \sqrt{2\mu C_{ox}\frac{W}{L}I_D}$$

$$r_o = \frac{1}{\lambda I_D}$$

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| ← [Unit 1: Introduction](../01-Introduction-to-VLSI/README.md) | [Course Home](../README.md) | [Unit 3: CMOS Inverter](../03-CMOS-Inverter/README.md) → |

---

### Chapter Navigation

| Chapter | Link |
|---------|------|
| Start Here → | [2.1 NMOS and PMOS Transistors](01-nmos-pmos-transistors.md) |
