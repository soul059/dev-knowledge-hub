# Chapter 2.3: Signal Addition and Multiplication

## Overview

Signal addition and multiplication are point-by-point (sample-by-sample) operations that combine two or more signals. These operations arise naturally in superposition, modulation, windowing, and filtering.

---

## 2.3.1 Signal Addition

### Definition

$$y(t) = x_1(t) + x_2(t) \quad \text{(CT)}$$
$$y[n] = x_1[n] + x_2[n] \quad \text{(DT)}$$

At each time instant, the output equals the **algebraic sum** of the input values.

### Graphical Construction

```
x₁(t):              x₂(t):              y(t) = x₁(t) + x₂(t):
    │                    │                       │
  2 ┤  ╭──╮           1 ┤──────╮              3 ┤  ╭╮
    │ ╱    ╲             │      ╲             2 ┤ ╱  ╲╲
  1 ┤╱      ╲          0 ┤──────╲──→ t       1 ┤╱     ╲╲
  0 ┤────────╲──→ t      0  1  2  3          0 ┤────────╲╲─→ t
    0  1  2  3  4                              0  1  2  3  4

Point-by-point:
t=0: y(0) = 0 + 1 = 1
t=1: y(1) = 2 + 1 = 3
t=2: y(2) = 1 + 0.5 ≈ 1.5
t=3: y(3) = 0 + 0 = 0
```

### Properties of Signal Addition

| Property | Formula |
|----------|---------|
| **Commutative** | $x_1 + x_2 = x_2 + x_1$ |
| **Associative** | $(x_1 + x_2) + x_3 = x_1 + (x_2 + x_3)$ |
| **Identity element** | $x + 0 = x$ |
| **Additive inverse** | $x + (-x) = 0$ |
| **Energy** | $E_y \neq E_{x_1} + E_{x_2}$ in general (cross terms!) |

### Energy of Sum

$$E_{x_1 + x_2} = E_{x_1} + E_{x_2} + 2\int_{-\infty}^{\infty} x_1(t) x_2(t) \, dt$$

The cross term $\int x_1 x_2 \, dt$ vanishes only if $x_1$ and $x_2$ are **orthogonal**.

### Superposition Principle

In **linear systems**, the response to a sum of inputs equals the sum of individual responses:

```
    x₁(t) ──►┌────────┐──► y₁(t)
              │ Linear │
    x₂(t) ──►│ System │──► y₂(t)
              └────────┘

    x₁(t) + x₂(t) ──►┌────────┐──► y₁(t) + y₂(t)
                       │ Linear │
                       │ System │
                       └────────┘
```

---

## 2.3.2 Signal Subtraction

$$y(t) = x_1(t) - x_2(t)$$

This is equivalent to addition of $x_1(t)$ and $-x_2(t)$ (amplitude-inverted $x_2$).

### Application: Differential Signaling

```
    ┌──────────────────────┐
    │  Positive wire (+)   │
────┤ x₁(t) = s(t) + n(t) ├──►  ┌────┐
    │                      │     │ Diff│
    │  Negative wire (-)   │     │ Amp │──► s₁ - s₂ = 2s(t)
────┤ x₂(t) = -s(t)+ n(t) ├──►  │    │   (noise cancels!)
    │                      │     └────┘
    └──────────────────────┘

    Noise n(t) is common to both wires → subtracted out
```

---

## 2.3.3 Signal Multiplication

### Definition

$$y(t) = x_1(t) \cdot x_2(t) \quad \text{(CT)}$$
$$y[n] = x_1[n] \cdot x_2[n] \quad \text{(DT)}$$

At each time instant, the output is the **product** of the input values.

### Graphical Construction

```
x₁(t):              x₂(t):              y(t) = x₁(t)·x₂(t):
    │                    │                       │
  2 ┤  ╭──╮           1 ┤────────             2 ┤  ╭╮
    │ ╱    ╲             │                       │ ╱  ╲
  1 ┤╱      ╲          0 ┤─────┬──→ t         0 ┤╱────╲──────→ t
  0 ┤────────╲──→ t      0    2                  0  1  2  3  4
    0  1  2  3  4        │                       │
                     x₂ = rect pulse          y = x₁ · x₂
                     (1 for t<2, 0 after)      (x₁ gated by x₂)
```

### Properties of Signal Multiplication

| Property | Formula |
|----------|---------|
| **Commutative** | $x_1 \cdot x_2 = x_2 \cdot x_1$ |
| **Associative** | $(x_1 \cdot x_2) \cdot x_3 = x_1 \cdot (x_2 \cdot x_3)$ |
| **Identity** | $x \cdot 1 = x$ |
| **Distributive** | $x_1 \cdot (x_2 + x_3) = x_1 \cdot x_2 + x_1 \cdot x_3$ |
| **Zero product** | $x \cdot 0 = 0$ |

---

## 2.3.4 Key Applications of Multiplication

### Application 1: Amplitude Modulation (AM)

```
Message m(t):             Carrier c(t):
    │                         │
  1 ┤  ╭──╮              1 ┤╱╲╱╲╱╲╱╲╱╲╱╲╱╲
    │ ╱    ╲                │
  0 ┤╱──────╲──→ t       0 ┤───────────────→ t
    │                       │
                         -1 ┤

AM signal y(t) = m(t)·c(t):
    │
    │  ╱╲╱╲                ← Envelope follows m(t)
    │╱╲╱╲╱╲╱╲
  0 ┤──────────╲╱╲╱╲──→ t
    │           ╲╱╲╱╲
    │
    Carrier oscillation "shaped" by message envelope
```

**Mathematical form**: $y(t) = m(t) \cos(\omega_c t)$

**Frequency domain effect**: Multiplication in time → Convolution in frequency → Spectrum shifts to $\pm \omega_c$.

### Application 2: Windowing

Multiplying a signal by a **window function** to extract a segment:

$$y(t) = x(t) \cdot w(t)$$

```
Signal x(t):                Window w(t):            Windowed y(t)=x(t)·w(t):
    │ ╱╲╱╲╱╲╱╲              │                          │
    │╱      ╲╱╲╱╲        1 ┤  ┌──────┐              │  ╱╲╱╲
  0 ┤────────────→ t       │  │      │            0 ┤─╱────╲─────→ t
    │                    0 ┤──┘      └──→ t          │
    │                       t₁      t₂               │ Only portion within
    Full signal             Gate window               │ [t₁, t₂] survives
```

Common window functions:

| Window | Shape | Sidelobe Level |
|--------|-------|----------------|
| Rectangular | Flat top | −13 dB (highest) |
| Hanning | Raised cosine | −31 dB |
| Hamming | Modified cosine | −43 dB |
| Blackman | Sum of cosines | −58 dB |

### Application 3: Sampling

Sampling is multiplication by an impulse train:

$$x_s(t) = x(t) \cdot \sum_{n=-\infty}^{\infty} \delta(t - nT_s)$$

```
x(t):                Impulse train p(t):      Sampled xs(t):
    │                    │                        │
    │ ╱╲╱╲╱╲         │ ↑  ↑  ↑  ↑  ↑          │ ↑  ↑
    │╱      ╲╱╲        │ │  │  │  │  │           │ │  │  ↑
  0 ┤──────────→ t   0 ┤─┼──┼──┼──┼──┼─→ t    0 ┤─┼──┼──┼──→ t
    │                    │                        │       │
                        Period Ts                  Amplitude = x(nTs)
```

---

## 2.3.5 Multiplication vs. Convolution

These two operations are fundamentally different but deeply related through Fourier analysis:

| Property | Multiplication | Convolution |
|----------|---------------|-------------|
| **Time domain** | $y(t) = x_1(t) \cdot x_2(t)$ | $y(t) = x_1(t) * x_2(t)$ |
| **Frequency domain** | $Y(\omega) = \frac{1}{2\pi}X_1(\omega) * X_2(\omega)$ | $Y(\omega) = X_1(\omega) \cdot X_2(\omega)$ |
| **Point-wise?** | Yes | No (integral/sum) |
| **Commutativity** | Yes | Yes |
| **Output duration** | Same as inputs | Sum of input durations |

> **Duality**: Multiplication in time ↔ Convolution in frequency (and vice versa).

---

## 2.3.6 Signal Operations in Block Diagrams

```
Addition (Summing junction):
    
    x₁(t) ───►(+)───► y(t) = x₁(t) + x₂(t)
               ↑
    x₂(t) ────┘


Multiplication (Multiplier):
    
    x₁(t) ───►(×)───► y(t) = x₁(t) · x₂(t)
               ↑
    x₂(t) ────┘


Scaling (Gain block):
    
    x(t) ───►[ A ]───► y(t) = A·x(t)


Combined system example (AM transmitter):
    
              ┌─────┐
    m(t) ────►│  A  ├───►(×)───►(+)───► [1 + Am(t)]cos(ωt)
              └─────┘    ↑       ↑
                         │       │
    cos(ωt) ─────────────┘   cos(ωt)
```

---

## 2.3.7 Worked Example: Signal Arithmetic

**Problem**: Given:
- $x_1(t) = 2u(t) - 2u(t-3)$ (rectangular pulse, height 2, from 0 to 3)
- $x_2(t) = t[u(t) - u(t-2)]$ (ramp from 0 to 2, then zero)

Find $y(t) = x_1(t) + x_2(t)$ and $z(t) = x_1(t) \cdot x_2(t)$.

**Solution** — Analyze by intervals:

| Interval | $x_1(t)$ | $x_2(t)$ | $y = x_1 + x_2$ | $z = x_1 \cdot x_2$ |
|----------|-----------|-----------|------------------|---------------------|
| $t < 0$ | 0 | 0 | 0 | 0 |
| $0 \leq t < 2$ | 2 | $t$ | $2 + t$ | $2t$ |
| $2 \leq t < 3$ | 2 | 0 | 2 | 0 |
| $t \geq 3$ | 0 | 0 | 0 | 0 |

```
y(t) = x₁(t) + x₂(t):        z(t) = x₁(t)·x₂(t):
    │                              │
  4 ┤        ╱                   4 ┤    ╱
    │       ╱                      │   ╱
  3 ┤      ╱                     3 ┤  ╱
    │     ╱                        │ ╱
  2 ┤    ╱──────┐                2 ┤╱
    │   ╱       │                  │
  1 ┤  ╱        │                0 ┤────────────→ t
    │ ╱         │                  0  1  2  3
  0 ┤╱──────────└──→ t
    0  1  2  3  4              z = 2t for [0,2], 0 elsewhere
```

---

## Summary Table

| Operation | Symbol | Time Domain | Frequency Domain Equivalent |
|-----------|--------|-------------|---------------------------|
| Addition | $+$ | $x_1(t) + x_2(t)$ | $X_1(\omega) + X_2(\omega)$ |
| Subtraction | $-$ | $x_1(t) - x_2(t)$ | $X_1(\omega) - X_2(\omega)$ |
| Multiplication | $\times$ | $x_1(t) \cdot x_2(t)$ | $\frac{1}{2\pi}X_1(\omega) * X_2(\omega)$ |
| Scaling | $A$ | $Ax(t)$ | $AX(\omega)$ |

---

## Quick Revision Questions

1. **If $x_1(t)$ and $x_2(t)$ are orthogonal, what is $E_{x_1+x_2}$?**
   - $E_{x_1+x_2} = E_{x_1} + E_{x_2}$ (cross-term is zero).

2. **What operation is used in AM to put a baseband signal onto a carrier?**
   - Multiplication: $y(t) = m(t) \cdot \cos(\omega_c t)$.

3. **What does multiplying in time correspond to in the frequency domain?**
   - Convolution in frequency (scaled by $1/2\pi$).

4. **How do you extract a portion of a signal between $t = 2$ and $t = 5$?**
   - Multiply by a window: $y(t) = x(t) \cdot [u(t-2) - u(t-5)]$.

5. **Is signal addition commutative? Associative?**
   - Yes to both: $x_1 + x_2 = x_2 + x_1$ and $(x_1 + x_2) + x_3 = x_1 + (x_2 + x_3)$.

6. **What is the difference between the output duration for multiplication vs. convolution?**
   - Multiplication: output duration = overlap of input durations. Convolution: output duration = sum of input durations.

---

[← Previous: Amplitude Scaling](02-amplitude-scaling.md) | [Next: Convolution Operation →](04-convolution-operation.md)
