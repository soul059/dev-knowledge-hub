# 🎭 Multi-Class SVM & Practical Tips

> **Unit 9, Chapter 7 — OvR, OvO, Class Imbalance, Probability Calibration & Complete Pipeline**

---

## 📋 Table of Contents

1. [Chapter Overview](#1-chapter-overview)
2. [One-vs-Rest (OvR) Strategy](#2-one-vs-rest-ovr-strategy)
3. [One-vs-One (OvO) Strategy](#3-one-vs-one-ovo-strategy)
4. [Computational Comparison](#4-computational-comparison)
5. [sklearn decision_function_shape](#5-sklearn-decision_function_shape)
6. [Practical Considerations](#6-practical-considerations)
7. [Class Imbalance Handling](#7-class-imbalance-handling)
8. [Probability Calibration — Platt Scaling](#8-probability-calibration--platt-scaling)
9. [SVM Practical Tips](#9-svm-practical-tips)
10. [Complete Pipeline Example](#10-complete-pipeline-example)
11. [Applications](#11-applications)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)

---

## 1. Chapter Overview

SVM is inherently a **binary classifier**. To handle multi-class problems (3+ classes), we need decomposition strategies that combine multiple binary SVMs:

```
Multi-class Problem (K classes)
    │
    ├── One-vs-Rest (OvR / OvA)
    │   └── Train K binary classifiers
    │       Class k vs "all other classes"
    │
    └── One-vs-One (OvO)
        └── Train K(K-1)/2 binary classifiers
            Every pair of classes
```

---

## 2. One-vs-Rest (OvR) Strategy

### Also Called: One-vs-All (OvA)

### How It Works

For K classes, train **K binary classifiers**:

```
Classifier 1: Class 1 vs {Class 2, 3, ..., K}    → f₁(x)
Classifier 2: Class 2 vs {Class 1, 3, ..., K}    → f₂(x)
Classifier 3: Class 3 vs {Class 1, 2, ..., K}    → f₃(x)
    ...
Classifier K: Class K vs {Class 1, 2, ..., K-1}  → fₖ(x)
```

### Prediction Rule

Assign to the class whose classifier gives the **highest confidence score**:

```
ŷ(x) = argmax  fₖ(x)
         k

where fₖ(x) = wₖ · x + bₖ  (decision function of classifier k)
```

### Example: 4-Class Problem

```
Classes: Cat, Dog, Bird, Fish

Classifier 1: Cat  vs {Dog, Bird, Fish}
Classifier 2: Dog  vs {Cat, Bird, Fish}
Classifier 3: Bird vs {Cat, Dog, Fish}
Classifier 4: Fish vs {Cat, Dog, Bird}

For a test image x:
f₁(x) = +2.3  (Cat score)
f₂(x) = -0.5  (Dog score)
f₃(x) = +1.1  (Bird score)
f₄(x) = -1.8  (Fish score)

Prediction: Cat (highest score = 2.3)
```

### ASCII Diagram: OvR

```
TRAINING (4 classes: A, B, C, D):

Classifier 1:  A vs Rest         Classifier 2:  B vs Rest
┌──────────────────┐             ┌──────────────────┐
│  A A A  │ B C D  │             │  B B B  │ A C D  │
│  A A    │ B D C  │             │  B B    │ A D C  │
│  A A A  │ C D B  │             │  B B B  │ C D A  │
│   (+1)  │  (-1)  │             │   (+1)  │  (-1)  │
└──────────────────┘             └──────────────────┘

Classifier 3:  C vs Rest         Classifier 4:  D vs Rest
┌──────────────────┐             ┌──────────────────┐
│  C C C  │ A B D  │             │  D D D  │ A B C  │
│  C C    │ A B D  │             │  D D    │ A B C  │
│  C C C  │ B A D  │             │  D D D  │ B A C  │
│   (+1)  │  (-1)  │             │   (+1)  │  (-1)  │
└──────────────────┘             └──────────────────┘

PREDICTION for new point x*:
f₁(x*) = +0.8   f₂(x*) = -1.2   f₃(x*) = +2.1   f₄(x*) = -0.3
                                     ↑
                              HIGHEST → Predict Class C
```

### Advantages & Disadvantages

```
✅ ADVANTAGES:
├── Simple to implement
├── Each classifier sees ALL training data
├── Only K classifiers needed
├── Interpretable (one w vector per class)
└── Works well when classes are well-separated

❌ DISADVANTAGES:
├── Imbalanced training sets (1 class vs K-1 classes)
├── Ambiguous regions possible (no classifier claims the point)
├── Scales poorly in the number of classes for some algorithms
└── Each classifier trained on full dataset (can be slow)
```

---

## 3. One-vs-One (OvO) Strategy

### How It Works

For K classes, train **K(K-1)/2 binary classifiers** — one for each pair of classes:

```
For K=4 classes {A, B, C, D}: Train 4×3/2 = 6 classifiers

Classifier 1: A vs B    → winner?
Classifier 2: A vs C    → winner?
Classifier 3: A vs D    → winner?
Classifier 4: B vs C    → winner?
Classifier 5: B vs D    → winner?
Classifier 6: C vs D    → winner?
```

### Prediction Rule: Voting

Each classifier casts a **vote** for one of its two classes. The class with the **most votes** wins:

```
For test point x*:

Classifier A-vs-B: Predicts A → Vote for A
Classifier A-vs-C: Predicts C → Vote for C
Classifier A-vs-D: Predicts A → Vote for A
Classifier B-vs-C: Predicts C → Vote for C
Classifier B-vs-D: Predicts D → Vote for D
Classifier C-vs-D: Predicts C → Vote for C

Vote count: A=2, B=0, C=3, D=1
Prediction: Class C (most votes) ✓
```

### ASCII Diagram: OvO

```
TRAINING (4 classes): 6 pairwise classifiers

  A vs B          A vs C          A vs D
┌─────────┐     ┌─────────┐     ┌─────────┐
│ A A│B B │     │ A A│C C │     │ A A│D D │
│ A  │B B │     │ A  │C C │     │ A  │D D │
│ +1 │ -1 │     │ +1 │ -1 │     │ +1 │ -1 │
└─────────┘     └─────────┘     └─────────┘

  B vs C          B vs D          C vs D
┌─────────┐     ┌─────────┐     ┌─────────┐
│ B B│C C │     │ B B│D D │     │ C C│D D │
│ B  │C C │     │ B  │D D │     │ C  │D D │
│ +1 │ -1 │     │ +1 │ -1 │     │ +1 │ -1 │
└─────────┘     └─────────┘     └─────────┘

Each classifier only uses data from its two classes!
(Much smaller training sets, but more classifiers)
```

### Advantages & Disadvantages

```
✅ ADVANTAGES:
├── Each classifier uses only 2 classes of data (SMALL training sets)
├── Training is FAST for each individual classifier
├── Naturally balanced (each classifier sees equal-ish classes)
├── Well-suited for SVM (SVM is inherently binary)
└── Default strategy for sklearn SVC

❌ DISADVANTAGES:
├── Many classifiers: K(K-1)/2 (grows quadratically)
├── Voting can produce ties
├── Each classifier doesn't see the "big picture" (only 2 classes)
├── Memory: must store K(K-1)/2 models
└── Prediction slower (must evaluate all classifiers)
```

---

## 4. Computational Comparison

### Training Complexity

```
Given: N total samples, K classes, ~N/K samples per class
SVM training complexity for n samples: approximately O(n²) to O(n³)

ONE-vs-REST (K classifiers):
├── Each classifier: trained on ALL N samples → O(N²) each
├── Total: K × O(N²) = O(K × N²)
└── For K=10, N=10000: 10 × 10000² = 10⁹

ONE-vs-ONE (K(K-1)/2 classifiers):
├── Each classifier: trained on ~2N/K samples → O((2N/K)²) each
├── Total: K(K-1)/2 × O((2N/K)²) = O(K(K-1)/2 × 4N²/K²) = O(2N²(K-1)/K)
└── For K=10, N=10000: 45 × (2000)² = 1.8 × 10⁸
```

### Comparison Table

| Metric | OvR | OvO |
|--------|-----|-----|
| **# Classifiers** | K | K(K-1)/2 |
| **Training data per classifier** | N (all) | ~2N/K (two classes) |
| **Training time** | O(K·N²) | O(N²·(K-1)/K) |
| **Prediction time** | O(K) | O(K²) |
| **Memory** | K models | K(K-1)/2 models |
| **Balanced training?** | ❌ (1 vs K-1) | ✅ (1 vs 1) |
| **sklearn default for SVC** | ❌ | ✅ |
| **sklearn default for LinearSVC** | ✅ | ❌ |

### When to Prefer Each

```
Prefer OvR when:
├── K is large (fewer classifiers)
├── Using linear models (fast training on full data)
├── Want interpretable class-specific weights
└── Using LinearSVC, SGDClassifier

Prefer OvO when:
├── Using kernel SVM (training on subsets is much faster)
├── K is moderate (K < 20)
├── Want naturally balanced classifiers
└── Using SVC (sklearn default)
```

### Scaling: Number of Classifiers

| K (classes) | OvR (K) | OvO (K(K-1)/2) |
|-------------|---------|-----------------|
| 3 | 3 | 3 |
| 5 | 5 | 10 |
| 10 | 10 | 45 |
| 20 | 20 | 190 |
| 50 | 50 | 1,225 |
| 100 | 100 | 4,950 |
| 1000 | 1,000 | 499,500 |

---

## 5. sklearn decision_function_shape

### The Parameter

```python
from sklearn.svm import SVC

# ── OvO (default) with OvR-shaped decision function ──
svm = SVC(kernel='rbf', decision_function_shape='ovr')
# Trains OvO internally but transforms output to OvR shape
# decision_function returns K scores (one per class)

# ── OvO with native decision function ──
svm = SVC(kernel='rbf', decision_function_shape='ovo')
# decision_function returns K(K-1)/2 scores (one per pair)
```

### Important Notes

```
┌────────────────────────────────────────────────────────────┐
│  sklearn SVC ALWAYS trains using OvO internally!           │
│                                                            │
│  decision_function_shape only affects the OUTPUT FORMAT    │
│  of .decision_function(), not the training process.        │
│                                                            │
│  'ovr' (default since sklearn 0.19):                       │
│  ├── Transforms OvO scores into OvR-style K scores         │
│  ├── decision_function(X) → shape (n_samples, K)           │
│  └── Each column = confidence for one class vs rest        │
│                                                            │
│  'ovo':                                                    │
│  ├── Returns raw OvO pairwise scores                       │
│  ├── decision_function(X) → shape (n_samples, K(K-1)/2)   │
│  └── Each column = score for one pair of classes           │
│                                                            │
│  For LinearSVC:                                            │
│  ├── Uses OvR by default (multi_class='ovr')               │
│  ├── Can switch to Crammer-Singer (multi_class='crammer_singer')│
│  └── OvR is usually preferred for LinearSVC               │
└────────────────────────────────────────────────────────────┘
```

### Code Example

```python
from sklearn.svm import SVC
from sklearn.datasets import load_iris
import numpy as np

X, y = load_iris(return_X_y=True)

# OvR-shaped output
svm_ovr = SVC(kernel='rbf', decision_function_shape='ovr')
svm_ovr.fit(X, y)
scores_ovr = svm_ovr.decision_function(X[:3])
print(f"OvR shape: {scores_ovr.shape}")  # (3, 3) — 3 scores per sample
print(f"OvR scores:\n{scores_ovr}")

# OvO-shaped output
svm_ovo = SVC(kernel='rbf', decision_function_shape='ovo')
svm_ovo.fit(X, y)
scores_ovo = svm_ovo.decision_function(X[:3])
print(f"\nOvO shape: {scores_ovo.shape}")  # (3, 3) — 3(3-1)/2 = 3 scores
print(f"OvO scores:\n{scores_ovo}")

# Both give the same predictions!
print(f"\nOvR predictions: {svm_ovr.predict(X[:5])}")
print(f"OvO predictions: {svm_ovo.predict(X[:5])}")
```

---

## 6. Practical Considerations

### Feature Scaling — CRITICAL

```
⚠️  THE #1 MISTAKE WITH SVM: Forgetting to scale features!

SVM uses distances (dot products, norms). Unscaled features cause:
├── Features with large ranges dominate
├── Kernel computation is biased
├── Optimization is slow (ill-conditioned)
└── Results are unreliable

ALWAYS scale with StandardScaler or MinMaxScaler!

# CORRECT:
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC())
])

# WRONG:
svm = SVC()
svm.fit(X_unscaled, y)  # ← Don't do this!
```

### When NOT to Use SVM

```
Don't use SVM when:
├── Dataset is very large (N > 100,000) → too slow (use LinearSVC or SGD)
├── Need probability estimates natively → use Logistic Regression
├── Need feature importance → use Random Forest or Linear models
├── Data is very high-dimensional AND sparse → LinearSVC is better
├── Online learning needed → use SGDClassifier with hinge loss
└── Real-time prediction with many support vectors → too slow
```

### SVM Alternatives for Large Data

| Dataset Size | Recommended | Why |
|-------------|-------------|-----|
| N < 10,000 | `SVC` (kernel) | Full kernel SVM, best accuracy |
| 10K < N < 100K | `LinearSVC` | Uses primal, O(N) training |
| N > 100K | `SGDClassifier(loss='hinge')` | Stochastic, O(1) per sample |
| Very large + nonlinear | `Nystroem + LinearSVC` | Approximates kernel |

```python
from sklearn.svm import SVC, LinearSVC
from sklearn.linear_model import SGDClassifier
from sklearn.kernel_approximation import Nystroem
from sklearn.pipeline import Pipeline

# For large nonlinear problems:
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('nystroem', Nystroem(kernel='rbf', gamma=0.1, n_components=300)),
    ('linear_svc', LinearSVC(C=1.0))
])
```

---

## 7. Class Imbalance Handling

### The Problem

```
Imbalanced dataset:
Class 0 (Healthy):  950 samples  (95%)
Class 1 (Disease):   50 samples  ( 5%)

Default SVM: Optimizes overall accuracy
├── Predicts "Healthy" for everything → 95% accuracy!
├── But misses ALL disease cases (0% recall for Class 1)
└── Terrible for medical diagnosis!
```

### Solution: class_weight Parameter

```python
from sklearn.svm import SVC

# ── Option 1: Balanced weights (automatic) ──
svm = SVC(kernel='rbf', class_weight='balanced')
# Weight for class k = N / (K × nₖ)
# Class 0: 1000 / (2 × 950) = 0.526
# Class 1: 1000 / (2 × 50)  = 10.0
# → Misclassifying Class 1 is penalized 19× more!

# ── Option 2: Custom weights ──
svm = SVC(kernel='rbf', class_weight={0: 1, 1: 20})
# Manually set: misclassifying Class 1 costs 20× more

# ── Option 3: Sample weights ──
svm = SVC(kernel='rbf')
sample_weights = np.where(y == 1, 10, 1)
svm.fit(X, y, sample_weight=sample_weights)
```

### How class_weight Works

```
Standard SVM:
    minimize ½‖w‖² + C Σᵢ ξᵢ

With class weights wₖ:
    minimize ½‖w‖² + Σₖ wₖ·C Σᵢ∈class_k ξᵢ

Effectively: different C for each class
├── Cₖ = wₖ × C
├── Higher weight → higher penalty for misclassifying that class
├── 'balanced': wₖ = N / (K × nₖ)
└── Minorities get higher C → harder to misclassify
```

### Other Imbalance Strategies

```
1. CLASS WEIGHTS (built into SVM)
   └── class_weight='balanced' or custom dict

2. RESAMPLING (before training)
   ├── Oversampling minority: SMOTE, RandomOverSampler
   └── Undersampling majority: RandomUnderSampler

3. THRESHOLD ADJUSTMENT (after training)
   ├── Use probability=True
   ├── Get P(y=1|x) via predict_proba
   └── Adjust threshold (e.g., classify as 1 if P > 0.3)

4. COST-SENSITIVE LEARNING
   └── Encode misclassification costs directly

5. ENSEMBLE METHODS
   └── BalancedBaggingClassifier with SVM base estimator
```

---

## 8. Probability Calibration — Platt Scaling

### The Problem: SVM Outputs Aren't Probabilities

```
SVM decision_function gives:
f(x) = w · x + b → any real number

This is NOT a probability!
f(x) = 2.5 doesn't mean "95% confident"
f(x) = 0.1 doesn't mean "51% confident"

But sometimes we NEED probabilities:
├── Medical diagnosis: "80% likely malignant"
├── Risk assessment: "35% default probability"
├── Ranking: Sort by confidence
└── Ensemble methods: Combine with other models
```

### Platt Scaling (1999)

Platt scaling fits a **sigmoid function** to the SVM decision values to convert them to probabilities:

```
                      1
P(y=1|x) = ─────────────────────
            1 + exp(A·f(x) + B)

where:
  f(x) = SVM decision function value
  A, B = parameters fit by maximum likelihood on a validation set
```

### How It Works

```
Step 1: Train SVM normally → get f(xᵢ) for all training points
Step 2: Fit sigmoid to (f(xᵢ), yᵢ) using cross-validation:
        P(y=1|f) = 1 / (1 + exp(A·f + B))
        Find A, B by minimizing cross-entropy loss
Step 3: For new point x:
        ├── Compute f(x) from SVM
        └── Apply sigmoid: P = 1 / (1 + exp(A·f(x) + B))

    P(y=1|x) ↑
         1.0  │                    ╱────────
              │                   ╱
         0.5  │- - - - - - - -  -╱- - - - - -
              │                 ╱
         0.0  │────────────────╱
              └──────────────────────────→ f(x)
                    -3  -2  -1  0  1  2  3
                         ↑
                    Decision boundary
                    (f(x) = 0 → P ≈ 0.5)
```

### sklearn Implementation

```python
from sklearn.svm import SVC
from sklearn.calibration import CalibratedClassifierCV

# ── Method 1: Built-in (recommended) ──
svm = SVC(kernel='rbf', probability=True)
# probability=True enables Platt scaling
# Uses 5-fold CV internally to fit A, B

svm.fit(X_train, y_train)
probabilities = svm.predict_proba(X_test)  # Returns P(y=k|x) for each class
print(f"Probabilities shape: {probabilities.shape}")
print(f"Sample: {probabilities[0]}")  # e.g., [0.15, 0.75, 0.10]

# ── Method 2: Explicit calibration (more control) ──
base_svm = SVC(kernel='rbf')
calibrated_svm = CalibratedClassifierCV(base_svm, cv=5, method='sigmoid')
# method='sigmoid': Platt scaling
# method='isotonic': Isotonic regression (non-parametric)

calibrated_svm.fit(X_train, y_train)
probabilities = calibrated_svm.predict_proba(X_test)
```

### Important Considerations

```
⚠️  CAVEATS of probability=True:

1. SLOWER TRAINING
   └── Platt scaling needs internal cross-validation
       (adds ~20% training time)

2. NOT ALWAYS WELL-CALIBRATED
   └── SVM decision values can be poorly mapped to probabilities
       especially with small datasets

3. CHANGES predict() BEHAVIOR
   └── predict() uses argmax of probabilities (may differ from
       argmax of decision_function in multi-class)

4. WHEN TO SKIP probability=True:
   ├── Only need class labels (not probabilities)
   ├── Speed is critical
   └── Large datasets (already slow)

5. BETTER ALTERNATIVE for well-calibrated probabilities:
   └── Use CalibratedClassifierCV with isotonic regression
       for better calibration (at cost of more data)
```

---

## 9. SVM Practical Tips

### The Complete SVM Checklist

```
BEFORE TRAINING:
┌─────────────────────────────────────────────────────────┐
│ □ Scale features (StandardScaler or MinMaxScaler)       │
│ □ Handle missing values (SVM can't handle NaN)          │
│ □ Encode categorical variables (one-hot or ordinal)     │
│ □ Check class balance (use class_weight if needed)      │
│ □ Choose appropriate kernel (start with RBF)            │
│ □ Start with default hyperparameters                    │
└─────────────────────────────────────────────────────────┘

TRAINING:
┌─────────────────────────────────────────────────────────┐
│ □ Use Pipeline (prevents data leakage from scaling)     │
│ □ Use cross-validation (never tune on test set!)        │
│ □ Tune C first (most impactful parameter)               │
│ □ Tune γ next (for RBF kernel)                          │
│ □ Monitor support vector count                           │
│ □ Compare with simpler models as baseline               │
└─────────────────────────────────────────────────────────┘

AFTER TRAINING:
┌─────────────────────────────────────────────────────────┐
│ □ Check train vs test accuracy (overfitting?)           │
│ □ Examine support vector count vs total samples         │
│ □ Look at confusion matrix                              │
│ □ Try different kernels if RBF isn't satisfactory       │
│ □ Consider if SVM is the right algorithm                │
└─────────────────────────────────────────────────────────┘
```

### Hyperparameter Tuning Order

```
1. Kernel selection:
   └── linear → rbf → poly (increasing complexity)

2. For RBF kernel:
   ├── C:     search [10⁻³, 10³] on log scale
   └── gamma: search [10⁻⁴, 10²] on log scale

3. For Polynomial kernel:
   ├── C:      search [10⁻², 10²]
   ├── degree: try [2, 3, 4]
   └── coef0:  try [0, 1]

4. Use GridSearchCV or RandomizedSearchCV
```

### Common Pitfalls

```
❌ PITFALL 1: Not scaling features
   Fix: Always use StandardScaler in a Pipeline

❌ PITFALL 2: Tuning on test set
   Fix: Use cross-validation, keep test set untouched

❌ PITFALL 3: Using SVM on very large data
   Fix: Use LinearSVC or SGDClassifier for N > 50K

❌ PITFALL 4: Ignoring class imbalance
   Fix: Use class_weight='balanced'

❌ PITFALL 5: Too many features without regularization
   Fix: Consider feature selection or PCA before SVM

❌ PITFALL 6: Not checking support vector count
   Fix: If n_SVs ≈ N, model may be overfitting or C is too small
```

---

## 10. Complete Pipeline Example

### End-to-End SVM Classification Pipeline

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.svm import SVC
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.model_selection import (train_test_split, GridSearchCV,
                                     cross_val_score, learning_curve)
from sklearn.pipeline import Pipeline
from sklearn.metrics import (classification_report, confusion_matrix,
                              ConfusionMatrixDisplay, roc_auc_score)
from sklearn.datasets import load_wine
import warnings
warnings.filterwarnings('ignore')


# ═══════════════════════════════════════════════════
# STEP 1: Load and Explore Data
# ═══════════════════════════════════════════════════

data = load_wine()
X, y = data.data, data.target
feature_names = data.feature_names
class_names = data.target_names

print("=" * 50)
print("STEP 1: DATA EXPLORATION")
print("=" * 50)
print(f"Features: {X.shape[1]}")
print(f"Samples:  {X.shape[0]}")
print(f"Classes:  {len(class_names)} → {list(class_names)}")
print(f"Class distribution: {np.bincount(y)}")
print(f"Feature ranges:")
for i, name in enumerate(feature_names):
    print(f"  {name:>30}: [{X[:, i].min():.2f}, {X[:, i].max():.2f}]")


# ═══════════════════════════════════════════════════
# STEP 2: Train-Test Split
# ═══════════════════════════════════════════════════

print("\n" + "=" * 50)
print("STEP 2: TRAIN-TEST SPLIT")
print("=" * 50)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

print(f"Training set: {X_train.shape[0]} samples")
print(f"Test set:     {X_test.shape[0]} samples")
print(f"Train class dist: {np.bincount(y_train)}")
print(f"Test class dist:  {np.bincount(y_test)}")


# ═══════════════════════════════════════════════════
# STEP 3: Build Pipeline
# ═══════════════════════════════════════════════════

print("\n" + "=" * 50)
print("STEP 3: PIPELINE CONSTRUCTION")
print("=" * 50)

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC(random_state=42))
])

print("Pipeline steps:")
for name, step in pipeline.steps:
    print(f"  {name}: {step.__class__.__name__}")


# ═══════════════════════════════════════════════════
# STEP 4: Hyperparameter Tuning
# ═══════════════════════════════════════════════════

print("\n" + "=" * 50)
print("STEP 4: HYPERPARAMETER TUNING")
print("=" * 50)

param_grid = [
    # Linear kernel
    {
        'svm__kernel': ['linear'],
        'svm__C': [0.01, 0.1, 1, 10, 100],
    },
    # RBF kernel
    {
        'svm__kernel': ['rbf'],
        'svm__C': [0.1, 1, 10, 100],
        'svm__gamma': [0.001, 0.01, 0.1, 1, 'scale'],
    },
    # Polynomial kernel
    {
        'svm__kernel': ['poly'],
        'svm__C': [0.1, 1, 10],
        'svm__degree': [2, 3],
        'svm__coef0': [0, 1],
        'svm__gamma': ['scale'],
    },
]

grid_search = GridSearchCV(
    pipeline, param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
    return_train_score=True,
    verbose=0
)

grid_search.fit(X_train, y_train)

print(f"Best parameters: {grid_search.best_params_}")
print(f"Best CV accuracy: {grid_search.best_score_:.4f}")

# Top 5 configurations
results = grid_search.cv_results_
top_5_idx = np.argsort(results['mean_test_score'])[-5:][::-1]
print("\nTop 5 configurations:")
for rank, idx in enumerate(top_5_idx, 1):
    print(f"  {rank}. CV Acc = {results['mean_test_score'][idx]:.4f} "
          f"± {results['std_test_score'][idx]:.4f}")
    print(f"     Train Acc = {results['mean_train_score'][idx]:.4f}")
    print(f"     Params: {results['params'][idx]}")


# ═══════════════════════════════════════════════════
# STEP 5: Evaluate on Test Set
# ═══════════════════════════════════════════════════

print("\n" + "=" * 50)
print("STEP 5: TEST SET EVALUATION")
print("=" * 50)

best_model = grid_search.best_estimator_
y_pred = best_model.predict(X_test)

print(f"Test Accuracy: {best_model.score(X_test, y_test):.4f}")
print(f"\nClassification Report:")
print(classification_report(y_test, y_pred, target_names=class_names))


# ═══════════════════════════════════════════════════
# STEP 6: Model Analysis
# ═══════════════════════════════════════════════════

print("\n" + "=" * 50)
print("STEP 6: MODEL ANALYSIS")
print("=" * 50)

svm_model = best_model.named_steps['svm']
print(f"Kernel: {svm_model.kernel}")
print(f"Support vectors per class: {svm_model.n_support_}")
print(f"Total support vectors: {sum(svm_model.n_support_)} / {len(X_train)} "
      f"({100*sum(svm_model.n_support_)/len(X_train):.1f}%)")

if svm_model.kernel == 'linear':
    print(f"\nTop 5 most important features (by |weight|):")
    for class_idx in range(len(class_names)):
        weights = np.abs(svm_model.coef_[class_idx])
        top_features = np.argsort(weights)[-5:][::-1]
        print(f"  Class '{class_names[class_idx]}':")
        for fi in top_features:
            print(f"    {feature_names[fi]:>30}: {weights[fi]:.4f}")


# ═══════════════════════════════════════════════════
# STEP 7: Confusion Matrix Visualization
# ═══════════════════════════════════════════════════

fig, ax = plt.subplots(figsize=(8, 6))
cm = confusion_matrix(y_test, y_pred)
disp = ConfusionMatrixDisplay(cm, display_labels=class_names)
disp.plot(ax=ax, cmap='Blues', values_format='d')
plt.title('SVM Confusion Matrix')
plt.tight_layout()
plt.savefig('svm_confusion_matrix.png', dpi=150, bbox_inches='tight')
plt.show()


# ═══════════════════════════════════════════════════
# STEP 8: Learning Curve
# ═══════════════════════════════════════════════════

train_sizes, train_scores, val_scores = learning_curve(
    best_model, X_train, y_train,
    train_sizes=np.linspace(0.1, 1.0, 10),
    cv=5, scoring='accuracy', n_jobs=-1
)

fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(train_sizes, train_scores.mean(axis=1), 'o-', label='Training')
ax.fill_between(train_sizes,
                train_scores.mean(axis=1) - train_scores.std(axis=1),
                train_scores.mean(axis=1) + train_scores.std(axis=1), alpha=0.1)
ax.plot(train_sizes, val_scores.mean(axis=1), 's-', label='Validation')
ax.fill_between(train_sizes,
                val_scores.mean(axis=1) - val_scores.std(axis=1),
                val_scores.mean(axis=1) + val_scores.std(axis=1), alpha=0.1)
ax.set_xlabel('Training Set Size')
ax.set_ylabel('Accuracy')
ax.set_title('SVM Learning Curve')
ax.legend()
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('svm_learning_curve.png', dpi=150, bbox_inches='tight')
plt.show()


print("\n" + "=" * 50)
print("PIPELINE COMPLETE!")
print("=" * 50)
```

### Quick Reference: sklearn SVM Classes

```python
# ── CLASSIFICATION ──
from sklearn.svm import SVC         # Kernel SVM (OvO), N < 10K
from sklearn.svm import LinearSVC   # Linear SVM (OvR), N < 100K
from sklearn.svm import NuSVC       # SVM with ν parameter

# ── REGRESSION ──
from sklearn.svm import SVR         # Kernel SVR, N < 10K
from sklearn.svm import LinearSVR   # Linear SVR, N < 100K
from sklearn.svm import NuSVR       # SVR with ν parameter

# ── LARGE-SCALE ──
from sklearn.linear_model import SGDClassifier  # loss='hinge' for SVM
from sklearn.linear_model import SGDRegressor   # loss='epsilon_insensitive'

# ── KERNEL APPROXIMATION (for large nonlinear) ──
from sklearn.kernel_approximation import Nystroem
from sklearn.kernel_approximation import RBFSampler
```

---

## 11. Applications

### Multi-Class SVM in Practice

| Application | # Classes | Strategy | Kernel |
|-------------|-----------|----------|--------|
| **Handwritten digits** | 10 | OvO | RBF |
| **Document categorization** | 20+ | OvR | Linear |
| **Image classification** | 1000+ | OvR | Linear (with features) |
| **Gene classification** | 5-20 | OvO | Linear/RBF |
| **Sentiment analysis** | 3-5 | OvR | Linear |
| **Medical diagnosis** | 3-10 | OvO | RBF |

### SVM in the Deep Learning Era

```
SVM's role has evolved:

BEFORE Deep Learning (pre-2012):
├── SVM was THE go-to classifier
├── Dominated many competitions (MNIST, text classification)
└── Combined with hand-crafted features

AFTER Deep Learning (post-2012):
├── Deep learning replaced SVM for many tasks (images, NLP)
├── SVM still relevant for:
│   ├── Small datasets (N < 10K)
│   ├── Tabular data (no spatial structure)
│   ├── When interpretability matters
│   ├── As final classifier on deep features
│   └── When training compute is limited
└── LinearSVC is still competitive for text classification
```

---

## 12. Summary Table

| Concept | Key Point |
|---------|-----------|
| **OvR (One-vs-Rest)** | K classifiers, each class vs all others |
| **OvO (One-vs-One)** | K(K-1)/2 classifiers, each pair of classes |
| **OvR prediction** | argmax of decision functions |
| **OvO prediction** | Majority voting |
| **OvR training time** | O(K·N²) — each sees all data |
| **OvO training time** | O(N²·(K-1)/K) — smaller subsets |
| **sklearn SVC default** | OvO training, decision_function_shape='ovr' |
| **LinearSVC default** | OvR (uses primal, faster) |
| **class_weight** | 'balanced' or dict — handles imbalanced data |
| **Platt scaling** | Converts SVM scores to probabilities via sigmoid |
| **probability=True** | Enables predict_proba() in sklearn SVC |
| **Feature scaling** | ALWAYS required — use StandardScaler |
| **Large data** | Use LinearSVC (N<100K) or SGDClassifier (N>100K) |
| **Pipeline** | StandardScaler → SVC → GridSearchCV |

---

## 13. Revision Questions

### Q1: OvR vs OvO
**For a 10-class problem with 5000 samples per class, compare the number of classifiers and approximate training data per classifier for OvR and OvO strategies.**

<details>
<summary>Answer</summary>

**OvR**:
- Number of classifiers: K = **10**
- Training data per classifier: N = 50,000 (all samples)
  - Each classifier: 5,000 positive vs 45,000 negative (imbalanced 1:9 ratio)

**OvO**:
- Number of classifiers: K(K-1)/2 = 10×9/2 = **45**
- Training data per classifier: 2 × (N/K) = 2 × 5,000 = **10,000**
  - Each classifier: 5,000 vs 5,000 (perfectly balanced)

**Training time comparison**:
- OvR: 10 × (50,000)² = 2.5 × 10¹⁰ (proportional)
- OvO: 45 × (10,000)² = 4.5 × 10⁹ (proportional)
- OvO is about **5.6× faster** despite having more classifiers!

This is why sklearn SVC uses OvO by default.

</details>

### Q2: Class Imbalance
**You're building an SVM for fraud detection: 99% legitimate (class 0) and 1% fraudulent (class 1). How would you configure the SVM in sklearn?**

<details>
<summary>Answer</summary>

```python
from sklearn.svm import SVC
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC(
        kernel='rbf',
        class_weight='balanced',   # Auto-adjust: fraud gets 99× more weight
        probability=True,          # For threshold tuning
    ))
])

# Tune with appropriate metric (NOT accuracy!)
param_grid = {
    'svm__C': [0.1, 1, 10, 100],
    'svm__gamma': ['scale', 0.01, 0.1],
}

grid = GridSearchCV(
    pipe, param_grid, cv=5,
    scoring='f1',              # F1-score better for imbalanced data
    # Or: scoring='recall' to prioritize catching fraud
    n_jobs=-1
)

# After training, adjust threshold:
# probas = grid.predict_proba(X_test)[:, 1]
# y_pred = (probas > 0.3).astype(int)  # Lower threshold to catch more fraud
```

Key points:
1. `class_weight='balanced'` gives fraud cases 99× weight
2. Use F1-score or recall for evaluation (NOT accuracy)
3. Use `probability=True` to enable threshold adjustment
4. Consider lowering decision threshold below 0.5

</details>

### Q3: Platt Scaling
**What is Platt scaling and when should you enable probability=True in sklearn SVC? When should you avoid it?**

<details>
<summary>Answer</summary>

**Platt scaling** fits a sigmoid function P(y=1|x) = 1/(1+exp(A·f(x)+B)) to convert SVM's decision function values to probability estimates. Parameters A and B are fit via internal cross-validation.

**Enable probability=True when**:
- You need probability estimates (e.g., medical risk scores)
- You want to adjust the classification threshold
- Combining SVM with other models in an ensemble
- Need to rank predictions by confidence
- Using predict_proba() downstream

**Avoid probability=True when**:
- You only need class labels (predict() works without it)
- Training speed is critical (adds ~20% overhead from internal CV)
- Dataset is very small (internal CV may be unreliable)
- You're using SVM as a fast binary classifier
- The calibration isn't needed for your application

**Alternative**: Use `CalibratedClassifierCV` with `method='isotonic'` for potentially better calibration, especially when you have enough validation data.

</details>

### Q4: Scaling
**A data scientist trained an RBF SVM without scaling features and got 60% accuracy. After adding StandardScaler, accuracy jumped to 95%. Explain why.**

<details>
<summary>Answer</summary>

The RBF kernel K(x,y) = exp(-γ‖x-y‖²) depends on **Euclidean distances** between points. Without scaling:

1. **Distance domination**: If Feature A ranges [0, 100000] and Feature B ranges [0, 1], then ‖x-y‖² ≈ (diff_A)². Feature B becomes invisible.

2. **Effective dimensionality reduction**: The kernel essentially operates in 1D (only Feature A matters), losing all information from other features.

3. **γ mismatch**: The default γ = 1/(n_features × X.var()) uses overall variance, which is dominated by large-range features.

4. **Optimization issues**: The QP solver struggles with ill-conditioned problems (large condition numbers from unscaled features).

After StandardScaler (zero mean, unit variance):
- All features contribute equally to distances
- γ is meaningful across all dimensions
- The kernel can capture relationships in ALL features
- The optimizer converges properly

This is why **feature scaling is the #1 most important preprocessing step for SVM**.

</details>

### Q5: Pipeline Design
**Design a complete sklearn pipeline for multi-class text classification using SVM. Include preprocessing, feature extraction, and hyperparameter tuning.**

<details>
<summary>Answer</summary>

```python
from sklearn.pipeline import Pipeline
from sklearn.svm import LinearSVC
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.model_selection import GridSearchCV
from sklearn.datasets import fetch_20newsgroups

# Load text data
train = fetch_20newsgroups(subset='train', remove=('headers', 'footers'))
test = fetch_20newsgroups(subset='test', remove=('headers', 'footers'))

# Pipeline: TF-IDF → Linear SVM (OvR)
pipe = Pipeline([
    ('tfidf', TfidfVectorizer(
        max_features=50000,
        ngram_range=(1, 2),
        sublinear_tf=True,      # Apply log to TF
        stop_words='english'
    )),
    # No StandardScaler needed: TF-IDF already normalizes
    ('svm', LinearSVC(
        multi_class='ovr',       # OvR for multi-class
        class_weight='balanced', # Handle any class imbalance
        max_iter=5000,
    ))
])

param_grid = {
    'tfidf__max_features': [10000, 50000],
    'tfidf__ngram_range': [(1, 1), (1, 2)],
    'svm__C': [0.1, 1, 10],
}

grid = GridSearchCV(pipe, param_grid, cv=3,
                    scoring='f1_macro', n_jobs=-1, verbose=1)
grid.fit(train.data, train.target)

print(f"Best params: {grid.best_params_}")
print(f"Best CV F1: {grid.best_score_:.4f}")
print(f"Test accuracy: {grid.score(test.data, test.target):.4f}")
```

Key design choices:
- **LinearSVC** instead of SVC (text data is high-dimensional, linear works well)
- **TF-IDF** vectorizer (standard for text)
- **No scaler** needed (TF-IDF already normalizes)
- **OvR** strategy (standard for LinearSVC with many classes)
- **F1-macro** scoring (handles multi-class evaluation fairly)

</details>

### Q6: Decision Function Shape
**Explain the difference between decision_function_shape='ovr' and 'ovo' in sklearn SVC. Does this parameter affect training?**

<details>
<summary>Answer</summary>

**No, it does NOT affect training!** sklearn SVC **always** trains using OvO internally, regardless of this parameter.

`decision_function_shape` only changes the **output format** of `.decision_function()`:

- **'ovr'** (default): Transforms the K(K-1)/2 OvO scores into K scores (one per class). Each score represents "this class vs all others." Output shape: (n_samples, K).

- **'ovo'**: Returns the raw K(K-1)/2 pairwise scores directly. Output shape: (n_samples, K(K-1)/2).

**Both give identical predictions** via `.predict()`.

The 'ovr' shape is generally preferred because:
1. More intuitive (one score per class)
2. Easier to compare across classes
3. Compatible with more sklearn utilities
4. Default since sklearn 0.19

Note: `LinearSVC` is different — it actually **trains** using OvR (or Crammer-Singer) and doesn't use this parameter.

</details>

---

## Navigation

| Previous | Up | Next Unit |
|----------|-------|-----------|
| [← SVM Regression](./06-svm-regression.md) | [Unit 9: SVM](../) | [Unit 10 →](../../10-Dimensionality-Reduction/) |

---

*📝 Part of the AIML Study Notes — Unit 9: Support Vector Machines*
*🔖 Chapter 7 of 7 — Multi-Class SVM & Practical Tips*
