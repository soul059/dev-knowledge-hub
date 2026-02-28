# 2.4 Cauchy–Riemann Equations

[← Previous: Differentiability](03-differentiability.md) | [Next: Analytic Functions →](05-analytic-functions.md)

---

## Chapter Overview

The **Cauchy–Riemann (C–R) equations** are the bridge between complex differentiability and real partial derivatives. They provide a practical test for analyticity and reveal the deep geometric structure underlying analytic functions — the fact that the real and imaginary parts are intimately coupled.

---

## 1. Derivation

### 1.1 Setup

Let $f(z) = u(x,y) + iv(x,y)$ and suppose $f'(z_0)$ exists. Then:

$$f'(z_0) = \lim_{\Delta z \to 0} \frac{f(z_0 + \Delta z) - f(z_0)}{\Delta z}$$

Since this limit must be the same for **every** approach direction, we examine two:

### 1.2 Approach Along the Real Axis ($\Delta z = h \in \mathbb{R}$)

$$f'(z_0) = \lim_{h \to 0} \frac{u(x_0+h, y_0) - u(x_0, y_0)}{h} + i\,\frac{v(x_0+h, y_0) - v(x_0, y_0)}{h}$$

$$= u_x(x_0, y_0) + i\,v_x(x_0, y_0)$$

### 1.3 Approach Along the Imaginary Axis ($\Delta z = ik$, $k \in \mathbb{R}$)

$$f'(z_0) = \lim_{k \to 0} \frac{u(x_0, y_0+k) - u(x_0, y_0)}{ik} + i\,\frac{v(x_0, y_0+k) - v(x_0, y_0)}{ik}$$

$$= \frac{u_y}{i} + v_y = v_y - iu_y$$

### 1.4 Equating

Setting the two expressions equal:

$$u_x + iv_x = v_y - iu_y$$

Equating real and imaginary parts:

$$\boxed{u_x = v_y \qquad \text{and} \qquad u_y = -v_x}$$

These are the **Cauchy–Riemann equations**.

---

## 2. The Theorem (Necessary and Sufficient Conditions)

### 2.1 Necessary Condition

> **Theorem (Necessary).** If $f(z) = u + iv$ is differentiable at $z_0$, then $u_x, u_y, v_x, v_y$ exist at $(x_0, y_0)$ and satisfy the C–R equations.

### 2.2 Sufficient Condition

> **Theorem (Sufficient).** If $u_x, u_y, v_x, v_y$ exist in a neighborhood of $(x_0, y_0)$, are **continuous** there, and satisfy the C–R equations at $(x_0, y_0)$, then $f$ is differentiable at $z_0$.

### 2.3 Important Distinction

| Condition | C–R equations hold | Partials continuous | $f$ differentiable? |
|-----------|-------------------|--------------------|--------------------|
| C–R necessary | ← guaranteed | — | assumed ✓ |
| C–R + cont. partials | assumed ✓ | assumed ✓ | guaranteed ✓ |
| C–R alone | assumed ✓ | maybe not | **not guaranteed** |

**Subtlety**: The C–R equations at a point, without continuity of the partial derivatives, do NOT guarantee differentiability.

---

## 3. Computing $f'(z)$ via C–R

Once we know $f$ is differentiable, the derivative can be expressed as:

$$f'(z) = u_x + iv_x = v_y - iu_y$$

or mixed:

$$f'(z) = u_x - iu_y = v_y + iv_x$$

---

## 4. Polar Form of C–R Equations

For $z = re^{i\theta}$, write $f(z) = u(r,\theta) + iv(r,\theta)$. The C–R equations become:

$$\boxed{u_r = \frac{1}{r}v_\theta \qquad \text{and} \qquad v_r = -\frac{1}{r}u_\theta}$$

The derivative in polar form:

$$f'(z) = e^{-i\theta}\left(u_r + iv_r\right)$$

---

## 5. Worked Examples

### Example 1: Verify C–R for $f(z) = z^2$

$f(z) = (x+iy)^2 = (x^2 - y^2) + i(2xy)$

So $u = x^2 - y^2$ and $v = 2xy$.

| Derivative | Value | C–R Check |
|-----------|-------|-----------|
| $u_x$ | $2x$ | $u_x = v_y$? $2x = 2x$ ✓ |
| $u_y$ | $-2y$ | $u_y = -v_x$? $-2y = -2y$ ✓ |
| $v_x$ | $2y$ | |
| $v_y$ | $2x$ | |

$f'(z) = u_x + iv_x = 2x + i(2y) = 2(x+iy) = 2z$ ✓

### Example 2: Show $f(z) = \bar{z}$ fails C–R

$f(z) = x - iy$, so $u = x$, $v = -y$.

| Check | Values | Result |
|-------|--------|--------|
| $u_x = v_y$? | $1 = -1$? | **FAILS** ✗ |
| $u_y = -v_x$? | $0 = 0$? | Holds |

C–R equations fail, so $\bar{z}$ is not differentiable. ✓

### Example 3: $f(z) = e^z$

$f(z) = e^x(\cos y + i\sin y)$, so $u = e^x \cos y$, $v = e^x \sin y$.

| Derivative | Value |
|-----------|-------|
| $u_x$ | $e^x \cos y$ |
| $u_y$ | $-e^x \sin y$ |
| $v_x$ | $e^x \sin y$ |
| $v_y$ | $e^x \cos y$ |

$u_x = e^x \cos y = v_y$ ✓ and $u_y = -e^x \sin y = -v_x$ ✓

$f'(z) = u_x + iv_x = e^x \cos y + ie^x \sin y = e^x(\cos y + i\sin y) = e^z$ ✓

---

## 6. Geometric Interpretation

### 6.1 The Jacobian Constraint

The Jacobian of $f = (u,v)$ as a real map is:

$$J = \begin{pmatrix} u_x & u_y \\ v_x & v_y \end{pmatrix}$$

The C–R equations force this to have the special form:

$$J = \begin{pmatrix} a & -b \\ b & a \end{pmatrix} \qquad \text{where } a = u_x, \; b = v_x$$

This matrix represents **rotation by $\theta = \arctan(b/a)$** followed by **scaling by $r = \sqrt{a^2+b^2}$**.

$$J = r \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix}$$

> **Geometric meaning**: An analytic function locally acts as a rotation + scaling — this is exactly why it's **conformal** (angle-preserving).

### 6.2 Determinant

$$\det(J) = u_x v_y - u_y v_x = u_x^2 + v_x^2 = |f'(z)|^2$$

The Jacobian determinant equals the squared modulus of the derivative.

```
Effect of C-R on grid mapping:

z-plane (orthogonal grid):      w-plane (still orthogonal!):

    y ▲  │  │  │  │              v ▲    ╱  ╱  ╱
      │──┼──┼──┼──┼                │  ╱  ╱  ╱
      │──┼──┼──┼──┼                │╱──╱──╱
      │──┼──┼──┼──┼                ╱──╱──╱
      └──┴──┴──┴──┴──► x          ╱──╱──╱──────► u

   C-R equations ensure angles between
   grid lines are preserved (90°→90°)
```

---

## 7. The C–R Equations and Laplace's Equation

If $f = u + iv$ is analytic, differentiating the C–R equations:

$$u_{xx} = v_{yx}, \quad u_{yy} = -v_{xy}$$

Adding (assuming $v_{xy} = v_{yx}$):

$$u_{xx} + u_{yy} = 0$$

Similarly:

$$v_{xx} + v_{yy} = 0$$

> **Both $u$ and $v$ satisfy Laplace's equation** — they are **harmonic functions**. (More in Ch. 2.6.)

---

## 8. Real-World Applications

| Application | Relevance |
|-------------|-----------|
| **Fluid Dynamics** | Velocity potential $\phi$ and stream function $\psi$ satisfy C–R: $\phi_x = \psi_y$, $\phi_y = -\psi_x$ |
| **Electrostatics** | Electric potential and flux function form a C–R pair |
| **Heat Conduction** | Isotherms and heat flow lines are C–R related |
| **Elasticity** | Stress functions in plane elasticity satisfy C–R-like equations |

```
Fluid Flow — C-R Structure:

  Stream function ψ = const        Velocity potential φ = const
  (streamlines)                    (equipotential lines)

   ──────────────────►             │  │  │  │  │  │
   ──────────────────►             │  │  │  │  │  │
   ──────────────────►             │  │  │  │  │  │
   ──────────────────►             │  │  │  │  │  │

   These two families are always ORTHOGONAL
   because φ and ψ satisfy the C-R equations
```

---

## Summary Table

| Concept | Formula / Fact |
|---------|----------------|
| C–R equations (Cartesian) | $u_x = v_y$, $u_y = -v_x$ |
| C–R equations (Polar) | $u_r = \frac{1}{r}v_\theta$, $v_r = -\frac{1}{r}u_\theta$ |
| Necessary condition | $f$ differentiable ⟹ C–R hold |
| Sufficient condition | C–R hold + partials continuous ⟹ $f$ differentiable |
| Derivative formulas | $f' = u_x + iv_x = v_y - iu_y$ |
| Jacobian structure | Rotation–dilation matrix $\begin{pmatrix} a & -b \\ b & a \end{pmatrix}$ |
| $\det(J)$ | $= \|f'(z)\|^2$ |
| Harmonic consequence | $u_{xx} + u_{yy} = 0$ and $v_{xx} + v_{yy} = 0$ |

---

## Quick Revision Questions

1. **Derive** the C–R equations from the definition of the complex derivative.

2. **Verify** the C–R equations for $f(z) = z^3$ and compute $f'(z)$ via partial derivatives.

3. **Show** that $f(z) = e^{\bar{z}}$ is not analytic anywhere.

4. **Derive** the polar form of the C–R equations from the Cartesian form using $x = r\cos\theta$, $y = r\sin\theta$.

5. **Explain** the geometric meaning of the Jacobian of an analytic function being a rotation–dilation matrix.

6. **Why** do both $u$ and $v$ satisfy Laplace's equation if $f = u + iv$ is analytic?

---

[← Previous: Differentiability](03-differentiability.md) | [Next: Analytic Functions →](05-analytic-functions.md)
