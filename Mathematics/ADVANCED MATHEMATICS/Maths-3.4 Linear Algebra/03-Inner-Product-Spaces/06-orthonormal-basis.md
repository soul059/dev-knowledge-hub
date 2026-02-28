# 3.6 Orthonormal Basis

[← Previous: Orthogonal Projection](05-orthogonal-projection.md) · [Back to README](../README.md) · [Next: Unit 4 — Eigenvalue Definitions →](../04-Eigenvalues-and-Eigenvectors/01-definitions.md)

---

## Chapter Overview

An **orthonormal basis** combines the power of a basis (unique representation) with the convenience of orthogonality (easy computations). This chapter summarises the properties of orthonormal bases, shows how they simplify computations, and introduces Parseval's identity.

---

## 1. Definition (Recap)

An **orthonormal basis** $\mathcal{B} = \{\mathbf{e}_1, \dots, \mathbf{e}_n\}$ of an inner product space $V$ satisfies:

$$\langle \mathbf{e}_i, \mathbf{e}_j \rangle = \delta_{ij} = \begin{cases} 1 & i = j \\ 0 & i \neq j \end{cases}$$

Every finite-dimensional inner product space has an orthonormal basis (by Gram-Schmidt).

---

## 2. Why Orthonormal Bases Are Superior

| Task | General Basis | Orthonormal Basis |
|------|--------------|-------------------|
| Find coordinates of $\mathbf{v}$ | Solve linear system | $c_i = \langle \mathbf{v}, \mathbf{e}_i \rangle$ |
| Compute $\|\mathbf{v}\|$ | Involves cross terms | $\|\mathbf{v}\|^2 = \sum c_i^2$ |
| Compute $\langle \mathbf{u}, \mathbf{v} \rangle$ | Involves basis inner products | $\langle \mathbf{u}, \mathbf{v} \rangle = \sum a_i b_i$ |
| Projection onto subspace | Needs $(A^TA)^{-1}$ | Simple summation formula |
| Matrix of transformation | General formula | $[T]_{ij} = \langle T(\mathbf{e}_j), \mathbf{e}_i \rangle$ |

---

## 3. Parseval's Identity

> If $\{\mathbf{e}_1, \dots, \mathbf{e}_n\}$ is an orthonormal basis and $\mathbf{v} = \sum c_i \mathbf{e}_i$, then:
>
> $$\|\mathbf{v}\|^2 = \sum_{i=1}^{n} |c_i|^2 = \sum_{i=1}^{n} |\langle \mathbf{v}, \mathbf{e}_i \rangle|^2$$

This is the generalisation of the Pythagorean theorem to $n$ dimensions.

**Proof:**

$$\|\mathbf{v}\|^2 = \left\langle \sum c_i \mathbf{e}_i, \sum c_j \mathbf{e}_j \right\rangle = \sum_{i,j} c_i \overline{c_j} \delta_{ij} = \sum |c_i|^2 \quad \blacksquare$$

---

## 4. Bessel's Inequality

If $\{\mathbf{e}_1, \dots, \mathbf{e}_k\}$ is an orthonormal set (not necessarily a basis) in $V$, then for any $\mathbf{v} \in V$:

$$\sum_{i=1}^{k} |\langle \mathbf{v}, \mathbf{e}_i \rangle|^2 \leq \|\mathbf{v}\|^2$$

Equality iff $\mathbf{v} \in \text{span}\{\mathbf{e}_1, \dots, \mathbf{e}_k\}$ (i.e., the set is a basis for $V$, and Parseval holds).

---

## 5. Orthonormal Basis and Matrix Representations

If $T : V \to V$ and $\mathcal{B} = \{\mathbf{e}_1, \dots, \mathbf{e}_n\}$ is orthonormal, then the matrix $A = [T]_{\mathcal{B}}$ has entries:

$$a_{ij} = \langle T(\mathbf{e}_j), \mathbf{e}_i \rangle$$

For a **self-adjoint** operator ($\langle T\mathbf{u}, \mathbf{v} \rangle = \langle \mathbf{u}, T\mathbf{v} \rangle$), this matrix is **symmetric**: $a_{ij} = a_{ji}$.

---

## 6. Orthonormal Bases in Function Spaces

### Fourier Basis on $[-\pi, \pi]$

$$\mathcal{B} = \left\{\frac{1}{\sqrt{2\pi}}, \frac{\cos x}{\sqrt{\pi}}, \frac{\sin x}{\sqrt{\pi}}, \frac{\cos 2x}{\sqrt{\pi}}, \frac{\sin 2x}{\sqrt{\pi}}, \dots\right\}$$

with $\langle f, g \rangle = \int_{-\pi}^{\pi} f(x) g(x)\,dx$.

The Fourier coefficients $a_n = \langle f, e_n \rangle$ give the best approximation of $f$ in the subspace spanned by the first $N$ basis functions.

```
  Signal decomposition via orthonormal basis:
  
  f(x) = c₁·e₁ + c₂·e₂ + c₃·e₃ + ...
  
  ┌───────────┐   ┌───┐ ┌───┐ ┌───┐
  │ Original  │ = │ c₁│·│ e₁│+│ c₂│·│ e₂│ + ...
  │  signal   │   │   │ │ ~ │ │   │ │~~ │
  └───────────┘   └───┘ └───┘ └───┘ └───┘
  
  Energy: ‖f‖² = c₁² + c₂² + c₃² + ... (Parseval)
```

---

## 7. Change of Orthonormal Basis

If $\mathcal{B}$ and $\mathcal{B}'$ are both orthonormal bases, the change of basis matrix $P$ is **orthogonal**: $P^T P = I$ (equivalently $P^{-1} = P^T$).

This is computationally wonderful — computing the inverse is free (just transpose).

---

## 8. Existence Theorem

> **Theorem.** Every finite-dimensional inner product space has an orthonormal basis.
>
> **Proof:** Start with any basis. Apply Gram-Schmidt. Normalise. ∎

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Orthonormal basis | $\langle \mathbf{e}_i, \mathbf{e}_j \rangle = \delta_{ij}$ |
| Coordinates | $c_i = \langle \mathbf{v}, \mathbf{e}_i \rangle$ (no linear system!) |
| Parseval's identity | $\|\mathbf{v}\|^2 = \sum |\langle \mathbf{v}, \mathbf{e}_i \rangle|^2$ |
| Bessel's inequality | $\sum |\langle \mathbf{v}, \mathbf{e}_i \rangle|^2 \leq \|\mathbf{v}\|^2$ (for partial ONB) |
| Matrix entries | $a_{ij} = \langle T(\mathbf{e}_j), \mathbf{e}_i \rangle$ |
| Change of ONB | Transition matrix is orthogonal ($P^{-1} = P^T$) |
| Fourier basis | ONB for function spaces → signal decomposition |
| Existence | Gram-Schmidt guarantees existence |

---

## Quick Revision Questions

1. State Parseval's identity. What is its geometric interpretation?

2. Find the coordinates of $(5, 3, 1)$ with respect to the ONB $\left\{\frac{1}{\sqrt{2}}(1,1,0), \frac{1}{\sqrt{2}}(1,-1,0), (0,0,1)\right\}$.

3. What is Bessel's inequality and when does equality hold?

4. Why is the change-of-basis matrix between two orthonormal bases always orthogonal?

5. How do Fourier series relate to orthonormal bases in function spaces?

6. If $\mathcal{B}$ is an ONB and $A = [T]_{\mathcal{B}}$, show that $a_{ij} = \langle T(\mathbf{e}_j), \mathbf{e}_i \rangle$.

---

[← Previous: Orthogonal Projection](05-orthogonal-projection.md) · [Back to README](../README.md) · [Next: Unit 4 — Eigenvalue Definitions →](../04-Eigenvalues-and-Eigenvectors/01-definitions.md)
