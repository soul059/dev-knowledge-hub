# 📘 Chapter 4: Cross-Validation

> **Unit 5: Scikit-learn Basics** · Foundations of AI/ML

---

## 🎯 Overview

A single train-test split gives one estimate of model performance that may vary depending on which samples end up in each set. **Cross-validation (CV)** provides a more robust evaluation by training and testing on multiple different subsets. This chapter covers all major CV strategies in scikit-learn.

---

## 1. Why Cross-Validation?

| Problem with Single Split | How CV Solves It |
|---|---|
| Results depend on which data is in test set | Averages over multiple splits |
| Small datasets waste data (less for training) | Every sample is used for both training and testing |
| High variance in performance estimate | Multiple folds reduce variance |

---

## 2. K-Fold Cross-Validation

Splits data into **K** equal-sized folds. Each fold serves as the test set once while the remaining K-1 folds form the training set.

```python
from sklearn.model_selection import KFold
import numpy as np

X = np.arange(20).reshape(10, 2)
y = np.array([0, 0, 0, 0, 0, 1, 1, 1, 1, 1])

kf = KFold(n_splits=5, shuffle=True, random_state=42)

for fold, (train_idx, test_idx) in enumerate(kf.split(X)):
    print(f"Fold {fold}: Train={train_idx}, Test={test_idx}")
```

### Manual K-Fold Loop

```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_iris

X, y = load_iris(return_X_y=True)
kf = KFold(n_splits=5, shuffle=True, random_state=42)

scores = []
for train_idx, test_idx in kf.split(X):
    X_train, X_test = X[train_idx], X[test_idx]
    y_train, y_test = y[train_idx], y[test_idx]

    model = LogisticRegression(max_iter=200)
    model.fit(X_train, y_train)
    score = model.score(X_test, y_test)
    scores.append(score)

print(f"Scores: {scores}")
print(f"Mean: {np.mean(scores):.4f} ± {np.std(scores):.4f}")
```

---

## 3. Stratified K-Fold

Preserves the **class distribution** in each fold — critical for imbalanced datasets.

```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

for fold, (train_idx, test_idx) in enumerate(skf.split(X, y)):
    train_dist = np.bincount(y[train_idx]) / len(train_idx)
    test_dist = np.bincount(y[test_idx]) / len(test_idx)
    print(f"Fold {fold}: Train={train_dist}, Test={test_dist}")
    # Each fold has approximately the same class proportions
```

> 💡 **Note:** `cross_val_score` uses `StratifiedKFold` by default for classifiers and `KFold` for regressors.

---

## 4. cross_val_score — The Easy Way

```python
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import RandomForestClassifier

clf = RandomForestClassifier(n_estimators=100, random_state=42)

# 5-fold stratified CV (default for classifiers)
scores = cross_val_score(clf, X, y, cv=5, scoring='accuracy')

print(f"Fold scores: {scores}")
print(f"Mean accuracy: {scores.mean():.4f} ± {scores.std():.4f}")
```

### Custom CV Strategy

```python
# Pass a specific CV splitter
cv_strategy = StratifiedKFold(n_splits=10, shuffle=True, random_state=42)
scores = cross_val_score(clf, X, y, cv=cv_strategy, scoring='f1_macro')
```

### Common Scoring Options

| Scoring String | Task | Description |
|---|---|---|
| `'accuracy'` | Classification | Fraction correct |
| `'f1'` | Binary classification | Harmonic mean of precision & recall |
| `'f1_macro'` | Multi-class | Macro-averaged F1 |
| `'roc_auc'` | Binary classification | Area under ROC curve |
| `'neg_mean_squared_error'` | Regression | Negative MSE (higher is better) |
| `'r2'` | Regression | R² score |

> ⚠️ Regression metrics are **negated** (e.g., `neg_mean_squared_error`) so that `cross_val_score` can maximize.

---

## 5. cross_val_predict — Get Predictions

Returns out-of-fold predictions for every sample.

```python
from sklearn.model_selection import cross_val_predict
from sklearn.metrics import accuracy_score

y_pred = cross_val_predict(clf, X, y, cv=5)

# Each prediction was made when that sample was in the test fold
accuracy = accuracy_score(y, y_pred)
print(f"CV Accuracy: {accuracy:.4f}")

# Get probabilities instead of labels
y_proba = cross_val_predict(clf, X, y, cv=5, method='predict_proba')
print(f"Probabilities shape: {y_proba.shape}")  # (150, 3) for 3 classes
```

> 💡 **Use case:** Build a confusion matrix or ROC curve from CV predictions to see overall model behavior.

---

## 6. Leave-One-Out (LOO) Cross-Validation

Each sample serves as a test set of size 1. N folds for N samples.

```python
from sklearn.model_selection import LeaveOneOut

loo = LeaveOneOut()
scores = cross_val_score(clf, X, y, cv=loo)

print(f"LOO scores: {len(scores)} folds")     # 150 folds for Iris
print(f"Mean accuracy: {scores.mean():.4f}")
```

| Pros | Cons |
|---|---|
| Maximum training data per fold | Very slow for large datasets |
| No randomness (deterministic) | High variance in estimates |
| Good for very small datasets | Computationally expensive |

---

## 7. Repeated K-Fold

Runs K-Fold multiple times with different random splits — reduces variance of the estimate.

```python
from sklearn.model_selection import RepeatedStratifiedKFold

rskf = RepeatedStratifiedKFold(n_splits=5, n_repeats=10, random_state=42)
scores = cross_val_score(clf, X, y, cv=rskf, scoring='accuracy')

print(f"Total folds: {len(scores)}")         # 50 (5 folds × 10 repeats)
print(f"Mean: {scores.mean():.4f} ± {scores.std():.4f}")
```

---

## 8. GroupKFold — Grouped Data

When data has groups (e.g., multiple samples per patient), ensure no group appears in both train and test.

```python
from sklearn.model_selection import GroupKFold

# Example: 10 samples from 5 patients (2 each)
groups = np.array([0, 0, 1, 1, 2, 2, 3, 3, 4, 4])

gkf = GroupKFold(n_splits=5)

for fold, (train_idx, test_idx) in enumerate(gkf.split(X[:10], y[:10], groups)):
    train_groups = set(groups[train_idx])
    test_groups = set(groups[test_idx])
    print(f"Fold {fold}: Train groups={train_groups}, Test groups={test_groups}")
    # No group overlap between train and test
```

---

## 9. TimeSeriesSplit

For time-ordered data — training set always precedes test set chronologically.

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)

X_ts = np.arange(24).reshape(12, 2)
y_ts = np.arange(12)

for fold, (train_idx, test_idx) in enumerate(tscv.split(X_ts)):
    print(f"Fold {fold}: Train={train_idx}, Test={test_idx}")

# Fold 0: Train=[0 1], Test=[2 3]
# Fold 1: Train=[0 1 2 3], Test=[4 5]
# ...training window expands forward in time

# Use with cross_val_score
from sklearn.linear_model import LinearRegression
scores = cross_val_score(LinearRegression(), X_ts, y_ts, cv=tscv,
                         scoring='neg_mean_squared_error')
```

---

## 10. Nested Cross-Validation

**Outer loop** evaluates model performance. **Inner loop** tunes hyperparameters. Prevents optimistic bias from hyperparameter tuning.

```python
from sklearn.model_selection import GridSearchCV, cross_val_score
from sklearn.svm import SVC

# Inner CV: hyperparameter tuning
param_grid = {'C': [0.1, 1, 10], 'kernel': ['rbf', 'linear']}
inner_cv = StratifiedKFold(n_splits=3, shuffle=True, random_state=42)
grid_search = GridSearchCV(SVC(), param_grid, cv=inner_cv, scoring='accuracy')

# Outer CV: performance evaluation
outer_cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
nested_scores = cross_val_score(grid_search, X, y, cv=outer_cv, scoring='accuracy')

print(f"Nested CV accuracy: {nested_scores.mean():.4f} ± {nested_scores.std():.4f}")
```

### Why Nested CV?

```
Without nesting:  GridSearchCV selects best params → same data evaluates → optimistic
With nesting:     Each outer fold has its own GridSearchCV → unbiased estimate
```

---

## 11. Cross-Validation for Hyperparameter Tuning

### GridSearchCV

```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier

param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [3, 5, 10, None],
    'min_samples_split': [2, 5, 10]
}

grid_search = GridSearchCV(
    RandomForestClassifier(random_state=42),
    param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,        # use all CPU cores
    verbose=1
)

grid_search.fit(X, y)

print(f"Best params: {grid_search.best_params_}")
print(f"Best score:  {grid_search.best_score_:.4f}")
print(f"Best model:  {grid_search.best_estimator_}")
```

### RandomizedSearchCV (Faster)

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, uniform

param_distributions = {
    'n_estimators': randint(50, 300),
    'max_depth': randint(3, 15),
    'min_samples_split': randint(2, 20)
}

random_search = RandomizedSearchCV(
    RandomForestClassifier(random_state=42),
    param_distributions,
    n_iter=20,         # number of random combinations to try
    cv=5,
    scoring='accuracy',
    random_state=42,
    n_jobs=-1
)

random_search.fit(X, y)
print(f"Best params: {random_search.best_params_}")
```

---

## 12. Interpreting CV Scores

```python
scores = cross_val_score(clf, X, y, cv=10, scoring='accuracy')

print(f"Individual fold scores: {scores}")
print(f"Mean:   {scores.mean():.4f}")
print(f"Std:    {scores.std():.4f}")
print(f"95% CI: {scores.mean():.4f} ± {1.96 * scores.std():.4f}")

# High std → model performance varies a lot between folds
# → possibly small dataset, or model is sensitive to data composition
```

| Observation | Interpretation |
|---|---|
| High mean, low std | Stable, well-performing model |
| High mean, high std | Good average but inconsistent |
| Low mean, low std | Consistently poor — change model |
| Low mean, high std | Unreliable — need more data or different approach |

---

## 📋 Summary Table

| CV Strategy | When to Use | Key Parameter |
|---|---|---|
| `KFold` | General purpose (regression) | `n_splits` |
| `StratifiedKFold` | Classification (preserves class ratio) | `n_splits` |
| `LeaveOneOut` | Very small datasets | — |
| `RepeatedStratifiedKFold` | More reliable estimates | `n_repeats` |
| `GroupKFold` | Grouped/clustered data | `n_splits` |
| `TimeSeriesSplit` | Temporal data | `n_splits` |
| `cross_val_score` | Quick CV evaluation | `scoring` |
| `cross_val_predict` | Get out-of-fold predictions | `method` |
| `GridSearchCV` | Exhaustive hyperparameter search | `param_grid` |
| `RandomizedSearchCV` | Random hyperparameter search | `n_iter` |

---

## ❓ Revision Questions

1. **Why is 5-fold or 10-fold cross-validation preferred over a single train-test split?**
2. **When should you use `StratifiedKFold` instead of regular `KFold`?**
3. **What is the difference between `cross_val_score` and `cross_val_predict`?**
4. **Why is Leave-One-Out CV impractical for large datasets?**
5. **Explain the purpose of nested cross-validation and why it gives unbiased estimates.**
6. **When would you use `GroupKFold` or `TimeSeriesSplit` instead of standard K-Fold?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← 03 - Train-Test Split](03-train-test-split.md) | [Unit 5: Scikit-learn Basics](../README.md) | [05 - Pipeline Creation →](05-pipeline-creation.md) |
