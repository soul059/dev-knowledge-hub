# 📊 Chapter 4: Cross-Validation Techniques

> **Module:** 03-Model-Evaluation | **Topic:** Robust Model Assessment via Cross-Validation
> **Objective:** Master every major cross-validation strategy, understand their trade-offs, and apply them correctly in real-world ML pipelines.

---

## 4.1 Why Cross-Validation?

A single train/test (holdout) split is **unreliable** because:

- Results depend heavily on *which* samples land in train vs. test.
- Small datasets suffer high **variance** in performance estimates.
- A lucky or unlucky split can be misleading.

**Cross-validation (CV)** addresses this by evaluating the model on **multiple different splits** and **averaging** the results, yielding a more stable and trustworthy estimate.

```
Single Holdout (Risky)          Cross-Validation (Robust)
┌──────────┬─────┐              Split 1: [Test][  Train   ]
│  Train   │Test │   ──►        Split 2: [ Train ][Test][ T ]
└──────────┴─────┘              Split 3: [  Train   ][Test]
One estimate only                Average of K estimates
```

> **Key Insight:** CV does NOT produce a single final model — it produces a **reliable performance estimate**. The final model is retrained on the full dataset.

---

## 4.2 K-Fold Cross-Validation

The dataset is divided into **K** equally-sized folds. In each iteration, one fold is held out for testing and the remaining **K−1** folds are used for training. The process repeats K times.

**Performance Formula:**

```
CV_score = (1/K) × Σ (Score_i)    for i = 1 to K
```

### ASCII Diagram — 5-Fold CV Rotation

```
Fold:       F1      F2      F3      F4      F5       Score
Iter 1:   [TEST] [TRAIN] [TRAIN] [TRAIN] [TRAIN]  → S₁
Iter 2:   [TRAIN] [TEST] [TRAIN] [TRAIN] [TRAIN]  → S₂
Iter 3:   [TRAIN] [TRAIN] [TEST] [TRAIN] [TRAIN]  → S₃
Iter 4:   [TRAIN] [TRAIN] [TRAIN] [TEST] [TRAIN]  → S₄
Iter 5:   [TRAIN] [TRAIN] [TRAIN] [TRAIN] [TEST]  → S₅

Final Score = (S₁ + S₂ + S₃ + S₄ + S₅) / 5
```

### Python — K-Fold CV

```python
from sklearn.model_selection import KFold, cross_val_score
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris

X, y = load_iris(return_X_y=True)
model = RandomForestClassifier(n_estimators=100, random_state=42)

kf = KFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=kf, scoring='accuracy')

print(f"Fold scores : {scores}")
print(f"Mean ± Std  : {scores.mean():.4f} ± {scores.std():.4f}")
```

---

## 4.3 Stratified K-Fold

Standard K-Fold can produce folds with **imbalanced class distributions**, especially on skewed datasets. **Stratified K-Fold** ensures each fold mirrors the original class proportions.

```
Original data: 70% Class-A, 30% Class-B

Standard K-Fold (fold 3):   85% A, 15% B  ← skewed!
Stratified K-Fold (fold 3): 70% A, 30% B  ← preserved ✓
```

```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=skf, scoring='accuracy')
print(f"Stratified Mean: {scores.mean():.4f}")
```

> **Best Practice:** Always prefer `StratifiedKFold` for **classification** tasks.

---

## 4.4 Leave-One-Out Cross-Validation (LOOCV)

A special case where **K = N** (number of samples). Each iteration uses one sample as the test set and the remaining N−1 for training.

```
Samples:   x₁   x₂   x₃   ...   xₙ
Iter 1:   [T ]  trn  trn  ...   trn   → S₁
Iter 2:    trn [T ]  trn  ...   trn   → S₂
  ...
Iter N:    trn  trn  trn  ...  [T ]   → Sₙ

CV_score = (1/N) × Σ Sᵢ
```

| Pros | Cons |
|------|------|
| Nearly **unbiased** estimate | **Very expensive** — N model fits |
| Uses maximum training data | High **variance** (each test set = 1 point) |
| Deterministic — no randomness | Impractical for large N |

```python
from sklearn.model_selection import LeaveOneOut

loo = LeaveOneOut()
scores = cross_val_score(model, X, y, cv=loo, scoring='accuracy')
print(f"LOOCV Accuracy: {scores.mean():.4f}  (N={len(scores)} fits)")
```

---

## 4.5 Leave-P-Out Cross-Validation

Generalizes LOOCV by leaving **P** samples out at a time. The number of iterations is **C(N, P)** — combinatorially explosive even for moderate P.

```
Leave-P-Out (P=2, N=5):  C(5,2) = 10 iterations
```

> Used mainly in **theoretical analysis**; rarely practical for P > 1.

---

## 4.6 Repeated K-Fold

Runs K-Fold **multiple rounds** with different random shuffles, then averages all results. This reduces variance further.

```python
from sklearn.model_selection import RepeatedStratifiedKFold

rskf = RepeatedStratifiedKFold(n_splits=5, n_repeats=10, random_state=42)
scores = cross_val_score(model, X, y, cv=rskf, scoring='accuracy')
print(f"Repeated 5-Fold (10x): {scores.mean():.4f} ± {scores.std():.4f}")
print(f"Total fits: {len(scores)}")   # 5 × 10 = 50 fits
```

---

## 4.7 Group K-Fold

When samples come in **groups** (e.g., multiple readings per patient), standard CV can leak information if the same group appears in both train and test.

**Group K-Fold** ensures all samples from a group stay together in the same fold.

```
Patient:   P1  P1  P2  P2  P3  P3  P4  P4
──────────────────────────────────────────
BAD split:  [Train P1] ... [Test P1]  ← data leakage!
Group CV:   [All of P1 in Train] [All of P3 in Test]  ✓
```

```python
from sklearn.model_selection import GroupKFold
import numpy as np

groups = np.array([1, 1, 2, 2, 3, 3, 4, 4, 5, 5])
gkf = GroupKFold(n_splits=5)

for fold, (train_idx, test_idx) in enumerate(gkf.split(X[:10], y[:10], groups)):
    print(f"Fold {fold}: Train groups {set(groups[train_idx])}, "
          f"Test groups {set(groups[test_idx])}")
```

---

## 4.8 Time Series Split

Standard CV **violates temporal order** — the model could train on future data and test on past data. **Time Series Split** uses an **expanding training window** and always tests on the next segment.

### ASCII Diagram — Time Series Split (5 splits)

```
Time ──────────────────────────────────────►

Split 1:  [TRAIN     ] [TEST]
Split 2:  [TRAIN          ] [TEST]
Split 3:  [TRAIN               ] [TEST]
Split 4:  [TRAIN                    ] [TEST]
Split 5:  [TRAIN                         ] [TEST]

✓ No future data leaks into training
✓ Mimics real-world deployment (train on past, predict future)
```

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)
scores = cross_val_score(model, X, y, cv=tscv, scoring='accuracy')

for i, (train_idx, test_idx) in enumerate(tscv.split(X)):
    print(f"Split {i}: Train[0..{train_idx[-1]}] → Test[{test_idx[0]}..{test_idx[-1]}]")
```

> **Application:** Stock price prediction, weather forecasting, demand planning.

---

## 4.9 Nested Cross-Validation

When you tune hyperparameters *and* evaluate performance using the **same** CV loop, you get an **optimistically biased** estimate. **Nested CV** fixes this with two loops:

```
┌─── Outer Loop (K₁ folds) ── Performance Estimation ───┐
│                                                        │
│  For each outer fold:                                  │
│   ┌─── Inner Loop (K₂ folds) ── Hyperparameter Tuning │
│   │  GridSearchCV / RandomizedSearchCV                 │
│   │  Select best hyperparameters                       │
│   └────────────────────────────────────────────────────┘
│   Train best model on outer-train, evaluate on outer-test
└────────────────────────────────────────────────────────┘
```

```python
from sklearn.model_selection import cross_val_score, GridSearchCV, StratifiedKFold

inner_cv = StratifiedKFold(n_splits=3, shuffle=True, random_state=42)
outer_cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

param_grid = {'n_estimators': [50, 100], 'max_depth': [3, 5, None]}
grid_search = GridSearchCV(model, param_grid, cv=inner_cv, scoring='accuracy')

nested_scores = cross_val_score(grid_search, X, y, cv=outer_cv, scoring='accuracy')
print(f"Nested CV Accuracy: {nested_scores.mean():.4f} ± {nested_scores.std():.4f}")
```

---

## 4.10 Choosing K — Trade-offs

| K Value | Bias | Variance | Computation | When to Use |
|---------|------|----------|-------------|-------------|
| K = 2 | High | Low | Fast | Very large datasets |
| **K = 5** | Moderate | Moderate | Moderate | **Good default** |
| **K = 10** | Low | Moderate | Moderate | **Most common choice** |
| K = N (LOO) | Lowest | High | Very slow | Very small datasets |

```
Bias-Variance Trade-off as K increases:

Bias    ████████░░  (K=2)  →  ██░░░░░░░░  (K=10)  →  ░░░░░░░░░░  (LOO)
Variance░░░░░░░░░░  (K=2)  →  ████░░░░░░  (K=10)  →  ████████░░  (LOO)
Cost    ██░░░░░░░░  (K=2)  →  ██████░░░░  (K=10)  →  ██████████  (LOO)
```

> **Rule of thumb:** Use **K = 5 or 10** with stratification and shuffling for most tasks.

---

## 4.11 Real-World ML Applications

| Domain | CV Strategy | Reason |
|--------|-------------|--------|
| Medical diagnosis | **Stratified K-Fold** | Rare disease → imbalanced classes |
| Patient drug trials | **Group K-Fold** | Multiple samples per patient |
| Stock price prediction | **Time Series Split** | Temporal ordering matters |
| Small clinical dataset | **LOOCV** | Maximize training data |
| Model selection pipeline | **Nested CV** | Unbiased tuning + evaluation |
| Kaggle competitions | **Repeated Stratified K-Fold** | Lowest variance estimate |

---

## 📋 Summary Comparison Table

| Technique | Folds / Iterations | Preserves Class Dist? | Handles Groups? | Handles Time? | Computation |
|-----------|-------------------|-----------------------|-----------------|---------------|-------------|
| K-Fold | K | ✗ | ✗ | ✗ | Low |
| Stratified K-Fold | K | ✓ | ✗ | ✗ | Low |
| LOOCV | N | ✗ | ✗ | ✗ | Very High |
| Leave-P-Out | C(N,P) | ✗ | ✗ | ✗ | Extreme |
| Repeated K-Fold | K × R | ✓ (if stratified) | ✗ | ✗ | Moderate |
| Group K-Fold | K | ✗ | ✓ | ✗ | Low |
| Time Series Split | K | ✗ | ✗ | ✓ | Low |
| Nested CV | K₁ × K₂ | ✓ (if stratified) | ✗ | ✗ | High |

---

## ❓ Revision Questions

1. **Why is a single holdout split considered unreliable**, and how does K-Fold CV address this problem?

2. **Explain Stratified K-Fold.** Why is it preferred over standard K-Fold for classification on imbalanced datasets?

3. **What are the pros and cons of LOOCV?** When would you choose it over 10-Fold CV?

4. **A medical study collects 5 blood samples per patient.** Which CV strategy prevents data leakage, and why?

5. **You are building a stock price forecasting model.** Why would standard K-Fold give misleading results? Which technique should you use instead?

6. **Describe Nested Cross-Validation.** Why is it necessary when combining hyperparameter tuning with model evaluation?

---

## 🧭 Navigation

| ← Previous | ↑ Up | Next → |
|------------|------|--------|
| [03 — Confusion Matrix](03-confusion-matrix.md) | [Model Evaluation](README.md) | [05 — Hyperparameter Tuning](05-hyperparameter-tuning.md) |
