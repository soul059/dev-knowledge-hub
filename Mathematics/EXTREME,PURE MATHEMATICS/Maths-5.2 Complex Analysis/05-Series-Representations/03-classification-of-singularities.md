# 5.3 Classification of Singularities

[← Previous: Laurent Series](02-laurent-series.md) | [Next: Zeros of Analytic Functions →](04-zeros-of-analytic-functions.md)

---

## Chapter Overview

An **isolated singularity** is a point where a function fails to be analytic but is analytic nearby. The Laurent series classifies singularities into three types — removable, pole, and essential — each with dramatically different behavior. This classification is fundamental to the residue calculus.

---

## 1. Isolated Singularities

### 1.1 Definition

$z_0$ is an **isolated singularity** of $f$ if:
- $f$ is not analytic at $z_0$, but
- $f$ is analytic in some punctured disk $0 < |z - z_0| < R$.

**Non-isolated singularities** (e.g., branch points of $\sqrt{z}$, or $1/\sin(1/z)$ at $z=0$) are not covered by this classification.

---

## 2. The Three Types

The Laurent series $f(z) = \sum_{n=-\infty}^{\infty} a_n(z-z_0)^n$ determines the type:

### 2.1 Removable Singularity

**Definition:** No negative powers in the Laurent series ($a_n = 0$ for all $n < 0$).

**Equivalent conditions:**
- $\lim_{z \to z_0} f(z)$ exists (and is finite)
- $\lim_{z \to z_0} (z-z_0)f(z) = 0$
- $f$ is bounded near $z_0$

**Action:** Defining $f(z_0) = \lim_{z \to z_0} f(z) = a_0$ makes $f$ analytic at $z_0$.

**Example:** $f(z) = \frac{\sin z}{z}$ at $z = 0$.

$$\frac{\sin z}{z} = \frac{z - z^3/6 + \cdots}{z} = 1 - \frac{z^2}{6} + \cdots$$

No negative powers → removable singularity. Set $f(0) = 1$.

### 2.2 Pole of Order $m$

**Definition:** The principal part has finitely many terms, with $(z-z_0)^{-m}$ being the most negative power ($a_{-m} \neq 0$, $a_n = 0$ for $n < -m$).

$$f(z) = \frac{a_{-m}}{(z-z_0)^m} + \cdots + \frac{a_{-1}}{z-z_0} + a_0 + a_1(z-z_0) + \cdots$$

**Equivalent conditions:**
- $\lim_{z \to z_0} |f(z)| = \infty$
- $\lim_{z \to z_0} (z-z_0)^m f(z)$ exists and is nonzero
- $f(z) = g(z)/(z-z_0)^m$ with $g$ analytic and $g(z_0) \neq 0$

**Example:** $f(z) = \frac{1}{z^2(z-1)}$ at $z = 0$: pole of order 2.

### 2.3 Essential Singularity

**Definition:** The principal part has infinitely many nonzero terms.

**Equivalent condition:** $\lim_{z \to z_0} f(z)$ does not exist (not even as $\infty$).

**Example:** $e^{1/z}$ at $z = 0$:

$$e^{1/z} = 1 + \frac{1}{z} + \frac{1}{2z^2} + \frac{1}{6z^3} + \cdots$$

Infinitely many negative powers → essential singularity.

---

## 3. Classification Decision Diagram

```
Given: isolated singularity of f at z₀

                 Laurent series principal part?
                          │
            ┌─────────────┼─────────────┐
            │             │             │
         No terms     Finitely      Infinitely
       (all a_{-n}=0)   many terms    many terms
            │             │             │
            ▼             ▼             ▼
        REMOVABLE       POLE        ESSENTIAL
        SINGULARITY   of order m   SINGULARITY
            │             │             │
     lim f(z) exists  lim|f(z)|=∞   lim f(z)
     and is finite                  does not exist
```

---

## 4. Behavior Near Each Type

| Type | $|f(z)|$ as $z \to z_0$ | Image of punctured neighborhood |
|------|------------------------|--------------------------------|
| Removable | Bounded | Bounded set |
| Pole of order $m$ | $\to \infty$ like $|z-z_0|^{-m}$ | $\mathbb{C} \setminus$ bounded set |
| Essential | Oscillates wildly | Dense in $\mathbb{C}$ (Casorati–Weierstrass) |

### 4.1 Casorati–Weierstrass Theorem

If $z_0$ is an essential singularity of $f$, then for any $w \in \mathbb{C}$ and any $\epsilon > 0$, the equation $|f(z) - w| < \epsilon$ has solutions arbitrarily close to $z_0$.

In other words: $f$ comes arbitrarily close to *every* complex value near an essential singularity.

### 4.2 Picard's Great Theorem (stated without proof)

Near an essential singularity, $f$ assumes every complex value with at most one exception, infinitely often.

**Example:** $e^{1/z}$ near $z = 0$ takes every nonzero complex value infinitely often. The only exception is $w = 0$.

---

## 5. Singularities at Infinity

Define $f$ to have a singularity at $\infty$ by studying $g(w) = f(1/w)$ at $w = 0$.

| $f$ at $\infty$ | $g(w) = f(1/w)$ at $w = 0$ | Example |
|-----------------|---------------------------|---------|
| Removable | Removable | $\frac{1}{z}$ |
| Pole of order $m$ | Pole of order $m$ | $z^m$ (polynomial) |
| Essential | Essential | $e^z$ |

---

## 6. Design Examples

### Example 1: Classify all singularities of $f(z) = \frac{e^z - 1}{z^2}$.

**At $z = 0$:**

$$e^z - 1 = z + \frac{z^2}{2} + \frac{z^3}{6} + \cdots$$

$$f(z) = \frac{1}{z} + \frac{1}{2} + \frac{z}{6} + \cdots$$

Principal part: $\frac{1}{z}$ → **Pole of order 1** (simple pole). Residue $= 1$.

**At $z = \infty$:** $g(w) = f(1/w) = \frac{e^{1/w} - 1}{1/w^2} = w^2(e^{1/w} - 1)$

$e^{1/w} - 1 = \frac{1}{w} + \frac{1}{2w^2} + \cdots$, so $g(w) = w + \frac{1}{2} + \frac{1}{6w} + \cdots$

Essential singularity at $w = 0$ → **Essential singularity at $\infty$**.

### Example 2: Classify singularities of $f(z) = \frac{z^3}{(z-1)^2(z+2)}$.

| Point | Analysis | Type |
|-------|----------|------|
| $z = 1$ | $(z-1)^2$ in denominator, numerator $\neq 0$ at $z=1$ | Pole of order 2 |
| $z = -2$ | $(z+2)$ in denominator, numerator $\neq 0$ at $z=-2$ | Simple pole |
| $z = \infty$ | $f(z) \sim z^3/z^3 = 1$ for large $z$ → bounded | Removable |

### Example 3: Show $z = 0$ is essential for $\sin(1/z)$.

$$\sin(1/z) = \frac{1}{z} - \frac{1}{6z^3} + \frac{1}{120z^5} - \cdots$$

Infinitely many negative powers → essential singularity. ✓

Also: $\sin(1/z)$ takes values $0, \pm 1, \pm 1/2, \ldots$ at $z = 1/(n\pi)$ for $n = 1, 2, \ldots$, which accumulate at $z = 0$.

---

## 7. Real-World Applications

| Application | Singularity Role |
|-------------|-----------------|
| **Control theory** | Poles of transfer function = system modes |
| **Signal processing** | Pole-zero analysis of filters |
| **Quantum mechanics** | Essential singularities in scattering amplitudes |
| **Fluid dynamics** | Poles = point vortices; essential singularities = chaotic flow |

---

## Summary Table

| Type | Principal Part | $\lim_{z\to z_0} f(z)$ | Example |
|------|---------------|------------------------|---------|
| Removable | None | Finite | $\sin z/z$ at $0$ |
| Pole of order $m$ | $\frac{a_{-m}}{(z-z_0)^m} + \cdots + \frac{a_{-1}}{z-z_0}$ | $\infty$ | $1/z^m$ at $0$ |
| Essential | $\sum_{n=1}^{\infty} a_{-n}(z-z_0)^{-n}$ | Does not exist | $e^{1/z}$ at $0$ |

---

## Quick Revision Questions

1. **Define** the three types of isolated singularities in terms of the Laurent series.

2. **Classify** the singularities of $\frac{1}{z(e^z - 1)}$.

3. **State** the Casorati–Weierstrass theorem and illustrate with $e^{1/z}$.

4. **Determine** the order of the pole of $\frac{\sin z}{z^5}$ at $z = 0$.

5. **Explain** why $\cos(1/z)$ has an essential singularity at $z = 0$.

6. **Classify** all singularities (including at $\infty$) of $\frac{z^2}{z^2+1}$.

---

[← Previous: Laurent Series](02-laurent-series.md) | [Next: Zeros of Analytic Functions →](04-zeros-of-analytic-functions.md)
