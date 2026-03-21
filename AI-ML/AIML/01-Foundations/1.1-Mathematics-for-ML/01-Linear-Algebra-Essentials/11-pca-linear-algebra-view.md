# Chapter 11: PCA — The Linear Algebra View

> **Unit 1 — Linear Algebra Essentials**
> PCA is dimensionality reduction through the lens of eigenvectors, covariance, and projection. This chapter ties together everything from the unit.

---

## 11.1 What Is PCA?

**Principal Component Analysis (PCA)** finds a new coordinate system where:
1. Axes (principal components) are **orthogonal**
2. First axis captures **maximum variance**, second captures next most, and so on
3. Dimensions can be reduced by dropping low-variance axes

```
Original space              PCA-rotated space

   y ↑     • •              PC2 ↑
     │   •  •  •                 │  • •
     │  • •  •               ───•──•──•──→ PC1
     │ •   •                    │•  •
     +──────→ x                 │
                            Most variance along PC1
                            → keep PC1, drop PC2 for 1D reduction
```

---

## 11.2 PCA as Eigendecomposition of the Covariance Matrix

### Step-by-Step Algorithm

```
Given: Data matrix X ∈ ℝⁿˣᵈ (n samples, d features)

Step 1: Center the data
        X̃ = X - μ       where μ = (1/n) Σᵢ xᵢ  (mean of each feature)

Step 2: Compute covariance matrix
        C = (1/(n-1)) X̃ᵀX̃    ∈ ℝᵈˣᵈ    (symmetric, PSD)

Step 3: Eigendecompose C
        C = QΛQᵀ
        Eigenvalues:   λ₁ ≥ λ₂ ≥ ... ≥ λ_d ≥ 0
        Eigenvectors:  q₁, q₂, ..., q_d  (principal components)

Step 4: Choose top k components
        W = [q₁ | q₂ | ... | qₖ]  ∈ ℝᵈˣᵏ   (projection matrix)

Step 5: Project data
        Z = X̃W  ∈ ℝⁿˣᵏ   (reduced-dimension representation)
```

---

## 11.3 Why the Covariance Matrix?

The **covariance matrix** C captures how features vary together:

```
C = [Var(x₁)      Cov(x₁,x₂)  ...  Cov(x₁,x_d)]
    [Cov(x₂,x₁)   Var(x₂)     ...  Cov(x₂,x_d)]
    [   ⋮             ⋮         ⋱       ⋮       ]
    [Cov(x_d,x₁)  Cov(x_d,x₂) ...   Var(x_d)   ]
```

**Properties:**
- Symmetric → real eigenvalues, orthogonal eigenvectors (spectral theorem)
- PSD → eigenvalues ≥ 0
- Diagonal entries = variance of each feature
- Off-diagonal = covariance between features

**Key insight:** Eigenvectors of C point in the directions of maximum variance. Eigenvalues tell you **how much** variance is in each direction.

---

## 11.4 The Optimization Perspective

PCA finds the direction w that **maximizes the variance** of the projected data:

```
Maximize:   Var(X̃w) = wᵀCw
Subject to: ‖w‖ = 1   (unit vector constraint)
```

Using Lagrange multipliers:

```
∇_w [wᵀCw - λ(wᵀw - 1)] = 0
2Cw - 2λw = 0
Cw = λw       ← eigenvalue equation!
```

The maximum variance direction **is the eigenvector** with the largest eigenvalue.

```
Variance along qᵢ = λᵢ

Total variance = Σᵢ λᵢ = tr(C)
Variance explained by top-k = (λ₁ + ... + λₖ) / (λ₁ + ... + λ_d)
```

---

## 11.5 Choosing the Number of Components (k)

### Explained Variance Ratio

```
Explained variance ratio for component i:
    EVR(i) = λᵢ / Σⱼ λⱼ

Cumulative explained variance:
    CEV(k) = (λ₁ + ... + λₖ) / (λ₁ + ... + λ_d)
```

### Common Strategies

| Strategy | Rule |
|----------|------|
| **Threshold** | Choose k such that CEV(k) ≥ 0.95 (keep 95% variance) |
| **Elbow method** | Plot CEV vs k; choose the "elbow" point |
| **Kaiser criterion** | Keep components with λᵢ > 1 (for standardized data) |
| **Scree plot** | Plot eigenvalues; look for the drop-off |

```
Scree Plot (eigenvalue vs component number):

λ │ ■
  │ ■
  │   ■
  │     ■
  │       ■ ■ ■ ■ ■ ■ ■    ← "elbow" around k = 4
  └─────────────────────→ component
  1  2  3  4  5  6  7  8
```

---

## 11.6 Geometric Interpretation

```
Original 2D data:                After PCA rotation:

  y ↑    ╱ · ·                   PC2 ↑
    │  ╱ ·  ·  ·                     │  · ·
    │╱  · ·  ·                    ───·──·──·──→ PC1
    ·   ·                            │·  ·
    +──────→ x                       │

The data forms an elongated cloud.
PC1 = direction of maximum spread (major axis of ellipse)
PC2 = perpendicular direction (minor axis)

After projection onto PC1 only (1D reduction):
───•──•──•──•──•──•──•──→ PC1
         (preserves the most information)
```

**PCA finds the axes of the data ellipsoid:**
- Eigenvectors = axis directions
- Eigenvalues = squared lengths of axes (proportional to variance)

---

## 11.7 Connection to SVD

PCA can be computed via SVD **without forming the covariance matrix**:

```
X̃ = UΣVᵀ     (SVD of centered data matrix)

Covariance: C = (1/(n-1)) X̃ᵀX̃ = (1/(n-1)) VΣᵀUᵀ UΣVᵀ
             = (1/(n-1)) VΣ²Vᵀ    (since UᵀU = I)

This is the eigendecomposition of C!
  Eigenvectors = columns of V = right singular vectors of X̃
  Eigenvalues  = σᵢ²/(n-1) = squared singular values / (n-1)
```

### Why SVD Is Preferred

| Method | Complexity | Numerical Stability |
|--------|-----------|-------------------|
| Eigendecomposition of C | O(d³) + O(nd²) to form C | Forming XᵀX can lose precision |
| SVD of X̃ | O(nd · min(n,d)) | More stable, avoids XᵀX |

Scikit-learn's PCA uses SVD internally for this reason.

---

## 11.8 Python Example: Full PCA Pipeline

```python
import numpy as np

# Generate correlated 2D data
np.random.seed(42)
mean = [5, 3]
cov = [[3, 2], [2, 2]]
X = np.random.multivariate_normal(mean, cov, 200)
print(f"Data shape: {X.shape}")  # (200, 2)

# Step 1: Center the data
X_mean = X.mean(axis=0)
X_centered = X - X_mean

# Step 2: Covariance matrix
C = np.cov(X_centered, rowvar=False)
print(f"Covariance matrix:\n{C}")

# Step 3: Eigendecomposition
eigenvalues, eigenvectors = np.linalg.eigh(C)
# Sort by descending eigenvalue
idx = np.argsort(eigenvalues)[::-1]
eigenvalues = eigenvalues[idx]
eigenvectors = eigenvectors[:, idx]

print(f"Eigenvalues: {eigenvalues}")
print(f"Eigenvectors:\n{eigenvectors}")

# Step 4: Explained variance
total_var = eigenvalues.sum()
for i, ev in enumerate(eigenvalues):
    print(f"PC{i+1}: variance={ev:.3f}, explained={ev/total_var:.1%}")

# Step 5: Project onto top-1 component
k = 1
W = eigenvectors[:, :k]
Z = X_centered @ W
print(f"Reduced shape: {Z.shape}")  # (200, 1)

# Step 6: Reconstruct (for visualization)
X_reconstructed = Z @ W.T + X_mean
recon_error = np.mean((X - X_reconstructed)**2)
print(f"Reconstruction MSE: {recon_error:.4f}")
```

### Using Scikit-learn

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=2)
Z = pca.fit_transform(X)

print(f"Explained variance ratio: {pca.explained_variance_ratio_}")
print(f"Components (eigenvectors):\n{pca.components_}")
print(f"Singular values: {pca.singular_values_}")

# Choose k to keep 95% variance
pca_95 = PCA(n_components=0.95)
Z_95 = pca_95.fit_transform(X)
print(f"Components needed for 95%: {pca_95.n_components_}")
```

---

## 11.9 PCA on Real Data: Iris Dataset

```python
from sklearn.datasets import load_iris
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler

# Load and standardize
iris = load_iris()
X = StandardScaler().fit_transform(iris.data)  # 150 × 4

# PCA to 2D
pca = PCA(n_components=2)
Z = pca.fit_transform(X)

print(f"Original dimensions: {X.shape[1]}")
print(f"Reduced dimensions:  {Z.shape[1]}")
print(f"Variance explained:  {pca.explained_variance_ratio_}")
print(f"Total:               {pca.explained_variance_ratio_.sum():.1%}")
# Typically: PC1 ≈ 72.8%, PC2 ≈ 23.0% → Total ≈ 95.8%

# Visualization description:
# In the 2D PCA plot, the three iris species (setosa, versicolor, virginica)
# form distinct clusters, showing that most of the discriminating information
# in the original 4D space is captured by just 2 principal components.
```

---

## 11.10 When PCA Fails

| Limitation | Explanation | Alternative |
|-----------|------------|-------------|
| Non-linear structure | PCA only finds linear projections | Kernel PCA, t-SNE, UMAP |
| Assumes variance = importance | High-variance noise misleads PCA | Feature selection first |
| Sensitive to scaling | Unscaled features with large range dominate | Always standardize first |
| Components hard to interpret | PCs are mixtures of original features | Use sparse PCA or NMF |
| Outliers | Distort covariance matrix | Use robust PCA |

---

## 11.11 PCA Checklist for ML Practitioners

```
□ Standardize features (zero mean, unit variance)
□ Fit PCA and check explained variance ratio
□ Choose k using 95% threshold or scree plot
□ Transform training AND test data with the same PCA
□ Consider: is the relationship between features actually linear?
□ Report the number of components and variance retained
```

---

## 11.12 Summary Table

| Concept | Key Formula / Fact |
|---------|-------------------|
| Covariance matrix | C = (1/(n−1)) X̃ᵀX̃ (symmetric, PSD) |
| PCA objective | Maximize wᵀCw subject to ‖w‖ = 1 |
| Solution | Eigenvectors of C sorted by eigenvalue |
| Eigenvalue λᵢ | = Variance along PC i |
| Explained variance | EVR(i) = λᵢ / Σλⱼ |
| Projection | Z = X̃W where W = [q₁ \| ... \| qₖ] |
| SVD connection | V from X̃ = UΣVᵀ gives principal components |
| Reconstruction | X̃ ≈ ZWᵀ |
| Scikit-learn | `PCA(n_components=k).fit_transform(X)` |

---

## 11.13 Quick Revision Questions

1. **Why must data be centered (mean-subtracted) before PCA?**
2. **What is the relationship between eigenvalues of the covariance matrix and variance explained?**
3. **Why is SVD preferred over eigendecomposition of XᵀX for computing PCA?**
4. **If the first 3 principal components explain 98% of variance, what can you conclude?**
5. **When would you use Kernel PCA instead of standard PCA?**
6. **Why is it important to apply the same PCA transformation to both training and test data?**

---

[← Previous: Chapter 10 — Singular Value Decomposition](./10-singular-value-decomposition.md)

---

> 🎓 **Congratulations!** You have completed Unit 1: Linear Algebra Essentials.
> These concepts form the mathematical backbone of machine learning — from data representation (vectors, matrices) to dimensionality reduction (eigendecomposition, SVD, PCA) to model training (linear transformations, matrix calculus).
