# Chapter 9: Eigenvalues and Eigenvectors

> **Unit 1 — Linear Algebra Essentials**
> Eigenvalues and eigenvectors reveal the "natural axes" of a transformation. They are the heart of PCA, spectral methods, and stability analysis.

---

## 9.1 Definition and Intuition

An **eigenvector** of a square matrix A is a non-zero vector v that, when transformed by A, only gets **scaled** (not rotated):

```
Av = λv
```

- **v** = eigenvector (the direction that is preserved)
- **λ** = eigenvalue (the scaling factor)

```
Most vectors change direction under A:        Eigenvectors only scale:

    ↑ Av                                        ↑ Av = λv
   ╱                                            │
  ╱    ← v rotated                              │  ← v stretched (λ > 1)
 ╱                                              │      or compressed (0 < λ < 1)
───→ v                                     ────→ v     or flipped (λ < 0)
```

---

## 9.2 Finding Eigenvalues: The Characteristic Equation

Starting from Av = λv:

```
Av = λv
Av - λv = 0
(A - λI)v = 0
```

For non-trivial v ≠ 0, the matrix (A − λI) must be **singular**:

```
det(A - λI) = 0     ← characteristic equation
```

This is a **polynomial of degree n** in λ, yielding up to n eigenvalues (counting multiplicity).

---

## 9.3 Detailed Worked Example (2×2)

```
A = [4  1]
    [2  3]

Step 1: Set up characteristic equation
det(A - λI) = det([4-λ   1 ]) = 0
                  [  2  3-λ]

(4-λ)(3-λ) - (1)(2) = 0
12 - 4λ - 3λ + λ² - 2 = 0
λ² - 7λ + 10 = 0
(λ - 5)(λ - 2) = 0

Eigenvalues: λ₁ = 5,  λ₂ = 2

Step 2: Find eigenvectors for each eigenvalue

For λ₁ = 5:
(A - 5I)v = 0  →  [-1  1][v₁] = [0]
                   [ 2 -2][v₂]   [0]

-v₁ + v₂ = 0  →  v₂ = v₁  →  v₁ = [1, 1]ᵀ (or any scalar multiple)

For λ₂ = 2:
(A - 2I)v = 0  →  [2  1][v₁] = [0]
                   [2  1][v₂]   [0]

2v₁ + v₂ = 0  →  v₂ = -2v₁  →  v₂ = [1, -2]ᵀ

Verification:
A · [1, 1]ᵀ = [4+1, 2+3]ᵀ = [5, 5]ᵀ = 5 · [1, 1]ᵀ  ✓
A · [1,-2]ᵀ = [4-2, 2-6]ᵀ = [2,-4]ᵀ = 2 · [1,-2]ᵀ  ✓
```

```python
import numpy as np
A = np.array([[4, 1], [2, 3]])
eigenvalues, eigenvectors = np.linalg.eig(A)
print("Eigenvalues:", eigenvalues)      # [5. 2.]
print("Eigenvectors:\n", eigenvectors)  # columns are eigenvectors
```

---

## 9.4 Properties of Eigenvalues and Eigenvectors

| Property | Statement |
|----------|-----------|
| Sum | λ₁ + λ₂ + ... + λₙ = tr(A) |
| Product | λ₁ · λ₂ · ... · λₙ = det(A) |
| Scaling | Eigenvalues of cA are cλᵢ |
| Power | Eigenvalues of Aⁿ are λᵢⁿ |
| Inverse | Eigenvalues of A⁻¹ are 1/λᵢ |
| Shift | Eigenvalues of A + cI are λᵢ + c |
| Symmetric | All eigenvalues are real; eigenvectors are orthogonal |
| Multiplicity | An eigenvalue can repeat (geometric vs algebraic multiplicity) |

### Quick Check for Our Example

```
tr(A) = 4 + 3 = 7 = λ₁ + λ₂ = 5 + 2  ✓
det(A) = 12 - 2 = 10 = λ₁ · λ₂ = 5 · 2  ✓
```

---

## 9.5 Diagonalization

If A has n linearly independent eigenvectors, it can be **diagonalized**:

```
A = PΛP⁻¹

where:
  P = [v₁ | v₂ | ... | vₙ]     (eigenvectors as columns)
  Λ = diag(λ₁, λ₂, ..., λₙ)    (eigenvalues on diagonal)
```

### Why Diagonalization Is Powerful

```
A² = (PΛP⁻¹)(PΛP⁻¹) = PΛ²P⁻¹
Aⁿ = PΛⁿP⁻¹ = P · diag(λ₁ⁿ, λ₂ⁿ, ..., λₙⁿ) · P⁻¹
```

Computing Aⁿ reduces to raising scalars to the nth power!

### Example

```python
A = np.array([[4, 1], [2, 3]])
eigenvalues, P = np.linalg.eig(A)
Lambda = np.diag(eigenvalues)
P_inv = np.linalg.inv(P)

# Verify: A = P @ Lambda @ P_inv
print(np.allclose(A, P @ Lambda @ P_inv))  # True

# Compute A^10 efficiently
A_10 = P @ np.diag(eigenvalues**10) @ P_inv
print(A_10)
```

---

## 9.6 Spectral Theorem

For **symmetric matrices** (A = Aᵀ), a stronger result holds:

```
A = QΛQᵀ     (spectral decomposition)

where:
  Q is orthogonal (Q⁻¹ = Qᵀ, columns are orthonormal eigenvectors)
  Λ is diagonal (real eigenvalues)
```

**This always works for symmetric matrices** — no need to check if diagonalizable.

```
A = Σᵢ λᵢ qᵢqᵢᵀ     (sum of rank-1 matrices, outer product form)
```

Each term λᵢqᵢqᵢᵀ is a **projection scaled by λᵢ** onto the eigenvector direction qᵢ.

```python
# Symmetric matrix
S = np.array([[5, 2], [2, 3]])
eigenvalues, Q = np.linalg.eigh(S)  # eigh for symmetric
print("Eigenvalues:", eigenvalues)   # real, guaranteed
print("Q @ Q.T:\n", Q @ Q.T)        # identity (orthogonal)
```

---

## 9.7 Eigendecomposition

The eigendecomposition `A = PΛP⁻¹` can be written as a sum of outer products:

```
A = λ₁ v₁wᵀ₁ + λ₂ v₂wᵀ₂ + ... + λₙ vₙwᵀₙ

where vᵢ = right eigenvectors (columns of P)
      wᵢᵀ = left eigenvectors (rows of P⁻¹)

For symmetric matrices: wᵢ = vᵢ, so:
A = λ₁ v₁v₁ᵀ + λ₂ v₂v₂ᵀ + ... + λₙ vₙvₙᵀ
```

This representation is the foundation of **PCA** and **spectral clustering**.

---

## 9.8 Special Cases and Edge Cases

### Complex Eigenvalues

Non-symmetric real matrices can have **complex eigenvalues** (always in conjugate pairs):

```
A = [0  -1]     Eigenvalues: ±i  (pure rotation, no scaling)
    [1   0]
```

### Repeated Eigenvalues

```
A = [2  1]      Characteristic equation: (2-λ)² = 0
    [0  2]      λ = 2 (algebraic multiplicity 2)

Only one independent eigenvector → NOT diagonalizable!
This is a "defective" matrix.
```

### Zero Eigenvalue

If λ = 0 is an eigenvalue, then Av = 0 for some v ≠ 0, meaning A is **singular** (det = 0).

---

## 9.9 ML Applications

### PCA (Principal Component Analysis)

```
Covariance matrix: Σ = (1/n) XᵀX

Eigendecomposition: Σ = QΛQᵀ

Principal components = eigenvectors of Σ (sorted by eigenvalue magnitude)
Eigenvalues = variance explained along each component

Keep top k eigenvectors → dimensionality reduction from n to k
```

### Google PageRank

```
The web is a directed graph → adjacency matrix M
PageRank = dominant eigenvector of the transition matrix
(eigenvector corresponding to eigenvalue λ = 1)
```

### Stability Analysis

```
System: x(t+1) = Ax(t)

After n steps: x(n) = Aⁿx(0) = PΛⁿP⁻¹x(0)

Stable if |λᵢ| < 1 for all eigenvalues → Λⁿ → 0
Unstable if any |λᵢ| > 1 → exponential growth
```

### Spectral Clustering

```
1. Build similarity graph → Laplacian matrix L
2. Compute k smallest eigenvectors of L
3. Cluster the eigenvector embeddings with k-means
```

---

## 9.10 Worked Example: 3×3 Matrix

```
A = [2  0  0]
    [0  3  4]
    [0  4  9]

Characteristic equation:
det(A - λI) = (2-λ)[(3-λ)(9-λ) - 16] = 0

(2-λ)(27 - 12λ + λ² - 16) = 0
(2-λ)(λ² - 12λ + 11) = 0
(2-λ)(λ-1)(λ-11) = 0

Eigenvalues: λ₁ = 1, λ₂ = 2, λ₃ = 11

Check: tr(A) = 14 = 1 + 2 + 11 ✓
       det(A) = 2(27-16) = 22 = 1·2·11 ✓
```

```python
A = np.array([[2,0,0],[0,3,4],[0,4,9]])
vals, vecs = np.linalg.eigh(A)
print("Eigenvalues:", vals)   # [ 1.  2. 11.]
```

---

## 9.11 Summary Table

| Concept | Key Formula | Notes |
|---------|------------|-------|
| Eigenvalue equation | Av = λv | Non-trivial v only |
| Characteristic eq. | det(A − λI) = 0 | Degree-n polynomial |
| tr(A) | Σ λᵢ | Sum of eigenvalues |
| det(A) | Π λᵢ | Product of eigenvalues |
| Diagonalization | A = PΛP⁻¹ | Requires n independent eigenvectors |
| Spectral theorem | A = QΛQᵀ | For symmetric A (guaranteed) |
| Matrix power | Aⁿ = PΛⁿP⁻¹ | Efficient computation |
| PCA | Σ = QΛQᵀ | Top eigenvectors = principal components |
| Stability | Stable iff \|λᵢ\| < 1 | For discrete-time systems |

---

## 9.12 Quick Revision Questions

1. **What is the geometric interpretation of an eigenvector?**
2. **How do you find eigenvalues from the characteristic equation?**
3. **When can a matrix NOT be diagonalized?**
4. **Why does the spectral theorem guarantee diagonalization for symmetric matrices?**
5. **In PCA, what do the eigenvalues of the covariance matrix represent?**
6. **If A has eigenvalues 2 and 5, what are the eigenvalues of A³?**

---

[← Previous: Chapter 8 — Linear Transformations](./08-linear-transformations.md) | [Next Chapter → Chapter 10: Singular Value Decomposition](./10-singular-value-decomposition.md)
