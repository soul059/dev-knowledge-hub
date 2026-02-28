# 3.3 Orthogonality

[← Previous: Norm and Distance](02-norm-and-distance.md) · [Back to README](../README.md) · [Next: Gram-Schmidt Process →](04-gram-schmidt-process.md)

---

## Chapter Overview

**Orthogonality** generalises the concept of perpendicularity. Orthogonal sets of vectors are maximally "independent" in a geometric sense and are the foundation of the Gram-Schmidt process, least-squares approximation, and the Spectral Theorem.

---

## 1. Definition

**Definition.** Two vectors $\mathbf{u}, \mathbf{v}$ in an inner product space are **orthogonal** if:

$$\langle \mathbf{u}, \mathbf{v} \rangle = 0$$

We write $\mathbf{u} \perp \mathbf{v}$.

```
        ▲ u
        │
        │
        │
  ──────┼──────▶ v
        │
  
  ⟨u, v⟩ = 0  ⟹  u ⊥ v
```

---

## 2. Orthogonal and Orthonormal Sets

| Term | Definition |
|------|-----------|
| **Orthogonal set** | $\langle \mathbf{v}_i, \mathbf{v}_j \rangle = 0$ for all $i \neq j$ |
| **Orthonormal set** | Orthogonal, and each vector has unit norm: $\|\mathbf{v}_i\| = 1$ |

An orthonormal set satisfies:

$$\langle \mathbf{v}_i, \mathbf{v}_j \rangle = \delta_{ij} = \begin{cases} 1 & \text{if } i = j \\ 0 & \text{if } i \neq j \end{cases}$$

### Standard Example

The standard basis $\{\mathbf{e}_1, \mathbf{e}_2, \mathbf{e}_3\}$ of $\mathbb{R}^3$ is orthonormal:

```
        z
        ▲ e₃
        │
        │      All pairs perpendicular:
        │      e₁·e₂ = 0
        ┼──────▶ y     e₁·e₃ = 0
       ╱        e₂     e₂·e₃ = 0
      ╱
     ╱ e₁
    ▼
    x
```

---

## 3. Key Theorem: Orthogonal Sets Are Independent

> **Theorem.** Any orthogonal set of **non-zero** vectors is linearly independent.

**Proof:** Suppose $c_1 \mathbf{v}_1 + \cdots + c_k \mathbf{v}_k = \mathbf{0}$.

Take inner product with $\mathbf{v}_j$:

$$\langle c_1 \mathbf{v}_1 + \cdots + c_k \mathbf{v}_k,\; \mathbf{v}_j \rangle = \langle \mathbf{0}, \mathbf{v}_j \rangle = 0$$

By linearity and orthogonality:

$$c_j \langle \mathbf{v}_j, \mathbf{v}_j \rangle = 0$$

Since $\mathbf{v}_j \neq \mathbf{0}$, $\langle \mathbf{v}_j, \mathbf{v}_j \rangle > 0$, so $c_j = 0$. ∎

---

## 4. Coordinates in an Orthogonal Basis (Fourier Coefficients)

If $\{\mathbf{v}_1, \dots, \mathbf{v}_n\}$ is an orthogonal basis of $V$, any $\mathbf{x} \in V$ has:

$$\mathbf{x} = \sum_{i=1}^{n} \frac{\langle \mathbf{x}, \mathbf{v}_i \rangle}{\langle \mathbf{v}_i, \mathbf{v}_i \rangle} \mathbf{v}_i$$

For an **orthonormal** basis, this simplifies to:

$$\mathbf{x} = \sum_{i=1}^{n} \langle \mathbf{x}, \mathbf{v}_i \rangle \mathbf{v}_i$$

The scalars $c_i = \langle \mathbf{x}, \mathbf{v}_i \rangle$ are the **Fourier coefficients**.

> **Why this is fantastic:** No system of equations to solve! Just compute inner products.

---

## 5. Orthogonal Complement

**Definition.** For a subspace $W$ of an inner product space $V$:

$$W^\perp = \{\mathbf{v} \in V \mid \langle \mathbf{v}, \mathbf{w} \rangle = 0 \text{ for all } \mathbf{w} \in W\}$$

### Properties

| Property | Statement |
|----------|-----------|
| Subspace | $W^\perp$ is a subspace of $V$ |
| Dimension | $\dim(W) + \dim(W^\perp) = \dim(V)$ |
| Double complement | $(W^\perp)^\perp = W$ |
| Trivial complements | $V^\perp = \{\mathbf{0}\}$ and $\{\mathbf{0}\}^\perp = V$ |
| Direct sum | $V = W \oplus W^\perp$ |

```
  V = W ⊕ W⊥
  
  ┌──────────────────────────────┐
  │           V                  │
  │                              │
  │    ┌───────┐  ┌───────┐      │
  │    │   W   │⊥ │  W⊥   │      │
  │    │       │  │       │      │
  │    │  dim k│  │dim n-k│      │
  │    └───────┘  └───────┘      │
  │                              │
  └──────────────────────────────┘
```

---

## 6. The Pythagorean Theorem (Generalised)

> If $\mathbf{u} \perp \mathbf{v}$, then $\|\mathbf{u} + \mathbf{v}\|^2 = \|\mathbf{u}\|^2 + \|\mathbf{v}\|^2$.

**Proof:**

$$\|\mathbf{u}+\mathbf{v}\|^2 = \langle \mathbf{u}+\mathbf{v}, \mathbf{u}+\mathbf{v} \rangle = \|\mathbf{u}\|^2 + 2\langle \mathbf{u},\mathbf{v}\rangle + \|\mathbf{v}\|^2 = \|\mathbf{u}\|^2 + \|\mathbf{v}\|^2$$

since $\langle \mathbf{u}, \mathbf{v}\rangle = 0$.

```
        ▲ u+v
        │╲
        │  ╲  ‖u+v‖
  ‖u‖   │    ╲
        │      ╲
  ──────┼────────╲────▶ v
              ‖v‖
  
  ‖u+v‖² = ‖u‖² + ‖v‖²
```

---

## 7. Orthogonal Complement for Matrix Spaces

For $A \in M_{m \times n}$:

| Relationship | |
|-------------|---|
| $(\text{Row}(A))^\perp = \text{Null}(A)$ | Row space and null space are orthogonal complements in $\mathbb{R}^n$ |
| $(\text{Col}(A))^\perp = \text{Null}(A^T)$ | Column space and left null space are orthogonal complements in $\mathbb{R}^m$ |

This gives the **four fundamental subspaces** of a matrix:

```
           ℝⁿ                              ℝᵐ
  ┌────────────────────┐         ┌────────────────────┐
  │                    │         │                    │
  │  Row(A)  │ Null(A) │    A    │  Col(A)  │ Null(Aᵀ)│
  │  dim = r │ dim=n-r │ ──────▶ │  dim = r │ dim=m-r │
  │          │         │         │          │         │
  │    ⊥     │    ⊥    │         │    ⊥     │    ⊥    │
  └────────────────────┘         └────────────────────┘
```

---

## 8. Example: Finding the Orthogonal Complement

Find $W^\perp$ where $W = \text{span}\{(1, 1, 0), (0, 1, 1)\}$ in $\mathbb{R}^3$.

$\mathbf{v} = (x,y,z) \in W^\perp \iff \langle \mathbf{v}, (1,1,0) \rangle = 0$ and $\langle \mathbf{v}, (0,1,1) \rangle = 0$.

$$\begin{cases} x + y = 0 \\ y + z = 0 \end{cases} \implies y = -x, \; z = x$$

$$W^\perp = \text{span}\{(1, -1, 1)\}$$

Check: $\dim(W) + \dim(W^\perp) = 2 + 1 = 3 = \dim(\mathbb{R}^3)$ ✓

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Orthogonal | $\langle \mathbf{u}, \mathbf{v} \rangle = 0$ |
| Orthonormal | Orthogonal + unit vectors ($\langle \mathbf{v}_i, \mathbf{v}_j \rangle = \delta_{ij}$) |
| Orthogonal ⟹ Independent | Key structural result |
| Fourier coefficients | $c_i = \frac{\langle \mathbf{x}, \mathbf{v}_i \rangle}{\|\mathbf{v}_i\|^2}$ |
| $W^\perp$ | All vectors orthogonal to every vector in $W$ |
| Decomposition | $V = W \oplus W^\perp$ |
| Pythagorean theorem | $\mathbf{u} \perp \mathbf{v} \Rightarrow \|\mathbf{u}+\mathbf{v}\|^2 = \|\mathbf{u}\|^2 + \|\mathbf{v}\|^2$ |
| Four subspaces | Row, Null, Col, Left-Null — orthogonal complement pairs |

---

## Quick Revision Questions

1. Are $(1, 2, -1)$ and $(3, 0, 3)$ orthogonal in $\mathbb{R}^3$? Show your calculation.

2. Find the orthogonal complement of $W = \text{span}\{(1, 0, 1)\}$ in $\mathbb{R}^3$.

3. If $\{\mathbf{v}_1, \mathbf{v}_2, \mathbf{v}_3\}$ is an orthonormal basis and $\mathbf{x} = 2\mathbf{v}_1 - 3\mathbf{v}_2 + \mathbf{v}_3$, compute $\|\mathbf{x}\|$.

4. Prove the generalised Pythagorean theorem for mutually orthogonal vectors.

5. What are the four fundamental subspaces of a matrix $A$, and how are they related?

6. Why is computing coordinates easy in an orthonormal basis compared to a general basis?

---

[← Previous: Norm and Distance](02-norm-and-distance.md) · [Back to README](../README.md) · [Next: Gram-Schmidt Process →](04-gram-schmidt-process.md)
