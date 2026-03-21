# 📘 7.5 — Advantages & Limitations of Decision Trees

> **Unit 7 · Decision Trees** — Study Notes  
> *Part 5 of 7 — When to use (and when not to use) decision trees, and how they compare with other ML algorithms.*

---

## 📑 Table of Contents

1. [Overview](#1-overview)
2. [Advantages of Decision Trees](#2-advantages-of-decision-trees)
3. [Limitations of Decision Trees](#3-limitations-of-decision-trees)
4. [When to Use Decision Trees](#4-when-to-use-decision-trees)
5. [When NOT to Use Decision Trees](#5-when-not-to-use-decision-trees)
6. [Comparison with Other Algorithms](#6-comparison-with-other-algorithms)
7. [Mitigating Limitations — Quick Reference](#7-mitigating-limitations--quick-reference)
8. [Worked Example — Instability Demonstration](#8-worked-example--instability-demonstration)
9. [Python — Demonstrating Strengths & Weaknesses](#9-python--demonstrating-strengths--weaknesses)
10. [Applications](#10-applications)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)
13. [Navigation](#13-navigation)

---

## 1. Overview

Decision trees are one of the most **popular and intuitive** algorithms in machine learning. However, like every algorithm, they come with trade-offs. Understanding these trade-offs is crucial for knowing **when** a decision tree is the right tool and when you should consider alternatives.

```
┌─────────────────────────────────────────────────────────┐
│           DECISION TREE TRADE-OFF SPECTRUM               │
│                                                          │
│  Interpretability  ████████████████████████████  HIGH     │
│  Speed (predict)   ████████████████████████████  HIGH     │
│  No preprocessing  ████████████████████████████  HIGH     │
│  Handles mixed     ████████████████████████████  HIGH     │
│  Nonlinear capture ██████████████████            MED      │
│  Accuracy (solo)   ████████████                  MED-LOW  │
│  Stability         ████████                      LOW      │
│  Smooth boundaries ████                          LOW      │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Advantages of Decision Trees

### 2.1 Interpretability / Explainability

Decision trees are **white-box models** — you can follow the path from root to leaf and understand *exactly* why a prediction was made.

```
"Why was this loan application denied?"

   [Income ≤ 40K?]
        ↓ Yes
   [Debt Ratio > 0.5?]
        ↓ Yes
   [Credit Score ≤ 580?]
        ↓ Yes
   → DENY

Explanation: "Income below 40K, debt ratio above 0.5, and 
              credit score below 580."
```

**Why this matters:**
- Regulatory compliance (GDPR "right to explanation", banking regulations).
- Doctor/patient trust in medical AI.
- Business stakeholders can validate the model's logic.
- Easier to debug than black-box models.

### 2.2 No Feature Scaling Required

Unlike algorithms that depend on distance (k-NN, SVM, neural networks) or magnitude (linear/logistic regression without regularisation), decision trees:

- Do **not** require normalisation or standardisation.
- Work with features on completely **different scales**.

```
Feature 1: Annual income ($)  →  range: [15,000 — 500,000]
Feature 2: Age (years)        →  range: [18 — 85]
Feature 3: Number of children →  range: [0 — 8]

Tree doesn't care about scale — it only compares feature values 
with thresholds using ≤ and >.
```

### 2.3 Handles Mixed Feature Types

Decision trees natively handle:
- **Continuous** features (e.g., temperature = 23.5°C)
- **Categorical** features (e.g., colour = Red/Green/Blue)
- **Ordinal** features (e.g., education = High School < Bachelor's < Master's)
- **Binary** features (e.g., is_student = True/False)

No need for one-hot encoding (though sklearn's implementation does require numeric input, so you still need to encode categoricals as integers).

### 2.4 Feature Importance Built In

Trees naturally rank features by importance — features used for splitting closer to the root are more important. This provides:

- **Automatic feature selection** (unimportant features are never split on).
- **Feature importance scores** (covered in detail in Part 6).
- **Insight into data structure** — which features matter most?

### 2.5 Captures Non-Linear Relationships

Trees can model complex non-linear decision boundaries through recursive partitioning:

```
Linear model (can't capture this):     Decision tree (can capture this):

  X₂ ↑                                X₂ ↑
     │  B B B A A A                       │ B B │ A A A
     │  B B B A A A                       │ B B │ A A A
     │  A A A B B B                       │─────┼──────
     │  A A A B B B                       │ A A │ B B B
     └────────────── X₁ →                 │ A A │ B B B
                                          └───────────── X₁ →
  Can't draw a single line               Axis-aligned splits work!
  to separate A from B
```

### 2.6 Fast Prediction

Prediction time is **O(depth)** — typically O(log n) if the tree is balanced.

| Algorithm | Prediction Time | With 1M training samples |
|-----------|----------------|--------------------------|
| Decision Tree | O(d) ≈ O(log n) | ~20 comparisons |
| k-NN | O(n) | ~1,000,000 distance calculations |
| SVM (kernel) | O(n_sv × d) | Depends on support vectors |
| Neural Network | O(layers × neurons) | Depends on architecture |

### 2.7 Handles Missing Values (Some Implementations)

- **CART** uses **surrogate splits** — if the primary split feature is missing, use a correlated feature.
- **C4.5** distributes samples with missing values probabilistically.
- **sklearn** does NOT handle missing values natively (you must impute first), but other implementations (XGBoost, LightGBM) do.

### 2.8 Requires Minimal Data Preparation

```
                   Tree         Linear Reg.    Neural Net    SVM
Scaling             ❌ No        ✅ Yes          ✅ Yes       ✅ Yes
One-hot encoding    ❌ No*       ✅ Yes          ✅ Yes       ✅ Yes
Impute missing      ❌ No*       ✅ Yes          ✅ Yes       ✅ Yes
Feature engineering ❌ Minimal   ✅ Often        ✅ Often     ✅ Often

* With appropriate implementations (not sklearn default)
```

---

## 3. Limitations of Decision Trees

### 3.1 Overfitting

The **number one weakness**. An unpruned tree will fit the training data perfectly, including noise.

```
Unpruned tree:
  - Memorises every training sample
  - Creates very specific rules that don't generalise
  - "If temperature = 23.47°C AND humidity = 67.2% → Class A"
  
  ↓ Solution: Pruning, max_depth, min_samples_leaf, ensembles
```

### 3.2 Instability (High Variance)

Small changes in the training data can produce a **completely different tree**. This is because:

1. If the top split changes, the entire tree structure changes.
2. The greedy algorithm amplifies small differences.

```
Training set 1:          Training set 2 (1 sample changed):

      [Age ≤ 30?]              [Income ≤ 50K?]
      /         \              /              \
   [Income≤50K] Leaf        [Age ≤ 25?]      Leaf
    /      \                  /      \
  Leaf    Leaf             Leaf     Leaf

  COMPLETELY DIFFERENT TREE from a tiny data change!
```

**Solution:** Ensemble methods (Random Forest averages many unstable trees to get a stable prediction).

### 3.3 Bias Toward High-Cardinality Features

Features with many distinct values (e.g., ID, ZIP code, timestamp) get unfair advantage because:
- More candidate split points → higher chance of finding a good split by luck.
- Information Gain is biased toward such features.

```
Feature "CustomerID" (unique per sample):
  - Can create perfectly pure splits (one sample per leaf)
  - Information Gain = maximum
  - But ZERO predictive value!
  
  ↓ Solution: Use Gain Ratio, or exclude irrelevant features
```

### 3.4 Axis-Aligned Splits Only

Standard decision trees can only split **parallel to feature axes** (using conditions like `X₁ ≤ t`). They cannot capture diagonal or circular boundaries efficiently.

```
  Optimal boundary (diagonal):     Tree's approximation:

  X₂ ↑                            X₂ ↑
     │ · B B B                        │ B│B│B│
     │ A · B B                        │──┼─┼─┤
     │ A A · B                        │A │A│B│
     │ A A A ·                        │──┼─┤ │
     └────────── X₁ →                 │A │A│ │
     Diagonal line                    └──┴─┴─┴── X₁ →
     (not axis-aligned)               Staircase approximation
                                      (many splits needed)
```

**Consequence:** For diagonal boundaries, a tree needs many more splits → deeper tree → more overfitting.

**Solutions:**
- Feature engineering (create a new feature X₁ − X₂)
- Use oblique decision trees (splits on linear combinations of features)
- Use other algorithms (SVM, neural networks)

### 3.5 Not Ideal for Extrapolation (Regression)

Regression trees predict the **mean of training samples** in each leaf. They cannot extrapolate beyond the range of training data.

```
    y ↑
      │         . . .     ← True relationship (linear, continues up)
      │       .
      │     .
      │   .
      │ . ─ ─ ─ ─ ─ ─   ← Tree prediction (flat line beyond training range)
      │
      └──────────────── x →
            training     test (extrapolation)
            range        region
```

### 3.6 Piecewise Constant Predictions (Regression)

Regression trees produce **step functions**, not smooth curves:

```
    y ↑
      │     ┌───┐
      │ ┌───┘   │         ← Tree prediction (steps)
      │ │       └───┐
      │─┘           └──   
      │   ~~~~~~~~~~~~    ← True function (smooth)
      └──────────────── x →
```

### 3.7 Computationally Expensive to Optimise Globally

Finding the **globally optimal** decision tree is NP-complete. The greedy approach finds a good (but not necessarily optimal) tree.

---

## 4. When to Use Decision Trees

| Scenario | Why Trees Work Well |
|----------|---------------------|
| **Explainability is critical** | Regulated industries, medical, legal |
| **Quick baseline model** | Fast to train, no preprocessing needed |
| **Mixed feature types** | Handles numeric + categorical together |
| **Feature importance analysis** | Understand which features matter |
| **Non-linear relationships** | Captures interactions automatically |
| **As base learners in ensembles** | Random Forest, XGBoost, LightGBM |
| **Tabular data** | Trees (especially ensembles) dominate tabular data benchmarks |
| **Small datasets** | Simple enough to not overfit with proper pruning |

---

## 5. When NOT to Use Decision Trees

| Scenario | Why Trees Struggle | Better Alternative |
|----------|--------------------|--------------------|
| **Linear relationships** | Tree needs many splits to approximate a line | Linear / Logistic Regression |
| **Image recognition** | Pixel-level features, spatial structure | CNNs |
| **Text / NLP** | Sequential structure, high dimensionality | RNNs, Transformers |
| **Very high accuracy needed** | Single tree limited | Ensemble methods (RF, GBM) |
| **Smooth regression** | Piecewise constant approximation | Linear regression, splines, neural nets |
| **Time series** | No temporal awareness built-in | ARIMA, LSTM |
| **Extrapolation** | Can't predict beyond training range | Linear models |

---

## 6. Comparison with Other Algorithms

### 6.1 Detailed Comparison Matrix

| Criterion | Decision Tree | Logistic Regression | k-NN | SVM | Random Forest | Neural Network |
|-----------|:---:|:---:|:---:|:---:|:---:|:---:|
| **Interpretability** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐ |
| **Handles non-linearity** | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **No preprocessing** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **Handles mixed types** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **Training speed** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **Prediction speed** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Accuracy (single)** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Stability** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Overfitting risk** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |

(⭐ = worst, ⭐⭐⭐⭐⭐ = best; for overfitting risk, more stars = higher risk)

### 6.2 Bias-Variance Decomposition

```
             High Variance ────────────────────── Low Variance
                  │                                     │
                  │  Decision Tree (unpruned)            │
High Bias         │         │                            │  Linear Regression
    │             │         ▼                            │       │
    │             │  Decision Tree (pruned)               │       │
    │             │              │                        │       │
    │             │              ▼                        │       │
    │             │       Random Forest                   │       │
    │             │       (averages away variance)        │       │
Low Bias          │                                      │       │
                  │                                      │       │
                  └──────────────────────────────────────┘
```

| Model | Bias | Variance |
|-------|------|----------|
| Decision tree (deep) | Low | **Very high** |
| Decision tree (shallow) | Moderate | Moderate |
| Linear regression | **High** (if relationship is non-linear) | Low |
| Random Forest | Low | Low (variance reduced by averaging) |
| Neural network (large) | Low | Can be high |

---

## 7. Mitigating Limitations — Quick Reference

| Limitation | Mitigation |
|-----------|------------|
| Overfitting | Pruning (pre or post), max_depth, min_samples_leaf, cross-validation |
| Instability | Ensemble methods (Random Forest, Bagging) |
| High-cardinality bias | Use Gain Ratio, or exclude ID-like features |
| Axis-aligned splits | Feature engineering, oblique trees |
| Can't extrapolate | Use linear models for extrapolation tasks |
| Piecewise constant | Use more leaves, or switch to regression models |
| Globally suboptimal | Accept (greedy is practical), or use ensembles |

---

## 8. Worked Example — Instability Demonstration

### Dataset: 6 samples

| X | Y | Class |
|---|---|-------|
| 1 | 5 | A |
| 2 | 4 | A |
| 3 | 3 | B |
| 4 | 2 | B |
| 5 | 1 | A |
| 6 | 6 | B |

### Tree on Full Dataset

```
Best split: X ≤ 2
      [X ≤ 2?]
      /      \
    Yes       No
    /           \
  A (2/2)    [Y ≤ 3?]
              /     \
            Yes      No
            /          \
          B (2/2)    [X ≤ 5?]
                      /     \
                    Yes      No
                    /          \
                  A (1/1)    B (1/1)
```

### Tree After Removing Sample 5 (X=5, Y=1, A)

New dataset: just 5 samples.

```
Best split: Y ≤ 3?
      [Y ≤ 3?]
      /      \
    Yes       No
    /           \
  B (2/2)    [X ≤ 2?]
              /     \
            Yes      No
            /          \
          A (2/2)    B (1/1)
```

**Completely different tree!** Removing just one sample changed the root split from `X ≤ 2` to `Y ≤ 3`, and the entire tree structure changed.

This is the **instability** problem in action.

---

## 9. Python — Demonstrating Strengths & Weaknesses

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.tree import DecisionTreeClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import make_moons, make_classification
from sklearn.model_selection import cross_val_score

# ──────────────────────────────────────────────────────
#  1. STRENGTH: Non-linear decision boundaries
# ──────────────────────────────────────────────────────
X, y = make_moons(n_samples=200, noise=0.25, random_state=42)

tree = DecisionTreeClassifier(max_depth=5, random_state=42)
lr = LogisticRegression(random_state=42)

tree.fit(X, y)
lr.fit(X, y)

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

for ax, model, name in zip(axes, [tree, lr], ['Decision Tree', 'Logistic Reg.']):
    xx, yy = np.meshgrid(np.linspace(X[:, 0].min()-0.5, X[:, 0].max()+0.5, 200),
                          np.linspace(X[:, 1].min()-0.5, X[:, 1].max()+0.5, 200))
    Z = model.predict(np.c_[xx.ravel(), yy.ravel()]).reshape(xx.shape)
    ax.contourf(xx, yy, Z, alpha=0.3, cmap='RdBu')
    ax.scatter(X[:, 0], X[:, 1], c=y, cmap='RdBu', edgecolor='k', s=30)
    score = cross_val_score(model, X, y, cv=5).mean()
    ax.set_title(f"{name}\nCV Accuracy: {score:.3f}")

plt.suptitle("Strength: Trees capture non-linear boundaries", fontsize=14)
plt.tight_layout()
plt.savefig("tree_nonlinear.png", dpi=150)
plt.show()

# ──────────────────────────────────────────────────────
#  2. WEAKNESS: Instability (variance)
# ──────────────────────────────────────────────────────
print("\n=== INSTABILITY DEMO ===")
X_full, y_full = make_classification(n_samples=100, n_features=4,
                                     random_state=42)

trees = []
for i in range(5):
    # Bootstrap sample (slightly different training sets)
    rng = np.random.RandomState(i)
    idx = rng.choice(len(X_full), size=len(X_full), replace=True)
    X_boot, y_boot = X_full[idx], y_full[idx]
    
    clf = DecisionTreeClassifier(random_state=42)
    clf.fit(X_boot, y_boot)
    trees.append(clf)
    print(f"  Bootstrap {i}: depth={clf.get_depth()}, "
          f"leaves={clf.get_n_leaves()}, "
          f"first split feature={clf.tree_.feature[0]}")

print("  → Different bootstrap samples → different trees!")

# ──────────────────────────────────────────────────────
#  3. WEAKNESS: Axis-aligned splits (diagonal boundary)
# ──────────────────────────────────────────────────────
print("\n=== AXIS-ALIGNED LIMITATION ===")

# Create data with a diagonal boundary: y = x
np.random.seed(42)
X_diag = np.random.randn(200, 2)
y_diag = (X_diag[:, 0] + X_diag[:, 1] > 0).astype(int)

for depth in [2, 5, 10]:
    clf = DecisionTreeClassifier(max_depth=depth, random_state=42)
    score = cross_val_score(clf, X_diag, y_diag, cv=5).mean()
    print(f"  Tree depth={depth:2d} → CV accuracy: {score:.3f}")

lr = LogisticRegression(random_state=42)
score_lr = cross_val_score(lr, X_diag, y_diag, cv=5).mean()
print(f"  Logistic Regression → CV accuracy: {score_lr:.3f}")
print("  → Linear model perfectly captures the diagonal boundary!")

# ──────────────────────────────────────────────────────
#  4. STRENGTH: No feature scaling needed
# ──────────────────────────────────────────────────────
print("\n=== NO SCALING NEEDED ===")
X_scaled = X_full.copy()
X_scaled[:, 0] *= 1000  # scale feature 0 to thousands
X_scaled[:, 1] *= 0.001  # scale feature 1 to thousandths

tree_orig = cross_val_score(
    DecisionTreeClassifier(random_state=42), X_full, y_full, cv=5
).mean()
tree_scaled = cross_val_score(
    DecisionTreeClassifier(random_state=42), X_scaled, y_full, cv=5
).mean()
lr_orig = cross_val_score(
    LogisticRegression(max_iter=1000, random_state=42), X_full, y_full, cv=5
).mean()
lr_scaled = cross_val_score(
    LogisticRegression(max_iter=1000, random_state=42), X_scaled, y_full, cv=5
).mean()

print(f"  Tree (original scale):  {tree_orig:.3f}")
print(f"  Tree (wildly scaled):   {tree_scaled:.3f}  ← same!")
print(f"  LogReg (original):      {lr_orig:.3f}")
print(f"  LogReg (wildly scaled): {lr_scaled:.3f}  ← may differ!")
```

---

## 10. Applications

| Use Case | Decision Tree's Role |
|----------|---------------------|
| **Medical triage** | Advantage: interpretable flowcharts for emergency room routing |
| **Credit scoring** | Advantage: regulatory compliance requires explainability |
| **Kaggle competitions** | Limitation overcome: used in ensembles (XGBoost, LightGBM) |
| **Customer segmentation** | Advantage: identifies interpretable customer groups |
| **Fraud detection** | Limitation: unstable alone; used in Random Forest for stability |
| **Recommendation systems** | Limitation: neural collaborative filtering outperforms trees |

---

## 11. Summary Table

| Category | Point | Rating |
|----------|-------|--------|
| **Advantage** | Interpretability / white-box | ⭐⭐⭐⭐⭐ |
| **Advantage** | No feature scaling needed | ⭐⭐⭐⭐⭐ |
| **Advantage** | Handles mixed feature types | ⭐⭐⭐⭐⭐ |
| **Advantage** | Built-in feature importance | ⭐⭐⭐⭐ |
| **Advantage** | Captures non-linear relationships | ⭐⭐⭐⭐ |
| **Advantage** | Fast prediction (O(depth)) | ⭐⭐⭐⭐⭐ |
| **Advantage** | Minimal data preparation | ⭐⭐⭐⭐⭐ |
| **Limitation** | Prone to overfitting | ⚠️⚠️⚠️⚠️⚠️ |
| **Limitation** | Instability (high variance) | ⚠️⚠️⚠️⚠️ |
| **Limitation** | Biased toward high-cardinality features | ⚠️⚠️⚠️ |
| **Limitation** | Axis-aligned splits only | ⚠️⚠️⚠️ |
| **Limitation** | Can't extrapolate (regression) | ⚠️⚠️⚠️ |
| **Limitation** | Piecewise constant predictions | ⚠️⚠️ |
| **Limitation** | Greedy (globally suboptimal) | ⚠️⚠️ |

---

## 12. Revision Questions

1. **List** five advantages of decision trees over other ML algorithms. For each, give a concrete scenario where this advantage matters.

2. **Explain** the instability problem of decision trees. Why are small changes in data amplified into large changes in tree structure?

3. What is the **axis-aligned split limitation**? Draw a dataset where a tree would struggle but a linear model would succeed.

4. **Compare** decision trees with logistic regression across these dimensions: interpretability, handling non-linearity, feature scaling requirements, and stability.

5. Your manager says "just use a deeper tree to get better accuracy." **Explain** why this advice is dangerous and what you would recommend instead.

6. **Describe** three ways to mitigate the overfitting problem in decision trees.

---

## 13. Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← 7.4 Pruning Techniques](04-pruning-techniques.md) | **7.5 Advantages & Limitations** | [7.6 Feature Importance →](06-feature-importance.md) |

| Module | Link |
|--------|------|
| 7.1 | [Tree Structure & Terminology](01-tree-structure-terminology.md) |
| 7.2 | [Splitting Criteria](02-splitting-criteria.md) |
| 7.3 | [Building a Decision Tree](03-building-decision-tree.md) |
| 7.4 | [Pruning Techniques](04-pruning-techniques.md) |
| 7.5 | **Advantages & Limitations** (you are here) |
| 7.6 | [Feature Importance](06-feature-importance.md) |
| 7.7 | [Decision Trees for Regression](07-decision-trees-regression.md) |

---

> *"A decision tree is a Swiss Army knife — useful in many situations, but you wouldn't use it to perform surgery."*
