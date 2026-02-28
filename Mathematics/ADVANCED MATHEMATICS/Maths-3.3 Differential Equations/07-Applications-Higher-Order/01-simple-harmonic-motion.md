# Chapter 7.1: Simple Harmonic Motion (SHM)

[← Previous: Cauchy-Euler Non-Homogeneous](../06-Non-Homogeneous-Linear-ODEs/04-cauchy-euler-non-homogeneous.md) | [Back to README](../README.md) | [Next: Damped and Forced Oscillations →](02-damped-and-forced-oscillations.md)

---

## Overview

Simple Harmonic Motion is the most fundamental oscillatory motion in physics, governed by a second-order linear ODE with constant coefficients. It models mass-spring systems, pendulums (small angle), LC circuits, and many other periodic phenomena.

---

## 1. The Spring-Mass System

```
     Fixed Wall
     ║
     ║   Spring (k)      Mass (m)
     ║╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍┃████████┃──→ x
     ║  ←── natural ──→  ┃████████┃
     ║      length       ┃████████┃
     ║                   
     ║════════════════════════════════════ surface
```

### Hooke's Law

Restoring force: $F = -kx$ (proportional to displacement, opposite direction)

### Newton's Second Law

$$m\frac{d^2x}{dt^2} = -kx$$

$$\boxed{\frac{d^2x}{dt^2} + \omega_0^2 x = 0, \qquad \omega_0 = \sqrt{\frac{k}{m}}}$$

where $\omega_0$ is the **natural (angular) frequency**.

---

## 2. Solution

Characteristic equation: $m^2 + \omega_0^2 = 0 \implies m = \pm i\omega_0$

$$\boxed{x(t) = c_1\cos\omega_0 t + c_2\sin\omega_0 t}$$

### Equivalent Forms

| Form | Expression | Parameters |
|------|-----------|-----------|
| Rectangular | $c_1\cos\omega_0 t + c_2\sin\omega_0 t$ | $c_1, c_2$ from ICs |
| Amplitude-Phase | $A\cos(\omega_0 t - \phi)$ | $A = \sqrt{c_1^2 + c_2^2}$, $\tan\phi = c_2/c_1$ |
| Amplitude-Phase | $A\sin(\omega_0 t + \psi)$ | $A = \sqrt{c_1^2 + c_2^2}$, $\tan\psi = c_1/c_2$ |
| Complex | $\text{Re}(Ae^{i(\omega_0 t - \phi)})$ | Complex amplitude |

---

## 3. Key Quantities

```
  x(t)
  │
A ├── ─ ─ ─ ─╲─ ─ ─ ─ ─ ─╲─ ─ ─ ─    amplitude A
  │         ╱   ╲        ╱   ╲
  │       ╱       ╲    ╱       ╲
  │     ╱           ╲╱           ╲
──┼───╱─────────────────────────────╲── t
  │ ╱               ╱╲               
  │╱             ╱      ╲
  │           ╱           ╲
-A├── ─ ─ ╱─ ─ ─ ─ ─ ─ ─ ╲─ ─ ─ ─
  │
  │←─────── T ──────────→│
  │     (period)
```

| Quantity | Symbol | Formula |
|----------|--------|---------|
| Angular frequency | $\omega_0$ | $\sqrt{k/m}$ |
| Period | $T$ | $2\pi/\omega_0 = 2\pi\sqrt{m/k}$ |
| Frequency | $f$ | $1/T = \omega_0/(2\pi)$ |
| Amplitude | $A$ | $\sqrt{c_1^2 + c_2^2}$ |
| Phase angle | $\phi$ | $\arctan(c_2/c_1)$ |

---

## 4. Worked Examples

### Example 1: Basic IVP

*A 2 kg mass on a spring ($k = 8$ N/m) is displaced 3 cm and released from rest. Find $x(t)$.*

$\omega_0 = \sqrt{8/2} = 2$ rad/s

$x'' + 4x = 0 \implies x = c_1\cos 2t + c_2\sin 2t$

ICs: $x(0) = 0.03$ m, $x'(0) = 0$

$c_1 = 0.03$, $2c_2 = 0 \implies c_2 = 0$

$$x(t) = 0.03\cos 2t \text{ meters}$$

Period: $T = 2\pi/2 = \pi \approx 3.14$ s

### Example 2: Initial Velocity

*Same spring-mass. Initially at equilibrium, given velocity 0.1 m/s.*

$x(0) = 0$: $c_1 = 0$

$x'(0) = 0.1$: $2c_2 = 0.1 \implies c_2 = 0.05$

$$x(t) = 0.05\sin 2t$$

Amplitude: $A = 0.05$ m = 5 cm

### Example 3: Both Initial Conditions

*$m = 1$ kg, $k = 25$ N/m, $x(0) = 0.04$ m, $x'(0) = -0.3$ m/s*

$\omega_0 = 5$

$x = 0.04\cos 5t + c_2\sin 5t$

$x' = -0.2\sin 5t + 5c_2\cos 5t$

$x'(0) = 5c_2 = -0.3 \implies c_2 = -0.06$

$$x = 0.04\cos 5t - 0.06\sin 5t$$

Amplitude: $A = \sqrt{0.04^2 + 0.06^2} = \sqrt{0.0052} \approx 0.072$ m

Phase: $\phi = \arctan(-0.06/0.04) = \arctan(-1.5) \approx -56.3°$

$$x = 0.072\cos(5t + 56.3°)$$

---

## 5. Energy in SHM

| Energy Type | Formula | At $x = A$ | At $x = 0$ |
|-------------|---------|-----------|-----------|
| Kinetic | $\frac{1}{2}mv^2$ | 0 | Maximum |
| Potential | $\frac{1}{2}kx^2$ | Maximum | 0 |
| Total | $E = \frac{1}{2}kA^2$ | $E$ | $E$ |

$$\boxed{E = \frac{1}{2}kA^2 = \frac{1}{2}m\omega_0^2 A^2 = \text{constant}}$$

```
  Energy
    │
  E ├─────────────────────────────── Total E
    │╲    ╱╲    ╱╲    ╱╲    ╱╲
    │  ╲╱    ╲╱    ╲╱    ╲╱    ╲╱   PE (potential)
    │  ╱╲    ╱╲    ╱╲    ╱╲    ╱╲
    │╱    ╲╱    ╲╱    ╲╱    ╲╱      KE (kinetic)
    ├──┬──┬──┬──┬──┬──┬──┬──┬── t
       T/4  T/2  3T/4  T
```

---

## 6. Simple Pendulum (Small Angle)

```
        ╱╱╱╱╱╱╱ (pivot)
          │
          │ L (length)
          │
          │  θ
          │╱
          ●  mass m
         ╱
        ╱ mg sin θ
```

For small angles ($\sin\theta \approx \theta$):

$$\boxed{\frac{d^2\theta}{dt^2} + \frac{g}{L}\theta = 0}$$

$$\omega_0 = \sqrt{g/L}, \quad T = 2\pi\sqrt{L/g}$$

> Note: Period is independent of mass and amplitude (for small angles)!

---

## 7. LC Circuit Analog

```
  ┌───⌇⌇⌇⌇───┐
  │    L      │
  │           │
  │   ┤├      │
  │    C      │
  └───────────┘
```

$L\dfrac{d^2q}{dt^2} + \dfrac{q}{C} = 0$

$$\omega_0 = \frac{1}{\sqrt{LC}}, \quad T = 2\pi\sqrt{LC}$$

### Mechanical–Electrical Analogy

| Mechanical | Electrical |
|-----------|-----------|
| Displacement $x$ | Charge $q$ |
| Velocity $v$ | Current $i$ |
| Mass $m$ | Inductance $L$ |
| Spring constant $k$ | $1/C$ |
| KE: $\frac{1}{2}mv^2$ | Magnetic: $\frac{1}{2}Li^2$ |
| PE: $\frac{1}{2}kx^2$ | Electric: $\frac{q^2}{2C}$ |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **ODE** | $x'' + \omega_0^2 x = 0$ |
| **Solution** | $x = A\cos(\omega_0 t - \phi)$ |
| **$\omega_0$** | $\sqrt{k/m}$ |
| **Period** | $T = 2\pi/\omega_0$ |
| **Energy** | $E = \frac{1}{2}kA^2$ (conserved) |
| **Pendulum** | $T = 2\pi\sqrt{L/g}$ |
| **LC circuit** | $T = 2\pi\sqrt{LC}$ |

---

## Quick Revision Questions

1. A 0.5 kg mass on a spring ($k = 50$ N/m) is released from $x = 0.1$ m. Find the period and maximum velocity.

2. Show that the total energy $\frac{1}{2}mv^2 + \frac{1}{2}kx^2$ is constant for SHM.

3. A pendulum has period 2 seconds on Earth ($g = 9.8$ m/s²). Find its length.

4. An LC circuit has $L = 0.1$ H and $C = 100\,\mu F$. Find the oscillation frequency.

5. If the amplitude of SHM doubles, by what factor does the total energy change?

6. Write the equation of motion for a vertical spring-mass system (include gravity) and show it still gives SHM about a shifted equilibrium.

---

[← Previous: Cauchy-Euler Non-Homogeneous](../06-Non-Homogeneous-Linear-ODEs/04-cauchy-euler-non-homogeneous.md) | [Back to README](../README.md) | [Next: Damped and Forced Oscillations →](02-damped-and-forced-oscillations.md)
