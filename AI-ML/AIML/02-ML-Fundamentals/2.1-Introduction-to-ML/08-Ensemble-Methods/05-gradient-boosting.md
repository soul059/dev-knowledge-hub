# Chapter 8.5: Gradient Boosting

> **Unit 8: Ensemble Methods** — File 5 of 8

---

## Table of Contents

1. [Introduction — The Gradient Boosting Framework](#1-introduction--the-gradient-boosting-framework)
2. [Fitting Residuals — The Core Idea](#2-fitting-residuals--the-core-idea)
3. [The Additive Model](#3-the-additive-model)
4. [Learning Rate (Shrinkage)](#4-learning-rate-shrinkage)
5. [Loss Functions](#5-loss-functions)
6. [Gradient Computation](#6-gradient-computation)
7. [Regularization Techniques](#7-regularization-techniques)
8. [Step-by-Step Example: Gradient Boosting for Regression](#8-step-by-step-example-gradient-boosting-for-regression)
9. [GradientBoostingClassifier & Regressor in Scikit-Learn](#9-gradientboostingclassifier--regressor-in-scikit-learn)
10. [Applications](#10-applications)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)

---

## 1. Introduction — The Gradient Boosting Framework

**Gradient Boosting** (Friedman, 1999/2001) is a generalization of boosting that frames the problem as **gradient descent in function space**. Instead of re-weighting samples (AdaBoost), it fits each new model to the **negative gradient of the loss function** — which, for squared error loss, simplifies to fitting the **residuals**.

### Why Gradient Boosting Is So Powerful

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  AdaBoost:  Re-weight samples → limited to exponential loss  │
│                                                              │
│  Gradient Boosting: Fit negative gradients → works with      │
│                     ANY differentiable loss function!         │
│                                                              │
│  This generality is what makes Gradient Boosting the         │
│  foundation of XGBoost, LightGBM, and CatBoost.             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### High-Level Algorithm

```
F₀(x) = initial prediction (e.g., mean of y for regression)

FOR t = 1, 2, ..., T:
    1. Compute pseudo-residuals (negative gradients):
       rᵢₜ = -∂L(yᵢ, F(xᵢ))/∂F(xᵢ)  at F = Fₜ₋₁

    2. Fit a weak learner hₜ to pseudo-residuals:
       hₜ ≈ rₜ   (learn the pattern in the residuals)

    3. Update the model:
       Fₜ(x) = Fₜ₋₁(x) + η · hₜ(x)

Final: F_T(x) = F₀(x) + η·h₁(x) + η·h₂(x) + ... + η·h_T(x)
```

---

## 2. Fitting Residuals — The Core Idea

### The Intuition

```
True function: y = f(x)

Step 0: Start with a constant prediction (mean)
        F₀(x) = ȳ = 5.0

  y │         ○         ○
  8 │                              ○
    │    ○
  5 │──────────────────────────────── F₀ = 5.0 (mean)
    │              ○
  2 │                         ○
    └─────────────────────────────── x

  Residuals: r₁ = y - F₀(x)
  
  r │         ○         ○
  3 │                              ○
    │    ○
  0 │──────────────────────────────── zero line
    │              ○
 -3 │                         ○
    └─────────────────────────────── x

Step 1: Fit h₁(x) to residuals r₁
        F₁(x) = F₀(x) + η · h₁(x)

  y │         ○         ○
  8 │              ___/‾‾‾‾‾‾‾‾‾‾‾‾ F₁ (better!)
    │    ○    /‾‾‾
  5 │───/────
    │ /            ○
  2 │                         ○
    └─────────────────────────────── x

  New residuals r₂ = y - F₁(x) are SMALLER

Step 2: Fit h₂(x) to r₂
        F₂(x) = F₁(x) + η · h₂(x)

  y │         ○         ○
  8 │           .....○.............. F₂ (even better!)
    │    ○....´       
  5 │.....      
    │              ○.
  2 │                .........○
    └─────────────────────────────── x

After many rounds: F_T ≈ f  (close to true function!)
```

### Why Residuals = Negative Gradient (for MSE)

```
Loss function:  L(y, F) = ½(y - F)²

Negative gradient w.r.t. F:
  -∂L/∂F = -(-(y - F)) = y - F = residual!

So for MSE loss, the negative gradient IS the residual.
This is why "fitting residuals" and "gradient descent in function space" 
are equivalent for squared error.
```

---

## 3. The Additive Model

### Structure of Gradient Boosting

```
F_T(x) = F₀(x) + Σ ηₜ · hₜ(x)
                  t=1

Where:
  F₀(x) = initial prediction (constant)
  hₜ(x) = t-th weak learner (typically a shallow decision tree)
  ηₜ    = learning rate (step size) for round t
  T     = total number of boosting rounds
```

### Comparison with Neural Network Layers

```
Neural Network:                    Gradient Boosting:
  Input → Layer1 → Layer2 → Out     F₀ → +h₁ → +h₂ → ... → +h_T → Out
  
  Layers transform features          Trees correct residuals
  Weights updated via backprop        New tree added each round
  Fixed architecture                  Architecture grows
  All layers updated together         Only newest tree trained
```

### Forward Stagewise Additive Modeling

Gradient Boosting is a special case of **forward stagewise additive modeling**:

```
At each stage t, we find hₜ and ηₜ that minimize:

(hₜ, ηₜ) = argmin  Σ L(yᵢ, Fₜ₋₁(xᵢ) + η · h(xᵢ))
            h,η    i=1

Key property: We NEVER go back and modify previous learners.
Each round adds ONE new component and moves on.
This is "greedy" optimization — efficient but not globally optimal.
```

---

## 4. Learning Rate (Shrinkage)

### The Role of the Learning Rate

The learning rate η (also called **shrinkage**) scales down the contribution of each tree:

```
Without shrinkage (η = 1.0):
  Fₜ = Fₜ₋₁ + hₜ          ← Full correction each round

With shrinkage (η = 0.1):
  Fₜ = Fₜ₋₁ + 0.1 · hₜ    ← Only 10% correction each round
```

### Why Smaller Learning Rate Is Better

```
  Test Error
    │
    │\
    │ \ η = 1.0
    │  \         .
    │   \      .´  ← overfits!
    │    \   .´
    │     \.´──────────── η = 0.1 (with more trees)
    │      \.
    │       `─.──────────── η = 0.01 (even more trees)
    │          `──────────  
    │
    └──────────────────────────── Boosting Rounds
    
  Smaller η = slower learning, but BETTER generalization
  Trade-off: smaller η → need MORE trees (more computation)
```

### The Learning Rate – n_estimators Trade-off

```
Rule of thumb (Friedman, 2001):

  η × T ≈ constant (for similar performance)

  η = 1.0,  T = 100  → fast but may overfit
  η = 0.1,  T = 1000 → slower but better generalization
  η = 0.01, T = 10000 → slowest but often best generalization

Practical recommendation:
  • Set η = 0.01 to 0.1
  • Use early stopping to find optimal T
  • More computation → better model (within limits)
```

### Mathematical Justification

Shrinkage creates a form of **regularization** by constraining the model's learning trajectory:

```
Without shrinkage:
  F₁₀₀ = F₀ + h₁ + h₂ + ... + h₁₀₀
  Each tree can make large corrections → overfitting risk

With shrinkage (η = 0.1):
  F₁₀₀₀ = F₀ + 0.1·h₁ + 0.1·h₂ + ... + 0.1·h₁₀₀₀
  Each tree contributes a small amount
  The PATH through function space is smoother
  This is analogous to L2 regularization in gradient descent
```

---

## 5. Loss Functions

### Common Loss Functions for Gradient Boosting

#### Regression Losses

```
1. Mean Squared Error (MSE / Least Squares):
   L(y, F) = ½(y - F)²
   Gradient: -(y - F) = F - y
   Pseudo-residual: y - F (the plain residual)
   Properties: Standard, sensitive to outliers

2. Mean Absolute Error (MAE / Least Absolute Deviation):
   L(y, F) = |y - F|
   Gradient: -sign(y - F)
   Pseudo-residual: sign(y - F)  (just the direction, ±1)
   Properties: Robust to outliers, but harder to optimize

3. Huber Loss (combines MSE and MAE):
   L(y, F) = ½(y-F)²           if |y-F| ≤ δ
             δ|y-F| - ½δ²       if |y-F| > δ
   Properties: Robust to outliers while smooth near zero
```

#### Classification Losses

```
4. Log Loss (Binary Cross-Entropy / Deviance):
   L(y, F) = log(1 + exp(-2yF))    (y ∈ {-1, +1})
   
   Or equivalently (y ∈ {0, 1}):
   L(y, F) = -[y·log(p) + (1-y)·log(1-p)]
   where p = σ(F) = 1/(1 + e⁻ᶠ)
   
   Pseudo-residual: y - p  (true label minus predicted probability)
   Properties: Standard for classification, well-calibrated probabilities

5. Exponential Loss:
   L(y, F) = exp(-yF)
   Properties: Equivalent to AdaBoost, sensitive to outliers
```

### Loss Function Comparison

```
  Loss
    │
  4 │\                                    
    │ \      MSE: (y-F)²/2
  3 │  \     MAE: |y-F|
    │   \    Huber: combination
  2 │    \   
    │  ___\ \___
  1 │ /   .\ \  \___    ← MAE (straight lines)
    │/   . ··\·····____  ← Huber (smooth transition)
  0 │···´     `──────── ← MSE (parabola, steep for outliers)
    └──┬──┬──┬──┬──┬──► y - F (residual)
      -3 -2 -1  0  1  2  3
    
  MSE:   Penalizes large errors quadratically → outlier sensitive
  MAE:   Constant penalty for direction → outlier robust
  Huber: Best of both — quadratic near 0, linear for outliers
```

### Choosing the Right Loss Function

| Scenario | Recommended Loss | Why |
|----------|-----------------|-----|
| Clean regression data | MSE | Simple, well-understood |
| Regression with outliers | Huber or MAE | Robust to extreme values |
| Binary classification | Log loss (deviance) | Well-calibrated probabilities |
| Ranking problems | Pairwise/Listwise losses | Captures relative ordering |
| Imbalanced classification | Weighted log loss | Adjusts for class frequencies |

---

## 6. Gradient Computation

### General Gradient Boosting Algorithm (Detailed)

```
ALGORITHM: Gradient Boosting Machine (GBM)
──────────────────────────────────────────

INPUT:  Training data {(xᵢ, yᵢ)}ⁿᵢ₌₁
        Differentiable loss function L(y, F)
        Number of iterations T
        Learning rate η
        Tree depth d

STEP 1: Initialize
        F₀(x) = argmin_γ Σ L(yᵢ, γ)
                         i=1

FOR t = 1 to T:

    STEP 2: Compute pseudo-residuals (negative gradients)
            For i = 1, ..., n:
            
            rᵢₜ = -[∂L(yᵢ, F(xᵢ)) / ∂F(xᵢ)]
                                                F=Fₜ₋₁

    STEP 3: Fit a regression tree hₜ to {(xᵢ, rᵢₜ)}ⁿᵢ₌₁
            The tree has J leaf nodes (regions R₁ₜ, R₂ₜ, ..., R_Jt)

    STEP 4: For each leaf j = 1, ..., J:
            Compute optimal leaf value:
            
            γⱼₜ = argmin_γ  Σ    L(yᵢ, Fₜ₋₁(xᵢ) + γ)
                           xᵢ∈Rⱼₜ

    STEP 5: Update model:
            Fₜ(x) = Fₜ₋₁(x) + η · Σ γⱼₜ · I(x ∈ Rⱼₜ)
                                    j=1

END FOR

OUTPUT: F_T(x)
```

### Pseudo-Residuals for Each Loss Function

| Loss Function | L(y, F) | Pseudo-residual rᵢ = -∂L/∂F |
|---------------|---------|-------------------------------|
| MSE | ½(y-F)² | y - F |
| MAE | \|y-F\| | sign(y - F) |
| Huber | (see above) | y-F if \|y-F\|≤δ, else δ·sign(y-F) |
| Log loss (binary) | log(1+e⁻²ʸᶠ) | 2y/(1+e²ʸᶠ) |
| Log loss (p form) | -y·log(p)-(1-y)·log(1-p) | y - p |

### Visualization: Gradient Descent in Function Space

```
                Function Space (infinite dimensional)
                
    L(F)  │                        
          │  *                     ← F₀ (initial guess)
          │    *                   
          │      *    *            ← F₁, F₂ (each step follows gradient)
          │            *           
          │              *  *      ← F₃, F₄
          │                  * *   ← approaching minimum
          │                    *── ← F_T (final model)
          │                        
          └────────────────────── "Direction" in function space
          
    Each * represents the ensemble after adding one more tree.
    Each step moves in the direction of steepest descent
    (the negative gradient of the loss).
```

---

## 7. Regularization Techniques

### The Three Pillars of GBM Regularization

```
┌──────────────────────────────────────────────────────────────────┐
│                REGULARIZATION IN GRADIENT BOOSTING               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. SHRINKAGE (Learning Rate η)                                  │
│     • Scale down each tree's contribution                        │
│     • η = 0.01 to 0.3 (smaller = more regularization)           │
│     • Trade-off: smaller η → need more trees                    │
│                                                                  │
│  2. TREE CONSTRAINTS                                             │
│     • max_depth: limit tree depth (3-8 typical)                  │
│     • min_samples_leaf: minimum samples in leaf                  │
│     • min_samples_split: minimum samples to split                │
│     • max_leaf_nodes: limit number of leaves                     │
│                                                                  │
│  3. SUBSAMPLING (Stochastic Gradient Boosting)                   │
│     • subsample: fraction of rows per tree (0.5-0.8)             │
│     • Column subsampling: fraction of features per tree          │
│     • Adds randomness → reduces correlation → better             │
│                                                                  │
│  Bonus: EARLY STOPPING                                           │
│     • Monitor validation error                                   │
│     • Stop when validation error stops improving                 │
│     • Most practical regularization technique                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Stochastic Gradient Boosting (Friedman, 2002)

```
Standard GBM:
  Each tree sees ALL training data → can overfit

Stochastic GBM:
  Each tree sees a RANDOM SUBSAMPLE → adds noise → regularizes

  At each round t:
    1. Randomly sample fraction f of training data (without replacement)
    2. Compute pseudo-residuals only on the subsample
    3. Fit tree only on the subsample
    
  Typical f = 0.5 to 0.8

Benefits:
  ✓ Reduces overfitting (like dropout in neural networks)
  ✓ Faster training (less data per tree)
  ✓ Often improves generalization
  ✓ Can be combined with column subsampling
```

### Early Stopping

```
  Error
    │
    │ \
    │  \  Training Error (keeps decreasing)
    │   \
    │    \___________________________________________
    │                    
    │     \
    │      \ Test Error
    │       \
    │        \______
    │               \________  ← Minimum test error
    │                        \_______  ← Overfitting begins
    │
    └──┬──────────────┬──────────┬──────────── Rounds
       0          STOP HERE    T_max
                 (early stop)

Implementation:
  1. Split data into train/validation
  2. Train GBM, tracking validation error each round
  3. Stop when validation error hasn't improved for n_iter_no_change rounds
  4. Use the model from the best round
```

---

## 8. Step-by-Step Example: Gradient Boosting for Regression

### Problem Setup

```
Dataset (predicting house price in $10K):
┌─────┬──────────┬──────────┬───────┐
│  i  │ Size(x₁) │ Age(x₂)  │ Price │
├─────┼──────────┼──────────┼───────┤
│  1  │   1200   │    5     │  24   │
│  2  │   1800   │   10     │  32   │
│  3  │   2400   │    3     │  45   │
│  4  │   1500   │   15     │  20   │
│  5  │   3000   │    2     │  55   │
└─────┴──────────┴──────────┴───────┘

Settings: MSE loss, η = 0.5, depth-1 trees (stumps), T = 3 rounds
```

### Step 0: Initialize with Mean

```
F₀(x) = mean(y) = (24 + 32 + 45 + 20 + 55) / 5 = 35.2
```

### Round 1

**Compute residuals (pseudo-residuals for MSE):**
```
rᵢ₁ = yᵢ - F₀(xᵢ)

r₁₁ = 24 - 35.2 = -11.2
r₂₁ = 32 - 35.2 =  -3.2
r₃₁ = 45 - 35.2 =  +9.8
r₄₁ = 20 - 35.2 = -15.2
r₅₁ = 55 - 35.2 = +19.8
```

**Fit a decision stump to residuals:**

Try splitting on x₁ ≤ 1500:
```
Left (x₁ ≤ 1500):  samples {1, 4}, residuals {-11.2, -15.2}
                    mean = -13.2
Right (x₁ > 1500): samples {2, 3, 5}, residuals {-3.2, +9.8, +19.8}
                    mean = +8.8

SSE = (-11.2 - (-13.2))² + (-15.2 - (-13.2))² 
    + (-3.2 - 8.8)² + (9.8 - 8.8)² + (19.8 - 8.8)²
    = 4 + 4 + 144 + 1 + 121 = 274
```

Try splitting on x₁ ≤ 2100:
```
Left (x₁ ≤ 2100):  samples {1, 2, 4}, residuals {-11.2, -3.2, -15.2}
                    mean = -9.87
Right (x₁ > 2100): samples {3, 5}, residuals {+9.8, +19.8}
                    mean = +14.8

SSE = (-11.2+9.87)² + (-3.2+9.87)² + (-15.2+9.87)²
    + (9.8-14.8)² + (19.8-14.8)²
    = 1.77 + 44.49 + 28.41 + 25 + 25 = 124.67  ← BETTER
```

**Best stump h₁: x₁ ≤ 2100**
```
h₁(x) = { -9.87   if x₁ ≤ 2100
         { +14.80  if x₁ > 2100
```

**Update model:**
```
F₁(x) = F₀(x) + η · h₁(x) = 35.2 + 0.5 · h₁(x)

Predictions:
F₁(x₁) = 35.2 + 0.5×(-9.87) = 35.2 - 4.93 = 30.27
F₁(x₂) = 35.2 + 0.5×(-9.87) = 35.2 - 4.93 = 30.27
F₁(x₃) = 35.2 + 0.5×(14.80) = 35.2 + 7.40 = 42.60
F₁(x₄) = 35.2 + 0.5×(-9.87) = 35.2 - 4.93 = 30.27
F₁(x₅) = 35.2 + 0.5×(14.80) = 35.2 + 7.40 = 42.60
```

### Round 2

**Compute new residuals:**
```
r₁₂ = 24 - 30.27 = -6.27
r₂₂ = 32 - 30.27 = +1.73
r₃₂ = 45 - 42.60 = +2.40
r₄₂ = 20 - 30.27 = -10.27
r₅₂ = 55 - 42.60 = +12.40

Note: Residuals are SMALLER than Round 1! Progress!
```

**Fit stump h₂ to new residuals:**

Best split: x₂ ≤ 7.5
```
Left (x₂ ≤ 7.5):  samples {1, 3, 5}, residuals {-6.27, 2.40, 12.40}
                   mean = +2.84
Right (x₂ > 7.5): samples {2, 4}, residuals {1.73, -10.27}
                   mean = -4.27

h₂(x) = { +2.84   if x₂ ≤ 7.5
         { -4.27   if x₂ > 7.5
```

**Update model:**
```
F₂(x) = F₁(x) + 0.5 · h₂(x)

F₂(x₁) = 30.27 + 0.5×2.84  = 31.69
F₂(x₂) = 30.27 + 0.5×(-4.27) = 28.13
F₂(x₃) = 42.60 + 0.5×2.84  = 44.02
F₂(x₄) = 30.27 + 0.5×(-4.27) = 28.13
F₂(x₅) = 42.60 + 0.5×2.84  = 44.02
```

### Round 3

**Compute residuals:**
```
r₁₃ = 24 - 31.69 = -7.69
r₂₃ = 32 - 28.13 = +3.87
r₃₃ = 45 - 44.02 = +0.98
r₄₃ = 20 - 28.13 = -8.13
r₅₃ = 55 - 44.02 = +10.98
```

**Fit h₃ and update F₃ similarly...**

### Tracking Progress

```
┌───────┬───────────────────────────────────────────────┬────────┐
│ Round │          Predictions                          │  MSE   │
├───────┼───────┬───────┬───────┬───────┬───────────────┼────────┤
│       │  y₁   │  y₂   │  y₃   │  y₄   │  y₅          │        │
│ True  │ 24.00 │ 32.00 │ 45.00 │ 20.00 │ 55.00        │   —    │
├───────┼───────┼───────┼───────┼───────┼───────────────┼────────┤
│  F₀   │ 35.20 │ 35.20 │ 35.20 │ 35.20 │ 35.20        │ 162.16 │
│  F₁   │ 30.27 │ 30.27 │ 42.60 │ 30.27 │ 42.60        │  64.81 │
│  F₂   │ 31.69 │ 28.13 │ 44.02 │ 28.13 │ 44.02        │  51.02 │
└───────┴───────┴───────┴───────┴───────┴───────────────┴────────┘

MSE decreasing with each round ✓
Each round corrects the errors of the previous model ✓
```

---

## 9. GradientBoostingClassifier & Regressor in Scikit-Learn

### Regression Example

```python
import numpy as np
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score

# Generate data
X, y = make_regression(n_samples=1000, n_features=10, noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Train Gradient Boosting Regressor
gbr = GradientBoostingRegressor(
    n_estimators=500,
    learning_rate=0.1,
    max_depth=3,
    min_samples_leaf=5,
    subsample=0.8,           # stochastic GB
    loss='squared_error',
    random_state=42
)
gbr.fit(X_train, y_train)

y_pred = gbr.predict(X_test)
print(f"MSE:  {mean_squared_error(y_test, y_pred):.2f}")
print(f"R²:   {r2_score(y_test, y_pred):.4f}")

# Staged predictions — see how error evolves
import matplotlib.pyplot as plt

train_errors = []
test_errors = []

for y_pred_train in gbr.staged_predict(X_train):
    train_errors.append(mean_squared_error(y_train, y_pred_train))
for y_pred_test in gbr.staged_predict(X_test):
    test_errors.append(mean_squared_error(y_test, y_pred_test))

plt.figure(figsize=(10, 6))
plt.plot(range(1, 501), train_errors, 'b-', label='Training MSE', alpha=0.7)
plt.plot(range(1, 501), test_errors, 'r-', label='Test MSE', alpha=0.7)
best_round = np.argmin(test_errors) + 1
plt.axvline(x=best_round, color='g', linestyle='--', label=f'Best round: {best_round}')
plt.xlabel('Boosting Round')
plt.ylabel('MSE')
plt.title('Gradient Boosting: Training vs Test Error')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

### Classification Example

```python
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

gbc = GradientBoostingClassifier(
    n_estimators=200,
    learning_rate=0.1,
    max_depth=3,
    min_samples_split=5,
    subsample=0.8,
    loss='log_loss',        # binary cross-entropy
    random_state=42
)
gbc.fit(X_train, y_train)

y_pred = gbc.predict(X_test)
print(f"Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(classification_report(y_test, y_pred))

# Feature importance
feature_importance = gbc.feature_importances_
sorted_idx = np.argsort(feature_importance)[-10:]
plt.figure(figsize=(10, 6))
plt.barh(range(10), feature_importance[sorted_idx])
plt.yticks(range(10), load_breast_cancer().feature_names[sorted_idx])
plt.xlabel('Feature Importance')
plt.title('Top 10 Feature Importances (GBM)')
plt.tight_layout()
plt.show()
```

### Early Stopping with Validation

```python
from sklearn.ensemble import GradientBoostingClassifier

gbc = GradientBoostingClassifier(
    n_estimators=1000,         # max rounds (will stop early)
    learning_rate=0.05,
    max_depth=3,
    subsample=0.8,
    validation_fraction=0.15,  # holdout 15% for validation
    n_iter_no_change=20,       # stop if no improvement for 20 rounds
    tol=1e-4,                  # minimum improvement threshold
    random_state=42
)
gbc.fit(X_train, y_train)

print(f"Stopped at {gbc.n_estimators_} rounds (max was 1000)")
print(f"Test accuracy: {gbc.score(X_test, y_test):.4f}")
```

### Key Parameters

```python
GradientBoostingRegressor(
    # === Boosting Parameters ===
    n_estimators=100,        # Number of boosting rounds
    learning_rate=0.1,       # Shrinkage (η)
    loss='squared_error',    # 'squared_error', 'absolute_error', 'huber', 'quantile'

    # === Tree Parameters ===
    max_depth=3,             # Depth of each tree (3-8 typical)
    min_samples_split=2,     # Min samples to split node
    min_samples_leaf=1,      # Min samples in leaf
    max_features=None,       # Features per split ('sqrt', 'log2', float)

    # === Stochastic Parameters ===
    subsample=1.0,           # Fraction of samples per tree (< 1.0 = stochastic)

    # === Early Stopping ===
    n_iter_no_change=None,   # Rounds without improvement before stopping
    validation_fraction=0.1, # Fraction held out for early stopping
    tol=1e-4,                # Improvement threshold

    random_state=None
)
```

---

## 10. Applications

| Domain | Application | Loss Function | Notes |
|--------|-------------|---------------|-------|
| **Finance** | Credit risk scoring | Log loss | Industry standard for FICO-style models |
| **E-commerce** | Product demand forecasting | MSE / Huber | Handles seasonal patterns well |
| **Healthcare** | Patient readmission prediction | Log loss | Interpretable feature importance |
| **Web Search** | Learning to Rank (LTR) | LambdaRank | Google, Bing use GBM for ranking |
| **Insurance** | Claim amount prediction | Huber loss | Robust to outlier claims |
| **Energy** | Load forecasting | MSE | Power grid demand prediction |
| **Real Estate** | Property valuation | Quantile loss | Predict price intervals |

---

## 11. Summary Table

| Aspect | Details |
|--------|---------|
| **Inventor** | Jerome Friedman (1999, 2001) |
| **Core Idea** | Fit new trees to negative gradients (pseudo-residuals) of loss |
| **For MSE Loss** | Equivalent to fitting residuals y - F(x) |
| **Generality** | Works with ANY differentiable loss function |
| **Base Learner** | Typically shallow decision trees (depth 3-8) |
| **Additive Model** | F_T = F₀ + η·h₁ + η·h₂ + ... + η·h_T |
| **Learning Rate** | η = 0.01 to 0.3; smaller = better generalization |
| **Regularization** | Shrinkage + tree constraints + subsampling + early stopping |
| **Stochastic GBM** | subsample < 1.0; adds randomness, reduces overfitting |
| **Early Stopping** | Monitor validation error, stop when it plateaus |
| **Bias/Variance** | Reduces bias; some risk of increasing variance |
| **Parallelizable** | Not inherently (sequential); tree building can be parallelized |
| **Limitations** | Sequential training (slow), sensitive to hyperparameters |

---

## 12. Revision Questions

**Q1.** Derive the pseudo-residuals for the Huber loss function. When does it behave like MSE, and when like MAE?

<details>
<summary>Answer</summary>

**Huber loss:**
```
L(y, F) = ½(y-F)²           if |y-F| ≤ δ
          δ|y-F| - ½δ²       if |y-F| > δ
```

**Pseudo-residual (negative gradient w.r.t. F):**
```
rᵢ = -∂L/∂F = { y - F        if |y-F| ≤ δ    (MSE behavior)
               { δ·sign(y-F)  if |y-F| > δ    (MAE behavior)
```

**Interpretation:**
- For **small residuals** (|y-F| ≤ δ): The pseudo-residual equals the actual residual, exactly like MSE. This gives accurate correction near the true value.
- For **large residuals** (|y-F| > δ): The pseudo-residual is capped at ±δ, like MAE. This prevents outliers from having excessive influence.
- The parameter δ controls the transition point. A common choice is the median of absolute residuals from the previous iteration.
</details>

**Q2.** Explain why a learning rate of 0.1 with 1000 trees often outperforms a learning rate of 1.0 with 100 trees, even though both have similar "total learning."

<details>
<summary>Answer</summary>

While η × T is roughly similar (0.1 × 1000 = 100 vs 1.0 × 100 = 100), the **path through function space** is very different:

**η = 1.0, T = 100:**
- Each tree makes a full correction
- Large steps in function space can overshoot the optimum
- The model can quickly overfit to training noise
- Little opportunity for "course correction"

**η = 0.1, T = 1000:**
- Each tree makes a small, cautious correction
- The model explores function space gradually
- Each new tree sees mostly signal (not noise from previous overfitting)
- Acts like strong regularization — similar to small step sizes in gradient descent
- More opportunities to find a better path through function space

This is analogous to **simulated annealing** or **SGD with small learning rate** — slower exploration tends to find better optima. Friedman (2001) called this the "shrinkage" effect and showed empirically that it consistently improves test performance.
</details>

**Q3.** Compare Gradient Boosting and AdaBoost. Show that AdaBoost is a special case of Gradient Boosting with exponential loss.

<details>
<summary>Answer</summary>

**Connection:** AdaBoost can be derived as Gradient Boosting with exponential loss L(y, F) = exp(-yF):

1. **Pseudo-residual for exponential loss:**
   rᵢ = -∂exp(-yᵢFᵢ)/∂Fᵢ = yᵢ · exp(-yᵢFᵢ)

2. **The sample weight in AdaBoost round t:**
   wᵢ⁽ᵗ⁾ = exp(-yᵢ · Fₜ₋₁(xᵢ)) (unnormalized)
   
   This is exactly the exponential loss evaluated at the current model — higher loss samples get higher weight.

3. **The optimal learner weight** αₜ = ½ ln((1-εₜ)/εₜ) can be derived by minimizing the exponential loss w.r.t. the step size.

**Key difference:** Gradient Boosting generalizes this to ANY differentiable loss:
- Exponential loss → AdaBoost (outlier-sensitive)
- Log loss → LogitBoost (better calibrated)
- Huber loss → Robust regression
- Custom loss → Custom behavior

AdaBoost is thus a special case, and Gradient Boosting's flexibility is what makes it more widely used today.
</details>

**Q4.** You train a GBM with 1000 trees and observe that training MSE = 0.5 but test MSE = 15.0. Diagnose the problem and propose three specific solutions.

<details>
<summary>Answer</summary>

**Diagnosis:** Severe **overfitting**. The large gap between training and test error (0.5 vs 15.0) indicates the model has memorized training data, including noise.

**Three solutions:**

1. **Reduce learning rate + use early stopping:**
   Set η = 0.01 (from default 0.1), use validation_fraction=0.15, n_iter_no_change=20. The model will stop before overfitting.

2. **Constrain tree complexity:**
   Reduce max_depth from default 3 to 2, increase min_samples_leaf to 10-20. Simpler trees generalize better.

3. **Add stochastic subsampling:**
   Set subsample=0.5 and max_features=0.7. This introduces randomness that acts as regularization, similar to dropout in neural networks.

**Additional measures:** Verify data quality (look for data leakage), increase training set size if possible, and use cross-validation to tune hyperparameters properly.
</details>

**Q5.** Explain the difference between subsampling rows (subsample parameter) and subsampling columns (max_features parameter) in Gradient Boosting. When would you use each?

<details>
<summary>Answer</summary>

**Row subsampling (subsample):**
- Each tree is trained on a random subset of training rows
- Similar to bootstrap sampling in bagging
- Introduces randomness at the **data level**
- Reduces overfitting by preventing any single tree from seeing all data
- Speeds up training (less data per tree)
- Typical values: 0.5 to 0.8

**Column subsampling (max_features):**
- Each tree (or each split) considers only a random subset of features
- Similar to the feature randomization in Random Forest
- Introduces randomness at the **feature level**
- Decorrelates successive trees
- Especially useful when some features dominate
- Typical values: 'sqrt', 0.5 to 0.8

**When to use each:**
- Row subsampling: When dataset is large and you want faster training + regularization
- Column subsampling: When you have many features, some dominant, or want to reduce feature-level overfitting
- Both together: Often the best strategy; XGBoost and LightGBM use both by default
</details>

**Q6.** Implement gradient boosting for regression from scratch (pseudocode or simplified Python) for MSE loss with decision stumps.

<details>
<summary>Answer</summary>

```python
import numpy as np

def gradient_boosting_from_scratch(X, y, n_estimators=100, learning_rate=0.1):
    n_samples = len(y)
    
    # Step 0: Initialize with mean
    F = np.full(n_samples, np.mean(y))
    models = []
    
    for t in range(n_estimators):
        # Step 1: Compute pseudo-residuals (for MSE: r = y - F)
        residuals = y - F
        
        # Step 2: Fit a decision stump to residuals
        # Try all features and thresholds, pick the one with lowest SSE
        best_feature, best_threshold, best_left_val, best_right_val = None, None, None, None
        best_sse = float('inf')
        
        for feature_idx in range(X.shape[1]):
            thresholds = np.unique(X[:, feature_idx])
            for threshold in thresholds:
                left_mask = X[:, feature_idx] <= threshold
                right_mask = ~left_mask
                if left_mask.sum() == 0 or right_mask.sum() == 0:
                    continue
                
                left_val = residuals[left_mask].mean()
                right_val = residuals[right_mask].mean()
                
                sse = (np.sum((residuals[left_mask] - left_val)**2) +
                       np.sum((residuals[right_mask] - right_val)**2))
                
                if sse < best_sse:
                    best_sse = sse
                    best_feature = feature_idx
                    best_threshold = threshold
                    best_left_val = left_val
                    best_right_val = right_val
        
        # Step 3: Update model
        predictions = np.where(
            X[:, best_feature] <= best_threshold,
            best_left_val, best_right_val
        )
        F += learning_rate * predictions
        
        models.append((best_feature, best_threshold, 
                       best_left_val, best_right_val))
        
        if (t + 1) % 20 == 0:
            mse = np.mean((y - F)**2)
            print(f"Round {t+1}: MSE = {mse:.4f}")
    
    return models, np.mean(y)  # return models and initial prediction
```

This simplified implementation shows the core loop: compute residuals → fit stump → update predictions. Production implementations add regularization, different losses, and efficient tree building.
</details>

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← 8.4 AdaBoost](./04-adaboost.md) | [Unit 8: Ensemble Methods](../README.md) | [8.6 XGBoost, LightGBM & CatBoost →](./06-xgboost-lightgbm-catboost.md) |

---

*© 2025 AIML Study Notes. Unit 8: Ensemble Methods — Chapter 5: Gradient Boosting.*
