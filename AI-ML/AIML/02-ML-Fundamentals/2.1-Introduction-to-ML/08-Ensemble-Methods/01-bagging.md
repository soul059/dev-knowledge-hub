# Chapter 8.1: Bagging (Bootstrap Aggregating)

> **Unit 8: Ensemble Methods** — File 1 of 8

---

## Table of Contents

1. [Introduction & Motivation](#1-introduction--motivation)
2. [Bootstrap Sampling](#2-bootstrap-sampling)
3. [The Bagging Algorithm](#3-the-bagging-algorithm)
4. [Why Bagging Reduces Variance](#4-why-bagging-reduces-variance)
5. [Mathematical Analysis of Variance Reduction](#5-mathematical-analysis-of-variance-reduction)
6. [Out-of-Bag (OOB) Evaluation](#6-out-of-bag-oob-evaluation)
7. [Feature Randomization](#7-feature-randomization)
8. [Step-by-Step Worked Example](#8-step-by-step-worked-example)
9. [BaggingClassifier & BaggingRegressor in Scikit-Learn](#9-baggingclassifier--baggingregressor-in-scikit-learn)
10. [Applications](#10-applications)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)

---

## 1. Introduction & Motivation

A single decision tree is powerful but **unstable** — small changes in training data can produce wildly different trees. This instability means **high variance**.

**Key Insight:** If we could somehow average out multiple unstable models, the averaged prediction would be far more stable and accurate than any single model.

```
Single Model Problem:
┌─────────────────────────────────────────────────────┐
│                                                     │
│   Training Data ──► Model ──► Prediction            │
│        │                         │                  │
│   (small change)            (large change)          │
│        │                         │                  │
│   Training Data' ──► Model' ──► Prediction'         │
│                                                     │
│   HIGH VARIANCE = UNSTABLE PREDICTIONS              │
└─────────────────────────────────────────────────────┘
```

**Leo Breiman** introduced Bagging in 1996 to address exactly this problem. The name stands for **B**ootstrap **AGG**regat**ING**.

### The Core Philosophy

> "Instead of building one model and hoping it generalizes well, build many models on slightly different data and average their predictions."

---

## 2. Bootstrap Sampling

### What Is a Bootstrap Sample?

A **bootstrap sample** is a dataset of size `n` drawn **with replacement** from the original dataset of size `n`.

```
Original Dataset D (n = 6):
┌─────┬─────┬─────┬─────┬─────┬─────┐
│ x₁  │ x₂  │ x₃  │ x₄  │ x₅  │ x₆  │
└─────┴─────┴─────┴─────┴─────┴─────┘

Bootstrap Sample D*₁ (draw 6 with replacement):
┌─────┬─────┬─────┬─────┬─────┬─────┐
│ x₂  │ x₂  │ x₅  │ x₁  │ x₅  │ x₃  │  ← x₂ and x₅ appear twice
└─────┴─────┴─────┴─────┴─────┴─────┘     x₄ and x₆ are missing

Bootstrap Sample D*₂ (draw 6 with replacement):
┌─────┬─────┬─────┬─────┬─────┬─────┐
│ x₁  │ x₃  │ x₃  │ x₆  │ x₄  │ x₁  │  ← x₁ and x₃ appear twice
└─────┴─────┴─────┴─────┴─────┴─────┘     x₂ and x₅ are missing
```

### The "63.2% Rule"

The probability that a specific sample `xᵢ` is **NOT** picked in any single draw is:

```
P(not picked in one draw) = (1 - 1/n)
```

Over `n` draws (one full bootstrap sample):

```
P(xᵢ never picked) = (1 - 1/n)ⁿ
```

As `n → ∞`:

```
lim   (1 - 1/n)ⁿ = e⁻¹ ≈ 0.368
n→∞
```

Therefore, each bootstrap sample contains approximately **63.2%** of unique original samples, and about **36.8%** are left out.

### Numerical Verification

| n     | (1 - 1/n)ⁿ | % Unique Samples |
|-------|-------------|-------------------|
| 10    | 0.3487      | 65.13%            |
| 50    | 0.3642      | 63.58%            |
| 100   | 0.3660      | 63.40%            |
| 1000  | 0.3677      | 63.23%            |
| ∞     | 0.3679      | 63.21%            |

---

## 3. The Bagging Algorithm

### Algorithm: Bagging

```
INPUT:  Training set D = {(x₁,y₁), ..., (xₙ,yₙ)}
        Base learning algorithm L (e.g., Decision Tree)
        Number of base learners B

PROCEDURE:
    FOR b = 1, 2, ..., B:
        1. Draw a bootstrap sample D*_b of size n from D (with replacement)
        2. Train base learner h_b = L(D*_b)
    END FOR

OUTPUT (Regression):
    ĥ(x) = (1/B) Σ h_b(x)           ← Average of predictions
            b=1

OUTPUT (Classification):
    ĥ(x) = argmax  Σ I(h_b(x) = c)  ← Majority vote
              c   b=1
```

### Visual Representation

```
                    ┌──────────────────┐
                    │  Original Data D │
                    │  (n samples)     │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │ Bootstrap   │ │ Bootstrap   │ │ Bootstrap   │
    │ Sample D*₁  │ │ Sample D*₂  │ │ Sample D*_B │
    │ (n samples) │ │ (n samples) │ │ (n samples) │
    └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
           │               │               │
           ▼               ▼               ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │   Model h₁  │ │   Model h₂  │ │   Model h_B │
    │ (full tree) │ │ (full tree) │ │ (full tree) │
    └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
           │               │               │
           ▼               ▼               ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │  Pred: ŷ₁   │ │  Pred: ŷ₂   │ │  Pred: ŷ_B  │
    └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
           │               │               │
           └───────────────┼───────────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │   AGGREGATE       │
                 │ Regression: Mean  │
                 │ Classify: Vote    │
                 └─────────┬─────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ Final ŷ     │
                    └─────────────┘
```

---

## 4. Why Bagging Reduces Variance

### The Bias-Variance Decomposition

For any model, the expected prediction error can be decomposed:

```
E[(y - ĥ(x))²] = Bias²(ĥ(x)) + Var(ĥ(x)) + σ²(irreducible noise)
```

**Bagging primarily targets the variance term.**

### Intuition: Averaging Reduces Variance

Consider `B` independent random variables Z₁, Z₂, ..., Z_B, each with variance σ²:

```
Var(Z̄) = Var((1/B) Σ Zᵢ) = σ²/B
```

If we could get B **independent** models, averaging them would reduce variance by a factor of B. But bootstrap samples are **not independent** — they share data. Still, the correlation is much less than 1, so substantial variance reduction occurs.

### Why Bias Stays (Approximately) the Same

Each bootstrap model is trained on a dataset with the same distribution as the original. Therefore:

```
E[h_b(x)] ≈ E[h(x)]    (same expected prediction)

Bias(ĥ_bagged) = E[ĥ_bagged(x)] - f(x)
               = E[(1/B) Σ h_b(x)] - f(x)
               = (1/B) Σ E[h_b(x)] - f(x)
               ≈ E[h(x)] - f(x)
               = Bias(h(x))
```

The bias of the bagged ensemble is approximately the same as the bias of any individual model.

---

## 5. Mathematical Analysis of Variance Reduction

### Variance of Correlated Predictors

Suppose we have `B` models with:
- Each model has variance σ²
- Pairwise correlation between any two models is ρ

Then the variance of the averaged prediction is:

```
Var(ĥ_bagged) = ρσ² + (1-ρ)/B · σ²
```

**Derivation:**

```
Var(1/B Σ h_b) = 1/B² · Var(Σ h_b)

                = 1/B² · [Σ Var(h_b) + Σ  Σ  Cov(h_b, h_b')]
                                        b≠b'

                = 1/B² · [B·σ² + B(B-1)·ρσ²]

                = σ²/B + (B-1)/B · ρσ²

                = ρσ² + (1-ρ)/B · σ²
```

### Analysis of the Result

```
Var(ĥ_bagged) = ρσ² + (1-ρ)σ²/B
                ─────   ──────────
               always     vanishes
               present    as B→∞
```

| Scenario | ρ | Variance | Interpretation |
|----------|---|----------|----------------|
| Perfect correlation | 1.0 | σ² | No reduction (identical models) |
| Moderate correlation | 0.5 | 0.5σ² + 0.5σ²/B | ~50% reduction for large B |
| No correlation | 0.0 | σ²/B | Maximum reduction (ideal) |

**Key Takeaway:** To maximize variance reduction, we need to **minimize the correlation ρ** between models. This is exactly what **Random Forests** do (see next chapter).

### Effect of Number of Base Learners

```
Variance vs. Number of Base Learners (ρ = 0.4, σ² = 1.0)

Var
1.0 │ *
    │
0.8 │   *
    │
0.6 │      *    *
    │               *     *     *     *     *     *
0.4 │─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  ← floor = ρσ²
    │
0.2 │
    │
0.0 └──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──────────── B
       1  2  5 10 20 30 50 75 100 200

Note: Variance decreases rapidly at first, then plateaus at ρσ²
      Adding more learners beyond ~50-100 gives diminishing returns
```

---

## 6. Out-of-Bag (OOB) Evaluation

### The Free Validation Set

Since each bootstrap sample leaves out ~36.8% of the original data, those left-out samples serve as a **free validation set** for that particular model.

```
Bootstrap Sample 1:  [x₁ x₃ x₃ x₅ x₂ x₁]  → OOB₁ = {x₄, x₆}
Bootstrap Sample 2:  [x₂ x₄ x₆ x₂ x₅ x₆]  → OOB₂ = {x₁, x₃}
Bootstrap Sample 3:  [x₁ x₅ x₄ x₅ x₃ x₁]  → OOB₃ = {x₂, x₆}
```

### OOB Prediction Process

For each sample `xᵢ`, collect predictions **only from models that did NOT train on xᵢ**:

```
For sample x₄:
  - NOT in Bootstrap 1 → h₁(x₄) = class A
  - In Bootstrap 2     → (skip)
  - NOT in Bootstrap 3 → h₃(x₄) = class B (if another model)
  ...
  OOB prediction for x₄ = majority_vote(h₁, h₃, ...)
```

### OOB Error Estimation

```
OOB Error = (1/n) Σ I(yᵢ ≠ ŷᵢ_OOB)
            i=1

Where ŷᵢ_OOB is the aggregated prediction from all models
that did NOT include xᵢ in their bootstrap sample.
```

**Why OOB is Valuable:**
- No need for a separate validation set or cross-validation
- Uses ~36.8% of models per prediction (sufficient for stable estimates)
- Provides an approximately unbiased estimate of test error
- Computationally cheaper than k-fold cross-validation

---

## 7. Feature Randomization

Bagging can be extended to also randomly sample **features** (columns) at each split or for each base learner.

### Types of Feature Randomization

```
┌──────────────────────────────────────────────────────────┐
│           Feature Randomization Strategies               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  1. Random Subspace Method (per model):                  │
│     Each model sees only a random subset of features     │
│                                                          │
│  2. Random Split Selection (per split):                  │
│     At each node, consider only a random subset          │
│     of features for splitting (→ Random Forest)          │
│                                                          │
│  3. Both: Combine sample + feature randomization         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Why Feature Randomization Helps

Without feature randomization, if one feature is very strong, **all trees will split on it first** → trees are highly correlated → less variance reduction.

```
Without feature randomization (ρ ≈ 0.8):
  Tree 1: Split on Feature A first → ...
  Tree 2: Split on Feature A first → ...
  Tree 3: Split on Feature A first → ...
  (Very correlated trees)

With feature randomization (ρ ≈ 0.3):
  Tree 1: Can't see Feature A → splits on Feature C
  Tree 2: Sees Feature A → splits on Feature A
  Tree 3: Can't see Feature A → splits on Feature D
  (More diverse, less correlated trees)
```

---

## 8. Step-by-Step Worked Example

### Problem Setup

We have a small regression dataset and want to demonstrate bagging with B=3 simple regression trees (depth=1, "stumps").

```
Original Dataset D:
┌───┬────┬────┬────┐
│ i │ x₁ │ x₂ │  y │
├───┼────┼────┼────┤
│ 1 │  1 │  5 │ 10 │
│ 2 │  2 │  3 │ 15 │
│ 3 │  3 │  4 │ 25 │
│ 4 │  4 │  2 │ 30 │
│ 5 │  5 │  1 │ 40 │
└───┴────┴────┴────┘
```

### Step 1: Create Bootstrap Samples

```
Bootstrap Sample D*₁ (indices drawn: 1, 3, 3, 5, 2):
┌───┬────┬────┬────┐
│ i │ x₁ │ x₂ │  y │
├───┼────┼────┼────┤
│ 1 │  1 │  5 │ 10 │
│ 3 │  3 │  4 │ 25 │
│ 3 │  3 │  4 │ 25 │
│ 5 │  5 │  1 │ 40 │
│ 2 │  2 │  3 │ 15 │
└───┴────┴────┴────┘
OOB₁ = {sample 4}

Bootstrap Sample D*₂ (indices drawn: 2, 4, 4, 1, 5):
┌───┬────┬────┬────┐
│ i │ x₁ │ x₂ │  y │
├───┼────┼────┼────┤
│ 2 │  2 │  3 │ 15 │
│ 4 │  4 │  2 │ 30 │
│ 4 │  4 │  2 │ 30 │
│ 1 │  1 │  5 │ 10 │
│ 5 │  5 │  1 │ 40 │
└───┴────┴────┴────┘
OOB₂ = {sample 3}

Bootstrap Sample D*₃ (indices drawn: 5, 1, 2, 2, 3):
┌───┬────┬────┬────┐
│ i │ x₁ │ x₂ │  y │
├───┼────┼────┼────┤
│ 5 │  5 │  1 │ 40 │
│ 1 │  1 │  5 │ 10 │
│ 2 │  2 │  3 │ 15 │
│ 2 │  2 │  3 │ 15 │
│ 3 │  3 │  4 │ 25 │
└───┴────┴────┴────┘
OOB₃ = {sample 4}
```

### Step 2: Train Decision Stumps

**Tree h₁ (trained on D*₁):**
```
Best split: x₁ ≤ 2.5
  Left (x₁ ≤ 2.5):  mean(10, 25, 15) = 16.67
  Right (x₁ > 2.5):  mean(25, 40) = 32.50
```

**Tree h₂ (trained on D*₂):**
```
Best split: x₁ ≤ 3.0
  Left (x₁ ≤ 3.0):  mean(15, 10) = 12.50
  Right (x₁ > 3.0):  mean(30, 30, 40) = 33.33
```

**Tree h₃ (trained on D*₃):**
```
Best split: x₁ ≤ 2.5
  Left (x₁ ≤ 2.5):  mean(10, 15, 15) = 13.33
  Right (x₁ > 2.5):  mean(40, 25) = 32.50
```

### Step 3: Aggregate Predictions

For a new point x_test = (3.5, 3):

```
h₁(x_test): x₁=3.5 > 2.5  → 32.50
h₂(x_test): x₁=3.5 > 3.0  → 33.33
h₃(x_test): x₁=3.5 > 2.5  → 32.50

Bagged prediction = (32.50 + 33.33 + 32.50) / 3 = 32.78
```

### Step 4: OOB Evaluation

```
Sample 4 (x₁=4, y=30):
  OOB for sample 4: models h₁ and h₃ (sample 4 not in D*₁ or D*₃)
  h₁(4, 2): x₁=4 > 2.5 → 32.50
  h₃(4, 2): x₁=4 > 2.5 → 32.50
  OOB prediction = (32.50 + 32.50) / 2 = 32.50
  OOB error for sample 4 = (30 - 32.50)² = 6.25

Sample 3 (x₁=3, y=25):
  OOB for sample 3: model h₂ only (sample 3 not in D*₂)
  h₂(3, 4): x₁=3 ≤ 3.0 → 12.50
  OOB prediction = 12.50
  OOB error for sample 3 = (25 - 12.50)² = 156.25

OOB MSE = (6.25 + 156.25) / 2 = 81.25
```

---

## 9. BaggingClassifier & BaggingRegressor in Scikit-Learn

### Classification Example

```python
import numpy as np
from sklearn.ensemble import BaggingClassifier
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report

# Generate synthetic dataset
X, y = make_classification(
    n_samples=1000, n_features=20, n_informative=10,
    n_redundant=5, random_state=42
)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# --- Single Decision Tree (baseline) ---
single_tree = DecisionTreeClassifier(random_state=42)
single_tree.fit(X_train, y_train)
y_pred_tree = single_tree.predict(X_test)
print(f"Single Tree Accuracy: {accuracy_score(y_test, y_pred_tree):.4f}")

# --- Bagging with Decision Trees ---
bagging_clf = BaggingClassifier(
    estimator=DecisionTreeClassifier(),   # base learner
    n_estimators=100,                     # number of trees
    max_samples=1.0,                      # fraction of samples per bootstrap
    max_features=1.0,                     # fraction of features per model
    bootstrap=True,                       # use bootstrap sampling
    bootstrap_features=False,             # don't bootstrap features
    oob_score=True,                       # compute OOB score
    n_jobs=-1,                            # use all CPU cores
    random_state=42
)
bagging_clf.fit(X_train, y_train)
y_pred_bag = bagging_clf.predict(X_test)

print(f"Bagging Accuracy:     {accuracy_score(y_test, y_pred_bag):.4f}")
print(f"OOB Score:            {bagging_clf.oob_score_:.4f}")
print(f"\nClassification Report:\n{classification_report(y_test, y_pred_bag)}")
```

### Regression Example

```python
from sklearn.ensemble import BaggingRegressor
from sklearn.tree import DecisionTreeRegressor
from sklearn.datasets import make_regression
from sklearn.metrics import mean_squared_error, r2_score

# Generate regression data
X, y = make_regression(
    n_samples=500, n_features=10, noise=15, random_state=42
)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Single tree baseline
single_tree = DecisionTreeRegressor(random_state=42)
single_tree.fit(X_train, y_train)
print(f"Single Tree MSE: {mean_squared_error(y_test, single_tree.predict(X_test)):.2f}")
print(f"Single Tree R²:  {r2_score(y_test, single_tree.predict(X_test)):.4f}")

# Bagging regressor
bagging_reg = BaggingRegressor(
    estimator=DecisionTreeRegressor(),
    n_estimators=100,
    max_samples=0.8,        # use 80% of samples
    max_features=0.8,       # use 80% of features
    bootstrap=True,
    oob_score=True,
    n_jobs=-1,
    random_state=42
)
bagging_reg.fit(X_train, y_train)
y_pred = bagging_reg.predict(X_test)
print(f"\nBagging MSE:     {mean_squared_error(y_test, y_pred):.2f}")
print(f"Bagging R²:      {r2_score(y_test, y_pred):.4f}")
print(f"OOB Score (R²):  {bagging_reg.oob_score_:.4f}")
```

### Key Parameters Explained

```python
BaggingClassifier(
    estimator=None,          # Base learner (default: DecisionTreeClassifier)
    n_estimators=10,         # Number of base learners
    max_samples=1.0,         # Number/fraction of samples per bootstrap
    max_features=1.0,        # Number/fraction of features per model
    bootstrap=True,          # Whether to use bootstrap sampling
    bootstrap_features=False,# Whether to bootstrap features
    oob_score=False,         # Whether to compute OOB score
    warm_start=False,        # Reuse previous fit to add more estimators
    n_jobs=None,             # Number of parallel jobs (-1 = all cores)
    random_state=None        # Random seed for reproducibility
)
```

### Visualizing the Effect of n_estimators

```python
import matplotlib.pyplot as plt

n_estimators_range = [1, 5, 10, 20, 50, 100, 200, 500]
train_scores = []
test_scores = []
oob_scores = []

for n in n_estimators_range:
    bag = BaggingClassifier(
        estimator=DecisionTreeClassifier(),
        n_estimators=n, oob_score=True,
        random_state=42, n_jobs=-1
    )
    bag.fit(X_train, y_train)
    train_scores.append(bag.score(X_train, y_train))
    test_scores.append(bag.score(X_test, y_test))
    oob_scores.append(bag.oob_score_)

plt.figure(figsize=(10, 6))
plt.plot(n_estimators_range, train_scores, 'b-o', label='Train')
plt.plot(n_estimators_range, test_scores, 'r-s', label='Test')
plt.plot(n_estimators_range, oob_scores, 'g--^', label='OOB')
plt.xlabel('Number of Estimators')
plt.ylabel('Accuracy')
plt.title('Bagging: Effect of Number of Estimators')
plt.legend()
plt.xscale('log')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

---

## 10. Applications

| Domain | Application | Why Bagging Helps |
|--------|-------------|-------------------|
| **Finance** | Credit scoring, stock prediction | Reduces variance in volatile financial data |
| **Healthcare** | Disease diagnosis from medical records | More robust predictions, reduces overfitting |
| **Remote Sensing** | Land cover classification from satellite imagery | Handles noisy, high-dimensional pixel data |
| **Manufacturing** | Quality control, defect detection | Stable predictions despite sensor noise |
| **E-commerce** | Customer churn prediction | Handles heterogeneous customer data well |
| **Ecology** | Species distribution modeling | Robust to sampling variability |

### When to Use Bagging

✅ **Use bagging when:**
- Base model has **high variance** (e.g., deep decision trees)
- Dataset has **noise** that causes overfitting
- You want a **stable, reliable** model
- Computational resources allow parallel training

❌ **Don't use bagging when:**
- Base model has **high bias** (e.g., linear regression on nonlinear data)
- Dataset is very **small** (bootstrap samples will be too similar)
- **Interpretability** is critical (ensemble of trees is harder to interpret)
- Training time is severely constrained

---

## 11. Summary Table

| Aspect | Details |
|--------|---------|
| **Full Name** | Bootstrap Aggregating |
| **Inventor** | Leo Breiman (1996) |
| **Core Idea** | Train multiple models on bootstrap samples, aggregate predictions |
| **Sampling** | With replacement, same size as original (n samples) |
| **Aggregation (Regression)** | Average of predictions |
| **Aggregation (Classification)** | Majority vote |
| **Primary Benefit** | Reduces variance |
| **Effect on Bias** | Approximately unchanged |
| **OOB Samples** | ~36.8% of data not in each bootstrap sample |
| **OOB Estimate** | Free validation — approximately unbiased |
| **Best Base Learners** | High-variance, low-bias models (e.g., unpruned trees) |
| **Parallelizable** | Yes — each model trained independently |
| **Key Hyperparameters** | n_estimators, max_samples, max_features |
| **Variance Formula** | Var = ρσ² + (1-ρ)σ²/B |
| **Limitations** | Cannot reduce bias; models still correlated |

---

## 12. Revision Questions

**Q1.** Explain why sampling **with replacement** is essential for bagging. What would happen if we sampled without replacement?

<details>
<summary>Answer</summary>

Sampling with replacement creates **diversity** among the bootstrap samples — each sample contains different subsets of the original data with some duplicates. This diversity leads to different models, which is necessary for variance reduction through averaging.

If we sampled **without replacement** with the same sample size as the original data, every bootstrap sample would be identical to the original dataset, producing identical models. Sampling without replacement with a smaller size would work but would discard data, increasing bias. Bootstrap sampling is the elegant solution that maintains sample size while introducing diversity.
</details>

**Q2.** Mathematically prove that approximately 63.2% of unique samples appear in each bootstrap sample of size n drawn from n samples.

<details>
<summary>Answer</summary>

P(sample xᵢ NOT selected in a single draw) = (n-1)/n = (1 - 1/n)

P(sample xᵢ NOT selected in any of n draws) = (1 - 1/n)ⁿ

As n → ∞: lim (1 - 1/n)ⁿ = e⁻¹ ≈ 0.3679

Therefore P(xᵢ IS selected at least once) = 1 - e⁻¹ ≈ 0.6321 = 63.2%
</details>

**Q3.** Given 5 models with individual variance σ² = 100 and pairwise correlation ρ = 0.6, what is the variance of the bagged ensemble? What if ρ = 0.2?

<details>
<summary>Answer</summary>

Using: Var(bagged) = ρσ² + (1-ρ)σ²/B

For ρ = 0.6, B = 5:
Var = 0.6 × 100 + (1-0.6) × 100/5 = 60 + 8 = **68**
(32% reduction from σ² = 100)

For ρ = 0.2, B = 5:
Var = 0.2 × 100 + (1-0.2) × 100/5 = 20 + 16 = **36**
(64% reduction from σ² = 100)

Lower correlation leads to much greater variance reduction.
</details>

**Q4.** Why is bagging ineffective when applied to linear regression models? What property of the base learner is required for bagging to work well?

<details>
<summary>Answer</summary>

Linear regression is a **stable** learner — small changes in training data produce similar models. The averaged prediction of many similar linear models is close to any single one, so variance reduction is minimal.

Bagging works best with **unstable** (high-variance) learners like deep decision trees, where small data changes cause large model changes. The diversity among unstable models is what bagging exploits. Additionally, linear regression already has low variance but potentially high bias, and bagging cannot reduce bias.
</details>

**Q5.** Explain the difference between OOB evaluation and k-fold cross-validation. What are the advantages and disadvantages of each?

<details>
<summary>Answer</summary>

**OOB Evaluation:**
- Uses the ~36.8% of samples left out of each bootstrap sample
- "Free" — no extra model training needed
- Approximately unbiased estimate of test error
- Only applicable to bagging-based methods
- Each sample's prediction uses ~63.2% of the ensemble models

**K-Fold Cross-Validation:**
- Splits data into k folds, trains k separate models
- Requires k × (training time) additional computation
- Can be applied to any model
- More established theoretical guarantees
- Uses all data for both training and validation
- More computationally expensive for ensembles (k × B models)

OOB is preferred for bagging ensembles due to its computational efficiency.
</details>

**Q6.** Design an experiment to show that bagging reduces variance but not bias. Describe the experimental setup, what you would measure, and what results you would expect.

<details>
<summary>Answer</summary>

**Setup:**
1. Define a known true function f(x) = sin(2πx)
2. Generate 100 different training datasets of size n=50 from f(x) + noise
3. For each dataset, train: (a) a single deep decision tree, (b) a bagged ensemble of 100 trees
4. Evaluate both on a fixed test grid

**Measurements:**
- For each test point x, compute across 100 datasets:
  - Bias² = (mean prediction - f(x))²
  - Variance = variance of predictions across datasets

**Expected Results:**
- **Variance:** Bagging should show dramatically lower variance than a single tree
- **Bias²:** Bagging should show approximately the same bias as a single tree (or very slightly higher)
- **Total Error:** Bagging should have lower total error, with the improvement coming entirely from variance reduction
</details>

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| ← [Unit 7](../../07-Decision-Trees/) | [Unit 8: Ensemble Methods](../README.md) | [8.2 Random Forests →](./02-random-forests.md) |

---

*© 2025 AIML Study Notes. Unit 8: Ensemble Methods — Chapter 1: Bagging.*
