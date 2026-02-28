# 4.1 Contours and Paths

[← Previous: Multi-valued Functions](../03-Elementary-Functions/06-multi-valued-functions.md) | [Next: Line Integrals →](02-line-integrals.md)

---

## Chapter Overview

Before integrating in the complex plane, we need a precise language for the curves along which we integrate. This chapter covers **paths**, **contours**, and their topological properties — connectedness, simple-connectedness, and homotopy — which determine when integrals depend (or do not depend) on the chosen path.

---

## 1. Parametric Curves

### 1.1 Definition

A **path** (or **curve**) in $\mathbb{C}$ is a continuous function

$$\gamma : [a,b] \to \mathbb{C}, \quad t \mapsto \gamma(t) = x(t) + iy(t)$$

- **Initial point:** $\gamma(a)$
- **Terminal point:** $\gamma(b)$
- **Closed curve:** $\gamma(a) = \gamma(b)$

### 1.2 Smooth Curves

A path $\gamma$ is **smooth** if $\gamma'(t) = x'(t) + iy'(t)$ exists, is continuous, and is nonzero on $(a,b)$.

### 1.3 Contour (Piecewise Smooth Curve)

A **contour** is a piecewise smooth curve:

$$\gamma = \gamma_1 + \gamma_2 + \cdots + \gamma_n$$

where each $\gamma_k$ is smooth, and the terminal point of $\gamma_k$ equals the initial point of $\gamma_{k+1}$.

```
Examples of contours:

(a) Smooth curve:           (b) Piecewise smooth:       (c) Closed contour:
                                    •
    •───────╮                      ╱│                        ╭───╮
            │                     ╱ │                       ╱     ╲
            ╰───•                •──╯                      •       •
                                                            ╲     ╱
                                                             ╰───╯
```

---

## 2. Common Contours

### 2.1 Line Segment

From $z_1$ to $z_2$:

$$\gamma(t) = z_1 + t(z_2 - z_1), \quad t \in [0,1]$$

$$\gamma(t) = (1-t)z_1 + tz_2$$

### 2.2 Circle

Circle of radius $R$ centered at $z_0$, traversed counterclockwise:

$$\gamma(t) = z_0 + Re^{it}, \quad t \in [0, 2\pi]$$

```
Circle contour C_R(z₀):

          ╭──────╮
         ╱   R    ╲
        │    z₀    │   ← counterclockwise (positive)
         ╲        ╱
          ╰──────╯
              ↺
```

### 2.3 Semicircular Contour

Upper semicircle of radius $R$:

$$\gamma(t) = Re^{it}, \quad t \in [0, \pi]$$

### 2.4 Rectangular Contour

```
Rectangular contour from -R to R with height h:

   -R+ih ─────────────── R+ih
     │                     │
     │    ← positive       │
     │     orientation     │
     │                     │
    -R ─────────────────── R
```

$$\gamma = \gamma_1 + \gamma_2 + \gamma_3 + \gamma_4$$

where $\gamma_1$: bottom, $\gamma_2$: right side, $\gamma_3$: top, $\gamma_4$: left side.

---

## 3. Orientation and Reversal

### 3.1 Positive Orientation

A simple closed curve is **positively oriented** (counterclockwise) if the interior is on the **left** as you traverse the curve.

### 3.2 Reverse Curve

If $\gamma: [a,b] \to \mathbb{C}$, the **reverse** is:

$$(-\gamma)(t) = \gamma(a + b - t), \quad t \in [a,b]$$

Key property: $\int_{-\gamma} f\,dz = -\int_{\gamma} f\,dz$

---

## 4. Topological Concepts

### 4.1 Simple Curve (Jordan Curve)

A closed curve $\gamma$ is **simple** if it does not self-intersect (except at its endpoints):

$$\gamma(t_1) \neq \gamma(t_2) \text{ for } t_1 \neq t_2 \text{ (unless } \{t_1, t_2\} = \{a, b\}\text{)}$$

```
Simple vs. Non-simple:

  Simple:                Not simple (self-intersecting):
   ╭───╮                      ╭───╮
  │     │                    │   ╳ │
  │     │                    │  ╱ ╲ │
   ╰───╯                      ╰╱───╲╯
```

### 4.2 Jordan Curve Theorem

Every simple closed curve divides the plane into exactly two regions:
- A bounded **interior**
- An unbounded **exterior**

### 4.3 Connected and Simply Connected Domains

| Domain Type | Definition | Example |
|-------------|-----------|---------|
| **Connected** | Any two points can be joined by a path in $D$ | Open disk $|z| < 1$ |
| **Simply connected** | Connected + every closed curve can be shrunk to a point | Open disk $|z| < 1$ |
| **Multiply connected** | Connected but not simply connected (has "holes") | Annulus $1 < |z| < 2$ |

```
Simply connected:              Multiply connected:

  ┌──────────────┐              ┌──────────────┐
  │              │              │     ╭──╮     │
  │  no holes    │              │     │xx│     │
  │              │              │     ╰──╯     │
  │              │              │   (hole!)    │
  └──────────────┘              └──────────────┘
```

### 4.4 Homotopy

Two closed curves $\gamma_0, \gamma_1$ in $D$ are **homotopic** if one can be continuously deformed into the other within $D$.

A curve is **null-homotopic** (homotopic to a point) in $D$ if and only if $D$ is simply connected.

```
Homotopy of curves in a simply connected domain:

  ╭─── γ₀ ───╮           ╭── γ_{1/3} ──╮         ╭─ γ_{2/3} ─╮        • ← point
 ╱             ╲         ╱               ╲       ╱              ╲
│      z₀       │  ──►  │      z₀        │ ──► │      z₀       │ ──►  z₀
 ╲             ╱         ╲               ╱       ╲              ╱
  ╰───────────╯           ╰─────────────╯         ╰────────────╯
```

---

## 5. Arc Length

For a smooth curve $\gamma: [a,b] \to \mathbb{C}$:

$$L(\gamma) = \int_a^b |\gamma'(t)|\,dt = \int_a^b \sqrt{[x'(t)]^2 + [y'(t)]^2}\,dt$$

**Example:** Circle $\gamma(t) = Re^{it}$, $t \in [0, 2\pi]$:

$$\gamma'(t) = iRe^{it}, \quad |\gamma'(t)| = R$$

$$L = \int_0^{2\pi} R\,dt = 2\pi R \quad \checkmark$$

---

## 6. Design Example

### Example: Parametrize the boundary of the triangle with vertices $0, 1, i$ (positively oriented).

**Step 1.** Identify the three sides:
- $\gamma_1$: from $0$ to $1$
- $\gamma_2$: from $1$ to $i$
- $\gamma_3$: from $i$ to $0$

**Step 2.** Parametrize each segment using $\gamma(t) = (1-t)z_1 + tz_2$:

$$\gamma_1(t) = t, \quad t \in [0,1]$$
$$\gamma_2(t) = (1-t) \cdot 1 + t \cdot i = 1-t+it, \quad t \in [0,1]$$
$$\gamma_3(t) = (1-t)i, \quad t \in [0,1]$$

**Step 3.** Compute derivatives:

$$\gamma_1'(t) = 1, \quad \gamma_2'(t) = -1+i, \quad \gamma_3'(t) = -i$$

**Step 4.** Arc length:

$$L = 1 + |-1+i| + |{-i}| = 1 + \sqrt{2} + 1 = 2 + \sqrt{2}$$

```
Triangle contour:

   i •
     │╲
  γ₃ │  ╲ γ₂
     │    ╲
     •─────• 1
     0  γ₁
     (counterclockwise = positive)
```

---

## 7. Real-World Applications

| Application | Role of Contours |
|-------------|-----------------|
| **Electrostatics** | Integration contours around charges |
| **Signal processing** | Integration around poles in transfer functions |
| **Aerodynamics** | Contour integration for lift (Kutta–Joukowski) |
| **Quantum field theory** | Feynman contours (Wick rotation) |

---

## Summary Table

| Concept | Definition / Key Property |
|---------|--------------------------|
| Path | Continuous $\gamma: [a,b] \to \mathbb{C}$ |
| Contour | Piecewise smooth path |
| Closed curve | $\gamma(a) = \gamma(b)$ |
| Simple curve | No self-intersections |
| Positive orientation | Interior on left; counterclockwise |
| Simply connected | Connected + no holes |
| Multiply connected | Has holes; some curves not null-homotopic |
| Arc length | $L = \int_a^b |\gamma'(t)|\,dt$ |
| Line segment parametrization | $(1-t)z_1 + tz_2$, $t \in [0,1]$ |
| Circle parametrization | $z_0 + Re^{it}$, $t \in [0, 2\pi]$ |

---

## Quick Revision Questions

1. **Parametrize** the circle $|z - 2i| = 3$ traversed clockwise.

2. **Explain** the difference between simply connected and multiply connected domains. Give an example of each.

3. **Prove** that the arc length of $\gamma(t) = e^{it}$, $t \in [0, \pi]$ is $\pi$.

4. **State** the Jordan Curve Theorem and explain its relevance to complex integration.

5. **Parametrize** the square with vertices $\pm 1 \pm i$ as a positively oriented contour.

6. **Decide**: Is the annulus $\{z : 1 < |z| < 3\}$ simply connected? Justify your answer.

---

[← Previous: Multi-valued Functions](../03-Elementary-Functions/06-multi-valued-functions.md) | [Next: Line Integrals →](02-line-integrals.md)
