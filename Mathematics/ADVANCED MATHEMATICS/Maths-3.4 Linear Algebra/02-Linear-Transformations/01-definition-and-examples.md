# 2.1 Linear Transformations — Definition and Examples

[← Previous: Basis and Dimension](../01-Vector-Spaces/06-basis-and-dimension.md) · [Back to README](../README.md) · [Next: Kernel and Range →](02-kernel-and-range.md)

---

## Chapter Overview

A **linear transformation** is a function between vector spaces that preserves the two fundamental operations: addition and scalar multiplication. These are the "structure-preserving maps" of Linear Algebra — analogous to continuous functions in analysis or homomorphisms in algebra. This chapter defines linear transformations, provides numerous examples, and shows how to verify linearity.

---

## 1. Definition

**Definition.** Let $V$ and $W$ be vector spaces over the same field $F$. A function $T : V \to W$ is called a **linear transformation** (or **linear map**) if for all $\mathbf{u}, \mathbf{v} \in V$ and all $c \in F$:

1. **Additivity:** $T(\mathbf{u} + \mathbf{v}) = T(\mathbf{u}) + T(\mathbf{v})$
2. **Homogeneity:** $T(c\mathbf{u}) = cT(\mathbf{u})$

Equivalently (combining both conditions):

$$T(c_1 \mathbf{u} + c_2 \mathbf{v}) = c_1 T(\mathbf{u}) + c_2 T(\mathbf{v})$$

More generally (the **superposition principle**):

$$T\left(\sum_{i=1}^{k} c_i \mathbf{v}_i\right) = \sum_{i=1}^{k} c_i T(\mathbf{v}_i)$$

---

## 2. Diagram — What Linearity Means

```
  Domain V                              Codomain W
  ┌──────────────────┐                ┌──────────────────┐
  │                  │                │                  │
  │  u ──────┐       │      T        │   T(u) ────┐     │
  │          ├─ u+v ─┼──────────────▶│            ├─ T(u)+T(v) = T(u+v)
  │  v ──────┘       │               │   T(v) ────┘     │
  │                  │                │                  │
  │  c·u ────────────┼──────────────▶│   c·T(u) = T(c·u)│
  │                  │                │                  │
  └──────────────────┘                └──────────────────┘
  
  "T respects the vector space structure"
```

---

## 3. Immediate Consequences

If $T: V \to W$ is linear, then:

| Property | Proof |
|----------|-------|
| $T(\mathbf{0}_V) = \mathbf{0}_W$ | $T(\mathbf{0}) = T(0 \cdot \mathbf{v}) = 0 \cdot T(\mathbf{v}) = \mathbf{0}$ |
| $T(-\mathbf{v}) = -T(\mathbf{v})$ | $T(-\mathbf{v}) = T((-1)\mathbf{v}) = (-1)T(\mathbf{v})$ |
| $T(\mathbf{u} - \mathbf{v}) = T(\mathbf{u}) - T(\mathbf{v})$ | Combine additivity with the above |

> **Quick test:** If $T(\mathbf{0}) \neq \mathbf{0}$, then $T$ is **not linear**.

---

## 4. Gallery of Examples

### Example 1: Matrix Multiplication

$T : \mathbb{R}^n \to \mathbb{R}^m$ defined by $T(\mathbf{x}) = A\mathbf{x}$ for a fixed $m \times n$ matrix $A$.

**Verification:**
- $T(\mathbf{u}+\mathbf{v}) = A(\mathbf{u}+\mathbf{v}) = A\mathbf{u}+A\mathbf{v} = T(\mathbf{u})+T(\mathbf{v})$ ✓
- $T(c\mathbf{u}) = A(c\mathbf{u}) = cA\mathbf{u} = cT(\mathbf{u})$ ✓

> **Fact:** Every linear transformation from $\mathbb{R}^n$ to $\mathbb{R}^m$ is of this form.

### Example 2: Rotation in $\mathbb{R}^2$

Rotation by angle $\theta$ counterclockwise:

$$T\begin{pmatrix}x\\y\end{pmatrix} = \begin{pmatrix}\cos\theta & -\sin\theta \\ \sin\theta & \cos\theta\end{pmatrix}\begin{pmatrix}x\\y\end{pmatrix}$$

```
        y                      y
        ▲                      ▲
        │   v                  │  T(v)
        │  ╱                   │ ╱
        │ ╱                    │╱  θ
   ─────┼──────▶ x        ────┼──────▶ x
        │                     │
  
  Rotation is linear — it's matrix multiplication!
```

### Example 3: Reflection Across the $x$-axis

$$T(x, y) = (x, -y)$$

Matrix: $\begin{pmatrix}1&0\\0&-1\end{pmatrix}$. Linear ✓.

### Example 4: Projection onto the $x$-axis

$$T(x, y) = (x, 0)$$

Matrix: $\begin{pmatrix}1&0\\0&0\end{pmatrix}$. Linear ✓.

```
        y
        ▲
        │  • (2,3)
        │  │
        │  │  projection
        │  ▼
   ─────┼──•──────▶ x
        │ (2,0)
```

### Example 5: Differentiation

$D : P_n(\mathbb{R}) \to P_{n-1}(\mathbb{R})$ defined by $D(p) = p'$ (the derivative).

- $D(p + q) = (p+q)' = p' + q' = D(p) + D(q)$ ✓
- $D(cp) = (cp)' = cp' = cD(p)$ ✓

### Example 6: Integration

$T : C[0,1] \to \mathbb{R}$ defined by $T(f) = \int_0^1 f(x)\,dx$.

Linear by properties of the integral.

### Example 7: Transpose Map

$T : M_{m \times n} \to M_{n \times m}$ defined by $T(A) = A^T$.

- $(A+B)^T = A^T + B^T$ ✓
- $(cA)^T = cA^T$ ✓

### Example 8: Trace

$T : M_{n \times n} \to \mathbb{R}$ defined by $T(A) = \text{tr}(A) = \sum a_{ii}$.

Linear ✓.

---

## 5. Non-Examples

| Function | Why Not Linear |
|----------|---------------|
| $T(x, y) = (x + 1, y)$ | $T(0,0) = (1,0) \neq (0,0)$ |
| $T(x) = x^2$ | $T(2x) = 4x^2 \neq 2x^2 = 2T(x)$ |
| $T(x, y) = (xy, 0)$ | $T((1,1)+(1,0)) = T(2,1) = (2,0)$, but $T(1,1)+T(1,0)=(1,0)+(0,0)=(1,0)$ |
| $T(x) = |x|$ | $T(-1) = 1 \neq -1 = -T(1)$ |
| $T(\mathbf{x}) = A\mathbf{x} + \mathbf{b}$ ($\mathbf{b} \neq 0$) | Affine, not linear: $T(\mathbf{0}) = \mathbf{b} \neq \mathbf{0}$ |

---

## 6. Composition of Linear Transformations

If $T : V \to W$ and $S : W \to U$ are both linear, then $S \circ T : V \to U$ is also linear.

$$S \circ T(\mathbf{u} + \mathbf{v}) = S(T(\mathbf{u}+\mathbf{v})) = S(T(\mathbf{u})+T(\mathbf{v})) = S(T(\mathbf{u})) + S(T(\mathbf{v}))$$

```
   V ──── T ────▶ W ──── S ────▶ U
   │                              ▲
   └──────── S ∘ T ───────────────┘
             (also linear)
```

---

## 7. Special Types

| Type | Definition | Example |
|------|-----------|---------|
| **Isomorphism** | Linear, bijective (one-to-one and onto) | $T : \mathbb{R}^2 \to \mathbb{R}^2$, $T(\mathbf{x}) = A\mathbf{x}$ with $\det A \neq 0$ |
| **Endomorphism** | $T : V \to V$ (same domain and codomain) | Rotation, reflection |
| **Automorphism** | Isomorphism with $T : V \to V$ | Invertible matrix transformation |
| **Zero map** | $T(\mathbf{v}) = \mathbf{0}$ for all $\mathbf{v}$ | Always linear |
| **Identity map** | $T(\mathbf{v}) = \mathbf{v}$ | Always linear |

---

## 8. Linearity Verification Checklist

```
  ┌──────────────────────────────────────────┐
  │ Given: T : V → W                         │
  │                                          │
  │ Quick check: Does T(0) = 0?              │
  │      NO  ──────▶  NOT LINEAR. Done.      │
  │      YES                                 │
  │      │                                   │
  │      ▼                                   │
  │ Check: T(u+v) = T(u)+T(v)  for all u,v  │
  │      │                                   │
  │      ▼                                   │
  │ Check: T(cu) = cT(u)  for all c, u      │
  │      │                                   │
  │      ▼                                   │
  │ Both hold? ──▶ LINEAR ✓                  │
  │                                          │
  └──────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Linear transformation | Preserves addition and scalar multiplication |
| Superposition | $T(\sum c_i\mathbf{v}_i) = \sum c_i T(\mathbf{v}_i)$ |
| $T(\mathbf{0}) = \mathbf{0}$ | Necessary condition (quick non-linearity test) |
| Matrix transformation | $T(\mathbf{x}) = A\mathbf{x}$ — every linear map $\mathbb{R}^n \to \mathbb{R}^m$ has this form |
| Differentiation | A linear map on polynomial / function spaces |
| Composition | Composition of linear maps is linear |
| Non-linear indicators | Constant term, product of variables, absolute value |

---

## Quick Revision Questions

1. Verify that $T(x,y,z) = (x+y, 2z, x-y+z)$ is a linear transformation from $\mathbb{R}^3$ to $\mathbb{R}^3$.

2. Is $T(x,y) = (x^2, y)$ linear? Justify.

3. If $T : V \to W$ is linear and $T(\mathbf{v}_1) = \mathbf{w}_1$, $T(\mathbf{v}_2) = \mathbf{w}_2$, find $T(3\mathbf{v}_1 - 2\mathbf{v}_2)$.

4. Prove that the composition of two linear transformations is linear.

5. Give an example of a function $T : \mathbb{R}^2 \to \mathbb{R}$ that is additive ($T(\mathbf{u}+\mathbf{v}) = T(\mathbf{u})+T(\mathbf{v})$) and homogeneous ($T(c\mathbf{u}) = cT(\mathbf{u})$).

6. Why is the function $T(x) = x + 1$ not a linear transformation from $\mathbb{R}$ to $\mathbb{R}$?

---

[← Previous: Basis and Dimension](../01-Vector-Spaces/06-basis-and-dimension.md) · [Back to README](../README.md) · [Next: Kernel and Range →](02-kernel-and-range.md)
