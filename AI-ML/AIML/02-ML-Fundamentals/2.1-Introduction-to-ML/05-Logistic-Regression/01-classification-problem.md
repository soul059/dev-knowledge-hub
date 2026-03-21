# 📘 Chapter 1: The Classification Problem

> **Unit 5: Logistic Regression** | **Section 1 of 7**
> Master the fundamental distinction between classification and regression, understand why linear regression fails for classification tasks, and explore the landscape of classification problems in machine learning.

---

## 📑 Table of Contents

1. [Regression vs Classification: The Core Distinction](#1-regression-vs-classification-the-core-distinction)
2. [Binary Classification](#2-binary-classification)
3. [Multi-Class Classification](#3-multi-class-classification)
4. [Why Linear Regression Fails for Classification](#4-why-linear-regression-fails-for-classification)
5. [The Decision Boundary Concept](#5-the-decision-boundary-concept)
6. [Real-World Classification Examples](#6-real-world-classification-examples)
7. [Python Implementation: Setting Up Classification Problems](#7-python-implementation)
8. [Summary Table](#8-summary-table)
9. [Revision Questions](#9-revision-questions)

---

## 1. Regression vs Classification: The Core Distinction

### What is Regression?

Regression predicts a **continuous numerical value**. The output can take any value within a range.

```
Regression Examples:
┌─────────────────────────────────────────────────────────┐
│  Input                    →  Output (Continuous)        │
│  ─────────────────────────────────────────────────      │
│  House features           →  Price ($150,000 - $2M)     │
│  Student study hours      →  Exam score (0 - 100)       │
│  Temperature, humidity    →  Rainfall (0mm - 500mm)     │
│  Ad spend                 →  Revenue ($0 - $1M+)        │
└─────────────────────────────────────────────────────────┘
```

### What is Classification?

Classification predicts a **discrete class label**. The output belongs to one of a finite set of categories.

```
Classification Examples:
┌─────────────────────────────────────────────────────────┐
│  Input                    →  Output (Discrete)          │
│  ─────────────────────────────────────────────────      │
│  Email text               →  Spam / Not Spam            │
│  Medical scan             →  Malignant / Benign         │
│  Image pixels             →  Cat / Dog / Bird           │
│  Transaction data         →  Fraudulent / Legitimate    │
└─────────────────────────────────────────────────────────┘
```

### Side-by-Side Comparison

```
    REGRESSION                          CLASSIFICATION
    
    y (price)                           y (class)
    │                                   │
    │          ·  ·                      │  ■ ■ ■ ■ ■ ■   Class 1
    │       ·  ·                    1.0 ─┤  ■ ■ ■ ■ ■
    │     · ·                           │
    │   ·  ·                            │
    │  · ·       ← continuous           │          ← discrete
    │ ·                                 │
    │·                              0.0 ─┤  ● ● ● ● ● ●   Class 0
    └──────────── x                     │  ● ● ● ● ●
                                        └──────────── x
    Output: any real number             Output: 0 or 1
```

### Mathematical Formulation

| Aspect | Regression | Classification |
|--------|-----------|----------------|
| **Output space** | y ∈ ℝ (continuous) | y ∈ {0, 1, ..., K-1} (discrete) |
| **Goal** | Predict quantity | Predict category |
| **Loss function** | MSE, MAE | Log loss, Hinge loss |
| **Evaluation** | R², RMSE | Accuracy, F1, AUC-ROC |
| **Model output** | Direct prediction | Probability → thresholding |

---

## 2. Binary Classification

Binary classification is the simplest and most fundamental classification task — predicting one of **exactly two classes**.

### Formal Definition

Given a feature vector **x** ∈ ℝⁿ, predict a label y ∈ {0, 1}:

```
y = {  0  (negative class)
    {  1  (positive class)
```

**Convention:** The class of interest is typically labeled as **1** (positive), and the other as **0** (negative).

### The Probability Perspective

Instead of directly predicting 0 or 1, a good classifier estimates the **probability** that an input belongs to the positive class:

```
P(y = 1 | x) = probability that x belongs to class 1

Decision Rule:
    If P(y = 1 | x) ≥ 0.5  →  predict ŷ = 1
    If P(y = 1 | x) < 0.5  →  predict ŷ = 0
```

This probabilistic approach gives us **confidence** in predictions, not just labels.

### Visual Representation

```
Binary Classification: Tumor Detection
    
    Tumor Size (x)
    ├───────┼───────┼───────┼───────┼───────┤
    1       2       3       4       5       6  cm
    
    ●       ●       ●       ■       ■       ■
    Benign  Benign  Benign  Malign  Malign  Malign
    (y=0)   (y=0)   (y=0)   (y=1)   (y=1)   (y=1)
    
    P(malignant):
    1.0 ┤                          ■───■───■
        │                      ╱
    0.5 ┤─ ─ ─ ─ ─ ─ ─ ─ ─ ╳ ─ ─ ─ ─ ─ ─ ─  threshold
        │                ╱
    0.0 ┤  ●───●───●
        └───────┼───────┼───────┼───────┼────
                1       2       3       4   tumor size
                            ↑
                    decision boundary
```

### Examples of Binary Classification

| Domain | Positive Class (1) | Negative Class (0) |
|--------|-------------------|-------------------|
| Email filtering | Spam | Not spam |
| Medical diagnosis | Disease present | Disease absent |
| Credit scoring | Default | No default |
| Sentiment analysis | Positive review | Negative review |
| Fraud detection | Fraudulent | Legitimate |

---

## 3. Multi-Class Classification

When there are **more than two categories**, we have multi-class classification.

### Formal Definition

Given **x** ∈ ℝⁿ, predict y ∈ {0, 1, 2, ..., K-1} where K > 2.

```
Multi-Class: Handwritten Digit Recognition

    ┌─────┐  ┌─────┐  ┌─────┐       ┌─────┐
    │  0  │  │  1  │  │  2  │  ...  │  9  │
    │ ○○○ │  │  │  │  │ ──┐ │       │ ○○○ │
    │ ○ ○ │  │  │  │  │   │ │       │ ○ ○ │
    │ ○ ○ │  │  │  │  │ ┌─┘ │       │ ○○○ │
    │ ○○○ │  │  │  │  │ └── │       │   ○ │
    └─────┘  └─────┘  └─────┘       └─────┘
    
    K = 10 classes, y ∈ {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
```

### Multi-Class vs Multi-Label

```
Multi-Class (one label per sample):
┌──────────────┐
│   Image      │ → "Cat"  (exactly ONE label)
└──────────────┘

Multi-Label (multiple labels per sample):
┌──────────────┐
│   Image      │ → "Outdoor", "Sunny", "Beach"  (MULTIPLE labels)
└──────────────┘
```

### Strategies for Multi-Class (Preview)

| Strategy | Approach | # Classifiers for K classes |
|----------|----------|-----------------------------|
| **One-vs-Rest (OvR)** | Each class vs all others | K |
| **One-vs-One (OvO)** | Each pair of classes | K(K-1)/2 |
| **Softmax (Multinomial)** | Single model, K outputs | 1 |

> 📌 We'll explore these in detail in [Chapter 6: Multi-Class Classification](06-multi-class-classification.md).

---

## 4. Why Linear Regression Fails for Classification

This is the **key motivation** for logistic regression. Let's see exactly why fitting a straight line to classification data breaks down.

### Attempt: Using Linear Regression for Classification

Suppose we fit ŷ = w·x + b to binary labels (0 and 1):

```
Scenario 1: Seems to work (small dataset)

    y
    1.0 ┤           ●       ●       ●
        │         ╱
    0.5 ┤─ ─ ─ ╳ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  threshold
        │     ╱
    0.0 ┤  ●       ●       ●
        └───┼───────┼───────┼───────┼───
            1       2       3       4
    
    Rule: if ŷ ≥ 0.5 → class 1, else class 0
    This SEEMS reasonable...
```

### Problem 1: Sensitivity to Outliers

```
Scenario 2: Add ONE outlier — everything shifts!

    y
    1.0 ┤  ●    ●    ●                              ●  (outlier at x=10)
        │                    ╱ new line
    0.5 ┤─ ─ ─ ─ ─ ─ ─ ─ ╳ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
        │              ╱       boundary shifted right!
    0.0 ┤  ●    ●    ●
        └───┼────┼────┼────┼────┼────┼────┼────┼────┼──
            1    2    3    4    5    6    7    8    9   10
    
    ✗ The outlier at x=10 DRAGS the regression line,
      shifting the decision boundary to the RIGHT.
    ✗ Points that were correctly classified are now WRONG!
```

### Problem 2: Predictions Outside [0, 1]

Linear regression outputs can be **any real number**, but probabilities must be in [0, 1]:

```
    ŷ = w·x + b

    ŷ
    1.5 ┤                              · · ·  ← ŷ > 1 (impossible probability!)
    1.0 ┤─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─·─ ─ ─ ─
        │                      ·
    0.5 ┤                  ·
        │              ·
    0.0 ┤─ ─ ─ ─ ─·─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
        │      ·
   -0.5 ┤  · ·                             ← ŷ < 0 (impossible probability!)
        └──────┼───────┼───────┼───────┼────
               1       2       3       4
    
    ✗ P(y=1|x) = -0.3 → MEANINGLESS
    ✗ P(y=1|x) = 1.5  → MEANINGLESS
```

### Problem 3: Non-Convex Loss Surface

When MSE is applied to classification with a linear model:

```
    MSE with classification labels:
    
    J(w) = 1/m Σ (yi - (w·xi + b))²
    
    This creates a NON-CONVEX cost surface when
    the true relationship is non-linear:
    
    J(w)
    │
    │ ╲        ╱╲         ╱
    │  ╲      ╱  ╲       ╱     ← multiple local minima!
    │   ╲    ╱    ╲     ╱
    │    ╲  ╱      ╲   ╱
    │     ╲╱        ╲ ╱
    │   local        global
    │   minimum      minimum
    └─────────────────────── w
    
    Gradient descent may get STUCK in a local minimum!
```

### Problem 4: The Threshold is Arbitrary

```
What threshold should we use?

    If ŷ = 0.4999  →  predict 0
    If ŷ = 0.5001  →  predict 1
    
    But ŷ from linear regression is NOT a probability!
    A threshold of 0.5 has no principled justification.
```

### The Solution: Logistic Regression

We need a function that:
1. ✅ Maps any real number to [0, 1]
2. ✅ Gives a valid probability interpretation
3. ✅ Creates a convex cost function
4. ✅ Is robust to outliers
5. ✅ Has a smooth, differentiable form for gradient descent

**That function is the sigmoid**, which we'll study in [Chapter 2](02-sigmoid-function.md).

```
The Logistic Regression Idea:

    Instead of:  ŷ = w·x + b                    (linear regression)
    We use:      ŷ = σ(w·x + b) = 1/(1+e^(-(w·x+b)))  (logistic regression)
    
    Linear part:  z = w·x + b      → any real number
    Sigmoid:      ŷ = σ(z)         → squashed to [0, 1]
    Threshold:    class = (ŷ ≥ 0.5) → discrete label
```

---

## 5. The Decision Boundary Concept

### What is a Decision Boundary?

A decision boundary is the **surface in feature space** that separates different classes. Points on one side are classified as class 0, points on the other side as class 1.

```
2D Feature Space with Decision Boundary:

    x₂
    │
    │  ■ ■        
    │    ■ ■    ╱ decision boundary
    │  ■   ■  ╱
    │    ■   ╱
    │  ■   ╱  ●
    │    ╱   ● ●
    │  ╱  ●  ●
    │╱  ●   ● ●
    │  ● ●   ●
    └─────────────── x₁
    
    Left of boundary  → Class 0 (●)
    Right of boundary → Class 1 (■)
```

### Linear Decision Boundary

For logistic regression with two features:

```
z = w₁x₁ + w₂x₂ + b = 0    ← this IS the decision boundary

Rearranging:
    x₂ = -(w₁/w₂)·x₁ - b/w₂

This is a STRAIGHT LINE in 2D feature space.
```

### In Higher Dimensions

| Dimensions | Decision Boundary |
|------------|-------------------|
| 1D (1 feature) | A point |
| 2D (2 features) | A line |
| 3D (3 features) | A plane |
| nD (n features) | A hyperplane |

```
1D Decision Boundary:

    ● ● ● ● ● │ ■ ■ ■ ■ ■
    Class 0    ↑    Class 1
           boundary
           (a point)
```

---

## 6. Real-World Classification Examples

### Example 1: Spam Detection

```
┌──────────────────────────────────────────────────────┐
│  SPAM DETECTION                                      │
│                                                      │
│  Features (x):                                       │
│  ├── Word frequencies ("free", "winner", "click")    │
│  ├── Number of exclamation marks                     │
│  ├── Presence of suspicious links                    │
│  ├── Sender reputation score                         │
│  └── Email length                                    │
│                                                      │
│  Labels (y):                                         │
│  ├── 0: Legitimate email (ham)                       │
│  └── 1: Spam                                         │
│                                                      │
│  Challenge: Imbalanced data (~80% ham, ~20% spam)    │
└──────────────────────────────────────────────────────┘
```

### Example 2: Medical Diagnosis

```
┌──────────────────────────────────────────────────────┐
│  BREAST CANCER DETECTION                             │
│                                                      │
│  Features (x):                                       │
│  ├── Cell radius mean                                │
│  ├── Cell texture mean                               │
│  ├── Cell perimeter                                  │
│  ├── Cell area                                       │
│  ├── Smoothness                                      │
│  └── ... (30 features in Wisconsin dataset)          │
│                                                      │
│  Labels (y):                                         │
│  ├── 0: Benign tumor                                 │
│  └── 1: Malignant tumor                              │
│                                                      │
│  Critical: False negatives (missing cancer) are      │
│  FAR more costly than false positives!               │
└──────────────────────────────────────────────────────┘
```

### Example 3: Image Classification

```
┌──────────────────────────────────────────────────────┐
│  HANDWRITTEN DIGIT RECOGNITION (MNIST)               │
│                                                      │
│  Features (x):                                       │
│  ├── 28 × 28 = 784 pixel intensities (0-255)        │
│                                                      │
│  Labels (y):                                         │
│  ├── y ∈ {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}            │
│  │   (10 classes → multi-class classification)       │
│                                                      │
│  Even logistic regression achieves ~92% accuracy!    │
└──────────────────────────────────────────────────────┘
```

---

## 7. Python Implementation: Setting Up Classification Problems {#7-python-implementation}

### Binary Classification with Synthetic Data

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_classification, load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression, LogisticRegression
from sklearn.metrics import accuracy_score

# ─── Generate a binary classification dataset ───
np.random.seed(42)
X, y = make_classification(
    n_samples=200,
    n_features=2,
    n_informative=2,
    n_redundant=0,
    n_clusters_per_class=1,
    random_state=42
)

print(f"Feature matrix shape: {X.shape}")   # (200, 2)
print(f"Labels shape: {y.shape}")           # (200,)
print(f"Unique labels: {np.unique(y)}")     # [0 1]
print(f"Class distribution: {np.bincount(y)}")  # [100 100]
```

### Demonstrating Why Linear Regression Fails

```python
# ─── 1D Example: Tumor classification ───
np.random.seed(42)

# Generate data: small tumors benign (0), large tumors malignant (1)
X_tumor = np.array([1.0, 1.5, 2.0, 2.5, 3.0, 3.5, 4.0, 4.5, 5.0, 5.5]).reshape(-1, 1)
y_tumor = np.array([0,   0,   0,   0,   0,   1,   1,   1,   1,   1])

# Fit LINEAR regression (wrong approach)
lin_reg = LinearRegression()
lin_reg.fit(X_tumor, y_tumor)

# Fit LOGISTIC regression (correct approach)
log_reg = LogisticRegression()
log_reg.fit(X_tumor, y_tumor)

# Predictions on a range of inputs
X_test = np.linspace(0, 7, 100).reshape(-1, 1)

# Linear regression predictions (can go outside [0, 1]!)
y_lin_pred = lin_reg.predict(X_test)

# Logistic regression predictions (always in [0, 1])
y_log_pred = log_reg.predict_proba(X_test)[:, 1]

print("Linear Regression predictions at extreme values:")
print(f"  x=0.0: ŷ = {lin_reg.predict([[0.0]])[0]:.3f}")    # Negative!
print(f"  x=7.0: ŷ = {lin_reg.predict([[7.0]])[0]:.3f}")    # > 1!

print("\nLogistic Regression predictions at extreme values:")
print(f"  x=0.0: P(y=1) = {log_reg.predict_proba([[0.0]])[0][1]:.3f}")  # ~0
print(f"  x=7.0: P(y=1) = {log_reg.predict_proba([[7.0]])[0][1]:.3f}")  # ~1
```

### Outlier Sensitivity Demonstration

```python
# ─── Showing how outliers break linear regression for classification ───

# Original data (no outlier)
X_orig = np.array([1, 2, 3, 4, 5, 6]).reshape(-1, 1)
y_orig = np.array([0, 0, 0, 1, 1, 1])

# Data with outlier
X_outlier = np.array([1, 2, 3, 4, 5, 6, 15]).reshape(-1, 1)
y_outlier = np.array([0, 0, 0, 1, 1, 1,  1])

# Fit linear regression on both
lr_orig = LinearRegression().fit(X_orig, y_orig)
lr_outlier = LinearRegression().fit(X_outlier, y_outlier)

# Find decision boundaries (where ŷ = 0.5)
# ŷ = w*x + b = 0.5  →  x = (0.5 - b) / w
boundary_orig = (0.5 - lr_orig.intercept_) / lr_orig.coef_[0]
boundary_outlier = (0.5 - lr_outlier.intercept_) / lr_outlier.coef_[0]

print(f"Decision boundary WITHOUT outlier: x = {boundary_orig:.2f}")
print(f"Decision boundary WITH outlier:    x = {boundary_outlier:.2f}")
print(f"Boundary shifted by: {boundary_outlier - boundary_orig:.2f} units!")

# Logistic regression is ROBUST to this outlier
logr_orig = LogisticRegression().fit(X_orig, y_orig)
logr_outlier = LogisticRegression().fit(X_outlier, y_outlier)

# Compare logistic boundaries (threshold at P=0.5)
X_range = np.linspace(0, 16, 1000).reshape(-1, 1)
probs_orig = logr_orig.predict_proba(X_range)[:, 1]
probs_outlier = logr_outlier.predict_proba(X_range)[:, 1]

log_boundary_orig = X_range[np.argmin(np.abs(probs_orig - 0.5))][0]
log_boundary_outlier = X_range[np.argmin(np.abs(probs_outlier - 0.5))][0]

print(f"\nLogistic boundary WITHOUT outlier: x = {log_boundary_orig:.2f}")
print(f"Logistic boundary WITH outlier:    x = {log_boundary_outlier:.2f}")
print(f"Logistic boundary shifted by: {log_boundary_outlier - log_boundary_orig:.2f} units (much less!)")
```

### Real Dataset: Breast Cancer Classification

```python
# ─── Real-world binary classification ───
from sklearn.datasets import load_breast_cancer
from sklearn.preprocessing import StandardScaler

# Load dataset
data = load_breast_cancer()
X, y = data.data, data.target
feature_names = data.feature_names
target_names = data.target_names

print(f"Dataset: {data.DESCR[:100]}...")
print(f"Samples: {X.shape[0]}, Features: {X.shape[1]}")
print(f"Classes: {target_names}")
print(f"Class distribution: {dict(zip(target_names, np.bincount(y)))}")

# Split and scale
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# Train logistic regression
clf = LogisticRegression(max_iter=10000, random_state=42)
clf.fit(X_train_scaled, y_train)

# Evaluate
train_acc = accuracy_score(y_train, clf.predict(X_train_scaled))
test_acc = accuracy_score(y_test, clf.predict(X_test_scaled))

print(f"\nTrain Accuracy: {train_acc:.4f}")
print(f"Test Accuracy:  {test_acc:.4f}")

# Feature importance (top 5 most important features)
importance = np.abs(clf.coef_[0])
top_5_idx = np.argsort(importance)[-5:][::-1]
print("\nTop 5 most important features:")
for idx in top_5_idx:
    print(f"  {feature_names[idx]:30s}: |w| = {importance[idx]:.4f}")
```

### Multi-Class Classification Example

```python
# ─── Multi-class: Iris dataset ───
from sklearn.datasets import load_iris

iris = load_iris()
X_iris, y_iris = iris.data, iris.target

print(f"Classes: {iris.target_names}")        # ['setosa' 'versicolor' 'virginica']
print(f"Features: {iris.feature_names}")
print(f"Shape: {X_iris.shape}")               # (150, 4)
print(f"Labels: {np.unique(y_iris)}")         # [0 1 2]

# Logistic regression handles multi-class automatically
X_tr, X_te, y_tr, y_te = train_test_split(
    X_iris, y_iris, test_size=0.3, random_state=42, stratify=y_iris
)

clf_multi = LogisticRegression(max_iter=200, multi_class='multinomial')
clf_multi.fit(X_tr, y_te)

print(f"\nCoefficient matrix shape: {clf_multi.coef_.shape}")  # (3, 4) — one row per class
print(f"Accuracy: {accuracy_score(y_te, clf_multi.predict(X_te)):.4f}")

# Probability predictions
sample = X_te[0:1]
probs = clf_multi.predict_proba(sample)[0]
print(f"\nSample prediction probabilities:")
for name, prob in zip(iris.target_names, probs):
    print(f"  {name:15s}: {prob:.4f}")
```

---

## 8. Summary Table

| Concept | Description | Key Takeaway |
|---------|-------------|--------------|
| **Regression** | Predict continuous values | y ∈ ℝ (any real number) |
| **Classification** | Predict discrete labels | y ∈ {0, 1, ..., K-1} |
| **Binary Classification** | Two classes | y ∈ {0, 1} |
| **Multi-Class** | More than two classes | y ∈ {0, 1, ..., K-1}, K > 2 |
| **Linear Regression for Classification** | Fails! | Predictions outside [0,1], outlier sensitive |
| **Decision Boundary** | Separating surface | Points → class based on which side |
| **Probability Output** | Confidence in prediction | P(y=1\|x), threshold at 0.5 |
| **Logistic Regression** | Proper classifier | Uses sigmoid to constrain output to [0,1] |

---

## 9. Revision Questions

### Q1: Fundamental Difference
**Q:** What is the fundamental difference between regression and classification? Give one example of each where the same domain (e.g., healthcare) uses both.

**A:** Regression predicts a **continuous** numerical output (e.g., predicting a patient's blood pressure value), while classification predicts a **discrete** class label (e.g., predicting whether a patient has diabetes: yes/no). In healthcare, predicting hospital stay duration (days) is regression; predicting disease presence is classification.

---

### Q2: Why Linear Regression Fails
**Q:** List three specific reasons why linear regression is inappropriate for binary classification tasks.

**A:**
1. **Predictions outside [0,1]:** Linear regression can output values like -0.3 or 1.5, which are meaningless as probabilities.
2. **Outlier sensitivity:** A single extreme point can shift the decision boundary dramatically, misclassifying previously correct points.
3. **Non-convex loss surface:** Using MSE with binary labels creates a non-convex optimization problem with local minima, making gradient descent unreliable.

---

### Q3: Decision Boundary
**Q:** If a logistic regression model has weights w₁ = 2, w₂ = -3, and bias b = 6, write the equation of the decision boundary and find x₂ when x₁ = 3.

**A:** The decision boundary is where z = w₁x₁ + w₂x₂ + b = 0:
- 2x₁ - 3x₂ + 6 = 0
- x₂ = (2x₁ + 6) / 3
- When x₁ = 3: x₂ = (2·3 + 6) / 3 = 12/3 = **4**

---

### Q4: Binary vs Multi-Class
**Q:** A hospital wants to classify X-ray images into four categories: normal, pneumonia, tuberculosis, and lung cancer. Is this binary or multi-class classification? How many classifiers would be needed using the One-vs-Rest strategy?

**A:** This is **multi-class classification** with K = 4 classes. Using One-vs-Rest (OvR), we need **K = 4** binary classifiers: (normal vs rest), (pneumonia vs rest), (tuberculosis vs rest), (lung cancer vs rest).

---

### Q5: Probability Interpretation
**Q:** A spam classifier outputs P(spam | email) = 0.73 for a given email. What does this number mean, and what would the classifier predict using the default threshold?

**A:** The output means the model estimates a **73% probability** that the email is spam. Using the default threshold of 0.5, since 0.73 ≥ 0.5, the classifier would predict **ŷ = 1 (spam)**. The confidence is moderately high but not absolute — 27% chance it's legitimate.

---

### Q6: Real-World Considerations
**Q:** In fraud detection, only 0.1% of transactions are fraudulent. Why might accuracy be a misleading metric here? What alternative metrics could you use?

**A:** With 99.9% legitimate transactions, a model that **always predicts "legitimate"** achieves 99.9% accuracy while catching zero fraud! This is the **class imbalance problem**. Better metrics include: **Precision** (of predicted frauds, how many are real), **Recall** (of actual frauds, how many did we catch), **F1-score** (harmonic mean of precision and recall), and **AUC-ROC** (performance across all thresholds).

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| ← [Unit 4: Linear Regression](../../04-Linear-Regression/) | **Chapter 1: Classification Problem** | [Chapter 2: Sigmoid Function →](02-sigmoid-function.md) |

### Unit 5 Chapters
1. **📍 Classification Problem** ← You are here
2. [Sigmoid Function](02-sigmoid-function.md)
3. [Decision Boundary](03-decision-boundary.md)
4. [Cost Function & Log Loss](04-cost-function-log-loss.md)
5. [Gradient Descent for Logistic Regression](05-gradient-descent-logistic.md)
6. [Multi-Class Classification](06-multi-class-classification.md)
7. [Regularization](07-regularization.md)

---

> *"The greatest challenge in classification is not building the model, but understanding what the model is really learning."*

© 2025 AI/ML Study Notes. All rights reserved.
