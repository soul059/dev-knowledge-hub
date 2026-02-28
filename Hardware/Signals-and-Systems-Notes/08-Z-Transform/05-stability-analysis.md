# Chapter 8.5: Stability and Causality Analysis

## Overview

For discrete-time LTI systems, stability and causality are determined by the pole locations of $H(z)$ relative to the **unit circle** $|z| = 1$. This chapter covers the unit circle criterion, the Jury stability test (discrete counterpart of Routh-Hurwitz), and the mapping between $s$-plane and $z$-plane stability regions.

---

## 8.5.1 Stability and Causality Conditions

### BIBO Stability

$$\sum_{n=-\infty}^{\infty} |h[n]| < \infty \quad \Leftrightarrow \quad \text{ROC of } H(z) \text{ includes the unit circle}$$

### Causality

$$h[n] = 0 \text{ for } n < 0 \quad \Leftrightarrow \quad \text{ROC is exterior of outermost pole}$$

### Causal + Stable

$$\text{All poles of } H(z) \text{ strictly inside the unit circle: } |p_i| < 1$$

```
STABILITY REGIONS IN z-PLANE:

        ╱──╲
      ╱      ╲  ← Unit circle |z| = 1
     │  STABLE │
     │  region │    UNSTABLE
     │    ○    │    region
     │  × ×   │
      ╲      ╱
        ╲──╱

  × inside → stable pole
  × on circle → marginally stable (simple)
                   unstable (repeated)
  × outside → unstable pole
```

---

## 8.5.2 Stability Classification

| Condition | Pole Location | Behavior |
|-----------|---------------|----------|
| **Asymptotically stable** | All $\|p_i\| < 1$ | Output decays to zero |
| **Marginally stable** | Simple poles with $\|p_i\| = 1$, rest inside | Bounded but non-decaying oscillation |
| **Unstable** | Any $\|p_i\| > 1$ OR repeated poles on $\|z\| = 1$ | Output grows without bound |

### Pole Location → Impulse Response Shape

```
POLE AT z = p → h[n] includes pⁿ term:

|p| < 1 (inside UC):        |p| = 1 (on UC):         |p| > 1 (outside UC):
  h[n]                       h[n]                      h[n]
  │╲                         │ │ │ │ │                 │          ╱
  │ ╲                        │ │ │ │ │                 │        ╱
  │  ╲╲                      │ │ │ │ │                 │      ╱
  │   ╲╲╲───→               │ │ │ │ │                 │    ╱
  └────────→ n               └──────────→ n            └──────→ n
  DECAYING                   CONSTANT                  GROWING
  (stable)                   (marginally stable)       (unstable)

Complex pole pair at re^(±jθ):
  h[n]
  │  ╱╲
  │ ╱  ╲  ╱╲
  │╱    ╲╱  ╲╱╲─── → damped sinusoid (r < 1)
  └──────────────→ n
```

---

## 8.5.3 Jury Stability Test

The **Jury test** determines whether all roots of a polynomial $D(z) = a_0 z^N + a_1 z^{N-1} + \cdots + a_N$ lie inside the unit circle, without finding the roots.

### Necessary Conditions (Quick Checks)

For $D(z) = a_0 z^N + a_1 z^{N-1} + \cdots + a_N$ with $a_0 > 0$:

1. $D(1) > 0$
2. $(-1)^N D(-1) > 0$
3. $|a_N| < a_0$

If any condition fails → **unstable**.

### Jury Array

Construct rows from the polynomial coefficients:

| Row | Elements |
|-----|----------|
| 1 | $a_N, a_{N-1}, \ldots, a_1, a_0$ |
| 2 | $a_0, a_1, \ldots, a_{N-1}, a_N$ |
| 3 | $b_0, b_1, \ldots, b_{N-1}$ |
| 4 | $b_{N-1}, \ldots, b_1, b_0$ |
| $\vdots$ | Continue until 1 element |

where $b_k = \begin{vmatrix} a_N & a_{N-1-k} \\ a_0 & a_{1+k} \end{vmatrix} = a_N a_{1+k} - a_0 a_{N-1-k}$

**Stability**: All first-column elements of odd rows must be positive (for $a_0 > 0$).

### Example: Second-Order System

$$D(z) = z^2 + a_1 z + a_2$$

Necessary and sufficient conditions:

$$\boxed{D(1) = 1 + a_1 + a_2 > 0}$$

$$\boxed{D(-1) = 1 - a_1 + a_2 > 0}$$

$$\boxed{|a_2| < 1}$$

```
STABILITY TRIANGLE (2nd order):

  a₁ ↑
   2 │
     │╲
     │  ╲  a₂ = a₁ - 1
   0 ├────╲────────
     │     ╲      │
     │STABLE╲     │
     │  region╲   │
  -2 ├────────╲───┤
     └──────────┴──→ a₂
    -1    0     1

  Bounded by:
  1 + a₁ + a₂ > 0  (left line)
  1 - a₁ + a₂ > 0  (right line)
  |a₂| < 1          (top/bottom)
```

---

## 8.5.4 Examples

### Example 1: Check Stability

$$H(z) = \frac{1}{z^2 - 0.5z - 0.06}$$

Roots: $z = \frac{0.5 \pm \sqrt{0.25 + 0.24}}{2} = \frac{0.5 \pm 0.7}{2}$

$z_1 = 0.6$, $z_2 = -0.1$. Both $|z| < 1$ → **STABLE** ✓

**Quick check**: $D(1) = 1-0.5-0.06 = 0.44 > 0$ ✓, $D(-1) = 1+0.5-0.06 = 1.44 > 0$ ✓, $|a_2| = |-0.06| = 0.06 < 1$ ✓

### Example 2: Unstable System

$$H(z) = \frac{z}{z^2 - 1.6z + 0.8}$$

Poles: $z = \frac{1.6 \pm \sqrt{2.56 - 3.2}}{2} = \frac{1.6 \pm j0.8}{2} = 0.8 \pm j0.4$

$|p| = \sqrt{0.64 + 0.16} = \sqrt{0.8} = 0.894 < 1$ → **STABLE** ✓

### Example 3: Marginally Stable

$$H(z) = \frac{1}{z^2 + 1}$$

Poles at $z = \pm j$ → $|p| = 1$ (on unit circle), simple poles → **marginally stable**.

---

## 8.5.5 Relationship: $s$-Plane ↔ $z$-Plane

With $z = e^{sT}$ (sampling period $T$):

| $s$-Plane Region | $z$-Plane Region | Stability |
|-------------------|-------------------|-----------|
| Left half-plane ($\sigma < 0$) | Inside unit circle ($\|z\| < 1$) | Stable |
| $j\omega$ axis ($\sigma = 0$) | Unit circle ($\|z\| = 1$) | Marginal |
| Right half-plane ($\sigma > 0$) | Outside unit circle ($\|z\| > 1$) | Unstable |
| Origin ($s = 0$) | $z = 1$ | DC |
| $s = j\pi/T$ | $z = -1$ | Nyquist frequency |

```
MAPPING: s-plane → z-plane (z = e^(sT))

s-plane:                    z-plane:
  jω ↑                       Im ↑
  jπ/T ─┤                         ╱─╲
     │ STABLE                    ╱ S  ╲
     │ (LHP)                   ╱──────╲
  ── ┼ ────→ σ            ──(─1)──0──(1)→ Re
     │                       ╲──────╱
     │                         ╲ S ╱
  -jπ/T ─┤                      ╲─╱
     
  LHP → inside UC    S = stable region
  jω axis → UC
  Horizontal lines → circles
  Vertical lines → radial lines
```

---

## 8.5.6 Stabilizing Unstable Systems

### Pole Placement via Feedback

If $H(z)$ has an unstable pole, use feedback to move poles inside the unit circle:

$$T(z) = \frac{H(z)}{1 + H(z)G(z)}$$

Choose $G(z)$ such that the poles of $T(z)$ (roots of $1 + H(z)G(z) = 0$) are inside $|z| = 1$.

### Example

$$H(z) = \frac{1}{z - 1.5} \quad (\text{unstable: pole at } z = 1.5)$$

With proportional feedback $G(z) = K$:

$$T(z) = \frac{1/(z-1.5)}{1 + K/(z-1.5)} = \frac{1}{z - 1.5 + K} = \frac{1}{z - (1.5-K)}$$

For stability: $|1.5 - K| < 1 \Rightarrow 0.5 < K < 2.5$

---

## 8.5.7 Comparison: Continuous vs Discrete Stability

| Aspect | Continuous ($s$-domain) | Discrete ($z$-domain) |
|--------|------------------------|----------------------|
| Stability boundary | $j\omega$ axis | Unit circle |
| Stable region | LHP: $\text{Re}(s) < 0$ | Inside: $\|z\| < 1$ |
| Stability test | Routh-Hurwitz | Jury test |
| Marginal | Simple poles on $j\omega$ | Simple poles on UC |
| DC response | $H(0)$ | $H(1)$ |
| Mapping | — | $z = e^{sT}$ |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| BIBO stability | ROC includes unit circle |
| Causal + stable | All poles $\|p_i\| < 1$ |
| Jury test | Algebraic check: $D(1)>0$, $(-1)^N D(-1)>0$, $\|a_N\| < a_0$ |
| 2nd-order stability | Triangle: $1+a_1+a_2>0$, $1-a_1+a_2>0$, $\|a_2\|<1$ |
| $s \to z$ map | $z = e^{sT}$; LHP → inside UC |
| Marginal stability | Simple poles on $\|z\|=1$ |

---

## Quick Revision Questions

1. **A DT system has poles at $z = 0.7 \pm 0.6j$. Is it stable?**
   - $|p| = \sqrt{0.49+0.36} = \sqrt{0.85} = 0.922 < 1$. Yes, **stable**.

2. **What are the Jury necessary conditions for $D(z) = z^3 - 0.5z^2 + 0.2z - 0.1$?**
   - $D(1) = 1-0.5+0.2-0.1 = 0.6 > 0$ ✓. $-D(-1) = -(-1-0.5-0.2-0.1) = 1.8 > 0$ ✓. $|a_3|/a_0 = 0.1 < 1$ ✓. Necessary conditions met.

3. **A pole at $z = 1$ means what for stability?**
   - Marginally stable if simple; unstable if repeated. Response includes a constant (like DC offset).

4. **How does the Routh-Hurwitz test relate to the Jury test?**
   - Routh-Hurwitz checks for roots in the LHP ($s$-plane). Jury checks for roots inside the unit circle ($z$-plane). They serve the same purpose for their respective domains.

5. **For second-order $D(z) = z^2 + 0.6z + 0.5$, check stability.**
   - $D(1) = 2.1 > 0$ ✓. $D(-1) = 1-0.6+0.5 = 0.9 > 0$ ✓. $|a_2| = 0.5 < 1$ ✓. **Stable**.

---

[← Previous: Transfer Function](04-transfer-function.md) | [Next: Unit 9 — Sampling →](../09-Sampling-and-Reconstruction/README.md)
