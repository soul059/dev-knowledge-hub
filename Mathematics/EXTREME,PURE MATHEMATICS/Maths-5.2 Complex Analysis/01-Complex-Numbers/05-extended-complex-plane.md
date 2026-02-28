# 1.5 Extended Complex Plane (Riemann Sphere)

[← Previous: Roots of Complex Numbers](04-roots-of-complex-numbers.md) | [Next: Unit 2 — Functions of a Complex Variable →](../02-Complex-Functions/01-functions-of-complex-variable.md)

---

## Chapter Overview

The **extended complex plane** $\hat{\mathbb{C}} = \mathbb{C} \cup \{\infty\}$ adds a single "point at infinity" to the complex plane. Via **stereographic projection**, this becomes the **Riemann sphere**, giving a compact topological space where every polynomial has roots and every rational function is continuous. This compactification is essential for understanding poles, residues, and Möbius transformations.

---

## 1. Motivation

Several natural situations require $\infty$:

- $f(z) = 1/z$ is undefined at $z = 0$, but $\lim_{z \to 0} f(z) = \infty$
- The function $f(z) = z^2$ "sends large values to infinity"
- Möbius transformations $\frac{az+b}{cz+d}$ send $z = -d/c$ to $\infty$

We want a framework where these are handled cleanly.

---

## 2. Definition of the Extended Complex Plane

### 2.1 One-Point Compactification

$$\hat{\mathbb{C}} = \mathbb{C} \cup \{\infty\}$$

The symbol $\infty$ (also written $\infty_{\mathbb{C}}$) represents a **single** point — there is no distinction between $+\infty$, $-\infty$, $i\infty$, etc.

### 2.2 Arithmetic with $\infty$

For $z \in \mathbb{C}$, $z \ne 0$:

| Operation | Result | Operation | Result |
|-----------|--------|-----------|--------|
| $z + \infty$ | $\infty$ | $z \cdot \infty$ | $\infty$ |
| $z / \infty$ | $0$ | $z / 0$ | $\infty$ |
| $\infty + \infty$ | **undefined** | $0 \cdot \infty$ | **undefined** |
| $\infty \cdot \infty$ | $\infty$ | $\infty / \infty$ | **undefined** |

### 2.3 Neighborhoods of $\infty$

A **neighborhood of $\infty$** is a set of the form:

$$\{z \in \mathbb{C} : |z| > R\} \cup \{\infty\}$$

for some $R > 0$. Intuitively: a disk's exterior plus the point at infinity.

```
Neighborhoods in the extended plane:

  Neighborhood of z₀:         Neighborhood of ∞:

  ╭───────────────────╮       ╭───────────────────╮
  │                   │       │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒│
  │    ╭───╮          │       │▒▒▒╭───────╮▒▒▒▒▒│
  │    │ z₀│          │       │▒▒▒│       │▒▒▒▒▒│
  │    ╰───╯          │       │▒▒▒│  |z|≤R│▒▒▒▒▒│
  │  |z-z₀| < ε      │       │▒▒▒│       │▒▒▒▒▒│
  │                   │       │▒▒▒╰───────╯▒▒▒▒▒│
  ╰───────────────────╯       │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒│
                              ╰───────────────────╯
                              ▒ = neighborhood of ∞
                                (|z| > R, plus ∞)
```

---

## 3. Stereographic Projection and the Riemann Sphere

### 3.1 Construction

Place a unit sphere $S^2$ on top of the complex plane so that the "south pole" $S = (0,0,0)$ touches the origin, and the "north pole" is $N = (0,0,1)$.

For each point $z \in \mathbb{C}$, draw a line from $N$ through the point $(x,y,0)$ corresponding to $z$. This line intersects the sphere at exactly one point $P$ (other than $N$). This defines a bijection:

$$\sigma: \mathbb{C} \to S^2 \setminus \{N\}$$

We extend this by defining $\sigma(\infty) = N$, giving a bijection $\hat{\mathbb{C}} \leftrightarrow S^2$.

```
Stereographic Projection (side view):

              N (North Pole = ∞)
              •
             ╱|╲
            ╱ | ╲
           ╱  |  ╲
          ╱   |   ╲     P = σ(z)
         ╱    |  • ╱        (point on sphere)
        ╱     │╱  ╱
       ╱    ╱ |  ╱
      ╱   ╱   | ╱
     ╱  ╱     |╱
────S─────────┼──────────────• z   (complex plane)
  (0,0,0)     │            (x,y,0)
              │
    Origin of complex plane sits at South Pole
    Line from N through z hits sphere at P
```

### 3.2 Explicit Formulas

**From $\mathbb{C}$ to $S^2$** (stereographic projection):

If $z = x + iy$, the corresponding point $(X, Y, Z)$ on the sphere is:

$$X = \frac{2x}{|z|^2 + 1}, \quad Y = \frac{2y}{|z|^2 + 1}, \quad Z = \frac{|z|^2 - 1}{|z|^2 + 1}$$

**From $S^2$ to $\mathbb{C}$** (inverse):

$$z = \frac{X + iY}{1 - Z}$$

### 3.3 Key Properties of Stereographic Projection

| Property | Description |
|----------|-------------|
| **Conformal** | Preserves angles between curves |
| **Circle-preserving** | Maps circles on $S^2$ to circles or lines in $\mathbb{C}$ |
| **South pole** | $S \leftrightarrow 0$ |
| **North pole** | $N \leftrightarrow \infty$ |
| **Equator** | $\leftrightarrow$ unit circle $|z| = 1$ |
| **Upper hemisphere** | $\leftrightarrow$ $|z| > 1$ (exterior of unit disk) |
| **Lower hemisphere** | $\leftrightarrow$ $|z| < 1$ (interior of unit disk) |

### 3.4 Circles and Lines — Unified View

On the Riemann sphere, **lines in $\mathbb{C}$** correspond to **circles through the north pole $N$**. Thus:

> On the Riemann sphere, there is no distinction between lines and circles. They are all circles on $S^2$.

```
Lines as circles through ∞:

  In ℂ (plane view):              On Riemann sphere:
  
        │                              N
        │  ← a line                   •──╮
        │     (extends to ∞           │   ╲  ← circle through N
        │      in both                │    │
        │      directions)            │   ╱
        │                              •──╯
        │                              S
```

---

## 4. Topology of $\hat{\mathbb{C}}$

### 4.1 Key Topological Facts

| Property | $\mathbb{C}$ | $\hat{\mathbb{C}}$ |
|----------|---------------|---------------------|
| Compact | No | **Yes** (Heine–Borel fails in $\mathbb{C}$) |
| Connected | Yes | Yes |
| Simply connected | Yes | Yes |
| Bounded | No | N/A (compact) |
| Homeomorphic to | $\mathbb{R}^2$ | $S^2$ (sphere) |

### 4.2 Convergence to $\infty$

We say $z_n \to \infty$ if for every $R > 0$, there exists $N$ such that $|z_n| > R$ for all $n > N$.

Equivalently, $1/z_n \to 0$.

---

## 5. Functions on $\hat{\mathbb{C}}$

### 5.1 Behavior at Infinity

To study $f(z)$ near $z = \infty$, substitute $w = 1/z$ and study $g(w) = f(1/w)$ near $w = 0$.

| $f(z)$ near $\infty$ | $g(w) = f(1/w)$ near $0$ | Classification |
|-----------------------|---------------------------|----------------|
| $f(z) \to L$ (finite) | $g(w) \to L$ | Removable singularity |
| $f(z) \to \infty$ | $g$ has a pole at $0$ | Pole at $\infty$ |
| Neither | $g$ has essential singularity | Essential singularity at $\infty$ |

### 5.2 Examples

| Function | Behavior at $\infty$ | Type |
|----------|---------------------|------|
| $f(z) = z^2$ | $z^2 \to \infty$ | Pole of order 2 |
| $f(z) = 1/z$ | $1/z \to 0$ | Zero at $\infty$ |
| $f(z) = e^z$ | Oscillates | Essential singularity |
| $f(z) = \frac{z+1}{z-1}$ | $\to 1$ | Removable ($f(\infty) = 1$) |

---

## 6. The Chordal Metric

### 6.1 Definition

The **chordal distance** between $z_1, z_2 \in \hat{\mathbb{C}}$ is the Euclidean distance between their images on the Riemann sphere:

$$d(z_1, z_2) = \frac{2|z_1 - z_2|}{\sqrt{1+|z_1|^2}\sqrt{1+|z_2|^2}}$$

For $z \in \mathbb{C}$:

$$d(z, \infty) = \frac{2}{\sqrt{1 + |z|^2}}$$

### 6.2 Properties

- $0 \le d(z_1, z_2) \le 2$ for all $z_1, z_2 \in \hat{\mathbb{C}}$
- $d$ makes $\hat{\mathbb{C}}$ a compact metric space
- $d(0, \infty) = 2$ (diameter of the sphere)

---

## 7. Design Example — Stereographic Projection Calculation

### Example: Find the image of $z = 1 + i$ on the Riemann sphere.

**Step 1.** Compute $|z|^2 = 1 + 1 = 2$.

**Step 2.** Apply the formulas:
$$X = \frac{2 \cdot 1}{2 + 1} = \frac{2}{3}, \quad Y = \frac{2 \cdot 1}{2 + 1} = \frac{2}{3}, \quad Z = \frac{2 - 1}{2 + 1} = \frac{1}{3}$$

**Answer:** The point $(2/3, 2/3, 1/3)$ on the unit sphere.

**Verification:** $X^2 + Y^2 + Z^2 = 4/9 + 4/9 + 1/9 = 9/9 = 1$ ✓

---

## 8. Real-World Applications

| Application | How the Extended Plane Is Used |
|-------------|-------------------------------|
| **Control Theory** | Nyquist plots are naturally drawn on $\hat{\mathbb{C}}$; stability at infinity matters |
| **Algebraic Geometry** | Projective line $\mathbb{P}^1 \cong \hat{\mathbb{C}}$ |
| **Relativity** | Celestial sphere of an observer modeled as Riemann sphere |
| **Antenna Patterns** | Radiation patterns mapped via stereographic projection |
| **Cartography** | Stereographic projection used in map-making (conformal maps) |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Extended plane | $\hat{\mathbb{C}} = \mathbb{C} \cup \{\infty\}$ |
| Riemann sphere | $\hat{\mathbb{C}} \cong S^2$ via stereographic projection |
| North pole | $N \leftrightarrow \infty$ |
| South pole | $S \leftrightarrow 0$ |
| Equator | $\leftrightarrow$ unit circle $\|z\| = 1$ |
| Compactness | $\hat{\mathbb{C}}$ is compact; $\mathbb{C}$ is not |
| Lines | Lines in $\mathbb{C}$ = circles through $N$ on sphere |
| Conformal | Stereographic projection preserves angles |
| Chordal metric | $d(z_1,z_2) = \frac{2\|z_1-z_2\|}{\sqrt{1+\|z_1\|^2}\sqrt{1+\|z_2\|^2}}$ |
| Behavior at $\infty$ | Study $f(1/w)$ near $w = 0$ |

---

## Quick Revision Questions

1. **Explain** why we add only *one* point at infinity (rather than directional infinities as in $\mathbb{R}$).

2. **Find** the stereographic projection of $z = i$ onto the Riemann sphere.

3. **Describe** the image of the real line $\mathbb{R}$ under stereographic projection.

4. **Classify** the behavior of $f(z) = z^3 + 2z$ at $z = \infty$.

5. **Prove** that stereographic projection maps circles in $\mathbb{C}$ to circles on $S^2$ (give the main idea without full computation).

6. **Compute** the chordal distance $d(0, 1)$ and $d(1, \infty)$.

---

[← Previous: Roots of Complex Numbers](04-roots-of-complex-numbers.md) | [Next: Unit 2 — Functions of a Complex Variable →](../02-Complex-Functions/01-functions-of-complex-variable.md)
