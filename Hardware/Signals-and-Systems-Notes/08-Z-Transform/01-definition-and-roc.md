# Chapter 8.1: Z-Transform — Definition and Region of Convergence

## Overview

The Z-Transform maps a discrete-time sequence $x[n]$ to a function of the complex variable $z$, providing the same algebraic convenience for difference equations that the Laplace Transform provides for differential equations. The **Region of Convergence (ROC)** is critical for uniquely identifying the underlying signal.

---

## 8.1.1 Bilateral Z-Transform Definition

$$X(z) = \sum_{n=-\infty}^{\infty} x[n]\,z^{-n}, \quad z \in \mathbb{C}$$

where $z = re^{j\omega}$, $r = |z|$, $\omega = \angle z$.

---

## 8.1.2 Unilateral (One-Sided) Z-Transform

$$X(z) = \sum_{n=0}^{\infty} x[n]\,z^{-n}$$

- Assumes $x[n] = 0$ for $n < 0$ or ignores negative-time terms
- Handles **initial conditions** in difference equations

---

## 8.1.3 Relation to Other Transforms

### Z-Transform ↔ Laplace Transform

With sampling period $T$: $z = e^{sT}$

$$X(z) = X_s(s)\big|_{s = \frac{1}{T}\ln z}$$

### Z-Transform ↔ DTFT

$$X(e^{j\omega}) = X(z)\big|_{z = e^{j\omega}} \quad \text{(if unit circle is in ROC)}$$

```
z-PLANE vs s-PLANE MAPPING (z = eˢᵀ):

s-plane:                  z-plane:
  jω ↑                     Im ↑
     │  LHP    RHP              ╱ ╲
     │  │      │            ╱   │   ╲
  ───┼──┼──────┼──→ σ    ╱─────┼─────╲──→ Re
     │  │      │         │     0 1    │
     │  │      │         ╲     │     ╱
                           ╲   │   ╱
  LHP → inside |z|=1          ╲ ╱
  RHP → outside |z|=1
  jω axis → unit circle
```

---

## 8.1.4 Common Z-Transform Pairs

### Pair 1: $\delta[n]$

$$X(z) = \sum_{n} \delta[n]\,z^{-n} = 1, \quad \text{ROC: all } z$$

### Pair 2: $u[n]$ (Unit step)

$$X(z) = \sum_{n=0}^{\infty} z^{-n} = \frac{1}{1 - z^{-1}} = \frac{z}{z-1}, \quad \text{ROC: } |z| > 1$$

### Pair 3: $a^n u[n]$

$$X(z) = \sum_{n=0}^{\infty} a^n z^{-n} = \sum_{n=0}^{\infty} (a/z)^n = \frac{1}{1-az^{-1}} = \frac{z}{z-a}, \quad \text{ROC: } |z| > |a|$$

### Pair 4: $-a^n u[-n-1]$ (Left-sided)

$$X(z) = \frac{1}{1-az^{-1}} = \frac{z}{z-a}, \quad \text{ROC: } |z| < |a|$$

> Same algebraic form as Pair 3 — only ROC differs! (Same principle as Laplace.)

### Pair 5: $n \cdot a^n u[n]$

$$X(z) = \frac{az^{-1}}{(1-az^{-1})^2} = \frac{az}{(z-a)^2}, \quad \text{ROC: } |z| > |a|$$

### Pair 6: $\cos(\omega_0 n)u[n]$

$$X(z) = \frac{1 - z^{-1}\cos\omega_0}{1 - 2z^{-1}\cos\omega_0 + z^{-2}}, \quad \text{ROC: } |z| > 1$$

### Pair 7: $a^n\sin(\omega_0 n)u[n]$

$$X(z) = \frac{az^{-1}\sin\omega_0}{1 - 2az^{-1}\cos\omega_0 + a^2z^{-2}}, \quad \text{ROC: } |z| > |a|$$

---

## 8.1.5 Complete Transform Table

| $x[n]$ | $X(z)$ | ROC |
|--------|--------|-----|
| $\delta[n]$ | $1$ | All $z$ |
| $\delta[n-k]$ | $z^{-k}$ | $z \neq 0$ (if $k>0$) |
| $u[n]$ | $z/(z-1)$ | $\|z\| > 1$ |
| $a^n u[n]$ | $z/(z-a)$ | $\|z\| > \|a\|$ |
| $-a^n u[-n-1]$ | $z/(z-a)$ | $\|z\| < \|a\|$ |
| $na^n u[n]$ | $az/(z-a)^2$ | $\|z\| > \|a\|$ |
| $n^2 a^n u[n]$ | $az(z+a)/(z-a)^3$ | $\|z\| > \|a\|$ |
| $\cos(\omega_0 n)u[n]$ | $\frac{z(z-\cos\omega_0)}{z^2 - 2z\cos\omega_0+1}$ | $\|z\|>1$ |
| $\sin(\omega_0 n)u[n]$ | $\frac{z\sin\omega_0}{z^2 - 2z\cos\omega_0+1}$ | $\|z\|>1$ |
| $a^n\cos(\omega_0 n)u[n]$ | $\frac{z(z-a\cos\omega_0)}{z^2-2az\cos\omega_0+a^2}$ | $\|z\|>\|a\|$ |

---

## 8.1.6 Region of Convergence Properties

### ROC Properties

1. ROC is a **ring** or **disk** in the $z$-plane (centered at origin)
2. ROC **contains no poles**
3. For **right-sided** sequences ($x[n] = 0, n < N_1$): ROC is **outside** the outermost pole
4. For **left-sided** sequences ($x[n] = 0, n > N_2$): ROC is **inside** the innermost pole
5. For **two-sided** sequences: ROC is a **ring** between two poles
6. For **finite-duration** sequences: ROC is all $z$ (possibly excluding $z = 0$ and/or $z = \infty$)

```
ROC SHAPES IN z-PLANE:

RIGHT-SIDED              TWO-SIDED               LEFT-SIDED
(|z| > r₁)              (r₁ < |z| < r₂)        (|z| < r₂)

    ╱──╲                     ╱──╲                    ╱──╲
  ╱ ╱──╲ ╲                ╱  ╱──╲  ╲              ╱??????╲
 │ │ ╳  │ │shaded        │ │ ╳  │ │shaded        │ ╳?????│
 │ │    │ │= ROC          │ │    │ │= ROC          │??????│
  ╲ ╲──╱ ╱                ╲  ╲──╱  ╱              ╲??????╱
    ╲──╱                     ╲──╱                    ╲──╱
  Outside                  Ring between            Inside
  outermost pole           two pole radii          innermost pole
```

---

## 8.1.7 ROC Examples

### Example 1: Right-sided

$$x[n] = (0.5)^n u[n] + (0.8)^n u[n]$$

$$X(z) = \frac{z}{z-0.5} + \frac{z}{z-0.8}$$

Poles at $z = 0.5, 0.8$. Both right-sided → ROC: $|z| > 0.8$.

### Example 2: Two-sided

$$x[n] = (0.5)^n u[n] - (0.8)^n u[-n-1]$$

- $(0.5)^n u[n]$ requires $|z| > 0.5$
- $-(0.8)^n u[-n-1]$ requires $|z| < 0.8$
- ROC: $0.5 < |z| < 0.8$ (ring)

### Example 3: No ROC exists

$$x[n] = (0.5)^n u[n] + (2)^n u[-n-1]$$

Requires both $|z| > 0.5$ and $|z| < 2$ → ROC exists: $0.5 < |z| < 2$ ✓

$$x[n] = (2)^n u[n] + (0.5)^n u[-n-1]$$

Requires $|z| > 2$ and $|z| < 0.5$ → empty intersection → ZT does **not** exist.

---

## 8.1.8 ROC ↔ Stability ↔ Causality

| Property | ROC Condition |
|----------|---------------|
| **Causal** | ROC is outside the outermost pole (includes $z = \infty$) |
| **Stable (BIBO)** | ROC includes the **unit circle** $\|z\| = 1$ |
| **Causal + Stable** | All poles **inside** the unit circle ($\|p_i\| < 1$) |
| **Anti-causal** | ROC includes $z = 0$ |

```
CAUSAL + STABLE: All poles inside unit circle

z-plane:
       ╱──╲
     ╱      ╲
    │  × ×   │  ← poles inside
    │    ○    │  ← unit circle
    │  ×     │
     ╲      ╱
       ╲──╱
  ROC: |z| > max|pᵢ|
  Unit circle in ROC ✓ → STABLE
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Z-Transform variable | $z = re^{j\omega}$ (complex) |
| Bilateral ZT | Sum from $-\infty$ to $\infty$ |
| Unilateral ZT | Sum from $0$ to $\infty$ |
| ROC shape | Ring/disk centered at origin |
| Same $X(z)$, different ROC | Different sequences |
| Right-sided | ROC outside outermost pole |
| Causal + stable | All poles inside unit circle |
| ZT ↔ LT | $z = e^{sT}$ |
| ZT → DTFT | Evaluate on unit circle |

---

## Quick Revision Questions

1. **Find the Z-Transform of $x[n] = 3(0.5)^n u[n]$ and its ROC.**
   - $X(z) = 3z/(z-0.5)$, ROC: $|z| > 0.5$.

2. **Why can the same $X(z)$ represent different signals?**
   - Different ROCs correspond to different signal types (right-sided vs left-sided).

3. **What is the Z-Transform of $\delta[n-3]$?**
   - $z^{-3}$, ROC: all $z$ except $z = 0$.

4. **For a causal stable DT system, where must all poles lie?**
   - Strictly inside the unit circle: $|p_i| < 1$.

5. **How do you obtain the DTFT from the Z-Transform?**
   - Set $z = e^{j\omega}$: $X(e^{j\omega}) = X(z)|_{z=e^{j\omega}}$, valid if unit circle is in ROC.

6. **Does the ZT of $(2)^n u[n] + (3)^n u[-n-1]$ exist?**
   - Requires $|z| > 2$ and $|z| < 3$ → ROC: $2 < |z| < 3$. Yes, it exists.

---

[← Previous: Unit 7 — System Analysis](../07-Laplace-Transform/05-system-analysis.md) | [Next: Z-Transform Properties →](02-properties.md)
