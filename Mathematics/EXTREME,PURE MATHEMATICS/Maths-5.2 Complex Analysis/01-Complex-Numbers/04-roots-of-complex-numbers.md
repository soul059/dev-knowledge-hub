# 1.4 Roots of Complex Numbers

[← Previous: De Moivre's Theorem](03-de-moivres-theorem.md) | [Next: Extended Complex Plane →](05-extended-complex-plane.md)

---

## Chapter Overview

Every nonzero complex number has exactly $n$ distinct $n$th roots. These roots are evenly spaced on a circle in the complex plane, forming a beautiful geometric pattern. This chapter develops the theory and provides explicit formulas using De Moivre's theorem.

---

## 1. The $n$th Root Problem

### 1.1 Problem Statement

Given $w \in \mathbb{C}$, $w \ne 0$, and $n \in \mathbb{Z}^+$, find all $z$ such that:

$$z^n = w$$

### 1.2 Fundamental Theorem

Every nonzero complex number $w$ has exactly **$n$ distinct** $n$th roots.

> This is a consequence of the **Fundamental Theorem of Algebra** — the polynomial $z^n - w = 0$ has exactly $n$ roots (counting multiplicity) in $\mathbb{C}$.

---

## 2. Formula for $n$th Roots

### 2.1 Derivation

Let $w = R\,e^{i\phi}$ where $R = |w|$ and $\phi = \arg(w)$.

We seek $z = r\,e^{i\theta}$ such that $z^n = w$:

$$r^n e^{in\theta} = R\,e^{i\phi}$$

Comparing moduli and arguments:

$$r^n = R \implies r = R^{1/n} = \sqrt[n]{R}$$

$$n\theta = \phi + 2k\pi \implies \theta = \frac{\phi + 2k\pi}{n}$$

### 2.2 The General Formula

$$\boxed{z_k = R^{1/n} \exp\!\left(i\,\frac{\phi + 2k\pi}{n}\right), \qquad k = 0, 1, 2, \ldots, n-1}$$

Or equivalently:

$$z_k = \sqrt[n]{R}\left(\cos\frac{\phi + 2k\pi}{n} + i\sin\frac{\phi + 2k\pi}{n}\right)$$

### 2.3 Why Only $n$ Distinct Roots?

For $k = n$, we get $\theta = \frac{\phi + 2n\pi}{n} = \frac{\phi}{n} + 2\pi$, which produces the same point as $k = 0$. Similarly, $k = n+1$ repeats $k = 1$, etc.

---

## 3. Geometric Interpretation

### 3.1 Roots on a Circle

All $n$th roots of $w$ lie on a circle of radius $R^{1/n}$ centered at the origin, equally spaced at angular intervals of $\frac{2\pi}{n}$.

```
Cube Roots of 8 (n=3, R=8, φ=0):
r = 8^(1/3) = 2,  angles: 0, 2π/3, 4π/3

              Im
               ▲
               │
        z₁ •   │            z₁ = 2e^(i·2π/3) = -1 + i√3
            ╲  │
             ╲ │
              ╲│
    ───────────O──────• z₀ ──► Re     z₀ = 2e^(i·0) = 2
              ╱│
             ╱ │
            ╱  │
        z₂ •   │            z₂ = 2e^(i·4π/3) = -1 - i√3
               │

    All roots on circle of radius 2
    Separated by 120° = 2π/3
```

```
Fourth Roots of 1 (n=4):

              Im
               ▲
               │
               • z₁ = i
               │
               │
    ─── z₂ •──O──• z₀ ──► Re
       -1      │      1
               │
               • z₃ = -i
               │

    z₀ = 1, z₁ = i, z₂ = -1, z₃ = -i
    Separated by 90° = π/2
```

### 3.2 The "Star" Pattern of Roots

```
Fifth Roots of 1 (regular pentagon):

                   z₁
                  •
               ╱     ╲
             ╱         ╲
    z₂  •                • z₀ = 1
          ╲             ╱
            ╲         ╱
              •─────•
            z₃       z₄

    Vertices of a regular pentagon inscribed in the unit circle
```

---

## 4. Roots of Unity

### 4.1 Definition

The $n$th **roots of unity** are the solutions of $z^n = 1$:

$$\omega_k = e^{2\pi i k/n}, \qquad k = 0, 1, \ldots, n-1$$

### 4.2 Primitive Root

The root $\omega = e^{2\pi i/n}$ is called a **primitive $n$th root of unity**. All other roots are powers of $\omega$:

$$\omega_k = \omega^k$$

### 4.3 Properties of Roots of Unity

| Property | Formula |
|----------|---------|
| Sum of all roots | $\displaystyle\sum_{k=0}^{n-1} \omega^k = 0$ |
| Product of all roots | $\displaystyle\prod_{k=0}^{n-1} \omega^k = (-1)^{n+1}$ |
| Cyclotomic polynomial | $z^n - 1 = \displaystyle\prod_{k=0}^{n-1}(z - \omega^k)$ |
| $\omega^n = 1$ | Periodicity |
| $|\omega^k| = 1$ | All on unit circle |
| $\overline{\omega^k} = \omega^{n-k}$ | Conjugate pairs |

### 4.4 Sum of Roots of Unity — Proof

$$S = \sum_{k=0}^{n-1} \omega^k = 1 + \omega + \omega^2 + \cdots + \omega^{n-1}$$

This is a geometric series with ratio $\omega \ne 1$:

$$S = \frac{1 - \omega^n}{1 - \omega} = \frac{1 - 1}{1 - \omega} = 0$$

### 4.5 Table of Low-Order Roots of Unity

| $n$ | Roots | Primitive root $\omega$ |
|-----|-------|------------------------|
| 2 | $1, -1$ | $-1$ |
| 3 | $1, e^{2\pi i/3}, e^{4\pi i/3}$ | $e^{2\pi i/3} = \frac{-1+i\sqrt{3}}{2}$ |
| 4 | $1, i, -1, -i$ | $i$ |
| 6 | $1, e^{i\pi/3}, e^{2i\pi/3}, -1, e^{4i\pi/3}, e^{5i\pi/3}$ | $e^{i\pi/3} = \frac{1+i\sqrt{3}}{2}$ |

---

## 5. General $n$th Roots via Roots of Unity

### 5.1 Key Observation

If $z_0$ is **one** particular $n$th root of $w$, then **all** $n$th roots of $w$ are:

$$z_k = z_0 \cdot \omega^k, \qquad k = 0, 1, \ldots, n-1$$

where $\omega = e^{2\pi i/n}$ is the primitive $n$th root of unity.

> The $n$th roots of $w$ are obtained by multiplying one root by all $n$th roots of unity.

---

## 6. Design Examples

### Example 1: Find all cube roots of $-8i$.

**Step 1.** Write $w = -8i$ in polar form:
$$|w| = 8, \quad \arg(w) = -\frac{\pi}{2}$$
$$w = 8\,e^{-i\pi/2}$$

**Step 2.** Apply the root formula with $n = 3$:
$$z_k = 8^{1/3}\exp\!\left(i\,\frac{-\pi/2 + 2k\pi}{3}\right) = 2\exp\!\left(i\,\frac{-\pi/2 + 2k\pi}{3}\right)$$

**Step 3.** Compute for $k = 0, 1, 2$:

| $k$ | Angle $\theta_k$ | $z_k$ |
|-----|-------------------|--------|
| 0 | $-\pi/6$ | $2e^{-i\pi/6} = \sqrt{3} - i$ |
| 1 | $-\pi/6 + 2\pi/3 = \pi/2$ | $2e^{i\pi/2} = 2i$ |
| 2 | $-\pi/6 + 4\pi/3 = 7\pi/6$ | $2e^{i7\pi/6} = -\sqrt{3} + i$ ... wait |

Let me recompute $k=2$: $\theta_2 = \frac{-\pi/2 + 4\pi}{3} = \frac{7\pi/2}{3} = \frac{7\pi}{6}$

$z_2 = 2(\cos(7\pi/6) + i\sin(7\pi/6)) = 2(-\frac{\sqrt{3}}{2} - \frac{1}{2}i) = -\sqrt{3} - i$

**Answer:**
$$z_0 = \sqrt{3} - i, \quad z_1 = 2i, \quad z_2 = -\sqrt{3} - i$$

**Verification**: $(2i)^3 = 8i^3 = -8i$ ✓

### Example 2: Solve $z^4 = -16$.

**Step 1.** $w = -16 = 16\,e^{i\pi}$ (modulus $16$, argument $\pi$)

**Step 2.** $z_k = 16^{1/4}\exp\!\left(i\,\frac{\pi + 2k\pi}{4}\right) = 2\exp\!\left(i\,\frac{(2k+1)\pi}{4}\right)$

**Step 3.** Compute:

| $k$ | Angle | $z_k$ (Cartesian) |
|-----|-------|--------------------|
| 0 | $\pi/4$ | $\sqrt{2} + i\sqrt{2}$ |
| 1 | $3\pi/4$ | $-\sqrt{2} + i\sqrt{2}$ |
| 2 | $5\pi/4$ | $-\sqrt{2} - i\sqrt{2}$ |
| 3 | $7\pi/4$ | $\sqrt{2} - i\sqrt{2}$ |

```
        Im
         ▲
         │
  z₁ •   │   • z₀
      ╲   │  ╱
       ╲  │ ╱
        ╲ │╱         Radius = 2
    ─────O──────────► Re
        ╱ │╲
       ╱  │ ╲
      ╱   │  ╲
  z₂ •   │   • z₃

  Four roots at ±√2 ± i√2, equally spaced
```

---

## 7. Cyclotomic Polynomials

### 7.1 Factorization of $z^n - 1$

$$z^n - 1 = \prod_{k=0}^{n-1}(z - e^{2\pi ik/n})$$

### 7.2 Useful Factorizations

| Expression | Factorization |
|------------|---------------|
| $z^2 - 1$ | $(z-1)(z+1)$ |
| $z^3 - 1$ | $(z-1)(z^2 + z + 1)$ |
| $z^4 - 1$ | $(z-1)(z+1)(z^2+1)$ |
| $z^6 - 1$ | $(z-1)(z+1)(z^2+z+1)(z^2-z+1)$ |
| $z^n - 1$ | $(z-1)(z^{n-1} + z^{n-2} + \cdots + z + 1)$ |

---

## 8. Real-World Applications

| Application | Description |
|-------------|-------------|
| **Discrete Fourier Transform** | The DFT matrix uses $n$th roots of unity as its entries |
| **Signal Processing** | FFT algorithm exploits symmetry of roots of unity |
| **Crystallography** | $n$-fold rotational symmetry described by $n$th roots of unity |
| **Filter Design** | Poles/zeros of digital filters placed at roots of unity |

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| $n$th roots of $w$ | $z_k = R^{1/n} e^{i(\phi + 2k\pi)/n}$, $k = 0, \ldots, n-1$ |
| Number of distinct roots | Exactly $n$ |
| Geometric arrangement | Regular $n$-gon on circle of radius $R^{1/n}$ |
| Primitive root of unity | $\omega = e^{2\pi i/n}$ |
| All roots of unity | $1, \omega, \omega^2, \ldots, \omega^{n-1}$ |
| Sum of roots of unity | $0$ |
| General roots from one | $z_k = z_0 \cdot \omega^k$ |
| $z^n - 1$ factorization | $\prod_{k=0}^{n-1}(z - \omega^k)$ |

---

## Quick Revision Questions

1. **Find** all fourth roots of $-1$ in both polar and Cartesian form.

2. **Show** that the sum of the $n$th roots of unity equals zero using the geometric series formula.

3. **Find** all cube roots of $1 + i$ and sketch them in the complex plane.

4. **Prove** that $|z_k| = R^{1/n}$ for every $n$th root $z_k$ of $w$ (i.e., all roots have the same modulus).

5. **Factor** $z^6 - 1$ completely over $\mathbb{R}$ and over $\mathbb{C}$.

6. **Explain** geometrically why the $n$th roots of any complex number form a regular $n$-gon.

---

[← Previous: De Moivre's Theorem](03-de-moivres-theorem.md) | [Next: Extended Complex Plane →](05-extended-complex-plane.md)
