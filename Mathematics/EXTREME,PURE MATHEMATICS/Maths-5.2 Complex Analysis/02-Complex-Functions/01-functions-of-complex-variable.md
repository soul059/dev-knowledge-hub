# 2.1 Functions of a Complex Variable

[← Previous: Extended Complex Plane](../01-Complex-Numbers/05-extended-complex-plane.md) | [Next: Limits and Continuity →](02-limits-and-continuity.md)

---

## Chapter Overview

A function of a complex variable $f: D \to \mathbb{C}$ maps complex numbers to complex numbers. Since $\mathbb{C} \cong \mathbb{R}^2$, such a function can be viewed as a pair of real-valued functions of two real variables. This dual perspective — complex vs. real — is a recurring theme throughout complex analysis.

---

## 1. Basic Definitions

### 1.1 Complex Function

A **function of a complex variable** is a rule $f$ that assigns to each $z$ in a set $D \subseteq \mathbb{C}$ (the **domain**) a unique complex number $w = f(z)$ in the **range** (or **codomain**).

$$f: D \to \mathbb{C}, \qquad w = f(z)$$

### 1.2 Decomposition into Real and Imaginary Parts

Writing $z = x + iy$ and $w = u + iv$:

$$f(z) = u(x, y) + i\,v(x, y)$$

where $u, v: \mathbb{R}^2 \to \mathbb{R}$ are real-valued functions.

| Complex function | $u(x,y)$ | $v(x,y)$ |
|-----------------|-----------|-----------|
| $f(z) = z^2$ | $x^2 - y^2$ | $2xy$ |
| $f(z) = \bar{z}$ | $x$ | $-y$ |
| $f(z) = |z|^2$ | $x^2 + y^2$ | $0$ |
| $f(z) = 1/z$ | $\frac{x}{x^2+y^2}$ | $\frac{-y}{x^2+y^2}$ |
| $f(z) = e^z$ | $e^x \cos y$ | $e^x \sin y$ |

### 1.3 Domain Conventions

Unless otherwise stated, the domain of $f$ is the **largest open set** on which the definition makes sense.

| Function | Natural Domain |
|----------|---------------|
| $f(z) = z^2 + 1$ | $\mathbb{C}$ (entire plane) |
| $f(z) = 1/z$ | $\mathbb{C} \setminus \{0\}$ |
| $f(z) = \frac{z}{z^2+1}$ | $\mathbb{C} \setminus \{i, -i\}$ |
| $f(z) = \sqrt{z}$ | Requires branch cut (see Unit 3) |

---

## 2. Mappings and Visualization

### 2.1 The Mapping Perspective

Since $f: \mathbb{C} \to \mathbb{C}$ maps a 2D plane to a 2D plane, we visualize it using **two copies of the complex plane**:

```
   z-plane (domain)                w-plane (range)
        Im                              Im
         ▲                               ▲
         │                               │
         │   • z                         │   • w = f(z)
         │                               │
    ─────O──────► Re            ─────O──────► Re
         │           ─── f ──►       │
         │                               │

    We draw the domain and the image separately
```

### 2.2 Image of Sets

Given $f$ and a set $S \subseteq D$, the **image** is:

$$f(S) = \{f(z) : z \in S\}$$

### 2.3 Example: $w = z^2$

The map $w = z^2$ in polar: if $z = re^{i\theta}$, then $w = r^2 e^{2i\theta}$.

**Effect**: Squares the radius and doubles the angle.

```
z-plane:                          w-plane (w = z²):

    Im                                Im
     ▲                                 ▲
     │  ╱                              │   │
     │╱  sector                        │   │   image sector
     │  0 ≤ θ ≤ π/4                   │   │   0 ≤ φ ≤ π/2
     │                                 │   │
─────O──────► Re              ─────O──────► Re

  A sector of angle π/4       maps to a sector of angle π/2
  (angle doubles)
```

**Grid Mapping of $w = z^2$**:
- Vertical lines $x = c$ map to leftward-opening parabolas
- Horizontal lines $y = c$ map to rightward-opening parabolas
- These two families of parabolas are **orthogonal** (because $z^2$ is analytic)

---

## 3. Important Classes of Functions

### 3.1 Polynomials

$$P(z) = a_n z^n + a_{n-1} z^{n-1} + \cdots + a_1 z + a_0$$

- Domain: all of $\mathbb{C}$
- By the Fundamental Theorem of Algebra: $P(z) = a_n \prod_{k=1}^{n}(z - z_k)$
- Every polynomial of degree $n$ has exactly $n$ roots (counting multiplicity)

### 3.2 Rational Functions

$$R(z) = \frac{P(z)}{Q(z)}$$

where $P$ and $Q$ are polynomials. Domain: $\mathbb{C} \setminus \{z : Q(z) = 0\}$.

### 3.3 Power Series

$$f(z) = \sum_{n=0}^{\infty} a_n (z - z_0)^n$$

Converges in a disk $|z - z_0| < R$ (radius of convergence). These **define** analytic functions (Unit 5).

### 3.4 Exponential and Trigonometric

$$e^z, \quad \sin z, \quad \cos z, \quad \sinh z, \quad \cosh z$$

These are entire functions (analytic on all of $\mathbb{C}$) — see Unit 3.

---

## 4. Injectivity, Surjectivity, and Inverses

### 4.1 Key Definitions

| Term | Meaning |
|------|---------|
| **Injective** (one-to-one) | $f(z_1) = f(z_2) \implies z_1 = z_2$ |
| **Surjective** (onto) | For every $w$, there exists $z$ with $f(z) = w$ |
| **Bijective** | Both injective and surjective |

### 4.2 Injectivity Failures

Complex functions can fail to be injective in illuminating ways:

- $f(z) = z^2$: two-to-one (except at $z=0$), since $f(z) = f(-z)$
- $f(z) = e^z$: infinite-to-one, since $e^{z+2\pi i} = e^z$ (periodic)

### 4.3 Branches

When $f$ is not injective, its "inverse" $f^{-1}$ is multi-valued. We handle this by choosing a **branch** — a single-valued selection of the inverse. (More in Unit 3.)

---

## 5. Special Mappings

### 5.1 Translation: $w = z + c$

Shifts every point by the vector $c$.

### 5.2 Rotation: $w = e^{i\alpha} z$

Rotates every point by angle $\alpha$ about the origin.

### 5.3 Scaling: $w = rz$ ($r > 0$)

Dilates by factor $r$.

### 5.4 Inversion: $w = 1/z$

$$w = \frac{1}{z} = \frac{\bar{z}}{|z|^2}$$

Combines inversion in the unit circle ($r \mapsto 1/r$) with reflection across the real axis ($\theta \mapsto -\theta$).

```
Effect of w = 1/z:

  z-plane:                     w-plane:
     Im                           Im
      ▲                            ▲
      │  large |z|                 │  small |w|
      │    •                       │
      │      •                     │  • •
      │        •                   │
  ────O──────────► Re         ────O──────────► Re
      │                            │
      │  small |z|                 │  large |w|
      │  • •                       │    •
      │                            │      •
      │                            │        •

  Inversion: near ↔ far,  inside ↔ outside unit circle
```

### 5.5 Linear Fractional (Möbius): $w = \frac{az+b}{cz+d}$

The most general conformal map of $\hat{\mathbb{C}}$ to itself. (Detailed study in Unit 7.)

---

## 6. Design Example — Mapping Visualization

### Example: Determine the image of the first quadrant under $w = z^2$.

**Step 1.** Write in polar: $z = re^{i\theta}$ with $r > 0$ and $0 < \theta < \pi/2$ (first quadrant).

**Step 2.** Apply: $w = r^2 e^{2i\theta}$.

**Step 3.** New radius $\rho = r^2$ ranges over $(0, \infty)$.

**Step 4.** New angle $\phi = 2\theta$ ranges over $(0, \pi)$ (upper half-plane).

**Conclusion**: The map $w = z^2$ sends the first quadrant to the upper half-plane.

```
z-plane (first quadrant):       w-plane (upper half-plane):

    Im ▲                            Im ▲
       │▒▒▒▒▒▒▒▒▒▒▒                   │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
       │▒▒▒▒▒▒▒▒▒▒▒                   │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
       │▒▒▒▒▒▒▒▒▒▒▒   w = z²         │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
       │▒▒▒▒▒▒▒▒▒▒▒  ──────►         │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
  ─────O───────────► Re          ─────O───────────────────► Re
       │                               │
```

---

## 7. Real-World Applications

| Application | Connection to Complex Functions |
|-------------|-------------------------------|
| **Fluid Dynamics** | Velocity field = derivative of complex potential |
| **Electrostatics** | Electric field lines and equipotentials from analytic functions |
| **Heat Flow** | Steady-state temperature as real part of analytic function |
| **Optics** | Lens aberration analysis via power series |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Complex function | $f(z) = u(x,y) + iv(x,y)$ |
| Domain | Largest open set where $f$ is defined |
| Mapping | $f: z\text{-plane} \to w\text{-plane}$ |
| Polynomial | $P(z) = a_n z^n + \cdots + a_0$, domain $\mathbb{C}$ |
| Rational | $P(z)/Q(z)$, domain $\mathbb{C} \setminus \{Q = 0\}$ |
| $w = z^2$ | Squares radii, doubles angles |
| $w = 1/z$ | Inversion + reflection |
| Multi-valuedness | Arises when $f$ is not injective; handled by branches |

---

## Quick Revision Questions

1. **Decompose** $f(z) = z^3$ into real and imaginary parts $u(x,y)$ and $v(x,y)$.

2. **Describe** the image of the circle $|z| = 2$ under $w = z^2$.

3. **Explain** why $f(z) = e^z$ is not injective on $\mathbb{C}$ and determine its period.

4. **Find** the image of the vertical line $\text{Re}(z) = 1$ under $w = 1/z$.

5. **Why** does the Fundamental Theorem of Algebra guarantee that every polynomial of degree $n \ge 1$ has a root in $\mathbb{C}$?

6. **Determine** the image of the unit disk $|z| < 1$ under $w = 2z + 3$.

---

[← Previous: Extended Complex Plane](../01-Complex-Numbers/05-extended-complex-plane.md) | [Next: Limits and Continuity →](02-limits-and-continuity.md)
