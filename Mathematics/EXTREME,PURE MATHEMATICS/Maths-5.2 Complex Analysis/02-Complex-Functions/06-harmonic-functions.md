# 2.6 Harmonic Functions

[← Previous: Analytic Functions](05-analytic-functions.md) | [Next: Unit 3 — Exponential Function →](../03-Elementary-Functions/01-exponential-function.md)

---

## Chapter Overview

Harmonic functions — solutions of Laplace's equation — arise naturally as the real and imaginary parts of analytic functions. This connection between complex analysis and potential theory is one of the most fruitful in mathematics, with applications across physics and engineering.

---

## 1. Definition and Basic Properties

### 1.1 Laplace's Equation

A real-valued function $\phi(x,y)$ with continuous second partial derivatives is **harmonic** on a domain $D$ if it satisfies **Laplace's equation**:

$$\boxed{\nabla^2 \phi = \frac{\partial^2 \phi}{\partial x^2} + \frac{\partial^2 \phi}{\partial y^2} = 0}$$

The operator $\nabla^2 = \frac{\partial^2}{\partial x^2} + \frac{\partial^2}{\partial y^2}$ is called the **Laplacian**.

### 1.2 Connection to Analytic Functions

> **Theorem.** If $f(z) = u(x,y) + iv(x,y)$ is analytic on a domain $D$, then both $u$ and $v$ are harmonic on $D$.

**Proof sketch**: From C–R: $u_x = v_y$ and $u_y = -v_x$. Differentiating:
$$u_{xx} = v_{yx}, \quad u_{yy} = -v_{xy}$$
Since $v_{xy} = v_{yx}$ (continuous partials): $u_{xx} + u_{yy} = 0$. Similarly for $v$.

### 1.3 Harmonic Conjugate

If $u$ is harmonic on a simply connected domain, there exists a harmonic function $v$ such that $f = u + iv$ is analytic. The function $v$ is called the **harmonic conjugate** of $u$.

> **Warning**: The harmonic conjugate is unique up to an additive constant. Also, $v$ is the harmonic conjugate of $u$, but $u$ is the harmonic conjugate of $-v$ (not $v$).

---

## 2. Examples

### 2.1 Verifying Harmonicity

| Function $\phi(x,y)$ | $\phi_{xx}$ | $\phi_{yy}$ | $\phi_{xx} + \phi_{yy}$ | Harmonic? |
|----------------------|-------------|-------------|------------------------|-----------|
| $x^2 - y^2$ | $2$ | $-2$ | $0$ | ✓ |
| $2xy$ | $0$ | $0$ | $0$ | ✓ |
| $e^x \cos y$ | $e^x \cos y$ | $-e^x \cos y$ | $0$ | ✓ |
| $\ln(x^2 + y^2)$ | $\frac{2(y^2-x^2)}{(x^2+y^2)^2}$ | $\frac{2(x^2-y^2)}{(x^2+y^2)^2}$ | $0$ | ✓ |
| $x^2 + y^2$ | $2$ | $2$ | $4$ | ✗ |
| $x^3$ | $6x$ | $0$ | $6x$ | ✗ |

### 2.2 Conjugate Pairs from Known Analytic Functions

| Analytic $f(z)$ | $u = \text{Re}(f)$ | $v = \text{Im}(f)$ |
|-----------------|---------------------|---------------------|
| $z^2$ | $x^2 - y^2$ | $2xy$ |
| $z^3$ | $x^3 - 3xy^2$ | $3x^2y - y^3$ |
| $e^z$ | $e^x \cos y$ | $e^x \sin y$ |
| $\frac{1}{z}$ | $\frac{x}{x^2+y^2}$ | $\frac{-y}{x^2+y^2}$ |
| $\text{Log}(z)$ | $\frac{1}{2}\ln(x^2+y^2)$ | $\arctan(y/x)$ |

---

## 3. Finding the Harmonic Conjugate

### 3.1 Method (via Integration of C–R Equations)

Given harmonic $u(x,y)$, find $v(x,y)$ such that $f = u + iv$ is analytic.

1. From $v_y = u_x$: integrate w.r.t. $y$ to get $v = \int u_x\, dy + g(x)$
2. From $v_x = -u_y$: differentiate $v$ w.r.t. $x$ and equate to determine $g(x)$

### 3.2 Detailed Example

**Given**: $u = x^3 - 3xy^2 + 2x$. Find $v$ and $f(z)$.

**Step 1.** Verify harmonic:
$$u_{xx} = 6x, \quad u_{yy} = -6x \implies u_{xx} + u_{yy} = 0 \; ✓$$

**Step 2.** From $v_y = u_x = 3x^2 - 3y^2 + 2$:
$$v = \int (3x^2 - 3y^2 + 2)\, dy = 3x^2 y - y^3 + 2y + g(x)$$

**Step 3.** From $v_x = -u_y = 6xy$:
$$v_x = 6xy + g'(x) = 6xy \implies g'(x) = 0 \implies g(x) = C$$

**Step 4.** Result:
$$v = 3x^2y - y^3 + 2y + C$$
$$f(z) = (x^3 - 3xy^2 + 2x) + i(3x^2y - y^3 + 2y) + iC = z^3 + 2z + iC$$

---

## 4. Properties of Harmonic Functions

### 4.1 Mean Value Property

> If $\phi$ is harmonic in a domain containing the closed disk $\overline{D(z_0, R)}$, then the value at the center equals the average over the circle:

$$\phi(x_0, y_0) = \frac{1}{2\pi}\int_0^{2\pi} \phi(x_0 + R\cos\theta, \, y_0 + R\sin\theta)\, d\theta$$

```
Mean Value Property:

              ╭────────────╮
           ╱  Average of φ  ╲
         ╱   on this circle   ╲
        │    ╭─────────╮       │
        │    │         │       │
        │    │   •z₀   │ R     │
        │    │  φ(z₀)  │       │
        │    ╰─────────╯       │
         ╲                   ╱
           ╲               ╱
              ╰────────────╯

  φ(z₀) = average of φ on the circle of radius R
```

### 4.2 Maximum Principle for Harmonic Functions

> A non-constant harmonic function on a domain $D$ cannot attain its maximum or minimum value at any interior point of $D$.

**Corollary**: If $D$ is bounded and $\phi$ is continuous on $\bar{D}$ and harmonic on $D$, then:

$$\max_{\bar{D}} \phi = \max_{\partial D} \phi \quad \text{and} \quad \min_{\bar{D}} \phi = \min_{\partial D} \phi$$

(Maxima and minima occur on the boundary.)

### 4.3 Uniqueness of Solutions

> **Dirichlet Problem**: Given a domain $D$ and boundary values $\phi = g$ on $\partial D$, there is at most one harmonic function on $D$ with those boundary values.

(Existence requires additional conditions on $D$ and $g$.)

---

## 5. The Dirichlet Problem

### 5.1 Statement

Find $\phi(x,y)$ harmonic in $D$ with prescribed boundary values:

$$\begin{cases} \nabla^2 \phi = 0 & \text{in } D \\ \phi = g & \text{on } \partial D \end{cases}$$

### 5.2 Solution via Conformal Mapping

For domains that can be conformally mapped to the unit disk, the solution uses:

1. Map $D$ to the unit disk via analytic $w = f(z)$
2. Solve the Dirichlet problem on the disk (Poisson integral formula)
3. Map back

### 5.3 Poisson Integral Formula (for the disk)

For the unit disk $|z| < 1$ with boundary values $\phi(e^{i\theta}) = g(\theta)$:

$$\phi(re^{i\alpha}) = \frac{1}{2\pi}\int_0^{2\pi} \frac{1 - r^2}{1 - 2r\cos(\alpha - \theta) + r^2}\, g(\theta)\, d\theta$$

---

## 6. Laplace's Equation in Polar Coordinates

$$\nabla^2 \phi = \frac{\partial^2 \phi}{\partial r^2} + \frac{1}{r}\frac{\partial \phi}{\partial r} + \frac{1}{r^2}\frac{\partial^2 \phi}{\partial \theta^2} = 0$$

### 6.1 Solutions by Separation of Variables

Seeking $\phi(r,\theta) = R(r)\Theta(\theta)$ leads to:

$$R(r) = Ar^n + Br^{-n}, \qquad \Theta(\theta) = C\cos(n\theta) + D\sin(n\theta)$$

for $n \ge 1$, and $R(r) = A + B\ln r$, $\Theta = 1$ for $n = 0$.

---

## 7. Design Example — Steady-State Temperature

### Problem: Find the steady-state temperature in the upper half-plane $y > 0$ if $\phi(x, 0) = 1$ for $|x| < 1$ and $\phi(x, 0) = 0$ for $|x| > 1$.

```
Temperature Distribution:

    y ▲
      │     Harmonic: ∇²φ = 0
      │
      │   Isotherms (lines of
      │   constant temperature)
      │   curve smoothly
      │
  ────┼───┤  φ=1  ├───────► x
      │  -1       1
      │    φ=0        φ=0
```

**Solution**: Using the Poisson integral for the half-plane:

$$\phi(x,y) = \frac{1}{\pi}\left[\arctan\frac{1-x}{y} + \arctan\frac{1+x}{y}\right]$$

This is the imaginary part of a suitable analytic function (details require conformal mapping techniques from Unit 7).

---

## 8. Real-World Applications

| Application | Physical Quantity | Domain |
|-------------|-------------------|--------|
| **Electrostatics** | Electric potential (voltage) | Region between conductors |
| **Heat conduction** | Steady-state temperature | Conducting plate |
| **Fluid dynamics** | Velocity potential | Irrotational flow region |
| **Gravity** | Gravitational potential | Space outside masses |
| **Elasticity** | Airy stress function | Elastic body |
| **Diffusion** | Steady-state concentration | Diffusion region |

```
Electrostatics Example — Parallel Plate Capacitor:

    ────────────────────  φ = V  (top plate)
    │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒│
    │▒▒▒ ∇²φ = 0  ▒▒▒│  (dielectric between plates)
    │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒│
    ────────────────────  φ = 0  (bottom plate)

    Solution: φ = V·y/d (linear in y)
    This is harmonic: φ_xx + φ_yy = 0 + 0 = 0 ✓
```

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Laplace's equation | $\nabla^2 \phi = \phi_{xx} + \phi_{yy} = 0$ |
| Harmonic function | Solution of Laplace's equation with $C^2$ partials |
| Analytic → Harmonic | $\text{Re}(f)$ and $\text{Im}(f)$ are harmonic |
| Harmonic conjugate | $v$ such that $u + iv$ is analytic; unique up to constant |
| Mean value property | $\phi(z_0) = $ average on any circle centered at $z_0$ |
| Maximum principle | No max/min at interior points (if non-constant) |
| Dirichlet problem | Find harmonic $\phi$ with prescribed boundary values |
| Poisson formula | Solves Dirichlet on the disk |
| Physical meaning | Potential (electric, gravitational, thermal, velocity) |

---

## Quick Revision Questions

1. **Define** a harmonic function and state Laplace's equation in both Cartesian and polar form.

2. **Verify** that $\phi = \ln\sqrt{x^2 + y^2}$ is harmonic for $(x,y) \ne (0,0)$ and identify the analytic function it comes from.

3. **Find** the harmonic conjugate of $u = e^x \cos y$.

4. **State** the mean value property and explain its intuitive meaning.

5. **Why** does the maximum principle for harmonic functions have important physical implications for temperature distributions?

6. **Solve** the Dirichlet problem: find a harmonic function on the unit disk with boundary value $\phi = \cos\theta$ on $|z| = 1$.

---

[← Previous: Analytic Functions](05-analytic-functions.md) | [Next: Unit 3 — Exponential Function →](../03-Elementary-Functions/01-exponential-function.md)
