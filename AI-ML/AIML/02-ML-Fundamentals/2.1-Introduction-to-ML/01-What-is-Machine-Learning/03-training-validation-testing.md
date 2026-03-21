# 📘 Chapter 3: Training, Validation & Testing Splits

> **Unit 2.1 – Introduction to Machine Learning** | File 3 of 7

---

## 📋 Overview

Every ML model must be evaluated on data it has **never seen during training**. This chapter
explains the three fundamental data splits — training, validation, and testing — why each
exists, how to choose split ratios, and how to avoid the critical pitfall of **data leakage**.

---

## 1. Purpose of Each Split

### 1.1 Training Set

- **What:** The largest portion of data used to **fit** the model parameters (weights, biases).
- **Role:** The model sees these examples repeatedly during training and adjusts θ to minimize loss.
- **Analogy:** The textbook a student reads and practices from.

### 1.2 Validation Set

- **What:** A held-out portion used to **tune hyperparameters** and make architecture decisions.
- **Role:** Evaluate performance during development without touching the test set. Used for
  early stopping, learning rate selection, model comparison.
- **Analogy:** Practice exams the student takes to gauge readiness.

### 1.3 Test Set

- **What:** A completely held-out portion used for **final, unbiased evaluation**.
- **Role:** Touched **only once** after all tuning is done. Provides the true generalization estimate.
- **Analogy:** The final exam — taken only once, no do-overs.

```
  ┌────────────────────────────────────────────────────────────┐
  │                    FULL DATASET                            │
  │                                                            │
  │  ┌──────────────────────┬──────────┬───────────┐           │
  │  │    TRAINING SET      │VALIDATION│ TEST SET  │           │
  │  │   (learn params)     │(tune HP) │ (final    │           │
  │  │                      │          │  eval)    │           │
  │  │        60-80%        │ 10-20%   │  10-20%   │           │
  │  └──────────────────────┴──────────┴───────────┘           │
  │                                                            │
  │  ──────────────── Information Flow ──────────────────      │
  │                                                            │
  │  Training ──▶ Model ──▶ Validate ──▶ Tune ──▶ Test        │
  │  (iterate)              (iterate)              (once!)     │
  └────────────────────────────────────────────────────────────┘
```

---

## 2. Typical Split Ratios

| Ratio (Train/Val/Test) | When to Use | Dataset Size Guideline |
|------------------------|-------------|----------------------|
| **60 / 20 / 20** | Default for moderate datasets | 1K – 100K samples |
| **70 / 15 / 15** | Balanced choice, slightly more training | 10K – 500K samples |
| **80 / 10 / 10** | Large datasets where 10% still gives reliable estimates | 100K+ samples |
| **90 / 5 / 5** | Very large datasets (millions of rows) | 1M+ samples |
| **No val set (K-Fold)** | Small datasets where every sample matters | < 5K samples |

**Key insight:** As dataset size grows, you can afford to allocate a smaller **percentage**
to validation/test because even a small fraction yields enough samples for reliable estimates.

### Example Calculation

For 50,000 samples at 70/15/15: Training = 35,000 | Validation = 7,500 | Testing = 7,500

---

## 3. The Holdout Method

The simplest strategy — **randomly** divide data into fixed partitions.

**Process:** Shuffle → Split into train/val/test → Train on training set only →
Evaluate on validation → Tune hyperparameters → Repeat → Final evaluation on test (ONCE).

**Pros:** Simple, fast, works well with large datasets.
**Cons:** Results depend on the particular random split; wasteful for small datasets.

---

## 4. Cross-Validation vs Holdout

### 4.1 K-Fold Cross-Validation

When data is **limited**, a single holdout split may not be representative.
K-Fold uses all data for both training and validation by rotating the folds.

```
  K = 5 Fold Cross-Validation:

  Fold 1:  [VAL ][TRAIN][TRAIN][TRAIN][TRAIN]
  Fold 2:  [TRAIN][VAL ][TRAIN][TRAIN][TRAIN]
  Fold 3:  [TRAIN][TRAIN][VAL ][TRAIN][TRAIN]
  Fold 4:  [TRAIN][TRAIN][TRAIN][VAL ][TRAIN]
  Fold 5:  [TRAIN][TRAIN][TRAIN][TRAIN][VAL ]

  Final score = average of 5 fold scores
  Score variance tells you how stable the model is
```

**Performance estimate:** `CV Score = (1/K) Σₖ Score(fold_k)` with `SE = σ / √K`

### 4.2 When to Use Which?

| Criteria | Holdout | K-Fold Cross-Validation |
|----------|---------|------------------------|
| **Dataset size** | Large (>50K) | Small to moderate (<50K) |
| **Speed** | Fast (train once) | Slow (train K times) |
| **Variance** | Higher (single split) | Lower (averaged over K folds) |
| **Use case** | Deep learning, large-scale | Classical ML, model selection |
| **Typical K** | — | 5 or 10 |

---

## 5. Data Leakage

### 5.1 What Is Data Leakage?

Data leakage occurs when **information from outside the training set** (validation or test)
**leaks into the model** during training, producing overly optimistic performance estimates
that **do not generalize** to real-world data.

### 5.2 Common Causes

```
  ┌───────────────────────────────────────────────────────────┐
  │              COMMON CAUSES OF DATA LEAKAGE                │
  │                                                           │
  │  1. Scaling/normalizing BEFORE splitting                  │
  │     → Test statistics leak into training                  │
  │                                                           │
  │  2. Feature selection using the ENTIRE dataset            │
  │     → Model "knows" which features help on test data      │
  │                                                           │
  │  3. Duplicate samples across splits                       │
  │     → Model memorizes test examples                       │
  │                                                           │
  │  4. Time-series data split randomly (not chronologically) │
  │     → Future information leaks into the past              │
  │                                                           │
  │  5. Target leakage: using features derived from target    │
  │     → Unrealistically perfect predictions                 │
  └───────────────────────────────────────────────────────────┘
```

### 5.3 Prevention Strategies

| Cause | Prevention |
|-------|------------|
| Pre-split preprocessing | **Always split first**, then fit scalers on training set only |
| Feature selection on full data | Perform feature selection **within** training folds only |
| Duplicate records | Deduplicate before splitting; use `group` parameter in splits |
| Time-series random split | Use **chronological** splits (past → train, future → test) |
| Target leakage | Audit features — remove any that wouldn't be available at prediction time |

**Golden rule:** *"Nothing from the future should influence the past."*

---

## 6. Python Code — Splitting with scikit-learn

```python
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

X, y = load_breast_cancer(return_X_y=True)

# Step 1: Split FIRST — separate test set (held out until final evaluation)
X_temp, X_test, y_temp, y_test = train_test_split(X, y, test_size=0.15, random_state=42, stratify=y)

# Step 2: Split remaining into train and validation (0.18 × 0.85 ≈ 0.15 → ratio ~ 70/15/15)
X_train, X_val, y_train, y_val = train_test_split(X_temp, y_temp, test_size=0.18, random_state=42, stratify=y_temp)

# Step 3: Fit scaler on TRAINING data only (prevents leakage!)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)   # fit + transform
X_val_s   = scaler.transform(X_val)         # transform only
X_test_s  = scaler.transform(X_test)        # transform only

# Step 4: Train and evaluate
model = LogisticRegression(max_iter=10000, random_state=42)
model.fit(X_train_s, y_train)

print(f"Validation Accuracy: {accuracy_score(y_val, model.predict(X_val_s)):.2%}")
print(f"Test Accuracy:       {accuracy_score(y_test, model.predict(X_test_s)):.2%}")

# Bonus: K-Fold Cross-Validation
cv_scores = cross_val_score(model, X_train_s, y_train, cv=5, scoring='accuracy')
print(f"5-Fold CV Accuracy:  {cv_scores.mean():.2%} ± {cv_scores.std():.2%}")
```

**Key takeaway:** Notice that `fit_transform` is called only on training data. The
validation and test sets use `transform` only — this prevents data leakage.

---

## 7. Summary Table

| Aspect | Training Set | Validation Set | Test Set |
|--------|-------------|---------------|----------|
| **Purpose** | Learn model parameters | Tune hyperparameters | Final unbiased evaluation |
| **Typical %** | 60–80% | 10–20% | 10–20% |
| **Touched during dev?** | Yes (repeatedly) | Yes (repeatedly) | **No** (only once at end) |
| **Preprocessing** | fit_transform | transform only | transform only |
| **Overfitting risk** | High if too complex | Moderate (indirect) | None if used correctly |
| **Alternative** | — | K-Fold CV | Separate holdout |

---

## ✅ Revision Questions

1. **Explain the purpose** of each of the three data splits. Why can't we just
   train and test on the same data?

2. **A dataset has 2,000 samples.** Which split strategy would you recommend —
   holdout (70/15/15) or 5-fold cross-validation? Justify your choice.

3. **What is data leakage?** Give two concrete examples of how it can occur
   and explain how to prevent each.

4. **In the Python code above**, why do we call `fit_transform()` on the training
   set but only `transform()` on validation and test sets?

5. **You are building a stock price predictor.** Should you split the data randomly
   or chronologically? Explain the consequences of choosing wrong.

6. **A colleague reports 99.5% test accuracy** on a fraud detection task. The model
   uses `transaction_flagged_as_fraud` as a feature. What is wrong, and which type
   of data leakage is this?

---

## 🧭 Navigation

| Previous | Next |
|----------|------|
| [← 02 – Supervised, Unsupervised & Reinforcement](02-supervised-unsupervised-reinforcement.md) | [04 – Overfitting and Underfitting →](04-overfitting-and-underfitting.md) |

[🔼 Back to Unit 2.1 Overview](../README.md)
