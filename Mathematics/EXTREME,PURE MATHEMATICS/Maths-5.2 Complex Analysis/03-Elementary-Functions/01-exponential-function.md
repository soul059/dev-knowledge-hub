# 3.1 The Complex Exponential Function

[← Previous: Harmonic Functions](../02-Complex-Functions/06-harmonic-functions.md) | [Next: Trigonometric Functions →](02-trigonometric-functions.md)

---

## Chapter Overview

The exponential function $e^z$ is the cornerstone of complex analysis. Its definition extends naturally from the real exponential via the Taylor series, and it connects to trigonometric and hyperbolic functions through Euler's formula. Unlike the real exponential, $e^z$ is **periodic** in $\mathbb{C}$ and is **surjective** onto $\mathbb{C} \setminus \{0\}$.

---

## 1. Definition

### 1.1 Via Taylor Series

$$e^z = \sum_{n=0}^{\infty} \frac{z^n}{n!} = 1 + z + \frac{z^2}{2!} + \frac{z^3}{3!} + \cdots$$

This series converges absolutely for **all** $z \in \mathbb{C}$ (radius of convergence $= \infty$).

### 1.2 In Terms of Real and Imaginary Parts

For $z = x + iy$:

$$\boxed{e^z = e^x(\cos y + i\sin y) = e^x e^{iy}}$$

Thus: $u(x,y) = e^x \cos y$ and $v(x,y) = e^x \sin y$.

### 1.3 Analyticity Check (C-R)

| | Value | C-R |
|--|-------|-----|
| $u_x = e^x \cos y$ | $v_y = e^x \cos y$ | $u_x = v_y$ ✓ |
| $u_y = -e^x \sin y$ | $v_x = e^x \sin y$ | $u_y = -v_x$ ✓ |

$e^z$ is **entire** with derivative $(e^z)' = e^z$.

---

## 2. Fundamental Properties

| Property | Formula |
|----------|---------|
| Derivative | $(e^z)' = e^z$ |
| Functional equation | $e^{z_1+z_2} = e^{z_1} \cdot e^{z_2}$ |
| Never zero | $e^z \ne 0$ for all $z$ |
| Modulus | $\|e^z\| = e^x = e^{\text{Re}(z)}$ |
| Argument | $\arg(e^z) = y + 2k\pi = \text{Im}(z) + 2k\pi$ |
| Periodicity | $e^{z+2\pi i} = e^z$ (period $2\pi i$) |
| Surjective | $e^z$ maps onto $\mathbb{C} \setminus \{0\}$ |
| Not injective | $e^{z_1} = e^{z_2} \iff z_1 - z_2 = 2k\pi i$ |

### 2.1 Proof: $e^z \ne 0$

$|e^z| = e^x > 0$ for all $x \in \mathbb{R}$. Since the modulus is positive, $e^z$ can never be zero.

### 2.2 Periodicity

$$e^{z + 2\pi i} = e^z \cdot e^{2\pi i} = e^z \cdot 1 = e^z$$

The **fundamental period** is $2\pi i$ (purely imaginary!).

---

## 3. Mapping Properties

### 3.1 Image of Horizontal Lines ($y = c$)

$e^{x+ic} = e^x e^{ic}$ — as $x$ varies, this traces a ray from the origin at angle $c$.

### 3.2 Image of Vertical Lines ($x = c$)

$e^{c+iy} = e^c e^{iy}$ — as $y$ varies, this traces a circle of radius $e^c$.

### 3.3 Image of Horizontal Strips

The **fundamental strip** $\{z : 0 \le \text{Im}(z) < 2\pi\}$ maps **bijectively** onto $\mathbb{C} \setminus \{0\}$.

```
Mapping w = e^z:

z-plane (horizontal strip):          w-plane:
    Im                                   Im
  2π ┤──────────────────             ▲
     │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒             │    ╱
     │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒             │  ╱  entire plane
   π ┤▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒   e^z       │╱  minus origin
     │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒  ─────►  ───O──────────► Re
     │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒             │╲
   0 ┤──────────────────             │  ╲
     └──────────────────► Re         │    ╲

  Horizontal lines → rays from origin
  Vertical lines → circles centered at origin
  Strip 0 ≤ Im(z) < 2π → ℂ \ {0}  (one-to-one!)
```

### 3.4 Grid Mapping Detail

```
z-plane grid:              w = e^z mapping:

  y ▲                         Im ▲
    │  │  │  │  │                │      rays (y = const)
  ──┼──┼──┼──┼──┼──            ──┼── ╱ ──── ╱ ────
  ──┼──┼──┼──┼──┼──              │ ╱      ╱
  ──┼──┼──┼──┼──┼──           ───O───────────► Re
  ──┼──┼──┼──┼──┼──              │ ╲      ╲
  ──┼──┼──┼──┼──┼──            circles (x = const)
    └──┴──┴──┴──┴──► x

  Orthogonal grid maps to orthogonal grid
  (circles ⊥ rays) — conformal mapping!
```

---

## 4. Solving $e^z = w$

Given $w \ne 0$, find all $z$ such that $e^z = w$.

Write $w = |w|e^{i\phi}$ where $\phi = \arg(w)$. Then:

$$e^z = e^{x+iy} = e^x e^{iy} = |w|e^{i\phi}$$

So $x = \ln|w|$ and $y = \phi + 2k\pi$, giving:

$$z = \ln|w| + i(\phi + 2k\pi) = \text{Log}(w) + 2k\pi i$$

This is the **complex logarithm** (next chapter in this unit).

---

## 5. Design Example

### Example: Find all $z$ such that $e^z = -2$.

**Step 1.** $w = -2$. Modulus: $|w| = 2$. Argument: $\arg(w) = \pi + 2k\pi$.

**Step 2.** $x = \ln 2$, $y = \pi + 2k\pi$.

**Step 3.** Solutions:
$$z = \ln 2 + i(2k+1)\pi, \quad k \in \mathbb{Z}$$

The principal solution ($k = 0$): $z = \ln 2 + i\pi$.

**Verification**: $e^{\ln 2 + i\pi} = 2 \cdot e^{i\pi} = 2(-1) = -2$ ✓

---

## 6. Real-World Applications

| Application | Usage |
|-------------|-------|
| **Signal Processing** | Complex exponentials $e^{i\omega t}$ are basis functions for Fourier analysis |
| **Quantum Mechanics** | Time evolution operator $e^{-iHt/\hbar}$ |
| **Control Systems** | Transfer function poles/zeros; system response $e^{st}$ |
| **Electrical Engineering** | Phasors: $V(t) = \text{Re}(V_0 e^{i\omega t})$ |

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| Definition | $e^z = e^x(\cos y + i\sin y)$ |
| Series | $e^z = \sum z^n/n!$ (converges everywhere) |
| Derivative | $(e^z)' = e^z$ |
| Modulus | $\|e^z\| = e^{\text{Re}(z)}$ |
| Period | $2\pi i$ |
| Never zero | $e^z \ne 0$ for all $z$ |
| Surjectivity | $e^z$ maps onto $\mathbb{C} \setminus \{0\}$ |
| Injectivity | One-to-one on any horizontal strip of height $< 2\pi$ |
| Solving $e^z = w$ | $z = \ln\|w\| + i\arg(w)$ (infinitely many solutions) |

---

## Quick Revision Questions

1. **Compute** $e^{2+i\pi/3}$ in the form $a + bi$.

2. **Prove** that $e^z$ is never zero using the modulus.

3. **Show** that $e^z$ is periodic with period $2\pi i$ and that this is the smallest period.

4. **Describe** the image of the rectangle $\{0 \le x \le 1, \; 0 \le y \le \pi\}$ under $w = e^z$.

5. **Find** all solutions of $e^z = 1 + i$.

6. **Explain** why $|e^{iy}| = 1$ for all real $y$, and interpret this geometrically.

---

[← Previous: Harmonic Functions](../02-Complex-Functions/06-harmonic-functions.md) | [Next: Trigonometric Functions →](02-trigonometric-functions.md)
