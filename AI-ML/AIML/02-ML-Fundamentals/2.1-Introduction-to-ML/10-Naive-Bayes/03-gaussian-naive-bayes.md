# 📘 Chapter 3: Gaussian Naive Bayes

> **Unit 10: Naive Bayes** | **Section 3 of 7**
> Naive Bayes for continuous features using the Gaussian (normal) distribution. Learn how to estimate parameters via MLE, walk through complete numerical classification, implement with scikit-learn, and compare with Linear Discriminant Analysis.

---

## 📑 Table of Contents

1. [From Discrete to Continuous Features](#1-from-discrete-to-continuous-features)
2. [The Gaussian Assumption](#2-the-gaussian-assumption)
3. [Parameter Estimation (MLE)](#3-parameter-estimation-mle)
4. [Classification Procedure](#4-classification-procedure)
5. [Complete Worked Example](#5-complete-worked-example)
6. [Decision Boundary Geometry](#6-decision-boundary-geometry)
7. [GaussianNB in scikit-learn](#7-gaussiannb-in-scikit-learn)
8. [When to Use Gaussian NB](#8-when-to-use-gaussian-nb)
9. [Comparison with LDA](#9-comparison-with-lda)
10. [Python Implementation](#10-python-implementation)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)

---

## 1. From Discrete to Continuous Features

In [Chapter 2](02-naive-assumption.md), we applied Naive Bayes to discrete features where P(xᵢ|y) could be estimated by counting. But what about **continuous features** like height, weight, temperature, or salary?

```
    Discrete features (counting works):
    
    P(Color=Red | Apple) = count(Red Apples) / count(Apples) = 5/8 = 0.625
    
    ─────────────────────────────────────────────────────────────────
    
    Continuous features (counting doesn't work):
    
    P(Height=175.3cm | Male) = ???
    
    Any specific real value has probability ≈ 0!
    We need a PROBABILITY DENSITY FUNCTION instead.
```

### Solution: Assume a Distribution

```
    ┌──────────────────────────────────────────────────────────┐
    │  For each feature xᵢ and each class y = c:              │
    │                                                          │
    │  Assume P(xᵢ | y = c) follows a known distribution     │
    │                                                          │
    │  Most common choice: GAUSSIAN (Normal) Distribution      │
    │                                                          │
    │  P(xᵢ | y = c) = N(μᵢc, σ²ᵢc)                         │
    │                                                          │
    │  Each feature has its OWN mean and variance              │
    │  for EACH class.                                         │
    └──────────────────────────────────────────────────────────┘
```

---

## 2. The Gaussian Assumption

### 2.1 The Gaussian PDF

For feature xᵢ given class y = c:

```
    ╔══════════════════════════════════════════════════════════════╗
    ║                                                              ║
    ║                        1                (xᵢ - μᵢc)²         ║
    ║  P(xᵢ|y=c) = ─────────────── · exp( - ───────────── )      ║
    ║               √(2π · σ²ᵢc)              2 · σ²ᵢc           ║
    ║                                                              ║
    ║  Where:                                                      ║
    ║    μᵢc = mean of feature i for class c                      ║
    ║    σ²ᵢc = variance of feature i for class c                 ║
    ║                                                              ║
    ╚══════════════════════════════════════════════════════════════╝
```

### 2.2 Visual: Per-Class Gaussians

```
    Feature x₁ (e.g., Height):
    
    P(x₁|y)
      │
      │     ╭──╮                 Class 0 (Female)
      │    ╱    ╲                μ₁₀ = 162, σ₁₀ = 6
      │   ╱      ╲     ╭──╮     Class 1 (Male)
      │  ╱        ╲   ╱    ╲    μ₁₁ = 175, σ₁₁ = 7
      │ ╱          ╲ ╱      ╲
      │╱            ╳        ╲
      │            ╱ ╲        ╲
      │           ╱   ╲        ╲
      │──────────╱─────╲────────╲────→ x₁
              150    162  170  175   190
                       ↑         ↑
                      μ₁₀       μ₁₁
    
    At a given height x₁, we compute the density under EACH class
    and use the higher one (weighted by prior) for classification.
```

### 2.3 Multiple Features — Independent Gaussians

Under the naive assumption, each feature has its own independent Gaussian:

```
    Class c has parameters:
    
    Feature 1: N(μ₁c, σ²₁c)   ← own mean and variance
    Feature 2: N(μ₂c, σ²₂c)   ← own mean and variance
    Feature 3: N(μ₃c, σ²₃c)   ← own mean and variance
    ...
    Feature n: N(μₙc, σ²ₙc)   ← own mean and variance
    
    Total parameters per class: 2n (one μ and one σ² per feature)
    Total parameters: 2 × n × K  (K classes)
    
    Example: 10 features, 3 classes → 2 × 10 × 3 = 60 parameters
```

---

## 3. Parameter Estimation (MLE)

### 3.1 Maximum Likelihood Estimates

For each feature i and class c, compute from training data:

```
    ╔═══════════════════════════════════════════════════════════╗
    ║                                                           ║
    ║  Mean (MLE):                                             ║
    ║                    1                                      ║
    ║    μ̂ᵢc = ────── Σ xᵢⱼ        (average of feature i     ║
    ║            Nc   j∈c            for samples in class c)    ║
    ║                                                           ║
    ║  Variance (MLE):                                         ║
    ║                    1                                      ║
    ║    σ̂²ᵢc = ────── Σ (xᵢⱼ - μ̂ᵢc)²   (variance of       ║
    ║            Nc   j∈c                   feature i in c)     ║
    ║                                                           ║
    ║  Prior:                                                   ║
    ║    P̂(y=c) = Nc / N                                       ║
    ║                                                           ║
    ║  Where Nc = number of training samples in class c        ║
    ║        N  = total number of training samples              ║
    ║                                                           ║
    ╚═══════════════════════════════════════════════════════════╝
```

### 3.2 Step-by-Step Estimation Process

```
    Training Data:
    
    Sample  Feature₁  Feature₂  Class
    ──────  ────────  ────────  ─────
      1       5.1       3.5      A
      2       4.9       3.0      A
      3       7.0       3.2      B
      4       6.4       3.2      B
      5       5.0       3.6      A
      6       6.9       3.1      B
    
    ─────────────────────────────────────
    
    For Class A (samples 1, 2, 5):
      μ₁A = (5.1 + 4.9 + 5.0) / 3 = 5.000
      σ²₁A = [(5.1-5)² + (4.9-5)² + (5.0-5)²] / 3 = 0.00667
      
      μ₂A = (3.5 + 3.0 + 3.6) / 3 = 3.367
      σ²₂A = [(3.5-3.367)² + (3.0-3.367)² + (3.6-3.367)²] / 3 = 0.0689
    
    For Class B (samples 3, 4, 6):
      μ₁B = (7.0 + 6.4 + 6.9) / 3 = 6.767
      σ²₁B = [(7.0-6.767)² + (6.4-6.767)² + (6.9-6.767)²] / 3 = 0.0689
      
      μ₂B = (3.2 + 3.2 + 3.1) / 3 = 3.167
      σ²₂B = [(3.2-3.167)² + (3.2-3.167)² + (3.1-3.167)²] / 3 = 0.00222
    
    Priors:
      P(A) = 3/6 = 0.500
      P(B) = 3/6 = 0.500
```

---

## 4. Classification Procedure

### 4.1 The Algorithm

```
    ┌─────────────────────────────────────────────────────────────┐
    │           GAUSSIAN NAIVE BAYES CLASSIFICATION              │
    │                                                             │
    │  TRAINING:                                                  │
    │  For each class c ∈ {C₁, ..., Cₖ}:                        │
    │    1. Compute prior: P(c) = Nc / N                         │
    │    2. For each feature i = 1, ..., n:                      │
    │       a. Compute μᵢc = mean of feature i in class c       │
    │       b. Compute σ²ᵢc = variance of feature i in class c  │
    │                                                             │
    │  PREDICTION for new point x = (x₁, ..., xₙ):             │
    │    For each class c:                                       │
    │      1. Start with log P(c)                               │
    │      2. For each feature i:                                │
    │         Add log N(xᵢ; μᵢc, σ²ᵢc)                        │
    │      3. score(c) = log P(c) + Σ log P(xᵢ|c)             │
    │    Return: class with highest score                        │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

### 4.2 Using Log Probabilities

In practice, we use **log probabilities** to avoid numerical underflow:

```
    Instead of:
        score(c) = P(c) × Π P(xᵢ|c)     ← product of many small numbers → underflow!
    
    We compute:
        log_score(c) = log P(c) + Σ log P(xᵢ|c)     ← sum of log values → stable!
    
    For Gaussian:
        log P(xᵢ|c) = -½ log(2π σ²ᵢc) - (xᵢ - μᵢc)² / (2σ²ᵢc)
    
    The argmax is the same because log is monotonically increasing.
```

---

## 5. Complete Worked Example

### Problem: Classify a Student's Grade

Features: Study Hours (x₁), Sleep Hours (x₂)
Classes: Pass (P), Fail (F)

**Training Data:**

| Student | Study Hours (x₁) | Sleep Hours (x₂) | Grade |
|---------|-------------------|-------------------|-------|
| 1 | 6.0 | 7.0 | Pass |
| 2 | 7.5 | 8.0 | Pass |
| 3 | 5.5 | 6.5 | Pass |
| 4 | 8.0 | 7.5 | Pass |
| 5 | 2.0 | 5.0 | Fail |
| 6 | 1.5 | 9.0 | Fail |
| 7 | 3.0 | 6.0 | Fail |
| 8 | 2.5 | 4.5 | Fail |

**Classify: New student with x₁ = 5.0, x₂ = 6.0**

### Step 1: Compute Priors

```
    P(Pass) = 4/8 = 0.500
    P(Fail) = 4/8 = 0.500
```

### Step 2: Compute Parameters (μ and σ²) for Each Class

**Class Pass (samples 1-4):**
```
    Study Hours (x₁):
      μ₁P = (6.0 + 7.5 + 5.5 + 8.0) / 4 = 27.0 / 4 = 6.750
      σ²₁P = [(6-6.75)² + (7.5-6.75)² + (5.5-6.75)² + (8-6.75)²] / 4
           = [0.5625 + 0.5625 + 1.5625 + 1.5625] / 4
           = 4.25 / 4 = 1.0625
    
    Sleep Hours (x₂):
      μ₂P = (7.0 + 8.0 + 6.5 + 7.5) / 4 = 29.0 / 4 = 7.250
      σ²₂P = [(7-7.25)² + (8-7.25)² + (6.5-7.25)² + (7.5-7.25)²] / 4
           = [0.0625 + 0.5625 + 0.5625 + 0.0625] / 4
           = 1.25 / 4 = 0.3125
```

**Class Fail (samples 5-8):**
```
    Study Hours (x₁):
      μ₁F = (2.0 + 1.5 + 3.0 + 2.5) / 4 = 9.0 / 4 = 2.250
      σ²₁F = [(2-2.25)² + (1.5-2.25)² + (3-2.25)² + (2.5-2.25)²] / 4
           = [0.0625 + 0.5625 + 0.5625 + 0.0625] / 4
           = 1.25 / 4 = 0.3125
    
    Sleep Hours (x₂):
      μ₂F = (5.0 + 9.0 + 6.0 + 4.5) / 4 = 24.5 / 4 = 6.125
      σ²₂F = [(5-6.125)² + (9-6.125)² + (6-6.125)² + (4.5-6.125)²] / 4
           = [1.2656 + 8.2656 + 0.0156 + 2.6406] / 4
           = 12.1875 / 4 = 3.0469
```

### Step 3: Parameter Summary

```
    ┌──────────────────────────────────────────────────────┐
    │              ESTIMATED PARAMETERS                    │
    │                                                      │
    │  Class    Feature          μ        σ²       σ      │
    │  ─────   ──────────    ──────    ──────   ──────    │
    │  Pass    Study Hours    6.750     1.0625   1.0308   │
    │  Pass    Sleep Hours    7.250     0.3125   0.5590   │
    │  Fail    Study Hours    2.250     0.3125   0.5590   │
    │  Fail    Sleep Hours    6.125     3.0469   1.7456   │
    │                                                      │
    │  Priors: P(Pass) = 0.50,  P(Fail) = 0.50           │
    └──────────────────────────────────────────────────────┘
```

### Step 4: Compute Likelihoods for Test Point (5.0, 6.0)

**P(x₁=5.0 | Pass):**
```
                        1                   (5.0 - 6.75)²
    P(x₁|P) = ──────────────────── · exp( - ──────────── )
               √(2π × 1.0625)               2 × 1.0625

             =     1      · exp( - 3.0625 )
               ────────            ────────
               2.5849               2.125

             =  0.3868  ×  exp(-1.4412)

             =  0.3868  ×  0.2367

             =  0.0916
```

**P(x₂=6.0 | Pass):**
```
                        1                   (6.0 - 7.25)²
    P(x₂|P) = ──────────────────── · exp( - ──────────── )
               √(2π × 0.3125)               2 × 0.3125

             =     1      · exp( - 1.5625 )
               ────────            ────────
               1.4014               0.625

             =  0.7135  ×  exp(-2.5000)

             =  0.7135  ×  0.0821

             =  0.0586
```

**P(x₁=5.0 | Fail):**
```
                        1                   (5.0 - 2.25)²
    P(x₁|F) = ──────────────────── · exp( - ──────────── )
               √(2π × 0.3125)               2 × 0.3125

             =     1      · exp( - 7.5625 )
               ────────            ────────
               1.4014               0.625

             =  0.7135  ×  exp(-12.1000)

             =  0.7135  ×  0.0000056

             =  0.0000040
```

**P(x₂=6.0 | Fail):**
```
                        1                   (6.0 - 6.125)²
    P(x₂|F) = ──────────────────── · exp( - ────────────── )
               √(2π × 3.0469)                2 × 3.0469

             =     1      · exp( - 0.01563 )
               ────────            ──────────
               4.3760               6.0938

             =  0.2285  ×  exp(-0.002564)

             =  0.2285  ×  0.9974

             =  0.2279
```

### Step 5: Compute Scores and Classify

```
    score(Pass) = P(Pass) × P(x₁|Pass) × P(x₂|Pass)
                = 0.500 × 0.0916 × 0.0586
                = 0.002684

    score(Fail) = P(Fail) × P(x₁|Fail) × P(x₂|Fail)
                = 0.500 × 0.0000040 × 0.2279
                = 0.000000456

    ─────────────────────────────────────────────────────────
    
    Normalize:
    total = 0.002684 + 0.000000456 = 0.002685
    
    P(Pass | x) = 0.002684 / 0.002685 = 0.99983 ≈ 99.98%
    P(Fail | x) = 0.000000456 / 0.002685 = 0.00017 ≈ 0.02%
    
    ╔═══════════════════════════════════════════════════╗
    ║  Classification: PASS (99.98% confidence)        ║
    ║                                                   ║
    ║  Why? Study Hours = 5.0 is much closer to the    ║
    ║  Pass mean (6.75) than the Fail mean (2.25),     ║
    ║  and the Fail class has very low density at 5.0   ║
    ╚═══════════════════════════════════════════════════╝
```

---

## 6. Decision Boundary Geometry

### 6.1 Where Does the Boundary Fall?

The decision boundary is where the scores are equal:

```
    P(C₁) · Π P(xᵢ|C₁) = P(C₂) · Π P(xᵢ|C₂)
    
    Taking log:
    log P(C₁) + Σ log P(xᵢ|C₁) = log P(C₂) + Σ log P(xᵢ|C₂)
```

For Gaussian NB, this produces a **quadratic** boundary in general:

```
    ┌───────────────────────────────────────────────────────────┐
    │  Equal variances across classes (σ²ᵢ₁ = σ²ᵢ₂):         │
    │    → LINEAR decision boundary (like LDA)                 │
    │                                                           │
    │  Different variances per class:                           │
    │    → QUADRATIC decision boundary (like QDA)              │
    │                                                           │
    │  Gaussian NB with different σ² per class per feature:    │
    │    → QUADRATIC boundary (but axis-aligned contours)      │
    └───────────────────────────────────────────────────────────┘
```

### 6.2 ASCII Visualization

```
    Equal variances → LINEAR boundary:
    
    x₂ │            ╱
       │   ○ ○    ╱  ● ●
       │     ○  ╱  ● ●
       │  ○   ╱  ●
       │    ╱  ●  ●
       │  ╱ ●
       │╱
       └──────────────────→ x₁
         Class 0    Class 1

    Different variances → QUADRATIC boundary:
    
    x₂ │         ╱ ╲
       │  ○ ○  ╱     ╲ ● ●
       │   ○ ╱    ●    ╲
       │   ╱   ●   ●    │
       │ ╱  ●   ●      │
       │     ●        ╱
       │            ╱
       └──────────────────→ x₁
         Class 0    Class 1
```

---

## 7. GaussianNB in scikit-learn

### 7.1 Basic Usage

```python
from sklearn.naive_bayes import GaussianNB
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report
from sklearn.datasets import load_iris
import numpy as np

# Load data
iris = load_iris()
X, y = iris.data, iris.target

# Split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42
)

# Train
gnb = GaussianNB()
gnb.fit(X_train, y_train)

# Predict
y_pred = gnb.predict(X_test)
y_proba = gnb.predict_proba(X_test)

# Evaluate
print(f"Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(f"\nClassification Report:")
print(classification_report(y_test, y_pred, target_names=iris.target_names))
```

### 7.2 Inspecting Learned Parameters

```python
# Access the learned parameters
print("Class Priors:")
for i, prior in enumerate(gnb.class_prior_):
    print(f"  P(class={i}) = {prior:.4f}")

print("\nClass Means (μᵢc):")
print(f"  {'Feature':<20}", end="")
for i in range(len(gnb.classes_)):
    print(f"  Class {i:>3}", end="")
print()
for j, name in enumerate(iris.feature_names):
    print(f"  {name:<20}", end="")
    for i in range(len(gnb.classes_)):
        print(f"  {gnb.theta_[i, j]:>7.3f}", end="")
    print()

print("\nClass Variances (σ²ᵢc):")
print(f"  {'Feature':<20}", end="")
for i in range(len(gnb.classes_)):
    print(f"  Class {i:>3}", end="")
print()
for j, name in enumerate(iris.feature_names):
    print(f"  {name:<20}", end="")
    for i in range(len(gnb.classes_)):
        print(f"  {gnb.var_[i, j]:>7.4f}", end="")
    print()
```

### 7.3 Key GaussianNB Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `priors` | None | Prior probabilities (auto-computed from data if None) |
| `var_smoothing` | 1e-9 | Added to variances for numerical stability |

### 7.4 Variance Smoothing

```python
# var_smoothing adds a small value to all variances to prevent division by zero
# Useful when a feature has (near) zero variance for some class

# Try different smoothing values
for vs in [1e-12, 1e-9, 1e-6, 1e-3, 1e-1]:
    gnb = GaussianNB(var_smoothing=vs)
    gnb.fit(X_train, y_train)
    acc = gnb.score(X_test, y_test)
    print(f"  var_smoothing={vs:.0e}: accuracy={acc:.4f}")
```

---

## 8. When to Use Gaussian NB

### 8.1 Best Use Cases

```
    ┌─────────────────────────────────────────────────────────┐
    │  ✅ USE Gaussian Naive Bayes when:                     │
    │                                                         │
    │  • Features are continuous and roughly bell-shaped      │
    │  • You need a very fast baseline classifier             │
    │  • Training data is limited (few hundreds of samples)   │
    │  • Features are approximately independent               │
    │  • Real-time prediction is required                     │
    │  • You need probabilistic outputs                      │
    │  • Multi-class problems (naturally handles K classes)   │
    │                                                         │
    ├─────────────────────────────────────────────────────────┤
    │  ❌ AVOID Gaussian Naive Bayes when:                   │
    │                                                         │
    │  • Features are heavily skewed (not Gaussian)           │
    │  • Features are strongly correlated                     │
    │  • You need the most accurate classifier possible       │
    │  • Features have complex multi-modal distributions      │
    │  • Feature interactions are important                   │
    └─────────────────────────────────────────────────────────┘
```

### 8.2 Handling Non-Gaussian Features

```
    If features aren't Gaussian, consider:
    
    1. TRANSFORM features: log, sqrt, Box-Cox → make them more Gaussian
    2. DISCRETIZE features: bin continuous values → use MultinomialNB
    3. USE kernel density estimation: sklearn's KernelDensity (advanced)
    4. MIX models: Gaussian for some features, Multinomial for others
```

---

## 9. Comparison with LDA

### 9.1 Gaussian NB vs Linear Discriminant Analysis

Both Gaussian NB and LDA assume Gaussian class-conditionals, but they differ in a key assumption:

```
    ┌──────────────────────────────────────────────────────────────┐
    │                                                              │
    │  GAUSSIAN NAIVE BAYES:                                      │
    │    • Assumes features are INDEPENDENT given class           │
    │    • Each feature has its own variance per class            │
    │    • Covariance matrix is DIAGONAL (off-diagonal = 0)      │
    │    • Fewer parameters: 2nK                                  │
    │                                                              │
    │  LINEAR DISCRIMINANT ANALYSIS (LDA):                        │
    │    • Allows features to be CORRELATED                      │
    │    • Estimates full covariance matrix (shared across classes)│
    │    • Covariance matrix has FULL structure                   │
    │    • More parameters: n(n+1)/2 + nK                        │
    │                                                              │
    └──────────────────────────────────────────────────────────────┘
```

### 9.2 Covariance Matrix Comparison

```
    Gaussian NB (diagonal covariance per class):
    
    Σ_c = ┌ σ²₁c   0     0   ┐      Only diagonal elements
          │  0    σ²₂c   0   │      (no feature correlations)
          └  0     0    σ²₃c ┘
    
    ───────────────────────────────────────
    
    LDA (full shared covariance):
    
    Σ = ┌ σ²₁    σ₁₂   σ₁₃ ┐        Full matrix
        │ σ₂₁   σ²₂    σ₂₃ │        (captures correlations)
        └ σ₃₁   σ₃₂   σ²₃  ┘        Same for all classes
```

### 9.3 Side-by-Side Comparison Table

| Aspect | Gaussian NB | LDA |
|--------|------------|-----|
| **Assumption** | Features independent \| class | Features jointly Gaussian |
| **Covariance** | Diagonal, per-class | Full, shared across classes |
| **Parameters** | O(nK) | O(n² + nK) |
| **Decision boundary** | Quadratic (general) | Linear |
| **With equal σ²** | Linear boundary | Linear boundary |
| **Training speed** | O(nN) | O(n²N) |
| **Small data** | Better (fewer params) | Worse (more params to estimate) |
| **Correlated features** | Worse (ignores correlations) | Better (models correlations) |
| **Handles many features** | Better | Worse (covariance matrix issues) |

---

## 10. Python Implementation

### 10.1 Gaussian NB from Scratch

```python
import numpy as np

class GaussianNBFromScratch:
    """Gaussian Naive Bayes classifier implemented from scratch."""
    
    def fit(self, X, y):
        self.classes = np.unique(y)
        self.n_classes = len(self.classes)
        n_samples, self.n_features = X.shape
        
        # Compute parameters for each class
        self.priors = np.zeros(self.n_classes)
        self.means = np.zeros((self.n_classes, self.n_features))
        self.variances = np.zeros((self.n_classes, self.n_features))
        
        for idx, c in enumerate(self.classes):
            X_c = X[y == c]
            self.priors[idx] = len(X_c) / n_samples
            self.means[idx] = X_c.mean(axis=0)
            self.variances[idx] = X_c.var(axis=0) + 1e-9  # smoothing
        
        return self
    
    def _log_gaussian_pdf(self, x, mean, var):
        """Log of Gaussian PDF for numerical stability."""
        return -0.5 * np.log(2 * np.pi * var) - 0.5 * ((x - mean) ** 2) / var
    
    def predict_log_proba(self, X):
        """Compute log posteriors for each class."""
        log_posteriors = np.zeros((len(X), self.n_classes))
        
        for idx in range(self.n_classes):
            log_prior = np.log(self.priors[idx])
            # Sum log likelihoods across all features
            log_likelihood = np.sum(
                self._log_gaussian_pdf(X, self.means[idx], self.variances[idx]),
                axis=1
            )
            log_posteriors[:, idx] = log_prior + log_likelihood
        
        return log_posteriors
    
    def predict(self, X):
        log_posteriors = self.predict_log_proba(X)
        return self.classes[np.argmax(log_posteriors, axis=1)]
    
    def predict_proba(self, X):
        log_posteriors = self.predict_log_proba(X)
        # Log-sum-exp trick for numerical stability
        max_log = np.max(log_posteriors, axis=1, keepdims=True)
        log_posteriors_shifted = log_posteriors - max_log
        posteriors = np.exp(log_posteriors_shifted)
        posteriors /= posteriors.sum(axis=1, keepdims=True)
        return posteriors


# ─── Test on Iris Dataset ───────────────────────────────────
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split

iris = load_iris()
X_train, X_test, y_train, y_test = train_test_split(
    iris.data, iris.target, test_size=0.3, random_state=42
)

# Our implementation
gnb_scratch = GaussianNBFromScratch()
gnb_scratch.fit(X_train, y_train)
y_pred_scratch = gnb_scratch.predict(X_test)

# sklearn implementation
from sklearn.naive_bayes import GaussianNB
gnb_sklearn = GaussianNB()
gnb_sklearn.fit(X_train, y_train)
y_pred_sklearn = gnb_sklearn.predict(X_test)

# Compare
acc_scratch = np.mean(y_pred_scratch == y_test)
acc_sklearn = np.mean(y_pred_sklearn == y_test)
match = np.mean(y_pred_scratch == y_pred_sklearn)

print(f"Scratch accuracy:  {acc_scratch:.4f}")
print(f"Sklearn accuracy:  {acc_sklearn:.4f}")
print(f"Predictions match: {match*100:.1f}%")
```

### 10.2 Complete Worked Example Verified

```python
import numpy as np

# ─── Student Grade Example from Section 5 ──────────────────
X = np.array([
    [6.0, 7.0], [7.5, 8.0], [5.5, 6.5], [8.0, 7.5],  # Pass
    [2.0, 5.0], [1.5, 9.0], [3.0, 6.0], [2.5, 4.5],  # Fail
])
y = np.array([0, 0, 0, 0, 1, 1, 1, 1])  # 0=Pass, 1=Fail

from sklearn.naive_bayes import GaussianNB
gnb = GaussianNB()
gnb.fit(X, y)

# Test point
x_test = np.array([[5.0, 6.0]])
pred = gnb.predict(x_test)
proba = gnb.predict_proba(x_test)

labels = ['Pass', 'Fail']
print(f"Test point: Study={x_test[0,0]}, Sleep={x_test[0,1]}")
print(f"Prediction: {labels[pred[0]]}")
print(f"P(Pass|x) = {proba[0,0]:.6f}")
print(f"P(Fail|x) = {proba[0,1]:.6f}")

# Show learned parameters
print(f"\nLearned Parameters:")
for i, label in enumerate(labels):
    print(f"  {label}:")
    print(f"    Prior = {gnb.class_prior_[i]:.3f}")
    print(f"    μ_study = {gnb.theta_[i,0]:.3f}, σ²_study = {gnb.var_[i,0]:.4f}")
    print(f"    μ_sleep = {gnb.theta_[i,1]:.3f}, σ²_sleep = {gnb.var_[i,1]:.4f}")
```

### 10.3 GaussianNB vs LDA Comparison

```python
import numpy as np
from sklearn.naive_bayes import GaussianNB
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.datasets import make_classification
from sklearn.model_selection import cross_val_score

print(f"{'Scenario':<35} {'GaussianNB':>12} {'LDA':>12}")
print("-" * 62)

# Scenario 1: Independent features
X1, y1 = make_classification(
    n_samples=500, n_features=10, n_informative=10,
    n_redundant=0, random_state=42
)
gnb_score = cross_val_score(GaussianNB(), X1, y1, cv=5).mean()
lda_score = cross_val_score(LinearDiscriminantAnalysis(), X1, y1, cv=5).mean()
print(f"{'Independent features':<35} {gnb_score:>12.4f} {lda_score:>12.4f}")

# Scenario 2: Correlated features
X2, y2 = make_classification(
    n_samples=500, n_features=10, n_informative=5,
    n_redundant=5, random_state=42
)
gnb_score = cross_val_score(GaussianNB(), X2, y2, cv=5).mean()
lda_score = cross_val_score(LinearDiscriminantAnalysis(), X2, y2, cv=5).mean()
print(f"{'Correlated features':<35} {gnb_score:>12.4f} {lda_score:>12.4f}")

# Scenario 3: Small sample size
X3, y3 = make_classification(
    n_samples=50, n_features=20, n_informative=10,
    n_redundant=5, random_state=42
)
gnb_score = cross_val_score(GaussianNB(), X3, y3, cv=5).mean()
lda_score = cross_val_score(LinearDiscriminantAnalysis(), X3, y3, cv=5).mean()
print(f"{'Small data (n=50, p=20)':<35} {gnb_score:>12.4f} {lda_score:>12.4f}")

# Scenario 4: High-dimensional
X4, y4 = make_classification(
    n_samples=200, n_features=100, n_informative=20,
    n_redundant=10, random_state=42
)
gnb_score = cross_val_score(GaussianNB(), X4, y4, cv=5).mean()
try:
    lda_score = cross_val_score(LinearDiscriminantAnalysis(), X4, y4, cv=5).mean()
except Exception:
    lda_score = float('nan')
print(f"{'High-dim (n=200, p=100)':<35} {gnb_score:>12.4f} {lda_score:>12.4f}")
```

---

## 11. Summary Table

| Concept | Formula / Detail | Key Insight |
|---------|-----------------|-------------|
| **Gaussian likelihood** | P(xᵢ\|c) = N(μᵢc, σ²ᵢc) | Each feature is Gaussian per class |
| **MLE for mean** | μ̂ᵢc = (1/Nc) Σ xᵢⱼ | Class-wise average of feature i |
| **MLE for variance** | σ̂²ᵢc = (1/Nc) Σ (xᵢⱼ - μ̂ᵢc)² | Class-wise variance of feature i |
| **Parameters** | 2 × n × K (mean + variance) | Linear in features and classes |
| **Log form** | log P = log prior + Σ log P(xᵢ\|c) | Avoids underflow |
| **Decision boundary** | Quadratic (general), Linear (equal σ²) | Depends on variance equality |
| **var_smoothing** | Add ε to all variances | Prevents division by zero |
| **vs LDA** | NB = diagonal cov; LDA = full cov | NB ignores correlations |
| **When to use** | Continuous, roughly Gaussian features | Fast, few parameters |
| **Strengths** | Speed, small data, interpretable | Great baseline |
| **Weaknesses** | Can't capture correlations | Poor with skewed data |

---

## 12. Revision Questions

### Q1: Parameter Estimation
**Q:** Given the following training data for Class A: feature values [2.0, 3.0, 4.0, 5.0, 6.0], compute the MLE estimates for μ and σ². Then compute P(x=3.5 | Class A).

**A:**
- μ = (2+3+4+5+6)/5 = 20/5 = **4.0**
- σ² = [(2-4)²+(3-4)²+(4-4)²+(5-4)²+(6-4)²]/5 = [4+1+0+1+4]/5 = 10/5 = **2.0**
- P(3.5|A) = (1/√(2π×2)) × exp(-(3.5-4)²/(2×2)) = (1/√12.566) × exp(-0.0625) = 0.2821 × 0.9394 = **0.2650**

---

### Q2: Classification by Hand
**Q:** Two classes with equal priors. Class 0: μ=10, σ²=4. Class 1: μ=15, σ²=9. Classify x=12 by computing both Gaussian densities and selecting the winning class.

**A:**
- P(12|C₀) = (1/√(8π)) × exp(-(12-10)²/8) = 0.1995 × exp(-0.5) = 0.1995 × 0.6065 = **0.1210**
- P(12|C₁) = (1/√(18π)) × exp(-(12-15)²/18) = 0.1330 × exp(-0.5) = 0.1330 × 0.6065 = **0.0807**
- Equal priors → compare densities: 0.1210 > 0.0807 → **Class 0** (x=12 is closer to μ₀=10 and Class 0 has smaller spread)

---

### Q3: Decision Boundary
**Q:** For two classes with equal priors, both with variance σ² but different means μ₀ and μ₁ on a single feature, derive the decision boundary. Where does it fall?

**A:** At the boundary, P(x|C₀)P(C₀) = P(x|C₁)P(C₁). With equal priors and equal σ²:
- exp(-(x-μ₀)²/(2σ²)) = exp(-(x-μ₁)²/(2σ²))
- (x-μ₀)² = (x-μ₁)²
- x = **(μ₀ + μ₁)/2** — the decision boundary is exactly at the **midpoint** of the two means, regardless of σ².

---

### Q4: Effect of Variance
**Q:** Class A has μ=50, σ²=100. Class B has μ=50, σ²=10 (same mean!). With equal priors, can Gaussian NB still distinguish them? At x=55, which class wins and why?

**A:** Yes! Even with equal means, different variances create a decision boundary. At x=55:
- P(55|A) = (1/√(200π)) × exp(-25/200) = 0.0399 × 0.8825 = **0.0352**
- P(55|B) = (1/√(20π)) × exp(-25/20) = 0.1262 × 0.2865 = **0.0362**
- **Class B wins** (barely) because its narrower distribution (σ²=10) has higher density near the mean. Far from the mean (e.g., x=70), Class A would win because its wider distribution has more density in the tails.

---

### Q5: GaussianNB vs LDA
**Q:** On a dataset where features are strongly correlated but the sample size is only 30 with 20 features, would you recommend GaussianNB or LDA? Justify your answer.

**A:** **GaussianNB**. LDA needs to estimate a 20×20 covariance matrix with 20×21/2 = 210 unique entries — far more than the 30 available samples. The covariance estimate would be singular or extremely noisy. GaussianNB only estimates 2×20 = 40 parameters per class, feasible with 30 samples. Despite ignoring the correlations (its bias), the drastically lower variance makes GaussianNB the better choice here. LDA would likely overfit catastrophically.

---

### Q6: Non-Gaussian Features
**Q:** You have a feature "income" that is heavily right-skewed (log-normal distribution). How would you handle this for Gaussian NB? Show the transformation and explain why it helps.

**A:** Apply a **log transformation**: x' = log(x). Since income is log-normal, log(income) is approximately normal/Gaussian, satisfying the GaussianNB assumption. Steps:
1. Transform: `X['income_log'] = np.log(X['income'])`
2. Drop original: use income_log instead of income
3. Fit GaussianNB on transformed feature

This works because P(log(x)|class) ≈ N(μ, σ²) when x is log-normal. The Gaussian PDF fits better, giving more reliable likelihood estimates. Alternative: use Box-Cox transformation (`scipy.stats.boxcox`) for automatic power transformation to normality.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 2: The Naive Assumption](02-naive-assumption.md) | **Chapter 3: Gaussian Naive Bayes** | [Chapter 4: Multinomial Naive Bayes →](04-multinomial-naive-bayes.md) |

### Unit 10 Chapters
1. [Bayes' Theorem Review](01-bayes-theorem-review.md)
2. [The Naive Assumption](02-naive-assumption.md)
3. **📍 Gaussian Naive Bayes** ← You are here
4. [Multinomial Naive Bayes](04-multinomial-naive-bayes.md)
5. [Bernoulli Naive Bayes](05-bernoulli-naive-bayes.md)
6. [Text Classification with Naive Bayes](06-text-classification.md)
7. [Laplace Smoothing & Practical Considerations](07-laplace-smoothing.md)

---

> *"Gaussian Naive Bayes — when you need a fast, probabilistic classifier and your features roughly follow the bell curve."*

© 2025 AI/ML Study Notes. All rights reserved.
