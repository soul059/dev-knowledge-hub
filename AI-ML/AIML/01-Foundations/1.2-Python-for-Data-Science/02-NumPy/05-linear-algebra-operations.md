# Chapter 5: Linear Algebra Operations

> **Unit 2 — NumPy** | Dot products, matrix decompositions, and solving systems of equations — the mathematical backbone of machine learning.

---

## 5.1 Dot Product and Matrix Multiplication

```python
import numpy as np

# Vector dot product
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
np.dot(a, b)           # 32 = 1*4 + 2*5 + 3*6

# Matrix-vector multiplication
A = np.array([[1, 2],
              [3, 4]])
v = np.array([5, 6])
np.dot(A, v)           # [17, 39]

# Matrix-matrix multiplication
B = np.array([[5, 6],
              [7, 8]])
np.matmul(A, B)        # [[19 22] [43 50]]

# @ operator — shorthand for matmul (Python 3.5+)
A @ B                  # same as np.matmul(A, B)
```

### `np.dot` vs `np.matmul` (`@`)

| Feature | `np.dot` | `np.matmul` / `@` |
|---------|----------|--------------------|
| 1-D × 1-D | Inner product | Inner product |
| 2-D × 2-D | Matrix multiply | Matrix multiply |
| N-D × N-D | Sum-product over last/second-to-last axes | Batch matrix multiply (broadcasts) |
| Scalars | Allowed | Not allowed |

> **Best Practice:** Use `@` for matrix multiplication — it's clearer and handles batches correctly.

---

## 5.2 Transpose

```python
A = np.array([[1, 2, 3],
              [4, 5, 6]])     # (2, 3)

A.T              # (3, 2) — transpose
np.transpose(A)  # equivalent

# For 1-D arrays, .T does nothing
v = np.array([1, 2, 3])
v.T.shape        # (3,) — still 1-D! Use reshape or newaxis for column vector
```

---

## 5.3 Determinant and Inverse

```python
A = np.array([[1, 2],
              [3, 4]])

# Determinant
np.linalg.det(A)      # -2.0

# Inverse (only for square, non-singular matrices)
A_inv = np.linalg.inv(A)
print(A_inv)
# [[-2.   1. ]
#  [ 1.5 -0.5]]

# Verify: A @ A_inv ≈ Identity
print(np.round(A @ A_inv))
# [[1. 0.]
#  [0. 1.]]
```

> **Warning:** Never compute `inv(A) @ b` to solve `Ax = b`. Use `np.linalg.solve` instead — it's faster and more numerically stable.

---

## 5.4 Solving Linear Systems

Solve **Ax = b** without computing the inverse:

```python
# 2x + y = 5
# x + 3y = 7
A = np.array([[2, 1],
              [1, 3]])
b = np.array([5, 7])

x = np.linalg.solve(A, b)
print(x)   # [1.6  1.8]

# Verify
print(A @ x)   # [5. 7.] ✓
```

---

## 5.5 Matrix Norm and Trace

```python
A = np.array([[1, 2],
              [3, 4]])

# Frobenius norm (default) — sqrt of sum of squared elements
np.linalg.norm(A)           # 5.477

# Vector norms
v = np.array([3, 4])
np.linalg.norm(v)           # 5.0   — L2 norm
np.linalg.norm(v, ord=1)    # 7.0   — L1 norm (Manhattan)
np.linalg.norm(v, ord=np.inf)  # 4.0 — max absolute value

# Trace — sum of diagonal elements
np.trace(A)                 # 5 = 1 + 4
```

---

## 5.6 Eigenvalues and Eigenvectors

```python
A = np.array([[4, -2],
              [1,  1]])

eigenvalues, eigenvectors = np.linalg.eig(A)
print("Eigenvalues:", eigenvalues)     # [3. 2.]
print("Eigenvectors (columns):\n", eigenvectors)

# Verify: A @ v = λ * v  for first eigenvector
v0 = eigenvectors[:, 0]
lam0 = eigenvalues[0]
print(np.allclose(A @ v0, lam0 * v0))  # True

# For symmetric matrices, use eigh (faster, guaranteed real eigenvalues)
S = np.array([[2, 1],
              [1, 3]])
eigenvalues_s, eigenvectors_s = np.linalg.eigh(S)
```

---

## 5.7 Singular Value Decomposition (SVD)

Any matrix **A = U Σ Vᵀ**.

```python
A = np.array([[1, 0, 0],
              [0, 1, 1]])

U, s, Vt = np.linalg.svd(A)
print("U shape:", U.shape)    # (2, 2)
print("s (singular values):", s)       # [1.414  1.0]
print("Vt shape:", Vt.shape)  # (3, 3)

# Reconstruct A from SVD components
S = np.zeros_like(A, dtype=float)
np.fill_diagonal(S, s)
A_reconstructed = U @ S @ Vt
print(np.allclose(A, A_reconstructed))  # True

# Reduced/economy SVD
U_r, s_r, Vt_r = np.linalg.svd(A, full_matrices=False)
# U_r: (2,2), s_r: (2,), Vt_r: (2,3)
```

---

## 5.8 Cholesky Decomposition

For symmetric positive-definite matrices: **A = L Lᵀ**.

```python
A = np.array([[4, 2],
              [2, 3]])

L = np.linalg.cholesky(A)
print(L)
# [[2.  0.       ]
#  [1.  1.4142...]]

print(np.allclose(L @ L.T, A))  # True
```

> Used in Gaussian processes and efficient sampling from multivariate normal distributions.

---

## 5.9 QR Decomposition

**A = QR** where Q is orthogonal and R is upper triangular.

```python
A = np.array([[1, -1, 4],
              [1,  4, -2],
              [1,  4,  2],
              [1, -1,  0]])

Q, R = np.linalg.qr(A)
print("Q shape:", Q.shape)   # (4, 3)
print("R shape:", R.shape)   # (3, 3)
print(np.allclose(Q @ R, A))  # True
```

---

## 5.10 Matrix Rank

```python
A = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

np.linalg.matrix_rank(A)  # 2  — rows are linearly dependent
```

---

## 5.11 ML Application: Solving the Normal Equation

Closed-form solution for linear regression: **θ = (XᵀX)⁻¹ Xᵀy**

```python
# Generate synthetic data: y = 3x + 7 + noise
np.random.seed(42)
X_raw = 2 * np.random.rand(100)
y = 3 * X_raw + 7 + np.random.randn(100) * 0.5

# Add bias column (column of 1s)
X = np.column_stack([np.ones(100), X_raw])   # (100, 2)

# Normal equation
theta = np.linalg.inv(X.T @ X) @ X.T @ y
print(f"Intercept: {theta[0]:.2f}, Slope: {theta[1]:.2f}")
# ≈ Intercept: 7.0, Slope: 3.0

# Better: use lstsq (handles rank deficiency, more stable)
theta_lstsq, residuals, rank, sv = np.linalg.lstsq(X, y, rcond=None)
print(f"lstsq → Intercept: {theta_lstsq[0]:.2f}, Slope: {theta_lstsq[1]:.2f}")
```

---

## 5.12 ML Application: PCA via Eigen Decomposition

```python
# Simulated dataset: 50 samples, 3 features
np.random.seed(0)
X = np.random.randn(50, 3) @ np.array([[2, 0, 0],
                                         [0, 1, 0],
                                         [0, 0, 0.5]])

# Step 1: Center the data
X_centered = X - X.mean(axis=0)

# Step 2: Covariance matrix
cov = (X_centered.T @ X_centered) / (len(X) - 1)

# Step 3: Eigen decomposition
eigenvalues, eigenvectors = np.linalg.eigh(cov)

# Step 4: Sort by descending eigenvalue
order = np.argsort(eigenvalues)[::-1]
eigenvalues = eigenvalues[order]
eigenvectors = eigenvectors[:, order]

# Step 5: Project onto top-2 principal components
X_pca = X_centered @ eigenvectors[:, :2]    # (50, 2)
print("Explained variance ratio:", eigenvalues / eigenvalues.sum())
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| Use `@` for matrix multiply | Readable and handles batch dimensions |
| Use `np.linalg.solve` instead of `inv` | Numerically stable, ~2× faster |
| Use `eigh` for symmetric matrices | Faster, eigenvalues guaranteed real |
| Use `lstsq` for regression | Handles rank-deficient cases |
| Check `matrix_rank` before inversion | Avoids singular matrix errors |
| Use `full_matrices=False` in SVD | Saves memory for non-square matrices |

---

## Summary Table

| Function | Purpose |
|----------|---------|
| `np.dot(a, b)` | Dot / matrix product |
| `A @ B` | Matrix multiplication |
| `np.linalg.inv(A)` | Matrix inverse |
| `np.linalg.det(A)` | Determinant |
| `np.linalg.solve(A, b)` | Solve Ax = b |
| `np.linalg.eig(A)` | Eigenvalues & eigenvectors |
| `np.linalg.svd(A)` | Singular value decomposition |
| `np.linalg.cholesky(A)` | Cholesky decomposition |
| `np.linalg.qr(A)` | QR decomposition |
| `np.linalg.norm(x)` | Vector / matrix norm |
| `np.trace(A)` | Sum of diagonal |
| `np.linalg.matrix_rank(A)` | Rank |
| `np.linalg.lstsq(X, y)` | Least-squares solution |

---

## Revision Questions

1. What is the difference between `np.dot(A, B)` and `A @ B` for 2-D arrays? For N-D arrays?
2. Why should you prefer `np.linalg.solve(A, b)` over `np.linalg.inv(A) @ b`?
3. Write code to verify that the eigenvectors returned by `np.linalg.eig` satisfy **Av = λv**.
4. Explain the shapes of U, s, and Vt returned by `np.linalg.svd` for a (5, 3) matrix.
5. Implement PCA from scratch using SVD instead of eigen decomposition.
6. What does a matrix rank of less than `min(m, n)` tell you about the system of equations?

---

[← Chapter 4: Vectorized Operations](04-vectorized-operations.md) | [Next → Chapter 6: Random Number Generation](06-random-number-generation.md)
