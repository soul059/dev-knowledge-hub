# Chapter 10: Singular Value Decomposition (SVD)

> **Unit 1 — Linear Algebra Essentials**
> SVD is arguably the most important matrix decomposition in data science and ML. It works for ANY matrix — not just square or symmetric ones.

---

## 10.1 Definition

Every matrix A ∈ ℝᵐˣⁿ can be decomposed as:

```
A = UΣVᵀ

where:
  U ∈ ℝᵐˣᵐ   — left singular vectors (orthogonal: UᵀU = I)
  Σ ∈ ℝᵐˣⁿ   — diagonal matrix of singular values σ₁ ≥ σ₂ ≥ ... ≥ 0
  V ∈ ℝⁿˣⁿ   — right singular vectors (orthogonal: VᵀV = I)
```

```
         A              =      U          ·       Σ          ·      Vᵀ
    ┌─────────┐          ┌─────────┐     ┌─────────┐     ┌─────────┐
    │         │          │         │     │σ₁       │     │         │
m   │  m × n  │    =  m │  m × m  │  ·  │  σ₂     │  ·  │  n × n  │
    │         │          │         │     │    ·  0  │     │         │
    └─────────┘          └─────────┘     │  0    · │     └─────────┘
         n                    m          └─────────┘          n
                                              n
```

**Key fact:** SVD always exists for any matrix, regardless of shape or rank.

---

## 10.2 Singular Values vs Eigenvalues

| Property | Eigenvalues | Singular Values |
|----------|------------|-----------------|
| Defined for | Square matrices only | Any matrix (m × n) |
| Can be | Real or complex, positive or negative | Always real, non-negative |
| Equation | Av = λv | Av = σu |
| Geometric meaning | Scaling along eigendirections | Scaling along principal axes |

### Connection

```
Singular values of A = square roots of eigenvalues of AᵀA (or AAᵀ)

σᵢ = √(λᵢ(AᵀA))
```

```python
import numpy as np
A = np.array([[3, 2], [2, 3], [1, 1]])
U, sigma, Vt = np.linalg.svd(A)
print("Singular values:", sigma)

# Verify connection to eigenvalues of AᵀA
eigenvalues = np.linalg.eigvalsh(A.T @ A)
print("√eigenvalues of AᵀA:", np.sqrt(np.sort(eigenvalues)[::-1]))
```

---

## 10.3 Computing SVD Step by Step

Given A ∈ ℝᵐˣⁿ:

```
Step 1: Compute AᵀA (n×n symmetric matrix)
Step 2: Find eigenvalues λᵢ and eigenvectors vᵢ of AᵀA
Step 3: σᵢ = √λᵢ (singular values, sorted descending)
Step 4: V = [v₁ | v₂ | ... | vₙ] (right singular vectors)
Step 5: uᵢ = (1/σᵢ) Avᵢ (left singular vectors for non-zero σᵢ)
Step 6: Complete U with orthonormal vectors spanning null(Aᵀ)
```

### Worked Example

```
A = [3  0]
    [4  5]

Step 1: AᵀA = [3 4][3 0] = [25  20]
              [0 5][4 5]   [20  25]

Step 2: det(AᵀA - λI) = (25-λ)² - 400 = λ² - 50λ + 225 = 0
        λ = (50 ± √(2500-900))/2 = (50 ± 40)/2
        λ₁ = 45, λ₂ = 5

Step 3: σ₁ = √45 = 3√5 ≈ 6.708
        σ₂ = √5 ≈ 2.236

Step 4: Eigenvectors of AᵀA:
        For λ₁ = 45: v₁ = [1, 1]ᵀ/√2
        For λ₂ = 5:  v₂ = [1, -1]ᵀ/√2

Step 5: u₁ = (1/σ₁) Av₁ = (1/3√5) [3/√2 + 0]   = [1/(√10)]  [3 ]
                                    [4/√2 + 5/√2]   [1/(√10)]  [9 ]
        u₁ = [3/√(90), 9/√(90)] = [1/√10, 3/√10]

        u₂ = (1/σ₂) Av₂ = (1/√5) [3/√2 - 0]     = [3/√10, -1/√10]
                                  [4/√2 - 5/√2]
```

```python
A = np.array([[3, 0], [4, 5]])
U, sigma, Vt = np.linalg.svd(A)
print(f"σ = {sigma}")
print(f"U =\n{U}")
print(f"Vᵀ =\n{Vt}")
# Reconstruct
print(f"Reconstruction error: {np.linalg.norm(A - U @ np.diag(sigma) @ Vt):.2e}")
```

---

## 10.4 SVD as Sum of Rank-1 Matrices

```
A = σ₁ u₁ v₁ᵀ + σ₂ u₂ v₂ᵀ + ... + σᵣ uᵣ vᵣᵀ

where r = rank(A) = number of non-zero singular values
```

Each term σᵢ uᵢ vᵢᵀ is a **rank-1 matrix**. The singular values tell you the **importance** of each component.

```
Importance: σ₁ ≥ σ₂ ≥ ... ≥ σᵣ > 0 = σᵣ₊₁ = ... = 0
            ↑                          ↑
         Most important           Noise / redundancy
```

---

## 10.5 Truncated SVD & Low-Rank Approximation

The **Eckart-Young theorem** states that the best rank-k approximation of A (in Frobenius or spectral norm) is:

```
Aₖ = σ₁ u₁ v₁ᵀ + σ₂ u₂ v₂ᵀ + ... + σₖ uₖ vₖᵀ    (keep top k terms)
```

**Approximation error:**

```
‖A - Aₖ‖²_F = σ²ₖ₊₁ + σ²ₖ₊₂ + ... + σ²ᵣ

Fraction of "energy" captured = (σ₁² + ... + σₖ²) / (σ₁² + ... + σᵣ²)
```

```python
A = np.random.randn(100, 50)
U, sigma, Vt = np.linalg.svd(A, full_matrices=False)

# Rank-5 approximation
k = 5
A_k = U[:, :k] @ np.diag(sigma[:k]) @ Vt[:k, :]
error = np.linalg.norm(A - A_k) / np.linalg.norm(A)
print(f"Relative error with rank-{k}: {error:.4f}")
```

---

## 10.6 Image Compression Example

An image (grayscale) is a matrix of pixel values. SVD can compress it:

```
Original image: 512 × 512 = 262,144 values

Full SVD: U(512×512) · Σ(512×512) · Vᵀ(512×512)

Rank-k approx: U(512×k) · Σ(k×k) · Vᵀ(k×512)
Storage: 512k + k + 512k = k(1025) values

k = 50:  50 × 1025 = 51,250 values → 80% compression!
```

```python
from PIL import Image

# Load grayscale image as matrix
# img = np.array(Image.open('photo.jpg').convert('L'), dtype=float)
# Example with random "image"
img = np.random.randint(0, 255, (256, 256)).astype(float)

U, sigma, Vt = np.linalg.svd(img, full_matrices=False)

# Reconstruct with different ranks
for k in [5, 20, 50, 100]:
    compressed = U[:, :k] @ np.diag(sigma[:k]) @ Vt[:k, :]
    error = np.linalg.norm(img - compressed) / np.linalg.norm(img)
    ratio = k * (256 + 1 + 256) / (256 * 256)
    print(f"Rank {k:3d}: error={error:.4f}, compression={1-ratio:.1%}")
```

```
Typical results for a real photo:
Rank   5: error=0.35, compression=96.1%    (very blurry)
Rank  20: error=0.15, compression=84.4%    (recognizable)
Rank  50: error=0.08, compression=60.9%    (decent quality)
Rank 100: error=0.03, compression=21.9%    (near-original)
```

---

## 10.7 Relation to Eigendecomposition

| | Eigendecomposition | SVD |
|---|---|---|
| **Formula** | A = PΛP⁻¹ | A = UΣVᵀ |
| **Requirements** | Square, n independent eigenvectors | Any matrix |
| **Orthogonal?** | Only for symmetric matrices | Always (U, V orthogonal) |
| **Values** | Can be negative/complex | Always non-negative real |
| **Connection** | — | U = eigenvecs of AAᵀ, V = eigenvecs of AᵀA |

For **symmetric PSD** matrices: SVD = eigendecomposition (σᵢ = λᵢ, U = V = Q).

---

## 10.8 The Four Fundamental Subspaces

SVD reveals all four fundamental subspaces of A:

```
                Column space          Null space of Aᵀ
                (range of A)          (left null space)
                = span{u₁,...,uᵣ}    = span{uᵣ₊₁,...,uₘ}
                    ↑                     ↑
               first r columns       remaining columns
                  of U                  of U

                Row space             Null space of A
                = span{v₁,...,vᵣ}    = span{vᵣ₊₁,...,vₙ}
                    ↑                     ↑
               first r columns       remaining columns
                  of V                  of V
```

---

## 10.9 ML Applications of SVD

### Latent Semantic Analysis (LSA)

```
Term-document matrix A (terms × documents)
A ≈ Aₖ = UₖΣₖVₖᵀ

Uₖ: term embeddings in latent space
Vₖ: document embeddings in latent space
Σₖ: importance of each latent "topic"

Similarity between documents: compare their rows in VₖΣₖ
```

### Recommender Systems

```
User-item rating matrix R (sparse, many missing entries)
R ≈ UₖΣₖVₖᵀ

Predicted rating for user i, item j:
r̂ᵢⱼ = Σₜ uᵢₜ σₜ vⱼₜ

This is matrix factorization — the basis of collaborative filtering.
```

### Noise Reduction / Denoising

```
Noisy data matrix = Signal + Noise
Signal lives in low-rank subspace (captured by top singular values)
Noise is spread across all singular values

Strategy: keep top-k singular values, discard rest
```

### Pseudo-Inverse via SVD

```
A⁺ = VΣ⁺Uᵀ

where Σ⁺ replaces each non-zero σᵢ with 1/σᵢ
```

### Dimensionality Reduction

```
TruncatedSVD in scikit-learn:
- Works directly on sparse matrices (unlike PCA)
- Used as preprocessing for text data
```

```python
from sklearn.decomposition import TruncatedSVD

# X = sparse TF-IDF matrix (n_docs × n_terms)
# svd = TruncatedSVD(n_components=100)
# X_reduced = svd.fit_transform(X)  # (n_docs × 100)
```

---

## 10.10 Compact / Reduced SVD

In practice, we often use the **reduced (economy) SVD**:

```
Full SVD:      A = U(m×m) · Σ(m×n) · Vᵀ(n×n)
Reduced SVD:   A = U(m×r) · Σ(r×r) · Vᵀ(r×n)     where r = rank(A)
Economy SVD:   A = U(m×p) · Σ(p×p) · Vᵀ(p×n)     where p = min(m,n)
```

```python
A = np.random.randn(100, 20)
# Full SVD
U_full, s, Vt_full = np.linalg.svd(A, full_matrices=True)
print(f"Full: U={U_full.shape}, Vt={Vt_full.shape}")  # (100,100), (20,20)

# Economy SVD
U_eco, s, Vt_eco = np.linalg.svd(A, full_matrices=False)
print(f"Economy: U={U_eco.shape}, Vt={Vt_eco.shape}")  # (100,20), (20,20)
```

---

## 10.11 Summary Table

| Concept | Key Formula / Fact |
|---------|-------------------|
| SVD definition | A = UΣVᵀ |
| Singular values | σᵢ = √(eigenvalues of AᵀA), always ≥ 0 |
| Rank | Number of non-zero singular values |
| Best rank-k approx | Aₖ = Σᵢ₌₁ᵏ σᵢ uᵢ vᵢᵀ (Eckart-Young) |
| Approximation error | ‖A − Aₖ‖²_F = Σᵢ₌ₖ₊₁ʳ σᵢ² |
| Pseudo-inverse | A⁺ = VΣ⁺Uᵀ |
| Image compression | Keep top-k singular values |
| LSA / NLP | Low-rank approx of term-document matrix |
| Recommender systems | Matrix factorization: R ≈ UΣVᵀ |

---

## 10.12 Quick Revision Questions

1. **What is the key difference between SVD and eigendecomposition?**
2. **How do singular values relate to eigenvalues of AᵀA?**
3. **What does the Eckart-Young theorem guarantee about truncated SVD?**
4. **In image compression, how does increasing k affect quality and storage?**
5. **How is SVD used in recommender systems?**
6. **Why is the reduced SVD preferred over full SVD in practice?**

---

[← Previous: Chapter 9 — Eigenvalues and Eigenvectors](./09-eigenvalues-and-eigenvectors.md) | [Next Chapter → Chapter 11: PCA — The Linear Algebra View](./11-pca-linear-algebra-view.md)
