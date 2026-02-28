# Chapter 7.1: Definition and Region of Convergence

## Overview

The Laplace Transform introduces the complex frequency $s = \sigma + j\omega$, replacing the purely imaginary $j\omega$ of the Fourier Transform. The **Region of Convergence (ROC)** specifies the values of $s$ for which the transform integral converges and is essential for unique signal identification.

---

## 7.1.1 Bilateral Laplace Transform

$$X(s) = \int_{-\infty}^{\infty} x(t) \, e^{-st} \, dt, \quad s = \sigma + j\omega$$

### Inverse

$$x(t) = \frac{1}{2\pi j} \int_{\sigma-j\infty}^{\sigma+j\infty} X(s) \, e^{st} \, ds$$

---

## 7.1.2 Unilateral (One-Sided) Laplace Transform

$$X(s) = \int_{0^-}^{\infty} x(t) \, e^{-st} \, dt$$

- Assumes $x(t) = 0$ for $t < 0$ (or ignores $t < 0$)
- Handles **initial conditions** naturally
- Most commonly used in engineering

---

## 7.1.3 Relation to Fourier Transform

$$X(s)\big|_{s = j\omega} = X(j\omega) \quad \text{(if the } j\omega \text{ axis is in the ROC)}$$

The Laplace Transform is the Fourier Transform of $x(t)e^{-\sigma t}$:

$$X(\sigma + j\omega) = \mathcal{F}\{x(t)e^{-\sigma t}\}$$

```
s-PLANE:

  jω ↑
     │
     │    ROC
     │  ╔═══════
     │  ║    
  ───┼──╠═══════──→ σ
     │  ║
     │  ╔═══════
     │
     
  If jω axis is in ROC → FT exists
  s = σ + jω (complex frequency)
```

---

## 7.1.4 Common Transform Pairs

### Pair 1: $e^{-at}u(t)$ (Right-sided exponential)

$$X(s) = \int_0^{\infty} e^{-at}e^{-st}dt = \frac{1}{s+a}, \quad \text{ROC: Re}(s) > -a$$

### Pair 2: $-e^{-at}u(-t)$ (Left-sided exponential)

$$X(s) = -\int_{-\infty}^{0} e^{-at}e^{-st}dt = \frac{1}{s+a}, \quad \text{ROC: Re}(s) < -a$$

> **Same algebraic expression**, different ROC! This is why ROC is essential.

```
SAME X(s) = 1/(s+a), DIFFERENT SIGNALS:

Right-sided (ROC: σ > -a):    Left-sided (ROC: σ < -a):
  │                              │
  │╲                           ╱ │
  │ ╲                         ╱  │
  │  ╲╲                      ╱   │
──┼───╲╲──→ t             ──╲╲──┼──→ t
  0                           0
  x = e⁻ᵃᵗu(t)              x = -e⁻ᵃᵗu(-t)

s-plane ROC:                 s-plane ROC:
  jω ↑                       jω ↑
  ───┤──║══════→ σ          ══════║──┤──→ σ
     │  -a                       -a  │
  ROC is RIGHT of -a         ROC is LEFT of -a
```

### Pair 3: $\delta(t)$

$$X(s) = 1, \quad \text{ROC: all } s$$

### Pair 4: $u(t)$

$$X(s) = \frac{1}{s}, \quad \text{ROC: Re}(s) > 0$$

### Pair 5: $t^n e^{-at}u(t)$

$$X(s) = \frac{n!}{(s+a)^{n+1}}, \quad \text{ROC: Re}(s) > -a$$

### Pair 6: $\cos(\omega_0 t)u(t)$

$$X(s) = \frac{s}{s^2 + \omega_0^2}, \quad \text{ROC: Re}(s) > 0$$

### Pair 7: $e^{-at}\sin(\omega_0 t)u(t)$

$$X(s) = \frac{\omega_0}{(s+a)^2 + \omega_0^2}, \quad \text{ROC: Re}(s) > -a$$

---

## 7.1.5 Complete Transform Table

| $x(t)$ | $X(s)$ | ROC |
|---------|--------|-----|
| $\delta(t)$ | $1$ | All $s$ |
| $u(t)$ | $1/s$ | $\sigma > 0$ |
| $t \cdot u(t)$ | $1/s^2$ | $\sigma > 0$ |
| $t^n u(t)$ | $n!/s^{n+1}$ | $\sigma > 0$ |
| $e^{-at}u(t)$ | $1/(s+a)$ | $\sigma > -a$ |
| $te^{-at}u(t)$ | $1/(s+a)^2$ | $\sigma > -a$ |
| $\cos\omega_0 t \cdot u(t)$ | $s/(s^2+\omega_0^2)$ | $\sigma > 0$ |
| $\sin\omega_0 t \cdot u(t)$ | $\omega_0/(s^2+\omega_0^2)$ | $\sigma > 0$ |
| $e^{-at}\cos\omega_0 t \cdot u(t)$ | $(s+a)/[(s+a)^2+\omega_0^2]$ | $\sigma > -a$ |
| $e^{-at}\sin\omega_0 t \cdot u(t)$ | $\omega_0/[(s+a)^2+\omega_0^2]$ | $\sigma > -a$ |

---

## 7.1.6 Region of Convergence (ROC)

### Properties of ROC

1. **ROC is a vertical strip** in the $s$-plane (parallel to $j\omega$ axis)
2. **ROC contains no poles** of $X(s)$
3. For **right-sided** signals: ROC is to the **right** of the rightmost pole
4. For **left-sided** signals: ROC is to the **left** of the leftmost pole
5. For **two-sided** signals: ROC is a **strip** between two poles
6. For **finite-duration** signals: ROC is the entire $s$-plane (possibly excluding $s = 0$ or $s = \infty$)
7. If $X(s)$ is **rational**, ROC is bounded by poles

### ROC and Signal Type

```
RIGHT-SIDED           TWO-SIDED           LEFT-SIDED
x = 0 for t < t₀     extends both ways    x = 0 for t > t₀

  jω ↑                jω ↑                jω ↑
     │  ╔══════      ═══╗ ╔═══          ══════╗  │
     │  ║                ║ ║                   ║  │
  ───┼──╠──────→   ──═══╬═╬═══→         ══════╬──┼→
     │  ║                ║ ║                   ║  │
     │  ╔══════      ═══╝ ╚═══          ══════╝  │
     │  σ₀              p₁ p₂               σ₀   │
  RIGHT of              STRIP              LEFT of
  rightmost pole     between poles       leftmost pole
```

---

## 7.1.7 ROC Examples

### Example 1: $x(t) = e^{-2t}u(t) + e^{-3t}u(t)$

$$X(s) = \frac{1}{s+2} + \frac{1}{s+3} = \frac{2s+5}{(s+2)(s+3)}$$

Poles: $s = -2, -3$. Both terms right-sided → ROC: $\text{Re}(s) > -2$.

### Example 2: $x(t) = e^{-2t}u(t) - e^{3t}u(-t)$

$$X(s) = \frac{1}{s+2} + \frac{1}{s-3}$$

- $e^{-2t}u(t)$ requires $\sigma > -2$
- $-e^{3t}u(-t)$ requires $\sigma < 3$
- ROC: $-2 < \text{Re}(s) < 3$ (strip between poles)

```
Example 2 ROC:
  jω ↑
     │
  ═══╗ ╔═══
     ║ ║
  ───╬═╬───→ σ
     ║ ║
  ═══╝ ╚═══
   -2   3
  ROC: -2 < σ < 3
```

### Example 3: $x(t) = e^{-2t}u(t) + e^{3t}u(t)$

Both right-sided: ROC must be right of $\max(-2, 3) = 3$. ROC: $\sigma > 3$.

---

## 7.1.8 ROC ↔ Stability ↔ Causality

| Property | ROC Condition |
|----------|---------------|
| **Causal + Stable** | ROC includes $j\omega$ axis AND is right half-plane |
| **Causal** | ROC is right of rightmost pole |
| **Stable** | ROC includes $j\omega$ axis |
| **Causal + Stable** | All poles in LHP ($\text{Re}(p) < 0$) |
| **Anti-causal + Stable** | All poles in RHP |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Laplace variable | $s = \sigma + j\omega$ (complex frequency) |
| Bilateral LT | Integral from $-\infty$ to $\infty$ |
| Unilateral LT | Integral from $0^-$ to $\infty$ |
| ROC | Values of $s$ where integral converges |
| Same $X(s)$, different ROC | Different signals! ROC is essential |
| Right-sided | ROC right of rightmost pole |
| Left-sided | ROC left of leftmost pole |
| Causal + stable | All poles in LHP |
| FT from LT | $X(j\omega) = X(s)\|_{s=j\omega}$ if $j\omega \in$ ROC |

---

## Quick Revision Questions

1. **Find the Laplace Transform of $3e^{-5t}u(t)$ and its ROC.**
   - $X(s) = 3/(s+5)$, ROC: $\text{Re}(s) > -5$.

2. **Why is ROC needed in addition to $X(s)$?**
   - The same $X(s)$ can correspond to different signals depending on the ROC.

3. **What is the ROC for a causal stable system?**
   - Right half-plane including the $j\omega$ axis; all poles have $\text{Re}(p) < 0$.

4. **Find the Laplace Transform of $tu(t)$.**
   - $1/s^2$, ROC: $\text{Re}(s) > 0$.

5. **If a signal is two-sided, what shape is the ROC?**
   - A vertical strip between two pole locations.

---

[← Previous: Unit 6 — Filtering Concepts](../06-Fourier-Transform/04-filtering-concepts.md) | [Next: Laplace Properties →](02-properties.md)
