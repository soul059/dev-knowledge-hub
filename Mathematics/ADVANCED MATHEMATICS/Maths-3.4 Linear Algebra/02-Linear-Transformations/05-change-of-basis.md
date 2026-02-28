# 2.5 Change of Basis

[← Previous: Matrix Representation](04-matrix-representation.md) · [Back to README](../README.md) · [Next: Unit 3 — Inner Product Definition →](../03-Inner-Product-Spaces/01-inner-product-definition.md)

---

## Chapter Overview

The same vector has different coordinate representations in different bases. The **change of basis matrix** translates between these representations. This chapter shows how to construct it, how it transforms the matrix of a linear map, and why it matters for diagonalisation.

---

## 1. The Problem

Let $V$ be a vector space with two bases:

- $\mathcal{B} = \{\mathbf{v}_1, \dots, \mathbf{v}_n\}$
- $\mathcal{B}' = \{\mathbf{v}_1', \dots, \mathbf{v}_n'\}$

A vector $\mathbf{x} \in V$ has coordinates $[\mathbf{x}]_{\mathcal{B}}$ in basis $\mathcal{B}$ and $[\mathbf{x}]_{\mathcal{B}'}$ in basis $\mathcal{B}'$.

**Question:** How do we convert from one to the other?

```
                    x ∈ V
                   ╱     ╲
                  ╱       ╲
          [x]_B ╱         ╲ [x]_B'
          (old coords)  (new coords)
               ╲         ╱
                ╲       ╱
                 P_B→B'
           (change of basis matrix)
```

---

## 2. The Change of Basis Matrix

**Definition.** The **change of basis matrix** from $\mathcal{B}$ to $\mathcal{B}'$, denoted $P_{\mathcal{B} \to \mathcal{B}'}$, is the matrix satisfying:

$$[\mathbf{x}]_{\mathcal{B}'} = P_{\mathcal{B} \to \mathcal{B}'} \cdot [\mathbf{x}]_{\mathcal{B}}$$

### Construction

Express each old basis vector as a linear combination of the new basis:

$$\mathbf{v}_j = p_{1j}\mathbf{v}'_1 + p_{2j}\mathbf{v}'_2 + \cdots + p_{nj}\mathbf{v}'_n$$

Then column $j$ of $P_{\mathcal{B} \to \mathcal{B}'}$ is $[\mathbf{v}_j]_{\mathcal{B}'} = (p_{1j}, \dots, p_{nj})^T$.

$$P_{\mathcal{B} \to \mathcal{B}'} = \Big[\; [\mathbf{v}_1]_{\mathcal{B}'} \;\Big|\; [\mathbf{v}_2]_{\mathcal{B}'} \;\Big|\; \cdots \;\Big|\; [\mathbf{v}_n]_{\mathcal{B}'} \;\Big]$$

---

## 3. Properties

| Property | Formula |
|----------|---------|
| Inverse | $P_{\mathcal{B}' \to \mathcal{B}} = (P_{\mathcal{B} \to \mathcal{B}'})^{-1}$ |
| Composition | $P_{\mathcal{B} \to \mathcal{B}''} = P_{\mathcal{B}' \to \mathcal{B}''} \cdot P_{\mathcal{B} \to \mathcal{B}'}$ |
| Identity | $P_{\mathcal{B} \to \mathcal{B}} = I$ |
| Invertibility | $P$ is always invertible |

---

## 4. Efficient Computation Using Row Reduction

When both bases are in $\mathbb{R}^n$, form and row-reduce:

$$\left[\; \mathcal{B}' \;\Big|\; \mathcal{B} \;\right] \xrightarrow{\text{row reduce}} \left[\; I \;\Big|\; P_{\mathcal{B} \to \mathcal{B}'} \;\right]$$

**Why:** We're solving $\mathbf{v}_j = P \cdot \mathbf{v}'_j$ for each column, which is the same as row-reducing $[\mathcal{B}' \mid \mathcal{B}]$.

---

## 5. Worked Example

$\mathcal{B} = \{(1, 1), (1, -1)\}$, $\mathcal{B}' = \{(1, 0), (0, 1)\}$ (standard basis).

Find $P_{\mathcal{B} \to \mathcal{B}'}$:

```
  [B' | B] = ┌            ┐       ┌            ┐
             │ 1  0 │ 1  1 │       │ 1  0 │ 1  1 │
             │ 0  1 │ 1 -1 │ ────▶ │ 0  1 │ 1 -1 │
             └            ┘       └            ┘
  
  Already reduced! (B' was the standard basis)
  
  P_{B→B'} = ┌       ┐
             │ 1   1 │
             │ 1  -1 │
             └       ┘
```

**Verify:** $[\mathbf{x}]_{\mathcal{B}} = (2, 3)^T$ means $\mathbf{x} = 2(1,1) + 3(1,-1) = (5, -1)$.

$$P \cdot [\mathbf{x}]_{\mathcal{B}} = \begin{pmatrix}1&1\\1&-1\end{pmatrix}\begin{pmatrix}2\\3\end{pmatrix} = \begin{pmatrix}5\\-1\end{pmatrix} = [\mathbf{x}]_{\mathcal{B}'} \quad ✓$$

---

## 6. Change of Basis for a Linear Transformation

If $T : V \to V$ has matrix $[T]_{\mathcal{B}}$ in basis $\mathcal{B}$ and we switch to basis $\mathcal{B}'$:

$$\boxed{[T]_{\mathcal{B}'} = P^{-1} [T]_{\mathcal{B}} P}$$

where $P = P_{\mathcal{B}' \to \mathcal{B}}$ (the matrix whose columns are the old-basis vectors expressed in the new basis).

```
  In basis B:          In basis B':
  
  [x]_B ── [T]_B ──▶ [T(x)]_B       [x]_B' ── [T]_B' ──▶ [T(x)]_B'
     │                    ▲               │                      ▲
     │ P                  │ P⁻¹           │                      │
     ▼                    │               ▼                      │
  [x]_B' ──────────────────            Same computation but
                   P⁻¹·[T]_B·P        in the new basis
```

This is a **similarity transformation** — the topic of Unit 5.

---

## 7. Example: Simplifying a Matrix via Change of Basis

$T : \mathbb{R}^2 \to \mathbb{R}^2$ with $[T]_{\mathcal{E}} = \begin{pmatrix}5 & 3 \\ 3 & 5\end{pmatrix}$ (standard basis $\mathcal{E}$).

Choose $\mathcal{B} = \{(1,1), (1,-1)\}$:

$P = \begin{pmatrix}1&1\\1&-1\end{pmatrix}$, $P^{-1} = \frac{1}{-2}\begin{pmatrix}-1&-1\\-1&1\end{pmatrix} = \begin{pmatrix}1/2&1/2\\1/2&-1/2\end{pmatrix}$

$$[T]_{\mathcal{B}} = P^{-1} [T]_{\mathcal{E}} P = \begin{pmatrix}1/2&1/2\\1/2&-1/2\end{pmatrix}\begin{pmatrix}5&3\\3&5\end{pmatrix}\begin{pmatrix}1&1\\1&-1\end{pmatrix}$$

$$= \begin{pmatrix}1/2&1/2\\1/2&-1/2\end{pmatrix}\begin{pmatrix}8&2\\8&-2\end{pmatrix} = \begin{pmatrix}8 & 0 \\ 0 & 2\end{pmatrix}$$

The matrix is now **diagonal**! The basis vectors $(1,1)$ and $(1,-1)$ are eigenvectors of $T$. This previews **diagonalisation** (Unit 5).

---

## 8. From General Bases (Both Domain and Codomain Change)

For $T : V \to W$ with old bases $\mathcal{B}, \mathcal{C}$ and new bases $\mathcal{B}', \mathcal{C}'$:

$$[T]_{\mathcal{B}'}^{\mathcal{C}'} = Q^{-1} [T]_{\mathcal{B}}^{\mathcal{C}} P$$

where $P = P_{\mathcal{B}' \to \mathcal{B}}$ and $Q = P_{\mathcal{C}' \to \mathcal{C}}$.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Change of basis matrix $P$ | Converts coordinates: $[\mathbf{x}]_{\mathcal{B}'} = P \cdot [\mathbf{x}]_{\mathcal{B}}$ |
| Column $j$ of $P$ | $[\mathbf{v}_j]_{\mathcal{B}'}$ — old basis vector in new coordinates |
| Computation | Row-reduce $[\mathcal{B}' \mid \mathcal{B}] \to [I \mid P]$ |
| Inverse | $P_{\mathcal{B}' \to \mathcal{B}} = P_{\mathcal{B} \to \mathcal{B}'}^{-1}$ |
| Transformation matrix | $[T]_{\mathcal{B}'} = P^{-1}[T]_{\mathcal{B}}P$ (similarity) |
| Why it matters | Right basis → diagonal/simpler matrix |

---

## Quick Revision Questions

1. Find the change of basis matrix from $\mathcal{B} = \{(1, 2), (3, 4)\}$ to the standard basis of $\mathbb{R}^2$.

2. If $P$ is the change of basis matrix from $\mathcal{B}$ to $\mathcal{B}'$, what is the change of basis matrix from $\mathcal{B}'$ to $\mathcal{B}$?

3. $T$ has matrix $\begin{pmatrix}3&1\\0&3\end{pmatrix}$ in the standard basis. Find its matrix in basis $\{(1,1),(0,1)\}$.

4. Why is a change of basis matrix always invertible?

5. How does the change of basis formula relate to the concept of similar matrices?

6. If $P = I$ (identity), what does that tell us about the two bases?

---

[← Previous: Matrix Representation](04-matrix-representation.md) · [Back to README](../README.md) · [Next: Unit 3 — Inner Product Definition →](../03-Inner-Product-Spaces/01-inner-product-definition.md)
