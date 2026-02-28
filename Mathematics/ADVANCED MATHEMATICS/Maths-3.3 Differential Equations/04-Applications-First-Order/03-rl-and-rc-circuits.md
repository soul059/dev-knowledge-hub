# Chapter 4.3: RL and RC Circuits

[← Previous: Newton's Law of Cooling](02-newtons-law-of-cooling.md) | [Back to README](../README.md) | [Next: Orthogonal Trajectories →](04-orthogonal-trajectories.md)

---

## Overview

Electrical circuits containing resistors with inductors (RL) or capacitors (RC) are governed by first-order linear ODEs. Kirchhoff's Voltage Law (KVL) provides the mathematical framework for these circuit equations.

---

## 1. Kirchhoff's Voltage Law (KVL)

> The sum of all voltage drops around any closed loop equals zero.

### Component Voltage Relations

| Component | Symbol | Voltage Drop |
|-----------|--------|-------------|
| Resistor ($R$) | ─/\/\/─ | $V_R = Ri$ |
| Inductor ($L$) | ─⌇⌇⌇─ | $V_L = L\dfrac{di}{dt}$ |
| Capacitor ($C$) | ─\|\|─ | $V_C = \dfrac{q}{C} = \dfrac{1}{C}\displaystyle\int i\, dt$ |
| EMF Source ($E$) | ─⊕─ | $E(t)$ |

where $i = dq/dt$ (current is rate of change of charge).

---

## 2. RL Circuits

### Circuit Diagram

```
        R (Ω)           L (H)
  ┌───/\/\/───┬───⌇⌇⌇⌇───┐
  │           │           │
  ⊕ E(t)     │ (i →)     │
  │           │           │
  └───────────┴───────────┘
         Switch (S)
```

### The ODE

By KVL: $L\dfrac{di}{dt} + Ri = E(t)$

$$\boxed{\frac{di}{dt} + \frac{R}{L}i = \frac{E(t)}{L}}$$

This is a **first-order linear ODE** with $P = R/L$ and $Q = E(t)/L$.

### Case A: Constant EMF ($E = E_0$)

Integrating factor: $\mu = e^{(R/L)t}$

$$i(t) = e^{-(R/L)t}\left[\int \frac{E_0}{L}e^{(R/L)t}\,dt + C\right]$$

$$i(t) = \frac{E_0}{R} + Ce^{-(R/L)t}$$

With initial condition $i(0) = 0$ (switch closes at $t = 0$):

$$\boxed{i(t) = \frac{E_0}{R}\left(1 - e^{-(R/L)t}\right)}$$

```
   i(t)
   │
E₀/R├─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  steady state
   │                 ___________
   │            ____╱
   │         __╱
   │       _╱
   │     _╱
   │   _╱
   │  ╱
   │_╱
   ├──┬──┬──┬──┬──┬──┬──┬──── t
   0  τ  2τ 3τ 4τ 5τ 6τ 7τ

   τ = L/R  (time constant)
```

### Time Constant: $\tau = L/R$

| Time | Current | % of Steady State |
|------|---------|-------------------|
| $t = \tau$ | $\dfrac{E_0}{R}(1 - e^{-1})$ | 63.2% |
| $t = 2\tau$ | $\dfrac{E_0}{R}(1 - e^{-2})$ | 86.5% |
| $t = 3\tau$ | $\dfrac{E_0}{R}(1 - e^{-3})$ | 95.0% |
| $t = 5\tau$ | $\dfrac{E_0}{R}(1 - e^{-5})$ | 99.3% |

### Case B: Sinusoidal EMF ($E = E_0 \sin\omega t$)

$$\frac{di}{dt} + \frac{R}{L}i = \frac{E_0}{L}\sin\omega t$$

Solution (I.F. method):

$$i(t) = \frac{E_0}{R^2 + \omega^2 L^2}(R\sin\omega t - \omega L\cos\omega t) + Ce^{-(R/L)t}$$

The transient part $Ce^{-(R/L)t}$ dies out; the steady-state oscillation persists.

---

## 3. RC Circuits

### Circuit Diagram

```
        R (Ω)           C (F)
  ┌───/\/\/───┬───||────┐
  │           │         │
  ⊕ E(t)     │ (i →)   │
  │           │         │
  └───────────┴─────────┘
         Switch (S)
```

### The ODE (in Terms of Charge $q$)

By KVL: $Ri + \dfrac{q}{C} = E(t)$

Since $i = dq/dt$:

$$\boxed{R\frac{dq}{dt} + \frac{q}{C} = E(t)}$$

$$\frac{dq}{dt} + \frac{1}{RC}q = \frac{E(t)}{R}$$

### Case A: Constant EMF ($E = E_0$), Charging

I.F.: $\mu = e^{t/(RC)}$

With $q(0) = 0$:

$$\boxed{q(t) = CE_0\left(1 - e^{-t/(RC)}\right)}$$

$$\boxed{i(t) = \frac{dq}{dt} = \frac{E_0}{R}e^{-t/(RC)}}$$

```
   q(t)                            i(t)
   │                                │
CE₀├─ ─ ─ ─ ─ ─ ─ ─ ─          E₀/R├── •
   │            ________            │    ╲
   │         __╱                    │     ╲__
   │       _╱                       │        ╲___
   │     _╱                         │            ╲_____
   │   _╱                           │                 ╲______
   │  ╱                             │
   │_╱                              │
   ├──┬──┬──┬──┬── t                ├──┬──┬──┬──┬── t
   0  τ  2τ 3τ 4τ                  0  τ  2τ 3τ 4τ

   Charge builds up                 Current decays
   τ = RC                           τ = RC
```

### Case B: Discharge ($E = 0$, initial charge $q_0$)

$$R\frac{dq}{dt} + \frac{q}{C} = 0 \implies q(t) = q_0 e^{-t/(RC)}$$

$$i(t) = -\frac{q_0}{RC}e^{-t/(RC)}$$

(Negative sign: current flows opposite to charging direction.)

### Time Constant: $\tau = RC$

| Analogy | RL Circuit | RC Circuit |
|---------|-----------|-----------|
| Time constant | $\tau = L/R$ | $\tau = RC$ |
| Steady-state current | $E_0/R$ | 0 (charged) |
| Steady-state charge | — | $CE_0$ |
| Transient | $e^{-t/\tau}$ | $e^{-t/\tau}$ |

---

## 4. Worked Examples

### Example 1: RL Circuit

*An RL circuit has $R = 10\,\Omega$, $L = 2\,H$, and $E = 50\,V$. Find $i(t)$ if $i(0) = 0$.*

$$\frac{di}{dt} + 5i = 25$$

I.F.: $e^{5t}$

$$i(t) = 5(1 - e^{-5t}) \text{ amperes}$$

Time constant: $\tau = L/R = 2/10 = 0.2$ s

At $t = 0.2$: $i = 5(1 - e^{-1}) \approx 3.16$ A

At $t = 1$: $i = 5(1 - e^{-5}) \approx 4.97$ A (nearly steady state)

### Example 2: RC Circuit Charging

*An RC circuit has $R = 1000\,\Omega$, $C = 10\,\mu F$, $E = 12\,V$. Find $q(t)$ and $i(t)$.*

Time constant: $\tau = RC = 1000 \times 10 \times 10^{-6} = 0.01$ s $= 10$ ms

$$q(t) = 12 \times 10 \times 10^{-6}(1 - e^{-100t}) = 1.2 \times 10^{-4}(1 - e^{-100t}) \text{ C}$$

$$i(t) = \frac{12}{1000}e^{-100t} = 0.012\,e^{-100t} \text{ A}$$

At $t = 10$ ms: $q \approx 7.59 \times 10^{-5}$ C, $i \approx 4.41$ mA

### Example 3: RL with Sinusoidal EMF

*$R = 4\,\Omega$, $L = 1\,H$, $E = 20\sin 2t$. Find $i(t)$ with $i(0) = 0$.*

$$\frac{di}{dt} + 4i = 20\sin 2t$$

I.F.: $e^{4t}$

$$ie^{4t} = 20\int e^{4t}\sin 2t\,dt = 20 \cdot \frac{e^{4t}}{20}(4\sin 2t - 2\cos 2t) + C$$

$$i(t) = 4\sin 2t - 2\cos 2t + Ce^{-4t}$$

$i(0) = 0$: $0 = -2 + C \implies C = 2$

$$\boxed{i(t) = 4\sin 2t - 2\cos 2t + 2e^{-4t}}$$

Transient: $2e^{-4t}$ (dies out)
Steady-state: $4\sin 2t - 2\cos 2t = 2\sqrt{5}\sin(2t - \arctan(1/2))$

### Example 4: RC Discharge

*A 50 $\mu F$ capacitor charged to 100 V discharges through $2\,k\Omega$. Find time for voltage to reach 10 V.*

$\tau = RC = 2000 \times 50 \times 10^{-6} = 0.1$ s

$V(t) = \dfrac{q}{C} = 100e^{-t/0.1} = 100e^{-10t}$

$10 = 100e^{-10t} \implies t = \dfrac{\ln 10}{10} \approx 0.23$ s

---

## 5. Comparison: RL vs RC

```
  RL CIRCUIT RESPONSE (i)            RC CIRCUIT RESPONSE (q)
   │                                  │
   │          _______________         │          _______________
   │       __╱                        │       __╱
   │     _╱                           │     _╱
   │   _╱                             │   _╱
   │  ╱                               │  ╱
   │_╱                                │_╱
   │                                  │
   ├──────────── t                    ├──────────── t

  CURRENT grows to E₀/R              CHARGE grows to CE₀
  τ = L/R                            τ = RC
  Energy stored in L: ½Li²           Energy stored in C: ½CV²
```

---

## Summary Table

| Concept | RL Circuit | RC Circuit |
|---------|-----------|-----------|
| **ODE** | $L\dfrac{di}{dt} + Ri = E$ | $R\dfrac{dq}{dt} + \dfrac{q}{C} = E$ |
| **Time constant** | $\tau = L/R$ | $\tau = RC$ |
| **Growth response** | $i = \dfrac{E}{R}(1 - e^{-t/\tau})$ | $q = CE(1 - e^{-t/\tau})$ |
| **Decay response** | $i = i_0 e^{-t/\tau}$ | $q = q_0 e^{-t/\tau}$ |
| **63.2% reached at** | $t = \tau$ | $t = \tau$ |
| **Essentially complete** | $t \approx 5\tau$ | $t \approx 5\tau$ |
| **Energy storage** | Magnetic: $\frac{1}{2}Li^2$ | Electric: $\frac{1}{2}CV^2$ |

---

## Quick Revision Questions

1. Derive the current in an RL circuit with $R = 5\,\Omega$, $L = 0.5\,H$, $E = 20\,V$, $i(0) = 0$.

2. What is the time constant of an RC circuit with $R = 2\,k\Omega$, $C = 100\,\mu F$? How long until the capacitor is 99% charged?

3. Show that in an RL circuit, the current reaches approximately 63.2% of its final value at $t = \tau$.

4. Write the ODE for an RC circuit with $E(t) = E_0 \cos\omega t$ and solve it.

5. A capacitor discharges from 50 V to 20 V in 3 seconds. Find $RC$.

6. Compare the energy stored in an inductor vs. a capacitor at steady state in their respective circuits.

---

[← Previous: Newton's Law of Cooling](02-newtons-law-of-cooling.md) | [Back to README](../README.md) | [Next: Orthogonal Trajectories →](04-orthogonal-trajectories.md)
