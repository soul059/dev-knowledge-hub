# 7.4 Data Compression and Low-Rank Approximation

[← Previous: Least Squares Problems](03-least-squares-problems.md) · [Back to README](../README.md) · [Next: Unit 8 — General Vector Spaces →](../08-Abstract-Vector-Spaces/01-general-vector-spaces.md)

---

## Chapter Overview

The SVD enables **optimal low-rank approximation** — representing a matrix (data) using far fewer numbers than the original. This is the mathematical foundation of data compression, dimensionality reduction, noise filtering, and Principal Component Analysis (PCA).

---

## 1. Outer Product Form of SVD

The SVD $A = U\Sigma V^T$ can be written as a sum of rank-1 matrices:

$$A = \sigma_1 \mathbf{u}_1 \mathbf{v}_1^T + \sigma_2 \mathbf{u}_2 \mathbf{v}_2^T + \dots + \sigma_r \mathbf{u}_r \mathbf{v}_r^T$$

Each term $\sigma_i \mathbf{u}_i \mathbf{v}_i^T$ is a **rank-1 matrix** (outer product), weighted by singular value $\sigma_i$.

Since $\sigma_1 \geq \sigma_2 \geq \dots$, the first terms capture the **most important** structure.

```
  A = σ₁·u₁v₁ᵀ + σ₂·u₂v₂ᵀ + σ₃·u₃v₃ᵀ + ...
      ─────────   ─────────   ─────────
      Most          Less        Even less
      important    important   important
      
  Truncate to k terms → best rank-k approximation!
```

---

## 2. The Eckart-Young-Mirsky Theorem

> **Theorem (Eckart-Young).** The best rank-$k$ approximation to $A$ (in both Frobenius and operator norms) is:
>
> $$A_k = \sigma_1 \mathbf{u}_1 \mathbf{v}_1^T + \sigma_2 \mathbf{u}_2 \mathbf{v}_2^T + \dots + \sigma_k \mathbf{u}_k \mathbf{v}_k^T$$
>
> with approximation error:
>
> $$\|A - A_k\|_F = \sqrt{\sigma_{k+1}^2 + \sigma_{k+2}^2 + \dots + \sigma_r^2}$$
>
> $$\|A - A_k\|_2 = \sigma_{k+1}$$

This is **optimal** — no other rank-$k$ matrix is closer to $A$.

---

## 3. Data Compression

An $m \times n$ matrix stores $mn$ numbers.

The rank-$k$ approximation $A_k$ stores:
- $k$ singular values: $k$ numbers
- $k$ left singular vectors ($m$-dimensional): $mk$ numbers
- $k$ right singular vectors ($n$-dimensional): $nk$ numbers
- **Total:** $k(1 + m + n) \approx k(m + n)$ numbers

**Compression ratio:** $\frac{mn}{k(m+n)}$

| $m \times n$ matrix | $k$ | Storage | Compression |
|:---:|:---:|:---:|:---:|
| $1000 \times 1000$ | 50 | $100{,}050$ vs $1{,}000{,}000$ | 10× |
| $1000 \times 1000$ | 10 | $20{,}010$ vs $1{,}000{,}000$ | 50× |

---

## 4. Image Compression Example

A greyscale image is an $m \times n$ matrix of pixel intensities.

```
  Original image        Rank-k approximation
  (rank = r)            (k << r)
  
  ┌──────────────┐      ┌──────────────┐
  │▓▓▓▓▓▓▓▓▓▓▓▓▓▓│      │▒▒▒▒▒▒▒▒▒▒▒▒▒▒│
  │▓▓▓▓▓▓▓▓▓▓▓▓▓▓│  →   │▒▒▒▒▒▒▒▒▒▒▒▒▒▒│   Blurrier but
  │▓▓▓▓▓▓▓▓▓▓▓▓▓▓│      │▒▒▒▒▒▒▒▒▒▒▒▒▒▒│   much smaller
  │▓▓▓▓▓▓▓▓▓▓▓▓▓▓│      │▒▒▒▒▒▒▒▒▒▒▒▒▒▒│   file
  └──────────────┘      └──────────────┘
  
  512×512 = 262,144     k=20: 20×1025 = 20,500
  numbers               numbers (13× compression)
```

**Quality increases with $k$:**
- $k = 1$: barely recognisable
- $k = 10$: rough shapes visible
- $k = 50$: good quality
- $k = r$: perfect (original)

---

## 5. Principal Component Analysis (PCA)

PCA finds directions of **maximum variance** in data — it is the SVD of the centred data matrix.

**Setup:** $n$ data points in $\mathbb{R}^p$, arranged as rows of $X$ ($n \times p$).

1. Centre: $\tilde{X} = X - \bar{X}$ (subtract column means)
2. SVD: $\tilde{X} = U\Sigma V^T$
3. Principal components: columns of $V$ (right singular vectors)
4. Variance along $i$-th PC: $\frac{\sigma_i^2}{n-1}$

```
  PCA finds the "natural axes" of your data:
  
      x₂                         PC₂
       │   · ·                      │  · ·
       │  · ·  ·                    │ · · ·
       │ ·  · ·  ·    Rotate to    │·  ·  ·
       │·  · ·  ·  ·  ─────────►   │ · · · ·
       │ ·  · ·  ·                  │  ·  ·
       │  · ·                       │ · ·
  ─────┼──────────→ x₁         ────┼──────────→ PC₁
       │                            │
  
  Original axes                  Principal axes
  (correlated)                  (uncorrelated, max variance)
```

### Dimensionality Reduction

Keep only the first $k$ principal components (explaining most variance):

$$\text{Fraction of variance explained} = \frac{\sigma_1^2 + \dots + \sigma_k^2}{\sigma_1^2 + \dots + \sigma_r^2}$$

Common rule: keep enough PCs to explain 95% of variance.

---

## 6. Noise Filtering

If data $A = A_{\text{signal}} + A_{\text{noise}}$:

- Signal lives in a low-rank subspace → large singular values
- Noise is spread across all directions → small singular values

**Strategy:** keep top $k$ singular values (signal), discard the rest (noise).

$$A_{\text{denoised}} = A_k = \sum_{i=1}^{k} \sigma_i \mathbf{u}_i \mathbf{v}_i^T$$

---

## 7. Summary of SVD Applications

| Application | How SVD helps |
|---|---|
| **Low-rank approximation** | Eckart-Young theorem → optimal $A_k$ |
| **Image compression** | Store $k(m+n)$ instead of $mn$ |
| **PCA / dimensionality reduction** | Top right singular vectors = principal components |
| **Noise filtering** | Truncate small singular values |
| **Recommendation systems** | Latent factor models (Netflix problem) |
| **Natural language processing** | Latent semantic analysis (LSA) |
| **Numerical rank** | Number of singular values above threshold |
| **Condition number** | $\kappa(A) = \sigma_1/\sigma_r$ |

---

## 8. Choosing $k$: The Scree Plot

Plot singular values $\sigma_1, \sigma_2, \dots$ and look for an "elbow":

```
  σ
  │
  │*
  │ *
  │  *
  │   *
  │    *·····*·····*·····*·····*    ← Elbow here: keep first 5
  │                                  singular values
  └────────────────────────────→ i
   1  2  3  4  5  6  7  8  9  10
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Outer product SVD | $A = \sum \sigma_i \mathbf{u}_i \mathbf{v}_i^T$ |
| Best rank-$k$ approx. | $A_k = \sum_{i=1}^k \sigma_i \mathbf{u}_i \mathbf{v}_i^T$ (Eckart-Young) |
| Error | $\|A - A_k\|_F = \sqrt{\sum_{i>k} \sigma_i^2}$ |
| Compression ratio | $mn / [k(m+n)]$ |
| PCA | SVD of centred data → principal components = columns of $V$ |
| Variance explained | $\sigma_i^2/(n-1)$ along $i$-th PC |
| Noise filtering | Truncate small singular values |
| Choosing $k$ | Scree plot elbow or 95% variance rule |

---

## Quick Revision Questions

1. State the Eckart-Young theorem. Why is the rank-$k$ SVD truncation optimal?

2. A $100 \times 100$ matrix has singular values $10, 5, 2, 1, 0.1, 0.01, \dots$. What rank-$k$ approximation would you use?

3. How does PCA relate to the SVD? What do the singular values represent?

4. Compute the compression ratio for a $500 \times 400$ image using rank-20 approximation.

5. Why does truncating small singular values act as a noise filter?

6. What is the Frobenius norm error of approximating a rank-5 matrix by its rank-3 SVD truncation?

---

[← Previous: Least Squares Problems](03-least-squares-problems.md) · [Back to README](../README.md) · [Next: Unit 8 — General Vector Spaces →](../08-Abstract-Vector-Spaces/01-general-vector-spaces.md)
