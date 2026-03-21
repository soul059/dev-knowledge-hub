# 📘 Chapter 6: Multi-Class Classification

> **Unit 5: Logistic Regression** | **Section 6 of 7**
> Extend logistic regression beyond binary classification. Master One-vs-Rest, One-vs-One, and Softmax (Multinomial) strategies. Understand the softmax function, multi-class cross-entropy, and practical implementations with sklearn.

---

## 📑 Table of Contents

1. [From Binary to Multi-Class](#1-from-binary-to-multi-class)
2. [One-vs-Rest (OvR / OvA) Strategy](#2-one-vs-rest-ovr--ova-strategy)
3. [One-vs-One (OvO) Strategy](#3-one-vs-one-ovo-strategy)
4. [Softmax Regression (Multinomial Logistic)](#4-softmax-regression-multinomial-logistic)
5. [The Softmax Function](#5-the-softmax-function)
6. [Cross-Entropy Loss for Multi-Class](#6-cross-entropy-loss-for-multi-class)
7. [Comparison of Strategies](#7-comparison-of-strategies)
8. [sklearn Implementation](#8-sklearn-implementation)
9. [Practical Example: Iris Classification](#9-practical-example-iris-classification)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. From Binary to Multi-Class

### The Challenge

Binary logistic regression outputs a single probability P(y=1|x). But what if we have K > 2 classes?

```
Binary (K=2):                  Multi-Class (K=4):
┌──────┐                       ┌──────┐
│      │ → P(spam) = 0.85      │      │ → P(setosa) = 0.05
│Email │ → P(not spam) = 0.15  │Image │ → P(versicolor) = 0.15
│      │                       │      │ → P(virginica) = 0.70
└──────┘                       │      │ → P(other) = 0.10
  One output                   └──────┘
                                 K outputs (sum to 1)
```

### Three Main Strategies

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Strategy 1: One-vs-Rest (OvR)                           │
│  Train K binary classifiers, each: class k vs all others│
│                                                          │
│  Strategy 2: One-vs-One (OvO)                            │
│  Train K(K-1)/2 binary classifiers, one for each pair   │
│                                                          │
│  Strategy 3: Softmax (Multinomial)                       │
│  Single model with K outputs using softmax function      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 2. One-vs-Rest (OvR / OvA) Strategy

### How It Works

For K classes, train **K separate binary classifiers**. Each classifier learns to distinguish one class from all others combined.

```
4-Class Problem: {Cat, Dog, Bird, Fish}

Classifier 1: Cat vs {Dog, Bird, Fish}
    y = 1 if Cat, y = 0 otherwise
    → outputs P(Cat | x)

Classifier 2: Dog vs {Cat, Bird, Fish}
    y = 1 if Dog, y = 0 otherwise
    → outputs P(Dog | x)

Classifier 3: Bird vs {Cat, Dog, Fish}
    y = 1 if Bird, y = 0 otherwise
    → outputs P(Bird | x)

Classifier 4: Fish vs {Cat, Dog, Bird}
    y = 1 if Fish, y = 0 otherwise
    → outputs P(Fish | x)
```

### Prediction Rule

```
At prediction time, run ALL K classifiers and pick the 
class with the HIGHEST probability:

    ŷ = argmax_k  P_k(y = k | x)

Example:
    P(Cat | x)  = 0.3
    P(Dog | x)  = 0.7  ← highest!
    P(Bird | x) = 0.2
    P(Fish | x) = 0.1

    Prediction: Dog
```

### Visual: OvR Decision Regions

```
3-Class OvR with 2 features:

    x₂
    │        Classifier 1         Classifier 2         Classifier 3
    │        (▲ vs rest)          (■ vs rest)          (● vs rest)
    │
    │     ▲▲▲ │ ○○○           ○○○ │ ■■■           ○○○○○ │ ○○○
    │     ▲▲▲ │ ○○○           ○○○ │ ■■■           ○○○○○ │ ○○○
    │     ▲▲▲ │ ○○○           ○○○ │ ■■■           ○○○ │ ●●●
    │     ○○○ │ ○○○           ○○○ │ ○○○           ○○○ │ ●●●
    └────────── x₁           ────── x₁           ────── x₁

    COMBINED: pick class with highest score at each point
    
    x₂
    │  ▲ ▲ ▲ ▲ ■ ■ ■ ■
    │  ▲ ▲ ▲ ▲ ■ ■ ■ ■
    │  ▲ ▲ ▲ ╱ ■ ■ ■ ■
    │  ▲ ▲ ╱ ─ ─ ─ ─ ─ 
    │  ▲ ╱ ● ● ● ● ● ●
    │  ╱ ● ● ● ● ● ● ●
    └────────────────── x₁
```

### OvR Advantages and Disadvantages

```
✓ Advantages:
  • Simple and intuitive
  • Only K classifiers needed
  • Each classifier is a standard binary logistic regression
  • Can use any binary classifier
  • Training is parallelizable

✗ Disadvantages:
  • Class imbalance: each classifier sees ~1/K positive vs ~(K-1)/K negative
  • Probabilities are NOT calibrated (don't sum to 1)
  • Ambiguous regions where multiple classifiers say "yes"
  • Performance degrades with many classes
```

### Python Implementation: OvR from Scratch

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

class OneVsRestLogistic:
    """One-vs-Rest Logistic Regression from scratch."""
    
    def __init__(self, learning_rate=0.1, n_iterations=1000):
        self.lr = learning_rate
        self.n_iter = n_iterations
        self.classifiers = {}
        self.classes = None
    
    def _sigmoid(self, z):
        return np.where(z >= 0, 1/(1+np.exp(-z)), np.exp(z)/(1+np.exp(z)))
    
    def _fit_binary(self, X, y_binary):
        """Train one binary classifier."""
        m, n = X.shape
        w = np.zeros(n)
        b = 0.0
        
        for _ in range(self.n_iter):
            z = X @ w + b
            y_pred = self._sigmoid(z)
            error = y_pred - y_binary
            w -= self.lr * (1/m) * (X.T @ error)
            b -= self.lr * (1/m) * np.sum(error)
        
        return w, b
    
    def fit(self, X, y):
        """Train K binary classifiers."""
        self.classes = np.unique(y)
        self.classifiers = {}
        
        for k in self.classes:
            y_binary = (y == k).astype(float)  # class k vs rest
            w, b = self._fit_binary(X, y_binary)
            self.classifiers[k] = (w, b)
            print(f"  Trained classifier for class {k}: "
                  f"acc = {np.mean((self._sigmoid(X @ w + b) >= 0.5) == y_binary):.3f}")
        
        return self
    
    def predict_scores(self, X):
        """Get raw scores for each class."""
        scores = np.zeros((len(X), len(self.classes)))
        for i, k in enumerate(self.classes):
            w, b = self.classifiers[k]
            scores[:, i] = self._sigmoid(X @ w + b)
        return scores
    
    def predict(self, X):
        """Predict the class with highest score."""
        scores = self.predict_scores(X)
        return self.classes[np.argmax(scores, axis=1)]
    
    def score(self, X, y):
        return np.mean(self.predict(X) == y)

# Usage
np.random.seed(42)
X, y = make_classification(n_samples=500, n_features=5, n_informative=4,
                            n_classes=3, n_clusters_per_class=1, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

ovr = OneVsRestLogistic(learning_rate=0.1, n_iterations=2000)
ovr.fit(X_train_s, y_train)
print(f"\nOvR Test Accuracy: {ovr.score(X_test_s, y_test):.4f}")
```

---

## 3. One-vs-One (OvO) Strategy

### How It Works

Train a binary classifier for **every pair** of classes. For K classes, this requires K(K-1)/2 classifiers.

```
4-Class Problem: {Cat, Dog, Bird, Fish}
K(K-1)/2 = 4·3/2 = 6 classifiers:

    Classifier 1: Cat vs Dog
    Classifier 2: Cat vs Bird
    Classifier 3: Cat vs Fish
    Classifier 4: Dog vs Bird
    Classifier 5: Dog vs Fish
    Classifier 6: Bird vs Fish
```

### Prediction Rule: Voting

```
At prediction time, run ALL classifiers and use VOTING:

    Cat vs Dog  → Dog    (Dog gets 1 vote)
    Cat vs Bird → Cat    (Cat gets 1 vote)
    Cat vs Fish → Cat    (Cat gets 1 vote)
    Dog vs Bird → Dog    (Dog gets 1 vote)
    Dog vs Fish → Dog    (Dog gets 1 vote)
    Bird vs Fish → Bird  (Bird gets 1 vote)

    Vote count: Cat=2, Dog=3, Bird=1, Fish=0
    
    Prediction: Dog (most votes)
```

### Comparison: OvR vs OvO Classifiers

```
Number of classifiers for K classes:

    K    OvR (K)    OvO (K(K-1)/2)
    ──   ────────   ──────────────
    3       3             3
    4       4             6
    5       5            10
    10     10            45
    26     26           325
    100   100          4950

    OvO grows QUADRATICALLY — impractical for many classes!
    But each classifier trains on only 2 classes' data (smaller).
```

### OvO Advantages and Disadvantages

```
✓ Advantages:
  • Each classifier only sees 2 classes (balanced, smaller dataset)
  • May give better decision boundaries between close classes
  • More robust to class imbalance

✗ Disadvantages:
  • K(K-1)/2 classifiers — QUADRATIC growth
  • Slow prediction (must run all classifiers)
  • Voting can be ambiguous (ties)
  • Not used much in practice for logistic regression
```

---

## 4. Softmax Regression (Multinomial Logistic)

### The Idea

Instead of K separate binary classifiers, train a **single model** with K output units that directly outputs probabilities for all K classes.

```
Binary Logistic:                 Softmax (Multinomial):

    Input x                          Input x
      │                                │
      ▼                                ▼
    [z = wᵀx + b]                [z₁ = w₁ᵀx + b₁]
      │                          [z₂ = w₂ᵀx + b₂]
      ▼                          [z₃ = w₃ᵀx + b₃]
    σ(z) → P(y=1)                     │
                                       ▼
                                 softmax(z₁,z₂,z₃)
                                       │
                                 P(y=0), P(y=1), P(y=2)
                                 (sum to 1.0)
```

### The Parameters

```
For K classes and n features:

    Weight MATRIX W: (n × K)      ← K weight vectors, one per class
    Bias VECTOR b:   (K × 1)      ← K biases
    
    Total parameters: K·n + K = K(n+1)
    
    Example: 10 features, 3 classes
    W: (10 × 3) = 30 weights
    b: (3 × 1)  = 3 biases
    Total: 33 parameters
    
    Logit for class k:
    zₖ = wₖᵀx + bₖ = Σⱼ wₖⱼxⱼ + bₖ
```

---

## 5. The Softmax Function

### Definition

```
┌──────────────────────────────────────────────────────────┐
│                                                           │
│                       e^(zₖ)                              │
│  softmax(z)ₖ = ─────────────────                         │
│                 Σⱼ₌₁ᴷ e^(zⱼ)                            │
│                                                           │
│  For class k, the softmax probability is:                │
│                                                           │
│  P(y = k | x) = e^(zₖ) / (e^(z₁) + e^(z₂) + ... + e^(zₖ))│
│                                                           │
│  Properties:                                              │
│  • All outputs are in (0, 1)                             │
│  • All outputs SUM to exactly 1                          │
│  • Reduces to sigmoid for K=2                            │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

### Worked Example

```
Given logits z = [2.0, 1.0, 0.1] for 3 classes:

Step 1: Compute exponentials
    e^2.0 = 7.389
    e^1.0 = 2.718
    e^0.1 = 1.105

Step 2: Sum of exponentials
    Σ = 7.389 + 2.718 + 1.105 = 11.212

Step 3: Divide each by sum
    P(class 0) = 7.389 / 11.212 = 0.659
    P(class 1) = 2.718 / 11.212 = 0.242
    P(class 2) = 1.105 / 11.212 = 0.099

Step 4: Verify
    0.659 + 0.242 + 0.099 = 1.000 ✓

Prediction: Class 0 (highest probability)
```

### Softmax is a Generalization of Sigmoid

```
For K = 2 classes with logits z₁ and z₂:

    P(y=1) = e^z₁ / (e^z₁ + e^z₂)
           = 1 / (1 + e^(z₂-z₁))
           = σ(z₁ - z₂)

    This is exactly the sigmoid of the difference!
    
    If we define z = z₁ - z₂ (single logit):
    P(y=1) = σ(z) = 1/(1+e^(-z))
    
    Binary logistic regression IS softmax with K=2!
```

### Numerical Stability of Softmax

```
Problem: e^z can overflow for large z values.

    z = [1000, 1001, 1002]
    e^1000 = overflow!

Solution: Subtract max(z) before exponentiating.

    z' = z - max(z) = [1000-1002, 1001-1002, 1002-1002]
                     = [-2, -1, 0]
    
    e^(-2) = 0.135,  e^(-1) = 0.368,  e^0 = 1.000
    
    P = [0.135, 0.368, 1.000] / 1.503 = [0.090, 0.245, 0.665]
    
    This gives the SAME result because:
    e^(zₖ - c) / Σe^(zⱼ - c) = e^(zₖ)·e^(-c) / (Σe^(zⱼ)·e^(-c))
                                = e^(zₖ) / Σe^(zⱼ)  ✓
```

### ASCII Visualization

```
Softmax transforms logits to probabilities:

    Logits (z)              Softmax              Probabilities
    ┌────────┐           ┌──────────┐          ┌────────────┐
    │ z₁ = 3 │ ────┐     │          │     ┌──→ │ P₁ = 0.84  │
    │ z₂ = 1 │ ────┼────→│ softmax  │─────┼──→ │ P₂ = 0.11  │
    │ z₃ = 0 │ ────┘     │          │     └──→ │ P₃ = 0.04  │
    └────────┘           └──────────┘          └────────────┘
    
    Any real numbers        Normalization          Sum = 1.0
    (can be negative)       via exponentials       (valid probabilities)
    
    "Winner" z₁=3 gets most probability (0.84)
    But others still get some (not hard assignment)
```

---

## 6. Cross-Entropy Loss for Multi-Class

### The Cost Function

```
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│  For a single example with one-hot label y = [y₁,...,yₖ]:    │
│                                                               │
│  L = -Σₖ₌₁ᴷ yₖ · log(P(y=k|x))                             │
│                                                               │
│  Since y is one-hot (exactly one yₖ = 1, rest are 0):       │
│  L = -log(P(y = true_class | x))                            │
│                                                               │
│  For the entire dataset:                                      │
│                                                               │
│  J = -(1/m) Σᵢ₌₁ᵐ Σₖ₌₁ᴷ yᵢₖ · log(P(yᵢ=k|xᵢ))          │
│                                                               │
│  = -(1/m) Σᵢ₌₁ᵐ log(P(yᵢ = true_classᵢ | xᵢ))            │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Worked Example

```
3-class problem, 1 example:

True label: class 2 → one-hot: y = [0, 0, 1]

Model predictions (softmax output):
    P = [0.1, 0.2, 0.7]

Cross-entropy loss:
    L = -(0·log(0.1) + 0·log(0.2) + 1·log(0.7))
      = -log(0.7)
      = 0.357

Only the TRUE class probability matters!
The model predicted 70% for the correct class → moderate loss.

If the model predicted P = [0.01, 0.02, 0.97]:
    L = -log(0.97) = 0.030  ← very small loss (confident and correct)

If the model predicted P = [0.5, 0.4, 0.1]:
    L = -log(0.1) = 2.303   ← large loss (wrong and confident)
```

### Binary Cross-Entropy is a Special Case

```
For K = 2:
    
    y = [1-y₁, y₁]  (one-hot for binary)
    P = [1-ŷ, ŷ]
    
    L = -(1-y₁)·log(1-ŷ) - y₁·log(ŷ)
      = -[y₁·log(ŷ) + (1-y₁)·log(1-ŷ)]
    
    This is exactly the binary cross-entropy from Chapter 4!
```

### Gradient for Softmax Cross-Entropy

```
The gradient has a beautifully simple form:

    ∂L/∂zₖ = Pₖ - yₖ    (for each class k)

where Pₖ = softmax(z)ₖ is the predicted probability for class k
      yₖ = 1 if true class is k, else 0

    ∂J/∂W = (1/m) Xᵀ (P - Y)
    ∂J/∂b = (1/m) Σ(P - Y)

Same clean form as binary logistic regression!
(P is now a matrix: m × K instead of a vector: m × 1)
```

---

## 7. Comparison of Strategies

### Summary Comparison Table

```
┌──────────────────────────────────────────────────────────────────┐
│                 │ One-vs-Rest   │ One-vs-One     │ Softmax       │
│ Aspect          │ (OvR)         │ (OvO)          │ (Multinomial) │
│─────────────────│───────────────│────────────────│───────────────│
│ # Classifiers   │ K             │ K(K-1)/2       │ 1             │
│ Training data   │ All m samples │ Subset per pair│ All m samples │
│ Parameters      │ K(n+1)        │ K(K-1)/2·(n+1)│ K(n+1)        │
│ Probabilities   │ Not calibrated│ N/A (voting)   │ Calibrated    │
│ Sum to 1?       │ No            │ N/A            │ Yes           │
│ Scalability     │ Good          │ Poor for K>>   │ Good          │
│ Implementation  │ Easy          │ Easy           │ Moderate      │
│ When to use     │ Default in    │ Few classes,   │ Best overall  │
│                 │ many libraries│ SVM-like       │ for logistic  │
│ sklearn default │ ✓ (for some)  │ (for SVM)      │ ✓ (LR)        │
└──────────────────────────────────────────────────────────────────┘
```

### Number of Classifiers Growth

```
    # Classifiers
    5000│                                        ○ OvO
        │                                     ○
    4000│                                   ○
        │                                 ○
    3000│                              ○
        │                           ○
    2000│                        ○
        │                     ○
    1000│                  ○
        │              ○
     500│          ○
        │      ○
      50│  ○
      10│○●──●──●──●──●──●──●──●──●──●──●──●     ● OvR/Softmax
        └──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──
           5  10 15 20 25 30 35 40 50 60 80 100
                         K (classes)
```

---

## 8. sklearn Implementation

### LogisticRegression multi_class Parameter

```python
from sklearn.linear_model import LogisticRegression

# ─── Option 1: Multinomial (Softmax) ───
model_multinomial = LogisticRegression(
    multi_class='multinomial',   # softmax approach
    solver='lbfgs',              # supports multinomial
    max_iter=1000,
    random_state=42
)

# ─── Option 2: One-vs-Rest ───
model_ovr = LogisticRegression(
    multi_class='ovr',           # one-vs-rest
    solver='lbfgs',
    max_iter=1000,
    random_state=42
)

# ─── Option 3: Auto (sklearn chooses) ───
model_auto = LogisticRegression(
    multi_class='auto',          # multinomial if solver supports it
    solver='lbfgs',
    max_iter=1000,
    random_state=42
)
```

### Solver Compatibility

```
┌────────────────────────────────────────────────────────────┐
│ Solver      │ OvR │ Multinomial │ L1  │ L2  │ ElasticNet │
│─────────────│─────│─────────────│─────│─────│────────────│
│ 'lbfgs'     │  ✓  │     ✓       │  ✗  │  ✓  │     ✗      │
│ 'newton-cg' │  ✓  │     ✓       │  ✗  │  ✓  │     ✗      │
│ 'sag'       │  ✓  │     ✓       │  ✗  │  ✓  │     ✗      │
│ 'saga'      │  ✓  │     ✓       │  ✓  │  ✓  │     ✓      │
│ 'liblinear' │  ✓  │     ✗       │  ✓  │  ✓  │     ✗      │
└────────────────────────────────────────────────────────────┘

Default solver: 'lbfgs' (supports both OvR and multinomial)
```

### Using OneVsRestClassifier and OneVsOneClassifier

```python
from sklearn.multiclass import OneVsRestClassifier, OneVsOneClassifier

# Explicit OvR wrapper (works with ANY binary classifier)
ovr_explicit = OneVsRestClassifier(
    LogisticRegression(max_iter=1000, random_state=42)
)

# Explicit OvO wrapper
ovo_explicit = OneVsOneClassifier(
    LogisticRegression(max_iter=1000, random_state=42)
)
```

---

## 9. Practical Example: Iris Classification

### Complete Multi-Class Pipeline

```python
import numpy as np
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.multiclass import OneVsRestClassifier, OneVsOneClassifier
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix

# ─── Load data ───
iris = load_iris()
X, y = iris.data, iris.target
feature_names = iris.feature_names
target_names = iris.target_names

print("Dataset Info:")
print(f"  Samples: {X.shape[0]}, Features: {X.shape[1]}")
print(f"  Classes: {target_names}")
print(f"  Class distribution: {np.bincount(y)}")

# ─── Split and scale ───
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# ─── Train all three strategies ───
models = {
    'OvR': LogisticRegression(multi_class='ovr', max_iter=1000, random_state=42),
    'Multinomial': LogisticRegression(multi_class='multinomial', max_iter=1000, random_state=42),
    'OvO': OneVsOneClassifier(LogisticRegression(max_iter=1000, random_state=42))
}

print("\nResults:")
print(f"{'Strategy':<15} {'Train Acc':>10} {'Test Acc':>10} {'CV Mean':>10}")
print("─" * 50)

for name, model in models.items():
    model.fit(X_train_s, y_train)
    train_acc = model.score(X_train_s, y_train)
    test_acc = model.score(X_test_s, y_test)
    
    # Cross-validation
    cv_scores = cross_val_score(model, X_train_s, y_train, cv=5)
    
    print(f"{name:<15} {train_acc:>10.4f} {test_acc:>10.4f} {cv_scores.mean():>10.4f}")
```

### Detailed Analysis of Multinomial Model

```python
# ─── Analyze the multinomial (softmax) model ───
model = LogisticRegression(multi_class='multinomial', max_iter=1000, random_state=42)
model.fit(X_train_s, y_train)

# Weight matrix
print("Weight Matrix W (shape: classes × features):")
print(f"{'Class':<15} ", end="")
for fname in feature_names:
    print(f"{fname[:12]:>14}", end="")
print(f"{'bias':>10}")
print("─" * 80)

for i, cls in enumerate(target_names):
    print(f"{cls:<15} ", end="")
    for j in range(len(feature_names)):
        print(f"{model.coef_[i, j]:>14.4f}", end="")
    print(f"{model.intercept_[i]:>10.4f}")

# Probability predictions
print("\nSample predictions (first 5 test examples):")
y_proba = model.predict_proba(X_test_s[:5])
y_pred = model.predict(X_test_s[:5])

print(f"{'True':>8} {'Pred':>8} {'P(setosa)':>12} {'P(versicolor)':>14} {'P(virginica)':>14}")
print("─" * 60)
for i in range(5):
    true_name = target_names[y_test[i]]
    pred_name = target_names[y_pred[i]]
    print(f"{true_name:>8} {pred_name:>8} {y_proba[i, 0]:>12.4f} "
          f"{y_proba[i, 1]:>14.4f} {y_proba[i, 2]:>14.4f}")

# Confusion matrix
print("\nConfusion Matrix:")
cm = confusion_matrix(y_test, model.predict(X_test_s))
print(f"{'':>15} {'Pred ' + target_names[0]:>12} {'Pred ' + target_names[1]:>15} {'Pred ' + target_names[2]:>14}")
for i, cls in enumerate(target_names):
    print(f"{'True ' + cls:>15} {cm[i, 0]:>12d} {cm[i, 1]:>15d} {cm[i, 2]:>14d}")

# Classification report
print("\nClassification Report:")
print(classification_report(y_test, model.predict(X_test_s), target_names=target_names))
```

### Softmax Implementation from Scratch

```python
class SoftmaxRegression:
    """Multinomial Logistic (Softmax) Regression from scratch."""
    
    def __init__(self, learning_rate=0.1, n_iterations=1000):
        self.lr = learning_rate
        self.n_iter = n_iterations
        self.W = None  # (n, K) weight matrix
        self.b = None  # (K,) bias vector
        self.cost_history = []
        self.classes = None
    
    def _softmax(self, z):
        """Numerically stable softmax."""
        z_shifted = z - np.max(z, axis=1, keepdims=True)
        exp_z = np.exp(z_shifted)
        return exp_z / np.sum(exp_z, axis=1, keepdims=True)
    
    def _one_hot(self, y, K):
        """Convert labels to one-hot encoding."""
        m = len(y)
        one_hot = np.zeros((m, K))
        one_hot[np.arange(m), y] = 1
        return one_hot
    
    def _compute_cost(self, Y_onehot, P):
        """Cross-entropy loss."""
        m = len(Y_onehot)
        P_clipped = np.clip(P, 1e-15, 1 - 1e-15)
        return -(1/m) * np.sum(Y_onehot * np.log(P_clipped))
    
    def fit(self, X, y):
        m, n = X.shape
        self.classes = np.unique(y)
        K = len(self.classes)
        
        # Initialize parameters
        self.W = np.zeros((n, K))
        self.b = np.zeros(K)
        self.cost_history = []
        
        # One-hot encode labels
        Y = self._one_hot(y, K)
        
        for i in range(self.n_iter):
            # Forward pass
            Z = X @ self.W + self.b          # (m, K)
            P = self._softmax(Z)              # (m, K) probabilities
            
            # Cost
            cost = self._compute_cost(Y, P)
            self.cost_history.append(cost)
            
            # Gradients
            error = P - Y                     # (m, K)
            dW = (1/m) * (X.T @ error)       # (n, K)
            db = (1/m) * np.sum(error, axis=0) # (K,)
            
            # Update
            self.W -= self.lr * dW
            self.b -= self.lr * db
        
        return self
    
    def predict_proba(self, X):
        Z = X @ self.W + self.b
        return self._softmax(Z)
    
    def predict(self, X):
        proba = self.predict_proba(X)
        return self.classes[np.argmax(proba, axis=1)]
    
    def score(self, X, y):
        return np.mean(self.predict(X) == y)

# ─── Train and compare with sklearn ───
softmax_model = SoftmaxRegression(learning_rate=0.5, n_iterations=2000)
softmax_model.fit(X_train_s, y_train)

sklearn_model = LogisticRegression(multi_class='multinomial', C=1e10, max_iter=5000, random_state=42)
sklearn_model.fit(X_train_s, y_train)

print("Softmax From Scratch vs sklearn:")
print(f"  Scratch Test Acc: {softmax_model.score(X_test_s, y_test):.4f}")
print(f"  sklearn Test Acc: {sklearn_model.score(X_test_s, y_test):.4f}")
print(f"  Final Cost: {softmax_model.cost_history[-1]:.6f}")
```

---

## 10. Summary Table

| Concept | Formula / Detail | Key Insight |
|---------|-----------------|-------------|
| **OvR** | K binary classifiers | Class k vs all others |
| **OvO** | K(K-1)/2 classifiers | Every pair of classes |
| **Softmax** | P(k) = eᶻᵏ / Σeᶻʲ | Single model, K outputs |
| **Sum constraint** | Σ P(k) = 1 | Softmax guarantees this |
| **Cross-entropy** | -Σ yₖ log(Pₖ) | Only true class matters |
| **Gradient** | ∂L/∂zₖ = Pₖ - yₖ | Same clean form! |
| **One-hot encoding** | y → [0,...,1,...,0] | Label format for softmax |
| **Numerical stability** | z' = z - max(z) | Prevents exp overflow |
| **Binary special case** | Softmax(K=2) = sigmoid | Unified framework |
| **sklearn default** | multi_class='auto' | Uses multinomial if supported |
| **Weight matrix** | W: (n × K) | One weight vector per class |

---

## 11. Revision Questions

### Q1: Strategy Comparison
**Q:** For a problem with K=20 classes, how many binary classifiers are needed for OvR and OvO? Which strategy would you prefer and why?

**A:**
- OvR: **K = 20** classifiers
- OvO: **K(K-1)/2 = 20·19/2 = 190** classifiers
- Prefer **OvR or Softmax** because 190 classifiers is expensive to train and predict with. Softmax is ideal as it uses a single model, produces calibrated probabilities, and scales linearly with K.

---

### Q2: Softmax Computation
**Q:** Compute the softmax probabilities for logits z = [1.0, 2.0, 3.0]. Which class is predicted? Verify the probabilities sum to 1.

**A:**
- e¹ = 2.718, e² = 7.389, e³ = 20.086
- Sum = 2.718 + 7.389 + 20.086 = **30.193**
- P(class 0) = 2.718/30.193 = **0.0900**
- P(class 1) = 7.389/30.193 = **0.2447**
- P(class 2) = 20.086/30.193 = **0.6652**
- Sum check: 0.0900 + 0.2447 + 0.6652 = **0.9999 ≈ 1.0** ✓
- Predicted: **Class 2** (highest probability)

---

### Q3: Cross-Entropy Loss
**Q:** A 4-class softmax model predicts P = [0.1, 0.6, 0.2, 0.1] and the true class is 1. Compute the cross-entropy loss. What would the loss be if the model was perfectly confident?

**A:**
- True class is 1, so y = [0, 1, 0, 0]
- L = -Σ yₖ log(Pₖ) = -(0·log(0.1) + 1·log(0.6) + 0·log(0.2) + 0·log(0.1))
- L = **-log(0.6) = 0.5108**
- If perfectly confident: P = [0, 1, 0, 0], L = -log(1) = **0** (minimum possible loss)

---

### Q4: OvR vs Softmax
**Q:** In OvR, each classifier outputs a probability, but these probabilities don't sum to 1. Give a concrete numerical example where this causes a problem. How does softmax avoid this?

**A:** OvR with 3 classes might output: P(A) = 0.8, P(B) = 0.7, P(C) = 0.6. Sum = 2.1 ≠ 1! This makes probabilities **uninterpretable** — what does "80% class A" mean if class B is also "70%"? Even worse: P(A) = 0.2, P(B) = 0.3, P(C) = 0.1 → sum = 0.6, and no class exceeds 0.5. Softmax avoids this by using a single normalization: all outputs share one denominator (Σeᶻʲ), **guaranteeing** they sum to 1.

---

### Q5: Numerical Stability
**Q:** Why can computing softmax on z = [1000, 1001, 1002] cause numerical overflow? Show the standard fix and prove it gives the same result.

**A:** e^1000 overflows float64 (max ≈ 10³⁰⁸). Fix: subtract max(z) = 1002. New z' = [-2, -1, 0]. Now e⁻² = 0.135, e⁻¹ = 0.368, e⁰ = 1.0 (all safe). Proof: softmax(z-c)ₖ = eᶻᵏ⁻ᶜ/Σeᶻʲ⁻ᶜ = (eᶻᵏ · e⁻ᶜ)/(Σeᶻʲ · e⁻ᶜ) = eᶻᵏ/Σeᶻʲ = softmax(z)ₖ. The constant cancels.

---

### Q6: Multi-Class Gradient
**Q:** For softmax regression, the gradient is ∂L/∂zₖ = Pₖ - yₖ. Explain intuitively why this form makes sense for both the true class and the other classes.

**A:** For the **true class** (yₖ = 1): gradient = Pₖ - 1. If P is close to 1 (correct), gradient ≈ 0 (no correction needed). If P is small (wrong), gradient is large and negative → pushes zₖ higher → increases Pₖ. For **other classes** (yₖ = 0): gradient = Pₖ - 0 = Pₖ. If Pₖ is large (wrongly predicted), gradient is large and positive → pushes zₖ lower → decreases Pₖ. This naturally increases the true class probability while decreasing all others.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 5: Gradient Descent](05-gradient-descent-logistic.md) | **Chapter 6: Multi-Class Classification** | [Chapter 7: Regularization →](07-regularization.md) |

### Unit 5 Chapters
1. [Classification Problem](01-classification-problem.md)
2. [Sigmoid Function](02-sigmoid-function.md)
3. [Decision Boundary](03-decision-boundary.md)
4. [Cost Function & Log Loss](04-cost-function-log-loss.md)
5. [Gradient Descent for Logistic Regression](05-gradient-descent-logistic.md)
6. **📍 Multi-Class Classification** ← You are here
7. [Regularization](07-regularization.md)

---

> *"The softmax function is nature's way of turning competition into cooperation — every class gets a voice, but only one gets the crown."*

© 2025 AI/ML Study Notes. All rights reserved.
