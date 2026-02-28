# 3.2 Norm and Distance

[← Previous: Inner Product Definition](01-inner-product-definition.md) · [Back to README](../README.md) · [Next: Orthogonality →](03-orthogonality.md)

---

## Chapter Overview

An inner product naturally induces a **norm** (length) and a **distance** (metric). This chapter defines the induced norm, states the Cauchy-Schwarz and Triangle inequalities, and covers the geometry of norms.

---

## 1. Induced Norm

**Definition.** If $\langle \cdot, \cdot \rangle$ is an inner product on $V$, the **induced norm** is:

$$\|\mathbf{v}\| = \sqrt{\langle \mathbf{v}, \mathbf{v} \rangle}$$

For the standard inner product: $\|\mathbf{v}\| = \sqrt{v_1^2 + v_2^2 + \cdots + v_n^2}$ (Euclidean length).

A vector with $\|\mathbf{v}\| = 1$ is called a **unit vector**.

---

## 2. Properties of a Norm

A function $\|\cdot\| : V \to \mathbb{R}$ is a norm if:

| # | Property | Statement |
|---|----------|-----------|
| N1 | Non-negativity | $\|\mathbf{v}\| \geq 0$, equality iff $\mathbf{v} = \mathbf{0}$ |
| N2 | Homogeneity | $\|c\mathbf{v}\| = |c| \cdot \|\mathbf{v}\|$ |
| N3 | Triangle inequality | $\|\mathbf{u} + \mathbf{v}\| \leq \|\mathbf{u}\| + \|\mathbf{v}\|$ |

Every induced norm satisfies these. (Not every norm comes from an inner product — the $\ell^1$ and $\ell^\infty$ norms don't.)

---

## 3. The Cauchy-Schwarz Inequality

> **Theorem (Cauchy-Schwarz).** For any $\mathbf{u}, \mathbf{v}$ in an inner product space:
>
> $$|\langle \mathbf{u}, \mathbf{v} \rangle| \leq \|\mathbf{u}\| \cdot \|\mathbf{v}\|$$
>
> Equality holds iff $\mathbf{u}$ and $\mathbf{v}$ are **linearly dependent**.

**Proof sketch:** For any $t \in \mathbb{R}$:

$$0 \leq \|\mathbf{u} - t\mathbf{v}\|^2 = \|\mathbf{u}\|^2 - 2t\langle \mathbf{u},\mathbf{v}\rangle + t^2\|\mathbf{v}\|^2$$

This is a quadratic in $t$ that is $\geq 0$, so its discriminant $\leq 0$:

$$4\langle \mathbf{u},\mathbf{v}\rangle^2 - 4\|\mathbf{u}\|^2 \|\mathbf{v}\|^2 \leq 0$$

$$\implies \langle \mathbf{u},\mathbf{v}\rangle^2 \leq \|\mathbf{u}\|^2 \|\mathbf{v}\|^2 \quad \blacksquare$$

### Specific Instances

| Setting | Cauchy-Schwarz Becomes |
|---------|----------------------|
| $\mathbb{R}^n$ (dot product) | $\left\|\sum u_i v_i\right\| \leq \sqrt{\sum u_i^2} \cdot \sqrt{\sum v_i^2}$ |
| $C[a,b]$ | $\left\|\int_a^b f(x)g(x)dx\right\| \leq \sqrt{\int f^2 dx} \cdot \sqrt{\int g^2 dx}$ |

---

## 4. The Triangle Inequality

> **Theorem.** $\|\mathbf{u} + \mathbf{v}\| \leq \|\mathbf{u}\| + \|\mathbf{v}\|$

**Proof:**

$$\|\mathbf{u}+\mathbf{v}\|^2 = \|\mathbf{u}\|^2 + 2\langle\mathbf{u},\mathbf{v}\rangle + \|\mathbf{v}\|^2 \leq \|\mathbf{u}\|^2 + 2\|\mathbf{u}\|\|\mathbf{v}\| + \|\mathbf{v}\|^2 = (\|\mathbf{u}\|+\|\mathbf{v}\|)^2$$

```
  Triangle Inequality — geometric view:
  
        ▲ u+v
       ╱│
      ╱ │
   u ╱  │ v
    ╱   │
   ╱    │
  •─────▶
  
  The "shortcut" (u+v) is never longer
  than going along u then v.
  
  ‖u+v‖ ≤ ‖u‖ + ‖v‖
```

---

## 5. Distance (Metric)

**Definition.** The **distance** between $\mathbf{u}$ and $\mathbf{v}$ is:

$$d(\mathbf{u}, \mathbf{v}) = \|\mathbf{u} - \mathbf{v}\|$$

Properties (making $V$ a **metric space**):

| # | Property | Statement |
|---|----------|-----------|
| M1 | Non-negativity | $d(\mathbf{u},\mathbf{v}) \geq 0$, $= 0$ iff $\mathbf{u}=\mathbf{v}$ |
| M2 | Symmetry | $d(\mathbf{u},\mathbf{v}) = d(\mathbf{v},\mathbf{u})$ |
| M3 | Triangle inequality | $d(\mathbf{u},\mathbf{w}) \leq d(\mathbf{u},\mathbf{v}) + d(\mathbf{v},\mathbf{w})$ |

---

## 6. Angle Between Vectors

By Cauchy-Schwarz, $-1 \leq \frac{\langle \mathbf{u}, \mathbf{v} \rangle}{\|\mathbf{u}\|\|\mathbf{v}\|} \leq 1$, so we can define:

$$\cos\theta = \frac{\langle \mathbf{u}, \mathbf{v} \rangle}{\|\mathbf{u}\| \cdot \|\mathbf{v}\|}$$

where $\theta \in [0, \pi]$ is the **angle** between $\mathbf{u}$ and $\mathbf{v}$.

```
           ╲ u
            ╲
             ╲ θ
              ╲───
               ╲  │
                ╲ │
                 ╲│
  ────────────────•────────────  v
  
  cos θ = ⟨u,v⟩ / (‖u‖·‖v‖)
  
  θ = 0°  : u ∥ v (same direction)
  θ = 90° : u ⊥ v (perpendicular)
  θ = 180°: u ∥ v (opposite direction)
```

---

## 7. Common Norm Families (Comparison)

While the inner-product-induced norm is the $\ell^2$ norm, other norms exist:

| Norm | Symbol | Formula (in $\mathbb{R}^n$) | Unit Ball Shape |
|------|--------|----------------------------|----------------|
| $\ell^1$ (Manhattan) | $\|\mathbf{x}\|_1$ | $\sum |x_i|$ | Diamond |
| $\ell^2$ (Euclidean) | $\|\mathbf{x}\|_2$ | $\sqrt{\sum x_i^2}$ | Circle |
| $\ell^\infty$ (Max) | $\|\mathbf{x}\|_\infty$ | $\max |x_i|$ | Square |

```
  Unit balls in ℝ²:
  
    ℓ¹                ℓ²                 ℓ∞
    ◇                 ○                  □
   ╱ ╲              ╱   ╲             ┌───┐
  ╱   ╲            │     │            │   │
  ╲   ╱            │     │            │   │
   ╲ ╱              ╲   ╱             └───┘
    ◇                 ○  
```

> Only the $\ell^2$ norm comes from an inner product (the **parallelogram law** test).

---

## 8. The Parallelogram Law

$$\|\mathbf{u} + \mathbf{v}\|^2 + \|\mathbf{u} - \mathbf{v}\|^2 = 2\|\mathbf{u}\|^2 + 2\|\mathbf{v}\|^2$$

This holds for **every** inner-product-induced norm. Conversely, if a norm satisfies this, it comes from an inner product (recovered via the **polarisation identity**):

$$\langle \mathbf{u}, \mathbf{v} \rangle = \frac{1}{4}\left(\|\mathbf{u}+\mathbf{v}\|^2 - \|\mathbf{u}-\mathbf{v}\|^2\right)$$

---

## 9. Real-World Applications

| Application | Norm/Distance Used |
|-------------|-------------------|
| GPS / navigation | Euclidean distance ($\ell^2$) |
| City block distance | Manhattan distance ($\ell^1$) |
| Error analysis | Root-mean-square = $\ell^2$ norm / $\sqrt{n}$ |
| Signal processing | Energy of signal = $\|f\|^2 = \int |f(t)|^2 dt$ |
| Machine learning | Cosine similarity = $\frac{\langle u,v \rangle}{\|u\| \|v\|}$ |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Induced norm | $\|\mathbf{v}\| = \sqrt{\langle \mathbf{v}, \mathbf{v} \rangle}$ |
| Unit vector | $\|\mathbf{v}\| = 1$; normalise by $\hat{\mathbf{v}} = \mathbf{v}/\|\mathbf{v}\|$ |
| Cauchy-Schwarz | $|\langle \mathbf{u}, \mathbf{v}\rangle| \leq \|\mathbf{u}\| \|\mathbf{v}\|$ |
| Triangle inequality | $\|\mathbf{u}+\mathbf{v}\| \leq \|\mathbf{u}\| + \|\mathbf{v}\|$ |
| Distance | $d(\mathbf{u}, \mathbf{v}) = \|\mathbf{u} - \mathbf{v}\|$ |
| Angle | $\cos\theta = \frac{\langle \mathbf{u},\mathbf{v}\rangle}{\|\mathbf{u}\|\|\mathbf{v}\|}$ |
| Parallelogram law | Characterises inner-product norms |

---

## Quick Revision Questions

1. Compute $\|(3, -4)\|$ using the standard inner product on $\mathbb{R}^2$.

2. State and prove the Cauchy-Schwarz inequality.

3. Verify the triangle inequality for $\mathbf{u} = (1,2)$ and $\mathbf{v} = (3, -1)$.

4. Find the angle between $(1, 1, 1)$ and $(1, 0, -1)$ in $\mathbb{R}^3$.

5. Does the $\ell^1$ norm on $\mathbb{R}^2$ come from an inner product? Justify using the parallelogram law.

6. Compute the distance between $f(x) = x$ and $g(x) = x^2$ using the inner product $\langle f,g \rangle = \int_0^1 fg\,dx$.

---

[← Previous: Inner Product Definition](01-inner-product-definition.md) · [Back to README](../README.md) · [Next: Orthogonality →](03-orthogonality.md)
