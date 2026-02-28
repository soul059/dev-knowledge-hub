# Chapter 2.2: Amplitude Scaling

## Overview

Amplitude scaling modifies the **dependent variable** (amplitude/magnitude) of a signal while keeping the independent variable (time) unchanged. This operation models gain, attenuation, inversion, and dynamic range adjustment in physical systems.

---

## 2.2.1 Definition

$$y(t) = A \cdot x(t)$$

where $A$ is a real (or complex) constant called the **scaling factor** or **gain**.

| Condition | Effect | Physical Meaning |
|-----------|--------|-----------------|
| $A > 1$ | **Amplification** | Signal made larger |
| $0 < A < 1$ | **Attenuation** | Signal made smaller |
| $A = -1$ | **Inversion** | Signal flipped about time axis |
| $A < -1$ | **Inversion + Amplification** | Flip and enlarge |
| $-1 < A < 0$ | **Inversion + Attenuation** | Flip and shrink |
| $A = 0$ | **Nulling** | Output is zero |

---

## 2.2.2 Graphical Illustration

```
x(t):                  2·x(t):               0.5·x(t):
    │                      │                      │
  2 ┤    ╭╮             4 ┤    ╭╮              1 ┤    ╭╮
    │   ╱  ╲               │   ╱  ╲               │   ╱  ╲
  1 ┤  ╱    ╲           2 ┤  ╱    ╲           0.5┤  ╱    ╲
    │ ╱      ╲             │ ╱      ╲             │ ╱      ╲
  0 ┤╱────────╲→ t      0 ┤╱────────╲→ t      0 ┤╱────────╲→ t
    │                      │                      │
  Amplified by 2:          Each value doubled      Each value halved
  peak 2 → 4              Shape unchanged         Shape unchanged


-x(t):                 -2·x(t):
    │                      │
  0 ┤╲────────╱→ t      0 ┤╲────────╱→ t
    │ ╲      ╱             │ ╲      ╱
 -1 ┤  ╲    ╱          -2 ┤  ╲    ╱
    │   ╲  ╱               │   ╲  ╱
 -2 ┤    ╰╯            -4 ┤    ╰╯
    │                      │
  Inverted                 Inverted + Amplified
```

---

## 2.2.3 Effect on Signal Properties

| Property | $x(t)$ | $A \cdot x(t)$ |
|----------|--------|--------------|
| **Amplitude range** | $[x_{min}, x_{max}]$ | $[Ax_{min}, Ax_{max}]$ if $A > 0$ |
| **Energy** | $E_x$ | $\|A\|^2 E_x$ |
| **Power** | $P_x$ | $\|A\|^2 P_x$ |
| **Peak value** | $x_{peak}$ | $\|A\| \cdot x_{peak}$ |
| **Zero crossings** | At $t_k$ | Same $t_k$ (unchanged) |
| **Frequency content** | $X(\omega)$ | $A \cdot X(\omega)$ (scaled spectrum) |
| **Shape** | Original | Identical (only magnitude changes) |

### Energy Scaling Proof

$$E_y = \int_{-\infty}^{\infty} |Ax(t)|^2 \, dt = |A|^2 \int_{-\infty}^{\infty} |x(t)|^2 \, dt = |A|^2 E_x$$

---

## 2.2.4 Amplitude Scaling in Circuit Systems

### Inverting Amplifier (Op-Amp)

```
             Rf
         ┌──/\/\/──────────┐
         │                 │
         │    ┌────────┐   │
    R₁   │  ─ │        │   │
  ──/\/\/─┴──►│  Op-Amp ├───┴──── Vout
  Vin        +│        │
          ────│        │
              └────────┘
              
    Gain: A = -Rf/R₁
    
    Vout = -(Rf/R₁) · Vin
```

| $R_f$ | $R_1$ | Gain $A$ | Type |
|-------|-------|----------|------|
| 10kΩ | 1kΩ | $-10$ | Inverting amplifier |
| 1kΩ | 1kΩ | $-1$ | Inverter (unity) |
| 1kΩ | 10kΩ | $-0.1$ | Inverting attenuator |

### Non-Inverting Amplifier

```
              Rf
          ┌──/\/\/──────────┐
          │                 │
          │    ┌────────┐   │
     R₁   │  ─ │        │   │
  ───/\/\/─┴──►│  Op-Amp ├───┴──── Vout
                +│        │
    Vin ────────│        │
                └────────┘
    
    Gain: A = 1 + Rf/R₁ > 1
    Always positive gain (non-inverting)
```

### Voltage Divider (Passive Attenuation)

```
         R₁
    ──/\/\/──┬──── Vout
             │
    Vin      R₂
             │
    ─────────┴──── GND
    
    Vout = (R₂/(R₁ + R₂)) · Vin
    
    Gain A = R₂/(R₁ + R₂) < 1 (always attenuation)
```

---

## 2.2.5 Dynamic Range and Decibel Scale

In practice, amplitude scaling is often expressed in **decibels (dB)**:

$$A_{dB} = 20 \log_{10}|A| \text{ (for voltage/amplitude)}$$

$$A_{dB} = 10 \log_{10}|A|^2 \text{ (for power)}$$

| Linear Gain $A$ | dB Value | Meaning |
|-----------------|----------|---------|
| 1 | 0 dB | Unity gain |
| 2 | 6.02 dB | Double amplitude |
| 10 | 20 dB | 10× amplitude |
| 100 | 40 dB | 100× amplitude |
| 0.5 | −6.02 dB | Half amplitude |
| 0.1 | −20 dB | 10% of amplitude |
| 0.01 | −40 dB | 1% of amplitude |

> **Quick rule**: Every factor of 2 ≈ 6 dB; every factor of 10 = 20 dB.

---

## 2.2.6 Amplitude Clipping and Non-Linear Scaling

In practice, amplifiers have **saturation limits**: output cannot exceed $\pm V_{sat}$.

$$y(t) = \begin{cases} V_{sat}, & Ax(t) > V_{sat} \\ Ax(t), & |Ax(t)| \leq V_{sat} \\ -V_{sat}, & Ax(t) < -V_{sat} \end{cases}$$

```
Linear (ideal):              Clipped (saturated):
    │                            │
    │    ╱╲                  Vsat┤ ───╱╲───
    │   ╱  ╲                     │   ╱  ╲
  0 ┤──╱────╲──→ t           0 ┤──╱────╲──→ t
    │ ╱      ╲                   │ ╱      ╲
    │╱        ╲           -Vsat ┤╱────────╲───
    │                            │
    No distortion               Flat-top distortion (harmonics)
```

> **Key insight**: Clipping is a **non-linear** amplitude operation and introduces **harmonic distortion**.

---

## 2.2.7 Applications of Amplitude Scaling

| Application | Scaling Type | System |
|-------------|-------------|--------|
| Audio volume control | Variable gain | Potentiometer / digital gain |
| Signal conditioning | Amplification | Instrumentation amplifier |
| Impedance matching | Attenuation | Resistive pad / transformer |
| Signal inversion | $A = -1$ | Inverting buffer |
| AGC (Automatic Gain Control) | Variable | Receiver front-end |
| Radar/Sonar | Time-varying gain | STC (Sensitivity Time Control) |

---

## Summary Table

| Concept | Formula | Key Effect |
|---------|---------|------------|
| Amplification | $y = Ax$, $A > 1$ | Increases magnitude |
| Attenuation | $y = Ax$, $0 < A < 1$ | Decreases magnitude |
| Inversion | $y = -x$ | Flips about time axis |
| Energy change | $E_y = \|A\|^2 E_x$ | Quadratic scaling |
| Power change | $P_y = \|A\|^2 P_x$ | Quadratic scaling |
| dB scale | $A_{dB} = 20\log_{10}\|A\|$ | Logarithmic representation |
| Clipping | $\|y\| \leq V_{sat}$ | Non-linear distortion |
| Zero crossings | Unchanged | Only magnitude changes |

---

## Quick Revision Questions

1. **If $x(t)$ has energy $E = 4$, what is the energy of $y(t) = 3x(t)$?**
   - $E_y = 9 \times 4 = 36$.

2. **What gain in dB does a factor of 100 represent for voltage signals?**
   - $20\log_{10}(100) = 40$ dB.

3. **An inverting amplifier has $R_f = 47\text{k}\Omega$ and $R_1 = 4.7\text{k}\Omega$. What is the gain?**
   - $A = -R_f/R_1 = -10$.

4. **Does amplitude scaling change the zero-crossing times of a signal?**
   - No. Zero crossings remain unchanged.

5. **What is the amplitude of $y(t) = -0.5 \sin(t)$?**
   - Amplitude = $|-0.5| = 0.5$; the signal is inverted and attenuated.

6. **Why does clipping introduce harmonic distortion?**
   - Clipping changes the waveform shape (flat tops), which adds frequency components (harmonics) not present in the original signal.

---

[← Previous: Time Shifting and Scaling](01-time-shifting-and-scaling.md) | [Next: Signal Addition and Multiplication →](03-signal-addition-and-multiplication.md)
