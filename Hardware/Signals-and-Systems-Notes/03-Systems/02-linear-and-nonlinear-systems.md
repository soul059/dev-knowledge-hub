# Chapter 3.2: Linear and Non-Linear Systems

## Overview

Linearity is the most important property in signals and systems because it enables **superposition** — the ability to break a complex problem into simpler sub-problems, solve each independently, and combine results. This chapter provides rigorous tests and extensive examples.

---

## 3.2.1 Definition of Linearity

A system $\mathcal{T}$ is **linear** if and only if it satisfies the **principle of superposition**:

$$\mathcal{T}\{a_1 x_1(t) + a_2 x_2(t)\} = a_1 \mathcal{T}\{x_1(t)\} + a_2 \mathcal{T}\{x_2(t)\}$$

for **all** inputs $x_1, x_2$ and **all** constants $a_1, a_2$.

### Two Sub-Properties

| Property | Condition | Meaning |
|----------|-----------|---------|
| **Additivity** (homogeneity of degree 1) | $\mathcal{T}\{x_1 + x_2\} = \mathcal{T}\{x_1\} + \mathcal{T}\{x_2\}$ | Response to sum = sum of responses |
| **Scaling** (homogeneity) | $\mathcal{T}\{ax\} = a\mathcal{T}\{x\}$ | Scaling input scales output |

> **Both** must hold for linearity. A system can satisfy one but not the other.

```
LINEAR SYSTEM TEST:

    x₁(t) ──►[T]──► y₁(t)
    x₂(t) ──►[T]──► y₂(t)

    Check: a₁x₁(t) + a₂x₂(t) ──►[T]──► a₁y₁(t) + a₂y₂(t) ?

    If YES for ALL a₁, a₂, x₁, x₂ → LINEAR
    If NO for ANY case → NON-LINEAR
```

---

## 3.2.2 Step-by-Step Testing Procedure

### Method

1. Let $x_1(t) \to y_1(t)$ and $x_2(t) \to y_2(t)$ through the system
2. Form input $x_3(t) = a_1 x_1(t) + a_2 x_2(t)$
3. Compute system output for $x_3(t)$: call it $y_3(t)$
4. Compute $a_1 y_1(t) + a_2 y_2(t)$
5. Compare: if $y_3(t) = a_1 y_1(t) + a_2 y_2(t)$ for all choices → Linear

### Quick Check: Zero-Input Test

> If $x(t) = 0$ does not produce $y(t) = 0$, the system is **immediately non-linear**.

---

## 3.2.3 Examples — Linear Systems

### Example 1: $y(t) = 5x(t)$

- $y_1 = 5x_1$, $y_2 = 5x_2$
- $\mathcal{T}\{a_1 x_1 + a_2 x_2\} = 5(a_1 x_1 + a_2 x_2) = a_1(5x_1) + a_2(5x_2) = a_1 y_1 + a_2 y_2$ ✓

**Linear** ✓

### Example 2: $y(t) = \frac{dx(t)}{dt}$

- $\mathcal{T}\{a_1 x_1 + a_2 x_2\} = \frac{d}{dt}(a_1 x_1 + a_2 x_2) = a_1 \frac{dx_1}{dt} + a_2 \frac{dx_2}{dt} = a_1 y_1 + a_2 y_2$ ✓

**Linear** ✓

### Example 3: $y(t) = \int_{-\infty}^{t} x(\tau) d\tau$

- $\mathcal{T}\{a_1 x_1 + a_2 x_2\} = \int_{-\infty}^{t}(a_1 x_1 + a_2 x_2)d\tau = a_1 \int x_1 d\tau + a_2 \int x_2 d\tau = a_1 y_1 + a_2 y_2$ ✓

**Linear** ✓

### Example 4: $y[n] = \sum_{k=-\infty}^{n} x[k]$ (Accumulator)

**Linear** ✓ (summation distributes over addition and scaling)

---

## 3.2.4 Examples — Non-Linear Systems

### Example 5: $y(t) = x^2(t)$

- $\mathcal{T}\{a x\} = (ax)^2 = a^2 x^2 \neq a \cdot x^2 = a \cdot \mathcal{T}\{x\}$ (unless $a = 0$ or $a = 1$)

**Non-linear** ✗ (fails scaling)

### Example 6: $y(t) = x(t) + 3$

- Zero-input test: $x = 0 \Rightarrow y = 3 \neq 0$

**Non-linear** ✗ (fails zero-input test)

### Example 7: $y(t) = |x(t)|$

- Let $x_1 = 1$, $x_2 = -1$, $a_1 = a_2 = 1$
- $\mathcal{T}\{x_1 + x_2\} = |0| = 0$
- $\mathcal{T}\{x_1\} + \mathcal{T}\{x_2\} = 1 + 1 = 2 \neq 0$

**Non-linear** ✗ (fails additivity)

### Example 8: $y(t) = e^{x(t)}$

- $\mathcal{T}\{ax\} = e^{ax} \neq a \cdot e^x$

**Non-linear** ✗

### Example 9: $y(t) = x(t) \cdot \cos(\omega_0 t)$

- $\mathcal{T}\{a_1 x_1 + a_2 x_2\} = (a_1 x_1 + a_2 x_2)\cos(\omega_0 t) = a_1 x_1 \cos(\omega_0 t) + a_2 x_2 \cos(\omega_0 t) = a_1 y_1 + a_2 y_2$ ✓

**Linear** ✓ (note: $\cos(\omega_0 t)$ is a system parameter, not dependent on input)

---

## 3.2.5 Tricky Cases and Common Mistakes

| System | Linear? | Reasoning |
|--------|---------|-----------|
| $y = ax + b$ ($b \neq 0$) | ✗ | DC offset violates zero-input |
| $y = \ln(x)$ | ✗ | $\ln(ax) = \ln(a) + \ln(x) \neq a\ln(x)$ |
| $y = \max(x, 0)$ (ReLU) | ✗ | $\max(x_1+x_2) \neq \max(x_1) + \max(x_2)$ |
| $y = x \cdot u(t)$ | ✓ | $u(t)$ is system coefficient, not input-dependent |
| $y = \text{Re}\{x(t)\}$ | ✓ for real inputs | Taking real part is linear over reals |
| $y = x^*(t)$ (conjugate) | ✗ over $\mathbb{C}$ | $\mathcal{T}\{jx\} = -jx \neq j \cdot x^*$ |

---

## 3.2.6 Incremental Linearity

Some systems are **incrementally linear** — linear about an operating point but not through the origin.

$$y(t) = ax(t) + b$$

This satisfies **additivity and scaling for increments** $\Delta x$ around the operating point:

$$\Delta y = a \cdot \Delta x$$

> This is the basis of **small-signal analysis** in electronics (e.g., transistor circuits).

```
Non-linear characteristic:       Linearized about Q:
    │         ╱                     │      ╱
    │        ╱ ← Operating         │     ╱ ← Tangent at Q
    │       ╱    point Q           │    ╱
    │      ╱●                       │   ● Q
    │     ╱                         │  ╱
    │   ╱╱                          │ ╱
    │ ╱╱                            │╱
  0 ┤╱─────────→ x              0 ┤───────→ Δx
```

---

## 3.2.7 Implications of Linearity

```
LINEAR → Superposition holds

Consequences:
┌──────────────────────────────────────────┐
│ 1. Can decompose input into simpler parts│
│ 2. Solve for each part individually      │
│ 3. Add results → total output            │
│                                          │
│ Input decomposition:                     │
│ x(t) = Σ aₖ φₖ(t)                       │
│         ↓                                │
│ y(t) = Σ aₖ T{φₖ(t)}                    │
│                                          │
│ Common decompositions:                   │
│   - Impulses:  x(t) = ∫x(τ)δ(t-τ)dτ    │
│   - Sinusoids: x(t) = Σ cₖ eʲωₖt       │
│   - Step functions                       │
└──────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Superposition | $T\{ax_1+bx_2\} = aT\{x_1\}+bT\{x_2\}$ |
| Additivity | $T\{x_1+x_2\} = T\{x_1\}+T\{x_2\}$ |
| Scaling (Homogeneity) | $T\{ax\} = aT\{x\}$ |
| Zero-input test | If $T\{0\} \neq 0$, system is non-linear |
| Non-linear indicators | Powers, absolute values, min/max, exponentials, logs with input |
| Linear indicators | Multiplication by constants, integrals, derivatives, sums, delays |
| Incremental linearity | Linear about an operating point (small-signal models) |

---

## Quick Revision Questions

1. **Is $y[n] = n \cdot x[n]$ linear?**
   - Yes. $T\{ax_1+bx_2\} = n(ax_1+bx_2) = anx_1+bnx_2 = aT\{x_1\}+bT\{x_2\}$.

2. **Is $y(t) = x(t) + 1$ linear?**
   - No. Zero-input test: $x=0 \Rightarrow y=1 \neq 0$.

3. **What two sub-properties must both hold for a system to be linear?**
   - Additivity and scaling (homogeneity).

4. **Is the system $y = \sqrt{x(t)}$ linear?**
   - No. $\sqrt{ax} = \sqrt{a}\sqrt{x} \neq a\sqrt{x}$ for general $a$.

5. **Why is linearity so important for signal analysis?**
   - It enables decomposition of the input into elementary signals, analyzing each separately, and superposing the results.

6. **Can a system satisfy additivity but not scaling?**
   - Yes. Example: $y[n] = x^*[n]$ (conjugation) is additive over reals but fails $T\{jx\} = jT\{x\}$ over complex.

---

[← Previous: System Classification](01-system-classification.md) | [Next: Time-Invariant and Time-Variant Systems →](03-time-invariant-and-time-variant.md)
