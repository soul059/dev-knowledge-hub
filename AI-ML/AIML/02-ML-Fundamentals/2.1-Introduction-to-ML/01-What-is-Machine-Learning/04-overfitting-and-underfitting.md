# 📉 Chapter 4: Overfitting and Underfitting

> **Module:** 01 – What is Machine Learning | **Topic:** The Two Failure Modes of Learning  
> **Difficulty:** ⭐⭐⭐ Intermediate | **Est. Time:** 40 minutes

---

## 📋 Overview

Every ML practitioner must understand the two fundamental failure modes that prevent models from generalizing well to unseen data. **Underfitting** occurs when a model is too simple to capture the underlying pattern, while **overfitting** occurs when a model memorizes noise instead of learning the true signal. Mastering the balance between these two extremes is central to building effective ML systems.

---

## 1. Definitions

### 1.1 Underfitting (High Bias)

A model **underfits** when it is **too simple** to capture the underlying structure of the data. It performs poorly on *both* training data and unseen test data.

```
Underfitting = Model fails to learn the pattern
             → High training error
             → High validation error
```

### 1.2 Overfitting (High Variance)

A model **overfits** when it learns the **noise and random fluctuations** in the training data rather than the true underlying relationship. It performs well on training data but poorly on unseen data.

```
Overfitting = Model memorizes the training data
            → Low training error
            → High validation error
```

---

## 2. Causes

| Factor | Underfitting | Overfitting |
|--------|-------------|-------------|
| **Model complexity** | Too simple (e.g., linear for curved data) | Too complex (e.g., degree-15 polynomial) |
| **Training data size** | Can occur regardless | More likely with small datasets |
| **Feature set** | Too few or irrelevant features | Too many features (incl. noise) |
| **Training duration** | Stopped too early | Trained for too many epochs |
| **Regularization** | Excessive regularization | Insufficient regularization |

---

## 3. Detecting Overfitting vs Underfitting

### 3.1 Training vs Validation Error Patterns

```
   Error
    ▲
    │
    │  xxxxxxxxxxxxxxxxxxxxxxxxx   ← Validation Error (Underfitting)
    │
    │  ooooooooooooooooooooooooo   ← Training Error   (Underfitting)
    │
    └──────────────────────────► Epochs / Complexity
         BOTH errors are HIGH → UNDERFITTING
```

```
   Error
    ▲
    │         x                    ← Validation Error (rises)
    │        x x       x x x x
    │       x   x x x x
    │     x
    │   x
    │  o o o
    │        o o o
    │              o o o o o o o   ← Training Error (very low)
    │
    └──────────────────────────► Epochs / Complexity
         GAP widens → OVERFITTING
```

### 3.2 The Model Complexity Curve

```
   Error
    ▲
    │
  H │ \                        /
  i │  \    Validation        /
  g │   \   Error            /
  h │    \                 /
    │     \              /
    │      \    _____ /
    │       \  /  ↑
    │        \/   Sweet Spot
  L │         \
  o │          \_______________
  w │           Training Error
    │
    └──────────────────────────────► Model Complexity
    │           │             │
    │ UNDERFIT  │  GOOD FIT   │ OVERFIT
    │   ZONE    │    ZONE     │  ZONE
```

**Key insight:** The sweet spot is where the validation error is minimized — the model is complex enough to capture patterns but not so complex that it memorizes noise.

---

## 4. Solutions and Remedies

### 4.1 Fixing Underfitting

| Strategy | How It Helps |
|----------|-------------|
| Increase model complexity | Use a more powerful model (e.g., polynomial, deep net) |
| Add more relevant features | Give the model more information to learn from |
| Reduce regularization | Allow the model more freedom to fit the data |
| Train longer | Let the model converge (more epochs) |
| Use non-linear models | Switch from linear regression to decision trees/neural nets |

### 4.2 Fixing Overfitting

| Strategy | How It Helps |
|----------|-------------|
| **More training data** | Harder for the model to memorize a large dataset |
| **L1 Regularization (Lasso)** | Adds penalty `λ·Σ|wᵢ|` → drives weights to zero (feature selection) |
| **L2 Regularization (Ridge)** | Adds penalty `λ·Σwᵢ²` → shrinks weights toward zero |
| **Dropout** (neural nets) | Randomly disables neurons during training |
| **Early stopping** | Stop training when validation error begins to rise |
| **Cross-validation** | Uses all data for both training and validation (k-fold) |
| **Pruning** (decision trees) | Removes branches that add little predictive power |
| **Feature selection** | Remove noisy or irrelevant features |

### 4.3 Regularization Formulas

**L1 (Lasso) — penalizes sum of absolute weights:**

```
Loss_L1 = Original_Loss + λ · Σ|wᵢ|
```

**L2 (Ridge) — penalizes sum of squared weights:**

```
Loss_L2 = Original_Loss + λ · Σ(wᵢ)²
```

**Elastic Net — combines L1 and L2:**

```
Loss_EN = Original_Loss + λ₁ · Σ|wᵢ| + λ₂ · Σ(wᵢ)²
```

> `λ` (lambda) is the **regularization strength**. Higher λ = stronger penalty = simpler model.

### 4.4 Early Stopping (ASCII Diagram)

```
   Error
    ▲
    │        x x x      x x x
    │      x       x x x          ← Validation Error
    │    x
    │  x        ← STOP HERE (minimum val error)
    │  o               |
    │    o o            |
    │        o o o o o o o o o     ← Training Error
    │                   |
    └───────────────────┼────────► Epochs
                  Early Stopping
                    Point
```

---

## 5. Python Example — Overfitting with Polynomial Regression

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split

# Generate synthetic data: y = sin(x) + noise
np.random.seed(42)
X = np.sort(np.random.uniform(0, 6, 30)).reshape(-1, 1)
y = np.sin(X).ravel() + np.random.normal(0, 0.2, 30)

X_train, X_val, y_train, y_val = train_test_split(X, y, test_size=0.3, random_state=42)

# Try different polynomial degrees
degrees = [1, 4, 15]
results = []

for d in degrees:
    poly = PolynomialFeatures(degree=d)
    X_train_poly = poly.fit_transform(X_train)
    X_val_poly = poly.transform(X_val)

    model = LinearRegression()
    model.fit(X_train_poly, y_train)

    train_err = mean_squared_error(y_train, model.predict(X_train_poly))
    val_err = mean_squared_error(y_val, model.predict(X_val_poly))

    label = "Underfit" if d == 1 else ("Good Fit" if d == 4 else "Overfit")
    results.append((d, train_err, val_err, label))
    print(f"Degree {d:2d} | Train MSE: {train_err:.4f} | Val MSE: {val_err:.4f} | {label}")

# Expected output (approximate):
# Degree  1 | Train MSE: 0.2800 | Val MSE: 0.3500 | Underfit
# Degree  4 | Train MSE: 0.0350 | Val MSE: 0.0520 | Good Fit
# Degree 15 | Train MSE: 0.0040 | Val MSE: 2.8100 | Overfit
```

**Interpretation:**
- **Degree 1** — Both errors high → underfitting (straight line cannot fit a sine curve)
- **Degree 4** — Low and balanced errors → good generalization
- **Degree 15** — Training error near zero but validation error explodes → overfitting

---

## 6. Real-World Examples

### 6.1 Underfitting Examples
- **Spam filter** using only email length as a feature — too little information to distinguish spam
- **Self-driving car** with a basic linear model — cannot capture complex road scenarios
- **Medical diagnosis** model ignoring patient history — misses critical patterns

### 6.2 Overfitting Examples
- **Fraud detection** trained on 50 transactions — memorizes specific users, fails on new ones
- **Image classifier** that memorizes specific training photos — fails on slightly different angles
- **Stock predictor** perfectly fitting past prices — captures random noise, not market trends

---

## 7. Summary Table — Overfitting vs Underfitting

| Aspect | Underfitting | Overfitting |
|--------|-------------|-------------|
| **Also called** | High Bias | High Variance |
| **Training error** | High | Very Low |
| **Validation error** | High | High |
| **Error gap** | Small (both high) | Large (train ≪ val) |
| **Model complexity** | Too simple | Too complex |
| **Captures noise?** | No | Yes |
| **Captures signal?** | No | Yes (plus noise) |
| **Fix: data** | Add features | Add more samples |
| **Fix: model** | More complex model | Regularization / simpler model |
| **Fix: training** | Train longer | Early stopping |

---

## 8. Key Takeaways

```
┌─────────────────────────────────────────────────────────────────┐
│  • Underfitting → model too simple → high bias → both errors   │
│    high → add complexity, features, or train longer             │
│  • Overfitting → model too complex → high variance → gap       │
│    between train/val error → regularize, more data, or simplify │
│  • The GOAL is the sweet spot: complex enough to learn the      │
│    pattern, simple enough to generalize                         │
│  • Regularization (L1/L2), dropout, early stopping, and        │
│    cross-validation are the primary tools to combat overfitting │
│  • Always monitor BOTH training AND validation errors           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. ✏️ Revision Questions

**Q1.** What is the key difference between overfitting and underfitting in terms of training and validation errors?

> **A:** Underfitting shows high error on both training and validation sets, while overfitting shows very low training error but high validation error — a large gap between them.

**Q2.** A model achieves 99% accuracy on training data but only 60% on test data. What is the problem and name three solutions.

> **A:** The model is overfitting. Solutions: (1) collect more training data, (2) apply L2 regularization, (3) use early stopping or dropout.

**Q3.** Write the L1 and L2 regularization loss formulas and explain how each affects model weights.

> **A:** L1: `Loss + λΣ|wᵢ|` pushes weights to exactly zero (sparse models). L2: `Loss + λΣwᵢ²` shrinks weights toward zero but rarely makes them exactly zero.

**Q4.** Why does increasing training data help reduce overfitting but not underfitting?

> **A:** More data makes it harder for the model to memorize noise (reduces overfitting). However, if the model is inherently too simple, more data won't help it capture the true pattern (underfitting persists).

**Q5.** Explain how early stopping works and what metric you monitor.

> **A:** Monitor validation error during training. When it starts increasing while training error continues to decrease, stop training at the epoch with the lowest validation error.

**Q6.** In the polynomial regression example, why does degree 15 overfit while degree 4 generalizes well?

> **A:** Degree 15 has far more parameters than data points, giving it enough flexibility to pass through every training point (including noise). Degree 4 has enough flexibility to approximate the sine curve without fitting the noise.

---

## 🧭 Navigation

| ⬅️ Previous | 🏠 Home | ➡️ Next |
|:---|:---:|---:|
| [03 – Training, Validation & Testing](03-training-validation-testing.md) | [Module Overview](README.md) | [05 – Bias-Variance Tradeoff](05-bias-variance-tradeoff.md) |

---

*📝 Last Updated: 2025 | Part of the AIML Study Guide*
