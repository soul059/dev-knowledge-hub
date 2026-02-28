# Chapter 7.2: Damped and Forced Oscillations

[← Previous: Simple Harmonic Motion](01-simple-harmonic-motion.md) | [Back to README](../README.md) | [Next: LCR Circuits →](03-lcr-circuits.md)

---

## Overview

Real oscillatory systems experience **damping** (friction, air resistance) and may be driven by an external **forcing function**. This leads to second-order non-homogeneous ODEs whose solutions exhibit transient and steady-state behavior, including the critical phenomenon of **resonance**.

---

## 1. Damped Free Oscillations

### The Model

$$\boxed{m\frac{d^2x}{dt^2} + c\frac{dx}{dt} + kx = 0}$$

```
     ║                           
     ║   Spring     Dashpot   Mass
     ║╍╍╍╍╍╍╍╍╍┃═══╤═══┃┃████████┃──→ x
     ║         ┃   │   ┃┃████████┃
     ║╍╍╍╍╍╍╍╍╍┃═══╧═══┃┃████████┃
     ║    k          c       m
     ║════════════════════════════════ surface
```

### Characteristic Equation

$$mm^2 + cm + k = 0 \implies m_{1,2} = \frac{-c \pm \sqrt{c^2 - 4mk}}{2m}$$

Define: $\gamma = \dfrac{c}{2m}$ (damping coefficient), $\omega_0 = \sqrt{k/m}$ (natural frequency)

$$m_{1,2} = -\gamma \pm \sqrt{\gamma^2 - \omega_0^2}$$

### Three Cases

```
           Discriminant: γ² - ω₀²
                │
     ┌──────────┼──────────┐
     │          │          │
  γ > ω₀    γ = ω₀     γ < ω₀
     │          │          │
OVERDAMPED  CRITICALLY  UNDERDAMPED
            DAMPED
     │          │          │
  Two real   Repeated   Complex
  negative   negative   conjugate
  roots      root       roots
```

---

## 2. Case Analysis

### Case 1: Overdamped ($c^2 > 4mk$ or $\gamma > \omega_0$)

$$x(t) = c_1 e^{(-\gamma + \sqrt{\gamma^2 - \omega_0^2})t} + c_2 e^{(-\gamma - \sqrt{\gamma^2 - \omega_0^2})t}$$

Both exponents are **negative** → no oscillation, slow return to equilibrium.

### Case 2: Critically Damped ($c^2 = 4mk$ or $\gamma = \omega_0$)

$$x(t) = (c_1 + c_2 t)e^{-\gamma t}$$

Fastest return without oscillation. Used in door closers, car suspensions.

### Case 3: Underdamped ($c^2 < 4mk$ or $\gamma < \omega_0$)

$$\boxed{x(t) = Ae^{-\gamma t}\cos(\omega_d t - \phi)}$$

where $\omega_d = \sqrt{\omega_0^2 - \gamma^2}$ is the **damped frequency**.

```
  x(t)    UNDERDAMPED OSCILLATION
   │
   │    Ae^(-γt) envelope
   │   ╱─ ─ ─╲─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
   │  ╱╲      ╲
   │ ╱   ╲     ╲___
   │╱     ╲       ╲__╲_____
───┼───────╲───────╲────────╲──────── t
   │        ╲       ╲___  __╱
   │         ╲          ╲╱
   │          ╲
   │           ╲─ ─ ─ ─ ─ ─ ─ ─ ─ ─
   │         -Ae^(-γt) envelope
```

### Comparison

| Case | Condition | Behavior | Oscillates? |
|------|-----------|----------|:-----------:|
| Overdamped | $c^2 > 4mk$ | Slow exponential decay | No |
| Critically damped | $c^2 = 4mk$ | Fastest non-oscillatory decay | No |
| Underdamped | $c^2 < 4mk$ | Decaying oscillations | Yes |

---

## 3. Forced Oscillations

### The Model

$$\boxed{mx'' + cx' + kx = F_0\cos\omega t}$$

External force: $F(t) = F_0\cos\omega t$ with driving frequency $\omega$.

### General Solution

$$x(t) = \underbrace{x_c(t)}_{\text{transient}} + \underbrace{x_p(t)}_{\text{steady-state}}$$

The transient $x_c$ dies out (contains $e^{-\gamma t}$). The steady-state persists:

$$\boxed{x_p(t) = \frac{F_0/m}{\sqrt{(\omega_0^2 - \omega^2)^2 + 4\gamma^2\omega^2}}\cos(\omega t - \delta)}$$

where $\delta = \arctan\dfrac{2\gamma\omega}{\omega_0^2 - \omega^2}$

### Amplitude of Steady-State Response

$$\boxed{A(\omega) = \frac{F_0/m}{\sqrt{(\omega_0^2 - \omega^2)^2 + 4\gamma^2\omega^2}}}$$

---

## 4. Resonance

### Resonance Frequency

Maximum amplitude occurs at:

$$\omega_r = \sqrt{\omega_0^2 - 2\gamma^2}$$

(Exists only if $\omega_0 > \sqrt{2}\gamma$)

### Undamped Resonance ($c = 0$, $\omega = \omega_0$)

When there is no damping and $\omega = \omega_0$:

$$x_p = \frac{F_0}{2m\omega_0}t\sin\omega_0 t$$

Amplitude **grows linearly with time** ($t\sin\omega_0 t$)!

```
  x(t)    RESONANCE (no damping, ω = ω₀)
   │
   │              ╱╲╱╲              Amplitude grows
   │           ╱╲╱    ╲╱╲           linearly: ~ t
   │        ╱╲╱          ╲╱╲
   │     ╱╲╱                ╲╱╲
   │  ╱╲╱                      ╲╱╲
───┼╱╱────────────────────────────╲╲── t
   │╱╲                          ╱╲╱
   │   ╲╱╲                  ╱╲╱
   │      ╲╱╲            ╱╲╱
   │         ╲╱╲      ╱╲╱
   │            ╲╱╲╱╲╱
```

### Amplitude vs. Driving Frequency

```
  A(ω)
   │           Smaller damping
   │            │╲  (sharper peak)
   │            │  ╲
   │           ╱│   ╲
   │          ╱ │    ╲╲
   │    ╱╲╲  ╱  │     ╲ ╲   Larger damping
   │   ╱   ╲╱   │      ╲  ╲  (broader peak)
   │  ╱     │    │       ╲   ╲___
   │ ╱      │    │        ╲______╲___
   │╱       │    │
   ├────────┼────┼──────────────── ω
   0       ω_r  ω₀
```

---

## 5. Worked Examples

### Example 1: Underdamped System

*$m = 1$ kg, $c = 2$ N·s/m, $k = 5$ N/m, $x(0) = 0.1$, $x'(0) = 0$*

$\gamma = c/(2m) = 1$, $\omega_0 = \sqrt{5}$, $\gamma < \omega_0$ → underdamped

$\omega_d = \sqrt{5 - 1} = 2$

$x = e^{-t}(c_1\cos 2t + c_2\sin 2t)$

$x(0) = c_1 = 0.1$

$x'(0) = -c_1 + 2c_2 = 0 \implies c_2 = 0.05$

$$x(t) = e^{-t}(0.1\cos 2t + 0.05\sin 2t)$$

### Example 2: Critically Damped

*$x'' + 4x' + 4x = 0$, $x(0) = 1$, $x'(0) = 2$*

$(m + 2)^2 = 0 \implies x = (c_1 + c_2 t)e^{-2t}$

$c_1 = 1$, $x'(0) = c_2 - 2c_1 = 2 \implies c_2 = 4$

$$x = (1 + 4t)e^{-2t}$$

### Example 3: Forced Undamped — Beats

*$x'' + 25x = 3\cos 4t$ (driving frequency $\omega = 4 \neq \omega_0 = 5$)*

$x_c = c_1\cos 5t + c_2\sin 5t$

$y_p = \dfrac{3}{25 - 16}\cos 4t = \dfrac{1}{3}\cos 4t$

With $x(0) = 0$, $x'(0) = 0$:

$x = -\dfrac{1}{3}\cos 5t + \dfrac{1}{3}\cos 4t = \dfrac{2}{3}\sin\left(\dfrac{t}{2}\right)\sin\left(\dfrac{9t}{2}\right)$

This produces **beats** — slow amplitude modulation:

```
  x(t)    BEATS (ω close to ω₀)
   │
   │    ╱╲          envelope
   │   ╱╱╲╲        ╱      ╲
   │  ╱╱  ╲╲      ╱        ╲
   │ ╱╱    ╲╲    ╱╱╲╲        ╲╲
───┼╱╱──────╲╲──╱╱──╲╲────────╲╲── t
   │╲╲      ╱╱  ╲╲  ╱╱        ╱╱
   │ ╲╲    ╱╱    ╲╲╱╱        ╱╱
   │  ╲╲  ╱╱      ╲╲
   │   ╲╲╱╱        ╲
   │    ╲╱
   │←──── beat period ────→│
```

### Example 4: Resonance

*$x'' + 25x = 3\cos 5t$ (driving frequency = natural frequency)*

$x_p = \dfrac{3t}{10}\sin 5t$ (resonance — multiply by $t$)

Amplitude grows as $\dfrac{3t}{10}$!

---

## 6. Quality Factor

The **Q-factor** measures the sharpness of resonance:

$$Q = \frac{\omega_0}{2\gamma} = \frac{\sqrt{mk}}{c}$$

| $Q$ value | Behavior |
|-----------|----------|
| $Q > 1/2$ | Underdamped |
| $Q = 1/2$ | Critically damped |
| $Q < 1/2$ | Overdamped |
| $Q \gg 1$ | Very sharp resonance peak |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Damped ODE** | $mx'' + cx' + kx = 0$ |
| **Overdamped** | $c^2 > 4mk$, no oscillation |
| **Critical** | $c^2 = 4mk$, fastest decay |
| **Underdamped** | $c^2 < 4mk$, $\omega_d = \sqrt{\omega_0^2 - \gamma^2}$ |
| **Forced ODE** | $mx'' + cx' + kx = F_0\cos\omega t$ |
| **Steady-state** | $x_p = A(\omega)\cos(\omega t - \delta)$ |
| **Resonance** | $\omega \approx \omega_0$, maximum amplitude |
| **Undamped resonance** | $x_p \propto t\sin\omega_0 t$ (grows) |
| **Q-factor** | $Q = \omega_0/(2\gamma)$ |

---

## Quick Revision Questions

1. Classify the motion of $x'' + 6x' + 9x = 0$ and solve with $x(0) = 2$, $x'(0) = -3$.

2. Find the damped frequency of $x'' + 2x' + 10x = 0$.

3. For $x'' + 0.1x' + 4x = 3\cos\omega t$, find the driving frequency $\omega$ that produces maximum amplitude.

4. Show that undamped resonance ($x'' + \omega_0^2 x = F_0\cos\omega_0 t$) gives $x_p = \dfrac{F_0}{2\omega_0}t\sin\omega_0 t$.

5. A system has $Q = 10$ and $\omega_0 = 100$ rad/s. Find the damping coefficient $\gamma$.

6. Explain physically why critically damped systems are preferred for instruments that need to settle quickly.

---

[← Previous: Simple Harmonic Motion](01-simple-harmonic-motion.md) | [Back to README](../README.md) | [Next: LCR Circuits →](03-lcr-circuits.md)
