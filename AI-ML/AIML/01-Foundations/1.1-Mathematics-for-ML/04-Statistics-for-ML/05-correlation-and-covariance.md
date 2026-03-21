# Chapter 5: Correlation and Covariance

> **Unit 4 — Statistics for ML** | Measuring relationships between features

---

## 5.1 Overview

Before feeding features into a model, you need to know *how* they relate.
Covariance and correlation quantify the direction and strength of linear
relationships — essential for feature selection, PCA, and detecting
multicollinearity.

| Concept | Tells you |
|---|---|
| Covariance | Direction of relationship (+ or −) |
| Pearson correlation | Direction AND strength (−1 to +1) |
| Spearman correlation | Strength of monotonic relationship |
| Covariance matrix | All pairwise covariances |

---

## 5.2 Covariance

### Definition

Covariance measures how two variables change together.

```
                1    n
  Cov(X, Y) = ───  Σ  (xᵢ − x̄)(yᵢ − ȳ)      (population)
                n  i=1

                 1     n
  Cov(X, Y) = ─────  Σ  (xᵢ − x̄)(yᵢ − ȳ)    (sample)
               n − 1  i=1
```

### Interpretation

| Cov(X, Y) | Meaning |
|---|---|
| > 0 | X and Y increase together |
| < 0 | X increases → Y decreases |
| ≈ 0 | No linear relationship |

### Limitation

Covariance is **scale-dependent** — the magnitude depends on the units of X
and Y, making it hard to compare across variable pairs.

### Example 1

```
X: [1, 2, 3, 4, 5]     x̄ = 3
Y: [2, 4, 5, 4, 5]     ȳ = 4

(xᵢ − x̄):  −2, −1,  0, 1,  2
(yᵢ − ȳ):  −2,  0,  1, 0,  1

Products:    4,  0,  0, 0,  2  → Sum = 6

Cov(X, Y) = 6 / (5 − 1) = 1.5   (positive → move together)
```

---

## 5.3 Pearson Correlation Coefficient (r)

### Formula

```
               Cov(X, Y)
  r(X, Y) = ─────────────
              σ_X × σ_Y
```

This **normalises** covariance to the range [−1, +1].

### Interpretation Scale

```
  −1.0          −0.5           0           +0.5          +1.0
   ├──────────────┼────────────┼────────────┼──────────────┤
   Strong        Moderate     None       Moderate       Strong
   negative      negative                positive      positive
```

### Properties

- Dimensionless (no units).
- Only captures **linear** relationships.
- r = 0 does NOT mean independent (could be non-linear).

### Example 2

```
From Example 1: Cov(X, Y) = 1.5
σ_X = √(Var(X)) = √(10/4) = √2.5 ≈ 1.581
σ_Y = √(Var(Y)) = √(6/4)  = √1.5  ≈ 1.225

r = 1.5 / (1.581 × 1.225) = 1.5 / 1.936 ≈ 0.775

Strong positive linear correlation.
```

### Python

```python
import numpy as np

X = np.array([1, 2, 3, 4, 5])
Y = np.array([2, 4, 5, 4, 5])

# Covariance matrix (2×2)
cov_matrix = np.cov(X, Y)
print("Cov(X,Y):", cov_matrix[0, 1])  # 1.5

# Pearson correlation
corr = np.corrcoef(X, Y)
print("r:", corr[0, 1])  # 0.7746
```

---

## 5.4 Spearman Rank Correlation (ρₛ)

Measures the strength of a **monotonic** (not necessarily linear) relationship
using ranks instead of raw values.

### Steps

```
1. Rank both X and Y.
2. Compute Pearson r on the ranks.
```

### Formula (shortcut for no ties)

```
               6 Σ dᵢ²
  ρₛ = 1 − ─────────────
             n(n² − 1)

  where dᵢ = rank(xᵢ) − rank(yᵢ)
```

### When to Use

| Situation | Use |
|---|---|
| Linear relationship, continuous data | Pearson |
| Monotonic but non-linear, or ordinal data | Spearman |
| Outliers present | Spearman (more robust) |

### Python

```python
from scipy.stats import spearmanr, pearsonr

X = [1, 2, 3, 4, 5]
Y = [1, 8, 27, 64, 125]  # Y = X³ (monotonic, non-linear)

print("Pearson r: ", pearsonr(X, Y))    # r ≈ 0.96
print("Spearman ρ:", spearmanr(X, Y))   # ρ = 1.0 (perfect monotonic)
```

---

## 5.5 Correlation ≠ Causation

```
  ┌──────────────┐         ┌──────────────┐
  │ Ice cream    │ ──r──►  │  Drowning    │
  │ sales        │         │  deaths      │
  └──────┬───────┘         └──────┬───────┘
         │                        │
         └──── Confound: ─────────┘
               Temperature (lurking variable)
```

**Key points:**
- Correlation shows association, not causation.
- Confounding variables can create spurious correlations.
- Causation requires controlled experiments or causal inference methods.

---

## 5.6 Covariance Matrix

For a dataset with d features, the covariance matrix **C** is a d × d
symmetric matrix where entry Cᵢⱼ = Cov(Xᵢ, Xⱼ).

```
        ⎡ Var(X₁)     Cov(X₁,X₂)  Cov(X₁,X₃) ⎤
  C  =  ⎢ Cov(X₂,X₁)  Var(X₂)     Cov(X₂,X₃) ⎥
        ⎣ Cov(X₃,X₁)  Cov(X₃,X₂)  Var(X₃)     ⎦
```

**Properties:**
- Diagonal = variances of each feature.
- Symmetric: Cᵢⱼ = Cⱼᵢ.
- Used in PCA (eigendecomposition of C).

### Python

```python
import numpy as np
import pandas as pd

df = pd.DataFrame({
    'height': [170, 165, 180, 175, 160],
    'weight': [65, 60, 80, 72, 55],
    'age':    [30, 25, 35, 28, 22]
})

cov_matrix = df.cov()
print(cov_matrix)
```

---

## 5.7 Correlation Matrix

Normalised version of the covariance matrix where each entry is a Pearson r.

```
        ⎡  1.00   r₁₂   r₁₃ ⎤
  R  =  ⎢  r₂₁   1.00   r₂₃ ⎥
        ⎣  r₃₁   r₃₂   1.00 ⎦
```

Diagonal is always 1.00 (each variable is perfectly correlated with itself).

### Python — Correlation Matrix & Heatmap

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

corr_matrix = df.corr()
print(corr_matrix)

# Heatmap
plt.figure(figsize=(8, 6))
sns.heatmap(corr_matrix, annot=True, cmap='coolwarm',
            vmin=-1, vmax=1, center=0, fmt='.2f')
plt.title("Feature Correlation Heatmap")
plt.tight_layout()
plt.show()
```

### Reading a Heatmap (ASCII Representation)

```
           height  weight   age
  height  [ 1.00   0.95   0.87 ]     Dark red  = strong positive
  weight  [ 0.95   1.00   0.80 ]     White     = no correlation
  age     [ 0.87   0.80   1.00 ]     Dark blue = strong negative
```

---

## 5.8 Feature Selection Using Correlation

### Strategy 1: Remove Redundant Features

If two features have |r| > 0.9, they carry nearly the same information.
Drop one to reduce dimensionality and multicollinearity.

```python
# Find highly correlated feature pairs
threshold = 0.9
corr = df.corr().abs()

# Upper triangle (avoid duplicates)
upper = corr.where(np.triu(np.ones(corr.shape), k=1).astype(bool))

# Columns to drop
to_drop = [col for col in upper.columns if any(upper[col] > threshold)]
print("Drop:", to_drop)
df_reduced = df.drop(columns=to_drop)
```

### Strategy 2: Correlation with Target

Select features most correlated with the target variable.

```python
target_corr = df.corr()['target'].drop('target').abs().sort_values(ascending=False)
print(target_corr)

# Keep top-k features
top_features = target_corr.head(10).index.tolist()
```

---

## 5.9 Multicollinearity

When two or more features are highly correlated, the model has trouble
isolating individual effects.

### Problems

- Inflated standard errors in linear regression coefficients.
- Unstable coefficients (small data change → big coefficient change).
- Misleading feature importance.

### Detection — Variance Inflation Factor (VIF)

```
            1
  VIF_j = ─────────
          1 − R²_j

  where R²_j is the R² from regressing feature j on all other features.
```

| VIF | Interpretation |
|---|---|
| 1 | No collinearity |
| 1–5 | Moderate |
| > 5 | High (investigate) |
| > 10 | Severe (take action) |

### Python — VIF Calculation

```python
from statsmodels.stats.outliers_influence import variance_inflation_factor
import pandas as pd

X = df[['feature1', 'feature2', 'feature3']]
vif_data = pd.DataFrame()
vif_data['Feature'] = X.columns
vif_data['VIF'] = [
    variance_inflation_factor(X.values, i) for i in range(X.shape[1])
]
print(vif_data)
```

### Remedies

1. Drop one of the correlated features.
2. Combine features (PCA, averaging).
3. Use regularisation (Ridge, Lasso).
4. Use tree-based models (less affected).

---

## 5.10 Real-World ML Applications

| Application | Technique |
|---|---|
| EDA / feature screening | Correlation matrix heatmap |
| Feature selection | Drop features with |r| > 0.9 |
| PCA | Eigendecomposition of covariance matrix |
| Portfolio theory (finance) | Covariance matrix for risk |
| Multicollinearity diagnosis | VIF |
| Non-linear monotonic relations | Spearman correlation |

---

## Summary Table

| Concept | Formula / Key Idea |
|---|---|
| Covariance | Cov(X,Y) = Σ(xᵢ−x̄)(yᵢ−ȳ)/(n−1) |
| Pearson r | Cov(X,Y) / (σ_X σ_Y), range [−1, +1] |
| Spearman ρ | Pearson r on ranks; captures monotonic |
| Correlation ≠ Causation | Confounders can create spurious r |
| Covariance matrix | d×d matrix, diagonal = variances |
| Correlation matrix | Normalised covariance matrix |
| Multicollinearity | High inter-feature correlation |
| VIF | 1/(1−R²_j); VIF > 10 = severe |

---

## Quick Revision Questions

1. Why is Pearson correlation preferred over raw covariance for comparison?
2. Give an example where Pearson r = 0 but the variables are clearly related.
3. What does the covariance matrix look like after PCA transforms the data?
4. You see two features with r = 0.95. What should you do before training a
   linear regression model?
5. When would Spearman correlation be more appropriate than Pearson?
6. How does Ridge regression help with multicollinearity?

---

*← Previous: [Confidence Intervals](04-confidence-intervals.md) | Next: [Statistical Significance](06-statistical-significance.md) →*
