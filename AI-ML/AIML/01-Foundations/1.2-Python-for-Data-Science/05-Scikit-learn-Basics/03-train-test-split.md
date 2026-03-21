# 📘 Chapter 3: Train-Test Split

> **Unit 5: Scikit-learn Basics** · Foundations of AI/ML

---

## 🎯 Overview

A machine learning model must be evaluated on data it has **never seen during training**. Splitting data into training and test sets is the most fundamental evaluation strategy. This chapter covers `train_test_split`, stratified splitting, time series considerations, data leakage, and best practices.

---

## 1. Why Split Data?

| Problem | What Happens |
|---|---|
| Training and testing on the **same data** | Model memorizes answers → inflated accuracy |
| No held-out test set | No way to estimate real-world performance |
| Information from test leaks into training | Overly optimistic evaluation (data leakage) |

The goal: **estimate how well your model will generalize to unseen data.**

---

## 2. train_test_split Function

```python
from sklearn.model_selection import train_test_split
from sklearn.datasets import load_iris

X, y = load_iris(return_X_y=True)

# Basic split — 80% train, 20% test
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

print(f"Training set:  {X_train.shape[0]} samples")
print(f"Test set:      {X_test.shape[0]} samples")
```

### Key Parameters

| Parameter | Description | Default |
|---|---|---|
| `test_size` | Fraction (0.0–1.0) or absolute number of test samples | `0.25` |
| `train_size` | Fraction or absolute number of training samples | complement of `test_size` |
| `random_state` | Seed for reproducible splits | `None` |
| `shuffle` | Whether to shuffle data before splitting | `True` |
| `stratify` | Array to stratify by (preserves class proportions) | `None` |

### Using Absolute Numbers

```python
# Exactly 50 samples for test
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=50, random_state=42
)
```

---

## 3. Reproducibility with random_state

```python
# Without random_state — different split each run
X_train1, X_test1, _, _ = train_test_split(X, y, test_size=0.2)
X_train2, X_test2, _, _ = train_test_split(X, y, test_size=0.2)
# X_train1 and X_train2 are likely DIFFERENT

# With random_state — identical split every run
X_train1, X_test1, _, _ = train_test_split(X, y, test_size=0.2, random_state=42)
X_train2, X_test2, _, _ = train_test_split(X, y, test_size=0.2, random_state=42)
# X_train1 and X_train2 are IDENTICAL
```

> 💡 **Best Practice:** Always set `random_state` during development for reproducibility. Use different seeds to check robustness.

---

## 4. Stratified Splitting for Imbalanced Data

When class distributions are imbalanced, a random split may leave some classes underrepresented in the test set.

```python
import numpy as np
from sklearn.datasets import make_classification

# Imbalanced dataset: 95% class 0, 5% class 1
X, y = make_classification(
    n_samples=1000, n_classes=2, weights=[0.95, 0.05], random_state=42
)

print(f"Class distribution: {np.bincount(y)}")  # [950, 50]

# ❌ Without stratify — test set may have very few minority samples
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
print(f"Test set classes: {np.bincount(y_test)}")  # proportions may vary

# ✅ With stratify — preserves class proportions in both splits
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)
print(f"Train classes: {np.bincount(y_train)}")  # ~[760, 40]
print(f"Test classes:  {np.bincount(y_test)}")   # ~[190, 10]
```

### Verifying Proportions

```python
print(f"Original:  {np.bincount(y) / len(y)}")
print(f"Train set: {np.bincount(y_train) / len(y_train)}")
print(f"Test set:  {np.bincount(y_test) / len(y_test)}")
# All should show approximately [0.95, 0.05]
```

---

## 5. Train / Validation / Test Philosophy

| Set | Purpose | Typical Size |
|---|---|---|
| **Training** | Learn model parameters | 60–80% |
| **Validation** | Tune hyperparameters, select model | 10–20% |
| **Test** | Final unbiased performance estimate | 10–20% |

### Two-Way Split (Simple)

```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
```

### Three-Way Split

```python
# First split: separate test set
X_temp, X_test, y_temp, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Second split: separate validation from training
X_train, X_val, y_train, y_val = train_test_split(
    X_temp, y_temp, test_size=0.25, random_state=42, stratify=y_temp
)
# Result: 60% train, 20% validation, 20% test

print(f"Train: {len(X_train)}, Val: {len(X_val)}, Test: {len(X_test)}")
```

---

## 6. Typical Split Ratios

| Dataset Size | Recommended Split | Notes |
|---|---|---|
| < 1,000 | Use **cross-validation** instead | Too small for fixed split |
| 1,000–10,000 | 80/20 or 70/15/15 | Standard practice |
| 10,000–100,000 | 80/10/10 | Enough data for smaller test |
| > 100,000 | 90/5/5 or 98/1/1 | Even 1% is a large test set |

---

## 7. Time Series Splitting

For time-ordered data, **random shuffling breaks temporal order** and causes data leakage (future data leaking into training).

```python
from sklearn.model_selection import TimeSeriesSplit

X = np.arange(20).reshape(10, 2)
y = np.arange(10)

# TimeSeriesSplit respects temporal order
tscv = TimeSeriesSplit(n_splits=5)

for fold, (train_idx, test_idx) in enumerate(tscv.split(X)):
    print(f"Fold {fold}: Train={train_idx}, Test={test_idx}")

# Fold 0: Train=[0 1 2 3 4], Test=[5 6]
# Fold 1: Train=[0 1 2 3 4 5 6], Test=[7 8]
# ...training set grows, test always follows chronologically
```

### Manual Time-Based Split

```python
import pandas as pd

df = pd.DataFrame({
    'date': pd.date_range('2020-01-01', periods=1000, freq='D'),
    'feature': np.random.randn(1000),
    'target': np.random.randint(0, 2, 1000)
})

# Split by date — no shuffle!
cutoff = pd.Timestamp('2022-06-01')
train = df[df['date'] < cutoff]
test  = df[df['date'] >= cutoff]

# ❌ WRONG for time series
# train_test_split(df, shuffle=True)  # shuffling leaks future data
```

---

## 8. Data Leakage Prevention

Data leakage = information from test/validation data influencing model training.

### Common Sources of Leakage

```python
# ❌ LEAKAGE: Scaling before splitting
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)        # fit on ALL data (includes test!)
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y)

# ✅ CORRECT: Split first, then scale
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)   # fit on train only
X_test_scaled  = scaler.transform(X_test)         # transform test
```

### Leakage Checklist

| Step | Risk | Solution |
|---|---|---|
| Feature engineering before split | High | Split first, then engineer |
| Scaling/encoding on full dataset | High | `fit` on train only |
| Using future data in time series | High | Use `TimeSeriesSplit` |
| Target leakage (feature derived from target) | Critical | Remove such features |
| Duplicate rows across splits | Medium | Deduplicate before splitting |

---

## 9. Splitting with Pandas DataFrames

```python
import pandas as pd

df = pd.DataFrame({
    'feature1': [1, 2, 3, 4, 5, 6, 7, 8],
    'feature2': [10, 20, 30, 40, 50, 60, 70, 80],
    'target': [0, 1, 0, 1, 0, 1, 0, 1]
})

X = df[['feature1', 'feature2']]
y = df['target']

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, random_state=42, stratify=y
)

# DataFrame indices are preserved
print(X_train.index.tolist())  # original row indices
print(type(X_train))           # <class 'pandas.core.frame.DataFrame'>
```

---

## 10. Multiple Arrays and DataFrames

```python
# Split multiple arrays together (keeps correspondence)
X_train, X_test, y_train, y_test, idx_train, idx_test = train_test_split(
    X, y, np.arange(len(X)), test_size=0.2, random_state=42
)
# idx_train/idx_test tell you which original rows ended up where
```

---

## 📋 Summary Table

| Concept | Key Point |
|---|---|
| `test_size=0.2` | 20% for test, 80% for train |
| `random_state=42` | Ensures reproducible split |
| `stratify=y` | Preserves class proportions |
| `shuffle=False` | Required for time series data |
| Three-way split | Train → tune hyperparameters; Val → select model; Test → final eval |
| Data leakage | Always split BEFORE any preprocessing |
| `TimeSeriesSplit` | Respects temporal ordering |

---

## ❓ Revision Questions

1. **Why is it important to set `random_state` when using `train_test_split`?**
2. **What does the `stratify` parameter do and when should you use it?**
3. **Describe how to perform a three-way split (train/validation/test) using `train_test_split`.**
4. **Why should you NOT shuffle time series data when splitting?**
5. **Give an example of data leakage caused by preprocessing before splitting.**
6. **How would you choose the right split ratio for a dataset with 500 samples vs. one with 1 million samples?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← 02 - Data Preprocessing](02-data-preprocessing.md) | [Unit 5: Scikit-learn Basics](../README.md) | [04 - Cross-Validation →](04-cross-validation.md) |
