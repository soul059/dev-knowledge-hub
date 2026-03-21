# 📘 7.6 — Feature Importance in Decision Trees

> **Unit 7 · Decision Trees** — Study Notes  
> *Part 6 of 7 — How trees rank features, Mean Decrease in Impurity (MDI), permutation importance, caveats, and Python visualisations.*

---

## 📑 Table of Contents

1. [Why Feature Importance Matters](#1-why-feature-importance-matters)
2. [How Trees Compute Feature Importance — Impurity-Based](#2-how-trees-compute-feature-importance--impurity-based)
3. [Mean Decrease in Impurity (MDI) / Gini Importance](#3-mean-decrease-in-impurity-mdi--gini-importance)
4. [Worked Example — MDI Calculation](#4-worked-example--mdi-calculation)
5. [Permutation Importance](#5-permutation-importance)
6. [MDI vs Permutation Importance](#6-mdi-vs-permutation-importance)
7. [Caveats and Pitfalls](#7-caveats-and-pitfalls)
8. [Interpreting Feature Importance Plots](#8-interpreting-feature-importance-plots)
9. [sklearn: feature_importances_](#9-sklearn-feature_importances_)
10. [Python — Complete Feature Importance Pipeline](#10-python--complete-feature-importance-pipeline)
11. [Applications](#11-applications)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)
14. [Navigation](#14-navigation)

---

## 1. Why Feature Importance Matters

Feature importance answers: **"Which features are most useful for making predictions?"**

```
┌───────────────────────────────────────────────────────────┐
│  Feature Importance Use Cases                              │
│                                                            │
│  1. Feature selection → remove unimportant features        │
│  2. Model interpretability → explain what drives predictions│
│  3. Data understanding → discover key patterns              │
│  4. Debugging → verify the model uses sensible features     │
│  5. Domain knowledge validation → does the model agree?     │
│  6. Communication → show stakeholders what matters          │
└───────────────────────────────────────────────────────────┘
```

---

## 2. How Trees Compute Feature Importance — Impurity-Based

### 2.1 The Core Idea

Every time a tree splits on a feature, it **reduces impurity** (Gini or Entropy). Features that reduce impurity the most — and do so early (affecting more samples) — are more important.

```
          [Feature A ≤ 5]         ← Feature A splits 1000 samples
          /              \           Impurity decrease = 0.15
      [Feat B ≤ 3]    [Feat A ≤ 8]  ← Feature B splits 600 samples
       /      \         /      \        Impurity decrease = 0.08
    [Leaf]  [Leaf]  [Leaf]  [Leaf]

  Feature A: used at 2 nodes, affects many samples → HIGH importance
  Feature B: used at 1 node, affects fewer samples → LOWER importance
```

### 2.2 Formula

For a feature *f*, its importance is the **weighted sum of impurity decreases** at all nodes where *f* is used for splitting:

```
                        Σ         nₜ
Importance(f) =    ──────────  × ─── × ΔImpurity(t)
               t where f is      N
               the split feature
```

Where:
- nₜ = number of samples reaching node *t*
- N = total number of training samples
- ΔImpurity(t) = impurity decrease at node *t*

```
ΔImpurity(t) = Impurity(t) − [ (nₗ/nₜ) × Impurity(left) + (nᵣ/nₜ) × Impurity(right) ]
```

The importances are then **normalised** so they sum to 1.

---

## 3. Mean Decrease in Impurity (MDI) / Gini Importance

### 3.1 Definition

**Mean Decrease in Impurity (MDI)**, also called **Gini importance** (when Gini is the criterion), is the default feature importance method in sklearn.

```
For each feature f:
    MDI(f) = Σ  (nₜ / N) × ΔGini(t)     for all nodes t that split on f
    
Normalised:  MDI_norm(f) = MDI(f) / Σ MDI(all features)
```

### 3.2 Properties of MDI

| Property | Detail |
|----------|--------|
| Always ≥ 0 | Impurity can only decrease after a split |
| Sums to 1 | After normalisation |
| Computed during training | No additional computation needed |
| Depends on tree structure | Different trees → different importances |
| Biased toward high-cardinality features | More split opportunities → higher importance |
| Biased toward continuous features | More threshold candidates than categorical |

### 3.3 How sklearn Computes It

From sklearn's source code, the importance of feature *f* for a tree is:

```python
# Pseudocode of sklearn's implementation
for each node t in tree:
    if t is not a leaf:
        f = feature used at node t
        weighted_decrease = (n_samples[t] / n_total) * (
            impurity[t] 
            - (n_samples[left] / n_samples[t]) * impurity[left]
            - (n_samples[right] / n_samples[t]) * impurity[right]
        )
        importance[f] += weighted_decrease

# Normalise
importance /= importance.sum()
```

---

## 4. Worked Example — MDI Calculation

### 4.1 Tree Structure

```
          [Feature 0 ≤ 2.5]           Node 0: n=100, Gini=0.480
           /              \
    [Feature 1 ≤ 1.8]   [Feature 0 ≤ 4.5]   Node 1: n=60, Gini=0.444
      /          \          /          \       Node 2: n=40, Gini=0.420
   Leaf A     Leaf B    Leaf C      Leaf D    Node 3-6: leaves
   n=30       n=30      n=25       n=15
   Gini=0.0   Gini=0.1  Gini=0.0   Gini=0.05
```

### 4.2 Compute ΔGini at Each Node

**Node 0** (splits on Feature 0):

```
ΔGini(0) = 0.480 − [(60/100) × 0.444 + (40/100) × 0.420]
         = 0.480 − [0.2664 + 0.1680]
         = 0.480 − 0.4344
         = 0.0456
         
Weighted: (100/100) × 0.0456 = 0.0456
```

**Node 1** (splits on Feature 1):

```
ΔGini(1) = 0.444 − [(30/60) × 0.0 + (30/60) × 0.1]
         = 0.444 − [0.0 + 0.05]
         = 0.394

Weighted: (60/100) × 0.394 = 0.2364
```

**Node 2** (splits on Feature 0):

```
ΔGini(2) = 0.420 − [(25/40) × 0.0 + (15/40) × 0.05]
         = 0.420 − [0.0 + 0.01875]
         = 0.40125

Weighted: (40/100) × 0.40125 = 0.16050
```

### 4.3 Aggregate by Feature

| Feature | Nodes Used | Raw Importance | Normalised |
|---------|-----------|----------------|------------|
| Feature 0 | Node 0, Node 2 | 0.0456 + 0.1605 = **0.2061** | 0.2061 / 0.4425 = **0.4658** |
| Feature 1 | Node 1 | **0.2364** | 0.2364 / 0.4425 = **0.5342** |
| **Total** | | **0.4425** | **1.0000** |

**Feature 1 is more important** — it produces a larger impurity decrease at its split, despite being used only once.

---

## 5. Permutation Importance

### 5.1 Concept

**Permutation importance** measures feature importance by **randomly shuffling** a feature's values and observing how much the model's performance **degrades**.

```
Original data:           Shuffled Feature 2:

  F1   F2   F3   y        F1   F2    F3   y
  1.0  3.2  0.5  A        1.0  0.8*  0.5  A   ← F2 values randomly
  2.0  1.5  0.8  B        2.0  3.2*  0.8  B      reassigned
  3.0  0.8  1.2  A        3.0  1.5*  1.2  A
                                ↑
                          shuffled column

If performance drops a lot → Feature 2 is important
If performance stays the same → Feature 2 is not important
```

### 5.2 Algorithm

```
function PERMUTATION_IMPORTANCE(model, X_val, y_val, n_repeats):
    baseline_score = model.score(X_val, y_val)
    
    for each feature f:
        importance_scores = []
        for r in 1..n_repeats:
            X_shuffled = copy(X_val)
            X_shuffled[:, f] = random_permutation(X_val[:, f])
            shuffled_score = model.score(X_shuffled, y_val)
            importance_scores.append(baseline_score - shuffled_score)
        
        importance[f] = mean(importance_scores)
        importance_std[f] = std(importance_scores)
    
    return importance, importance_std
```

### 5.3 Properties

| Property | Detail |
|----------|--------|
| **Model-agnostic** | Works with any model, not just trees |
| **Computed on validation set** | Reflects generalisation, not training fit |
| **Can be negative** | If shuffling *improves* performance (noisy feature) |
| **Accounts for interactions** | Shuffling breaks feature's interactions |
| **Unbiased** | Not affected by cardinality or scale |
| **Slower to compute** | Requires re-evaluation for each feature × n_repeats |

### 5.4 sklearn Implementation

```python
from sklearn.inspection import permutation_importance

result = permutation_importance(
    model, X_test, y_test,
    n_repeats=30,
    random_state=42,
    scoring='accuracy',
)

# result.importances_mean → array of mean importance per feature
# result.importances_std  → array of std per feature
# result.importances      → full matrix (n_features × n_repeats)
```

---

## 6. MDI vs Permutation Importance

### 6.1 Comparison Table

| Aspect | MDI (Gini Importance) | Permutation Importance |
|--------|----------------------|------------------------|
| **How** | Sum of weighted impurity decreases | Performance drop after shuffling |
| **Computed on** | Training data (during fit) | Validation/test data |
| **Speed** | Very fast (byproduct of training) | Slower (needs re-evaluation) |
| **Model-specific** | Tree-specific | Model-agnostic |
| **Bias** | Biased toward high-cardinality & continuous features | Unbiased |
| **Handles correlations** | Poorly (inflates correlated features) | Also affected, but less biased |
| **Can be negative** | No (always ≥ 0) | Yes (noisy features) |
| **sklearn attribute** | `model.feature_importances_` | `permutation_importance()` |

### 6.2 When to Use Which

```
Use MDI when:
  ✅ Quick exploratory analysis
  ✅ All features are similar in type and cardinality
  ✅ You just want a rough ranking

Use Permutation Importance when:
  ✅ You need reliable, unbiased importance
  ✅ Features have different cardinalities
  ✅ You want to assess importance on validation data
  ✅ You're comparing across different model types
  ✅ High-stakes decisions (medical, financial)
```

---

## 7. Caveats and Pitfalls

### 7.1 Correlated Features

When two features are **highly correlated**, both MDI and permutation importance can give misleading results:

```
Example: Feature A and Feature B are 95% correlated.

MDI result:
  Feature A: 0.40    ← Split on A; B is redundant
  Feature B: 0.05    ← Rarely used (A already captures the info)
  
  Misleading! Feature B is actually just as predictive as A.

Permutation result:
  Feature A: 0.15    ← Shuffling A hurts, but B partially compensates
  Feature B: 0.12    ← Shuffling B hurts, but A partially compensates
  
  Both underestimated! The correlated partner absorbs some of the impact.
```

**Solutions:**
- Use **drop-column importance** (retrain without the feature — most accurate but expensive).
- Group correlated features together.
- Use SHAP values for more nuanced attribution.

### 7.2 High-Cardinality Bias (MDI)

```
Feature "ZipCode" (10,000 unique values):
  → Many possible split points
  → High chance of finding a "good" split by chance
  → MDI: HIGH importance (misleading!)
  
Feature "Gender" (2 values):
  → Few split points
  → Even if Gender is truly predictive, MDI may rank it lower

Solution: Use permutation importance instead.
```

### 7.3 Random/Noisy Features

```
Expected: Random feature should have 0 importance

MDI:  Random feature may get non-zero importance
      (can always reduce impurity on training data by chance)

Permutation (on validation data):
      Random feature → near-zero or negative importance ✅
```

### 7.4 Features Not Used in the Tree

If a feature was never selected for splitting (because another feature was always better), its MDI importance = 0. But this doesn't mean the feature is useless — it might be important if the more dominant feature were removed.

### 7.5 Multicollinearity Summary

```
Number of     MDI Reliability     Permutation      Recommended
Correlated                        Reliability       Method
Features
──────────────────────────────────────────────────────────────
None          ✅ Reliable          ✅ Reliable       Either
Low (r<0.5)   ✅ Mostly OK         ✅ Mostly OK      Either
Medium        ⚠️  Skewed            ⚠️  Dampened      Drop-column
High (r>0.9)  ❌ Misleading        ⚠️  Dampened      SHAP / Drop-col
```

---

## 8. Interpreting Feature Importance Plots

### 8.1 Horizontal Bar Chart (Most Common)

```
Feature Importance (MDI)
═══════════════════════════════════════════════

petal_width   ████████████████████████████  0.44
petal_length  █████████████████████████     0.40
sepal_length  ████                          0.07
sepal_width   ████████                      0.09

              0.0    0.1    0.2    0.3    0.4
```

**How to read:**
- Longer bar = more important feature.
- `petal_width` and `petal_length` dominate → they drive most predictions.
- `sepal_length` and `sepal_width` have low importance → they contribute little.

### 8.2 Box Plot (Permutation Importance)

```
Permutation Importance (30 repeats)
═══════════════════════════════════════════════

petal_width   ├──[====|=====]──┤    0.25 ± 0.04
petal_length  ├──[===|====]──┤      0.22 ± 0.03
sepal_width   ├[==|=]┤              0.03 ± 0.02
sepal_length  ├[=|=]┤              0.02 ± 0.01

              0.0    0.1    0.2    0.3
```

**How to read:**
- Box shows variability across repeats.
- Wider box = less reliable estimate.
- If the box crosses zero → feature may not be truly important.

---

## 9. sklearn: feature_importances_

### 9.1 The Attribute

After fitting a `DecisionTreeClassifier` or `DecisionTreeRegressor`, the `.feature_importances_` attribute contains the normalised MDI importances:

```python
from sklearn.tree import DecisionTreeClassifier

clf = DecisionTreeClassifier(random_state=42)
clf.fit(X_train, y_train)

importances = clf.feature_importances_
# array([0.07, 0.09, 0.40, 0.44])
# These correspond to features in order: X[:, 0], X[:, 1], X[:, 2], X[:, 3]

# Properties
print(f"Sum: {importances.sum():.4f}")       # 1.0000
print(f"All ≥ 0: {(importances >= 0).all()}")  # True
print(f"Shape: {importances.shape}")          # (n_features,)
```

### 9.2 For Ensembles

In ensemble models (Random Forest, Gradient Boosting), `.feature_importances_` is the **average** across all trees:

```python
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X_train, y_train)

# Average MDI across 100 trees
importances = rf.feature_importances_

# You can also get per-tree importances
per_tree = np.array([tree.feature_importances_ for tree in rf.estimators_])
# per_tree.shape = (100, n_features)
stds = per_tree.std(axis=0)
```

---

## 10. Python — Complete Feature Importance Pipeline

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import load_breast_cancer
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.inspection import permutation_importance

# ── Load data ──
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.3, random_state=42
)
feature_names = data.feature_names

# ──────────────────────────────────────────────────────
#  1. MDI Importance (Decision Tree)
# ──────────────────────────────────────────────────────
dt = DecisionTreeClassifier(max_depth=5, random_state=42)
dt.fit(X_train, y_train)

mdi_importances = dt.feature_importances_
sorted_idx = np.argsort(mdi_importances)

print("=== MDI Importance (Decision Tree, top 10) ===")
for i in sorted_idx[-10:][::-1]:
    print(f"  {feature_names[i]:30s} {mdi_importances[i]:.4f}")

# ──────────────────────────────────────────────────────
#  2. MDI Importance (Random Forest — more stable)
# ──────────────────────────────────────────────────────
rf = RandomForestClassifier(n_estimators=200, max_depth=7, random_state=42)
rf.fit(X_train, y_train)

rf_importances = rf.feature_importances_
rf_sorted = np.argsort(rf_importances)

# Per-tree variability
per_tree_imp = np.array([t.feature_importances_ for t in rf.estimators_])
rf_stds = per_tree_imp.std(axis=0)

print("\n=== MDI Importance (Random Forest, top 10) ===")
for i in rf_sorted[-10:][::-1]:
    print(f"  {feature_names[i]:30s} {rf_importances[i]:.4f} ± {rf_stds[i]:.4f}")

# ──────────────────────────────────────────────────────
#  3. Permutation Importance
# ──────────────────────────────────────────────────────
perm_result = permutation_importance(
    rf, X_test, y_test,
    n_repeats=30,
    random_state=42,
    scoring='accuracy',
)

perm_sorted = np.argsort(perm_result.importances_mean)

print("\n=== Permutation Importance (Random Forest, top 10) ===")
for i in perm_sorted[-10:][::-1]:
    print(f"  {feature_names[i]:30s} "
          f"{perm_result.importances_mean[i]:.4f} ± "
          f"{perm_result.importances_std[i]:.4f}")

# ──────────────────────────────────────────────────────
#  4. Visualisation — Side by Side
# ──────────────────────────────────────────────────────
fig, axes = plt.subplots(1, 3, figsize=(20, 8))

# Plot 1: Decision Tree MDI
top_n = 15
top_idx = np.argsort(mdi_importances)[-top_n:]
axes[0].barh(range(top_n), mdi_importances[top_idx], color='steelblue')
axes[0].set_yticks(range(top_n))
axes[0].set_yticklabels(feature_names[top_idx], fontsize=9)
axes[0].set_xlabel('Importance')
axes[0].set_title('Decision Tree — MDI Importance')

# Plot 2: Random Forest MDI
top_idx_rf = np.argsort(rf_importances)[-top_n:]
axes[1].barh(range(top_n), rf_importances[top_idx_rf],
             xerr=rf_stds[top_idx_rf], color='forestgreen', alpha=0.8)
axes[1].set_yticks(range(top_n))
axes[1].set_yticklabels(feature_names[top_idx_rf], fontsize=9)
axes[1].set_xlabel('Importance')
axes[1].set_title('Random Forest — MDI Importance')

# Plot 3: Permutation Importance
top_idx_perm = np.argsort(perm_result.importances_mean)[-top_n:]
axes[2].barh(range(top_n), perm_result.importances_mean[top_idx_perm],
             xerr=perm_result.importances_std[top_idx_perm],
             color='coral', alpha=0.8)
axes[2].set_yticks(range(top_n))
axes[2].set_yticklabels(feature_names[top_idx_perm], fontsize=9)
axes[2].set_xlabel('Importance')
axes[2].set_title('Random Forest — Permutation Importance')

plt.suptitle('Feature Importance: MDI vs Permutation', fontsize=14, y=1.01)
plt.tight_layout()
plt.savefig('feature_importance_comparison.png', dpi=150, bbox_inches='tight')
plt.show()

# ──────────────────────────────────────────────────────
#  5. Demonstrating MDI bias with a random feature
# ──────────────────────────────────────────────────────
print("\n=== BIAS DEMO: Adding a random feature ===")

rng = np.random.RandomState(42)
X_train_aug = np.column_stack([X_train, rng.randn(X_train.shape[0])])
X_test_aug = np.column_stack([X_test, rng.randn(X_test.shape[0])])

dt_aug = DecisionTreeClassifier(max_depth=10, random_state=42)
dt_aug.fit(X_train_aug, y_train)

aug_importances = dt_aug.feature_importances_
random_feat_mdi = aug_importances[-1]

perm_aug = permutation_importance(dt_aug, X_test_aug, y_test,
                                  n_repeats=30, random_state=42)
random_feat_perm = perm_aug.importances_mean[-1]

print(f"  Random feature MDI importance:         {random_feat_mdi:.4f}")
print(f"  Random feature Permutation importance:  {random_feat_perm:.4f}")
print(f"  → MDI gives non-zero importance to random noise!")
print(f"  → Permutation importance correctly identifies it as useless.")
```

---

## 11. Applications

| Domain | Feature Importance Application |
|--------|--------------------------------|
| **Healthcare** | Identify which symptoms/biomarkers are most predictive of a disease |
| **Finance** | Determine which financial indicators most influence credit risk |
| **Marketing** | Find which customer attributes best predict churn |
| **Manufacturing** | Discover which process parameters most affect defect rates |
| **Genomics** | Rank genes by their association with a phenotype |
| **Environmental science** | Identify key climate variables affecting biodiversity |

---

## 12. Summary Table

| Concept | Key Points |
|---------|-----------|
| **Feature importance** | Ranks features by their contribution to predictions |
| **MDI / Gini importance** | Sum of weighted impurity decreases; fast; biased |
| **Permutation importance** | Shuffle feature, measure performance drop; unbiased; slower |
| **sklearn attribute** | `model.feature_importances_` → MDI |
| **sklearn function** | `permutation_importance()` → permutation-based |
| **Correlated features** | Both methods can be misleading; use drop-column or SHAP |
| **High-cardinality bias** | MDI inflates importance of features with many unique values |
| **Random features** | MDI: non-zero (spurious). Permutation: ~zero (correct) |
| **Best practice** | Use permutation importance on validation set for reliable results |
| **Ensemble advantage** | Random Forest MDI is more stable than single-tree MDI |

---

## 13. Revision Questions

1. **Explain** how Mean Decrease in Impurity (MDI) is calculated. Write the formula and describe each component.

2. A decision tree splits on Feature A at three nodes (with weighted impurity decreases of 0.10, 0.05, 0.03) and Feature B at one node (weighted decrease 0.15). **Compute** the normalised importance of each feature.

3. **Describe** the permutation importance algorithm. Why is it considered less biased than MDI?

4. Feature X has MDI importance of 0.35, but permutation importance of 0.02. **What might explain this discrepancy?** (Hint: consider training vs validation, overfitting.)

5. You have two highly correlated features (r = 0.95). **Explain** how correlation affects both MDI and permutation importance. What alternative would you recommend?

6. **Why does sklearn give non-zero MDI importance to a purely random feature?** What type of importance correctly identifies it as useless?

---

## 14. Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← 7.5 Advantages & Limitations](05-advantages-and-limitations.md) | **7.6 Feature Importance** | [7.7 Decision Trees for Regression →](07-decision-trees-regression.md) |

| Module | Link |
|--------|------|
| 7.1 | [Tree Structure & Terminology](01-tree-structure-terminology.md) |
| 7.2 | [Splitting Criteria](02-splitting-criteria.md) |
| 7.3 | [Building a Decision Tree](03-building-decision-tree.md) |
| 7.4 | [Pruning Techniques](04-pruning-techniques.md) |
| 7.5 | [Advantages & Limitations](05-advantages-and-limitations.md) |
| 7.6 | **Feature Importance** (you are here) |
| 7.7 | [Decision Trees for Regression](07-decision-trees-regression.md) |

---

> *"Not all features are created equal — the tree knows which ones matter."*
