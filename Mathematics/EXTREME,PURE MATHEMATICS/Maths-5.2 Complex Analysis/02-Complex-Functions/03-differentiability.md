# 2.3 Differentiability

[← Previous: Limits and Continuity](02-limits-and-continuity.md) | [Next: Cauchy–Riemann Equations →](04-cauchy-riemann-equations.md)

---

## Chapter Overview

The definition of the complex derivative looks identical to the real derivative, but its consequences are vastly more powerful. Because the limit must exist regardless of direction in the plane, complex differentiability is an extraordinarily strong condition — far more restrictive than real differentiability. A function that is complex-differentiable is called **holomorphic** (or **analytic**), and it automatically possesses remarkable properties.

---

## 1. Definition of the Complex Derivative

### 1.1 The Derivative

The **derivative** of $f$ at $z_0$ is defined as:

$$f'(z_0) = \lim_{z \to z_0} \frac{f(z) - f(z_0)}{z - z_0} = \lim_{\Delta z \to 0} \frac{f(z_0 + \Delta z) - f(z_0)}{\Delta z}$$

provided this limit exists. Here $\Delta z \in \mathbb{C}$ may approach $0$ from **any direction**.

### 1.2 Critical Observation

In $\mathbb{R}$: $\Delta x$ approaches $0$ along the number line (from left or right).

In $\mathbb{C}$: $\Delta z$ approaches $0$ from **any direction** in the plane — infinitely many paths.

```
Difference quotient — direction dependence:

         Im
          ▲     Δz = Δx
          │   ← horizontal approach
          │
    Δz ╲  │  ╱ Δz
        ╲ │ ╱
         ╲│╱ ← all must give same limit
    ──────•z₀──────► Re
         ╱│╲
        ╱ │ ╲
    Δz ╱  │  ╲ Δz
          │
          │   ← vertical approach
          │     Δz = iΔy
```

> The requirement that the difference quotient converges to the same value along **every** path is what makes complex differentiability so powerful.

---

## 2. Differentiation Rules

All the familiar rules carry over from real calculus:

| Rule | Formula |
|------|---------|
| Constant | $(c)' = 0$ |
| Power | $(z^n)' = nz^{n-1}$ |
| Sum | $(f + g)' = f' + g'$ |
| Product | $(fg)' = f'g + fg'$ |
| Quotient | $(f/g)' = \frac{f'g - fg'}{g^2}$ |
| Chain | $(f \circ g)'(z) = f'(g(z)) \cdot g'(z)$ |

### 2.1 Examples

| $f(z)$ | $f'(z)$ |
|--------|---------|
| $z^3$ | $3z^2$ |
| $1/z$ | $-1/z^2$ |
| $z^2 + 3z + 2$ | $2z + 3$ |
| $\frac{z}{z+1}$ | $\frac{1}{(z+1)^2}$ |

---

## 3. Functions That Are NOT Complex-Differentiable

### 3.1 $f(z) = \bar{z}$

Compute the difference quotient:

$$\frac{f(z_0 + \Delta z) - f(z_0)}{\Delta z} = \frac{\overline{z_0 + \Delta z} - \bar{z_0}}{\Delta z} = \frac{\overline{\Delta z}}{\Delta z}$$

**Along the real axis**: $\Delta z = h \in \mathbb{R}$, so $\frac{\bar{h}}{h} = 1$

**Along the imaginary axis**: $\Delta z = ik$, so $\frac{\overline{ik}}{ik} = \frac{-ik}{ik} = -1$

Since $1 \ne -1$, the limit does not exist. **$\bar{z}$ is nowhere differentiable.**

### 3.2 $f(z) = |z|^2 = z\bar{z}$

$$\frac{|z_0 + \Delta z|^2 - |z_0|^2}{\Delta z}$$

At $z_0 = 0$: $\frac{|\Delta z|^2}{\Delta z} = \frac{\Delta z \cdot \overline{\Delta z}}{\Delta z} = \overline{\Delta z} \to 0$

At $z_0 \ne 0$: can show the limit depends on direction.

> $f(z) = |z|^2$ is differentiable **only at $z_0 = 0$**, with $f'(0) = 0$.

### 3.3 Pattern — Which Functions Are Differentiable?

| Function | Differentiable? | Where? |
|----------|----------------|--------|
| $z^n$ | Yes | Everywhere |
| $\bar{z}$ | No | Nowhere |
| $\|z\|^2$ | Yes | Only at $z = 0$ |
| $\text{Re}(z) = x$ | No | Nowhere |
| $\text{Im}(z) = y$ | No | Nowhere |
| $z\bar{z}$ | Yes | Only at $z = 0$ |
| $e^z$ | Yes | Everywhere |

> **Lesson**: Functions involving $\bar{z}$, $|z|$, $\text{Re}(z)$, or $\text{Im}(z)$ explicitly are typically not complex-differentiable (except possibly at isolated points).

---

## 4. Differentiability vs. Real Differentiability

### 4.1 Real-Differentiable Functions

$f: \mathbb{R}^2 \to \mathbb{R}^2$ given by $(x,y) \mapsto (u(x,y), v(x,y))$ is **real-differentiable** at $(x_0, y_0)$ if the Jacobian matrix exists:

$$J_f = \begin{pmatrix} u_x & u_y \\ v_x & v_y \end{pmatrix}$$

This is a **much weaker** condition than complex differentiability.

### 4.2 The Hierarchy

```
                Complex-differentiable (holomorphic)
                          ⊂
                Real-differentiable (Jacobian exists)
                          ⊂
                Continuous
                          ⊂
                Defined

  Complex differentiability ⟹ real differentiability ⟹ continuity
  But NONE of the reverse implications hold!
```

### 4.3 What Extra Condition Is Needed?

A real-differentiable function $f = u + iv$ is complex-differentiable **if and only if** the Cauchy–Riemann equations hold (next chapter):

$$u_x = v_y, \qquad u_y = -v_x$$

---

## 5. Geometric Meaning of the Derivative

### 5.1 Conformal Property

If $f'(z_0) \ne 0$, then $f$ is **conformal** (angle-preserving) at $z_0$:

- Two curves meeting at angle $\alpha$ at $z_0$ are mapped to curves meeting at angle $\alpha$ at $f(z_0)$
- The local magnification factor is $|f'(z_0)|$
- The local rotation angle is $\arg f'(z_0)$

```
Conformal mapping at z₀ (f'(z₀) ≠ 0):

  z-plane:                      w-plane:
       γ₂                           f(γ₂)
      ╱                             ╱
     ╱ α                           ╱ α  (same angle!)
    •z₀                           •f(z₀)
     ╲                             ╲
      ╲                             ╲
       γ₁                           f(γ₁)

  Angle α is preserved (conformal)
  Lengths scaled by |f'(z₀)|
  Directions rotated by arg(f'(z₀))
```

### 5.2 Where Conformality Fails

At points where $f'(z_0) = 0$ (called **critical points**), angles are multiplied. If $f(z) - f(z_0) \sim c(z - z_0)^k$ near $z_0$, then angles are multiplied by $k$.

---

## 6. Higher Derivatives

If $f$ is complex-differentiable in a neighborhood of $z_0$ (i.e., **analytic**), then $f$ has derivatives of **all orders**:

$$f'(z_0), \quad f''(z_0), \quad f'''(z_0), \quad \ldots$$

all exist. This is **dramatically** different from real analysis, where a function can be differentiable once but not twice.

> **Remarkable fact**: Complex differentiability (once) implies infinite differentiability. (Proof in Unit 4 via Cauchy integral formula.)

---

## 7. Design Example

### Example: Show that $f(z) = z^3$ is differentiable and find $f'(z)$ from the definition.

**Step 1.** Compute the difference quotient:
$$\frac{(z + \Delta z)^3 - z^3}{\Delta z}$$

**Step 2.** Expand $(z + \Delta z)^3 = z^3 + 3z^2 \Delta z + 3z(\Delta z)^2 + (\Delta z)^3$

**Step 3.** 
$$\frac{3z^2 \Delta z + 3z(\Delta z)^2 + (\Delta z)^3}{\Delta z} = 3z^2 + 3z \cdot \Delta z + (\Delta z)^2$$

**Step 4.** Take $\Delta z \to 0$:
$$f'(z) = 3z^2$$

This works regardless of the direction of $\Delta z$, confirming differentiability everywhere.

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Complex derivative | $f'(z_0) = \lim_{\Delta z \to 0} \frac{f(z_0+\Delta z)-f(z_0)}{\Delta z}$ |
| Direction independence | $\Delta z \to 0$ from ANY direction must give same limit |
| Differentiation rules | Same as real calculus (power, product, chain, etc.) |
| $\bar{z}$ | Nowhere differentiable |
| $\|z\|^2$ | Differentiable only at $z = 0$ |
| Hierarchy | Holomorphic ⊂ Real-differentiable ⊂ Continuous |
| Conformal | $f'(z_0) \ne 0 \implies$ angle-preserving at $z_0$ |
| Infinite differentiability | Holomorphic ⟹ $C^\infty$ (proved later) |

---

## Quick Revision Questions

1. **State** the definition of the complex derivative and explain how it differs from the real derivative.

2. **Show** from the definition that $f(z) = 1/z$ has $f'(z) = -1/z^2$.

3. **Prove** that $f(z) = \text{Re}(z)$ is not complex-differentiable at any point.

4. **What** is the geometric interpretation of $|f'(z_0)|$ and $\arg f'(z_0)$?

5. **Explain** why $f(z) = |z|^2$ is differentiable only at $z = 0$.

6. **Why** is the statement "complex differentiability implies infinite differentiability" so surprising compared to real analysis?

---

[← Previous: Limits and Continuity](02-limits-and-continuity.md) | [Next: Cauchy–Riemann Equations →](04-cauchy-riemann-equations.md)
