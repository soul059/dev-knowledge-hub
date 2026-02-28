# Chapter 4.2: Newton's Law of Cooling

[← Previous: Growth and Decay](01-growth-and-decay.md) | [Back to README](../README.md) | [Next: RL and RC Circuits →](03-rl-and-rc-circuits.md)

---

## Overview

Newton's Law of Cooling states that the rate of change of temperature of an object is proportional to the difference between its temperature and the surrounding temperature. This leads to a first-order linear ODE with wide practical applications.

---

## 1. The Law

$$\boxed{\frac{dT}{dt} = -k(T - T_s)}$$

where:
- $T(t)$ = temperature of the object at time $t$
- $T_s$ = temperature of the surroundings (assumed constant)
- $k > 0$ = cooling constant (depends on the object and medium)

---

## 2. Solution

This is separable:

$$\frac{dT}{T - T_s} = -k \, dt \implies \ln|T - T_s| = -kt + C_0$$

$$T - T_s = Ce^{-kt} \implies \boxed{T(t) = T_s + (T_0 - T_s)e^{-kt}}$$

where $T_0 = T(0)$ is the initial temperature.

```
   T
   │
T₀ ├── •                          Cooling (T₀ > Tₛ)
   │    ╲__
   │       ╲___
   │           ╲_____
   │                 ╲________
Tₛ ├─────────────────────────── ─ ─  ambient temperature
   │                  ________
   │           ______╱
   │       ___╱
   │    __╱                        Warming (T₀ < Tₛ)
T₀ ├── •
   │
   ├──┬──┬──┬──┬──┬──┬──┬──── t
   0  1  2  3  4  5  6  7
```

### Key Properties

| Property | Value |
|----------|-------|
| Equilibrium (as $t \to \infty$) | $T \to T_s$ |
| Cooling or warming? | Depends on sign of $T_0 - T_s$ |
| Time to reach $T_s$? | Never exactly (asymptotic) |
| Larger $k$ | Faster approach to $T_s$ |

---

## 3. Worked Examples

### Example 1: Coffee Cooling

*A coffee at 90°C cools to 70°C in 5 minutes in a room at 20°C. When will it reach 40°C?*

**Setup**: $T_s = 20$, $T_0 = 90$, $T(5) = 70$

$$T(t) = 20 + 70e^{-kt}$$

**Find $k$**: $70 = 20 + 70e^{-5k}$

$$50 = 70e^{-5k} \implies e^{-5k} = \frac{5}{7} \implies k = \frac{1}{5}\ln\frac{7}{5} \approx 0.0673$$

**Find $t$ for $T = 40$**: 

$$40 = 20 + 70e^{-kt} \implies 20 = 70e^{-kt} \implies e^{-kt} = \frac{2}{7}$$

$$t = \frac{\ln(7/2)}{k} = \frac{\ln(7/2)}{\frac{1}{5}\ln(7/5)} = 5 \cdot \frac{\ln 3.5}{\ln 1.4} \approx 5 \times 3.726 \approx 18.6 \text{ minutes}$$

### Example 2: Forensic Application (Time of Death)

*A body at 32°C is found in a room at 22°C. One hour later, the temperature is 28°C. Normal body temperature is 37°C. When did death occur?*

**Setup**: Let $t = 0$ be when body is found. $T_s = 22$, $T(0) = 32$, $T(1) = 28$.

$$T(t) = 22 + 10e^{-kt}$$

**Find $k$**: $28 = 22 + 10e^{-k}$

$$6 = 10e^{-k} \implies k = \ln(10/6) = \ln(5/3) \approx 0.5108$$

**Find time of death** ($T = 37$, at time $t_d < 0$):

$$37 = 22 + 10e^{-kt_d} \implies 15 = 10e^{-kt_d}$$

$$e^{-kt_d} = 1.5 \implies t_d = \frac{-\ln 1.5}{k} = \frac{-0.4055}{0.5108} \approx -0.79 \text{ hours}$$

**Death occurred about 47 minutes before the body was found.**

### Example 3: Warming

*A frozen object at $-10°C$ is placed in a room at $25°C$. After 15 minutes, it is at $0°C$. When will it reach $20°C$?*

$$T(t) = 25 - 35e^{-kt}$$

$0 = 25 - 35e^{-15k} \implies e^{-15k} = 25/35 = 5/7$

$k = \dfrac{\ln(7/5)}{15}$

For $T = 20$: $20 = 25 - 35e^{-kt} \implies 35e^{-kt} = 5 \implies e^{-kt} = 1/7$

$t = \dfrac{\ln 7}{k} = \dfrac{15\ln 7}{\ln(7/5)} \approx 86.7$ minutes

---

## 4. Limitations and Extensions

### When Newton's Law Breaks Down

- **Large temperature differences**: radiation effects become important (Stefan-Boltzmann law)
- **Moving medium**: forced convection changes the effective $k$
- **Non-uniform objects**: requires heat equation (PDE)

### Extended Model: Variable Ambient Temperature

If $T_s = T_s(t)$, the ODE becomes:

$$\frac{dT}{dt} + kT = kT_s(t)$$

Still a linear first-order ODE, solvable by integrating factor!

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Law** | $dT/dt = -k(T - T_s)$ |
| **Solution** | $T(t) = T_s + (T_0 - T_s)e^{-kt}$ |
| **Equilibrium** | $T \to T_s$ as $t \to \infty$ |
| **Finding $k$** | Use two temperature readings at known times |
| **Forensics** | Work backward to find when $T = 37°C$ |
| **Assumption** | Surrounding temperature $T_s$ is constant |

---

## Quick Revision Questions

1. A metal rod at 150°C cools to 100°C in 10 minutes in a 25°C room. Find $k$.

2. Using Newton's Law, can an object ever reach exactly the temperature of its surroundings?

3. Two objects in the same room have $k_1 = 0.1$ and $k_2 = 0.3$. Which cools faster?

4. Set up the ODE for an object that warms from 5°C in a 30°C room.

5. If a body is at 34°C at noon and 30°C at 1 PM (room at 22°C, normal = 37°C), estimate time of death.

6. How does Newton's Law relate to the exponential decay model of Chapter 4.1?

---

[← Previous: Growth and Decay](01-growth-and-decay.md) | [Back to README](../README.md) | [Next: RL and RC Circuits →](03-rl-and-rc-circuits.md)
