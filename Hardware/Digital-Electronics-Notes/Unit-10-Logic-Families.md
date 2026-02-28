# Unit 10: Logic Families and Digital IC Characteristics

---

## Table of Contents
1. [Introduction to Logic Families](#1-introduction-to-logic-families)
2. [Digital IC Characteristics](#2-digital-ic-characteristics)
3. [TTL (Transistor-Transistor Logic)](#3-ttl-transistor-transistor-logic)
4. [CMOS (Complementary MOS)](#4-cmos-complementary-mos)
5. [ECL (Emitter-Coupled Logic)](#5-ecl-emitter-coupled-logic)
6. [BiCMOS Technology](#6-bicmos-technology)
7. [Logic Family Comparison](#7-logic-family-comparison)
8. [Interfacing Between Families](#8-interfacing-between-families)
9. [Practical Considerations](#9-practical-considerations)
10. [Summary Tables](#10-summary-tables)
11. [Quick Revision Questions](#11-quick-revision-questions)

---

## 1. Introduction to Logic Families

### What is a Logic Family?

```
┌─────────────────────────────────────────────────────────────────┐
│                    LOGIC FAMILIES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DEFINITION:                                                    │
│    A group of digital ICs that share:                          │
│      • Same technology (transistor type)                       │
│      • Compatible voltage levels                               │
│      • Similar electrical characteristics                      │
│      • Can be directly interconnected                          │
│                                                                 │
│  MAIN FAMILIES:                                                 │
│                                                                 │
│    ┌─────────────────────────────────────────────────┐         │
│    │                                                 │         │
│    │     BIPOLAR              MOS                   │         │
│    │        │                  │                    │         │
│    │   ┌────┴────┐      ┌──────┴──────┐            │         │
│    │   │         │      │             │            │         │
│    │  TTL       ECL    NMOS         CMOS           │         │
│    │   │                              │             │         │
│    │   ├── Standard                BiCMOS          │         │
│    │   ├── LS (Low-power Schottky)  (hybrid)       │         │
│    │   ├── ALS                                     │         │
│    │   ├── F (Fast)                                │         │
│    │   └── AS (Advanced Schottky)                  │         │
│    │                                                 │         │
│    └─────────────────────────────────────────────────┘         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Evolution of Logic Families

```
HISTORICAL DEVELOPMENT:

1960s:
  ┌─────────────────────────────────────────────────────────────┐
  │  RTL → DTL → TTL                                           │
  │  (Resistor)  (Diode)  (Transistor-Transistor Logic)       │
  │                                                             │
  │  First integrated logic families                           │
  └─────────────────────────────────────────────────────────────┘

1970s:
  ┌─────────────────────────────────────────────────────────────┐
  │  ECL (for speed)                                           │
  │  NMOS (for density)                                        │
  │  CMOS introduced (for low power)                           │
  └─────────────────────────────────────────────────────────────┘

1980s-Present:
  ┌─────────────────────────────────────────────────────────────┐
  │  CMOS dominates                                            │
  │  BiCMOS for specialized applications                       │
  │  Low-voltage CMOS (3.3V, 2.5V, 1.8V, 1.2V...)            │
  └─────────────────────────────────────────────────────────────┘

TODAY:
  > 95% of digital ICs are CMOS
  CMOS offers: Low power, high density, wide voltage range
```

---

## 2. Digital IC Characteristics

### Key Parameters

```
┌─────────────────────────────────────────────────────────────────┐
│                 DIGITAL IC PARAMETERS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. VOLTAGE LEVELS:                                             │
│     VIH: Minimum input voltage for HIGH                        │
│     VIL: Maximum input voltage for LOW                         │
│     VOH: Minimum output voltage for HIGH                       │
│     VOL: Maximum output voltage for LOW                        │
│                                                                 │
│  2. NOISE MARGIN:                                               │
│     NMH = VOH - VIH (High-state noise margin)                  │
│     NML = VIL - VOL (Low-state noise margin)                   │
│                                                                 │
│  3. POWER DISSIPATION:                                          │
│     Static power (when not switching)                          │
│     Dynamic power (during switching)                           │
│                                                                 │
│  4. PROPAGATION DELAY:                                          │
│     tPHL: Delay for HIGH to LOW transition                     │
│     tPLH: Delay for LOW to HIGH transition                     │
│     tP = (tPHL + tPLH) / 2                                     │
│                                                                 │
│  5. FAN-OUT:                                                    │
│     Number of inputs one output can drive                      │
│                                                                 │
│  6. FAN-IN:                                                     │
│     Number of inputs on a gate                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Voltage Level Definitions

```
VOLTAGE LEVELS AND NOISE MARGINS:

       VCC ─────┬─────────────────────────────────────────
                │
       VOH(min)─┼─────────────────┐
                │                 │   VALID HIGH
       VIH(min)─┼─────────────────┤   OUTPUT RANGE
                │                 │
                │  ┌──────────────┤
                │  │  Noise       │
                │  │  Margin      │ NMH = VOH - VIH
                │  │  (HIGH)      │
                │  └──────────────┤
       VIH ─────┼─────────────────┘
                │
                │      UNDEFINED
                │      REGION
                │      (Forbidden)
                │
       VIL ─────┼─────────────────┐
                │  ┌──────────────┤
                │  │  Noise       │
                │  │  Margin      │ NML = VIL - VOL
                │  │  (LOW)       │
                │  └──────────────┤
       VIL(max)─┼─────────────────┤
                │                 │   VALID LOW
       VOL(max)─┼─────────────────┤   OUTPUT RANGE
                │                 │
       GND ─────┴─────────────────┴───────────────────────

NOISE IMMUNITY:
  Higher noise margin = better noise immunity
  Signal can be corrupted by up to NM before error
```

### Propagation Delay

```
PROPAGATION DELAY MEASUREMENT:

       Input
    ────────┐          ┌─────────────────
            │          │
            │          │
            └──────────┘
                        
                ◄─────────►
                   tPHL
                   
       Output
    ────────────────┐          ┌─────────
                    │          │
                    │          │
                    └──────────┘
                    
                        ◄─────────►
                           tPLH

    Measured at 50% points of transitions
    
    Average propagation delay:
    tP = (tPHL + tPLH) / 2

SPEED-POWER PRODUCT:

    Speed-Power Product = tP × PD
    
    Lower is better!
    
    Typical values:
      TTL: ~100 pJ
      CMOS: ~1-10 pJ (varies with frequency)
      ECL: ~50 pJ
```

### Fan-Out Calculation

```
FAN-OUT:

                    ┌───────────────────────────────────────┐
                    │                                       │
    ┌────┐          │   ┌────┐                             │
    │    ├──────────┼──►│    │  Load 1                     │
    │    │          │   └────┘                             │
    │Gate│          │   ┌────┐                             │
    │    ├──────────┼──►│    │  Load 2                     │
    │    │          │   └────┘                             │
    │    │          │   ┌────┐                             │
    │    ├──────────┼──►│    │  Load 3                     │
    │    │          │   └────┘                             │
    └────┘          │     ...                               │
   Driver           │   ┌────┐                             │
                    └──►│    │  Load N                     │
                        └────┘                             │

DC FAN-OUT:

For HIGH state:
  Fan-out(H) = IOH / IIH
  
For LOW state:
  Fan-out(L) = IOL / IIL
  
Fan-out = min(Fan-out(H), Fan-out(L))

EXAMPLE (Standard TTL):
  IOH = -400 μA, IIH = 40 μA
  IOL = 16 mA, IIL = 1.6 mA
  
  Fan-out(H) = 400/40 = 10
  Fan-out(L) = 16000/1600 = 10
  
  Fan-out = 10 (can drive 10 TTL inputs)
```

---

## 3. TTL (Transistor-Transistor Logic)

### Basic TTL Inverter

```
TTL NAND GATE (2-INPUT):

       VCC (+5V)
         │
         │
        [R1] 4kΩ
         │
         ├───────────────────────────┐
         │                           │
    ┌────┴────┐                     [R2] 1.6kΩ
    │    E    │                      │
A ──┤    B    ├─────┐                ├────── Y (Output)
    │    C    │     │                │
    └────┬────┘     │           ┌────┴────┐
         │          │           │ Q2   C  │
         │          │           │    E    │
B ───────┤          └───────────┤    B    │
         │                      └────┬────┘
         │                           │
         │                          [R3] 1kΩ
         │                           │
        ─┴─                         ─┴─
        GND                         GND

    Q1: Multi-emitter input transistor
    Q2: Phase splitter
    (Simplified - actual has totem-pole output)

TOTEM-POLE OUTPUT STAGE:

       VCC
         │
        [R]
         │
    ┌────┴────┐
    │  Q3     │  ← Upper transistor (HIGH driver)
    └────┬────┘
         │
         ├─────────────── Output
         │
    ┌────┴────┐
    │  Q4     │  ← Lower transistor (LOW driver)
    └────┬────┘
         │
        ─┴─
        GND

When output HIGH: Q3 ON, Q4 OFF
When output LOW:  Q3 OFF, Q4 ON
Never both ON (would short VCC to GND!)
```

### TTL Subfamilies

```
TTL SUBFAMILY COMPARISON:

┌────────────┬────────┬──────────┬───────────┬──────────────┐
│ Subfamily  │  tP    │   PD     │ Speed-Pwr │ Description  │
│            │  (ns)  │  (mW)    │  Product  │              │
├────────────┼────────┼──────────┼───────────┼──────────────┤
│ Standard   │   10   │   10     │   100 pJ  │ Original     │
│ 74xx       │        │          │           │              │
├────────────┼────────┼──────────┼───────────┼──────────────┤
│ Low-Power  │   33   │    1     │   33 pJ   │ Low power,   │
│ 74Lxx      │        │          │           │ slow         │
├────────────┼────────┼──────────┼───────────┼──────────────┤
│ Schottky   │    3   │   19     │   57 pJ   │ Fast,        │
│ 74Sxx      │        │          │           │ high power   │
├────────────┼────────┼──────────┼───────────┼──────────────┤
│ Low-Power  │   10   │    2     │   20 pJ   │ Good         │
│ Schottky   │        │          │           │ compromise   │
│ 74LSxx     │        │          │           │ (popular)    │
├────────────┼────────┼──────────┼───────────┼──────────────┤
│ Advanced   │    4   │    1     │    4 pJ   │ Improved LS  │
│ Low-Power  │        │          │           │              │
│ Schottky   │        │          │           │              │
│ 74ALSxx    │        │          │           │              │
├────────────┼────────┼──────────┼───────────┼──────────────┤
│ Fast       │    3   │    6     │   18 pJ   │ Very fast    │
│ 74Fxx      │        │          │           │              │
└────────────┴────────┴──────────┴───────────┴──────────────┘

SCHOTTKY CLAMPING:
  Uses Schottky diode to prevent transistor saturation
  Prevents storage time delay
  Results in faster switching
```

### TTL Characteristics

```
TTL VOLTAGE SPECIFICATIONS (74LS Series):

       +5V ───────────────────────────────────────────────
                   │
       VOH(min) ───┼─── 2.7V ─────────┐
                   │                  │ Valid HIGH
       VIH(min) ───┼─── 2.0V ─────────┤ output region
                   │                  │
                   │  Noise Margin    │ NMH = 2.7 - 2.0
                   │  (HIGH) = 0.7V   │     = 0.7V
                   │                  │
       VIL(max) ───┼─── 0.8V ─────────┤
                   │                  │
                   │  Noise Margin    │ NML = 0.8 - 0.5
                   │  (LOW) = 0.3V    │     = 0.3V
                   │                  │
       VOL(max) ───┼─── 0.5V ─────────┤ Valid LOW
                   │                  │ output region
                   │                  │
       GND ───────────────────────────┴───────────────────

TTL SPECIFICATIONS SUMMARY:
┌─────────────────┬──────────────────┐
│   Parameter     │    Value         │
├─────────────────┼──────────────────┤
│ VCC             │ 5V ± 5%          │
│ VOH(min)        │ 2.7V (2.4V std)  │
│ VOL(max)        │ 0.5V (0.4V std)  │
│ VIH(min)        │ 2.0V             │
│ VIL(max)        │ 0.8V             │
│ IIH             │ 20μA (LS)        │
│ IIL             │ -0.4mA (LS)      │
│ IOH             │ -400μA (LS)      │
│ IOL             │ 8mA (LS)         │
│ Fan-out         │ 20 (LS)          │
└─────────────────┴──────────────────┘
```

---

## 4. CMOS (Complementary MOS)

### Basic CMOS Inverter

```
CMOS INVERTER:

       VDD
         │
    ┌────┴────┐
    │         │
    │  PMOS   │ ← P-channel MOSFET
    │         │
    └────┬────┘
         │
A ───────┼──────────────── Y (Output)
         │
    ┌────┴────┐
    │         │
    │  NMOS   │ ← N-channel MOSFET
    │         │
    └────┬────┘
         │
        ─┴─
        GND

OPERATION:

Input A = LOW (0V):
  PMOS: Gate low, VGS negative → ON
  NMOS: Gate low, VGS = 0 → OFF
  Output Y = VDD (HIGH) ✓

Input A = HIGH (VDD):
  PMOS: Gate high, VGS = 0 → OFF
  NMOS: Gate high, VGS positive → ON
  Output Y = GND (LOW) ✓

TRUTH TABLE:
┌───────┬────────┐
│   A   │   Y    │
├───────┼────────┤
│   0   │   1    │
│   1   │   0    │
└───────┴────────┘

KEY ADVANTAGE:
  One transistor always OFF
  → No static DC path
  → Near-zero static power!
```

### CMOS NAND and NOR Gates

```
CMOS 2-INPUT NAND:

       VDD
         │
    ┌────┴────┐
    │  PMOS   │────┐
    └────┬────┘    │
   A ────┤         │
         │         │
    ┌────┴────┐    │
    │  PMOS   │────┼──── Y = (A·B)'
    └────┬────┘    │
   B ────┤         │
         │         │
    ┌────┴────┐    │
   A ────┤  NMOS   │────┘
    └────┬────┘
         │
    ┌────┴────┐
   B ────┤  NMOS   │
    └────┬────┘
         │
        ─┴─ GND

PMOS: In parallel (either ON = HIGH output)
NMOS: In series (both ON = LOW output)


CMOS 2-INPUT NOR:

       VDD
         │
    ┌────┴────┐
   A ────┤  PMOS   │
    └────┬────┘
         │
    ┌────┴────┐
   B ────┤  PMOS   │───── Y = (A+B)'
    └────┬────┘
         │
         ├─────────────┐
         │             │
    ┌────┴────┐   ┌────┴────┐
   A ────┤  NMOS   │   │  NMOS   │──── B
    └────┬────┘   └────┬────┘
         │             │
         └──────┬──────┘
               ─┴─ GND

PMOS: In series (both OFF = HIGH output)
NMOS: In parallel (either ON = LOW output)
```

### CMOS Characteristics

```
CMOS ADVANTAGES:

1. EXTREMELY LOW STATIC POWER:
   - No DC path when stable
   - Only leakage current (~nA)
   - Static power ≈ 10 nW per gate

2. HIGH NOISE MARGIN:
   - Symmetric around VDD/2
   - NMH ≈ NML ≈ 0.45 × VDD
   - Better than TTL

3. WIDE VOLTAGE RANGE:
   - 4000 series: 3V to 18V
   - 74HC: 2V to 6V
   - Modern: 1.2V to 3.3V

4. HIGH FAN-OUT:
   - Input is essentially capacitive
   - DC fan-out > 50 (capacitive load limited)

5. RAIL-TO-RAIL OUTPUT:
   - VOH ≈ VDD
   - VOL ≈ GND

CMOS DISADVANTAGES:

1. DYNAMIC POWER:
   PD = C × VDD² × f
   - Power increases with frequency
   - Dominant at high speeds

2. SPEED (older CMOS):
   - Slower than TTL/ECL
   - Modern CMOS competitive

3. ESD SENSITIVITY:
   - Input protection required
   - Handle with care
```

### CMOS Subfamilies

```
CMOS SUBFAMILY COMPARISON:

┌────────────┬────────┬──────────┬───────────┬───────────────┐
│ Subfamily  │  VDD   │  tP(ns)  │ Compatible│  Description  │
├────────────┼────────┼──────────┼───────────┼───────────────┤
│ 4000B      │ 3-18V  │  50-100  │   CMOS    │ Original CMOS │
│            │        │          │           │ (slow)        │
├────────────┼────────┼──────────┼───────────┼───────────────┤
│ 74HC       │ 2-6V   │   10     │   CMOS    │ High-speed    │
│            │        │          │           │ CMOS          │
├────────────┼────────┼──────────┼───────────┼───────────────┤
│ 74HCT      │ 5V     │   10     │   TTL     │ HC with TTL   │
│            │        │          │           │ thresholds    │
├────────────┼────────┼──────────┼───────────┼───────────────┤
│ 74AC       │ 2-5.5V │    3     │   CMOS    │ Advanced CMOS │
│            │        │          │           │               │
├────────────┼────────┼──────────┼───────────┼───────────────┤
│ 74ACT      │ 5V     │    3     │   TTL     │ AC with TTL   │
│            │        │          │           │ thresholds    │
├────────────┼────────┼──────────┼───────────┼───────────────┤
│ 74LVC      │ 1.65-  │   3-5    │   TTL     │ Low-voltage   │
│            │ 3.6V   │          │           │ CMOS          │
├────────────┼────────┼──────────┼───────────┼───────────────┤
│ 74AUC      │ 0.8-   │   1-2    │   CMOS    │ Ultra-low     │
│            │ 2.7V   │          │           │ voltage       │
└────────────┴────────┴──────────┴───────────┴───────────────┘

HC vs HCT:
  HC: CMOS input levels (VIH = 0.7×VDD, VIL = 0.3×VDD)
  HCT: TTL input levels (VIH = 2.0V, VIL = 0.8V)
  Use HCT to interface with TTL!
```

### CMOS Power Dissipation

```
CMOS POWER COMPONENTS:

                    Total Power
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       Static        Dynamic       Short-Circuit
       Power          Power           Power
          │              │              │
     ▼─────────▼    ▼─────────▼    ▼─────────▼
     
     Pstatic =      Pdynamic =     Psc = ISC×VDD
     Ileakage×VDD   CL×VDD²×f     (during transition)
     (very small)   (dominant)

DYNAMIC POWER EQUATION:

Pdynamic = CL × VDD² × f × α

Where:
  CL = Load capacitance
  VDD = Supply voltage
  f = Clock frequency
  α = Activity factor (0-1)

EXAMPLE:
  CL = 10 pF
  VDD = 3.3V
  f = 100 MHz
  α = 0.25

  Pdynamic = 10×10⁻¹² × (3.3)² × 100×10⁶ × 0.25
           = 10×10⁻¹² × 10.89 × 25×10⁶
           = 2.72 mW per gate

POWER REDUCTION STRATEGIES:
  1. Lower VDD (quadratic effect!)
  2. Lower frequency
  3. Reduce capacitance
  4. Clock gating (reduce α)
```

---

## 5. ECL (Emitter-Coupled Logic)

### ECL Basic Circuit

```
ECL OR/NOR GATE:

           VCC (Ground in ECL)
             │
    ┌────────┼────────────────────────────────┐
    │        │                                │
    │       [RC1]                [RC2]        │
    │        │                    │           │
    │        ├──────────┬─────────┤           │
    │        │          │         │           │
    │   ┌────┴────┐     │    ┌────┴────┐     │
    │   │         │     │    │         │     │
A ──┤──►│   Q1    │     │    │   Q3    │◄────┤── VREF
    │   │         │     │    │   (ref) │     │   (-1.3V)
    │   └────┬────┘     │    └────┬────┘     │
    │        │          │         │           │
B ──┤──►┌────┴────┐     │         │           │
    │   │         │     │         │           │
    │   │   Q2    │     │         │           │
    │   │         │     │         │           │
    │   └────┬────┘     │         │           │
    │        │          │         │           │
    │        └──────────┴─────────┘           │
    │                   │                     │
    │                  [RE] Current source    │
    │                   │                     │
    │                  ─┴─                    │
    │                 VEE (-5.2V)             │
    │                                         │
    │        NOR ─────────┤        OR ───────┤
    │        output       │        output    │
    │                     │                  │
    │     ┌───────────────┤    ┌─────────────┤
    │     │               │    │             │
    │  ┌──┴──┐         ┌──┴──┐              │
    │  │ EF  │         │ EF  │ Emitter      │
    │  │     │         │     │ followers    │
    │  └──┬──┘         └──┬──┘              │
    │     │               │                  │
    └─────┴───────────────┴──────────────────┘
          NOR             OR

KEY FEATURES:
  • Differential amplifier (Q1/Q2 vs Q3)
  • Non-saturating transistors (FAST!)
  • Small voltage swing (~0.8V)
  • Complementary outputs (OR and NOR)
```

### ECL Characteristics

```
ECL VOLTAGE LEVELS:

       GND (VCC) ────────────────────────────────────
                    │
       VOH ─────────┼─── -0.9V ───────┐
                    │                  │ HIGH level
                    │                  │
                    │    ~0.8V         │ Voltage swing
                    │    swing         │ (small = fast!)
                    │                  │
       VOL ─────────┼─── -1.7V ───────┤
                    │                  │ LOW level
                    │                  │
       VEE ─────────────── -5.2V ─────┴───────────────


ECL ADVANTAGES:
  1. VERY FAST (sub-nanosecond)
  2. Non-saturating (no storage time)
  3. Complementary outputs
  4. Constant current (low noise)
  5. Small voltage swing

ECL DISADVANTAGES:
  1. HIGH POWER DISSIPATION (~25mW/gate)
  2. Negative supply voltages
  3. Small noise margin (~0.3V)
  4. Complex interface with other families
  5. Expensive

ECL SPECIFICATIONS:
┌─────────────────┬──────────────────┐
│   Parameter     │    Value         │
├─────────────────┼──────────────────┤
│ VEE             │ -5.2V            │
│ VOH             │ -0.9V            │
│ VOL             │ -1.7V            │
│ VIH             │ -1.1V            │
│ VIL             │ -1.5V            │
│ Propagation delay│ 0.5-1 ns        │
│ Power/gate      │ 25-40 mW         │
│ Noise margin    │ ~0.3V            │
└─────────────────┴──────────────────┘

APPLICATIONS:
  • High-speed computing
  • Telecommunications
  • Supercomputers (historical)
  • Test equipment
```

---

## 6. BiCMOS Technology

### BiCMOS Concept

```
┌─────────────────────────────────────────────────────────────────┐
│                        BiCMOS                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Combines CMOS and Bipolar transistors on same chip            │
│                                                                 │
│  CMOS: Low power, high density                                 │
│  Bipolar: High drive capability, fast                          │
│                                                                 │
│  BiCMOS = Best of both!                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

BiCMOS INVERTER:

       VDD
         │
    ┌────┴────┐
    │  PMOS   │──────┐
    └────┬────┘      │
   A ────┤           │
         │      ┌────┴────┐
    ┌────┴────┐ │   NPN   │──────── Y
    │  NMOS   │ │   Q1    │
    └────┬────┘ └────┬────┘
         │           │
         │      ┌────┴────┐
         └──────┤   NPN   │
                │   Q2    │
                └────┬────┘
                     │
                    ─┴─
                    GND

CMOS: Logic operation, low quiescent current
Bipolar: Output drivers, high current capability

ADVANTAGES:
  • High drive current (bipolar output)
  • Low input power (CMOS input)
  • Fast switching
  • Good for driving large capacitive loads
```

---

## 7. Logic Family Comparison

### Comprehensive Comparison

```
LOGIC FAMILY COMPARISON TABLE:

┌──────────────┬────────┬────────┬────────┬────────┬──────────┐
│  Parameter   │  TTL   │  CMOS  │  ECL   │ BiCMOS │ Low-V    │
│              │  (LS)  │  (HC)  │        │        │ CMOS     │
├──────────────┼────────┼────────┼────────┼────────┼──────────┤
│ VDD          │  5V    │ 2-6V   │ -5.2V  │  5V    │ 1.8-3.3V │
├──────────────┼────────┼────────┼────────┼────────┼──────────┤
│ VOH (min)    │ 2.7V   │ VDD-   │ -0.9V  │ 3.5V   │ VDD-0.1  │
│              │        │ 0.1V   │        │        │          │
├──────────────┼────────┼────────┼────────┼────────┼──────────┤
│ VOL (max)    │ 0.5V   │ 0.1V   │ -1.7V  │ 0.5V   │ 0.1V     │
├──────────────┼────────┼────────┼────────┼────────┼──────────┤
│ VIH (min)    │ 2.0V   │ 0.7VDD │ -1.1V  │ 2.0V   │ 0.7VDD   │
├──────────────┼────────┼────────┼────────┼────────┼──────────┤
│ VIL (max)    │ 0.8V   │ 0.3VDD │ -1.5V  │ 0.8V   │ 0.3VDD   │
├──────────────┼────────┼────────┼────────┼────────┼──────────┤
│ Noise margin │ 0.3-   │ ~0.45  │ ~0.2V  │ ~0.4V  │ ~0.4VDD  │
│              │ 0.7V   │ VDD    │        │        │          │
├──────────────┼────────┼────────┼────────┼────────┼──────────┤
│ Prop. delay  │ 10ns   │ 10ns   │ 1ns    │ 3ns    │ 3-5ns    │
├──────────────┼────────┼────────┼────────┼────────┼──────────┤
│ Static power │ 2mW    │ ~0     │ 25mW   │ 0.5mW  │ ~0       │
├──────────────┼────────┼────────┼────────┼────────┼──────────┤
│ Fan-out      │ 20     │ >50    │ >10    │ >30    │ >50      │
├──────────────┼────────┼────────┼────────┼────────┼──────────┤
│ Density      │ Low    │ High   │ Low    │ Medium │ Very High│
└──────────────┴────────┴────────┴────────┴────────┴──────────┘
```

### Selection Guidelines

```
WHEN TO USE EACH FAMILY:

TTL (74LS, 74F):
  ✓ Legacy system maintenance
  ✓ Simple prototyping
  ✓ Industrial environments
  ✗ New designs (use CMOS instead)

CMOS (74HC, 74AC):
  ✓ Battery-powered devices
  ✓ Low power applications
  ✓ High noise environments
  ✓ Most new digital designs
  
CMOS with TTL levels (74HCT, 74ACT):
  ✓ Interfacing with TTL
  ✓ Mixed TTL/CMOS systems
  ✓ 5V systems

ECL:
  ✓ Ultra-high speed (>500 MHz)
  ✓ Telecommunications
  ✗ Power-sensitive applications
  ✗ General purpose (too specialized)

Low-Voltage CMOS (74LVC, 74AUC):
  ✓ Modern low-voltage processors
  ✓ 3.3V, 2.5V, 1.8V systems
  ✓ Mobile devices
  ✓ High-speed with low power

BiCMOS:
  ✓ High-speed with high drive
  ✓ Large capacitive loads
  ✓ Mixed-signal designs
```

---

## 8. Interfacing Between Families

### TTL to CMOS Interface

```
TTL DRIVING CMOS:

PROBLEM: TTL VOH(min) = 2.7V, but CMOS VIH(min) = 3.5V (for 5V)
         TTL may not reach CMOS HIGH threshold!

SOLUTION 1: Pull-up Resistor

    TTL                               CMOS
    Output ────────┬─────────────────► Input
                   │
                  [Rp]  (1-10kΩ)
                   │
                  VDD

    When TTL output is HIGH, Rp pulls it closer to VDD


SOLUTION 2: Use 74HCT (TTL-compatible CMOS)

    TTL                               74HCT
    Output ─────────────────────────► Input
    
    74HCT has TTL input thresholds:
      VIH = 2.0V (matches TTL VOH)
      VIL = 0.8V
      
    No additional components needed!


SOLUTION 3: Level Shifter IC

    TTL        ┌─────────────┐         CMOS
    (5V) ──────┤   Level     ├───────► (3.3V)
               │  Shifter    │
               └─────────────┘
```

### CMOS to TTL Interface

```
CMOS DRIVING TTL:

PROBLEM: TTL inputs need significant current (IIL = 1.6mA for standard)
         CMOS outputs have limited drive current

SOLUTION 1: Use Buffer (if fan-out exceeded)

    CMOS       ┌───────────┐          TTL
    Output ────┤  Buffer   ├────────► Inputs
               │ (high     │
               │  drive)   │
               └───────────┘


SOLUTION 2: CMOS output usually OK for modern TTL

    For 74HC driving 74LS:
      IOL(HC) = 4mA, IIL(LS) = 0.4mA
      Fan-out = 4mA / 0.4mA = 10 ✓
      
    Usually works directly!


VOLTAGE LEVEL INTERFACING:

    3.3V CMOS to 5V TTL:
    
    3.3V CMOS VOH = 3.2V
    5V TTL VIH = 2.0V
    3.2V > 2.0V ✓ (HIGH level OK)
    
    3.3V CMOS VOL = 0.1V
    5V TTL VIL = 0.8V
    0.1V < 0.8V ✓ (LOW level OK)
    
    Usually works! (but verify specifications)
```

### Interfacing Summary

```
INTERFACING COMPATIBILITY MATRIX:

              │ To:                                    │
              │ TTL    CMOS   HCT    ECL    3.3V CMOS │
    ──────────┼───────────────────────────────────────┤
    From:     │                                       │
    TTL       │  OK    Pull-up OK    Special OK       │
    CMOS      │  OK*   OK     OK     Special OK*      │
    HCT       │  OK    OK     OK     Special OK*      │
    ECL       │  Xlator Xlator Xlator OK    Xlator   │
    3.3V CMOS │  OK**  OK**   OK**  Special OK       │
    ──────────┴───────────────────────────────────────┘
    
    OK = Direct connection works
    OK* = Check current drive capability
    OK** = Check voltage tolerance of 3.3V CMOS outputs
    Pull-up = Need pull-up resistor
    Special = Requires level translation
    Xlator = Requires voltage translator IC

IMPORTANT CONSIDERATIONS:
  1. Check voltage levels (VIH, VIL, VOH, VOL)
  2. Check current requirements (IIH, IIL, IOH, IOL)
  3. Check voltage tolerance of inputs
  4. Consider noise margin
  5. Consider speed requirements
```

---

## 9. Practical Considerations

### Unused Inputs

```
HANDLING UNUSED INPUTS:

NEVER leave digital inputs floating!

TTL UNUSED INPUTS:
    ┌────────────────────────────────────────────────┐
    │                                                │
    │  Option 1: Tie to VCC through resistor        │
    │                                                │
    │      VCC                                       │
    │       │                                        │
    │      [R] 1kΩ                                   │
    │       │                                        │
    │    Unused ─┤  AND gate                        │
    │    Input   │                                   │
    │                                                │
    │  Option 2: Connect to used input              │
    │                                                │
    │    Used ────┬──┤  AND gate (if fan-out OK)   │
    │    Input    └──┤                              │
    │                                                │
    └────────────────────────────────────────────────┘

CMOS UNUSED INPUTS:
    ┌────────────────────────────────────────────────┐
    │                                                │
    │  MUST tie to VDD or GND (not floating!)       │
    │                                                │
    │  Floating CMOS input:                         │
    │    • Input floats to mid-rail                 │
    │    • Both transistors partially ON            │
    │    • Draws excessive current                  │
    │    • May oscillate                            │
    │    • Can damage the IC!                       │
    │                                                │
    │  Proper connection:                           │
    │                                                │
    │    For AND/NAND: Tie unused inputs to VDD     │
    │    For OR/NOR:   Tie unused inputs to GND     │
    │                                                │
    │    VDD ───── Unused AND input                 │
    │    GND ───── Unused OR input                  │
    │                                                │
    └────────────────────────────────────────────────┘
```

### Power Supply Decoupling

```
DECOUPLING CAPACITORS:

    ┌────────────────────────────────────────────────────┐
    │                                                    │
    │  Every IC needs decoupling capacitors!            │
    │                                                    │
    │     VCC (Power supply)                            │
    │       │                                            │
    │       │    IC 1      IC 2      IC 3               │
    │       ├─────┬────────┬────────┬──────             │
    │       │     │        │        │                    │
    │      ─┴─   ─┴─      ─┴─      ─┴─                  │
    │      ─┬─   ─┬─      ─┬─      ─┬─   Decoupling    │
    │      0.1μF 0.1μF   0.1μF   0.1μF   capacitors     │
    │       │     │        │        │                    │
    │       ├─────┴────────┴────────┴──────             │
    │       │                                            │
    │      GND                                           │
    │                                                    │
    │  Place capacitor as close to IC as possible!      │
    │                                                    │
    └────────────────────────────────────────────────────┘

WHY DECOUPLING IS NEEDED:
  • ICs draw current spikes when switching
  • Current spikes cause voltage dips
  • Decoupling cap provides local charge reservoir
  • Prevents noise on power supply

TYPICAL VALUES:
  TTL: 0.01 - 0.1 μF per IC
  CMOS: 0.1 μF per IC
  High-speed: 0.01 μF ceramic + 1 μF tantalum
```

### ESD Protection

```
ELECTROSTATIC DISCHARGE (ESD) PROTECTION:

    ┌────────────────────────────────────────────────────┐
    │                                                    │
    │  CMOS is especially sensitive to ESD!             │
    │                                                    │
    │  CMOS INPUT PROTECTION (built-in):                │
    │                                                    │
    │     External ──[R]──┬──┬── Internal               │
    │     Input           │  │   Circuit                │
    │                     │  │                          │
    │                   ──┴──┴──                        │
    │                   ▼     ▼                         │
    │                   D1    D2  Protection            │
    │                   │     │   diodes                │
    │                  VDD   GND                        │
    │                                                    │
    │  [R] = input resistance (~200Ω)                   │
    │  D1, D2 = clamping diodes                         │
    │                                                    │
    │  STILL HANDLE WITH CARE:                          │
    │    • Use grounded wrist strap                     │
    │    • Use ESD-safe packaging                       │
    │    • Ground yourself before handling ICs          │
    │    • Don't touch pins directly                    │
    │                                                    │
    └────────────────────────────────────────────────────┘

ESD DAMAGE LEVELS:
  Human body model: 2000V typical discharge
  CMOS damage threshold: 1000-2000V
  Newer processes: May be <500V!
  
  Even if IC survives, may be weakened (latent damage)
```

---

## 10. Summary Tables

### Logic Family Quick Reference

| Family | VDD | Speed | Power | Noise Margin | Best For |
|--------|-----|-------|-------|--------------|----------|
| TTL LS | 5V | 10ns | 2mW | 0.4V | Legacy |
| TTL F | 5V | 3ns | 6mW | 0.4V | Speed |
| 74HC | 2-6V | 10ns | ~0 | High | General |
| 74HCT | 5V | 10ns | ~0 | 0.4V | TTL compat |
| 74AC | 2-5.5V | 3ns | ~0 | High | High speed |
| 74LVC | 1.65-3.6V | 4ns | ~0 | High | Low voltage |
| ECL | -5.2V | 1ns | 25mW | 0.2V | Ultra-high speed |

### Input/Output Specifications

| Parameter | TTL (LS) | CMOS (HC) | ECL |
|-----------|----------|-----------|-----|
| VOH (min) | 2.7V | VDD-0.1V | -0.9V |
| VOL (max) | 0.5V | 0.1V | -1.7V |
| VIH (min) | 2.0V | 0.7×VDD | -1.1V |
| VIL (max) | 0.8V | 0.3×VDD | -1.5V |
| IOH | -400μA | -4mA | -400μA |
| IOL | 8mA | 4mA | -10mA |

### IC Numbering

| Prefix | Family | Example |
|--------|--------|---------|
| 74xx | Standard TTL | 7400 |
| 74Lxx | Low-power TTL | 74L00 |
| 74Sxx | Schottky TTL | 74S00 |
| 74LSxx | Low-power Schottky | 74LS00 |
| 74ALSxx | Advanced LS | 74ALS00 |
| 74Fxx | Fast TTL | 74F00 |
| 74HCxx | High-speed CMOS | 74HC00 |
| 74HCTxx | CMOS, TTL levels | 74HCT00 |
| 74ACxx | Advanced CMOS | 74AC00 |
| 74LVCxx | Low-voltage CMOS | 74LVC00 |

---

## 11. Quick Revision Questions

### Question 1
What is noise margin and why is it important?

<details>
<summary>Click for Answer</summary>

```
NOISE MARGIN:

DEFINITION:
  The amount of noise voltage that can be tolerated
  on a digital signal without causing errors.

TWO TYPES:

1. HIGH-STATE NOISE MARGIN (NMH):
   NMH = VOH(min) - VIH(min)
   
   Maximum noise that can be added to HIGH output
   without falling below HIGH input threshold.

2. LOW-STATE NOISE MARGIN (NML):
   NML = VIL(max) - VOL(max)
   
   Maximum noise that can be added to LOW output
   without rising above LOW input threshold.

EXAMPLE (74LS):
  NMH = 2.7V - 2.0V = 0.7V
  NML = 0.8V - 0.5V = 0.3V

IMPORTANCE:
  • Determines immunity to electrical noise
  • Higher margin = more reliable operation
  • Critical in noisy environments
  • Affects maximum cable length
  • Important for industrial/automotive use

COMPARISON:
  TTL: NMH=0.7V, NML=0.3V (asymmetric)
  CMOS: NMH≈NML≈0.45×VDD (symmetric, better)
  ECL: ~0.2V (poor, but system is diff)
```
</details>

### Question 2
Why does CMOS have near-zero static power dissipation?

<details>
<summary>Click for Answer</summary>

```
CMOS NEAR-ZERO STATIC POWER:

CIRCUIT ANALYSIS:

In CMOS inverter:
  - PMOS connects output to VDD
  - NMOS connects output to GND
  - PMOS and NMOS are complementary

WHEN INPUT = LOW:
  PMOS: Gate low relative to source → ON
  NMOS: Gate low → OFF
  
  Current path: VDD → PMOS → Output
  No path to GND! (NMOS is OFF)
  
WHEN INPUT = HIGH:
  PMOS: Gate high → OFF
  NMOS: Gate high → ON
  
  Current path: Output → NMOS → GND
  No path from VDD! (PMOS is OFF)

KEY INSIGHT:
  ┌─────────────────────────────────────────┐
  │                                         │
  │  In EITHER stable state:               │
  │  One transistor is ALWAYS OFF          │
  │                                         │
  │  No DC path from VDD to GND            │
  │                                         │
  │  Only leakage current flows (~nA)      │
  │                                         │
  │  Static power ≈ 0 (ideally)            │
  │                                         │
  └─────────────────────────────────────────┘

CONTRAST WITH TTL:
  TTL has resistors that always draw current
  Even in stable state, current flows
  TTL static power: ~mW per gate

CMOS POWER COMES FROM:
  1. Leakage (very small, ~nW)
  2. Dynamic switching (P = CV²f)
  3. Short-circuit during transitions
```
</details>

### Question 3
What is fan-out and how do you calculate it?

<details>
<summary>Click for Answer</summary>

```
FAN-OUT:

DEFINITION:
  Maximum number of inputs that one output can drive
  while maintaining proper voltage levels.

CALCULATION:

For HIGH state:
  Fan-out(H) = |IOH| / IIH

For LOW state:
  Fan-out(L) = |IOL| / |IIL|

Final Fan-out = min(Fan-out(H), Fan-out(L))

EXAMPLE (74LS):
  IOH = -400 μA (source current)
  IIH = 20 μA (input leakage)
  IOL = 8 mA (sink current)
  IIL = -400 μA (input current for LOW)

  Fan-out(H) = 400μA / 20μA = 20
  Fan-out(L) = 8mA / 0.4mA = 20
  
  Fan-out = 20

CMOS FAN-OUT:
  CMOS inputs are essentially capacitive
  DC fan-out is theoretically infinite
  
  But PRACTICAL limit:
    - Capacitive loading increases delay
    - Each input adds ~5pF load
    - High fan-out = slower switching
    
  Rule of thumb: Keep fan-out < 10 for speed

EXCEEDING FAN-OUT:
  • Voltage levels degraded
  • Noise margin reduced
  • May cause logic errors
  • Use buffer to increase drive
```
</details>

### Question 4
How do you interface 3.3V CMOS with 5V TTL?

<details>
<summary>Click for Answer</summary>

```
3.3V CMOS TO 5V TTL INTERFACE:

CASE 1: 3.3V CMOS DRIVING 5V TTL

Check voltage levels:
  3.3V CMOS VOH = ~3.2V
  5V TTL VIH = 2.0V
  3.2V > 2.0V ✓ (HIGH works!)

  3.3V CMOS VOL = ~0.1V
  5V TTL VIL = 0.8V
  0.1V < 0.8V ✓ (LOW works!)

USUALLY WORKS DIRECTLY!
(but verify current requirements)

CASE 2: 5V TTL DRIVING 3.3V CMOS

PROBLEM: 5V may exceed 3.3V CMOS input rating!
         Most 3.3V CMOS is NOT 5V tolerant!

SOLUTIONS:

1. Use 5V-tolerant CMOS:
   Some LVC parts are 5V tolerant
   Check datasheet for "5V tolerant" or VIN max

2. Resistor divider:
           5V TTL        Resistor        3.3V CMOS
           Output ───[R1]───┬───────────► Input
                            │
                           [R2]
                            │
                           GND
   
   Choose R1:R2 for 3.3V max output

3. Level shifter IC:
   Use dedicated voltage translator
   (e.g., 74LVC245 with direction control)

4. Diode clamp:
           5V TTL                    3.3V CMOS
           Output ─────[R]────┬──────► Input
                              │
                              ▼ (Schottky)
                              │
                            3.3V

RECOMMENDED:
  Use level shifter IC for reliable operation
```
</details>

### Question 5
What are the advantages of ECL over TTL for high-speed applications?

<details>
<summary>Click for Answer</summary>

```
ECL ADVANTAGES FOR HIGH SPEED:

1. NON-SATURATING TRANSISTORS:
   TTL: Transistors go into saturation
        → Storage time delay to exit saturation
        → Limits switching speed
   
   ECL: Transistors never saturate
        → No storage time
        → Very fast switching (sub-ns)

2. SMALL VOLTAGE SWING:
   TTL: ~3.5V swing (0.4V to 3.4V)
   ECL: ~0.8V swing (-1.7V to -0.9V)
   
   Smaller swing = less time to charge/discharge
   → Faster transitions

3. CONSTANT CURRENT:
   ECL uses constant current sources
   → Reduced power supply noise
   → Less ground bounce
   → Better signal integrity

4. DIFFERENTIAL SIGNALING:
   ECL provides complementary outputs (OR/NOR)
   → Can use differential signaling
   → Better common-mode noise rejection

5. CONTROLLED EDGE RATES:
   ECL designed for controlled transitions
   → Reduced ringing and reflections
   → Better transmission line behavior

ECL SPEED:
  Propagation delay: 0.5-2 ns
  Toggle frequency: >1 GHz possible

TRADE-OFF:
  Speed comes at cost of power (~25mW/gate)
  Used only where speed is critical
```
</details>

### Question 6
What precautions should be taken when handling CMOS ICs?

<details>
<summary>Click for Answer</summary>

```
CMOS HANDLING PRECAUTIONS:

1. ESD PROTECTION:
   ┌─────────────────────────────────────────┐
   │ • Use grounded wrist strap              │
   │ • Work on ESD-safe mat                  │
   │ • Use anti-static bags for storage      │
   │ • Ground yourself before handling       │
   │ • Avoid synthetic clothing (static)     │
   │ • Handle ICs by edges, not pins         │
   │ • Keep humidity >40% if possible        │
   └─────────────────────────────────────────┘

2. UNUSED INPUTS:
   ┌─────────────────────────────────────────┐
   │ • NEVER leave inputs floating           │
   │ • Tie unused inputs to VDD or GND       │
   │ • Floating input → high current → damage│
   │ • AND/NAND unused: tie to VDD           │
   │ • OR/NOR unused: tie to GND             │
   └─────────────────────────────────────────┘

3. POWER SUPPLY:
   ┌─────────────────────────────────────────┐
   │ • Add decoupling capacitors (0.1μF)     │
   │ • Place cap close to IC                 │
   │ • Connect VDD before applying signals   │
   │ • Never exceed maximum VDD              │
   └─────────────────────────────────────────┘

4. INPUT VOLTAGE:
   ┌─────────────────────────────────────────┐
   │ • Never exceed VIN > VDD + 0.3V         │
   │ • Never apply VIN < VSS - 0.3V          │
   │ • Use protection diodes if needed       │
   │ • Check 5V tolerance for 3.3V parts     │
   └─────────────────────────────────────────┘

5. LATCH-UP PREVENTION:
   ┌─────────────────────────────────────────┐
   │ • Apply power before signals            │
   │ • Don't exceed absolute max ratings     │
   │ • Use input protection on external pins │
   │ • Limit current with series resistors   │
   └─────────────────────────────────────────┘
```
</details>

---

## Navigation

| Previous | [Home](README.md) | Next |
|:--------:|:-----------------:|:----:|
| [Unit 9: Memory](Unit-09-Memory-and-Programmable-Logic.md) | Return to Index | End of Course |

---

## Congratulations!

You have completed the Digital Electronics study notes. Review all units for a comprehensive understanding of:

1. **Number Systems and Codes** - Binary, octal, hex, BCD, Gray code
2. **Boolean Algebra and Logic Gates** - Fundamental operations and theorems
3. **Minimization Techniques** - K-maps, Quine-McCluskey
4. **Combinational Circuits** - Adders, MUX, decoders, ALU
5. **Flip-Flops** - Latches, edge-triggered, conversions
6. **Counters** - Asynchronous, synchronous, Mod-N
7. **Registers** - Shift registers, universal register
8. **Finite State Machines** - Moore, Mealy, design procedure
9. **Memory and PLDs** - ROM, RAM, PLA, PAL, FPGA
10. **Logic Families** - TTL, CMOS, ECL, interfacing

---

*End of Unit 10 and Course*
