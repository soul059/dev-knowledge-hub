# Chapter 1.5: Even and Odd Signals

## Overview

Every signal can be uniquely decomposed into an **even component** and an **odd component**. This decomposition is a powerful analytical tool used extensively in Fourier analysis and system characterization.

---

## 1.5.1 Definition

### Even Signal

A signal $x(t)$ is **even** (symmetric) if:

$$x(-t) = x(t) \quad \forall t$$

A DT signal $x[n]$ is **even** if: $x[-n] = x[n]$

```
Even Signal (symmetric about t = 0):
    │
  2 ┤     ╭╮
    │    ╱  ╲
  1 ┤   ╱    ╲
    │  ╱      ╲
  0 ┤─╱────────╲──────→ t
    │
   -3  -2  -1  0  1  2  3
    │
    Mirror image about y-axis
    Left side = Right side
```

### Odd Signal

A signal $x(t)$ is **odd** (anti-symmetric) if:

$$x(-t) = -x(t) \quad \forall t$$

A DT signal $x[n]$ is **odd** if: $x[-n] = -x[n]$

> **Important**: For an odd signal, $x(0) = 0$ (always).

```
Odd Signal (anti-symmetric about origin):
    │
  2 ┤              ╱
    │            ╱
  1 ┤          ╱
    │        ╱
  0 ┤───────●──────────→ t
    │      ╱
 -1 ┤    ╱
    │  ╱
 -2 ┤╱
    │
   -3  -2  -1  0  1  2  3
    │
    x(0) = 0 always for odd signals
    Rotational symmetry about origin
```

---

## 1.5.2 Examples of Even and Odd Signals

| Signal | Even or Odd? | Reason |
|--------|-------------|--------|
| $\cos(\omega t)$ | Even | $\cos(-\omega t) = \cos(\omega t)$ |
| $\sin(\omega t)$ | Odd | $\sin(-\omega t) = -\sin(\omega t)$ |
| $e^{-a\|t\|}$ | Even | $e^{-a\|-t\|} = e^{-a\|t\|}$ |
| $t$ | Odd | $-t = -(t)$ |
| $t^2$ | Even | $(-t)^2 = t^2$ |
| $t^3$ | Odd | $(-t)^3 = -t^3$ |
| $u(t)$ | Neither | $u(-t) \neq u(t)$ and $u(-t) \neq -u(t)$ |
| $\delta(t)$ | Even | $\delta(-t) = \delta(t)$ |

```
cos(t) — Even:                  sin(t) — Odd:
    │                               │
  1 ┤  ╲    ╱                    1 ┤        ╱╲
    │   ╲  ╱                       │       ╱  ╲
  0 ┤────╲╱─────→ t             0 ┤──────●────╲──→ t
    │   ╱  ╲                       │     ╱      ╲
 -1 ┤  ╱    ╲                  -1 ┤    ╱        ╲╱
    │                               │
    │ Mirror symmetric              │ Anti-symmetric
```

---

## 1.5.3 Even-Odd Decomposition

**Theorem**: Any signal can be uniquely decomposed into the sum of an even part and an odd part:

$$x(t) = x_e(t) + x_o(t)$$

where:

$$\boxed{x_e(t) = \frac{x(t) + x(-t)}{2}} \qquad \boxed{x_o(t) = \frac{x(t) - x(-t)}{2}}$$

### Verification

- $x_e(t) + x_o(t) = \frac{x(t) + x(-t)}{2} + \frac{x(t) - x(-t)}{2} = x(t)$ ✓
- $x_e(-t) = \frac{x(-t) + x(t)}{2} = x_e(t)$ → Even ✓
- $x_o(-t) = \frac{x(-t) - x(t)}{2} = -x_o(t)$ → Odd ✓

---

## 1.5.4 Worked Examples

### Example 1: Decompose $x(t) = e^{at}$

**Step 1**: Find $x(-t) = e^{-at}$

**Step 2**: Even part:
$$x_e(t) = \frac{e^{at} + e^{-at}}{2} = \cosh(at)$$

**Step 3**: Odd part:
$$x_o(t) = \frac{e^{at} - e^{-at}}{2} = \sinh(at)$$

**Result**: $e^{at} = \cosh(at) + \sinh(at)$ ✓

```
x(t) = eᵃᵗ:            Even: cosh(at):        Odd: sinh(at):
    │      ╱                │                      │      ╱
    │     ╱                 │  ╲    ╱              │     ╱
    │    ╱                  │   ╲  ╱               │    ╱
    │   ╱                   │    ╲╱                │   ╱
  1 ┤──╱                  1 ┤────╲─              0 ┤──●──────→ t
    │ ╱                     │                      │╱
    │╱                      │                     ╱│
    ╱───────→ t             └──────→ t          ╱  │
```

### Example 2: Decompose the Unit Step $u(t)$

**Step 1**: $u(-t)$ is the "anti-causal" step function

**Step 2**: Even part:
$$u_e(t) = \frac{u(t) + u(-t)}{2} = \frac{1}{2} \quad (\text{for } t \neq 0)$$

**Step 3**: Odd part:
$$u_o(t) = \frac{u(t) - u(-t)}{2} = \frac{1}{2}\text{sgn}(t)$$

**Result**: 
$$\boxed{u(t) = \frac{1}{2} + \frac{1}{2}\text{sgn}(t)}$$

```
u(t):               u(-t):              u_e(t) = 1/2:        u_o(t) = sgn(t)/2:
    │                   │                     │                     │
  1 ┤    ┌──────     1 ┤──────┐           1/2 ┤────────────     1/2 ┤        ┌─────
    │    │              │     │               │                     │        │
  0 ┤────┘           0 ┤     └──          0/2 ┤                   0 ┤────────●
    │                   │                     │                     │        │
   -2  0  2  → t      -2  0  2  → t        → t                -1/2 ┤────────┘
                                                                   -2  0  2  → t
```

### Example 3: DT Signal Decomposition

Given $x[n] = \{1, \underline{3}, 2, 5\}$ for $n = -1, 0, 1, 2$:

**Step 1**: Find $x[-n] = \{5, 2, \underline{3}, 1\}$ for $n = -2, -1, 0, 1$

**Step 2**: Tabulate values:

| $n$ | $x[n]$ | $x[-n]$ | $x_e[n] = \frac{x[n]+x[-n]}{2}$ | $x_o[n] = \frac{x[n]-x[-n]}{2}$ |
|-----|---------|----------|----------------------------------|----------------------------------|
| $-2$ | $0$ | $5$ | $2.5$ | $-2.5$ |
| $-1$ | $1$ | $2$ | $1.5$ | $-0.5$ |
| $0$ | $3$ | $3$ | $3$ | $0$ |
| $1$ | $2$ | $1$ | $1.5$ | $0.5$ |
| $2$ | $5$ | $0$ | $2.5$ | $2.5$ |

**Verification**: $x_e[n]$ is symmetric: $x_e[-1] = x_e[1] = 1.5$ ✓, $x_o[0] = 0$ ✓

---

## 1.5.5 Properties of Even and Odd Signals

### Algebraic Properties

| Operation | Result |
|-----------|--------|
| Even + Even | Even |
| Odd + Odd | Odd |
| Even × Even | Even |
| Odd × Odd | Even |
| Even × Odd | Odd |
| Even + Odd | Neither (in general) |

### Integral/Summation Properties

**Even signal integration**:
$$\int_{-a}^{a} x_e(t) \, dt = 2 \int_{0}^{a} x_e(t) \, dt$$

**Odd signal integration**:
$$\int_{-a}^{a} x_o(t) \, dt = 0$$

> **Significance**: When computing Fourier coefficients, if the signal is even, the sine terms vanish; if odd, the cosine terms vanish.

### Energy Properties

For a real signal $x(t) = x_e(t) + x_o(t)$:

$$E_x = E_{x_e} + E_{x_o}$$

The cross-energy term is zero:
$$\int_{-\infty}^{\infty} x_e(t) \cdot x_o(t) \, dt = 0$$

> Even and odd components are **energy-orthogonal**.

---

## 1.5.6 Conjugate Symmetry for Complex Signals

For a complex signal $x(t)$:

**Conjugate Symmetric (Even)**:
$$x(-t) = x^*(t)$$

**Conjugate Anti-Symmetric (Odd)**:
$$x(-t) = -x^*(t)$$

Any complex signal can be decomposed:
$$x(t) = x_{cs}(t) + x_{ca}(t)$$

where:
$$x_{cs}(t) = \frac{x(t) + x^*(-t)}{2}, \qquad x_{ca}(t) = \frac{x(t) - x^*(-t)}{2}$$

---

## 1.5.7 Applications

| Application | How Even/Odd is Used |
|-------------|---------------------|
| **Fourier Series** | Even signals → only cosine terms; Odd signals → only sine terms |
| **Fourier Transform** | Real even signal → real even transform; Real odd signal → imaginary odd transform |
| **System Analysis** | Even impulse response → zero-phase filter; Odd → 90° phase shift |
| **Signal Estimation** | Autocorrelation is always even |
| **Image Processing** | Even symmetry exploited for DCT (JPEG compression) |

---

## 1.5.8 Decision Flowchart

```
        Given signal x(t)
              │
              ▼
      Is x(-t) = x(t)?
         ╱         ╲
       YES          NO
        │            │
      EVEN    Is x(-t) = -x(t)?
     SIGNAL      ╱         ╲
               YES          NO
                │            │
              ODD        NEITHER
             SIGNAL    (decompose into
                        even + odd parts)
```

---

## Summary Table

| Concept | Formula/Condition | Key Point |
|---------|-------------------|-----------|
| Even signal | $x(-t) = x(t)$ | Symmetric about $t = 0$ |
| Odd signal | $x(-t) = -x(t)$ | Anti-symmetric; $x(0) = 0$ |
| Even part | $x_e(t) = \frac{x(t)+x(-t)}{2}$ | Always even |
| Odd part | $x_o(t) = \frac{x(t)-x(-t)}{2}$ | Always odd |
| Decomposition | $x(t) = x_e(t) + x_o(t)$ | Unique decomposition |
| Even × Even | Even | Sign rules like numbers |
| Odd × Odd | Even | |
| Even × Odd | Odd | |
| $\int_{-a}^{a} x_o(t)dt$ | $= 0$ | Odd integral over symmetric limits |
| Energy | $E_x = E_{x_e} + E_{x_o}$ | Cross-energy = 0 |

---

## Quick Revision Questions

1. **Is $x(t) = t \cdot \cos(t)$ even or odd?**
   - $t$ is odd, $\cos(t)$ is even → product is **odd**.

2. **What is the even part of $e^{2t}$?**
   - $x_e(t) = (e^{2t} + e^{-2t})/2 = \cosh(2t)$.

3. **Why must an odd signal always be zero at $t = 0$?**
   - From $x(-t) = -x(t)$, setting $t = 0$: $x(0) = -x(0)$, so $2x(0) = 0$, hence $x(0) = 0$.

4. **Decompose $u(t)$ into even and odd parts.**
   - $u_e(t) = 1/2$ (constant) and $u_o(t) = \text{sgn}(t)/2$.

5. **What is $\int_{-5}^{5} t^3 \, dt$?**
   - Zero, because $t^3$ is an odd function and the integration limits are symmetric.

6. **If a signal is both even and odd, what must the signal be?**
   - The zero signal: $x(t) = 0$ for all $t$.

---

[← Previous: Standard Signals](04-standard-signals.md) | [Back to Unit 1](README.md) | [Next Unit: Signal Operations →](../02-Signal-Operations/README.md)
