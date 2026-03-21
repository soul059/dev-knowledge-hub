# 7. Explained Variance Ratio

[← Previous: Choosing Components](./06-choosing-components.md) | [Next: PCA for Visualization →](./08-pca-for-visualization.md)

---

## Chapter Overview

The **explained variance ratio** is the most important diagnostic metric for PCA. It tells us what fraction of the total information (variance) each principal component captures, enabling informed decisions about how many components to keep. This chapter covers the mathematics of computing explained variance, interpreting individual and cumulative ratios, visualization techniques, and practical guidelines.

---

## 7.1 Computing Explained Variance

### From Eigenvalues

```
Given eigenvalues λ₁ ≥ λ₂ ≥ ... ≥ λ_d from PCA:

  Total variance = Σᵢ λᵢ = trace(S) = Σᵢ Var(xᵢ)

  Explained variance of PCk:
    EV(k) = λₖ

  Explained variance RATIO of PCk:
    EVR(k) = λₖ / Σᵢ λᵢ

  Cumulative explained variance ratio for first k PCs:
    CEVR(k) = Σᵢ₌₁ᵏ λᵢ / Σᵢ₌₁ᵈ λᵢ

  Properties:
    • EVR(k) ∈ [0, 1]
    • Σₖ EVR(k) = 1
    • EVR(1) ≥ EVR(2) ≥ ... ≥ EVR(d)  (monotonically decreasing)
    • CEVR(d) = 1 (all components = 100%)
```

### From Singular Values

```
If using SVD (X̃ = UΣVᵀ):

  Explained variance of PCk:
    EV(k) = σₖ² / (N-1)

  Explained variance ratio:
    EVR(k) = σₖ² / Σᵢ σᵢ²

  (Squared singular values are proportional to eigenvalues)
```

---

## 7.2 Interpreting the Ratios

### Individual Ratios

```
EVR(k) tells us: "PCk alone captures X% of the total variance"

Example (Iris dataset, 4 features):
  PC1: EVR = 72.96%   → "PC1 alone captures 73% of all variance"
  PC2: EVR = 22.85%   → "PC2 adds another 23%"
  PC3: EVR =  3.67%   → "PC3 adds only 3.7%"
  PC4: EVR =  0.52%   → "PC4 is almost pure noise"

Interpretation:
  • PC1 = 73% → data is strongly 1-dimensional
  • PC1+PC2 = 96% → 2D captures almost everything
  • PC3+PC4 = 4.2% → these are mostly noise
```

### Cumulative Ratios

```
CEVR(k) tells us: "The first k PCs together capture X% of variance"

  k=1:  73.0%    ████████████████████████████████████
  k=2:  95.8%    ███████████████████████████████████████████████
  k=3:  99.5%    █████████████████████████████████████████████████
  k=4: 100.0%    ██████████████████████████████████████████████████

  → 2 PCs (out of 4) capture 95.8% — great reduction!
```

### What "Unexplained Variance" Means

```
Unexplained variance when keeping k PCs:

  Unexplained = 1 - CEVR(k) = Σᵢ₌ₖ₊₁ᵈ EVR(i)

  This represents:
  1. Information LOST by dimensionality reduction
  2. Reconstruction ERROR (in variance units)
  3. Likely a mix of NOISE and some SIGNAL

  If unexplained = 5% (keeping 95% variance):
    → At most 5% of the information is lost
    → Often much of that 5% is noise → net benefit!
```

---

## 7.3 Visualization Techniques

### Bar Plot + Cumulative Line

```python
import numpy as np
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_digits

# Load data
digits = load_digits()
X = StandardScaler().fit_transform(digits.data)

# Fit PCA
pca = PCA()
pca.fit(X)

# Individual and cumulative variance
individual = pca.explained_variance_ratio_
cumulative = np.cumsum(individual)

# ASCII visualization (first 20 PCs)
n_show = 20
print("=== Explained Variance Analysis ===\n")
print(f"{'PC':>4} | {'Individual':>10} | {'Cumulative':>10} | Individual Bar")
print(f"{'-'*4}-+-{'-'*10}-+-{'-'*10}-+-{'-'*30}")

for i in range(n_show):
    bar_len = int(individual[i] / individual[0] * 25)
    marker = ""
    if cumulative[i] >= 0.95 and (i == 0 or cumulative[i-1] < 0.95):
        marker = " ← 95%"
    print(f"PC{i+1:>2} | {individual[i]:>9.4f} | {cumulative[i]:>9.4f} | {'█' * bar_len}{marker}")

# Find thresholds
for thresh in [0.80, 0.90, 0.95, 0.99]:
    k = np.argmax(cumulative >= thresh) + 1
    print(f"\n{thresh*100:.0f}% variance → {k} components (out of {len(individual)})")

# For matplotlib visualization:
# import matplotlib.pyplot as plt
# fig, ax1 = plt.subplots(figsize=(10, 5))
# ax1.bar(range(1, n_show+1), individual[:n_show], alpha=0.6, label='Individual')
# ax2 = ax1.twinx()
# ax2.plot(range(1, n_show+1), cumulative[:n_show], 'r-o', label='Cumulative')
# ax2.axhline(y=0.95, color='g', linestyle='--', label='95% threshold')
# ax1.set_xlabel('Principal Component')
# ax1.set_ylabel('Individual Explained Variance')
# ax2.set_ylabel('Cumulative Explained Variance')
# plt.title('Explained Variance Analysis')
# plt.legend()
# plt.show()
```

---

## 7.4 Practical Interpretation Guidelines

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  Interpreting Explained Variance Patterns:                           │
│                                                                      │
│  Pattern 1: ONE dominant PC                                         │
│    PC1=90%, PC2=5%, PC3=3%, ...                                     │
│    → Data is essentially 1-dimensional                              │
│    → Strong linear trend or one dominant factor                     │
│                                                                      │
│  Pattern 2: FEW significant PCs                                     │
│    PC1=45%, PC2=30%, PC3=15%, PC4=5%, ...                           │
│    → Data has 3 main "factors" or "modes"                          │
│    → Good candidate for dimensionality reduction                    │
│                                                                      │
│  Pattern 3: GRADUAL decline                                         │
│    PC1=15%, PC2=12%, PC3=10%, PC4=9%, ...                           │
│    → Data is intrinsically high-dimensional                        │
│    → PCA may not help much                                         │
│    → Many features contribute independently                        │
│                                                                      │
│  Pattern 4: FLAT (all equal)                                        │
│    PC1=10%, PC2=10%, PC3=10%, ... (10 PCs × 10%)                   │
│    → No structure → PCA useless                                    │
│    → Data is isotropic (uniform in all directions)                 │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 7.5 Real-World Examples

### Example 1: Iris (4D → 2D)

```python
from sklearn.datasets import load_iris
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA

iris = load_iris()
X = StandardScaler().fit_transform(iris.data)
pca = PCA().fit(X)

print("Iris Dataset (4 features):")
print(f"  PC1: {pca.explained_variance_ratio_[0]*100:.1f}%  → sepal/petal size factor")
print(f"  PC2: {pca.explained_variance_ratio_[1]*100:.1f}%  → shape factor")
print(f"  PC3: {pca.explained_variance_ratio_[2]*100:.1f}%  → minor variation")
print(f"  PC4: {pca.explained_variance_ratio_[3]*100:.1f}%  → noise")
print(f"  → 2 PCs capture {sum(pca.explained_variance_ratio_[:2])*100:.1f}% — excellent!")
```

### Example 2: Digits (64D → ~20D)

```python
from sklearn.datasets import load_digits

digits = load_digits()
X = StandardScaler().fit_transform(digits.data)
pca = PCA().fit(X)

cumulative = np.cumsum(pca.explained_variance_ratio_)
for thresh in [0.80, 0.90, 0.95]:
    k = np.argmax(cumulative >= thresh) + 1
    print(f"Digits: {thresh*100:.0f}% → {k} PCs (out of 64)")

print(f"\nCompression ratio at 95%: {64/np.argmax(cumulative>=0.95)+1:.1f}x")
```

---

## 7.6 Explained Variance and Reconstruction Quality

```python
import numpy as np
from sklearn.decomposition import PCA
from sklearn.datasets import load_digits
from sklearn.preprocessing import StandardScaler

digits = load_digits()
X = StandardScaler().fit_transform(digits.data)

k_values = [2, 5, 10, 20, 30, 50, 64]

print("=== Reconstruction Quality vs. Components ===\n")
print(f"{'k':>4} | {'Var Explained':>13} | {'MSE':>10} | {'Quality':>20}")
print(f"{'-'*4}-+-{'-'*13}-+-{'-'*10}-+-{'-'*20}")

for k in k_values:
    pca = PCA(n_components=k)
    X_reduced = pca.fit_transform(X)
    X_reconstructed = pca.inverse_transform(X_reduced)
    
    var_explained = sum(pca.explained_variance_ratio_)
    mse = np.mean((X - X_reconstructed) ** 2)
    
    quality = "█" * int(var_explained * 20)
    print(f"{k:>4} | {var_explained*100:>12.1f}% | {mse:>10.4f} | {quality}")
```

---

## 7.7 Summary Table

| Metric | Formula | Range | Interpretation |
|--------|---------|-------|----------------|
| **Explained variance** | λₖ | [0, ∞) | Absolute variance of PCk |
| **Explained variance ratio** | λₖ/Σλ | [0, 1] | Fraction of total variance by PCk |
| **Cumulative EVR** | Σᵢ₌₁ᵏ λᵢ/Σλ | [0, 1] | Total variance in first k PCs |
| **Unexplained** | 1 - CEVR(k) | [0, 1] | Information lost |
| **Total variance** | Σλ = trace(S) | [0, ∞) | Sum of all feature variances |

---

## 7.8 Revision Questions

1. **Define explained variance ratio** mathematically. How is it computed from eigenvalues? From singular values?

2. **The first 3 PCs of a 10-feature dataset** have eigenvalues λ₁=5, λ₂=3, λ₃=1 and the total variance is 12. Compute the individual and cumulative explained variance ratios for these 3 PCs.

3. **Interpret the following variance pattern:** PC1=15%, PC2=14%, PC3=12%, PC4=11%, PC5=10%, ..., PC10=3%. What does this tell you about the data structure? Would PCA be effective?

4. **Explain the relationship** between explained variance and reconstruction error. If you keep 95% of variance, what can you say about the reconstruction quality?

5. **Why is the explained variance ratio** always monotonically decreasing? Prove this from the eigenvalue ordering.

6. **Compare two datasets:** Dataset A has 50 features with 95% variance in 5 PCs. Dataset B has 50 features with 95% variance in 40 PCs. What does this tell you about the intrinsic dimensionality of each?

---

[← Previous: Choosing Components](./06-choosing-components.md) | [Next: PCA for Visualization →](./08-pca-for-visualization.md)
