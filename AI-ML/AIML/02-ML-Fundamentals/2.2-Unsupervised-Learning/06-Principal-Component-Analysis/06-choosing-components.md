# 6. Choosing the Number of Components

[← Previous: SVD Approach](./05-svd-approach.md) | [Next: Explained Variance Ratio →](./07-explained-variance-ratio.md)

---

## Chapter Overview

One of the most important practical decisions in PCA is: **how many principal components k should we keep?** Too few → lose important information. Too many → retain noise, no dimensionality benefit. This chapter covers multiple methods: the **explained variance threshold** (e.g., 95% rule), the **scree plot**, the **Kaiser criterion**, **cumulative variance plots**, and **cross-validation** approaches.

---

## 6.1 The Trade-Off

```
  Information                    Dimensionality
  Preserved                      Reduction
    100% ┤                         100% ┤█████████████████
         │                              │
     90% ┤                          80% ┤████████████████
         │                              │
     80% ┤                          60% ┤██████████████
         │                              │
     70% ┤                          40% ┤████████████
         │                              │
     60% ┤                          20% ┤██████████
         │                              │
     50% ┤                           0% ┤
         └──┬──┬──┬──┬──┬──            └──┬──┬──┬──┬──┬──
            1  2  3  4  5  6               1  2  3  4  5  6
         Components kept               Components kept

  Want HIGH information            Want LOW dimensionality
  → keep MORE components           → keep FEWER components

  Goal: Find the "sweet spot" where we get ENOUGH information
  with the FEWEST components.
```

---

## 6.2 Method 1: Cumulative Variance Threshold (95% Rule)

### The Most Common Method

```
Rule: Keep the smallest k such that cumulative explained variance ≥ threshold

  Common thresholds:
    95% — standard default (most common)
    90% — more aggressive reduction
    99% — conservative (keeps more components)
    85% — used when extreme reduction needed

  Cumulative variance:
    CV(k) = (λ₁ + λ₂ + ... + λₖ) / (λ₁ + λ₂ + ... + λ_d)
    
  Choose k = min{k : CV(k) ≥ 0.95}
```

### Visual Example

```
  Cumulative Explained Variance
  100%│                         ─────────────────
     │                     ╱
  95%│· · · · · · · · · ·╱· · · · · · · · · · ← threshold
     │                  ╱
  90%│               ╱╱
     │             ╱╱
  80%│          ╱╱
     │        ╱╱
  70%│      ╱╱
     │    ╱╱
  50%│  ╱╱
     │╱╱
   0%│
     └──┬──┬──┬──┬──┬──┬──┬──┬──┬──
        1  2  3  4  5  6  7  8  9  10
                       ↑
                    k = 5 needed for 95%
```

### Python Implementation

```python
import numpy as np
from sklearn.decomposition import PCA
from sklearn.datasets import load_digits
from sklearn.preprocessing import StandardScaler

# Load high-dimensional data
digits = load_digits()
X = StandardScaler().fit_transform(digits.data)  # 64 features

# Fit PCA with all components
pca = PCA()
pca.fit(X)

cumulative_var = np.cumsum(pca.explained_variance_ratio_)

# Find k for different thresholds
thresholds = [0.80, 0.85, 0.90, 0.95, 0.99]
print(f"Dataset: {X.shape[0]} samples, {X.shape[1]} features")
print(f"\n{'Threshold':>10} | {'k needed':>8} | {'Reduction':>10}")
print(f"{'-'*10}-+-{'-'*8}-+-{'-'*10}")

for threshold in thresholds:
    k = np.argmax(cumulative_var >= threshold) + 1
    reduction = (1 - k / X.shape[1]) * 100
    print(f"{threshold*100:>9.0f}% | {k:>8} | {reduction:>9.1f}%")

# Using PCA with threshold directly
pca_95 = PCA(n_components=0.95)  # Pass float = variance threshold!
X_reduced = pca_95.fit_transform(X)
print(f"\nPCA(n_components=0.95):")
print(f"  Components selected: {pca_95.n_components_}")
print(f"  Reduced shape: {X_reduced.shape}")
```

---

## 6.3 Method 2: Scree Plot (Elbow Method)

### Concept

```
The scree plot shows eigenvalues (variance per PC) vs. component number.
Look for an "elbow" — a sharp drop-off where eigenvalues become small.

  Eigenvalue
  (Variance)
     │
  10 │██
     │██
   8 │██
     │██ ██
   6 │██ ██
     │██ ██
   4 │██ ██ ██
     │██ ██ ██
   2 │██ ██ ██ ██                        ← ELBOW HERE
     │██ ██ ██ ██ ▪▪ ▪▪ ▪▪ ▪▪ ▪▪ ▪▪    ← noise floor
   0 │██ ██ ██ ██ ▪▪ ▪▪ ▪▪ ▪▪ ▪▪ ▪▪
     └──┬──┬──┬──┬──┬──┬──┬──┬──┬──
        1  2  3  4  5  6  7  8  9  10
              ↑
         Keep 4 components
         (before the elbow)

  Components BEFORE the elbow: Signal (keep)
  Components AFTER the elbow:  Noise (discard)
```

### Python: ASCII Scree Plot

```python
import numpy as np
from sklearn.decomposition import PCA
from sklearn.datasets import load_iris
from sklearn.preprocessing import StandardScaler

X = StandardScaler().fit_transform(load_iris().data)
pca = PCA().fit(X)

print("=== Scree Plot ===\n")
print(f"{'PC':>4} | {'Eigenvalue':>10} | {'% Var':>6} | {'Cum %':>6} | Bar")
print(f"{'-'*4}-+-{'-'*10}-+-{'-'*6}-+-{'-'*6}-+----")

cumulative = 0
for i, (ev, ratio) in enumerate(zip(pca.explained_variance_, 
                                      pca.explained_variance_ratio_)):
    cumulative += ratio
    bar_len = int(ratio * 50)
    print(f"PC{i+1:>2} | {ev:>10.4f} | {ratio*100:>5.1f}% | {cumulative*100:>5.1f}% | {'█' * bar_len}")
```

---

## 6.4 Method 3: Kaiser Criterion (Eigenvalue > 1)

### The Rule

```
Kaiser's Rule: Keep components whose eigenvalue > 1
               (when working with STANDARDIZED data / correlation matrix)

Rationale:
  After standardization, each original feature has variance = 1.
  A principal component with λ > 1 captures more variance
  than a single original feature → worth keeping.
  A PC with λ < 1 captures less than one feature → discard.

  ┌────────────────────────────────────────────────────────────┐
  │  ⚠️ IMPORTANT: Kaiser criterion only applies when PCA is  │
  │  performed on the CORRELATION matrix (standardized data).  │
  │  For the covariance matrix, eigenvalue > 1 has no special │
  │  meaning (depends on the scale of the data).              │
  └────────────────────────────────────────────────────────────┘
```

### Python

```python
# Kaiser criterion
eigenvalues = pca.explained_variance_

# Only valid for standardized data!
k_kaiser = np.sum(eigenvalues > 1.0)
print(f"\nKaiser Criterion: Keep {k_kaiser} components (eigenvalue > 1)")
for i, ev in enumerate(eigenvalues):
    marker = "✓ KEEP" if ev > 1.0 else "✗ DROP"
    print(f"  PC{i+1}: λ = {ev:.4f}  {marker}")
```

---

## 6.5 Method 4: Cross-Validation

### Using Reconstruction Error

```
For supervised learning tasks, use CV on the DOWNSTREAM task:

  For k in [1, 2, 3, ..., d]:
    1. Apply PCA(k) to training data
    2. Train classifier/regressor on reduced data
    3. Evaluate on validation set
    4. Record performance

  Choose k that maximizes validation performance.

This is the GOLD STANDARD when the goal is supervised learning,
because it directly optimizes what matters — prediction quality.
```

```python
import numpy as np
from sklearn.decomposition import PCA
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_digits

digits = load_digits()
X, y = digits.data, digits.target

k_values = [5, 10, 15, 20, 30, 40, 50, 64]

print("=== Cross-Validation for Component Selection ===\n")
print(f"{'k':>4} | {'CV Accuracy':>12} | {'Bar':>30}")
print(f"{'-'*4}-+-{'-'*12}-+-{'-'*30}")

best_k, best_score = 0, 0
for k in k_values:
    pipe = Pipeline([
        ('scaler', StandardScaler()),
        ('pca', PCA(n_components=k)),
        ('clf', LogisticRegression(max_iter=1000, random_state=42))
    ])
    scores = cross_val_score(pipe, X, y, cv=5, scoring='accuracy')
    mean_score = scores.mean()
    
    bar = '█' * int(mean_score * 30)
    marker = ""
    if mean_score > best_score:
        best_score = mean_score
        best_k = k
        marker = " ← best"
    
    print(f"{k:>4} | {mean_score:>11.4f} | {bar}{marker}")

print(f"\nOptimal k = {best_k} (accuracy = {best_score:.4f})")
```

---

## 6.6 Comparison of Methods

| Method | Formula/Rule | Pros | Cons |
|--------|-------------|------|------|
| **95% variance** | min k : CV(k)≥0.95 | Simple, widely used | Arbitrary threshold |
| **Scree plot** | Visual elbow | Intuitive | Subjective, elbow may be unclear |
| **Kaiser** | λₖ > 1 | Simple rule | Only for standardized data, can over/under-select |
| **Cross-validation** | Max downstream metric | Optimal for supervised tasks | Computationally expensive |
| **Parallel analysis** | Compare with random | Statistically rigorous | Complex to implement |

### When to Use Which

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Unsupervised task (clustering, visualization):                  │
│    → 95% variance rule or scree plot                            │
│                                                                  │
│  Supervised task (classification, regression):                   │
│    → Cross-validation on downstream task                         │
│                                                                  │
│  Quick analysis / exploration:                                   │
│    → Kaiser criterion (if data is standardized)                 │
│                                                                  │
│  Publication / rigorous analysis:                                │
│    → Parallel analysis or cross-validation                      │
│                                                                  │
│  sklearn shortcut:                                               │
│    → PCA(n_components=0.95) — automatic 95% selection           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 6.7 Summary Table

| Method | Criterion | Best For | sklearn |
|--------|-----------|----------|---------|
| **Cumulative variance** | CV(k) ≥ threshold | General purpose | `PCA(n_components=0.95)` |
| **Scree plot** | Elbow in eigenvalue plot | Visual analysis | Plot `explained_variance_` |
| **Kaiser** | λₖ > 1 | Standardized data | Check `explained_variance_` |
| **Cross-validation** | Max test performance | Supervised learning | Pipeline + GridSearchCV |
| **Fixed k** | Domain knowledge | Known structure | `PCA(n_components=k)` |

---

## 6.8 Revision Questions

1. **Explain the 95% cumulative variance rule.** How do you implement it in sklearn with a single parameter?

2. **What is a scree plot** and how do you interpret it? What do you do when there is no clear elbow?

3. **Why does the Kaiser criterion** (eigenvalue > 1) only apply to standardized data? What would happen if you applied it to unstandardized data with different-scale features?

4. **When is cross-validation** the best method for choosing k? Why might the optimal k from CV differ from the 95% variance rule?

5. **A dataset with 50 features** has the following cumulative variance: PC1=40%, PC2=55%, PC3=65%, PC4=73%, PC5=79%, PC10=92%, PC15=97%. How many components would each method (95% rule, Kaiser if all λ>1 for first 8) suggest? Which would you choose and why?

6. **What are the risks** of keeping too few or too many components? Relate to bias-variance trade-off.

---

[← Previous: SVD Approach](./05-svd-approach.md) | [Next: Explained Variance Ratio →](./07-explained-variance-ratio.md)
