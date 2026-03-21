# Chapter 7: Bias-Variance Tradeoff

> **Unit 4 — Statistics for ML** | The fundamental tension in every model

---

## 7.1 Overview

Every supervised ML model battles two sources of error: **bias** (wrong
assumptions) and **variance** (sensitivity to training data). Mastering this
tradeoff is the key to building models that generalise well.

```
  Total Error  =  Bias²  +  Variance  +  Irreducible Noise
```

This chapter breaks down each component, shows how model complexity controls
the tradeoff, and presents practical strategies for improvement.

---

## 7.2 Bias — Systematic Error

### Definition

Bias measures how far the model's average prediction is from the true value.

```
  Bias(f̂) = E[f̂(x)] − f(x)
```

**High bias** means the model is too simple to capture the underlying pattern —
it **underfits**.

### Characteristics of High-Bias Models

- Training error is high.
- Training and test errors are close (both bad).
- Model is too rigid (e.g., linear model for non-linear data).

### Examples

| Model | Bias Level |
|---|---|
| Linear regression on non-linear data | High bias |
| Shallow decision tree (max_depth=2) | High bias |
| Deep neural network | Low bias (can fit complex patterns) |

---

## 7.3 Variance — Sensitivity to Training Data

### Definition

Variance measures how much the model's predictions change across different
training sets.

```
  Var(f̂) = E[(f̂(x) − E[f̂(x)])²]
```

**High variance** means the model memorises training data quirks — it
**overfits**.

### Characteristics of High-Variance Models

- Training error is very low (near zero).
- Test error is much higher than training error (large gap).
- Model is too flexible (e.g., deep tree, high-degree polynomial).

### Examples

| Model | Variance Level |
|---|---|
| k-NN with k=1 | High variance |
| Decision tree (no pruning) | High variance |
| Linear regression | Low variance |

---

## 7.4 Irreducible Noise (ε)

```
  y = f(x) + ε
```

This noise comes from:
- Measurement errors in the data.
- Missing features that affect the outcome.
- Inherent randomness in the process.

No model can reduce this. It sets the **floor** for error.

---

## 7.5 Total Error Decomposition

For a model f̂ predicting target y = f(x) + ε at point x:

```
  E[(y − f̂(x))²] = Bias²(f̂(x)) + Var(f̂(x)) + σ²_ε

  ┌──────────────┐   ┌──────────┐   ┌──────────┐   ┌─────────────┐
  │ Expected      │ = │  Bias²   │ + │ Variance │ + │ Irreducible │
  │ Test Error    │   │          │   │          │   │ Noise (σ²)  │
  └──────────────┘   └──────────┘   └──────────┘   └─────────────┘
```

### Derivation Sketch

```
E[(y − f̂)²]
= E[(f + ε − f̂)²]
= E[(f − E[f̂])²] + E[(f̂ − E[f̂])²] + E[ε²]
=     Bias²       +     Variance     +   σ²
```

---

## 7.6 The Tradeoff — Model Complexity Curve

As model complexity increases, bias decreases but variance increases.

```
  Error
    │
    │ \                                      ╱
    │  \    Total Error                    ╱
    │   \        ╲                       ╱
    │    \        ╲                    ╱
    │     \        ╲   ●            ╱
    │      \        ╲ Sweet       ╱
    │       \        ╲ Spot     ╱        ← Variance
    │        \        ╲      ╱
    │         ╲        ╲  ╱
    │          ╲      ╱ ╲
    │           ╲  ╱     ╲
    │    Bias → ╲╱        ╲
    │          ╱ ╲         ╲
    │        ╱    ╲         ╲
    │      ╱       ──────────────────
    │    ╱          Irreducible Noise
    └──────────────────────────────────── Complexity
     Simple                         Complex
     (Underfit)                     (Overfit)

  Sweet spot = optimal complexity that minimises total error.
```

### Summary

| Regime | Bias | Variance | Symptom |
|---|---|---|---|
| Underfitting | High | Low | Train error ≈ Test error (both high) |
| Sweet spot | Moderate | Moderate | Good generalisation |
| Overfitting | Low | High | Train error ≪ Test error |

---

## 7.7 Diagnosing Bias vs Variance

### Using Training and Validation Errors

```
  Scenario 1 — High Bias (Underfitting):
  Train error: 15%
  Val error:   17%
  Gap: small, both errors high

  Scenario 2 — High Variance (Overfitting):
  Train error:  2%
  Val error:   18%
  Gap: large, train error low

  Scenario 3 — Good Fit:
  Train error:  5%
  Val error:    7%
  Gap: small, both errors acceptable
```

### Python — Diagnostic Example

```python
from sklearn.model_selection import learning_curve
from sklearn.tree import DecisionTreeClassifier
import numpy as np
import matplotlib.pyplot as plt

clf = DecisionTreeClassifier(max_depth=3)
train_sizes, train_scores, val_scores = learning_curve(
    clf, X, y, cv=5, train_sizes=np.linspace(0.1, 1.0, 10),
    scoring='accuracy'
)

plt.plot(train_sizes, train_scores.mean(axis=1), label='Training')
plt.plot(train_sizes, val_scores.mean(axis=1), label='Validation')
plt.xlabel('Training Set Size')
plt.ylabel('Accuracy')
plt.legend()
plt.title('Learning Curve')
plt.show()
```

---

## 7.8 Learning Curves Interpretation

### High Bias (Underfitting)

```
  Accuracy
    │    ─────────────── Training
    │    ─────────────── Validation
    │
    │  Both converge to a LOW value.
    │  Adding more data won't help much.
    └────────────────────────────── n
```

**Fix:** Increase model complexity, add features, reduce regularisation.

### High Variance (Overfitting)

```
  Accuracy
    │  ────────────────── Training (high)
    │
    │         gap
    │
    │  ─────────────────  Validation (low, but improving)
    │
    │  Gap decreases slowly with more data.
    └────────────────────────────── n
```

**Fix:** Get more data, reduce complexity, increase regularisation.

### Good Fit

```
  Accuracy
    │  ────────────────── Training
    │  ────────────────── Validation (close to training)
    │
    │  Both converge to a HIGH value with small gap.
    └────────────────────────────── n
```

---

## 7.9 Strategies to Reduce Bias

| Strategy | How It Helps |
|---|---|
| Use a more complex model | Captures more patterns |
| Add polynomial / interaction features | Richer representation |
| Reduce regularisation strength | Less constraint on model |
| Use ensemble methods (boosting) | Sequentially reduces bias |
| Train longer (neural nets) | More epochs to learn |
| Remove irrelevant constraints | Let model be more flexible |

---

## 7.10 Strategies to Reduce Variance

| Strategy | How It Helps |
|---|---|
| Get more training data | More data → less overfitting |
| Reduce model complexity | Fewer parameters to overfit |
| Increase regularisation (L1, L2) | Penalises large weights |
| Use ensemble methods (bagging) | Averages out noise |
| Feature selection | Remove noisy features |
| Dropout (neural nets) | Random neuron deactivation |
| Early stopping | Stop before overfitting |
| Cross-validation | Better estimate of true error |

---

## 7.11 Regularisation Effect on Bias-Variance

Regularisation adds a penalty to the loss function:

```
  Ridge (L2):  Loss = MSE + λ Σ wⱼ²
  Lasso (L1):  Loss = MSE + λ Σ |wⱼ|
```

```
  λ (regularisation strength)
  ─────────────────────────────────────────►
  λ = 0              λ moderate           λ large
  No penalty         Sweet spot           Heavy penalty
  Low bias           Balanced             High bias
  High variance      Optimal              Low variance
  (Overfit)          (Good fit)           (Underfit)
```

### Python — Regularisation Sweep

```python
from sklearn.linear_model import Ridge
from sklearn.model_selection import cross_val_score
import numpy as np

alphas = [0.001, 0.01, 0.1, 1, 10, 100, 1000]
for alpha in alphas:
    model = Ridge(alpha=alpha)
    scores = cross_val_score(model, X, y, cv=5, scoring='neg_mean_squared_error')
    print(f"λ={alpha:>7}  CV MSE = {-scores.mean():.4f} ± {scores.std():.4f}")
```

---

## 7.12 Ensemble Methods Effect

### Bagging (e.g., Random Forest) — Reduces Variance

```
  Training Data
     │
     ├── Bootstrap Sample 1 → Model 1 ──┐
     ├── Bootstrap Sample 2 → Model 2 ──┼── Average → Low Variance
     ├── Bootstrap Sample 3 → Model 3 ──┤
     └── Bootstrap Sample B → Model B ──┘

  Each model has high variance individually,
  but averaging cancels out the noise.
```

### Boosting (e.g., XGBoost) — Reduces Bias

```
  Iteration 1: Fit Model₁ → Compute residuals r₁
  Iteration 2: Fit Model₂ to r₁ → Compute residuals r₂
  Iteration 3: Fit Model₃ to r₂ → Compute residuals r₃
       ⋮
  Final: f̂ = Model₁ + η·Model₂ + η·Model₃ + ...

  Each iteration corrects errors of the previous one → reduces bias.
  (But too many iterations → overfitting → variance creeps back.)
```

### Summary of Ensembles

| Method | Primary Benefit | Risk |
|---|---|---|
| Bagging | ↓ Variance | Doesn't help much with bias |
| Boosting | ↓ Bias | Can overfit if not regularised |
| Stacking | ↓ Both (meta-learning) | Complexity, overfitting |

---

## 7.13 Practical Workflow

```
  1. Start with a simple baseline model.
  2. Evaluate training and validation error.
  3. If high bias  → increase complexity, try boosting, add features.
  4. If high variance → regularise, get more data, try bagging.
  5. Plot learning curves to confirm diagnosis.
  6. Iterate until validation error is minimised.
```

```python
# Quick diagnostic
train_acc = model.score(X_train, y_train)
val_acc   = model.score(X_val, y_val)

gap = train_acc - val_acc
print(f"Train: {train_acc:.4f}, Val: {val_acc:.4f}, Gap: {gap:.4f}")

if train_acc < 0.80:
    print("→ High bias: try a more complex model")
elif gap > 0.10:
    print("→ High variance: try regularisation or more data")
else:
    print("→ Good balance")
```

---

## 7.14 Real-World ML Applications

| Scenario | Diagnosis | Action |
|---|---|---|
| Linear model on image data | High bias | Switch to CNN |
| Deep tree on small dataset | High variance | Prune or use Random Forest |
| Neural net, train acc 99%, test 70% | Overfitting | Dropout, early stopping |
| k-NN with k=1 | High variance | Increase k |
| k-NN with k=n | High bias | Decrease k |
| XGBoost overtrained | Variance creep | Reduce n_estimators, add regularisation |

---

## Summary Table

| Concept | Key Idea |
|---|---|
| Bias | E[f̂(x)] − f(x); systematic error |
| Variance | Sensitivity to training data fluctuations |
| Irreducible noise | σ²; cannot be reduced by any model |
| Total error | Bias² + Variance + σ² |
| Underfitting | High bias, low variance |
| Overfitting | Low bias, high variance |
| Regularisation | ↑ λ → ↑ bias, ↓ variance |
| Bagging | Reduces variance via averaging |
| Boosting | Reduces bias via sequential correction |
| Learning curves | Diagnose bias vs variance visually |

---

## Quick Revision Questions

1. Write the bias-variance decomposition formula and explain each term.
2. You observe train accuracy = 98 % and test accuracy = 72 %. Is this high
   bias or high variance? What would you try first?
3. How does increasing k in k-NN affect bias and variance?
4. Why does bagging reduce variance but not bias?
5. A learning curve shows both training and validation errors converging to
   a high value. What is the diagnosis and the remedy?
6. Explain how early stopping acts as a form of regularisation.

---

*← Previous: [Statistical Significance](06-statistical-significance.md) | Next: [Unit 5 →](../../05-Information-Theory/) →*
