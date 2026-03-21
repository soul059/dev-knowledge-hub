# Chapter 8.7: Stacking (Stacked Generalization)

> **Unit 8: Ensemble Methods** — File 7 of 8

---

## Table of Contents

1. [Introduction — The Meta-Learning Concept](#1-introduction--the-meta-learning-concept)
2. [Base Learners and Meta-Learner](#2-base-learners-and-meta-learner)
3. [Cross-Validated Stacking (Avoiding Overfitting)](#3-cross-validated-stacking-avoiding-overfitting)
4. [Blending vs Stacking](#4-blending-vs-stacking)
5. [StackingClassifier in Scikit-Learn](#5-stackingclassifier-in-scikit-learn)
6. [Choosing Base and Meta Models](#6-choosing-base-and-meta-models)
7. [Multi-Level Stacking](#7-multi-level-stacking)
8. [Practical Example](#8-practical-example)
9. [Applications](#9-applications)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. Introduction — The Meta-Learning Concept

**Stacking** (Stacked Generalization), introduced by David Wolpert in 1992, is an ensemble technique that **learns how to combine multiple models** rather than using a fixed rule (like voting or averaging).

### The Key Insight

```
Simple Ensemble (Voting/Averaging):
  Model A: prediction_A
  Model B: prediction_B     → FIXED RULE: average(A, B, C)
  Model C: prediction_C

Stacking (Meta-Learning):
  Model A: prediction_A
  Model B: prediction_B     → LEARNED RULE: meta_model(A, B, C)
  Model C: prediction_C

The meta-model LEARNS the optimal way to combine base predictions!
Maybe Model A is great for certain regions of the input space,
and Model C is better for others — the meta-model captures this.
```

### Why Is This Powerful?

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Voting:    H(x) = w₁h₁(x) + w₂h₂(x) + w₃h₃(x)      │
│             Fixed weights, linear combination           │
│                                                         │
│  Stacking:  H(x) = g(h₁(x), h₂(x), h₃(x))            │
│             g is a learned function!                     │
│             Can capture non-linear interactions          │
│             between base model predictions              │
│                                                         │
│  Example: "When Model A says class 1 AND Model B says   │
│           class 0, trust Model C" — stacking can learn  │
│           rules like this automatically!                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Base Learners and Meta-Learner

### Architecture

```
                           Training Data
                           (X, y)
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
         ┌─────────┐    ┌─────────┐    ┌─────────┐
         │ Base     │    │ Base     │    │ Base     │
         │ Learner 1│    │ Learner 2│    │ Learner 3│
         │ (e.g. RF)│    │ (e.g.SVM)│    │(e.g. KNN)│
         └────┬─────┘    └────┬─────┘    └────┬─────┘
              │               │               │
              ▼               ▼               ▼
           Pred₁           Pred₂           Pred₃
              │               │               │
              └───────────────┼───────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │   Meta-Features   │
                    │ [Pred₁,Pred₂,Pred₃]│
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   Meta-Learner    │
                    │(e.g. Logistic Reg)│
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Final Prediction  │
                    └──────────────────┘

Level 0: Base learners (diverse models)
Level 1: Meta-learner (combines base predictions)
```

### Terminology

| Term | Meaning |
|------|---------|
| **Base learners** (Level 0) | The individual models that make initial predictions |
| **Meta-learner** (Level 1) | The model that learns to combine base predictions |
| **Meta-features** | The predictions from base learners, used as input to the meta-learner |
| **Stacked generalization** | The full framework of training base learners + meta-learner |

---

## 3. Cross-Validated Stacking (Avoiding Overfitting)

### The Overfitting Problem with Naive Stacking

```
NAIVE (WRONG) APPROACH:
  1. Train base models on FULL training data
  2. Get base model predictions on FULL training data
  3. Train meta-learner on these predictions
  
  Problem: Base models have already SEEN the training data!
           Their predictions on training data are OPTIMISTIC
           (they've memorized the training set)
           
           The meta-learner learns to combine memorized predictions
           → SEVERE OVERFITTING
```

### The Cross-Validated Solution

```
CORRECT APPROACH — K-Fold Cross-Validated Stacking:

Training Data: [Fold 1 | Fold 2 | Fold 3 | Fold 4 | Fold 5]

For base model h₁ (Random Forest):

  Iteration 1: Train on [2,3,4,5] → Predict on [1] → meta_features for fold 1
  Iteration 2: Train on [1,3,4,5] → Predict on [2] → meta_features for fold 2
  Iteration 3: Train on [1,2,4,5] → Predict on [3] → meta_features for fold 3
  Iteration 4: Train on [1,2,3,5] → Predict on [4] → meta_features for fold 4
  Iteration 5: Train on [1,2,3,4] → Predict on [5] → meta_features for fold 5

  Result: Out-of-fold predictions for ALL training samples
          (each prediction made by a model that NEVER saw that sample)

Repeat for each base model → Stack predictions into meta-feature matrix

Then train meta-learner on this UNBIASED meta-feature matrix
```

### Visual Walkthrough (3-Fold CV, 3 Base Models)

```
Step 1: Generate out-of-fold predictions for each base model
═══════════════════════════════════════════════════════════

Training Data: [──Fold 1──|──Fold 2──|──Fold 3──]

Base Model 1 (Random Forest):
  Train on F2+F3 → Predict F1:  [p₁₁, p₁₂, p₁₃]
  Train on F1+F3 → Predict F2:  [p₁₄, p₁₅, p₁₆]
  Train on F1+F2 → Predict F3:  [p₁₇, p₁₈, p₁₉]
  
  Full OOF predictions: [p₁₁, p₁₂, p₁₃, p₁₄, p₁₅, p₁₆, p₁₇, p₁₈, p₁₉]

Base Model 2 (SVM):  
  [p₂₁, p₂₂, p₂₃, p₂₄, p₂₅, p₂₆, p₂₇, p₂₈, p₂₉]

Base Model 3 (KNN):
  [p₃₁, p₃₂, p₃₃, p₃₄, p₃₅, p₃₆, p₃₇, p₃₈, p₃₉]


Step 2: Create meta-feature matrix
═══════════════════════════════════

Meta-features:
┌─────────┬─────────┬─────────┬─────────┐
│ Sample   │  RF_pred │ SVM_pred│ KNN_pred│
├─────────┼─────────┼─────────┼─────────┤
│    1     │   p₁₁   │   p₂₁  │   p₃₁  │
│    2     │   p₁₂   │   p₂₂  │   p₃₂  │
│   ...    │   ...   │   ...  │   ...  │
│    9     │   p₁₉   │   p₂₉  │   p₃₉  │
└─────────┴─────────┴─────────┴─────────┘

Step 3: Train meta-learner on meta-features + true labels
═════════════════════════════════════════════════════════

Meta-Learner (Logistic Regression):
  Input:  [RF_pred, SVM_pred, KNN_pred]
  Output: Final prediction
  
  Learns: "RF is best for these patterns, SVM for those..."
```

### At Prediction Time

```
New sample x_new:

  1. Each base model predicts (using models trained on FULL data):
     RF(x_new)  = 0.82
     SVM(x_new) = 0.75
     KNN(x_new) = 0.68

  2. Stack into meta-features: [0.82, 0.75, 0.68]

  3. Meta-learner predicts:
     Final = LogReg([0.82, 0.75, 0.68]) = 0.79

Note: For test predictions, base models are retrained on 
the FULL training data (not the CV folds)
```

---

## 4. Blending vs Stacking

### Blending (Simplified Stacking)

```
BLENDING:
  1. Split training data into TRAIN (80%) and HOLDOUT (20%)
  2. Train base models on TRAIN
  3. Get base model predictions on HOLDOUT
  4. Train meta-learner on HOLDOUT predictions

STACKING:
  1. Use K-fold CV to get out-of-fold predictions for ALL data
  2. Train meta-learner on ALL out-of-fold predictions

Comparison:
┌──────────────────┬──────────────────────┬─────────────────────┐
│    Aspect        │      Blending        │      Stacking       │
├──────────────────┼──────────────────────┼─────────────────────┤
│ Meta-learner     │ Trained on 20% of    │ Trained on 100% of  │
│ training data    │ the data             │ the data (OOF)      │
├──────────────────┼──────────────────────┼─────────────────────┤
│ Data efficiency  │ Less efficient       │ More efficient      │
│                  │ (wastes 20%)         │ (uses all data)     │
├──────────────────┼──────────────────────┼─────────────────────┤
│ Implementation   │ Simpler              │ More complex        │
├──────────────────┼──────────────────────┼─────────────────────┤
│ Leakage risk     │ Lower (holdout)      │ Possible if CV done │
│                  │                      │ incorrectly         │
├──────────────────┼──────────────────────┼─────────────────────┤
│ Computation      │ Trains base models   │ Trains base models  │
│                  │ once                 │ K times             │
├──────────────────┼──────────────────────┼─────────────────────┤
│ Typical use      │ Quick prototyping    │ Competition/         │
│                  │                      │ production          │
└──────────────────┴──────────────────────┴─────────────────────┘
```

### Blending Implementation

```python
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

# Split: 80% for base training, 20% for meta training
X_base, X_blend, y_base, y_blend = train_test_split(
    X_train, y_train, test_size=0.2, random_state=42
)

# Train base models on X_base
rf = RandomForestClassifier(n_estimators=100, random_state=42).fit(X_base, y_base)
svm = SVC(probability=True, random_state=42).fit(X_base, y_base)
knn = KNeighborsClassifier(n_neighbors=5).fit(X_base, y_base)

# Get predictions on blend set
blend_features = np.column_stack([
    rf.predict_proba(X_blend)[:, 1],
    svm.predict_proba(X_blend)[:, 1],
    knn.predict_proba(X_blend)[:, 1]
])

# Train meta-learner
meta = LogisticRegression().fit(blend_features, y_blend)

# Predict on test set
test_features = np.column_stack([
    rf.predict_proba(X_test)[:, 1],
    svm.predict_proba(X_test)[:, 1],
    knn.predict_proba(X_test)[:, 1]
])
y_pred = meta.predict(test_features)
print(f"Blending Accuracy: {accuracy_score(y_test, y_pred):.4f}")
```

---

## 5. StackingClassifier in Scikit-Learn

### Basic Usage

```python
from sklearn.ensemble import StackingClassifier, RandomForestClassifier
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import accuracy_score

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Define stacking ensemble
stacking_clf = StackingClassifier(
    estimators=[
        ('rf', RandomForestClassifier(n_estimators=100, random_state=42)),
        ('svm', SVC(probability=True, random_state=42)),
        ('knn', KNeighborsClassifier(n_neighbors=5)),
        ('gb', GradientBoostingClassifier(n_estimators=100, random_state=42))
    ],
    final_estimator=LogisticRegression(max_iter=1000),
    cv=5,                    # 5-fold CV for generating meta-features
    stack_method='auto',     # 'predict_proba' if available, else 'predict'
    passthrough=False,       # don't pass original features to meta-learner
    n_jobs=-1
)

stacking_clf.fit(X_train, y_train)
y_pred = stacking_clf.predict(X_test)

print(f"Stacking Accuracy: {accuracy_score(y_test, y_pred):.4f}")

# Compare with individual models
for name, model in stacking_clf.named_estimators_.items():
    print(f"  {name} Accuracy: {model.score(X_test, y_test):.4f}")
```

### Key Parameters

```python
StackingClassifier(
    estimators=[              # List of (name, estimator) tuples
        ('name1', model1),    # Base learners
        ('name2', model2),
    ],
    final_estimator=None,     # Meta-learner (default: LogisticRegression)
    cv=5,                     # Cross-validation strategy
    stack_method='auto',      # 'auto', 'predict_proba', 'decision_function', 'predict'
    passthrough=False,        # If True, add original features to meta-features
    n_jobs=None               # Parallel jobs
)
```

### Using Probabilities vs Class Labels

```python
# stack_method='predict_proba' — uses class probabilities
# This is generally BETTER because it provides more information

# For binary classification with 3 base models using probabilities:
# Meta-features: [p₁_class1, p₂_class1, p₃_class1]  (3 features)

# For binary classification using class labels:
# Meta-features: [label₁, label₂, label₃]  (3 features, but less info)

# For multi-class (K classes) with probabilities:
# Meta-features: [p₁_c1, p₁_c2, ..., p₁_cK, p₂_c1, ..., p₃_cK]
# (K × num_base_models features)
```

### Passthrough: Including Original Features

```python
# passthrough=True adds original features alongside meta-features

stacking_with_passthrough = StackingClassifier(
    estimators=[
        ('rf', RandomForestClassifier(n_estimators=100, random_state=42)),
        ('svm', SVC(probability=True, random_state=42)),
    ],
    final_estimator=LogisticRegression(max_iter=1000),
    cv=5,
    passthrough=True,   # Meta-features = [RF_prob, SVM_prob, x₁, x₂, ..., xₚ]
    n_jobs=-1
)

# When to use passthrough=True:
# ✓ When base models are weak and miss important features
# ✓ When you have few base models
# ✗ When you have many original features (meta-learner may overfit)
# ✗ When base models are already very strong
```

---

## 6. Choosing Base and Meta Models

### Principles for Choosing Base Learners

```
┌──────────────────────────────────────────────────────────────┐
│                  BASE LEARNER SELECTION                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. DIVERSITY is CRITICAL                                    │
│     Use different TYPES of models, not variations of one     │
│                                                              │
│     Good: RF + SVM + KNN + GBM + LR                          │
│     Bad:  RF₁ + RF₂ + RF₃ + RF₄ + RF₅                       │
│                                                              │
│  2. Each base learner should be REASONABLY accurate          │
│     A terrible model adds noise, not information             │
│                                                              │
│  3. Models should make DIFFERENT errors                       │
│     If all models fail on the same examples,                 │
│     the meta-learner can't help                              │
│                                                              │
│  4. Mix LINEAR and NON-LINEAR models                          │
│     Linear: LogReg, LinearSVM, Ridge                         │
│     Non-linear: RF, GBM, KNN, SVM-RBF, Neural Net           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Typical Base Learner Combinations

```
Combination 1 (Classic):
  • Random Forest
  • Gradient Boosting (XGBoost/LightGBM)
  • SVM (RBF kernel)
  • K-Nearest Neighbors
  • Logistic Regression

Combination 2 (Tree-focused):
  • XGBoost
  • LightGBM
  • CatBoost
  • Extra Trees
  • Random Forest

Combination 3 (Diverse):
  • Neural Network (MLP)
  • XGBoost
  • SVM (Linear + RBF)
  • KNN
  • Ridge Regression
```

### Choosing the Meta-Learner

```
┌──────────────────────────────────────────────────────────────┐
│                  META-LEARNER SELECTION                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SIMPLE is usually BETTER for the meta-learner!              │
│                                                              │
│  Why? The meta-learner operates on very FEW features         │
│  (one per base model). A complex meta-learner would          │
│  overfit these few features easily.                          │
│                                                              │
│  Recommended meta-learners:                                  │
│    Classification:                                           │
│      ✓ Logistic Regression (most common, usually best)       │
│      ✓ Ridge Classifier                                      │
│      ✓ Small Neural Network                                  │
│      ✗ Random Forest (too complex, overfits meta-features)   │
│      ✗ Deep Neural Network (overkill)                        │
│                                                              │
│    Regression:                                               │
│      ✓ Linear Regression / Ridge                             │
│      ✓ Lasso (L1 — can select which base models to keep)     │
│      ✓ ElasticNet                                            │
│                                                              │
│  Exception: If using passthrough=True, a slightly more       │
│  complex meta-learner may be appropriate.                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Measuring Diversity

```python
import numpy as np
from sklearn.metrics import cohen_kappa_score

def compute_diversity(models, X, y):
    """Compute pairwise disagreement between models."""
    predictions = [model.predict(X) for model in models]
    n_models = len(models)
    
    print("Pairwise Disagreement Matrix:")
    print("=" * 40)
    
    for i in range(n_models):
        for j in range(i + 1, n_models):
            disagreement = np.mean(predictions[i] != predictions[j])
            kappa = cohen_kappa_score(predictions[i], predictions[j])
            print(f"  Model {i+1} vs Model {j+1}: "
                  f"Disagreement={disagreement:.3f}, "
                  f"Kappa={kappa:.3f}")
    
    # Higher disagreement = more diversity = better for stacking
```

---

## 7. Multi-Level Stacking

### The Concept

```
Level 0:  Model A    Model B    Model C    Model D    Model E
          │          │          │          │          │
          ▼          ▼          ▼          ▼          ▼
          pred_A     pred_B     pred_C     pred_D     pred_E
          │          │          │          │          │
          ├──────────┤          ├──────────┤          │
          │          │          │          │          │
Level 1:  Meta-1     │          Meta-2     │          │
          (RF+SVM    │          (GBM+KNN   │          │
           combo)    │           combo)    │          │
          │          │          │          │          │
          ▼          │          ▼          │          │
          pred_M1    │          pred_M2    │          │
          │          │          │          │          │
          └──────────┼──────────┘          │          │
                     │                     │          │
Level 2:          Meta-Final               │          │
                  (LogReg on               │          │
                   M1, B, M2,              │          │
                   D, E)                   │          │
                     │                                
                     ▼
               Final Prediction
```

### Implementation Considerations

```
Multi-level stacking requires VERY careful cross-validation:

Level 0: K-fold CV → out-of-fold predictions for Level 1
Level 1: K-fold CV on OOF predictions → out-of-fold for Level 2
Level 2: Train on OOF predictions from Level 1

EACH LEVEL must use cross-validation to avoid leakage!

Practical advice:
  • 2 levels is usually enough (diminishing returns beyond that)
  • 3+ levels rarely helps and increases complexity
  • More levels = more risk of overfitting to the CV scheme
  • More levels = harder to debug and maintain
```

### Multi-Level Stacking in Code

```python
from sklearn.ensemble import StackingClassifier, RandomForestClassifier
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier

# Level 1 stacker (combine RF + SVM)
level1_stack1 = StackingClassifier(
    estimators=[
        ('rf', RandomForestClassifier(n_estimators=100, random_state=42)),
        ('svm', SVC(probability=True, random_state=42)),
    ],
    final_estimator=LogisticRegression(max_iter=1000),
    cv=5
)

# Level 1 stacker (combine GBM + KNN)
level1_stack2 = StackingClassifier(
    estimators=[
        ('gb', GradientBoostingClassifier(n_estimators=100, random_state=42)),
        ('knn', KNeighborsClassifier(n_neighbors=5)),
    ],
    final_estimator=LogisticRegression(max_iter=1000),
    cv=5
)

# Level 2: Final meta-learner
final_stack = StackingClassifier(
    estimators=[
        ('stack1', level1_stack1),
        ('stack2', level1_stack2),
        ('lr', LogisticRegression(max_iter=1000)),  # direct base model
    ],
    final_estimator=LogisticRegression(max_iter=1000),
    cv=5
)

final_stack.fit(X_train, y_train)
print(f"Multi-level Stacking Accuracy: {final_stack.score(X_test, y_test):.4f}")
```

---

## 8. Practical Example

### Complete Stacking Pipeline

```python
import numpy as np
import pandas as pd
from sklearn.datasets import load_wine
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.ensemble import (
    StackingClassifier, RandomForestClassifier,
    GradientBoostingClassifier, ExtraTreesClassifier
)
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.linear_model import LogisticRegression, RidgeClassifier
from sklearn.neural_network import MLPClassifier
from sklearn.metrics import accuracy_score, classification_report

# ============================================================
# 1. Load Data
# ============================================================
X, y = load_wine(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, stratify=y, random_state=42
)
print(f"Training: {X_train.shape}, Test: {X_test.shape}")

# ============================================================
# 2. Define Base Learners (DIVERSE!)
# ============================================================
# Some models need scaling, so we wrap them in pipelines
base_learners = [
    ('rf', RandomForestClassifier(n_estimators=200, random_state=42)),
    ('gb', GradientBoostingClassifier(n_estimators=100, learning_rate=0.1, random_state=42)),
    ('et', ExtraTreesClassifier(n_estimators=200, random_state=42)),
    ('svm', Pipeline([
        ('scaler', StandardScaler()),
        ('svm', SVC(probability=True, kernel='rbf', random_state=42))
    ])),
    ('knn', Pipeline([
        ('scaler', StandardScaler()),
        ('knn', KNeighborsClassifier(n_neighbors=7))
    ])),
    ('mlp', Pipeline([
        ('scaler', StandardScaler()),
        ('mlp', MLPClassifier(hidden_layer_sizes=(50, 25), max_iter=500, random_state=42))
    ]))
]

# ============================================================
# 3. Evaluate Individual Models
# ============================================================
print("\nIndividual Model Performance (5-fold CV):")
print("=" * 50)
for name, model in base_learners:
    scores = cross_val_score(model, X_train, y_train, cv=5, scoring='accuracy')
    print(f"  {name:6s}: {scores.mean():.4f} ± {scores.std():.4f}")

# ============================================================
# 4. Build Stacking Ensemble
# ============================================================
stacking_clf = StackingClassifier(
    estimators=base_learners,
    final_estimator=LogisticRegression(max_iter=1000, C=1.0),
    cv=5,
    stack_method='predict_proba',
    n_jobs=-1
)

# ============================================================
# 5. Evaluate Stacking
# ============================================================
stacking_scores = cross_val_score(
    stacking_clf, X_train, y_train, cv=5, scoring='accuracy'
)
print(f"\n{'='*50}")
print(f"Stacking: {stacking_scores.mean():.4f} ± {stacking_scores.std():.4f}")
print(f"{'='*50}")

# ============================================================
# 6. Train Final Model and Evaluate on Test Set
# ============================================================
stacking_clf.fit(X_train, y_train)
y_pred = stacking_clf.predict(X_test)

print(f"\nTest Set Results:")
print(f"  Stacking Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(f"\nClassification Report:")
print(classification_report(y_test, y_pred, target_names=load_wine().target_names))

# ============================================================
# 7. Compare with Individual Models on Test Set
# ============================================================
print("\nTest Set — Individual vs Stacking:")
print("=" * 50)
for name, model in base_learners:
    model.fit(X_train, y_train)
    acc = accuracy_score(y_test, model.predict(X_test))
    print(f"  {name:6s}: {acc:.4f}")
print(f"  {'STACK':6s}: {accuracy_score(y_test, y_pred):.4f}  ← Stacking")

# ============================================================
# 8. Inspect Meta-Learner Weights
# ============================================================
meta_model = stacking_clf.final_estimator_
if hasattr(meta_model, 'coef_'):
    # For multi-class, coef_ has shape (n_classes, n_meta_features)
    print(f"\nMeta-Learner Coefficients (shows how much each base model contributes):")
    base_names = [name for name, _ in base_learners]
    
    # Each base model contributes n_classes probability features
    n_classes = len(np.unique(y))
    for cls_idx in range(n_classes):
        print(f"\n  Class {cls_idx}:")
        for i, name in enumerate(base_names):
            start = i * n_classes
            end = start + n_classes
            coefs = meta_model.coef_[cls_idx, start:end]
            print(f"    {name}: {coefs}")
```

### Stacking for Regression

```python
from sklearn.ensemble import StackingRegressor, RandomForestRegressor
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.linear_model import Ridge, Lasso
from sklearn.svm import SVR
from sklearn.neighbors import KNeighborsRegressor
from sklearn.datasets import make_regression

X, y = make_regression(n_samples=500, n_features=10, noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

stacking_reg = StackingRegressor(
    estimators=[
        ('rf', RandomForestRegressor(n_estimators=100, random_state=42)),
        ('gb', GradientBoostingRegressor(n_estimators=100, random_state=42)),
        ('svr', Pipeline([('scaler', StandardScaler()), ('svr', SVR())])),
        ('knn', Pipeline([('scaler', StandardScaler()), ('knn', KNeighborsRegressor())])),
    ],
    final_estimator=Ridge(alpha=1.0),
    cv=5,
    n_jobs=-1
)

stacking_reg.fit(X_train, y_train)
from sklearn.metrics import mean_squared_error, r2_score
y_pred = stacking_reg.predict(X_test)
print(f"Stacking R²:  {r2_score(y_test, y_pred):.4f}")
print(f"Stacking MSE: {mean_squared_error(y_test, y_pred):.2f}")
```

---

## 9. Applications

| Domain | Application | Base Learners | Notes |
|--------|-------------|---------------|-------|
| **Kaggle Competitions** | Almost all winning solutions | XGB + LGBM + CatBoost + NN | Standard practice in competitions |
| **Medical Diagnosis** | Disease prediction | RF + SVM + LR + NN | Combine diverse clinical models |
| **Finance** | Credit scoring | GBM + LogReg + RF | Regulatory-compliant ensemble |
| **NLP** | Sentiment analysis | BERT + XGBoost + CNN | Deep learning + classical ML |
| **Computer Vision** | Image classification | Multiple CNN architectures | ResNet + EfficientNet + ViT |
| **Climate Science** | Weather forecasting | Multiple physics models | "Multi-model ensemble" |

### When to Use Stacking

```
✅ USE stacking when:
  • Maximum accuracy is the goal
  • You have diverse, reasonably accurate base models
  • Computation time is not critical
  • You're in a competition or high-stakes application

❌ AVOID stacking when:
  • Interpretability is required
  • Training time is critical
  • Dataset is very small (CV-based stacking needs enough data)
  • A single well-tuned model is already excellent
  • Deployment simplicity is a priority
```

---

## 10. Summary Table

| Aspect | Details |
|--------|---------|
| **Inventor** | David Wolpert (1992) |
| **Core Idea** | Train a meta-learner to combine base model predictions |
| **Level 0** | Base learners (diverse models: RF, SVM, GBM, KNN, etc.) |
| **Level 1** | Meta-learner (simple model: LogReg, Ridge, etc.) |
| **Key Requirement** | Cross-validation for generating meta-features |
| **Why CV Needed** | Prevents leakage — base model predictions on training data are optimistic |
| **stack_method** | 'predict_proba' (preferred) or 'predict' |
| **passthrough** | Optionally include original features for meta-learner |
| **Diversity** | Critical — use different model types, not variations of one |
| **Meta-Learner** | Keep it SIMPLE (LogReg is usually best) |
| **Blending** | Simplified version using holdout instead of CV |
| **Multi-Level** | Rarely needed; 2 levels usually sufficient |
| **Strength** | Often achieves highest accuracy |
| **Weakness** | Complex, computationally expensive, hard to interpret |

---

## 11. Revision Questions

**Q1.** Explain why naive stacking (without cross-validation) leads to overfitting. Illustrate with a concrete example.

<details>
<summary>Answer</summary>

**Naive stacking overfits because:**

Consider a base model that memorizes the training data (e.g., k-NN with k=1, achieving 100% training accuracy). 

In naive stacking:
1. Train k-NN on all training data
2. Predict on training data → all predictions are perfect (100% accuracy)
3. Meta-learner sees "k-NN always predicts correctly" and learns to trust it completely
4. On test data, k-NN's predictions are much worse → the meta-learner's trust is misplaced

With cross-validation:
1. For fold 1: Train k-NN on folds 2-5, predict fold 1 → predictions are honest (not perfect)
2. Meta-learner sees k-NN's TRUE generalization performance
3. If k-NN generalizes poorly, the meta-learner learns to weight it down

The key issue is that training set predictions are **biased estimates** of generalization performance. Cross-validation provides **unbiased** out-of-fold predictions that accurately reflect each base model's true ability.
</details>

**Q2.** You have 5 base models, each achieving ~85% accuracy but making different errors. The best single model achieves 87%. What accuracy range might you expect from stacking, and what factors determine this?

<details>
<summary>Answer</summary>

**Expected range:** 88-92% accuracy, potentially higher depending on:

1. **Diversity of errors:** If all 5 models fail on the same 15% of examples, stacking can't help (output ≈ 85%). If they fail on completely different examples, stacking could theoretically reach ~99% (each failure is outvoted).

2. **Base model accuracy:** Higher individual accuracy gives stacking more to work with.

3. **Error correlation:** Low correlation between model errors → high stacking benefit. This can be measured by pairwise disagreement or diversity metrics.

4. **Meta-learner choice:** A well-chosen meta-learner that captures the complementary strengths of base models maximizes improvement.

5. **Dataset size:** Enough data for reliable cross-validation of meta-features.

**Rule of thumb:** Stacking typically adds 1-3% absolute accuracy improvement over the best single model. The improvement is larger when base models are diverse and the dataset is complex.
</details>

**Q3.** Why should the meta-learner be simple (e.g., Logistic Regression) rather than complex (e.g., Random Forest)?

<details>
<summary>Answer</summary>

The meta-learner operates on a **very small feature space**: one feature per base model (or K features per model if using multi-class probabilities). With 5 base models and binary classification, the meta-learner has only 5 input features.

**Why simple works better:**
1. **Few features:** With only 5-15 meta-features, a complex model like Random Forest would overfit. There aren't enough features for tree splitting to be meaningful.

2. **Low-dimensional input:** Logistic Regression is highly effective in low dimensions. It learns a simple linear combination: "trust model A this much, model B this much."

3. **Regularization:** Simple models inherently regularize against overfitting to the CV-generated meta-features.

4. **Interpretability:** The meta-learner's coefficients tell you directly how much each base model contributes.

5. **Training data quality:** Meta-features from CV can have slight biases. A complex meta-learner might fit these biases, while a simple one smooths over them.

**Exception:** If `passthrough=True` adds many original features, a slightly more complex meta-learner (e.g., regularized GBM) may be appropriate.
</details>

**Q4.** Compare stacking with voting ensembles. When does stacking provide a significant advantage over simple averaging?

<details>
<summary>Answer</summary>

**Voting (averaging):** H(x) = Σ wᵢ hᵢ(x) / Σ wᵢ (fixed linear combination)
**Stacking:** H(x) = g(h₁(x), h₂(x), ...) (learned combination)

**Stacking wins when:**
1. **Models have region-specific expertise:** Model A is best for region R₁, Model B is best for region R₂. The meta-learner can learn this pattern and route accordingly.

2. **Models have complementary strengths:** One model is great at precision, another at recall. The meta-learner balances these optimally.

3. **Non-linear interactions matter:** "When Model A says 0.9 AND Model B says 0.3, the true answer is usually class 0" — stacking captures this; voting cannot.

**Voting is sufficient when:**
1. All models are roughly equally accurate across all regions
2. Models are already diverse and complementary
3. The dataset is too small for reliable CV-based stacking
4. Simplicity and interpretability are priorities

In practice, stacking provides 0.5-2% accuracy improvement over voting. The improvement is larger with more diverse base models and more complex decision boundaries.
</details>

**Q5.** Design a stacking ensemble for a multi-class text classification problem with 10 classes. Specify your base learners, meta-learner, and explain your design choices.

<details>
<summary>Answer</summary>

**Base Learners (Level 0):**

1. **TF-IDF + Logistic Regression** — Strong linear baseline for text, fast training, captures direct word-class associations.

2. **TF-IDF + SVM (Linear kernel)** — Excellent for high-dimensional text features, different optimization than LogReg, good at finding margins.

3. **TF-IDF + Random Forest** — Captures non-linear feature interactions, robust to noisy features, different inductive bias from linear models.

4. **TF-IDF + XGBoost** — Gradient boosting on text features, captures complex patterns, strong on tabular/structured features.

5. **TF-IDF + Multinomial Naive Bayes** — Probabilistic model with independence assumption, very different from others, fast.

**Meta-Learner:** Logistic Regression with L2 regularization
- Input: 5 models × 10 class probabilities = 50 meta-features
- Simple enough to avoid overfitting 50 features
- L2 regularization prevents overreliance on any single model
- Softmax output gives well-calibrated probabilities

**Design Choices:**
- All models use the same TF-IDF representation (consistency)
- `stack_method='predict_proba'` for maximum information
- 5-fold CV for meta-feature generation
- Models span linear (LogReg, SVM), non-linear (RF, XGB), and probabilistic (NB) paradigms
- No `passthrough=True` since TF-IDF is high-dimensional
</details>

**Q6.** Explain the concept of "information leakage" in stacking. How does k-fold cross-validation prevent it, and are there scenarios where even CV-based stacking might have subtle leakage?

<details>
<summary>Answer</summary>

**Information leakage:** When the meta-learner receives predictions from base models that were trained on the same data used to evaluate them. This makes base model predictions appear artificially good.

**How CV prevents it:** Cross-validation ensures that each training sample's meta-feature is predicted by a model that **never** saw that sample during training. For fold k:
- Base model trained on folds {1,...,K}\{k}
- Predicts only on fold k
- These are honest, out-of-sample predictions

**Subtle leakage scenarios (even with CV):**

1. **Temporal leakage:** If data has time-ordering (e.g., stock prices) and CV doesn't respect it, future information leaks into past predictions. Solution: Use TimeSeriesSplit instead of random KFold.

2. **Group leakage:** If samples are correlated (e.g., multiple measurements from the same patient), random CV might put correlated samples in different folds. Solution: Use GroupKFold.

3. **Feature engineering leakage:** If feature preprocessing (like StandardScaler or target encoding) is done BEFORE CV splitting, information from the entire dataset leaks. Solution: Include preprocessing inside the CV loop (using sklearn Pipelines).

4. **Overlapping data:** If base models were pre-trained on data that overlaps with the stacking dataset, their predictions carry implicit knowledge of the target.

Proper stacking requires the entire pipeline — preprocessing + base model training + prediction — to be contained within the CV loop.
</details>

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← 8.6 XGBoost, LightGBM & CatBoost](./06-xgboost-lightgbm-catboost.md) | [Unit 8: Ensemble Methods](../README.md) | [8.8 Voting Classifiers →](./08-voting-classifiers.md) |

---

*© 2025 AIML Study Notes. Unit 8: Ensemble Methods — Chapter 7: Stacking.*
