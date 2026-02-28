# Chapter 4.1: Growth and Decay Problems

[← Previous: Singular Solutions](../03-First-Order-Higher-Degree/04-singular-solutions.md) | [Back to README](../README.md) | [Next: Newton's Law of Cooling →](02-newtons-law-of-cooling.md)

---

## Overview

The simplest and most widespread application of first-order ODEs is modeling quantities that grow or decay at a rate **proportional** to their current value. This single principle governs population dynamics, radioactive decay, compound interest, and chemical reactions.

---

## 1. The Fundamental Law

> **Law of Natural Growth/Decay**: The rate of change of a quantity is proportional to the quantity itself.

$$\boxed{\frac{dN}{dt} = kN}$$

| $k$ | Behavior | Example |
|-----|----------|---------|
| $k > 0$ | Exponential growth | Population, bacteria |
| $k < 0$ | Exponential decay | Radioactive decay, drug metabolism |
| $k = 0$ | No change | Equilibrium |

### Solution

$$N(t) = N_0 e^{kt}$$

where $N_0 = N(0)$ is the initial quantity.

```
   N
   │         ╱ k > 0 (growth)
   │        ╱
   │       ╱
   │      ╱
   │    _╱
N₀ ├── •──────────── ──── k = 0 (constant)
   │    ╲_
   │      ╲
   │       ╲         k < 0 (decay)
   │        ╲_____
   ├──┬──┬──┬──┬──┬── t
   0  1  2  3  4  5
```

---

## 2. Population Growth

### Malthusian (Exponential) Model

$$\frac{dP}{dt} = rP, \quad P(0) = P_0 \implies P(t) = P_0 e^{rt}$$

**Doubling time**: $t_d = \dfrac{\ln 2}{r}$

### Worked Example

*A bacteria culture starts with 1000 organisms and doubles in 3 hours. How many after 10 hours?*

**Given**: $P_0 = 1000$, $P(3) = 2000$

**Find $r$**: $2000 = 1000 e^{3r} \implies e^{3r} = 2 \implies r = \dfrac{\ln 2}{3}$

**At $t = 10$**: $P(10) = 1000 \cdot e^{10\ln 2/3} = 1000 \cdot 2^{10/3} \approx 10{,}079$

### Logistic Model (Improved)

Real populations have a **carrying capacity** $K$:

$$\frac{dP}{dt} = rP\left(1 - \frac{P}{K}\right)$$

This is a **Bernoulli equation** with $n = 2$. Solution:

$$P(t) = \frac{K}{1 + \left(\frac{K - P_0}{P_0}\right)e^{-rt}}$$

```
   P
   K ─ ─ ─────────────────────  carrying capacity
   │          _______________
   │        _/
   │      _/    S-shaped
   │    _/      (logistic)
   │   /
P₀ │──•        vs pure exponential ╱
   │ /                            ╱
   │/                            ╱
   ├──┬──┬──┬──┬──┬──┬──┬──── t
   0  2  4  6  8  10 12 14
```

---

## 3. Radioactive Decay

### Law

$$\frac{dN}{dt} = -\lambda N \implies N(t) = N_0 e^{-\lambda t}$$

### Half-Life

The **half-life** $t_{1/2}$ is the time for half the material to decay:

$$\frac{N_0}{2} = N_0 e^{-\lambda t_{1/2}} \implies t_{1/2} = \frac{\ln 2}{\lambda} \approx \frac{0.693}{\lambda}$$

### Common Isotopes

| Isotope | Half-Life | Application |
|---------|-----------|-------------|
| Carbon-14 | 5,730 years | Archaeological dating |
| Uranium-238 | 4.5 billion years | Geological dating |
| Iodine-131 | 8 days | Medical imaging |
| Cobalt-60 | 5.27 years | Cancer treatment |

### Worked Example: Carbon Dating

*A fossil has 30% of its original C-14. How old is it?*

$$0.30 N_0 = N_0 e^{-\lambda t}$$

$$\ln 0.30 = -\lambda t = -\frac{\ln 2}{5730} \cdot t$$

$$t = \frac{-\ln 0.30 \cdot 5730}{\ln 2} = \frac{1.204 \times 5730}{0.693} \approx 9{,}953 \text{ years}$$

---

## 4. Chemical Reactions (First-Order Kinetics)

In a first-order chemical reaction, the rate of consumption of reactant is proportional to its concentration:

$$\frac{d[A]}{dt} = -k[A] \implies [A](t) = [A]_0 e^{-kt}$$

### Worked Example

*If 90% of substance A decomposes in 20 minutes, find $k$ and the time for 99% decomposition.*

$[A](20) = 0.10[A]_0$:

$$e^{-20k} = 0.10 \implies k = \frac{\ln 10}{20} \approx 0.1151 \text{ min}^{-1}$$

For 99% decomposition: $e^{-kt} = 0.01$

$$t = \frac{\ln 100}{k} = \frac{2\ln 10}{\ln 10/20} = 40 \text{ minutes}$$

> **Insight**: It takes twice as long for 99% as for 90%!

---

## 5. Compound Interest (Continuous)

If principal $P$ grows at annual rate $r$ compounded continuously:

$$\frac{dP}{dt} = rP \implies P(t) = P_0 e^{rt}$$

**Comparison with discrete compounding**:

| Compounding | Formula | $P_0 = \$1000$, $r = 5\%$, $t = 10$ |
|-------------|---------|--------------------------------------|
| Annual | $P_0(1+r)^t$ | $\$1,628.89$ |
| Monthly | $P_0(1+r/12)^{12t}$ | $\$1,647.01$ |
| Continuous | $P_0 e^{rt}$ | $\$1,648.72$ |

---

## 6. Mixing Problems

A related application: a tank contains a well-stirred solution and liquid flows in and out.

### Setup

```
    Inflow                        Outflow
    ═══╗                         ╔═══
       ║  rate: rᵢₙ L/min       ║ rate: rₒᵤₜ L/min
       ║  concentration: cᵢₙ    ║ concentration: Q(t)/V(t)
       ▼                         ▼
   ┌────────────────────────┐
   │                        │
   │   Tank: V(t) liters    │
   │   Q(t) = amount of     │
   │   solute (kg)           │
   │                        │
   └────────────────────────┘

   Rate of change = Rate in - Rate out

   dQ/dt = rᵢₙ · cᵢₙ - rₒᵤₜ · Q(t)/V(t)
```

### Worked Example

*A 100L tank initially has pure water. Brine with 2 kg/L flows in at 5 L/min. Well-mixed solution flows out at 5 L/min. Find $Q(t)$.*

$$\frac{dQ}{dt} = 5 \times 2 - 5 \times \frac{Q}{100} = 10 - \frac{Q}{20}$$

This is linear: $\dfrac{dQ}{dt} + \dfrac{Q}{20} = 10$

I.F. $= e^{t/20}$

$$Qe^{t/20} = 200e^{t/20} + C$$

$$Q = 200 + Ce^{-t/20}$$

Apply $Q(0) = 0$: $C = -200$

$$Q(t) = 200(1 - e^{-t/20})$$

As $t \to \infty$: $Q \to 200$ kg (100L × 2 kg/L, as expected).

---

## Summary Table

| Application | ODE | Solution | Key Parameter |
|-------------|-----|----------|---------------|
| Population growth | $dP/dt = rP$ | $P = P_0 e^{rt}$ | Doubling time: $\ln 2/r$ |
| Radioactive decay | $dN/dt = -\lambda N$ | $N = N_0 e^{-\lambda t}$ | Half-life: $\ln 2/\lambda$ |
| Chemical reaction | $d[A]/dt = -k[A]$ | $[A] = [A]_0 e^{-kt}$ | Rate constant $k$ |
| Compound interest | $dP/dt = rP$ | $P = P_0 e^{rt}$ | Interest rate $r$ |
| Logistic growth | $dP/dt = rP(1-P/K)$ | $P = K/(1+ae^{-rt})$ | Carrying capacity $K$ |
| Mixing problem | $dQ/dt = r_{in}c_{in} - r_{out}Q/V$ | Linear ODE solution | Tank volume $V$ |

---

## Quick Revision Questions

1. A substance decays from 80g to 50g in 2 hours. Find the half-life.

2. A population triples every 5 years. Starting from 10,000, what's the population after 12 years?

3. In a mixing problem, if inflow rate ≠ outflow rate, what changes about $V(t)$?

4. If the half-life of a substance is 10 days, what fraction remains after 30 days?

5. Set up (but don't solve) the DE for a tank of 200L with 50g salt, where pure water flows in at 3 L/min and the mixture flows out at 3 L/min.

6. The logistic model approaches what value as $t \to \infty$? Why is this more realistic than the Malthusian model?

---

[← Previous: Singular Solutions](../03-First-Order-Higher-Degree/04-singular-solutions.md) | [Back to README](../README.md) | [Next: Newton's Law of Cooling →](02-newtons-law-of-cooling.md)
