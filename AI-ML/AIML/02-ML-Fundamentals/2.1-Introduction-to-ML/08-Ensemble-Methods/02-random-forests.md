# Chapter 8.2: Random Forests

> **Unit 8: Ensemble Methods** — File 2 of 8

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Random Forest Algorithm](#2-random-forest-algorithm)
3. [Feature Subsampling — The Key Innovation](#3-feature-subsampling--the-key-innovation)
4. [Why Feature Randomization Helps](#4-why-feature-randomization-helps)
5. [Hyperparameters Deep Dive](#5-hyperparameters-deep-dive)
6. [Out-of-Bag (OOB) Score](#6-out-of-bag-oob-score)
7. [Feature Importance](#7-feature-importance)
8. [Comparison: Random Forest vs Single Decision Tree](#8-comparison-random-forest-vs-single-decision-tree)
9. [Practical Tuning Guide](#9-practical-tuning-guide)
10. [Complete Python Example](#10-complete-python-example)
11. [Applications](#11-applications)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)

---

## 1. Introduction

**Random Forest** is an extension of bagging that adds an extra layer of randomness: at each node split, only a **random subset of features** is considered. This seemingly simple idea leads to one of the most successful and widely used machine learning algorithms.

> "Random Forests grow many classification trees. To classify a new object, put the input vector down each tree. Each tree gives a classification. We say the tree 'votes' for that class. The forest chooses the class having the most votes." — Leo Breiman, 2001

### Why Random Forest Over Plain Bagging?

```
Bagging:
  - Bootstrap samples → Different training data per tree
  - BUT: If one feature dominates, all trees split on it first
  - Trees are CORRELATED → limited variance reduction

Random Forest:
  - Bootstrap samples → Different training data per tree
  - PLUS: Random feature subset at each split
  - Trees are LESS CORRELATED → greater variance reduction

Recall:  Var(ensemble) = ρσ² + (1-ρ)σ²/B
                          ↑
                    RF reduces this ρ
```

---

## 2. Random Forest Algorithm

### Algorithm: Random Forest

```
INPUT:  Training set D = {(x₁,y₁), ..., (xₙ,yₙ)} with p features
        Number of trees B
        Feature subset size m (m ≤ p)

PROCEDURE:
    FOR b = 1, 2, ..., B:
        1. Draw bootstrap sample D*_b of size n from D
        2. Grow a decision tree T_b on D*_b:
           At EACH internal node:
             a. Randomly select m features from p available features
             b. Find the best split among those m features
             c. Split the node into two children
           Grow tree to maximum depth (no pruning)
    END FOR

OUTPUT (Classification):
    ĥ(x) = majority vote of {T₁(x), T₂(x), ..., T_B(x)}

OUTPUT (Regression):
    ĥ(x) = (1/B) Σ T_b(x)
```

### Visual Architecture

```
                         Original Data (n samples, p features)
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                 │                 │
                    ▼                 ▼                 ▼
            Bootstrap D*₁      Bootstrap D*₂      Bootstrap D*_B
                    │                 │                 │
                    ▼                 ▼                 ▼
         ┌──── Tree T₁ ────┐  ┌── Tree T₂ ──┐  ┌── Tree T_B ──┐
         │                  │  │              │  │              │
         │  At each split:  │  │  At each     │  │  At each     │
         │  pick m random   │  │  split:      │  │  split:      │
         │  features, find  │  │  pick m      │  │  pick m      │
         │  best split      │  │  random      │  │  random      │
         │  among them      │  │  features    │  │  features    │
         │                  │  │              │  │              │
         │  ┌───┐           │  │   ┌───┐     │  │   ┌───┐     │
         │  │ f₃│           │  │   │ f₁│     │  │   │ f₇│     │
         │  └─┬─┘           │  │   └─┬─┘     │  │   └─┬─┘     │
         │   / \            │  │    / \      │  │    / \      │
         │ ┌─┐ ┌─┐          │  │  ┌─┐ ┌─┐   │  │  ┌─┐ ┌─┐   │
         │ │f₇│ │f₁│        │  │  │f₅│ │f₃│  │  │  │f₂│ │f₅│  │
         │ └──┘ └──┘        │  │  └──┘ └──┘  │  │  └──┘ └──┘  │
         └──────────────────┘  └──────────────┘  └──────────────┘
                    │                 │                 │
                    ▼                 ▼                 ▼
               Pred: A           Pred: B           Pred: A
                    │                 │                 │
                    └─────────────────┼─────────────────┘
                                      │
                                      ▼
                           ┌─────────────────┐
                           │  Majority Vote:  │
                           │     Class A      │
                           │   (2 out of 3)   │
                           └─────────────────┘
```

---

## 3. Feature Subsampling — The Key Innovation

### How Many Features to Sample?

The parameter `m` (number of features considered at each split) is the most important hyperparameter distinguishing Random Forest from Bagging.

```
┌────────────────────────────────────────────────────┐
│          Feature Subset Size (m) Guidelines        │
├────────────────────────────────────────────────────┤
│                                                    │
│  Classification:  m = √p   (square root of total) │
│  Regression:      m = p/3  (one-third of total)   │
│  Full bagging:    m = p    (all features)          │
│                                                    │
│  Where p = total number of features                │
│                                                    │
│  Examples (p = 100 features):                      │
│    Classification: m = √100 = 10                   │
│    Regression:     m = 100/3 ≈ 33                  │
│                                                    │
│  Examples (p = 20 features):                       │
│    Classification: m = √20 ≈ 4-5                   │
│    Regression:     m = 20/3 ≈ 7                    │
│                                                    │
└────────────────────────────────────────────────────┘
```

### The Spectrum from Bagging to Extra Trees

```
m = p (all features)     m = √p (RF default)      m = 1 (single feature)
        │                        │                        │
        ▼                        ▼                        ▼
   ┌─────────┐            ┌─────────┐             ┌─────────┐
   │ Bagging  │            │ Random  │             │ Extremely│
   │ (high ρ) │            │ Forest  │             │Randomized│
   │          │            │(medium ρ)│            │Trees     │
   │ Low bias │            │ Balance │             │(low ρ)   │
   │ Mod var  │            │ bias/var│             │ High bias│
   │ reduct.  │            │ Good var│             │ Max var  │
   │          │            │ reduct. │             │ reduct.  │
   └─────────┘            └─────────┘             └─────────┘

   ◄──────────── Increasing Randomness ──────────────►
   ◄──── More Correlated ────────── Less Correlated ──►
   ◄──── Lower Bias ──────────────── Higher Bias ────►
   ◄──── Less Var Reduction ──── More Var Reduction ──►
```

### How Feature Subsampling Works at Each Node

```
Available features: {f₁, f₂, f₃, f₄, f₅, f₆, f₇, f₈, f₉}  (p = 9)
Feature subset size: m = 3 (≈√9)

Node A (root):
  Random subset: {f₂, f₅, f₈}
  Best split among {f₂, f₅, f₈}: Split on f₅ ≤ 3.2
                     ┌────┐
                     │ f₅ │
                     │≤3.2│
                     └─┬──┘
                      / \

Node B (left child):
  Random subset: {f₁, f₃, f₇}    ← NEW random draw!
  Best split among {f₁, f₃, f₇}: Split on f₃ ≤ 1.5

Node C (right child):
  Random subset: {f₄, f₆, f₉}    ← Another NEW random draw!
  Best split among {f₄, f₆, f₉}: Split on f₆ ≤ 7.0
```

**Key:** The random feature subset is drawn **independently at every single node**, not once per tree.

---

## 4. Why Feature Randomization Helps

### The Dominant Feature Problem

Without feature randomization, if feature `f₁` is highly predictive:

```
Without feature randomization:

Tree 1:      Tree 2:      Tree 3:      Tree 4:
  ┌─┐          ┌─┐          ┌─┐          ┌─┐
  │f₁│         │f₁│         │f₁│         │f₁│    ← ALL trees split on f₁
  └┬─┘         └┬─┘         └┬─┘         └┬─┘
  / \          / \          / \          / \
 f₃  f₂      f₂  f₃      f₃  f₂      f₂  f₃

Correlation between trees: HIGH (ρ ≈ 0.8)
Variance reduction: SMALL
```

With feature randomization:

```
With feature randomization (m = √p):

Tree 1:      Tree 2:      Tree 3:      Tree 4:
  ┌─┐          ┌─┐          ┌─┐          ┌─┐
  │f₁│         │f₅│         │f₃│         │f₁│    ← DIFFERENT first splits
  └┬─┘         └┬─┘         └┬─┘         └┬─┘
  / \          / \          / \          / \
 f₇  f₄      f₁  f₈      f₆  f₁      f₉  f₂

Correlation between trees: LOW (ρ ≈ 0.3)
Variance reduction: LARGE
```

### Mathematical Justification

Recall from the bagging chapter:

```
Var(RF) = ρσ² + (1-ρ)σ²/B
```

| Method | Typical ρ | Var (B=100, σ²=1) | Improvement |
|--------|-----------|---------------------|-------------|
| Bagging (m=p) | 0.7 | 0.703 | 29.7% |
| RF (m=√p) | 0.3 | 0.307 | 69.3% |
| Extra Trees (m=1) | 0.1 | 0.109 | 89.1% |

The trade-off: smaller m → lower ρ → better variance reduction, but also higher bias because each split is constrained.

---

## 5. Hyperparameters Deep Dive

### Complete Hyperparameter Reference

```python
RandomForestClassifier(
    # === Core Parameters ===
    n_estimators=100,        # Number of trees in the forest
    criterion='gini',        # Split quality: 'gini' or 'entropy'
    max_features='sqrt',     # Features per split: 'sqrt', 'log2', int, float

    # === Tree Depth & Size ===
    max_depth=None,          # Maximum depth of each tree (None = unlimited)
    min_samples_split=2,     # Min samples to split internal node
    min_samples_leaf=1,      # Min samples in leaf node
    max_leaf_nodes=None,     # Max number of leaf nodes

    # === Sampling ===
    bootstrap=True,          # Whether to use bootstrap samples
    max_samples=None,        # Number of samples per bootstrap (None = n)

    # === Evaluation ===
    oob_score=False,         # Compute out-of-bag score

    # === Computational ===
    n_jobs=None,             # Parallel cores (-1 = all)
    random_state=None,       # Reproducibility seed

    # === Class Imbalance ===
    class_weight=None,       # 'balanced', 'balanced_subsample', or dict
)
```

### Hyperparameter Impact Analysis

```
┌──────────────────┬────────────────────┬─────────────────────────┐
│  Hyperparameter  │ Increase Effect    │ Typical Values          │
├──────────────────┼────────────────────┼─────────────────────────┤
│ n_estimators     │ ↓ variance         │ 100-1000                │
│                  │ ↑ computation      │ More is rarely harmful  │
├──────────────────┼────────────────────┼─────────────────────────┤
│ max_features     │ ↑ correlation (ρ)  │ 'sqrt' (clf), 'auto'/   │
│                  │ ↓ bias per tree    │  0.33 (reg)             │
│                  │ ↑ strength of tree │                         │
├──────────────────┼────────────────────┼─────────────────────────┤
│ max_depth        │ ↓ bias per tree    │ None (full), 10-30      │
│                  │ ↑ variance per tree│                         │
│                  │ ↑ overfitting risk │                         │
├──────────────────┼────────────────────┼─────────────────────────┤
│ min_samples_split│ ↑ regularization   │ 2, 5, 10               │
│                  │ ↑ bias             │                         │
│                  │ ↓ variance         │                         │
├──────────────────┼────────────────────┼─────────────────────────┤
│ min_samples_leaf │ ↑ regularization   │ 1, 5, 10               │
│                  │ Smoother boundaries│                         │
├──────────────────┼────────────────────┼─────────────────────────┤
│ max_samples      │ ↓ correlation      │ 0.6-1.0                │
│                  │ ↑ diversity        │                         │
│                  │ Slightly ↑ bias    │                         │
└──────────────────┴────────────────────┴─────────────────────────┘
```

### n_estimators: How Many Trees?

```
Accuracy
  │
  │
  │        .---------.---------.---------
  │      .´
  │    .´
  │  .´
  │.´
  │
  └──────┬──────┬──────┬──────┬──────────► n_estimators
        50    100    200    500   1000

Rules:
  • More trees = better (never hurts accuracy)
  • Diminishing returns after ~100-500 trees
  • Cost: linear increase in training time & memory
  • Start with 100, increase if accuracy still improving
```

---

## 6. Out-of-Bag (OOB) Score

### OOB in Random Forest Context

Each tree in a Random Forest uses a bootstrap sample containing ~63.2% of the data. The remaining ~36.8% (OOB samples) serve as a built-in validation set.

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import cross_val_score

X, y = load_breast_cancer(return_X_y=True)

# Train RF with OOB scoring
rf = RandomForestClassifier(
    n_estimators=500,
    oob_score=True,
    random_state=42,
    n_jobs=-1
)
rf.fit(X, y)

# Compare OOB vs Cross-Validation
cv_scores = cross_val_score(rf, X, y, cv=5, scoring='accuracy')

print(f"OOB Score:          {rf.oob_score_:.4f}")
print(f"5-Fold CV Mean:     {cv_scores.mean():.4f} ± {cv_scores.std():.4f}")
# Typically very close — OOB is a reliable estimate!
```

### When OOB Score Replaces Cross-Validation

```
Traditional Workflow:                 OOB Workflow:
┌──────────────────────┐              ┌──────────────────────┐
│ Split: Train/Val/Test│              │ Split: Train/Test    │
│                      │              │                      │
│ For each fold:       │              │ Train RF on train:   │
│   Train RF           │              │   → OOB score is     │
│   Evaluate on val    │              │     computed FREE     │
│                      │              │                      │
│ Average val scores   │              │ Evaluate on test     │
│ Evaluate on test     │              │                      │
│                      │              │                      │
│ Cost: k × training   │              │ Cost: 1 × training   │
└──────────────────────┘              └──────────────────────┘
```

---

## 7. Feature Importance

### Mean Decrease in Impurity (MDI)

Also called **Gini Importance**. For each feature, sum the weighted impurity decrease across all nodes in all trees where that feature is used for splitting.

```
For feature j:

Importance(j) = (1/B) Σ   Σ     [n_t/n · ΔImpurity(t)]
                     b=1 t∈T_b
                          where t splits on feature j

Where:
  B = number of trees
  T_b = tree b
  n_t = number of samples reaching node t
  n = total training samples
  ΔImpurity(t) = impurity(t) - Σ (n_child/n_t) · impurity(child)
```

### Permutation Importance (More Reliable)

Measures how much the model's performance drops when a feature's values are randomly shuffled.

```
Algorithm: Permutation Importance for feature j
1. Compute baseline score S on test data
2. Randomly shuffle column j of test data
3. Compute new score S_j on shuffled data
4. Importance(j) = S - S_j
5. Repeat steps 2-4 multiple times, average

  Score
    │
    │ ████  Baseline score
    │ ████
    │ ████  ▓▓▓▓  Score after shuffling f₁ (big drop → important)
    │ ████  ▓▓▓▓
    │ ████  ▓▓▓▓  ░░░░  Score after shuffling f₂ (small drop)
    │ ████  ▓▓▓▓  ░░░░
    └─────────────────────
      Base   f₁     f₂
```

### Python Implementation

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.inspection import permutation_importance

X, y = load_breast_cancer(return_X_y=True)
feature_names = load_breast_cancer().feature_names

rf = RandomForestClassifier(n_estimators=200, random_state=42)
rf.fit(X, y)

# --- MDI (Gini) Importance ---
mdi_importances = rf.feature_importances_
sorted_idx = np.argsort(mdi_importances)[-10:]  # top 10

plt.figure(figsize=(10, 6))
plt.barh(range(10), mdi_importances[sorted_idx])
plt.yticks(range(10), feature_names[sorted_idx])
plt.xlabel('Mean Decrease in Impurity')
plt.title('Top 10 Features — MDI Importance')
plt.tight_layout()
plt.show()

# --- Permutation Importance ---
perm_importance = permutation_importance(
    rf, X, y, n_repeats=30, random_state=42, n_jobs=-1
)
sorted_idx_perm = np.argsort(perm_importance.importances_mean)[-10:]

plt.figure(figsize=(10, 6))
plt.boxplot(
    perm_importance.importances[sorted_idx_perm].T,
    vert=False, labels=feature_names[sorted_idx_perm]
)
plt.xlabel('Decrease in Accuracy')
plt.title('Top 10 Features — Permutation Importance')
plt.tight_layout()
plt.show()
```

### MDI vs Permutation Importance

| Aspect | MDI (Gini) | Permutation |
|--------|------------|-------------|
| **Speed** | Fast (computed during training) | Slow (requires re-evaluation) |
| **Bias** | Biased toward high-cardinality features | Unbiased |
| **Correlated Features** | Splits importance among correlated features | Can detect the set matters |
| **Requires Test Data** | No | Yes (recommended) |
| **Reliability** | Moderate | High |

---

## 8. Comparison: Random Forest vs Single Decision Tree

### Side-by-Side Comparison

```
                    Single Decision Tree          Random Forest
                    ────────────────────          ─────────────
Structure:          One tree                      B trees (ensemble)

                         ┌─┐                    ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐
                         │ │                    │ │ │ │ │ │ │ │ │ │
                        / \                    /\ /\ /\ /\ /\
                       /   \                   ...  ...  ...
                      / \  / \
                     (full tree)

Bias:               Low (if deep)              Low (similar)
Variance:           HIGH                       LOW
Overfitting:        Prone                      Resistant
Interpretability:   Excellent                  Limited
Training Speed:     Fast                       Slower (B× trees)
Prediction Speed:   Fast                       Slower (B× trees)
```

### Empirical Comparison

```python
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import cross_val_score
import numpy as np

X, y = make_classification(
    n_samples=2000, n_features=20, n_informative=10,
    n_redundant=5, random_state=42
)

# Single Decision Tree
dt = DecisionTreeClassifier(random_state=42)
dt_scores = cross_val_score(dt, X, y, cv=10, scoring='accuracy')

# Random Forest
rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf_scores = cross_val_score(rf, X, y, cv=10, scoring='accuracy')

print(f"Decision Tree:  {dt_scores.mean():.4f} ± {dt_scores.std():.4f}")
print(f"Random Forest:  {rf_scores.mean():.4f} ± {rf_scores.std():.4f}")
print(f"\nImprovement:    {(rf_scores.mean() - dt_scores.mean())*100:.2f}%")
print(f"Std Reduction:  {(1 - rf_scores.std()/dt_scores.std())*100:.1f}%")
```

### Decision Boundary Visualization

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_moons

X, y = make_moons(n_samples=300, noise=0.3, random_state=42)

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# Create mesh grid
x_min, x_max = X[:, 0].min() - 0.5, X[:, 0].max() + 0.5
y_min, y_max = X[:, 1].min() - 0.5, X[:, 1].max() + 0.5
xx, yy = np.meshgrid(np.linspace(x_min, x_max, 200),
                      np.linspace(y_min, y_max, 200))
grid = np.c_[xx.ravel(), yy.ravel()]

models = [
    ("Single Tree (depth=None)", DecisionTreeClassifier(random_state=42)),
    ("Random Forest (10 trees)", RandomForestClassifier(n_estimators=10, random_state=42)),
    ("Random Forest (100 trees)", RandomForestClassifier(n_estimators=100, random_state=42))
]

for ax, (title, model) in zip(axes, models):
    model.fit(X, y)
    Z = model.predict(grid).reshape(xx.shape)
    ax.contourf(xx, yy, Z, alpha=0.3, cmap='RdBu')
    ax.scatter(X[:, 0], X[:, 1], c=y, cmap='RdBu', edgecolors='k', s=20)
    ax.set_title(f"{title}\nAccuracy: {model.score(X, y):.3f}")
    ax.set_xlim(x_min, x_max)
    ax.set_ylim(y_min, y_max)

plt.tight_layout()
plt.show()
```

---

## 9. Practical Tuning Guide

### Step-by-Step Tuning Strategy

```
Step 1: Start with defaults
  → n_estimators=100, max_features='sqrt', max_depth=None
  → Evaluate with OOB score or cross-validation
  → This is often already competitive!

Step 2: Increase n_estimators
  → Try 200, 500, 1000
  → Monitor OOB score — stop when it plateaus
  → More trees is never harmful (only costs time)

Step 3: Tune max_features
  → Try: 'sqrt', 'log2', 0.3, 0.5, 0.7
  → This is the MOST impactful parameter
  → Lower values → more randomness → lower correlation

Step 4: Optionally constrain tree depth
  → max_depth: 10, 15, 20, 30, None
  → min_samples_leaf: 1, 2, 5, 10
  → Helps with very noisy data or very large trees

Step 5: Handle class imbalance (if needed)
  → class_weight='balanced' or 'balanced_subsample'
```

### Hyperparameter Search with RandomizedSearchCV

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, uniform

param_distributions = {
    'n_estimators': randint(100, 1000),
    'max_features': ['sqrt', 'log2', 0.3, 0.5, 0.7],
    'max_depth': [None, 10, 20, 30, 50],
    'min_samples_split': randint(2, 20),
    'min_samples_leaf': randint(1, 10),
    'bootstrap': [True],
    'class_weight': [None, 'balanced', 'balanced_subsample']
}

rf = RandomForestClassifier(random_state=42)

search = RandomizedSearchCV(
    rf,
    param_distributions,
    n_iter=100,           # number of random combinations
    cv=5,
    scoring='accuracy',
    random_state=42,
    n_jobs=-1,
    verbose=1
)
search.fit(X_train, y_train)

print(f"Best Score: {search.best_score_:.4f}")
print(f"Best Params: {search.best_params_}")

# Evaluate on test set
best_rf = search.best_estimator_
print(f"Test Score: {best_rf.score(X_test, y_test):.4f}")
```

### Common Pitfalls & Solutions

| Pitfall | Symptom | Solution |
|---------|---------|----------|
| Too few trees | High variance in predictions | Increase n_estimators to 500+ |
| max_features too high | Trees too similar, less diversity | Reduce max_features (try 'sqrt') |
| Trees too deep | Memory issues, slow training | Set max_depth=20-30 |
| Class imbalance | Poor recall on minority class | Use class_weight='balanced' |
| High-cardinality categoricals | Biased feature importance | Use permutation importance |
| Correlated features | Feature importance misleading | Use permutation importance |

---

## 10. Complete Python Example

### End-to-End Random Forest Pipeline

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split, learning_curve
from sklearn.metrics import (
    accuracy_score, classification_report,
    confusion_matrix, ConfusionMatrixDisplay
)
from sklearn.datasets import load_wine
from sklearn.preprocessing import StandardScaler
from sklearn.inspection import permutation_importance

# ============================================================
# 1. Load and Explore Data
# ============================================================
data = load_wine()
X, y = data.data, data.target
feature_names = data.feature_names
target_names = data.target_names

print(f"Dataset: {X.shape[0]} samples, {X.shape[1]} features")
print(f"Classes: {target_names}")
print(f"Class distribution: {np.bincount(y)}")

# ============================================================
# 2. Train/Test Split (no scaling needed for RF)
# ============================================================
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, stratify=y, random_state=42
)

# ============================================================
# 3. Train Random Forest
# ============================================================
rf = RandomForestClassifier(
    n_estimators=500,
    max_features='sqrt',
    max_depth=None,
    min_samples_leaf=1,
    oob_score=True,
    random_state=42,
    n_jobs=-1
)
rf.fit(X_train, y_train)

# ============================================================
# 4. Evaluate
# ============================================================
y_pred = rf.predict(X_test)

print(f"\n{'='*50}")
print(f"RESULTS")
print(f"{'='*50}")
print(f"Training Accuracy: {rf.score(X_train, y_train):.4f}")
print(f"Test Accuracy:     {accuracy_score(y_test, y_pred):.4f}")
print(f"OOB Score:         {rf.oob_score_:.4f}")
print(f"\nClassification Report:")
print(classification_report(y_test, y_pred, target_names=target_names))

# ============================================================
# 5. Feature Importance Analysis
# ============================================================
# MDI Importance
importances = rf.feature_importances_
std = np.std([tree.feature_importances_ for tree in rf.estimators_], axis=0)
sorted_idx = np.argsort(importances)

fig, axes = plt.subplots(1, 2, figsize=(16, 6))

# MDI
axes[0].barh(range(len(sorted_idx)), importances[sorted_idx], xerr=std[sorted_idx])
axes[0].set_yticks(range(len(sorted_idx)))
axes[0].set_yticklabels(np.array(feature_names)[sorted_idx])
axes[0].set_title('MDI (Gini) Importance')

# Permutation Importance
perm_imp = permutation_importance(rf, X_test, y_test, n_repeats=30, random_state=42)
sorted_idx_p = np.argsort(perm_imp.importances_mean)
axes[1].boxplot(
    perm_imp.importances[sorted_idx_p].T, vert=False,
    labels=np.array(feature_names)[sorted_idx_p]
)
axes[1].set_title('Permutation Importance')

plt.tight_layout()
plt.show()

# ============================================================
# 6. Learning Curve
# ============================================================
train_sizes, train_scores, val_scores = learning_curve(
    rf, X_train, y_train, cv=5,
    train_sizes=np.linspace(0.1, 1.0, 10),
    scoring='accuracy', n_jobs=-1
)

plt.figure(figsize=(10, 6))
plt.plot(train_sizes, train_scores.mean(axis=1), 'b-o', label='Training')
plt.plot(train_sizes, val_scores.mean(axis=1), 'r-s', label='Validation')
plt.fill_between(train_sizes,
                 train_scores.mean(axis=1) - train_scores.std(axis=1),
                 train_scores.mean(axis=1) + train_scores.std(axis=1), alpha=0.1)
plt.fill_between(train_sizes,
                 val_scores.mean(axis=1) - val_scores.std(axis=1),
                 val_scores.mean(axis=1) + val_scores.std(axis=1), alpha=0.1)
plt.xlabel('Training Set Size')
plt.ylabel('Accuracy')
plt.title('Learning Curve — Random Forest')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

---

## 11. Applications

| Domain | Application | Notes |
|--------|-------------|-------|
| **Bioinformatics** | Gene expression classification | Handles thousands of features well |
| **Finance** | Fraud detection, credit scoring | Robust, handles mixed feature types |
| **Healthcare** | Disease prediction, drug response | Feature importance aids interpretability |
| **Remote Sensing** | Land cover classification | Scales to large satellite datasets |
| **NLP** | Text classification (with TF-IDF features) | Competitive on many text tasks |
| **Computer Vision** | Object detection (pre-deep learning) | Microsoft Kinect used RF for pose estimation |
| **Ecology** | Species distribution, habitat modeling | Handles missing data, mixed features |

---

## 12. Summary Table

| Aspect | Details |
|--------|---------|
| **Inventor** | Leo Breiman (2001) |
| **Core Idea** | Bagging + random feature subsets at each split |
| **Key Innovation** | Feature subsampling at each node → decorrelates trees |
| **max_features (Classification)** | √p (square root of feature count) |
| **max_features (Regression)** | p/3 (one-third of feature count) |
| **Default Tree Count** | 100 (sklearn), often 500+ in practice |
| **Tree Type** | Fully grown, unpruned |
| **Aggregation** | Majority vote (clf) / Average (reg) |
| **Bias** | Low (similar to individual deep tree) |
| **Variance** | Much lower than individual tree |
| **Feature Importance** | MDI (built-in) and Permutation (preferred) |
| **OOB Score** | Free validation estimate |
| **Scaling Required** | No — tree-based, invariant to monotonic transforms |
| **Handles Missing Values** | Not natively in sklearn (but possible in some implementations) |
| **Parallelizable** | Yes — trees are independent |

---

## 13. Revision Questions

**Q1.** Explain the two sources of randomness in a Random Forest. How does each contribute to model diversity?

<details>
<summary>Answer</summary>

**Source 1: Bootstrap sampling** — Each tree is trained on a different bootstrap sample (sampling with replacement). This means different trees see different subsets of the training data, leading to different tree structures.

**Source 2: Feature subsampling** — At each node split, only a random subset of m features (out of p total) is considered. This forces trees to use different features for splitting, even when a single dominant feature exists.

Bootstrap sampling creates diversity through **data variation**, while feature subsampling creates diversity through **feature variation**. Together, they reduce the correlation (ρ) between trees, which directly reduces the ensemble variance: Var = ρσ² + (1-ρ)σ²/B.
</details>

**Q2.** Why is `max_features=√p` recommended for classification and `max_features=p/3` for regression? What happens at the extremes?

<details>
<summary>Answer</summary>

These values represent an empirically validated trade-off between individual tree **strength** (accuracy) and inter-tree **correlation**:

- **√p for classification**: Classification tasks often have sharp decision boundaries. Using √p provides enough features for each tree to make reasonable splits while maintaining diversity.
- **p/3 for regression**: Regression tasks benefit from slightly more features per split because the continuous output requires finer-grained splits.

**At extremes:**
- **m = p**: This is just bagging — no feature randomization. Trees are highly correlated. Minimal variance reduction beyond bagging.
- **m = 1**: Extremely randomized trees. Each split uses a random feature. Trees are very decorrelated (great for variance), but individual trees are weak (high bias), potentially hurting overall accuracy.
</details>

**Q3.** A Random Forest with 500 trees achieves an OOB score of 0.92 and a test score of 0.91. A colleague suggests using 5-fold CV instead. Is that necessary? Justify your answer.

<details>
<summary>Answer</summary>

No, 5-fold CV is **not necessary** in this case. The OOB score (0.92) is very close to the test score (0.91), confirming that OOB is providing a reliable estimate of generalization performance.

**Reasons OOB is sufficient:**
1. The OOB estimate uses ~36.8% of the data per prediction, which is similar to a ~3-fold CV
2. With 500 trees, each sample is in the OOB set for ~184 trees — enough for a stable estimate
3. OOB is essentially "free" — no additional model training needed
4. The close match between OOB and test scores validates the OOB estimate

Using 5-fold CV would require training 5 × 500 = 2,500 trees instead of 500, a 5× increase in computation for minimal additional information.
</details>

**Q4.** You have a dataset with 1000 features, 50 of which are informative and 950 are noise. Would Random Forest perform well? What max_features would you recommend?

<details>
<summary>Answer</summary>

Random Forest can handle this, but requires careful tuning:

With default `max_features=√1000 ≈ 32`, at each split the model considers 32 random features. The probability of including at least one informative feature: P(≥1 informative) = 1 - C(950,32)/C(1000,32) ≈ 1 - (950/1000)^32 ≈ 0.81. This means ~19% of splits might use only noise features.

**Recommendations:**
1. Try `max_features` values: 50, 100, 200 (higher than √p to ensure informative features are considered)
2. Use a larger number of trees (500+) to compensate for weaker individual trees
3. Consider feature selection before RF to remove obvious noise features
4. Use permutation importance after training to identify and potentially remove irrelevant features
</details>

**Q5.** Explain why Random Forest doesn't require feature scaling, while algorithms like SVM and k-NN do. What implications does this have for practical use?

<details>
<summary>Answer</summary>

Random Forest makes decisions based on **thresholds within individual features** — it asks "is feature_j ≤ threshold?" The answer doesn't change if you multiply the feature by a constant. Decision trees (and therefore RFs) are **invariant to monotonic transformations** of individual features.

In contrast, SVM and k-NN use **distance metrics** (e.g., Euclidean distance) that compare magnitudes across features. If one feature ranges [0, 1000] and another [0, 1], the first dominates the distance calculation unless scaled.

**Practical implications:**
- RF can directly handle features with different units (age in years, income in dollars, height in cm)
- No need for StandardScaler or MinMaxScaler preprocessing
- Simplifies the pipeline and reduces data leakage risks from fitting scalers
- Makes RF easier to use for beginners and rapid prototyping
</details>

**Q6.** Design a feature importance experiment that demonstrates the limitations of MDI (Gini) importance. Describe the setup, expected results, and when you should use permutation importance instead.

<details>
<summary>Answer</summary>

**Experiment:**
1. Create a dataset with 3 features: f₁ (informative, continuous), f₂ (informative copy of f₁, i.e., correlated), f₃ (uninformative but high-cardinality, e.g., random IDs 1-1000)

2. Train a Random Forest and compute both MDI and permutation importance.

**Expected Results:**
- **MDI Importance:** f₃ (random IDs) will appear highly important despite being useless, because high-cardinality features create many possible split points, making it easy to find apparently good splits. Also, f₁ and f₂ will split importance roughly equally, underestimating each one's true importance.
- **Permutation Importance:** f₃ will correctly show near-zero importance (shuffling it doesn't affect predictions). f₁ and f₂ will both show moderate importance (shuffling either one still leaves the correlated copy intact).

**When to use Permutation Importance:**
- When you have high-cardinality features (IDs, zip codes, dates)
- When features are correlated
- When you need an unbiased assessment of feature relevance
- When you have a proper test set to evaluate on
</details>

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← 8.1 Bagging](./01-bagging.md) | [Unit 8: Ensemble Methods](../README.md) | [8.3 Boosting Concepts →](./03-boosting-concepts.md) |

---

*© 2025 AIML Study Notes. Unit 8: Ensemble Methods — Chapter 2: Random Forests.*
