# 5. SVD Approach to PCA

[← Previous: Eigenvalue Decomposition](./04-eigenvalue-decomposition.md) | [Next: Choosing Components →](./06-choosing-components.md)

---

## Chapter Overview

While PCA is often taught via eigendecomposition of the covariance matrix, in practice **Singular Value Decomposition (SVD)** is the preferred computational method. SVD operates directly on the centered data matrix (not the covariance matrix), is more numerically stable, can handle non-square matrices, and enables efficient truncated computation. This chapter establishes the relationship between SVD and the covariance eigendecomposition, demonstrates computational advantages, and shows how sklearn uses SVD internally.

---

## 5.1 What is SVD?

### Definition

Any matrix X (N×d) can be decomposed as:

```
X = U Σ Vᵀ

where:
  X  = N×d  data matrix (centered)
  U  = N×N  orthogonal matrix (left singular vectors)
  Σ  = N×d  diagonal matrix (singular values σ₁ ≥ σ₂ ≥ ... ≥ 0)
  Vᵀ = d×d  orthogonal matrix (right singular vectors)

Properties:
  UᵀU = I_N     (columns of U are orthonormal)
  VᵀV = I_d     (columns of V are orthonormal)
  Σ has at most min(N,d) non-zero entries on the diagonal
```

### Shapes Diagram

```
  X     =    U    ·    Σ    ·    Vᵀ
(N×d)     (N×N)    (N×d)     (d×d)

  ┌───┐    ┌─────┐  ┌───┐    ┌───┐
  │   │    │     │  │╲  │    │   │
  │   │    │     │  │ ╲ │    │   │
  │   │  = │     │  │  ╲│    │   │
  │   │    │     │  │   │    │   │
  │   │    │     │  │   │    └───┘
  │   │    │     │  │   │
  └───┘    └─────┘  └───┘

  Data     Left      Singular   Right
  matrix   singular  values     singular
           vectors   (diagonal) vectors
```

### Economy (Thin) SVD

When N > d (more samples than features), use the thin SVD:

```
X = U_r Σ_r Vᵀ

  U_r = N×r   (only first r = min(N,d) columns of U)
  Σ_r = r×r   (r×r diagonal)
  Vᵀ  = d×d   (same as before, but only r rows matter)

  This is MUCH more memory-efficient!
  N=10000, d=100 → thin SVD uses 100× less memory for U
```

---

## 5.2 Relationship Between SVD and PCA

### The Key Connection

```
Given centered data X̃ (N×d), the covariance matrix is:

  S = (1/(N-1)) X̃ᵀX̃

SVD of X̃:
  X̃ = UΣVᵀ

Then:
  X̃ᵀX̃ = (UΣVᵀ)ᵀ(UΣVᵀ)
       = VΣᵀUᵀUΣVᵀ
       = VΣᵀΣVᵀ        (since UᵀU = I)
       = V(ΣᵀΣ)Vᵀ

So:
  S = (1/(N-1)) V(ΣᵀΣ)Vᵀ

Comparing with eigendecomposition of S:
  S = WΛWᵀ

We get:
  ┌───────────────────────────────────────────────────────────┐
  │                                                           │
  │  W = V          (eigenvectors of S = right singular      │
  │                   vectors of X̃)                          │
  │                                                           │
  │  λₖ = σₖ²/(N-1)  (eigenvalues of S = squared singular   │
  │                     values divided by N-1)                │
  │                                                           │
  │  Principal Components = columns of V                      │
  │  Explained Variance = σₖ²/(N-1)                          │
  │  Projected Data = X̃V = UΣ                                │
  │                                                           │
  └───────────────────────────────────────────────────────────┘
```

### Mapping Between EVD and SVD

```
┌──────────────────────────────┬──────────────────────────────┐
│   Eigendecomposition (EVD)   │   Singular Value Decomp (SVD)│
├──────────────────────────────┼──────────────────────────────┤
│ Compute S = (1/(N-1))X̃ᵀX̃   │ Compute X̃ = UΣVᵀ directly   │
│ Solve Sw = λw               │ Singular vectors V           │
│ Eigenvectors W              │ Right singular vectors V (=W)│
│ Eigenvalues Λ               │ σₖ² / (N-1)  (= λₖ)         │
│ Project: Z = X̃W             │ Project: Z = UΣ (or X̃V)     │
└──────────────────────────────┴──────────────────────────────┘
```

---

## 5.3 Computational Advantages of SVD

### Why SVD is Better in Practice

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  Advantage 1: No need to compute covariance matrix                  │
│  ─────────────────────────────────────────────────                   │
│  EVD: Must compute X̃ᵀX̃ (d×d) first → O(Nd²)                      │
│  SVD: Works directly on X̃ (N×d) → avoids squaring                  │
│                                                                      │
│  When N >> d: Both are O(Nd²)                                       │
│  When N << d: SVD on X̃X̃ᵀ (N×N) → O(N²d), much faster!            │
│                                                                      │
│  Advantage 2: Better numerical stability                            │
│  ───────────────────────────────────────                             │
│  Computing X̃ᵀX̃ SQUARES the condition number:                       │
│    cond(X̃ᵀX̃) = cond(X̃)²                                           │
│  This amplifies floating-point errors.                              │
│  SVD avoids this by never forming the product.                      │
│                                                                      │
│  Advantage 3: Truncated SVD (only compute top k)                    │
│  ─────────────────────────────────────────────                       │
│  Don't need ALL singular values — just the largest k.               │
│  Randomized SVD algorithms: O(Ndk) instead of O(Nd²)               │
│  HUGE savings when k << d                                           │
│                                                                      │
│  Advantage 4: Works when N < d                                      │
│  ─────────────────────────────                                       │
│  Covariance matrix X̃ᵀX̃ is d×d and rank-deficient when N < d.      │
│  SVD handles this naturally (only min(N,d) singular values).        │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Numerical Stability Example

```
Consider nearly collinear features:
  Feature 1: [1.000000, 2.000000, 3.000000]
  Feature 2: [1.000001, 2.000001, 3.000001]

  X̃ᵀX̃: condition number ≈ 10¹²   (extremely ill-conditioned!)
  X̃:    condition number ≈ 10⁶    (much better)

  SVD of X̃ gives accurate eigenvectors.
  EVD of X̃ᵀX̃ may give garbage due to floating-point errors.
```

---

## 5.4 Truncated SVD

### Concept

```
Full SVD:  X = U Σ Vᵀ                        (all min(N,d) components)
Truncated: X ≈ U_k Σ_k V_kᵀ                 (only top k components)

  X     ≈   U_k   ·  Σ_k  ·  V_kᵀ
(N×d)     (N×k)    (k×k)    (k×d)

This is the BEST rank-k approximation to X
in terms of Frobenius norm (Eckart-Young theorem):

  ||X - U_k Σ_k V_kᵀ||_F ≤ ||X - B||_F  for any rank-k matrix B
```

### Computational Savings

```
Full SVD of N×d matrix: O(min(N²d, Nd²))
Truncated SVD (top k):  O(Ndk)

Example: N=10000, d=1000, k=50
  Full:      O(10⁹)      → ~minutes
  Truncated: O(5×10⁸)    → ~seconds
  Speedup:   ~20×

For k << min(N,d), truncated SVD is much faster!
```

### Randomized SVD

```
For very large matrices, randomized algorithms:

1. Random projection: Y = X·Ω  where Ω is d×(k+p) random matrix
2. QR factorization: Q, R = QR(Y)   → Q captures column space
3. Reduced problem: B = QᵀX         → small (k+p)×d matrix
4. SVD of B: B = Û Σ̃ Ṽᵀ
5. Recover: U ≈ QÛ,  Σ ≈ Σ̃,  V ≈ Ṽ

Complexity: O(Nd·k) + O(k²·(N+d))
Much faster when k << min(N,d)!

sklearn uses this by default: PCA(svd_solver='auto')
→ Chooses randomized when N,d are large and k is small
```

---

## 5.5 Python Implementation

### Manual SVD-based PCA

```python
import numpy as np

def pca_svd(X, n_components=None):
    """PCA via SVD from scratch."""
    N, d = X.shape
    
    # Step 1: Center
    mean = X.mean(axis=0)
    X_centered = X - mean
    
    # Step 2: SVD (economy form)
    U, sigma, Vt = np.linalg.svd(X_centered, full_matrices=False)
    
    # sigma contains singular values in descending order
    # Vt rows = principal component directions
    
    if n_components is None:
        n_components = min(N, d)
    
    # Step 3: Select top k
    U_k = U[:, :n_components]
    sigma_k = sigma[:n_components]
    Vt_k = Vt[:n_components, :]
    
    # Step 4: Project (two equivalent ways)
    Z = U_k * sigma_k         # Method 1: Z = U_k · Σ_k
    # Z = X_centered @ Vt_k.T  # Method 2: Z = X̃ · V_k (equivalent)
    
    # Eigenvalues from singular values
    eigenvalues = sigma ** 2 / (N - 1)
    explained_ratio = eigenvalues[:n_components] / eigenvalues.sum()
    
    return {
        'Z': Z,
        'components': Vt_k,            # PC directions (as rows)
        'singular_values': sigma_k,
        'eigenvalues': eigenvalues[:n_components],
        'explained_ratio': explained_ratio,
        'mean': mean
    }

# Test
from sklearn.datasets import load_iris
X = load_iris().data

result_svd = pca_svd(X, n_components=2)
print("=== PCA via SVD ===")
print(f"Singular values: {result_svd['singular_values'].round(3)}")
print(f"Eigenvalues:     {result_svd['eigenvalues'].round(3)}")
print(f"Explained ratio: {result_svd['explained_ratio'].round(4)}")
print(f"Cumulative:      {np.cumsum(result_svd['explained_ratio']).round(4)}")
```

### Comparing EVD vs SVD Results

```python
import numpy as np
from sklearn.datasets import load_iris

X = load_iris().data
X_c = X - X.mean(axis=0)

# Method 1: EVD
cov = np.cov(X_c, rowvar=False)
eigenvalues_evd, eigenvectors_evd = np.linalg.eigh(cov)
idx = np.argsort(eigenvalues_evd)[::-1]
eigenvalues_evd = eigenvalues_evd[idx]
eigenvectors_evd = eigenvectors_evd[:, idx]

# Method 2: SVD
U, sigma, Vt = np.linalg.svd(X_c, full_matrices=False)
eigenvalues_svd = sigma**2 / (len(X) - 1)

print("=== Comparison: EVD vs SVD ===")
print(f"\nEigenvalues (EVD): {eigenvalues_evd.round(6)}")
print(f"Eigenvalues (SVD): {eigenvalues_svd.round(6)}")
print(f"Match: {np.allclose(eigenvalues_evd, eigenvalues_svd)}")

# PC directions may differ by sign (both valid)
for i in range(4):
    dot = abs(np.dot(eigenvectors_evd[:, i], Vt[i]))
    print(f"PC{i+1} alignment: {dot:.6f} (should be ≈1)")
```

### sklearn's SVD Solver Options

```python
from sklearn.decomposition import PCA, TruncatedSVD
import numpy as np

X = np.random.randn(1000, 50)

# Option 1: Full SVD (default for small data)
pca_full = PCA(n_components=10, svd_solver='full')
pca_full.fit(X)

# Option 2: Randomized SVD (default for large data with few components)
pca_rand = PCA(n_components=10, svd_solver='randomized', random_state=42)
pca_rand.fit(X)

# Option 3: Auto (sklearn chooses best solver)
pca_auto = PCA(n_components=10, svd_solver='auto')
pca_auto.fit(X)

# Option 4: ARPACK (for very large sparse matrices)
# pca_arpack = PCA(n_components=10, svd_solver='arpack')

# TruncatedSVD: For sparse matrices (no centering!)
from scipy.sparse import random as sparse_random
X_sparse = sparse_random(1000, 500, density=0.1, random_state=42)
tsvd = TruncatedSVD(n_components=10, random_state=42)
X_reduced = tsvd.fit_transform(X_sparse)

print("SVD Solver Comparison:")
print(f"  Full:       explained var = {pca_full.explained_variance_ratio_.sum():.4f}")
print(f"  Randomized: explained var = {pca_rand.explained_variance_ratio_.sum():.4f}")
print(f"  Auto:       explained var = {pca_auto.explained_variance_ratio_.sum():.4f}")
print(f"  Results match: {np.allclose(pca_full.explained_variance_ratio_, pca_auto.explained_variance_ratio_, atol=0.01)}")
```

---

## 5.6 When to Use Which Approach

```
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│  Use EVD (eigendecomposition of covariance) when:                 │
│  • d is small (< 100) — covariance matrix is manageable           │
│  • You need the covariance matrix for other purposes              │
│  • Teaching/learning (more intuitive derivation)                   │
│                                                                    │
│  Use Full SVD when:                                                │
│  • d is moderate (100-1000)                                        │
│  • Need all components                                             │
│  • Numerical precision is important                                │
│                                                                    │
│  Use Truncated/Randomized SVD when:                               │
│  • d is large (> 1000)                                             │
│  • Only need k << d components                                     │
│  • Speed is important                                              │
│  • Data may be sparse                                              │
│                                                                    │
│  sklearn PCA(svd_solver='auto') chooses intelligently:             │
│  • Small data → full SVD                                           │
│  • Large data, few components → randomized SVD                     │
│  • Sparse data → TruncatedSVD (separate class)                    │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 5.7 Summary Table

| Aspect | EVD Approach | SVD Approach |
|--------|-------------|-------------|
| **Operates on** | Covariance S (d×d) | Data X̃ (N×d) |
| **Computes** | S = WΛWᵀ | X̃ = UΣVᵀ |
| **PC directions** | Eigenvectors W | Right singular vectors V |
| **Variances** | Eigenvalues λₖ | σₖ²/(N-1) |
| **Projections** | Z = X̃W | Z = UΣ |
| **Complexity** | O(Nd² + d³) | O(Nd·min(N,d)) |
| **Truncated** | Not naturally | O(Ndk) with randomized |
| **Numerical stability** | Squares condition # | Preserves condition # |
| **Sparse data** | Difficult | TruncatedSVD works |
| **sklearn default** | No | Yes |

---

## 5.8 Revision Questions

1. **Write the SVD factorization** of a matrix X. What are U, Σ, and Vᵀ? What constraints do they satisfy?

2. **Derive the relationship** between SVD of X̃ and eigendecomposition of the covariance matrix. Show that V = W and λₖ = σₖ²/(N-1).

3. **Why is SVD more numerically stable** than eigendecomposition of the covariance matrix? What happens to the condition number?

4. **What is truncated SVD?** How does it reduce computation? State the Eckart-Young theorem.

5. **When would you use TruncatedSVD** (from sklearn) instead of PCA? What is the key difference regarding data centering?

6. **Given a dataset** with N=100 samples and d=10,000 features where you need 50 PCs, which approach would you use and why? Estimate the computational cost of each option.

---

[← Previous: Eigenvalue Decomposition](./04-eigenvalue-decomposition.md) | [Next: Choosing Components →](./06-choosing-components.md)
