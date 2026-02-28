# 1.2 Polar Form and Euler's Formula

[← Previous: Algebraic and Geometric Representation](01-algebraic-geometric-representation.md) | [Next: De Moivre's Theorem →](03-de-moivres-theorem.md)

---

## Chapter Overview

While the Cartesian form $z = x + iy$ is natural for addition, the **polar form** $z = r e^{i\theta}$ is far more natural for multiplication, division, and powers. Euler's formula $e^{i\theta} = \cos\theta + i\sin\theta$ is the bridge between the two and is one of the most important identities in all of mathematics.

---

## 1. Polar Coordinates in the Complex Plane

Any nonzero complex number $z = x + iy$ can be written in terms of its distance from the origin and its angle with the positive real axis.

```
        Im
         ▲
         │        • z = x + iy
         │       ╱|
         │      ╱ |
         │  r  ╱  | y
         │    ╱   |
         │   ╱    |
         │  ╱ θ   |
    ─────O╱───────┤──────► Re
         │    x
```

### 1.1 Modulus and Argument

$$z = x + iy \quad\Longrightarrow\quad r = |z| = \sqrt{x^2 + y^2}, \qquad \theta = \arg(z)$$

where $\theta$ satisfies:

$$x = r\cos\theta, \qquad y = r\sin\theta$$

### 1.2 Polar Form

$$\boxed{z = r(\cos\theta + i\sin\theta)}$$

Often abbreviated as $z = r\,\text{cis}\,\theta$.

### 1.3 Computing the Argument

$$\theta = \arg(z) = \arctan\!\left(\frac{y}{x}\right) \quad \text{(adjusted for quadrant)}$$

| Quadrant | Condition | $\arg(z)$ |
|----------|-----------|-----------|
| I | $x > 0, y > 0$ | $\arctan(y/x)$ |
| II | $x < 0, y > 0$ | $\pi + \arctan(y/x)$ |
| III | $x < 0, y < 0$ | $-\pi + \arctan(y/x)$ |
| IV | $x > 0, y < 0$ | $\arctan(y/x)$ |
| Positive real axis | $x > 0, y = 0$ | $0$ |
| Negative real axis | $x < 0, y = 0$ | $\pi$ |
| Positive imaginary | $x = 0, y > 0$ | $\pi/2$ |
| Negative imaginary | $x = 0, y < 0$ | $-\pi/2$ |

### 1.4 Principal Argument

The **argument** is multi-valued: if $\theta_0$ is one value, then $\theta_0 + 2k\pi$ ($k \in \mathbb{Z}$) are all valid arguments.

The **principal argument** $\text{Arg}(z)$ is the unique value in $(-\pi, \pi]$:

$$\arg(z) = \text{Arg}(z) + 2k\pi, \quad k \in \mathbb{Z}$$

```
Argument Diagram — Multi-valuedness:

         Im
          ▲
          │     • z
          │    ╱
          │   ╱
          │  ╱ θ₀ = Arg(z)
     ─────O╱─────────► Re
          │
          │  Also valid:
          │  θ₀ + 2π, θ₀ + 4π, θ₀ - 2π, ...
```

---

## 2. Euler's Formula

### 2.1 The Formula

$$\boxed{e^{i\theta} = \cos\theta + i\sin\theta}$$

This is not a *definition*; it follows naturally from the Taylor series:

$$e^{i\theta} = \sum_{n=0}^{\infty} \frac{(i\theta)^n}{n!} = \underbrace{\sum_{k=0}^{\infty} \frac{(-1)^k \theta^{2k}}{(2k)!}}_{\cos\theta} + i\underbrace{\sum_{k=0}^{\infty} \frac{(-1)^k \theta^{2k+1}}{(2k+1)!}}_{\sin\theta}$$

### 2.2 Exponential (Polar) Form

Combining with the polar form:

$$\boxed{z = r\,e^{i\theta}}$$

where $r = |z|$ and $\theta = \arg(z)$.

### 2.3 Special Values

| $\theta$ | $e^{i\theta}$ | Point on Unit Circle |
|-----------|---------------|---------------------|
| $0$ | $1$ | $(1, 0)$ |
| $\pi/6$ | $\frac{\sqrt{3}}{2} + \frac{1}{2}i$ | |
| $\pi/4$ | $\frac{\sqrt{2}}{2} + \frac{\sqrt{2}}{2}i$ | |
| $\pi/3$ | $\frac{1}{2} + \frac{\sqrt{3}}{2}i$ | |
| $\pi/2$ | $i$ | $(0, 1)$ |
| $\pi$ | $-1$ | $(-1, 0)$ |
| $3\pi/2$ | $-i$ | $(0, -1)$ |
| $2\pi$ | $1$ | $(1, 0)$ — full circle |

```
Unit Circle:  |z| = 1,  z = e^(iθ)

                  i (π/2)
                  •
              ╱       ╲
           ╱               ╲
    (-1) •         O          • (1)    ← θ = 0
     (π)   ╲               ╱
              ╲       ╱
                  •
                -i (3π/2)

    Every point on the unit circle is e^(iθ) for some θ
```

### 2.4 Euler's Identity

Setting $\theta = \pi$:

$$\boxed{e^{i\pi} + 1 = 0}$$

This stunning equation connects the five fundamental constants: $e$, $i$, $\pi$, $1$, and $0$.

---

## 3. Arithmetic in Polar Form

### 3.1 Multiplication

$$z_1 z_2 = r_1 e^{i\theta_1} \cdot r_2 e^{i\theta_2} = r_1 r_2 \, e^{i(\theta_1 + \theta_2)}$$

> **Multiply moduli, add arguments.**

```
Multiplication = Scale + Rotate:

         Im                           Im
          ▲                            ▲
          │  z₂                        │     z₁z₂
          │ ╱                          │    ╱
          │╱ θ₂                        │   ╱
     ─────O──────► Re            ─────O──╱──────► Re
          │╲ θ₁                        │  θ₁+θ₂
          │ ╲                          │
          │  z₁                        │

     |z₁z₂| = |z₁|·|z₂|
     arg(z₁z₂) = arg(z₁) + arg(z₂)
```

### 3.2 Division

$$\frac{z_1}{z_2} = \frac{r_1}{r_2}\, e^{i(\theta_1 - \theta_2)}$$

> **Divide moduli, subtract arguments.**

### 3.3 Conjugation

$$\overline{r e^{i\theta}} = r e^{-i\theta}$$

> **Negate the argument (reflect across real axis).**

### 3.4 Inverse

$$\frac{1}{z} = \frac{1}{r}\, e^{-i\theta}$$

---

## 4. Connections Between Forms

### 4.1 Conversion Table

| From | To | Method |
|------|----|--------|
| Cartesian → Polar | $r = \sqrt{x^2+y^2}$, $\theta = \text{Arg}(z)$ | Compute modulus and argument |
| Polar → Cartesian | $x = r\cos\theta$, $y = r\sin\theta$ | Expand $e^{i\theta}$ |

### 4.2 Useful Identities from Euler's Formula

$$\cos\theta = \frac{e^{i\theta} + e^{-i\theta}}{2}$$

$$\sin\theta = \frac{e^{i\theta} - e^{-i\theta}}{2i}$$

These formulas are the starting point for complex definitions of trigonometric functions (Unit 3).

---

## 5. Argument Properties

### 5.1 Multi-valued Nature

The argument function satisfies:

$$\arg(z_1 z_2) = \arg(z_1) + \arg(z_2) \pmod{2\pi}$$

**Warning**: The principal argument $\text{Arg}$ does NOT satisfy this additively in general:

$$\text{Arg}(z_1 z_2) \ne \text{Arg}(z_1) + \text{Arg}(z_2) \quad \text{in general}$$

**Counterexample**: Let $z_1 = z_2 = -1$. Then $\text{Arg}(z_1) = \text{Arg}(z_2) = \pi$, but $z_1 z_2 = 1$ so $\text{Arg}(z_1 z_2) = 0 \ne 2\pi$.

### 5.2 Argument of Quotient

$$\arg\!\left(\frac{z_1}{z_2}\right) = \arg(z_1) - \arg(z_2) \pmod{2\pi}$$

---

## 6. Design Example — Step by Step

### Example: Convert $z = -1 + i\sqrt{3}$ to polar form.

**Step 1.** Find the modulus:
$$r = |z| = \sqrt{(-1)^2 + (\sqrt{3})^2} = \sqrt{1 + 3} = 2$$

**Step 2.** Find the argument. Since $x = -1 < 0$ and $y = \sqrt{3} > 0$, the point is in Quadrant II.
$$\theta = \pi - \arctan\!\left(\frac{\sqrt{3}}{1}\right) = \pi - \frac{\pi}{3} = \frac{2\pi}{3}$$

**Step 3.** Write in polar/exponential form:
$$z = 2\left(\cos\frac{2\pi}{3} + i\sin\frac{2\pi}{3}\right) = 2e^{i \cdot 2\pi/3}$$

**Verification**: $2\cos(2\pi/3) = 2(-1/2) = -1$ ✓ and $2\sin(2\pi/3) = 2(\sqrt{3}/2) = \sqrt{3}$ ✓

---

## 7. Real-World Applications

### 7.1 Phasors in Electrical Engineering

In AC circuit analysis, sinusoidal voltages and currents are represented as the real parts of complex exponentials:

$$v(t) = V_0 \cos(\omega t + \phi) = \text{Re}\!\left(V_0 \, e^{i(\omega t + \phi)}\right)$$

The **phasor** $\tilde{V} = V_0 e^{i\phi}$ captures amplitude and phase in a single complex number.

```
Phasor Diagram for Series RLC:

         Im
          ▲
          │    VL = jωLI
          │    ↑
          │    │
          │    │
     ─────O────┼────────► Re
          │    │    VR = RI
          │    │
          │    ↓
          │    VC = I/(jωC)

   V = VR + VL + VC  (vector/phasor sum)
```

### 7.2 Rotation in Computer Graphics

To rotate a 2D point $(x,y)$ by angle $\theta$ about the origin, represent it as $z = x + iy$ and compute:

$$z' = z \cdot e^{i\theta}$$

This is simpler and more elegant than the rotation matrix approach.

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| Polar form | $z = r(\cos\theta + i\sin\theta)$ |
| Exponential form | $z = re^{i\theta}$ |
| Euler's formula | $e^{i\theta} = \cos\theta + i\sin\theta$ |
| Euler's identity | $e^{i\pi} + 1 = 0$ |
| Modulus from polar | $\|z\| = r$ |
| Argument | $\arg(z) = \theta$ (multi-valued) |
| Principal argument | $\text{Arg}(z) \in (-\pi, \pi]$ |
| Multiplication | $r_1 e^{i\theta_1} \cdot r_2 e^{i\theta_2} = r_1 r_2 e^{i(\theta_1+\theta_2)}$ |
| Division | $\frac{r_1 e^{i\theta_1}}{r_2 e^{i\theta_2}} = \frac{r_1}{r_2} e^{i(\theta_1-\theta_2)}$ |
| $\cos$ from exponentials | $\cos\theta = \frac{e^{i\theta}+e^{-i\theta}}{2}$ |
| $\sin$ from exponentials | $\sin\theta = \frac{e^{i\theta}-e^{-i\theta}}{2i}$ |

---

## Quick Revision Questions

1. **Convert** $z = -2 - 2i$ to polar form $re^{i\theta}$ with $\theta = \text{Arg}(z)$.

2. **Compute** the product $(3e^{i\pi/4})(2e^{i\pi/3})$ in both polar and Cartesian form.

3. **Prove** Euler's formula using the Taylor series for $e^x$, $\cos x$, and $\sin x$.

4. **Why** is $\text{Arg}(z_1 z_2) \ne \text{Arg}(z_1) + \text{Arg}(z_2)$ in general? Give a specific counterexample.

5. **Express** $\cos\theta$ and $\sin\theta$ in terms of $e^{i\theta}$ and $e^{-i\theta}$.

6. **Show** that $|e^{i\theta}| = 1$ for all $\theta \in \mathbb{R}$, i.e., $e^{i\theta}$ lies on the unit circle.

---

[← Previous: Algebraic and Geometric Representation](01-algebraic-geometric-representation.md) | [Next: De Moivre's Theorem →](03-de-moivres-theorem.md)
