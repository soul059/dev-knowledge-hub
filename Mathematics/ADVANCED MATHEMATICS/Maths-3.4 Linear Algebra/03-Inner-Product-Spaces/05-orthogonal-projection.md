# 3.5 Orthogonal Projection

[← Previous: Gram-Schmidt Process](04-gram-schmidt-process.md) · [Back to README](../README.md) · [Next: Orthonormal Basis →](06-orthonormal-basis.md)

---

## Chapter Overview

**Orthogonal projection** answers: "What is the closest point in a subspace $W$ to a given vector $\mathbf{v}$?" This gives the **best approximation** of $\mathbf{v}$ by elements of $W$, and is the theoretical basis for least-squares fitting, Fourier series, and regression.

---

## 1. Projection onto a Line (1-D Subspace)

The **orthogonal projection** of $\mathbf{v}$ onto $\mathbf{u}$ ($\mathbf{u} \neq \mathbf{0}$):

$$\text{proj}_{\mathbf{u}}(\mathbf{v}) = \frac{\langle \mathbf{v}, \mathbf{u} \rangle}{\langle \mathbf{u}, \mathbf{u} \rangle} \mathbf{u}$$

```
        ▲ v
        │╲
        │  ╲
        │    ╲  v - proj_u(v)  (the residual, ⊥ u)
        │     ╲
  ──────┼──────╲──────▶ u
        │  proj_u(v)
        │  (shadow of v on u)
```

The residual $\mathbf{v} - \text{proj}_{\mathbf{u}}(\mathbf{v})$ is orthogonal to $\mathbf{u}$.

---

## 2. Projection onto a Subspace

**Definition.** Let $W$ be a subspace of an inner product space $V$. The **orthogonal projection** of $\mathbf{v}$ onto $W$ is the unique vector $\hat{\mathbf{v}} \in W$ such that $\mathbf{v} - \hat{\mathbf{v}} \in W^\perp$.

$$\mathbf{v} = \underbrace{\hat{\mathbf{v}}}_{\in W} + \underbrace{(\mathbf{v} - \hat{\mathbf{v}})}_{\in W^\perp}$$

```
                     • v
                    ╱│
                  ╱  │
                ╱    │  v - v̂  (⊥ to W)
              ╱      │
            ╱        │
  ─────────•─────────┼──── W (subspace)
           v̂ = proj_W(v)
           (closest point in W)
```

---

## 3. Formula Using Orthogonal Basis

If $\{\mathbf{u}_1, \dots, \mathbf{u}_k\}$ is an **orthogonal** basis for $W$:

$$\text{proj}_W(\mathbf{v}) = \sum_{i=1}^{k} \frac{\langle \mathbf{v}, \mathbf{u}_i \rangle}{\langle \mathbf{u}_i, \mathbf{u}_i \rangle} \mathbf{u}_i$$

If the basis is **orthonormal**:

$$\text{proj}_W(\mathbf{v}) = \sum_{i=1}^{k} \langle \mathbf{v}, \mathbf{u}_i \rangle\, \mathbf{u}_i$$

---

## 4. Formula Using Any Basis (Projection Matrix)

If $W = \text{Col}(A)$ for some $m \times k$ matrix $A$ with linearly independent columns:

$$\text{proj}_W(\mathbf{v}) = A(A^T A)^{-1} A^T \mathbf{v}$$

The **projection matrix** is:

$$P = A(A^T A)^{-1} A^T$$

### Properties of the Projection Matrix

| Property | Statement |
|----------|-----------|
| Idempotent | $P^2 = P$ (projecting twice is the same as once) |
| Symmetric | $P^T = P$ |
| Rank | $\text{rank}(P) = k = \dim(W)$ |
| Eigenvalues | Only 0 and 1 |
| Complement | $I - P$ projects onto $W^\perp$ |

---

## 5. Worked Example 1: Projection onto a Plane in $\mathbb{R}^3$

Project $\mathbf{v} = (1, 2, 3)$ onto $W = \text{span}\{(1, 0, 1), (0, 1, 1)\}$.

**Using orthogonal basis (Gram-Schmidt first):**

$\mathbf{u}_1 = (1, 0, 1)$

$\mathbf{u}_2 = (0, 1, 1) - \frac{\langle (0,1,1), (1,0,1) \rangle}{\langle (1,0,1), (1,0,1) \rangle}(1, 0, 1) = (0,1,1) - \frac{1}{2}(1,0,1) = (-\frac{1}{2}, 1, \frac{1}{2})$

Now project:

$\frac{\langle \mathbf{v}, \mathbf{u}_1 \rangle}{\|\mathbf{u}_1\|^2} = \frac{1+0+3}{2} = 2$

$\frac{\langle \mathbf{v}, \mathbf{u}_2 \rangle}{\|\mathbf{u}_2\|^2} = \frac{-1/2+2+3/2}{1/4+1+1/4} = \frac{3}{3/2} = 2$

$$\hat{\mathbf{v}} = 2(1,0,1) + 2(-\frac{1}{2}, 1, \frac{1}{2}) = (2,0,2) + (-1, 2, 1) = (1, 2, 3)$$

In this case, $\mathbf{v}$ is already in $W$! (The residual is $\mathbf{0}$.)

---

## 6. Worked Example 2: Best Approximation

Project $\mathbf{v} = (1, 0, 0)$ onto $W = \text{span}\{(1, 1, 0), (0, 1, 1)\}$.

$\mathbf{u}_1 = (1, 1, 0)$, $\mathbf{u}_2 = (0,1,1) - \frac{1}{2}(1,1,0) = (-\frac{1}{2}, \frac{1}{2}, 1)$

$\frac{\langle \mathbf{v}, \mathbf{u}_1 \rangle}{\|\mathbf{u}_1\|^2} = \frac{1}{2}$

$\frac{\langle \mathbf{v}, \mathbf{u}_2 \rangle}{\|\mathbf{u}_2\|^2} = \frac{-1/2}{3/2} = -\frac{1}{3}$

$$\hat{\mathbf{v}} = \frac{1}{2}(1,1,0) + \left(-\frac{1}{3}\right)\left(-\frac{1}{2}, \frac{1}{2}, 1\right) = \left(\frac{1}{2}, \frac{1}{2}, 0\right) + \left(\frac{1}{6}, -\frac{1}{6}, -\frac{1}{3}\right) = \left(\frac{2}{3}, \frac{1}{3}, -\frac{1}{3}\right)$$

**Error:**

$$\mathbf{v} - \hat{\mathbf{v}} = \left(\frac{1}{3}, -\frac{1}{3}, \frac{1}{3}\right), \quad \|\mathbf{v}-\hat{\mathbf{v}}\| = \frac{1}{\sqrt{3}}$$

---

## 7. The Best Approximation Theorem

> **Theorem.** Let $W$ be a finite-dimensional subspace of an inner product space $V$, and $\mathbf{v} \in V$. Then $\text{proj}_W(\mathbf{v})$ is the **unique closest point** in $W$ to $\mathbf{v}$:
>
> $$\|\mathbf{v} - \text{proj}_W(\mathbf{v})\| \leq \|\mathbf{v} - \mathbf{w}\| \quad \text{for all } \mathbf{w} \in W$$
>
> with equality iff $\mathbf{w} = \text{proj}_W(\mathbf{v})$.

```
  Distance from v to W:
  
                 • v
                 │
    ‖v - v̂‖     │  ← shortest distance (perpendicular)
                 │
  ───────────────•───── W
                 v̂
  
  Any other w ∈ W gives ‖v - w‖ ≥ ‖v - v̂‖
```

---

## 8. Real-World Applications

| Application | Projection Idea |
|-------------|-----------------|
| **Least squares** | Project $\mathbf{b}$ onto $\text{Col}(A)$ → best approximate solution of $A\mathbf{x} = \mathbf{b}$ |
| **Fourier series** | Project a function onto $\text{span}\{\sin(nx), \cos(nx)\}$ |
| **Regression** | Project data onto the column space of the design matrix |
| **Signal denoising** | Project noisy signal onto a low-dimensional subspace |
| **Computer graphics** | Project 3D objects onto 2D screen plane |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| $\text{proj}_{\mathbf{u}}(\mathbf{v})$ | $\frac{\langle \mathbf{v},\mathbf{u}\rangle}{\langle \mathbf{u},\mathbf{u}\rangle}\mathbf{u}$ |
| $\text{proj}_W(\mathbf{v})$ | Sum of projections onto orthogonal basis vectors of $W$ |
| Projection matrix | $P = A(A^TA)^{-1}A^T$ |
| $P^2 = P$ | Projecting twice = once |
| Best approximation | $\text{proj}_W(\mathbf{v})$ minimises $\|\mathbf{v} - \mathbf{w}\|$ over $\mathbf{w} \in W$ |
| Residual | $\mathbf{v} - \hat{\mathbf{v}} \in W^\perp$ |
| Decomposition | $\mathbf{v} = \text{proj}_W(\mathbf{v}) + \text{proj}_{W^\perp}(\mathbf{v})$ |

---

## Quick Revision Questions

1. Project $(3, 4)$ onto the line spanned by $(1, 2)$ in $\mathbb{R}^2$.

2. Find the projection matrix for the subspace $W = \text{span}\{(1, 1, 1)^T\}$ in $\mathbb{R}^3$.

3. Prove that the projection matrix $P$ satisfies $P^2 = P$.

4. Why is $\text{proj}_W(\mathbf{v})$ the closest point in $W$ to $\mathbf{v}$?

5. If $\mathbf{v} \in W$, what is $\text{proj}_W(\mathbf{v})$?

6. Compute the distance from $(1, 1, 1)$ to the plane $W = \text{span}\{(1, 0, 0), (0, 1, 0)\}$.

---

[← Previous: Gram-Schmidt Process](04-gram-schmidt-process.md) · [Back to README](../README.md) · [Next: Orthonormal Basis →](06-orthonormal-basis.md)
