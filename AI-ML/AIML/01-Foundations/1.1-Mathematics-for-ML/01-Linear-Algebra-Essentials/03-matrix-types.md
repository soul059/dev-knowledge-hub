# Chapter 3: Matrix Types

> **Unit 1 — Linear Algebra Essentials**
> Recognizing special matrix types unlocks shortcuts, optimizations, and deeper understanding of ML algorithms.

---

## 3.1 Identity Matrix (I)

The **identity matrix** is a square matrix with 1s on the diagonal and 0s everywhere else.

```
I₃ = [1  0  0]
     [0  1  0]
     [0  0  1]
```

**Properties:**
- AI = IA = A for any conformable matrix A
- Acts as the "1" of matrix multiplication
- det(I) = 1
- All eigenvalues = 1

```python
import numpy as np
I = np.eye(3)
A = np.array([[2,3],[4,5]])
print(A @ np.eye(2))  # A unchanged
```

**ML Relevance:** Ridge regression adds λI to XᵀX → `(XᵀX + λI)⁻¹Xᵀy` — this makes the matrix invertible (regularization).

---

## 3.2 Diagonal Matrix

A **diagonal matrix** has non-zero elements only on the main diagonal.

```
D = [d₁  0   0 ]
    [0   d₂  0 ]
    [0   0   d₃]
```

**Properties:**
- `D⁻¹ = diag(1/d₁, 1/d₂, ..., 1/dₙ)` — trivial to invert
- `Dⁿ = diag(d₁ⁿ, d₂ⁿ, ..., dₙⁿ)` — trivial to exponentiate
- det(D) = d₁ · d₂ · ... · dₙ
- Eigenvalues are the diagonal entries themselves

```python
D = np.diag([2, 3, 5])
print(np.linalg.inv(D))  # diag(0.5, 0.333, 0.2)
```

**ML Relevance:** The Σ matrix in SVD is diagonal. Feature scaling matrices are diagonal.

---

## 3.3 Symmetric Matrix

A matrix A is **symmetric** if `A = Aᵀ` (i.e., aᵢⱼ = aⱼᵢ).

```
A = [1  2  3]      Aᵀ = [1  2  3]
    [2  5  6]           [2  5  6]    A = Aᵀ  ✓
    [3  6  9]           [3  6  9]
```

**Properties:**
- All eigenvalues are **real**
- Eigenvectors corresponding to distinct eigenvalues are **orthogonal**
- Can always be diagonalized: A = QΛQᵀ (spectral theorem)
- xᵀAx is always real for any vector x

**How they arise in ML:**

| Matrix | Formula | Symmetric? |
|--------|---------|------------|
| Covariance matrix | `(1/n) XᵀX` | Always ✓ |
| Gram matrix | `XXᵀ` or `XᵀX` | Always ✓ |
| Kernel matrix | `K(xᵢ, xⱼ)` | Always ✓ |
| Hessian | `∂²L/∂wᵢ∂wⱼ` | Always ✓ (for smooth L) |
| Distance matrix | `D(xᵢ, xⱼ)` | Always ✓ |

```python
X = np.random.randn(100, 5)
cov = X.T @ X / len(X)
print(np.allclose(cov, cov.T))  # True — always symmetric
```

---

## 3.4 Orthogonal Matrix

A square matrix Q is **orthogonal** if its columns are orthonormal vectors:

```
QᵀQ = QQᵀ = I    ⟹    Q⁻¹ = Qᵀ
```

**Properties:**
- Preserves lengths: ‖Qx‖ = ‖x‖
- Preserves angles: (Qx)ᵀ(Qy) = xᵀy
- det(Q) = ±1
- Represents **rotations** (det = +1) and **reflections** (det = −1)

```
Rotation matrix (2D, angle θ):

Q = [cos θ   -sin θ]
    [sin θ    cos θ]

QᵀQ = [cos²θ + sin²θ         0        ]  = I  ✓
      [       0         cos²θ + sin²θ  ]
```

```python
theta = np.pi / 4  # 45 degrees
Q = np.array([[np.cos(theta), -np.sin(theta)],
              [np.sin(theta),  np.cos(theta)]])
print(np.allclose(Q.T @ Q, np.eye(2)))  # True
```

**ML Relevance:** U and V in SVD are orthogonal. PCA uses orthogonal projection. Orthogonal weight initialization in RNNs.

---

## 3.5 Triangular Matrices

### Upper Triangular

```
U = [u₁₁  u₁₂  u₁₃]     All entries below diagonal are 0
    [ 0   u₂₂  u₂₃]
    [ 0    0   u₃₃]
```

### Lower Triangular

```
L = [l₁₁   0    0 ]     All entries above diagonal are 0
    [l₂₁  l₂₂   0 ]
    [l₃₁  l₃₂  l₃₃]
```

**Properties:**
- det = product of diagonal entries
- Easy to solve Lx = b or Ux = b via forward/back substitution
- LU decomposition: A = LU

**ML Relevance:** Cholesky decomposition of covariance matrices (A = LLᵀ). Used in efficient Gaussian process inference.

---

## 3.6 Sparse Matrix

A **sparse matrix** has mostly zero entries. Stored efficiently using special formats.

```
Dense (wasteful):        Sparse (CSR format stores only):
[0  0  3  0]             values:  [3, 1, 2, 5]
[0  0  0  0]             col_idx: [2, 0, 1, 3]
[1  0  0  0]             row_ptr: [0, 1, 1, 3, 4]
[0  2  0  5]
```

```python
from scipy import sparse
A = sparse.csr_matrix([[0,0,3,0],
                        [0,0,0,0],
                        [1,0,0,0],
                        [0,2,0,5]])
print(A.nnz)         # 4 non-zero elements
print(A.toarray())   # convert back to dense
```

**ML Relevance:**

| Application | Why Sparse |
|-------------|-----------|
| TF-IDF / Bag-of-Words | Most words absent from most documents |
| One-hot encoding | Only 1 non-zero per category |
| Adjacency matrices | Most nodes not directly connected |
| Recommendation systems | Users rate few items |

---

## 3.7 Positive Definite & Positive Semi-Definite Matrices

### Positive Definite (PD)

A symmetric matrix A is **positive definite** if for all non-zero vectors x:

```
xᵀAx > 0
```

Equivalently: **all eigenvalues > 0**.

### Positive Semi-Definite (PSD)

```
xᵀAx ≥ 0    for all x
```

Equivalently: **all eigenvalues ≥ 0**.

```
Geometric intuition:
PD  → the quadratic form xᵀAx defines a bowl shape (unique minimum)
PSD → bowl shape but may have flat directions
Indefinite → saddle shape (has both positive and negative eigenvalues)
```

### Quick Test

```python
A = np.array([[2, 1], [1, 3]])
eigenvalues = np.linalg.eigvalsh(A)
print(eigenvalues)                    # [1.38, 3.62] — both > 0
print("PD" if all(eigenvalues > 0) else "Not PD")  # PD
```

### Why PD/PSD Matter in ML

| Context | Requirement |
|---------|------------|
| Covariance matrices | Always PSD (PD if no redundant features) |
| Kernel matrices | Must be PSD (Mercer's condition) |
| Hessian at minimum | Must be PD (confirms local minimum) |
| Loss functions | Convexity ⟺ Hessian is PSD everywhere |
| Cholesky decomposition | Only works for PD matrices |

---

## 3.8 Idempotent, Nilpotent, and Involutory Matrices

| Type | Definition | Example |
|------|-----------|---------|
| **Idempotent** | A² = A | Projection matrices: P = X(XᵀX)⁻¹Xᵀ |
| **Nilpotent** | Aᵏ = 0 for some k | Strictly triangular matrices |
| **Involutory** | A² = I (its own inverse) | Reflection matrices, Pauli matrices |

**ML Relevance:** The hat matrix H = X(XᵀX)⁻¹Xᵀ in linear regression is idempotent — applying the projection twice gives the same result.

---

## 3.9 Summary Table

| Matrix Type | Key Property | Inverse | Eigenvalues | ML Usage |
|------------|-------------|---------|-------------|----------|
| Identity (I) | AI = IA = A | I | All 1s | Regularization |
| Diagonal | Non-zeros only on diagonal | diag(1/dᵢ) | Diagonal entries | SVD (Σ), scaling |
| Symmetric | A = Aᵀ | Symmetric (if exists) | All real | Covariance, kernel |
| Orthogonal | QᵀQ = I | Qᵀ | \|λ\| = 1 | PCA, SVD (U, V) |
| Triangular | Zeros above/below diagonal | Triangular | Diagonal entries | LU, Cholesky |
| Sparse | Mostly zeros | Sparse methods | Depends | NLP, graphs |
| Positive Definite | xᵀAx > 0 | Also PD | All > 0 | Convex optimization |
| PSD | xᵀAx ≥ 0 | May not exist | All ≥ 0 | Kernels, covariance |

---

## 3.10 Quick Revision Questions

1. **Why is the covariance matrix always symmetric and PSD?**
2. **What is the computational advantage of inverting a diagonal matrix vs. a general matrix?**
3. **If Q is orthogonal, why does Q⁻¹ = Qᵀ? What does this imply for computation?**
4. **Why are sparse matrix formats important for NLP tasks like TF-IDF?**
5. **How do you check if a symmetric matrix is positive definite using eigenvalues?**
6. **In Ridge regression, why do we add λI to XᵀX?**

---

[← Previous: Chapter 2 — Matrix Operations](./02-matrix-operations.md) | [Next Chapter → Chapter 4: Matrix Multiplication Interpretations](./04-matrix-multiplication-interpretations.md)
