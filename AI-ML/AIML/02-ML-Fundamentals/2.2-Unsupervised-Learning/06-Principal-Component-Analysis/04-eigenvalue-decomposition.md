# 4. Eigenvalue Decomposition Approach

[← Previous: Mathematical Formulation](./03-mathematical-formulation.md) | [Next: SVD Approach →](./05-svd-approach.md)

---

## Chapter Overview

This chapter presents the **complete eigenvalue decomposition (EVD) procedure** for PCA: compute the covariance matrix, eigendecompose it, sort eigenvalues, select the top k, and project. A full step-by-step numerical example demonstrates every calculation, and Python code shows both manual and sklearn implementations.

---

## 4.1 The Complete EVD Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  PCA via Eigenvalue Decomposition:                                  │
│                                                                     │
│  Input: X ∈ ℝ^{N×d}  (N data points, d features)                  │
│  Output: Z ∈ ℝ^{N×k}  (projected data in k dimensions)            │
│                                                                     │
│  Step 1: Center → X̃ = X - 1·x̄ᵀ                                  │
│  Step 2: Covariance → S = (1/(N-1)) X̃ᵀX̃                         │
│  Step 3: Eigendecompose → S = WΛWᵀ                                │
│  Step 4: Sort → λ₁ ≥ λ₂ ≥ ... ≥ λ_d                              │
│  Step 5: Select top k → W_k = [w₁|w₂|...|w_k]                    │
│  Step 6: Project → Z = X̃ · W_k                                    │
│                                                                     │
│  Optional: Reconstruct → X̂ = Z · W_kᵀ + x̄                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4.2 Step-by-Step Numerical Example

### The Dataset

```
6 data points in 3D:

  X = [ 2   4   1  ]
      [ 3   6   2  ]
      [ 5   7   3  ]
      [ 4   8   4  ]
      [ 6   9   5  ]
      [ 4   6   3  ]
```

### Step 1: Center the Data

```
Mean:
  x̄ = [(2+3+5+4+6+4)/6, (4+6+7+8+9+6)/6, (1+2+3+4+5+3)/6]
     = [24/6, 40/6, 18/6]
     = [4.0, 6.667, 3.0]

Centered data X̃:
  X̃ = [ 2-4.0    4-6.667    1-3.0  ]   = [-2.0   -2.667   -2.0]
      [ 3-4.0    6-6.667    2-3.0  ]     [-1.0   -0.667   -1.0]
      [ 5-4.0    7-6.667    3-3.0  ]     [ 1.0    0.333    0.0]
      [ 4-4.0    8-6.667    4-3.0  ]     [ 0.0    1.333    1.0]
      [ 6-4.0    9-6.667    5-3.0  ]     [ 2.0    2.333    2.0]
      [ 4-4.0    6-6.667    3-3.0  ]     [ 0.0   -0.667    0.0]

Verify: Column means ≈ 0 ✓
```

### Step 2: Covariance Matrix

```
S = (1/(N-1)) X̃ᵀX̃ = (1/5) X̃ᵀX̃

X̃ᵀX̃:
  Element (1,1) = (-2)²+(-1)²+1²+0²+2²+0² = 4+1+1+0+4+0 = 10
  Element (1,2) = (-2)(-2.667)+(-1)(-0.667)+1(0.333)+0(1.333)+2(2.333)+0(-0.667)
                = 5.333+0.667+0.333+0+4.667+0 = 11.0
  Element (1,3) = (-2)(-2)+(-1)(-1)+1(0)+0(1)+2(2)+0(0)
                = 4+1+0+0+4+0 = 9.0
  Element (2,2) = (-2.667)²+(-0.667)²+0.333²+1.333²+2.333²+(-0.667)²
                = 7.111+0.444+0.111+1.778+5.444+0.444 = 15.333
  Element (2,3) = (-2.667)(-2)+(-0.667)(-1)+0.333(0)+1.333(1)+2.333(2)+(-0.667)(0)
                = 5.333+0.667+0+1.333+4.667+0 = 12.0
  Element (3,3) = (-2)²+(-1)²+0²+1²+2²+0² = 4+1+0+1+4+0 = 10

X̃ᵀX̃ = [ 10.0    11.0    9.0  ]
       [ 11.0    15.333  12.0 ]
       [ 9.0     12.0    10.0 ]

S = (1/5) × X̃ᵀX̃ = [ 2.0     2.2     1.8  ]
                     [ 2.2     3.067   2.4  ]
                     [ 1.8     2.4     2.0  ]
```

### Step 3: Eigendecompose

```
Solve: det(S - λI) = 0

| 2.0-λ    2.2      1.8   |
| 2.2      3.067-λ  2.4   | = 0
| 1.8      2.4      2.0-λ |

Characteristic polynomial (using cofactor expansion):
  (2-λ)[(3.067-λ)(2-λ) - 2.4×2.4] - 2.2[2.2(2-λ) - 2.4×1.8]
  + 1.8[2.2×2.4 - (3.067-λ)×1.8] = 0

After expansion (computed numerically):
  -λ³ + 7.067λ² - 0.764λ + 0.004 = 0

Solutions (eigenvalues):
  λ₁ ≈ 6.847    (largest — PC1)
  λ₂ ≈ 0.215    (middle — PC2)
  λ₃ ≈ 0.005    (smallest — PC3)

Check: λ₁ + λ₂ + λ₃ = 7.067 = trace(S) = 2.0 + 3.067 + 2.0 ✓
```

### Step 4: Eigenvectors

```
For λ₁ ≈ 6.847 → solve (S - 6.847I)w = 0:

  [-4.847   2.2     1.8  ] [w₁]   [0]
  [ 2.2    -3.780   2.4  ] [w₂] = [0]
  [ 1.8     2.4    -4.847] [w₃]   [0]

  Solving → w₁ = [-0.524, -0.668, -0.528]  (normalized)

For λ₂ ≈ 0.215 → similarly:
  w₂ = [0.708, -0.054, -0.704]  (normalized)

For λ₃ ≈ 0.005 → similarly:
  w₃ = [-0.473, 0.742, -0.475]  (normalized)

Verify orthogonality:
  w₁·w₂ ≈ 0 ✓,  w₁·w₃ ≈ 0 ✓,  w₂·w₃ ≈ 0 ✓
```

### Step 5: Explained Variance

```
Total variance = λ₁ + λ₂ + λ₃ = 7.067

  PC1: λ₁/total = 6.847/7.067 = 96.9%   ████████████████████
  PC2: λ₂/total = 0.215/7.067 =  3.0%   █
  PC3: λ₃/total = 0.005/7.067 =  0.1%   ▪

Cumulative:
  PC1:        96.9%
  PC1+PC2:    99.9%    ← keeping 2 of 3 dims captures 99.9%!
  PC1+PC2+3:  100.0%

→ The data effectively lives in a 1D subspace!
  (All three features are highly correlated)
```

### Step 6: Project (3D → 1D)

```
W₁ = w₁ = [-0.524, -0.668, -0.528]  (keeping only PC1)

Z = X̃ · W₁:

  z₁ = [-2.0, -2.667, -2.0] · [-0.524, -0.668, -0.528]
     = 1.048 + 1.782 + 1.056 = 3.886

  z₂ = [-1.0, -0.667, -1.0] · [-0.524, -0.668, -0.528]
     = 0.524 + 0.446 + 0.528 = 1.498

  z₃ = [1.0, 0.333, 0.0] · [-0.524, -0.668, -0.528]
     = -0.524 - 0.223 + 0 = -0.747

  z₄ = [0.0, 1.333, 1.0] · [-0.524, -0.668, -0.528]
     = 0 - 0.891 - 0.528 = -1.419

  z₅ = [2.0, 2.333, 2.0] · [-0.524, -0.668, -0.528]
     = -1.048 - 1.559 - 1.056 = -3.663

  z₆ = [0.0, -0.667, 0.0] · [-0.524, -0.668, -0.528]
     = 0 + 0.446 + 0 = 0.446

Projected 1D data:
  Z = [3.886, 1.498, -0.747, -1.419, -3.663, 0.446]

  3D → 1D with only 3.1% information loss!
```

---

## 4.3 Reconstruction from Reduced Dimensions

```
Reconstruct from k PCs:

  X̂ = Z · W_kᵀ + x̄    (approximate reconstruction)

For k=1:
  x̂₁ = 3.886 · [-0.524, -0.668, -0.528] + [4.0, 6.667, 3.0]
      = [-2.036, -2.596, -2.052] + [4.0, 6.667, 3.0]
      = [1.964, 4.071, 0.948]

  Original: [2.0, 4.0, 1.0]
  Reconstructed: [1.964, 4.071, 0.948]
  Error: [0.036, 0.071, 0.052]  ← very small!

Reconstruction error:
  MSE = (1/N) Σₙ ||xₙ - x̂ₙ||²
      = Σ_{j=k+1}^{d} λⱼ     (sum of DISCARDED eigenvalues!)
      = λ₂ + λ₃ = 0.215 + 0.005 = 0.220
```

---

## 4.4 Python Implementation

### Manual Implementation

```python
import numpy as np

def pca_eigendecomposition(X, n_components=None):
    """PCA via eigenvalue decomposition from scratch."""
    N, d = X.shape
    
    # Step 1: Center
    mean = X.mean(axis=0)
    X_centered = X - mean
    
    # Step 2: Covariance matrix
    cov_matrix = (X_centered.T @ X_centered) / (N - 1)
    
    # Step 3: Eigendecompose
    eigenvalues, eigenvectors = np.linalg.eigh(cov_matrix)
    
    # Step 4: Sort by eigenvalue (descending)
    idx = np.argsort(eigenvalues)[::-1]
    eigenvalues = eigenvalues[idx]
    eigenvectors = eigenvectors[:, idx]
    
    # Step 5: Select top k components
    if n_components is None:
        n_components = d
    W_k = eigenvectors[:, :n_components]
    
    # Step 6: Project
    Z = X_centered @ W_k
    
    # Explained variance
    total_var = eigenvalues.sum()
    explained_ratio = eigenvalues[:n_components] / total_var
    
    return {
        'Z': Z,                    # Projected data
        'components': W_k.T,       # Principal components (as rows)
        'eigenvalues': eigenvalues,
        'explained_ratio': explained_ratio,
        'mean': mean,
        'W_k': W_k
    }

# Test with the example data
X = np.array([
    [2, 4, 1],
    [3, 6, 2],
    [5, 7, 3],
    [4, 8, 4],
    [6, 9, 5],
    [4, 6, 3]
], dtype=float)

result = pca_eigendecomposition(X, n_components=2)

print("=== PCA via Eigendecomposition ===\n")
print(f"Eigenvalues: {result['eigenvalues'].round(4)}")
print(f"\nExplained variance ratio:")
for i, (ev, ratio) in enumerate(zip(result['eigenvalues'], 
                                     result['eigenvalues']/result['eigenvalues'].sum())):
    bar = '█' * int(ratio * 30)
    print(f"  PC{i+1}: λ={ev:.4f}  ({ratio*100:.1f}%)  {bar}")

print(f"\nCumulative variance: {np.cumsum(result['explained_ratio']).round(4)}")
print(f"\nProjected data (first 2 PCs):\n{result['Z'].round(4)}")

# Reconstruction
X_centered = X - result['mean']
X_reconstructed = result['Z'] @ result['W_k'].T + result['mean']
recon_error = np.mean(np.sum((X - X_reconstructed)**2, axis=1))
print(f"\nReconstruction error (2 PCs): {recon_error:.6f}")
```

### Using sklearn

```python
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_iris
import numpy as np

# Load real dataset
iris = load_iris()
X = iris.data  # 150 samples, 4 features
y = iris.target

# Standardize (important for PCA!)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Fit PCA
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)

print("=== Iris Dataset PCA ===\n")
print(f"Original shape: {X.shape}")
print(f"Projected shape: {X_pca.shape}")

print(f"\nPrincipal Components (loadings):")
print(f"  Features: {iris.feature_names}")
for i, comp in enumerate(pca.components_):
    print(f"  PC{i+1}: {comp.round(3)}")

print(f"\nEigenvalues (explained variance):")
for i, (var, ratio) in enumerate(zip(pca.explained_variance_, 
                                      pca.explained_variance_ratio_)):
    bar = '█' * int(ratio * 30)
    print(f"  PC{i+1}: λ={var:.3f}  ({ratio*100:.1f}%)  {bar}")

print(f"\nCumulative: {np.cumsum(pca.explained_variance_ratio_).round(4)}")
print(f"→ 2 PCs capture {pca.explained_variance_ratio_.sum()*100:.1f}% of variance")

# Key sklearn attributes
print(f"\n=== Key sklearn PCA Attributes ===")
print(f"  pca.components_            : {pca.components_.shape} (PC directions)")
print(f"  pca.explained_variance_    : {pca.explained_variance_.round(3)} (eigenvalues)")
print(f"  pca.explained_variance_ratio_: {pca.explained_variance_ratio_.round(3)} (proportions)")
print(f"  pca.mean_                  : {pca.mean_.round(3)} (feature means)")
print(f"  pca.n_components_          : {pca.n_components_} (components kept)")
```

---

## 4.5 Standardization: When and Why

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  CRITICAL: Should you standardize before PCA?                        │
│                                                                      │
│  YES (StandardScaler) when:                                          │
│  • Features have DIFFERENT units (cm vs kg vs $)                    │
│  • Features have DIFFERENT scales (age:0-100 vs salary:0-1M)       │
│  • You want PCA on the CORRELATION matrix (not covariance)          │
│                                                                      │
│  NO (just center) when:                                              │
│  • All features have SAME units and similar scales                   │
│  • Features are already normalized                                   │
│  • You want PCA on the COVARIANCE matrix                            │
│  • Variance differences are meaningful (e.g., image pixels)          │
│                                                                      │
│  Without standardization: PCA is dominated by high-variance          │
│  features regardless of their importance!                            │
│                                                                      │
│  Example: height (cm) variance = 100, weight (kg) variance = 400    │
│  → PCA picks weight direction even if height is more informative    │
│  → StandardScaler fixes this by making both variance = 1            │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 4.6 Summary Table

| Step | Input | Output | Formula |
|------|-------|--------|---------|
| **Center** | Raw X (N×d) | X̃ (N×d, zero-mean) | X̃ = X - 1·x̄ᵀ |
| **Covariance** | X̃ | S (d×d, symmetric) | S = (1/(N-1))X̃ᵀX̃ |
| **Eigendecompose** | S | W (d×d), Λ (d×d) | S = WΛWᵀ |
| **Sort** | W, Λ | Ordered by λ descending | λ₁ ≥ λ₂ ≥ ... |
| **Select k** | W, k | W_k (d×k) | First k columns |
| **Project** | X̃, W_k | Z (N×k) | Z = X̃·W_k |
| **Reconstruct** | Z, W_k, x̄ | X̂ (N×d) | X̂ = Z·W_kᵀ + x̄ |

---

## 4.7 Revision Questions

1. **List all six steps** of PCA via eigenvalue decomposition. For each step, state the input, output, and formula.

2. **Given the covariance matrix** S = [[5, 3], [3, 2]], compute the eigenvalues and eigenvectors. What are PC1 and PC2? What percentage of variance does each capture?

3. **Why does PCA use the eigendecomposition** of the covariance matrix? How does the eigenvalue equation Sw=λw arise from the variance maximization problem?

4. **The reconstruction error** when keeping k PCs equals the sum of the discarded eigenvalues. Prove this result.

5. **When should you standardize** data before PCA and when should you only center? Give specific examples.

6. **Compute PCA by hand** for the data points [(1,3), (3,5), (5,7), (2,4)]. Find the covariance matrix, eigenvalues, PC1 direction, and the 1D projection.

---

[← Previous: Mathematical Formulation](./03-mathematical-formulation.md) | [Next: SVD Approach →](./05-svd-approach.md)
