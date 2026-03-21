# ✅ Chapter 7: Validation Strategies

> **Unit 3 – Model Evaluation** | Choose the right validation scheme for reliable, unbiased performance estimates.

---

## 7.1 Hold-Out Validation

Split data into **train** and **test** (e.g., 80/20) once.

```
┌───────────────────────────────────────────────┐
│  ██████████████████████████  │  ░░░░░░░░░░░░  │
│         Training (80 %)      │   Test (20 %)   │
└───────────────────────────────────────────────┘
```

| Pros | Cons |
|---|---|
| Fast, simple | High variance — score depends on split |
| Works for very large datasets | Small datasets lose valuable training data |

**When to use**: datasets > 100 k samples where a single split is representative enough.

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)
```

---

## 7.2 K-Fold Cross-Validation

Partition data into **K** equally-sized folds. Train on K−1 folds, validate on the remaining one. Repeat K times.

```
Fold 1: [ ░░░░ | ████ | ████ | ████ | ████ ]   ← validate on fold 1
Fold 2: [ ████ | ░░░░ | ████ | ████ | ████ ]   ← validate on fold 2
Fold 3: [ ████ | ████ | ░░░░ | ████ | ████ ]   ← validate on fold 3
Fold 4: [ ████ | ████ | ████ | ░░░░ | ████ ]   ← validate on fold 4
Fold 5: [ ████ | ████ | ████ | ████ | ░░░░ ]   ← validate on fold 5

Final Score = mean(score_1, score_2, ..., score_K)
```

The variance of the estimate decreases as K increases:

```
SE(score) = std(scores) / √K
```

| K Value | Trade-off |
|---|---|
| K = 5 | Good balance of bias and variance; most common |
| K = 10 | Lower bias, more computation |
| K = N (LOO) | Nearly unbiased but very high variance and cost |

```python
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import GradientBoostingClassifier

model = GradientBoostingClassifier(n_estimators=100, random_state=42)
scores = cross_val_score(model, X, y, cv=5, scoring="accuracy")
print(f"Mean Accuracy: {scores.mean():.4f} ± {scores.std():.4f}")
```

---

## 7.3 Stratified Validation

**Stratified K-Fold** ensures each fold preserves the original class distribution — critical for **imbalanced datasets**.

```
Original dataset:  90 % Class 0  |  10 % Class 1

Regular K-Fold (fold 3):   95 % / 5 %   ← skewed!
Stratified K-Fold (fold 3): 90 % / 10 %  ← preserved ✓
```

Without stratification, minority-class metrics (recall, F1) become unreliable.

```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

for fold, (train_idx, val_idx) in enumerate(skf.split(X, y), 1):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]
    model.fit(X_train, y_train)
    score = model.score(X_val, y_val)
    print(f"Fold {fold}: {score:.4f}")
```

> **Rule of thumb**: Always use stratified splits for classification unless you have a specific reason not to.

---

## 7.4 Time-Series Validation

Standard K-Fold **randomly** shuffles data, violating temporal order and causing **data leakage** (future data used to predict the past).

### Walk-Forward (Expanding Window) Validation

```
Split 1: [ Train████ | Val░░ |            ]
Split 2: [ Train████████ | Val░░ |        ]
Split 3: [ Train████████████ | Val░░ |    ]
Split 4: [ Train████████████████ | Val░░  ]
                                     ──▶ time
```

The training window **grows**; validation always lies in the future relative to training.

### Sliding Window Variant

```
Split 1: [ Train████ | Val░░ |            ]
Split 2: [    Train████ | Val░░ |         ]
Split 3: [        Train████ | Val░░ |     ]
```

Fixed-size training window — useful when older data becomes less relevant (concept drift).

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)

for fold, (train_idx, val_idx) in enumerate(tscv.split(X), 1):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]
    model.fit(X_train, y_train)
    score = model.score(X_val, y_val)
    print(f"Fold {fold}: {score:.4f}")
```

---

## 7.5 Group-Based Validation

When samples share a **group identity** (e.g., same patient, same user, same device), related samples must stay in the **same fold** to prevent data leakage.

```
Patient A: [s1, s2, s3]  ──▶ ALL in Train   or ALL in Val
Patient B: [s4, s5]      ──▶ ALL in Train   or ALL in Val
Patient C: [s6, s7, s8]  ──▶ ALL in Train   or ALL in Val
```

```python
from sklearn.model_selection import GroupKFold
import numpy as np

groups = np.array([1,1,1, 2,2, 3,3,3, 4,4, 5,5,5])

gkf = GroupKFold(n_splits=5)
for fold, (train_idx, val_idx) in enumerate(gkf.split(X, y, groups), 1):
    print(f"Fold {fold} — Train groups: {set(groups[train_idx])}, "
          f"Val groups: {set(groups[val_idx])}")
    model.fit(X[train_idx], y[train_idx])
    score = model.score(X[val_idx], y[val_idx])
    print(f"  Score: {score:.4f}")
```

---

## 7.6 Nested Cross-Validation

When you **tune hyperparameters** and **evaluate** performance, using the same CV loop introduces optimistic bias. **Nested CV** separates the two concerns:

```
┌─ Outer Loop (K₁ = 5 folds) ── Performance Estimation ──────────┐
│                                                                  │
│  Fold 1:  [ Train ████████████████████ ]  [ Test ░░░░ ]         │
│           ┌─ Inner Loop (K₂ = 3 folds) ── Hyperparameter Tuning │
│           │  Sub-fold A: [Tr██████ | Val░░]                      │
│           │  Sub-fold B: [Tr██ | Val░░ | Tr██]                   │
│           │  Sub-fold C: [Val░░ | Tr██████]                      │
│           └──────────────────────────────────────────────────────│
│           Best params → retrain on full Train → score on Test    │
│                                                                  │
│  Fold 2–5: (repeat)                                              │
└──────────────────────────────────────────────────────────────────┘

Final Score = mean(outer_test_score_1, ..., outer_test_score_K₁)
```

```python
from sklearn.model_selection import cross_val_score, GridSearchCV
from sklearn.svm import SVC

inner_cv = StratifiedKFold(n_splits=3, shuffle=True, random_state=42)
outer_cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

param_grid = {"C": [0.1, 1, 10], "gamma": ["scale", "auto"]}

clf = GridSearchCV(
    SVC(kernel="rbf"), param_grid,
    cv=inner_cv, scoring="accuracy", n_jobs=-1
)

nested_scores = cross_val_score(clf, X, y, cv=outer_cv, scoring="accuracy")
print(f"Nested CV Accuracy: {nested_scores.mean():.4f} ± {nested_scores.std():.4f}")
```

---

## 7.7 Choosing the Right Strategy — Decision Flowchart

```
                      ┌──────────────────┐
                      │ Is data ordered  │
                      │   in time?       │
                      └────────┬─────────┘
                        Yes ╱     ╲ No
                  ┌────────╱       ╲────────────┐
                  │ TimeSeriesSplit │            │
                  │ (walk-forward)  │  ┌─────────▼──────────┐
                  └─────────────────┘  │ Are there natural  │
                                       │ groups / clusters? │
                                       └────────┬───────────┘
                                         Yes ╱     ╲ No
                                  ┌─────────╱       ╲──────────┐
                                  │  GroupKFold      │          │
                                  └──────────────────┘  ┌──────▼─────────┐
                                                        │ Classification │
                                                        │ & imbalanced?  │
                                                        └───────┬────────┘
                                                         Yes ╱    ╲ No
                                                  ┌─────────╱      ╲────────┐
                                                  │ StratifiedKFold │       │
                                                  └─────────────────┘ ┌─────▼────┐
                                                                      │ KFold /  │
                                                                      │ Hold-Out │
                                                                      └──────────┘
```

**Also ask**: Do I need unbiased evaluation **and** tuning? → **Nested CV**.

---

## 7.8 Anti-Patterns to Avoid

| Anti-Pattern | Why It's Harmful | Fix |
|---|---|---|
| **Using test set for tuning** | Optimistic, biased estimate | Separate validation & test sets, or use nested CV |
| **Shuffling time-series data** | Future leaks into training | Use `TimeSeriesSplit` |
| **Ignoring groups** | Related samples in train + val | Use `GroupKFold` |
| **Not stratifying imbalanced data** | Folds miss minority class | Use `StratifiedKFold` |
| **Preprocessing before splitting** | Scaling/imputation leaks stats | Fit preprocessor on train only (use `Pipeline`) |
| **Reporting inner-CV score as final** | Optimistically biased | Report outer-loop score from nested CV |

### Example — Preprocessing Leak Fix

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("clf", LogisticRegression())
])

# Correct: scaler fits only on each training fold internally
scores = cross_val_score(pipe, X, y, cv=5, scoring="accuracy")
```

---

## 7.9 Summary — Strategy Comparison Table

| Strategy | Variance of Estimate | Handles Imbalance | Temporal Order | Groups | Cost |
|---|---|---|---|---|---|
| Hold-Out | High | With `stratify` | ✗ | ✗ | Very Low |
| K-Fold | Medium | ✗ | ✗ | ✗ | Medium |
| Stratified K-Fold | Medium | ✓ | ✗ | ✗ | Medium |
| TimeSeriesSplit | Medium | ✗ | ✓ | ✗ | Medium |
| GroupKFold | Medium | ✗ | ✗ | ✓ | Medium |
| Nested CV | Low (unbiased) | ✓ (if stratified) | ✓ (if TS inner) | ✓ (if grouped) | High |

---

## 7.10 Real-World Scenario — Medical Imaging

**Problem**: Classify lung X-rays as normal vs pneumonia. Dataset has 3 000 images from 800 patients (multiple scans per patient), with 70 % normal / 30 % pneumonia.

**Correct strategy**: **Stratified GroupKFold**

1. **Groups** = patient IDs → prevents the same patient appearing in train and val.
2. **Stratification** → each fold maintains the 70/30 class ratio.
3. **Nested CV** outer loop for unbiased evaluation; inner loop for tuning CNN learning rate.

```python
from sklearn.model_selection import StratifiedGroupKFold

sgkf = StratifiedGroupKFold(n_splits=5, shuffle=True, random_state=42)

for fold, (train_idx, val_idx) in enumerate(sgkf.split(X, y, groups), 1):
    assert set(groups[train_idx]).isdisjoint(set(groups[val_idx]))
    print(f"Fold {fold} — Train: {len(train_idx)}, Val: {len(val_idx)}")
```

Without group splitting, the model memorises patient-specific features and reports 96 % accuracy that drops to 82 % on truly new patients.

---

## 7.11 Revision Questions

1. **Explain** why hold-out validation has high variance and describe a scenario where it is still appropriate.
2. In K-Fold CV, how does increasing K affect (a) bias of the estimate and (b) computational cost? What is the extreme case when K = N?
3. A dataset contains 95 % negative and 5 % positive samples. **Why** is `StratifiedKFold` essential, and what could go wrong without it?
4. You are building a stock-price prediction model. **Which** validation strategy must you use and **why** would standard K-Fold give misleadingly optimistic results?
5. A clinical trial dataset has multiple measurements per patient. **Describe** the data leakage risk and the correct validation strategy to mitigate it.
6. **Compare** nested cross-validation with standard `GridSearchCV` + `cross_val_score`. When is nested CV necessary and what bias does it eliminate?

---

## Navigation

| ← Previous | Next → |
|---|---|
| [06 – Learning Curves](06-learning-curves.md) | — *(End of Unit 3)* |
