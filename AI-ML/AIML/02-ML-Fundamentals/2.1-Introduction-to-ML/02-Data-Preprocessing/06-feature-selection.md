# Chapter 6: Feature Selection

> **Unit 2 · Data Preprocessing** — Select the *right* features to build faster, simpler, and more accurate models.

---

## 6.1 Why Feature Selection Matters

### The Curse of Dimensionality
As features grow, data becomes **sparse** in high-dimensional space. Models need exponentially more samples:

```
Samples needed ∝ k^d     (k = bins per feature, d = dimensions)
```

| Features (d) | k=10 bins | Samples Needed |
|:---:|:---:|:---:|
| 2 | 10 | 100 |
| 5 | 10 | 100,000 |
| 10 | 10 | 10 billion |

### Three Core Reasons

| Problem | How Feature Selection Helps |
|---|---|
| **Overfitting** | Fewer irrelevant features → less noise fitting |
| **Training time** | Fewer columns → faster gradient/distance computations |
| **Interpretability** | Smaller feature set → easier to explain predictions |

---

## 6.2 The Three Categories — Overview

```
┌───────────────────────────────────────────────────────────┐
│                 FEATURE SELECTION METHODS                  │
├──────────────────┬──────────────────┬─────────────────────┤
│  FILTER METHODS  │ WRAPPER METHODS  │  EMBEDDED METHODS   │
│  Score features  │ Train models on  │  Selection inside   │
│  with stats;     │ feature subsets;  │  model training;    │
│  model-agnostic  │ model-dependent  │  speed + accuracy   │
│                  │                  │                     │
│ • Correlation    │ • Forward Sel.   │ • L1 / Lasso        │
│ • Chi-Square     │ • Backward Elim. │ • Tree Importance   │
│ • Mutual Info    │ • RFE            │ • ElasticNet        │
│ • Variance Thr.  │                  │                     │
├──────────────────┼──────────────────┼─────────────────────┤
│ Speed: ★★★★★     │ Speed: ★★☆☆☆     │ Speed: ★★★★☆        │
│ Accuracy: ★★★☆☆  │ Accuracy: ★★★★★  │ Accuracy: ★★★★☆     │
└──────────────────┴──────────────────┴─────────────────────┘
```

---

## 6.3 Filter Methods

Filter methods rank features **before** any model is trained.

### 6.3.1 Variance Threshold — Drop near-zero variance features (no information):
```
Var(X) = (1/n) Σ (xᵢ − μ)²
```

### 6.3.2 Correlation Analysis (Pearson)
```
r(X,Y) = Σ(xᵢ−x̄)(yᵢ−ȳ) / √[Σ(xᵢ−x̄)² · Σ(yᵢ−ȳ)²]
|r| > 0.8  →  features are highly correlated → drop one
```

### 6.3.3 Chi-Square Test (χ²) — For **categorical** target with **non-negative** features:
```
χ² = Σ (Oᵢ − Eᵢ)² / Eᵢ       (O = observed, E = expected)
```

### 6.3.4 Mutual Information — Captures any (linear or non-linear) dependency:
```
MI(X;Y) = Σ Σ p(x,y) · log[ p(x,y) / (p(x)·p(y)) ]
MI = 0  →  X and Y are independent
```

### Python — Filter Methods

```python
from sklearn.feature_selection import (
    VarianceThreshold, SelectKBest, chi2, mutual_info_classif
)
import pandas as pd, numpy as np

vt = VarianceThreshold(threshold=0.01)
X_vt = vt.fit_transform(X)
print("Kept:", vt.get_support().sum(), "of", X.shape[1])

skb = SelectKBest(score_func=chi2, k=10)
X_chi = skb.fit_transform(X, y)
print(pd.Series(skb.scores_, index=feature_names).nlargest(10))

skb_mi = SelectKBest(score_func=mutual_info_classif, k=10)
X_mi = skb_mi.fit_transform(X, y)
```

---

## 6.4 Wrapper Methods

Wrappers treat feature selection as a **search problem** — training a model for every candidate subset.

### 6.4.1 Forward Selection
```
Start: {}  →  Try {A},{B},{C} → best={B}  →  Try {B,A},{B,C} → best={B,C}
→  Try {B,C,A} → no improvement → STOP  →  Result: {B, C}
```

### 6.4.2 Backward Elimination
```
Start: {A,B,C,D}  →  Remove each, evaluate → drop D → {A,B,C}
→  Remove each, evaluate → drop A → {B,C}  →  Removing any hurts → STOP
```

### 6.4.3 Recursive Feature Elimination (RFE)
RFE fits a model, **ranks features by importance**, removes the weakest, and repeats:
```
All features → Train model → Rank → Drop least important → Repeat
```

### Python — RFE
```python
from sklearn.feature_selection import RFE, RFECV
from sklearn.ensemble import RandomForestClassifier

estimator = RandomForestClassifier(n_estimators=100, random_state=42)

rfe = RFE(estimator, n_features_to_select=10, step=1)
rfe.fit(X_train, y_train)
print("Selected:", np.array(feature_names)[rfe.support_])

# RFECV — auto-select optimal count via cross-validation
rfecv = RFECV(estimator, step=1, cv=5, scoring='accuracy')
rfecv.fit(X_train, y_train)
print("Optimal features:", rfecv.n_features_)
```

---

## 6.5 Embedded Methods

Feature selection is **built into** the model training algorithm.

### 6.5.1 L1 Regularisation (Lasso)
Lasso adds an L1 penalty that forces some coefficients to **exactly zero**:
```
Loss = Σ(yᵢ − ŷᵢ)² + λ Σ|wⱼ|
λ ↑  →  more coefficients become 0  →  fewer features kept
```

### 6.5.2 Tree-Based Feature Importance
Importance = total **reduction in impurity** (Gini / entropy) across all splits:
```
Importance(fⱼ) = Σ  (n_t / n) · ΔImpurity(t)     over splits on fⱼ
```

### Python — Embedded Methods
```python
from sklearn.linear_model import LassoCV
from sklearn.feature_selection import SelectFromModel
from sklearn.ensemble import GradientBoostingClassifier

# Lasso-based selection
lasso = LassoCV(cv=5, random_state=42).fit(X_train, y_train)
model_lasso = SelectFromModel(lasso, prefit=True)
X_lasso = model_lasso.transform(X_train)
print("Lasso kept:", X_lasso.shape[1], "features")

# Tree-based importance
gb = GradientBoostingClassifier(n_estimators=200, random_state=42)
gb.fit(X_train, y_train)
importances = pd.Series(gb.feature_importances_, index=feature_names)
print(importances.nlargest(10))
X_tree = X_train[importances.nlargest(10).index.tolist()]
```

---

## 6.6 Feature Selection vs Dimensionality Reduction

| Aspect | Feature Selection | Dimensionality Reduction (PCA) |
|---|---|---|
| **Output** | Subset of original features | New transformed features |
| **Interpretability** | High — original columns kept | Low — components are mixtures |
| **Information loss** | Drops some features entirely | Compresses all features |
| **Reversible** | Yes — just re-add columns | Lossy — approximate inverse only |
| **Use when** | You need explainability | You need max variance in fewer dims |

---

## 6.7 Real-World Example — Customer Churn Prediction

```
Dataset: 50 features (call duration, complaints, billing, demographics …)

Step 1  Filter   → VarianceThreshold removes 8 constant cols       → 42
Step 2  Filter   → Correlation > 0.9: drop redundant pairs         → 35
Step 3  Embedded → GradientBoosting importance: keep top 15        → 15
Step 4  Wrapper  → RFECV with LogisticRegression confirms optimal  → 12

Result: 4× faster training, accuracy 81% → 84%, 12 explainable drivers.
```

---

## 6.8 Method Comparison Table

| Method | Category | Speed | Non-Linear? | Model-Agnostic? | Best For |
|---|---|---|---|---|---|
| Variance Threshold | Filter | ★★★★★ | — | Yes | Quick cleanup |
| Correlation | Filter | ★★★★★ | No | Yes | Removing redundancy |
| Chi-Square | Filter | ★★★★☆ | No | Yes | Categorical targets |
| Mutual Information | Filter | ★★★★☆ | Yes | Yes | Non-linear dependencies |
| Forward Selection | Wrapper | ★★☆☆☆ | Depends | No | Small feature sets |
| Backward Elim. | Wrapper | ★★☆☆☆ | Depends | No | Starting from all features |
| RFE / RFECV | Wrapper | ★★☆☆☆ | Depends | No | Optimal subset with CV |
| Lasso (L1) | Embedded | ★★★★☆ | No | No | Linear sparse models |
| Tree Importance | Embedded | ★★★★☆ | Yes | No | Non-linear importance |

---

## 6.9 Summary

| Concept | Key Takeaway |
|---|---|
| Curse of dimensionality | More features → sparser data → worse generalisation |
| Filter methods | Fast, model-agnostic statistical scoring |
| Wrapper methods | Accurate but expensive; search over subsets |
| Embedded methods | Selection during training (Lasso zeros, tree importance) |
| Selection vs Reduction | Selection keeps original features; reduction creates new ones |

---

## 6.10 Revision Questions

1. **Explain** the curse of dimensionality and how feature selection mitigates it.
2. **Compare** filter, wrapper, and embedded methods — give one example of each and state when you would prefer it.
3. A dataset has 200 features with 30 pairs having Pearson |r| > 0.95. **What** do you do and **why**?
4. **Why** does L1 (Lasso) drive coefficients to exactly zero while L2 (Ridge) does not?
5. You have 10,000 features and limited compute. **Which** selection strategy do you apply first?
6. **Differentiate** PCA and feature selection. When would you choose feature selection over PCA?

---

[⬅️ Previous: 05-feature-engineering.md](05-feature-engineering.md) | [➡️ Next: 07-data-splitting-strategies.md](07-data-splitting-strategies.md)
