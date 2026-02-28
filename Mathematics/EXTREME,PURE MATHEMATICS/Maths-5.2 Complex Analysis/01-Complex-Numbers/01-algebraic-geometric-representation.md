# 1.1 Algebraic and Geometric Representation of Complex Numbers

[← Back to README](../README.md) | [Next: Polar Form and Euler's Formula →](02-polar-form-eulers-formula.md)

---

## Chapter Overview

Complex numbers extend the real number system by adjoining an element $i$ satisfying $i^2 = -1$. This simple idea unlocks an extraordinarily rich mathematical world. In this chapter we develop the **algebraic** structure of $\mathbb{C}$ (addition, multiplication, conjugation) and the **geometric** interpretation via the Argand plane.

---

## 1. Motivation — Why Complex Numbers?

The equation $x^2 + 1 = 0$ has no solution in $\mathbb{R}$. We *define* a new symbol $i$ (the **imaginary unit**) such that

$$i^2 = -1$$

and then form numbers of the type $z = a + bi$ where $a, b \in \mathbb{R}$.

> **Historical note.** Cardano (1545) and Bombelli (1572) encountered complex numbers while solving cubic equations with real coefficients — even when all three roots are real, the intermediate steps may require $\sqrt{-1}$.

---

## 2. Algebraic Representation

### 2.1 Definition

A **complex number** is an ordered pair $(a, b)$ of real numbers, written as

$$z = a + bi$$

where:
- $a = \text{Re}(z)$ is the **real part**
- $b = \text{Im}(z)$ is the **imaginary part**

The set of all complex numbers is denoted $\mathbb{C}$.

### 2.2 Equality

Two complex numbers are equal if and only if their real parts and imaginary parts are separately equal:

$$a + bi = c + di \iff a = c \text{ and } b = d$$

### 2.3 Arithmetic Operations

Let $z_1 = a + bi$ and $z_2 = c + di$.

| Operation | Formula | Example ($z_1 = 3+2i$, $z_2 = 1-4i$) |
|-----------|---------|---------------------------------------|
| Addition | $(a+c) + (b+d)i$ | $4 - 2i$ |
| Subtraction | $(a-c) + (b-d)i$ | $2 + 6i$ |
| Multiplication | $(ac - bd) + (ad + bc)i$ | $(3+8) + (-12+2)i = 11 - 10i$ |
| Division | $\dfrac{ac+bd}{c^2+d^2} + \dfrac{bc-ad}{c^2+d^2}\,i$ | $\dfrac{3-8}{1+16} + \dfrac{2+12}{1+16}\,i = \dfrac{-5+14i}{17}$ |

**Division trick**: Multiply numerator and denominator by the conjugate of the denominator:

$$\frac{z_1}{z_2} = \frac{z_1 \cdot \bar{z_2}}{z_2 \cdot \bar{z_2}} = \frac{z_1 \cdot \bar{z_2}}{|z_2|^2}$$

### 2.4 Field Axioms

$(\mathbb{C}, +, \cdot)$ forms a **field**:
- Additive identity: $0 = 0 + 0i$
- Multiplicative identity: $1 = 1 + 0i$
- Every $z \ne 0$ has a multiplicative inverse: $z^{-1} = \dfrac{\bar{z}}{|z|^2}$
- All field axioms (commutativity, associativity, distributivity) hold

> **Key difference from $\mathbb{R}$**: $\mathbb{C}$ is **not ordered** — we cannot say $z_1 < z_2$ for complex numbers. There is no total ordering on $\mathbb{C}$ compatible with its field structure.

### 2.5 Complex Conjugate

The **conjugate** of $z = a + bi$ is

$$\bar{z} = a - bi$$

**Properties of conjugation:**

| Property | Formula |
|----------|---------|
| Involution | $\overline{\bar{z}} = z$ |
| Addition | $\overline{z_1 + z_2} = \bar{z}_1 + \bar{z}_2$ |
| Multiplication | $\overline{z_1 \cdot z_2} = \bar{z}_1 \cdot \bar{z}_2$ |
| Real part | $\text{Re}(z) = \dfrac{z + \bar{z}}{2}$ |
| Imaginary part | $\text{Im}(z) = \dfrac{z - \bar{z}}{2i}$ |
| Modulus squared | $z \cdot \bar{z} = |z|^2$ |
| Real condition | $z \in \mathbb{R} \iff z = \bar{z}$ |
| Pure imaginary | $z$ is pure imaginary $\iff z = -\bar{z}$ |

### 2.6 Modulus (Absolute Value)

$$|z| = |a + bi| = \sqrt{a^2 + b^2}$$

**Properties:**

| Property | Formula |
|----------|---------|
| Non-negativity | $|z| \ge 0$, with $|z| = 0 \iff z = 0$ |
| Multiplicativity | $|z_1 z_2| = |z_1| \cdot |z_2|$ |
| Triangle inequality | $|z_1 + z_2| \le |z_1| + |z_2|$ |
| Reverse triangle ineq. | $\big||z_1| - |z_2|\big| \le |z_1 - z_2|$ |
| Conjugate | $|\bar{z}| = |z|$ |

---

## 3. Geometric Representation — The Argand Plane

Every complex number $z = x + iy$ corresponds to a point $(x, y)$ in the plane.

```
        Im (y-axis)
         ▲
         │
    3i ──┤          • z = 3 + 2i
         │        ╱
    2i ──┤      ╱  |z| = √13
         │    ╱
     i ──┤  ╱  θ = arg(z)
         │╱───────────────── Re (x-axis)
    ─────O────┤────┤────┤──►
         │    1    2    3
   -i  ──┤
         │         • z̄ = 3 - 2i
  -2i  ──┤
         │
```

### 3.1 Key Geometric Interpretations

| Algebraic Element | Geometric Meaning |
|-------------------|-------------------|
| $z = x + iy$ | Point $(x, y)$ in the plane |
| $|z|$ | Distance from the origin to $z$ |
| $|z_1 - z_2|$ | Distance between $z_1$ and $z_2$ |
| $\bar{z}$ | Reflection of $z$ across the real axis |
| $iz$ | Rotation of $z$ by $90°$ counterclockwise |
| $-z$ | Rotation of $z$ by $180°$ (point reflection through origin) |
| $\text{Re}(z) = 0$ | Imaginary axis |
| $\text{Im}(z) = 0$ | Real axis |

### 3.2 Geometric Effects of Arithmetic

```
Addition as Vector Sum:
                        Im
                         ▲
                         │    • z₁ + z₂
                         │   ╱│
                    z₂ • ╱  │  (parallelogram law)
                      ╱  ╱   │
                    ╱  ╱     │
                  ╱  ╱       │
                ╱  ╱         │
              ╱──╱───────────┤──► Re
             O         z₁ •
```

- **Addition**: Follows the **parallelogram law** (vector addition)
- **Subtraction**: $z_1 - z_2$ is the vector from $z_2$ to $z_1$
- **Multiplication by $e^{i\theta}$**: Rotation by $\theta$ (see next chapter)
- **Multiplication by $r > 0$**: Scaling (dilation) by factor $r$

### 3.3 Important Sets in the Complex Plane

| Set | Definition | Geometry |
|-----|-----------|----------|
| $|z - z_0| = r$ | Circle of radius $r$ centered at $z_0$ | ○ |
| $|z - z_0| < r$ | Open disk | Interior of circle |
| $|z - z_0| \le r$ | Closed disk | Circle + interior |
| $|z - z_1| = |z - z_2|$ | Perpendicular bisector of segment $z_1 z_2$ | Straight line |
| $\text{Re}(z) = c$ | Vertical line $x = c$ | │ |
| $\text{Im}(z) = c$ | Horizontal line $y = c$ | ─ |

```
Regions in the Complex Plane:

  |z| < 1 (open unit disk)       |z - z₀| = R (circle)

       Im                              Im
        ▲                               ▲
        │   ╭───╮                        │
        │  ╱     ╲                       │    ╭───╮
        │ │   O   │                      │   ╱  •  ╲ z₀
        │  ╲     ╱                       │  │   R   │
        │   ╰───╯                        │   ╲     ╱
   ─────┼──────────► Re            ──────┼────╰───╯──► Re
        │                                │
```

---

## 4. Topology of $\mathbb{C}$

### 4.1 Open and Closed Sets

| Concept | Definition |
|---------|-----------|
| **Neighborhood** | $N(z_0, \varepsilon) = \{z : |z - z_0| < \varepsilon\}$ (open disk) |
| **Interior point** | $z_0$ is interior to $S$ if some neighborhood of $z_0$ lies entirely in $S$ |
| **Boundary point** | Every neighborhood of $z_0$ contains points in $S$ and points not in $S$ |
| **Open set** | Every point is an interior point |
| **Closed set** | Contains all its boundary points |
| **Bounded set** | $S \subset D(0, R)$ for some $R > 0$ |
| **Compact set** | Closed and bounded (Heine–Borel) |
| **Connected set** | Cannot be split into two non-empty disjoint open subsets |
| **Domain** | Open and connected set |
| **Region** | Domain together with some/all/none of its boundary |

### 4.2 Domains — Key Examples

```
Simply Connected:                Multiply Connected:

  ╭────────────╮                  ╭────────────╮
  │            │                  │   ╭────╮   │
  │    No      │                  │   │hole│   │
  │   holes    │                  │   ╰────╯   │
  │            │                  │            │
  ╰────────────╯                  ╰────────────╯

  Every closed curve can           Some closed curves cannot
  be shrunk to a point             be shrunk to a point
```

- A **simply connected domain** has no "holes" — every closed curve in the domain can be continuously shrunk to a point within the domain.
- This distinction is crucial for Cauchy's theorem (Unit 4).

---

## 5. Design Example — Step by Step

### Example: Express $\dfrac{(1 + i)^3}{(1 - i)^2}$ in the form $a + bi$.

**Step 1.** Compute $(1+i)^2 = 1 + 2i + i^2 = 2i$

**Step 2.** Compute $(1+i)^3 = (1+i)(2i) = 2i + 2i^2 = -2 + 2i$

**Step 3.** Compute $(1-i)^2 = 1 - 2i + i^2 = -2i$

**Step 4.** Divide:
$$\frac{-2 + 2i}{-2i} = \frac{(-2+2i)}{-2i} \cdot \frac{2i}{2i} = \frac{(-2+2i)(2i)}{-2i \cdot 2i}$$

$$= \frac{-4i + 4i^2}{-4i^2} = \frac{-4 - 4i}{4} = -1 - i$$

**Answer:** $-1 - i$

---

## 6. Real-World Applications

| Application | How Complex Numbers Are Used |
|-------------|------------------------------|
| **Electrical Engineering** | AC circuits: impedance $Z = R + iX$ combines resistance & reactance |
| **Quantum Mechanics** | Wave functions are complex-valued; probability amplitudes |
| **Signal Processing** | Fourier transforms use $e^{i\omega t}$; phasors for sinusoidal signals |
| **Fluid Dynamics** | Complex potential $\Phi + i\Psi$ encodes velocity fields |
| **Control Theory** | Pole-zero diagrams in the complex $s$-plane |
| **Fractals** | Mandelbrot and Julia sets defined via complex iteration |

### AC Circuit Phasor Diagram (ASCII)

```
        Im (Reactance)
         ▲
         │    Z = R + jXL
         │   ╱
    XL ──┤  ╱ |Z|
         │ ╱
         │╱ φ = arctan(XL/R)
    ─────O────────────────► Re (Resistance)
         │                R
         │
   -XC ──┤  ╲
         │   ╲ Z = R - jXC
         │    ╲
         │

  Impedance: Z = R + j(XL - XC)
  |Z| = √(R² + (XL-XC)²)
  Phase angle: φ = arctan((XL-XC)/R)
```

---

## Summary Table

| Concept | Key Formula / Fact |
|---------|--------------------|
| Complex number | $z = a + bi$, $\;a,b \in \mathbb{R}$ |
| Conjugate | $\bar{z} = a - bi$ |
| Modulus | $\|z\| = \sqrt{a^2 + b^2}$ |
| Modulus via conjugate | $\|z\|^2 = z\bar{z}$ |
| Inverse | $z^{-1} = \bar{z}/\|z\|^2$ |
| Triangle inequality | $\|z_1 + z_2\| \le \|z_1\| + \|z_2\|$ |
| Geometric meaning | Point $(a,b)$ in the Argand plane |
| $\|z_1 - z_2\|$ | Distance between $z_1$ and $z_2$ |
| $\mathbb{C}$ as a field | Not ordered; algebraically closed |
| Simply connected | No holes — key for integration theorems |

---

## Quick Revision Questions

1. **Compute** $(2 + 3i)(4 - i)$ and express the result in the form $a + bi$.

2. **Find** the modulus and conjugate of $z = \dfrac{1 + i}{1 - i}$.

3. **Prove** that $|z_1 z_2| = |z_1| \cdot |z_2|$ using the conjugate property $|z|^2 = z\bar{z}$.

4. **Describe geometrically** the set $\{z \in \mathbb{C} : |z - 1 - i| = 2\}$.

5. **Show** that $\text{Re}(z) = \dfrac{z + \bar{z}}{2}$ and $\text{Im}(z) = \dfrac{z - \bar{z}}{2i}$.

6. **Explain** why $\mathbb{C}$ cannot be given a total ordering compatible with its field operations.

---

[← Back to README](../README.md) | [Next: Polar Form and Euler's Formula →](02-polar-form-eulers-formula.md)
