# Chapter 8.8: Voting Classifiers

> **Unit 8: Ensemble Methods** — File 8 of 8

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Hard Voting (Majority Voting)](#2-hard-voting-majority-voting)
3. [Soft Voting (Average Probabilities)](#3-soft-voting-average-probabilities)
4. [Weighted Voting](#4-weighted-voting)
5. [VotingClassifier in Scikit-Learn](#5-votingclassifier-in-scikit-learn)
6. [When to Use Voting](#6-when-to-use-voting)
7. [Diversity of Base Models](#7-diversity-of-base-models)
8. [Comparison with Other Ensemble Methods](#8-comparison-with-other-ensemble-methods)
9. [Practical Example](#9-practical-example)
10. [Applications](#10-applications)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)

---

## 1. Introduction

**Voting** is the simplest ensemble method: combine multiple models by **voting** on the final prediction. Despite its simplicity, voting can be surprisingly effective — especially when base models are diverse and individually strong.

```
┌─────────────────────────────────────────────────────────────┐
│                        VOTING                                │
│                                                             │
│  Input x → Model A: "Cat"   ┐                              │
│          → Model B: "Cat"   ├──→ VOTE → "Cat" (majority)   │
│          → Model C: "Dog"   ┘                              │
│                                                             │
│  The simplest ensemble: just ask everyone and go with       │
│  the majority. No training, no meta-learning, just voting.  │
└─────────────────────────────────────────────────────────────┘
```

### Types of Voting

```
┌──────────────┬──────────────────────────────┐
│   Type       │   How It Works               │
├──────────────┼──────────────────────────────┤
│ Hard Voting  │ Majority of class labels     │
│ Soft Voting  │ Average of probabilities     │
│ Weighted     │ Weighted labels/probabilities │
└──────────────┴──────────────────────────────┘
```

---

## 2. Hard Voting (Majority Voting)

### How It Works

Each base model makes a class prediction. The final prediction is the class that receives the **most votes**.

```
For input x, with 5 base models:

  Model 1: Class A  ─┐
  Model 2: Class B   │
  Model 3: Class A   ├──→ Tally: A=3, B=2  →  Final: Class A
  Model 4: Class A   │
  Model 5: Class B  ─┘
```

### Mathematical Formulation

```
H(x) = argmax  Σ I(hₖ(x) = c)
          c    k=1

Where:
  H(x) = ensemble prediction
  hₖ(x) = prediction of model k
  B = number of base models
  c = class label
  I(·) = indicator function
```

### Why Majority Voting Works — The Math

Assume `B` independent models, each with accuracy `p > 0.5`:

```
P(majority correct) = Σ C(B,k) · pᵏ · (1-p)^(B-k)
                     k=⌈B/2⌉

For B = 5, p = 0.7:

P(≥3 correct) = C(5,3)·0.7³·0.3² + C(5,4)·0.7⁴·0.3¹ + C(5,5)·0.7⁵·0.3⁰
              = 10·0.343·0.09 + 5·0.2401·0.3 + 1·0.16807·1
              = 0.3087 + 0.3601 + 0.1681
              = 0.8369

Individual: 70%  →  Majority (5 models): 83.7%  ✓
```

### Accuracy vs Number of Voters

```
  Ensemble Accuracy
  1.0 │                            ___________  p = 0.8
      │                    _______/
  0.9 │              _____/
      │           __/                    _______  p = 0.7
  0.8 │        __/              ________/
      │      _/            ____/
  0.7 │    _/          ___/                 ____  p = 0.6
      │   /        ___/               ____/
  0.6 │  /      __/              ____/
      │ /    __/            ____/
  0.5 │/___________________________                p = 0.5 (random)
      │
  0.4 │\____
      │     \____                               p = 0.4
  0.3 │          \______
      └──┬──┬──┬──┬──┬──┬──┬──┬──┬──► Number of voters
         1  3  5  7  9  11 15 21 31

Key insight:
  • p > 0.5: More voters → accuracy increases toward 1.0
  • p = 0.5: More voters → stays at 0.5 (random)
  • p < 0.5: More voters → accuracy DECREASES (amplifies error!)
```

### Tie-Breaking

```
When B is even, ties are possible:

  B = 4: [A, A, B, B]  →  TIE!

Strategies:
  1. Use ODD number of models (prevents ties for binary)
  2. Random tie-breaking
  3. Default to one class (e.g., positive class)
  4. Use soft voting instead (continuous probabilities rarely tie)

For multi-class, ties are possible even with odd B:
  B = 3: [A, B, C]  →  TIE!  (each class gets 1 vote)
```

---

## 3. Soft Voting (Average Probabilities)

### How It Works

Each base model predicts **class probabilities**. The final prediction is the class with the highest **average probability**.

```
For binary classification (class 0 vs class 1):

  Model 1: P(class 1) = 0.90  ─┐
  Model 2: P(class 1) = 0.40   │   Average P(class 1) = 
  Model 3: P(class 1) = 0.85   ├──→ (0.90+0.40+0.85+0.70+0.60)/5
  Model 4: P(class 1) = 0.70   │   = 0.69
  Model 5: P(class 1) = 0.60  ─┘
  
  Average > 0.5 → Final prediction: class 1
```

### Mathematical Formulation

```
H(x) = argmax  (1/B) Σ P_k(y = c | x)
          c           k=1

Where P_k(y = c | x) is model k's predicted probability for class c.
```

### Why Soft Voting Is Usually Better Than Hard Voting

```
Scenario: 3 models, true class = 1

Hard Voting:
  Model A: predicts 0 (P(1) = 0.49)  ─┐
  Model B: predicts 0 (P(1) = 0.49)   ├──→ Majority: class 0  ✗ WRONG!
  Model C: predicts 1 (P(1) = 0.99)  ─┘

Soft Voting:
  Model A: P(1) = 0.49  ─┐
  Model B: P(1) = 0.49   ├──→ Average P(1) = (0.49+0.49+0.99)/3 = 0.657
  Model C: P(1) = 0.99  ─┘
  
  Average > 0.5 → class 1  ✓ CORRECT!

Why? Model C is VERY confident about class 1 (0.99).
     Hard voting ignores this confidence.
     Soft voting captures it through probability averaging.
```

### Visualization

```
  Hard Voting (binary):
  ┌────────────────────────────────┐
  │ Model predictions:             │
  │   A: ████████░░ → 0 (80% conf)│  
  │   B: ██████░░░░ → 0 (60% conf)│  → Hard vote: 0 (2 vs 1)
  │   C: █████████░ → 1 (90% conf)│
  └────────────────────────────────┘

  Soft Voting (binary):
  ┌────────────────────────────────┐
  │ Model probabilities for cls 1: │
  │   A: P(1) = 0.20              │
  │   B: P(1) = 0.40              │  → Avg P(1) = (0.20+0.40+0.90)/3
  │   C: P(1) = 0.90              │           = 0.50
  └────────────────────────────────┘
  Borderline case! Confidence matters.
```

---

## 4. Weighted Voting

### Assigning Weights to Models

Not all models are equally good. **Weighted voting** allows better models to have more influence.

```
Weighted Hard Voting:
  H(x) = argmax  Σ wₖ · I(hₖ(x) = c)
            c    k=1

Weighted Soft Voting:
  H(x) = argmax  Σ wₖ · P_k(y = c | x) / Σ wₖ
            c    k=1                        k=1
```

### Example

```
Three models with different accuracies:
  Model A (accuracy=0.95): weight = 3
  Model B (accuracy=0.85): weight = 2  
  Model C (accuracy=0.75): weight = 1

For a new sample:
  Model A: Class 1 (P=0.92)
  Model B: Class 0 (P=0.70 for class 0)
  Model C: Class 0 (P=0.60 for class 0)

Hard Voting (unweighted):
  Class 0: 2 votes, Class 1: 1 vote → Class 0

Hard Voting (weighted):
  Class 0: 2+1 = 3 votes, Class 1: 3 votes → TIE!

Soft Voting (weighted):
  P(class 1) = (3×0.92 + 2×0.30 + 1×0.40) / (3+2+1)
             = (2.76 + 0.60 + 0.40) / 6
             = 3.76 / 6 = 0.627 → Class 1  ✓

The highly accurate Model A's strong confidence (0.92) overrides
the two weaker models' moderate predictions.
```

### How to Determine Weights

```
Method 1: Based on validation accuracy
  wₖ = accuracy_k  (or normalized)

Method 2: Based on cross-validation score
  wₖ = mean_cv_score_k

Method 3: Inverse of error rate
  wₖ = 1 / (1 - accuracy_k)

Method 4: Optimization
  Search for weights that maximize validation accuracy
  (essentially a simplified form of stacking)

Method 5: Equal weights (default)
  wₖ = 1 for all k (reduces to simple voting)
```

---

## 5. VotingClassifier in Scikit-Learn

### Basic Usage

```python
from sklearn.ensemble import VotingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# ============================================================
# Hard Voting
# ============================================================
hard_voting = VotingClassifier(
    estimators=[
        ('lr', LogisticRegression(max_iter=10000, random_state=42)),
        ('rf', RandomForestClassifier(n_estimators=100, random_state=42)),
        ('svm', SVC(random_state=42)),  # No probability needed for hard voting
        ('knn', KNeighborsClassifier(n_neighbors=5)),
    ],
    voting='hard'
)
hard_voting.fit(X_train, y_train)
print(f"Hard Voting Accuracy: {hard_voting.score(X_test, y_test):.4f}")

# ============================================================
# Soft Voting
# ============================================================
soft_voting = VotingClassifier(
    estimators=[
        ('lr', LogisticRegression(max_iter=10000, random_state=42)),
        ('rf', RandomForestClassifier(n_estimators=100, random_state=42)),
        ('svm', SVC(probability=True, random_state=42)),  # probability=True needed!
        ('knn', KNeighborsClassifier(n_neighbors=5)),
    ],
    voting='soft'
)
soft_voting.fit(X_train, y_train)
print(f"Soft Voting Accuracy: {soft_voting.score(X_test, y_test):.4f}")

# ============================================================
# Weighted Soft Voting
# ============================================================
weighted_voting = VotingClassifier(
    estimators=[
        ('lr', LogisticRegression(max_iter=10000, random_state=42)),
        ('rf', RandomForestClassifier(n_estimators=100, random_state=42)),
        ('svm', SVC(probability=True, random_state=42)),
        ('knn', KNeighborsClassifier(n_neighbors=5)),
    ],
    voting='soft',
    weights=[2, 3, 1, 1]  # RF gets highest weight
)
weighted_voting.fit(X_train, y_train)
print(f"Weighted Voting Accuracy: {weighted_voting.score(X_test, y_test):.4f}")
```

### Key Parameters

```python
VotingClassifier(
    estimators=[              # List of (name, estimator) tuples
        ('name1', model1),
        ('name2', model2),
    ],
    voting='hard',            # 'hard' or 'soft'
    weights=None,             # List of weights for each estimator
    n_jobs=None,              # Parallel training (-1 = all cores)
    flatten_transform=True,   # Shape of transform output
    verbose=False
)
```

### VotingRegressor

```python
from sklearn.ensemble import VotingRegressor
from sklearn.linear_model import LinearRegression, Ridge
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.datasets import make_regression
from sklearn.metrics import r2_score

X, y = make_regression(n_samples=500, n_features=10, noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

voting_reg = VotingRegressor(
    estimators=[
        ('lr', LinearRegression()),
        ('rf', RandomForestRegressor(n_estimators=100, random_state=42)),
        ('gb', GradientBoostingRegressor(n_estimators=100, random_state=42)),
    ],
    weights=[1, 2, 3]  # GB gets highest weight
)
voting_reg.fit(X_train, y_train)

# Individual model comparison
for name, model in [('lr', LinearRegression()), 
                     ('rf', RandomForestRegressor(100, random_state=42)),
                     ('gb', GradientBoostingRegressor(100, random_state=42))]:
    model.fit(X_train, y_train)
    print(f"  {name:3s} R²: {r2_score(y_test, model.predict(X_test)):.4f}")

print(f"  Voting R²: {r2_score(y_test, voting_reg.predict(X_test)):.4f}")
```

---

## 6. When to Use Voting

### Decision Framework

```
┌─────────────────────────────────────────────────────────┐
│         WHEN TO USE VOTING ENSEMBLES                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ✅ USE voting when:                                     │
│     • You have several decent models already trained     │
│     • You want a QUICK accuracy boost with no effort     │
│     • Models are diverse (different algorithms)          │
│     • Deployment simplicity matters                      │
│     • You don't want to retrain a meta-learner           │
│     • Baseline before trying stacking                    │
│                                                         │
│  ❌ AVOID voting when:                                    │
│     • All models are very similar (same algorithm)       │
│     • One model is clearly much better than others       │
│     • You need maximum accuracy (use stacking instead)   │
│     • Models disagree on which is more trustworthy       │
│       for different input regions                        │
│                                                         │
│  💡 PREFER SOFT over HARD voting when:                   │
│     • All models support predict_proba()                 │
│     • Models have well-calibrated probabilities          │
│     • You want to capture confidence information         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Voting as a Starting Point

```
Development Pipeline:
  
  Step 1: Train individual models
          → LR: 85%, RF: 88%, SVM: 86%, GBM: 89%
  
  Step 2: Try simple voting (baseline ensemble)
          → Hard Voting: 90%, Soft Voting: 91%
          → Already better than any single model!
  
  Step 3: If needed, try weighted voting
          → Weighted Soft: 91.5%
          
  Step 4: If still need more, try stacking
          → Stacking: 92.5%
          → Worth the complexity? Depends on your needs.
```

---

## 7. Diversity of Base Models

### Why Diversity Matters

```
Scenario A: All models are similar (LOW diversity)
═════════════════════════════════════════════════

  Model 1 (RF):   Correct: ████████░░  Error: ██
  Model 2 (RF*):  Correct: ████████░░  Error: ██  ← Same errors!
  Model 3 (RF**): Correct: ████████░░  Error: ██
  
  Voting: ████████░░ Error: ██  (No improvement!)
  
Scenario B: Models are diverse (HIGH diversity)
═════════════════════════════════════════════════

  Model 1 (RF):   Correct: ████████░░  Error: ░██
  Model 2 (SVM):  Correct: █████████░  Error: █░█  ← Different errors!
  Model 3 (KNN):  Correct: ████████░░  Error: ██░
  
  Voting: █████████░ Error: ░░█  (Only where ALL fail!)
```

### Sources of Diversity

```
┌──────────────────────────────────────────────────────────┐
│              SOURCES OF MODEL DIVERSITY                   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  1. ALGORITHM DIVERSITY (most important for voting)      │
│     Use fundamentally different learning algorithms:     │
│     - Tree-based: RF, GBM, Decision Tree                 │
│     - Distance-based: KNN, SVM (RBF)                     │
│     - Linear: Logistic Regression, Linear SVM            │
│     - Probabilistic: Naive Bayes                         │
│     - Neural: MLP, Deep Learning                         │
│                                                          │
│  2. HYPERPARAMETER DIVERSITY                             │
│     Same algorithm with different settings:              │
│     - RF (100 trees) + RF (500 trees, max_depth=5)       │
│     - KNN (k=3) + KNN (k=15) + KNN (k=50)               │
│     Less effective than algorithm diversity               │
│                                                          │
│  3. FEATURE DIVERSITY                                    │
│     Each model uses different feature subsets:            │
│     - Model 1: features [1-10]                           │
│     - Model 2: features [5-15]                           │
│     - Model 3: features [10-20]                          │
│                                                          │
│  4. DATA DIVERSITY                                       │
│     Each model trained on different data samples:        │
│     - This is what bagging does automatically             │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Measuring Diversity

```python
import numpy as np

def measure_diversity(models, X, y):
    """Compute diversity metrics for a set of models."""
    predictions = np.array([m.predict(X) for m in models])  # shape: (B, n)
    n_models = len(models)
    n_samples = len(y)
    
    # 1. Pairwise Disagreement
    disagreements = []
    for i in range(n_models):
        for j in range(i + 1, n_models):
            dis = np.mean(predictions[i] != predictions[j])
            disagreements.append(dis)
    avg_disagreement = np.mean(disagreements)
    
    # 2. Entropy of predictions
    # For each sample, how much do models disagree?
    entropies = []
    for s in range(n_samples):
        votes = predictions[:, s]
        unique, counts = np.unique(votes, return_counts=True)
        probs = counts / n_models
        entropy = -np.sum(probs * np.log2(probs + 1e-10))
        entropies.append(entropy)
    avg_entropy = np.mean(entropies)
    
    # 3. Correlation of errors
    errors = (predictions != y).astype(int)
    error_corrs = []
    for i in range(n_models):
        for j in range(i + 1, n_models):
            corr = np.corrcoef(errors[i], errors[j])[0, 1]
            error_corrs.append(corr)
    avg_error_corr = np.mean(error_corrs)
    
    print(f"Average Pairwise Disagreement: {avg_disagreement:.4f}")
    print(f"Average Prediction Entropy:    {avg_entropy:.4f}")
    print(f"Average Error Correlation:     {avg_error_corr:.4f}")
    print(f"\nHigher disagreement & entropy = MORE diversity (good!)")
    print(f"Lower error correlation = MORE diversity (good!)")
```

---

## 8. Comparison with Other Ensemble Methods

### Ensemble Methods Landscape

```
┌───────────────────────────────────────────────────────────────┐
│                  ENSEMBLE METHODS COMPARISON                  │
├───────────┬──────────┬──────────┬──────────┬─────────────────┤
│  Aspect   │  Voting  │ Bagging  │ Boosting │   Stacking      │
├───────────┼──────────┼──────────┼──────────┼─────────────────┤
│ Base      │ Diverse  │ Same     │ Same     │ Diverse         │
│ models    │ types    │ type     │ type     │ types           │
├───────────┼──────────┼──────────┼──────────┼─────────────────┤
│ Training  │ Independent│ Parallel│ Sequential│ Independent    │
│ strategy  │           │ (bootstr)│ (correct)│ + meta-learner  │
├───────────┼──────────┼──────────┼──────────┼─────────────────┤
│Combination│ Vote/Avg │ Vote/Avg│ Weighted │ Learned         │
│ rule      │ (fixed)  │ (fixed) │ sum      │ (meta-model)    │
├───────────┼──────────┼──────────┼──────────┼─────────────────┤
│ Reduces   │ Variance │ Variance│ Bias     │ Both            │
│           │ (some)   │ (mainly)│ (mainly) │ (potentially)   │
├───────────┼──────────┼──────────┼──────────┼─────────────────┤
│ Complexity│ Very Low │ Low     │ Medium   │ High            │
├───────────┼──────────┼──────────┼──────────┼─────────────────┤
│ Accuracy  │ Good     │ Good    │ Very Good│ Best            │
│ boost     │          │         │          │ (potentially)   │
├───────────┼──────────┼──────────┼──────────┼─────────────────┤
│ Overfit   │ Resistant│ Very    │ Can      │ Moderate risk   │
│ risk      │          │ resistant│ overfit  │ (CV helps)      │
├───────────┼──────────┼──────────┼──────────┼─────────────────┤
│ Example   │ Voting   │ Random  │ XGBoost  │ Stacking        │
│ algorithm │ Clf      │ Forest  │ AdaBoost │ Classifier      │
└───────────┴──────────┴──────────┴──────────┴─────────────────┘
```

### When Each Method Wins

```
Use Voting when:
  → You already have trained models of different types
  → You want a quick, reliable accuracy boost
  → Simplicity and interpretability matter

Use Bagging when:
  → Your base model has high variance (overfits)
  → You want a robust, parallelizable ensemble

Use Boosting when:
  → Your base model has high bias (underfits)
  → Maximum predictive power is needed
  → Data is relatively clean

Use Stacking when:
  → You need the absolute best accuracy
  → You have diverse, individually strong models
  → Computational resources are available
```

### Accuracy Comparison (Typical Results)

```
  Accuracy
    │
  96│                                               ████ Stacking
    │                                          ████
  94│                                     ████
    │                                ████ ████ Weighted Soft Voting
  92│                           ████
    │                      ████ ████ Soft Voting
  90│                 ████
    │            ████ ████ Hard Voting
  88│       ████
    │  ████ ████ Best Single Model
  86│
    │
  84│
    └──────────────────────────────────────────────
                    Methods
```

---

## 9. Practical Example

### Complete Voting Ensemble Pipeline

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import load_wine
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.ensemble import (
    VotingClassifier, RandomForestClassifier,
    GradientBoostingClassifier, ExtraTreesClassifier
)
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.naive_bayes import GaussianNB
from sklearn.metrics import accuracy_score, classification_report

# ============================================================
# 1. Load Data
# ============================================================
X, y = load_wine(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, stratify=y, random_state=42
)

# ============================================================
# 2. Define Base Models (diverse!)
# ============================================================
models = {
    'lr': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', LogisticRegression(max_iter=10000, random_state=42))
    ]),
    'rf': RandomForestClassifier(n_estimators=200, random_state=42),
    'gb': GradientBoostingClassifier(n_estimators=100, random_state=42),
    'svm': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', SVC(probability=True, random_state=42))
    ]),
    'knn': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', KNeighborsClassifier(n_neighbors=7))
    ]),
    'nb': GaussianNB(),
    'et': ExtraTreesClassifier(n_estimators=200, random_state=42)
}

# ============================================================
# 3. Evaluate Individual Models
# ============================================================
print("Individual Model Performance:")
print("=" * 50)
individual_scores = {}
for name, model in models.items():
    scores = cross_val_score(model, X_train, y_train, cv=5, scoring='accuracy')
    individual_scores[name] = scores.mean()
    print(f"  {name:4s}: {scores.mean():.4f} ± {scores.std():.4f}")

# ============================================================
# 4. Hard Voting
# ============================================================
hard_voter = VotingClassifier(
    estimators=list(models.items()),
    voting='hard',
    n_jobs=-1
)
hard_scores = cross_val_score(hard_voter, X_train, y_train, cv=5)
print(f"\nHard Voting: {hard_scores.mean():.4f} ± {hard_scores.std():.4f}")

# ============================================================
# 5. Soft Voting
# ============================================================
soft_voter = VotingClassifier(
    estimators=list(models.items()),
    voting='soft',
    n_jobs=-1
)
soft_scores = cross_val_score(soft_voter, X_train, y_train, cv=5)
print(f"Soft Voting: {soft_scores.mean():.4f} ± {soft_scores.std():.4f}")

# ============================================================
# 6. Weighted Soft Voting (weight by CV accuracy)
# ============================================================
weights = [individual_scores[name] for name, _ in models.items()]
weights = [w / min(weights) for w in weights]  # normalize

weighted_voter = VotingClassifier(
    estimators=list(models.items()),
    voting='soft',
    weights=weights,
    n_jobs=-1
)
weighted_scores = cross_val_score(weighted_voter, X_train, y_train, cv=5)
print(f"Weighted Soft: {weighted_scores.mean():.4f} ± {weighted_scores.std():.4f}")

# ============================================================
# 7. Final Evaluation on Test Set
# ============================================================
print(f"\n{'='*50}")
print("Test Set Results:")
print(f"{'='*50}")

best_voter = weighted_voter  # or whichever was best on CV
best_voter.fit(X_train, y_train)
y_pred = best_voter.predict(X_test)
print(f"Voting Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(classification_report(y_test, y_pred, target_names=load_wine().target_names))

# Compare with individual models
print("\nIndividual vs Ensemble on Test Set:")
for name, model in models.items():
    model.fit(X_train, y_train)
    acc = accuracy_score(y_test, model.predict(X_test))
    print(f"  {name:4s}: {acc:.4f}")
print(f"  {'VOTE':4s}: {accuracy_score(y_test, y_pred):.4f} ← Ensemble")

# ============================================================
# 8. Visualize Results
# ============================================================
names = list(models.keys()) + ['Hard', 'Soft', 'Weighted']
scores = [individual_scores[k] for k in models.keys()] + \
         [hard_scores.mean(), soft_scores.mean(), weighted_scores.mean()]

colors = ['steelblue'] * len(models) + ['coral', 'tomato', 'red']

plt.figure(figsize=(12, 6))
bars = plt.bar(names, scores, color=colors, edgecolor='black')
plt.ylabel('5-Fold CV Accuracy')
plt.title('Individual Models vs Voting Ensembles')
plt.xticks(rotation=45)
plt.ylim(min(scores) - 0.02, max(scores) + 0.02)

# Add value labels
for bar, score in zip(bars, scores):
    plt.text(bar.get_x() + bar.get_width()/2., bar.get_height() + 0.002,
             f'{score:.4f}', ha='center', va='bottom', fontsize=9)

plt.axhline(y=max(individual_scores.values()), color='gray', 
            linestyle='--', alpha=0.5, label='Best Single Model')
plt.legend()
plt.tight_layout()
plt.show()
```

### Comparing Voting Strategies on Multiple Datasets

```python
from sklearn.datasets import load_iris, load_wine, load_breast_cancer

datasets = {
    'Iris': load_iris(return_X_y=True),
    'Wine': load_wine(return_X_y=True),
    'Breast Cancer': load_breast_cancer(return_X_y=True)
}

for ds_name, (X, y) in datasets.items():
    print(f"\n{'='*50}")
    print(f"Dataset: {ds_name}")
    print(f"{'='*50}")
    
    # Base models
    estimators = [
        ('lr', Pipeline([('s', StandardScaler()), 
                          ('c', LogisticRegression(max_iter=10000))])),
        ('rf', RandomForestClassifier(100, random_state=42)),
        ('svm', Pipeline([('s', StandardScaler()), 
                           ('c', SVC(probability=True, random_state=42))])),
        ('knn', Pipeline([('s', StandardScaler()), 
                           ('c', KNeighborsClassifier())])),
    ]
    
    # Individual scores
    best_individual = 0
    for name, model in estimators:
        score = cross_val_score(model, X, y, cv=5).mean()
        best_individual = max(best_individual, score)
        print(f"  {name:4s}: {score:.4f}")
    
    # Ensemble scores
    hard = VotingClassifier(estimators, voting='hard')
    soft = VotingClassifier(estimators, voting='soft')
    
    hard_score = cross_val_score(hard, X, y, cv=5).mean()
    soft_score = cross_val_score(soft, X, y, cv=5).mean()
    
    print(f"  Hard Voting: {hard_score:.4f}")
    print(f"  Soft Voting: {soft_score:.4f}")
    print(f"  Improvement: {(max(hard_score, soft_score) - best_individual)*100:+.2f}%")
```

---

## 10. Applications

| Domain | Application | Base Models | Voting Type |
|--------|-------------|-------------|-------------|
| **Medical Diagnosis** | Disease prediction | RF + SVM + NN | Soft (calibrated probs) |
| **Sentiment Analysis** | Review classification | BERT + LSTM + LR | Soft |
| **Image Classification** | Object recognition | Multiple CNNs | Soft (probability avg) |
| **Fraud Detection** | Transaction screening | GBM + RF + LogReg | Weighted soft |
| **Weather Forecasting** | Multi-model ensemble | Multiple NWP models | Weighted average |
| **Election Forecasting** | Poll aggregation | Multiple pollster models | Weighted average |
| **Spam Detection** | Email filtering | NB + SVM + RF | Hard (fast inference) |

---

## 11. Summary Table

| Aspect | Details |
|--------|---------|
| **Core Idea** | Combine predictions from multiple models by voting |
| **Hard Voting** | Majority class label |
| **Soft Voting** | Average of predicted probabilities |
| **Weighted Voting** | Models weighted by accuracy/importance |
| **Best For** | Quick ensemble when diverse models are available |
| **Requirement** | Models should be individually accurate + diverse |
| **Soft vs Hard** | Soft usually better (captures confidence) |
| **Soft Requirement** | All models must support predict_proba() |
| **Bias/Variance** | Primarily reduces variance |
| **Complexity** | Very low — no additional training |
| **vs Stacking** | Simpler but less powerful (fixed vs learned combination) |
| **vs Bagging** | Voting uses DIFFERENT models; bagging uses same model type |
| **Overfitting** | Highly resistant (no meta-model to overfit) |
| **Diversity** | Critical — use different algorithm types |
| **sklearn Class** | VotingClassifier, VotingRegressor |

---

## 12. Revision Questions

**Q1.** Prove mathematically that the majority vote of 7 independent models, each with 70% accuracy, achieves higher accuracy than any individual model.

<details>
<summary>Answer</summary>

For majority correct, we need ≥ 4 out of 7 correct.

P(majority correct) = Σ C(7,k) × 0.7ᵏ × 0.3⁷⁻ᵏ for k = 4,5,6,7

k=4: C(7,4) × 0.7⁴ × 0.3³ = 35 × 0.2401 × 0.027 = 0.2269
k=5: C(7,5) × 0.7⁵ × 0.3² = 21 × 0.16807 × 0.09 = 0.3177
k=6: C(7,6) × 0.7⁶ × 0.3¹ = 7 × 0.117649 × 0.3 = 0.2471
k=7: C(7,7) × 0.7⁷ × 0.3⁰ = 1 × 0.0824 × 1.0 = 0.0824

P(majority correct) = 0.2269 + 0.3177 + 0.2471 + 0.0824 = **0.8740**

Individual accuracy: 70%
Ensemble accuracy: **87.4%**
Improvement: +17.4%

This works because each model independently "catches" different errors, and the majority mechanism filters out minority mistakes.
</details>

**Q2.** Give an example where soft voting outperforms hard voting, and another where they give the same result. What property of the base models determines when soft voting helps?

<details>
<summary>Answer</summary>

**Soft wins:**
3 models, true class = 1
- Model A: predicts 0 (P(1) = 0.49)
- Model B: predicts 0 (P(1) = 0.45)
- Model C: predicts 1 (P(1) = 0.99)

Hard voting: class 0 (2-1 majority) ✗
Soft voting: P(1) = (0.49 + 0.45 + 0.99)/3 = 0.643 → class 1 ✓

Soft voting captures that Model C is EXTREMELY confident.

**Same result:**
3 models, true class = 1
- Model A: predicts 1 (P(1) = 0.90)
- Model B: predicts 1 (P(1) = 0.85)
- Model C: predicts 0 (P(1) = 0.30)

Hard voting: class 1 (2-1) ✓
Soft voting: P(1) = (0.90 + 0.85 + 0.30)/3 = 0.683 → class 1 ✓

**Key property:** Soft voting helps when **confidence is unevenly distributed** — some models are very confident and others are near the decision boundary. If all models are equally confident in their predictions, hard and soft voting give similar results. Well-calibrated probabilities are important for soft voting to work correctly.
</details>

**Q3.** You have 4 models with test accuracies: A=92%, B=88%, C=85%, D=90%. Design a weighted voting scheme and calculate the prediction for a sample where A and D predict class 1, while B and C predict class 0.

<details>
<summary>Answer</summary>

**Weight assignment (proportional to accuracy):**
- wₐ = 0.92
- w_B = 0.88
- w_C = 0.85
- w_D = 0.90

**Hard weighted voting:**
- Class 1 votes: wₐ + w_D = 0.92 + 0.90 = 1.82
- Class 0 votes: w_B + w_C = 0.88 + 0.85 = 1.73

Class 1 wins (1.82 > 1.73) → **Prediction: Class 1**

**Note:** With unweighted voting, it would be a 2-2 tie! The weights break the tie in favor of the more accurate models (A and D), which is the right behavior.

**Alternative weight scheme (inverse error):**
- wₐ = 1/(1-0.92) = 12.5
- w_B = 1/(1-0.88) = 8.33
- w_C = 1/(1-0.85) = 6.67
- w_D = 1/(1-0.90) = 10.0

Class 1: 12.5 + 10.0 = 22.5
Class 0: 8.33 + 6.67 = 15.0 → Still Class 1, but with greater margin.
</details>

**Q4.** Explain why using 5 Random Forests with different random seeds is a poor voting ensemble, while using RF + SVM + KNN + LogReg + GBM is a good one.

<details>
<summary>Answer</summary>

**5 Random Forests with different seeds — POOR ensemble:**

1. **Low diversity:** All models use the same algorithm (decision trees with bagging). Different random seeds only change bootstrap samples and feature subsets — the fundamental learning mechanism is identical.

2. **Correlated errors:** Since all models learn the same way (recursive partitioning), they tend to make the same types of mistakes. If RF fails on a region of the input space, ALL five RFs will likely fail there.

3. **Diminishing returns:** RF is already an ensemble of trees. Adding more RF instances is like increasing n_estimators within a single RF — you get diminishing returns beyond ~200-500 trees.

4. **No complementary strengths:** All models share the same inductive bias.

**RF + SVM + KNN + LogReg + GBM — GOOD ensemble:**

1. **High diversity:** Each algorithm has a fundamentally different inductive bias:
   - RF: axis-aligned splits, handles interactions
   - SVM: finds maximum margin hyperplane
   - KNN: local density-based classification
   - LogReg: global linear decision boundary
   - GBM: sequential bias reduction

2. **Uncorrelated errors:** Where RF fails (e.g., smooth linear boundaries), LogReg excels. Where KNN fails (high dimensions), RF and GBM excel.

3. **Complementary strengths:** Linear models capture global trends, tree models capture local patterns, KNN captures local clusters.
</details>

**Q5.** Compare VotingClassifier with StackingClassifier. Under what conditions would voting match or exceed stacking performance?

<details>
<summary>Answer</summary>

**Voting matches/exceeds stacking when:**

1. **Base models are equally accurate across all regions:** If no model is systematically better in specific parts of the input space, a learned combination (stacking) has nothing to gain over equal averaging.

2. **Dataset is very small:** Stacking requires cross-validation for meta-features. With small data, the CV estimates are noisy, and the meta-learner might overfit these noisy meta-features. Voting has no such risk.

3. **Base models are well-calibrated:** If all models produce well-calibrated probabilities, soft voting is already near-optimal for combining them.

4. **Models are few and similar:** With 2-3 similar models, there aren't enough meta-features for stacking's meta-learner to learn meaningful patterns.

**Stacking exceeds voting when:**

1. **Models have region-specific expertise:** Model A is better for young patients, Model B for older patients. The meta-learner can learn this routing.

2. **Models have different calibration:** One model's "0.9" means 90% probability, another's "0.9" means 75%. The meta-learner adjusts for this.

3. **Non-linear interactions exist:** "When A says 1 and B says 0, trust C" — voting can't capture this, stacking can.

4. **Enough data exists** for reliable cross-validated meta-feature generation.
</details>

**Q6.** Design a voting ensemble for a real-time fraud detection system. Consider accuracy, latency, and the need for calibrated probabilities. Justify your choices.

<details>
<summary>Answer</summary>

**System Design:**

**Base Models (chosen for diversity + speed):**
1. **Logistic Regression** (fast, linear decision boundary, naturally calibrated probabilities)
2. **LightGBM** (fast inference, captures non-linear patterns, handles missing values)
3. **Random Forest (50 trees)** (moderate speed, robust, different from boosting)

**Not included:** SVM (slow for large data), KNN (slow at prediction time), deep neural nets (high latency).

**Voting Strategy:** Weighted Soft Voting
- Weights based on validation AUC: LightGBM likely gets highest weight
- Soft voting for calibrated probability output (needed for risk scoring)

**Architecture:**
```
Transaction → [Feature Extraction]
              → LR predict_proba()     (< 1ms)
              → LightGBM predict()     (< 2ms)
              → RF predict_proba()     (< 3ms)
              → Weighted Average       (< 0.1ms)
              → Risk Score (0-1)
              → Threshold → Block/Allow
```

**Total latency:** ~5ms (parallel execution possible: ~3ms)

**Calibration:** Apply Platt scaling or isotonic regression to each base model's probabilities before averaging, ensuring the final risk score is well-calibrated for downstream decision-making (e.g., "transactions with score > 0.8 are reviewed").

**Fallback:** If any model fails, the system falls back to the remaining models (graceful degradation — a key advantage of voting ensembles in production).
</details>

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← 8.7 Stacking](./07-stacking.md) | [Unit 8: Ensemble Methods](../README.md) | [Unit 9 →](../../09-Model-Evaluation/) |

---

*© 2025 AIML Study Notes. Unit 8: Ensemble Methods — Chapter 8: Voting Classifiers.*
