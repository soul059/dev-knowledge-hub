# 6.3 Orthogonal Matrices

[← Previous: Spectral Theorem](02-spectral-theorem.md) · [Back to README](../README.md) · [Next: Quadratic Forms →](04-quadratic-forms.md)

---

## Chapter Overview

**Orthogonal matrices** are the "rigid motion" matrices of linear algebra — they preserve lengths and angles. They appear as the $Q$ in the spectral theorem ($A = QDQ^T$), in QR factorisation, and throughout applied mathematics.

---

## 1. Definition

> **Definition.** A square matrix $Q$ is **orthogonal** if:
>
> $$Q^T Q = Q Q^T = I$$
>
> Equivalently: $Q^{-1} = Q^T$.

This means the columns of $Q$ form an **orthonormal basis** for $\mathbb{R}^n$.

---

## 2. Equivalent Characterisations

The following are equivalent for an $n \times n$ matrix $Q$:

| Condition | |
|-----------|--|
| $Q^TQ = I$ | Definition |
| $Q^{-1} = Q^T$ | Inverse equals transpose |
| Columns of $Q$ are orthonormal | $\mathbf{q}_i \cdot \mathbf{q}_j = \delta_{ij}$ |
| Rows of $Q$ are orthonormal | Same condition, transposed |
| $\|Q\mathbf{x}\| = \|\mathbf{x}\|$ for all $\mathbf{x}$ | Preserves length |
| $\langle Q\mathbf{x}, Q\mathbf{y}\rangle = \langle \mathbf{x}, \mathbf{y}\rangle$ | Preserves inner product |
| $\det(Q) = \pm 1$ | Necessary (not sufficient alone) |

---

## 3. Properties

| Property | Result |
|----------|--------|
| $\det(Q)$ | $= \pm 1$ |
| Eigenvalues | $|\lambda| = 1$ (all lie on unit circle) |
| $Q^{-1}$ | $= Q^T$ (trivial to compute) |
| $Q_1 Q_2$ | Orthogonal (product of orthogonal matrices is orthogonal) |
| $Q^T$ | Orthogonal |
| $Q^{-1}$ | Orthogonal |

**Proof that $|\lambda| = 1$:** $Q\mathbf{v} = \lambda \mathbf{v} \implies \|\mathbf{v}\| = \|Q\mathbf{v}\| = |\lambda| \|\mathbf{v}\| \implies |\lambda| = 1$ ∎

---

## 4. Geometric Interpretation

Orthogonal matrices represent **rigid motions** (isometries) of $\mathbb{R}^n$:

| $\det(Q)$ | Geometric meaning |
|-----------|-------------------|
| $+1$ | **Rotation** (proper orthogonal, belongs to $SO(n)$) |
| $-1$ | **Reflection** (or rotation + reflection) |

```
  Rotation (det = +1):          Reflection (det = -1):
  
      ↑ y                           ↑ y
      │    /·                        │ ·\
      │   / ·                        │ ·  \
      │  /  ·                        │·    \
      │ /   ·                        │      \
      │/  θ ·                        │       \
  ────┼──────→ x                ─────┼────────→ x
      │                             │
  
  Preserves orientation          Reverses orientation
```

### Examples in $\mathbb{R}^2$

**Rotation by angle $\theta$:**

$$R(\theta) = \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix}, \quad \det = 1$$

**Reflection across line at angle $\theta/2$:**

$$S(\theta) = \begin{pmatrix} \cos\theta & \sin\theta \\ \sin\theta & -\cos\theta \end{pmatrix}, \quad \det = -1$$

---

## 5. Examples in $\mathbb{R}^3$

### Rotation about $z$-axis:

$$R_z(\theta) = \begin{pmatrix} \cos\theta & -\sin\theta & 0 \\ \sin\theta & \cos\theta & 0 \\ 0 & 0 & 1 \end{pmatrix}$$

### Reflection through $xy$-plane:

$$S_{xy} = \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & -1 \end{pmatrix}$$

### Permutation matrix (orthogonal!):

$$P = \begin{pmatrix} 0 & 1 & 0 \\ 0 & 0 & 1 \\ 1 & 0 & 0 \end{pmatrix}, \quad P^T P = I$$

---

## 6. The Orthogonal Group $O(n)$ and $SO(n)$

| Group | Definition | Elements |
|-------|-----------|----------|
| $O(n)$ | All $n \times n$ orthogonal matrices | Rotations + reflections |
| $SO(n)$ | Orthogonal matrices with $\det = +1$ | Rotations only |

$O(n)$ is a group under multiplication:
- **Closure:** $Q_1, Q_2 \in O(n) \implies Q_1 Q_2 \in O(n)$
- **Identity:** $I \in O(n)$
- **Inverse:** $Q^{-1} = Q^T \in O(n)$
- **Associativity:** Matrix multiplication is associative

---

## 7. Connection to Orthogonal Diagonalisation

In the spectral theorem $A = QDQ^T$:

- $Q$ is orthogonal → $Q^{-1} = Q^T$
- The transformation $Q$ rotates/reflects to the eigenbasis
- In the eigenbasis, $A$ acts as simple scaling by eigenvalues
- $Q^T$ rotates back to the original basis

```
  A = Q D Qᵀ
  
  Original     Rotate to        Scale by        Rotate
  coords  ───► eigenbasis  ───► eigenvalues ───► back
           Qᵀ                D                Q
```

---

## 8. Constructing Orthogonal Matrices

Methods to build orthogonal matrices:
1. **Gram-Schmidt** on any basis → orthonormal columns → orthogonal matrix
2. **Householder reflections:** $H = I - 2\mathbf{u}\mathbf{u}^T$ (where $\|\mathbf{u}\| = 1$)
3. **Givens rotations:** rotate in a 2D subspace
4. **Matrix exponential of skew-symmetric:** $Q = e^S$ where $S = -S^T$

**Householder reflection:**

$$H = I - 2\mathbf{u}\mathbf{u}^T$$

$H$ is symmetric ($H^T = H$), orthogonal ($H^TH = I$), and involutory ($H^2 = I$).

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Orthogonal matrix | $Q^TQ = I$, columns form ONB |
| Inverse | $Q^{-1} = Q^T$ (free!) |
| Determinant | $\det(Q) = \pm 1$ |
| Eigenvalues | $|\lambda| = 1$ |
| $\det = +1$ | Rotation ($SO(n)$) |
| $\det = -1$ | Reflection |
| Length preservation | $\|Q\mathbf{x}\| = \|\mathbf{x}\|$ |
| Householder | $H = I - 2\mathbf{uu}^T$: symmetric, orthogonal, involutory |

---

## Quick Revision Questions

1. Verify that $R(\theta) = \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix}$ is orthogonal.

2. Prove that the product of two orthogonal matrices is orthogonal.

3. Show that a Householder matrix $H = I - 2\mathbf{uu}^T$ satisfies $H^2 = I$.

4. Find all $2 \times 2$ orthogonal matrices with $\det = +1$.

5. Why can't an orthogonal matrix have eigenvalue $\lambda = 2$?

---

[← Previous: Spectral Theorem](02-spectral-theorem.md) · [Back to README](../README.md) · [Next: Quadratic Forms →](04-quadratic-forms.md)
