# Chapter 8.4: Discrete-Time Transfer Function

## Overview

The **discrete-time transfer function** $H(z)$ characterizes an LTI discrete-time system in the $z$-domain. It relates input and output through $Y(z) = H(z)X(z)$, enables pole-zero analysis on the $z$-plane, and distinguishes between FIR and IIR systems. Difference equations map directly to $H(z)$ using the $z^{-1}$ delay operator.

---

## 8.4.1 Definition

$$H(z) = \frac{Y(z)}{X(z)} = \mathcal{Z}\{h[n]\}$$

For a general linear constant-coefficient difference equation (LCCDE):

$$\sum_{k=0}^{N} a_k y[n-k] = \sum_{k=0}^{M} b_k x[n-k]$$

$$H(z) = \frac{\sum_{k=0}^{M} b_k z^{-k}}{\sum_{k=0}^{N} a_k z^{-k}} = \frac{b_0 + b_1 z^{-1} + \cdots + b_M z^{-M}}{a_0 + a_1 z^{-1} + \cdots + a_N z^{-N}}$$

---

## 8.4.2 Poles and Zeros in the $z$-Plane

### Factored Form

$$H(z) = \frac{b_0}{a_0} \cdot \frac{(1-q_1 z^{-1})(1-q_2 z^{-1})\cdots(1-q_M z^{-1})}{(1-p_1 z^{-1})(1-p_2 z^{-1})\cdots(1-p_N z^{-1})}$$

Or equivalently in positive powers of $z$:

$$H(z) = K \cdot z^{N-M} \cdot \frac{(z-q_1)(z-q_2)\cdots(z-q_M)}{(z-p_1)(z-p_2)\cdots(z-p_N)}$$

```
z-PLANE POLE-ZERO PLOT:

Example: H(z) = (z-0.5)/[(z-0.8)(z+0.3)]

  Im(z) ↑
        │         Unit Circle
    1 ──┤       ╱──╲
        │     ╱      ╲
        │    │  ○  ×  │
   ─────×────┤──┼─────┤────→ Re(z)
  -0.3  │    │  0.5 0.8
        │     ╲      ╱
   -1 ──┤       ╲──╱
        │
  × = poles at 0.8, -0.3
  ○ = zero at 0.5
  All poles INSIDE unit circle → STABLE
```

---

## 8.4.3 FIR vs IIR Systems

| Property | FIR (Finite Impulse Response) | IIR (Infinite Impulse Response) |
|----------|------|------|
| $h[n]$ duration | Finite ($h[n] = 0$ for $n > M$) | Infinite |
| Poles | Only at $z = 0$ | At $z = p_i \neq 0$ |
| $H(z)$ form | Polynomial in $z^{-1}$ | Rational function |
| Feedback | No | Yes (recursive) |
| Stability | Always stable | Can be unstable |
| Phase | Can be linear | Generally nonlinear |
| Order needed | Higher for given spec | Lower |

### FIR Example

$$h[n] = \{1, 2, 3, 2, 1\}, \quad n = 0,1,2,3,4$$

$$H(z) = 1 + 2z^{-1} + 3z^{-2} + 2z^{-3} + z^{-4}$$

Only poles at $z = 0$ (four of them). Always stable.

### IIR Example

$$y[n] = 0.5y[n-1] + x[n]$$

$$H(z) = \frac{1}{1 - 0.5z^{-1}} = \frac{z}{z-0.5}$$

Pole at $z = 0.5$. $h[n] = (0.5)^n u[n]$ — infinite duration.

---

## 8.4.4 Block Diagram Representations

### Direct Form I

```
x[n]──→(+)──→(+)──→(+)──→ y[n]
        ↑     ↑     ↑       │
       b₀   b₁z⁻¹  b₂z⁻²  │
                             │
       ←────────────────────(+)
              ↑     ↑
           -a₁z⁻¹ -a₂z⁻²
```

### Direct Form II (Canonic)

$$H(z) = \frac{b_0 + b_1 z^{-1} + b_2 z^{-2}}{1 + a_1 z^{-1} + a_2 z^{-2}}$$

```
DIRECT FORM II (2nd order):

x[n]──→(+)──→ w[n] ──┬──→ [b₀] ──→(+)──→ y[n]
        ↑             │              ↑
        │          [z⁻¹]          [b₁]
        │             │              ↑
        │          [z⁻¹]──┬──→────────
        │             │   │          ↑
        │             │  [b₂]────────
        └──[-a₁]──────┤
        └──[-a₂]──────┘

Uses minimum number of delay elements (N delays for order N)
```

### Cascade Form

$$H(z) = H_1(z) \cdot H_2(z) \cdot \ldots$$

Factor $H(z)$ into second-order sections (biquads):

$$H(z) = \prod_k \frac{b_{0k} + b_{1k}z^{-1} + b_{2k}z^{-2}}{1 + a_{1k}z^{-1} + a_{2k}z^{-2}}$$

### Parallel Form

$$H(z) = H_1(z) + H_2(z) + \ldots$$

Via partial fraction expansion of $H(z)$.

---

## 8.4.5 From Difference Equation to Transfer Function

### Example 1

$$y[n] - 1.5y[n-1] + 0.5y[n-2] = x[n] + 0.5x[n-1]$$

$$Y(z)(1 - 1.5z^{-1} + 0.5z^{-2}) = X(z)(1 + 0.5z^{-1})$$

$$H(z) = \frac{1 + 0.5z^{-1}}{1 - 1.5z^{-1} + 0.5z^{-2}} = \frac{z(z+0.5)}{(z-1)(z-0.5)}$$

Zeros: $z = 0, -0.5$. Poles: $z = 1, 0.5$.

> Pole at $z = 1$ → marginally stable (on the unit circle).

### Example 2: Moving Average Filter

$$y[n] = \frac{1}{M}\sum_{k=0}^{M-1} x[n-k]$$

$$H(z) = \frac{1}{M}\sum_{k=0}^{M-1} z^{-k} = \frac{1}{M}\cdot\frac{1 - z^{-M}}{1 - z^{-1}} = \frac{1}{M}\cdot\frac{z^M - 1}{z^{M-1}(z-1)}$$

This is an FIR filter. $M$ zeros uniformly spaced on the unit circle.

---

## 8.4.6 Frequency Response from $H(z)$

$$H(e^{j\omega}) = H(z)\big|_{z = e^{j\omega}}$$

This evaluates $H(z)$ on the **unit circle**.

$$|H(e^{j\omega})| = \text{magnitude response (periodic with } 2\pi\text{)}$$

$$\angle H(e^{j\omega}) = \text{phase response}$$

### Geometric Interpretation

$$|H(e^{j\omega})| = |K| \cdot \frac{\prod |e^{j\omega} - q_i|}{\prod |e^{j\omega} - p_i|}$$

- Each factor $|e^{j\omega} - p_i|$ is the distance from pole $p_i$ to the point $e^{j\omega}$ on the unit circle
- At frequencies near a pole → magnitude peaks
- At frequencies near a zero → magnitude dips

```
GEOMETRIC FREQUENCY RESPONSE:

z-plane:
       ╱──╲
     ╱      ╲ ← unit circle
    │    ×   │ ← pole near circle
    │    │   │   → gain peak at this ω
     ╲  ○  ╱ ← zero
       ╲──╱    → gain null at this ω
       
|H(eʲω)|
  │     ╱╲  ← peak from nearby pole
  │    ╱  ╲
  │   ╱    ╲
  │──╱──────╲──────
  └──────────────→ ω
  0         π
```

---

## 8.4.7 Common Transfer Functions

| System | $H(z)$ | Type |
|--------|--------|------|
| Unit delay | $z^{-1}$ | FIR |
| First-order recursive | $1/(1-az^{-1})$ | IIR |
| Moving average ($M$-point) | $\frac{1}{M}\frac{1-z^{-M}}{1-z^{-1}}$ | FIR |
| First difference | $1-z^{-1}$ | FIR |
| Accumulator | $1/(1-z^{-1})$ | IIR |
| Second-order resonator | $1/(1-2r\cos\theta\,z^{-1}+r^2z^{-2})$ | IIR |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Transfer function | $H(z) = Y(z)/X(z)$ for zero ICs |
| Poles determine | Stability, natural response shape |
| Zeros determine | Frequency response nulls |
| FIR | No feedback, always stable, finite $h[n]$ |
| IIR | Has feedback (recursive), can be unstable |
| Frequency response | $H(e^{j\omega})$: evaluate on unit circle |
| Direct Form II | Minimum delay elements (canonic) |

---

## Quick Revision Questions

1. **Find $H(z)$ for $y[n] = 0.8y[n-1] + x[n] - x[n-1]$.**
   - $H(z) = \frac{1-z^{-1}}{1-0.8z^{-1}} = \frac{z(z-1)}{z(z-0.8)} = \frac{z-1}{z-0.8}$. Zero at $z=1$, pole at $z=0.8$.

2. **Is $H(z) = 1 + 2z^{-1} + z^{-2}$ FIR or IIR?**
   - FIR. It's a polynomial in $z^{-1}$ with no denominator (no feedback).

3. **How do you find the frequency response from $H(z)$?**
   - Substitute $z = e^{j\omega}$ and evaluate magnitude and phase.

4. **What is the advantage of cascade (biquad) form?**
   - Lower coefficient sensitivity to quantization, easier to design and tune individual second-order sections.

5. **A system has poles at $z = 0.9e^{\pm j\pi/4}$. What type of frequency response does it have?**
   - A resonant peak near $\omega = \pi/4$ (poles close to unit circle cause a gain peak at that frequency).

---

[← Previous: Inverse Z-Transform](03-inverse-z-transform.md) | [Next: Stability and Causality Analysis →](05-stability-analysis.md)
