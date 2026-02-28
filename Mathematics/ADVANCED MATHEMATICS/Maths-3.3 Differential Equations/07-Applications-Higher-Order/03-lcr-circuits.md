# Chapter 7.3: LCR Circuits

[← Previous: Damped and Forced Oscillations](02-damped-and-forced-oscillations.md) | [Back to README](../README.md) | [Next: Beam Deflection →](04-beam-deflection.md)

---

## Overview

Series LCR circuits are the electrical analog of the damped spring-mass system. The governing second-order ODE exhibits identical mathematical behavior — underdamped oscillations, critical damping, resonance — making all mechanical results directly transferable.

---

## 1. The Series LCR Circuit

```
        R (Ω)        L (H)        C (F)
  ┌───/\/\/──┬───⌇⌇⌇⌇──┬────┤├────┐
  │          │          │         │
  ⊕ E(t)    │          │         │
  │          │    (i →) │         │
  └──────────┴──────────┴─────────┘
```

### KVL Around the Loop

$$L\frac{di}{dt} + Ri + \frac{q}{C} = E(t)$$

Since $i = dq/dt$:

$$\boxed{L\frac{d^2q}{dt^2} + R\frac{dq}{dt} + \frac{q}{C} = E(t)}$$

Or in terms of current (differentiate):

$$L\frac{d^2i}{dt^2} + R\frac{di}{dt} + \frac{i}{C} = E'(t)$$

---

## 2. Complete Analogy Table

| Mechanical | Electrical | Role |
|-----------|-----------|------|
| Mass $m$ | Inductance $L$ | Inertia |
| Damping $c$ | Resistance $R$ | Dissipation |
| Spring constant $k$ | $1/C$ | Restoring |
| Displacement $x$ | Charge $q$ | Configuration |
| Velocity $v$ | Current $i$ | Flow |
| External force $F(t)$ | EMF $E(t)$ | Driving |
| $mx'' + cx' + kx = F$ | $Lq'' + Rq' + q/C = E$ | Governing equation |

---

## 3. Free Oscillations ($E = 0$)

$$Lq'' + Rq' + \frac{q}{C} = 0$$

Characteristic equation: $Lm^2 + Rm + 1/C = 0$

$$m = \frac{-R \pm \sqrt{R^2 - 4L/C}}{2L}$$

### Three Cases

| Case | Condition | Circuit Behavior |
|------|-----------|-----------------|
| **Overdamped** | $R^2 > 4L/C$ | Charge decays without oscillation |
| **Critically damped** | $R^2 = 4L/C$ → $R = 2\sqrt{L/C}$ | Fastest decay, no oscillation |
| **Underdamped** | $R^2 < 4L/C$ | Damped oscillation of charge/current |

### Underdamped Solution

$$q(t) = Qe^{-\alpha t}\cos(\omega_d t - \phi)$$

where:
- $\alpha = R/(2L)$ (damping rate)
- $\omega_0 = 1/\sqrt{LC}$ (natural frequency)
- $\omega_d = \sqrt{\omega_0^2 - \alpha^2}$ (damped frequency)

```
  i(t)    UNDERDAMPED LCR (free oscillation)
   │
   │   ╱╲
   │  ╱  ╲    ╱╲
   │ ╱    ╲  ╱  ╲     ╱╲
───┼╱──────╲╱────╲───╱──╲─── t
   │        ╲     ╲ ╱    ╲__
   │         ╲     ╲╱
   │
   │  Oscillation decays due to R
   │  Energy dissipated as heat in R
```

---

## 4. Forced Oscillations ($E = E_0\sin\omega t$)

$$Lq'' + Rq' + \frac{q}{C} = E_0\sin\omega t$$

### Steady-State Current

$$\boxed{i_{ss}(t) = \frac{E_0}{Z}\sin(\omega t - \phi)}$$

where:

### Impedance

$$\boxed{Z = \sqrt{R^2 + \left(\omega L - \frac{1}{\omega C}\right)^2}}$$

### Phase Angle

$$\boxed{\tan\phi = \frac{\omega L - 1/(\omega C)}{R} = \frac{X_L - X_C}{R}}$$

### Reactances

| Quantity | Symbol | Formula |
|----------|--------|---------|
| Inductive reactance | $X_L$ | $\omega L$ |
| Capacitive reactance | $X_C$ | $1/(\omega C)$ |
| Net reactance | $X$ | $X_L - X_C$ |
| Impedance | $Z$ | $\sqrt{R^2 + X^2}$ |

```
  IMPEDANCE TRIANGLE

       Z ╱│
        ╱  │
       ╱   │  X = ωL - 1/(ωC)
      ╱    │  (reactance)
     ╱  φ  │
    ╱──────┘
       R
    (resistance)
```

---

## 5. Electrical Resonance

### Resonance Condition

Maximum current amplitude when impedance is minimum: $X_L = X_C$

$$\omega L = \frac{1}{\omega C} \implies \boxed{\omega_r = \frac{1}{\sqrt{LC}}}$$

At resonance:
- $Z = R$ (minimum impedance)
- $i_{max} = E_0/R$ (maximum current)
- $\phi = 0$ (current in phase with voltage)

```
  |i|    RESONANCE CURVE
   │
E₀/R├── ─ ─╲─ ─ ─    Low R (sharp peak)
   │      ╱ ╲
   │     ╱   ╲
   │    ╱     ╲        ╱╲ High R (broad)
   │   ╱       ╲      ╱  ╲
   │  ╱         ╲    ╱    ╲
   │ ╱           ╲__╱      ╲___
   │╱                          ╲___
   ├──────────┼────────────────── ω
              ω_r = 1/√(LC)
```

### Quality Factor

$$Q = \frac{\omega_r L}{R} = \frac{1}{R}\sqrt{\frac{L}{C}}$$

Higher $Q$ → sharper resonance peak → better frequency selectivity (used in radio tuning).

### Bandwidth

$$\Delta\omega = \frac{\omega_r}{Q} = \frac{R}{L}$$

---

## 6. Worked Examples

### Example 1: Free Oscillation

*$L = 0.5$ H, $R = 10\,\Omega$, $C = 200\,\mu F$. Initial charge $q_0 = 0.01$ C, $i(0) = 0$.*

$\alpha = R/(2L) = 10$

$\omega_0 = 1/\sqrt{0.5 \times 2\times10^{-4}} = 1/\sqrt{10^{-4}} = 100$ rad/s

$R^2 = 100$, $4L/C = 4(0.5)/(2\times10^{-4}) = 10000$

$R^2 < 4L/C$ → **Underdamped**

$\omega_d = \sqrt{10000 - 100} = \sqrt{9900} \approx 99.5$ rad/s

$q(t) = e^{-10t}(0.01\cos 99.5t + c_2\sin 99.5t)$

$i(0) = 0$: $q'(0) = -10(0.01) + 99.5c_2 = 0 \implies c_2 \approx 0.001$

$$q(t) \approx e^{-10t}(0.01\cos 99.5t + 0.001\sin 99.5t)$$

### Example 2: Forced LCR at Resonance

*$L = 1$ H, $R = 100\,\Omega$, $C = 1\,\mu F$, $E = 10\sin\omega t$.*

$\omega_r = 1/\sqrt{10^{-6}} = 1000$ rad/s

At resonance: $Z = R = 100\,\Omega$

$i_{max} = E_0/R = 10/100 = 0.1$ A

$Q = \omega_r L/R = 1000/100 = 10$

Bandwidth: $\Delta\omega = 1000/10 = 100$ rad/s

### Example 3: Impedance Calculation

*$R = 300\,\Omega$, $L = 0.5$ H, $C = 20\,\mu F$, $\omega = 200$ rad/s*

$X_L = 200 \times 0.5 = 100\,\Omega$

$X_C = 1/(200 \times 20\times10^{-6}) = 250\,\Omega$

$X = 100 - 250 = -150\,\Omega$ (capacitive)

$Z = \sqrt{300^2 + 150^2} = \sqrt{112500} \approx 335.4\,\Omega$

$\phi = \arctan(-150/300) = -26.6°$ (current leads voltage)

### Example 4: Finding Resonance Frequency

*Design an LCR circuit to resonate at $f = 1$ MHz. If $L = 10\,\mu H$, find $C$.*

$\omega_r = 2\pi \times 10^6$ rad/s

$C = \dfrac{1}{\omega_r^2 L} = \dfrac{1}{(2\pi\times10^6)^2 \times 10^{-5}} \approx 2.53$ nF

---

## 7. Power in LCR Circuits

### Average Power

$$\boxed{P_{avg} = \frac{E_0^2 R}{2Z^2} = \frac{E_0^2}{2R} \text{ (at resonance)}}$$

At resonance, all power is dissipated in $R$; none is stored on average in $L$ or $C$.

---

## Summary Table

| Concept | Details |
|---------|---------|
| **LCR ODE** | $Lq'' + Rq' + q/C = E(t)$ |
| **Natural frequency** | $\omega_0 = 1/\sqrt{LC}$ |
| **Damping rate** | $\alpha = R/(2L)$ |
| **Impedance** | $Z = \sqrt{R^2 + (\omega L - 1/\omega C)^2}$ |
| **Resonance** | $\omega_r = 1/\sqrt{LC}$, $Z_{min} = R$ |
| **Q-factor** | $Q = \omega_r L/R$ |
| **Bandwidth** | $\Delta\omega = R/L$ |
| **At resonance** | $i = E_0/R$, $\phi = 0$ |

---

## Quick Revision Questions

1. An LCR circuit has $L = 2$ H, $R = 40\,\Omega$, $C = 0.01$ F. Is it overdamped, critically damped, or underdamped?

2. Find the resonance frequency and Q-factor for $L = 50$ mH, $R = 10\,\Omega$, $C = 0.2\,\mu F$.

3. At what frequency is the impedance of a series LCR circuit equal to $R$?

4. Show that $Q = \omega_r L/R = 1/(R\omega_r C)$.

5. If $Q = 50$ and $f_r = 100$ kHz, what is the bandwidth in Hz?

6. Derive the steady-state current amplitude $E_0/Z$ using the D-operator or undetermined coefficients.

---

[← Previous: Damped and Forced Oscillations](02-damped-and-forced-oscillations.md) | [Back to README](../README.md) | [Next: Beam Deflection →](04-beam-deflection.md)
