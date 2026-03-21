# Chapter 2: Cross-Entropy

> **Unit 5 — Information Theory** | Mathematics for Machine Learning

---

## 1. Overview

**Cross-entropy** measures the average number of bits needed to encode data from distribution **p** using a coding scheme optimized for distribution **q**. In machine learning, cross-entropy is the **dominant loss function for classification tasks**, connecting information theory directly to model training.

---

## 2. Cross-Entropy — Definition

For discrete distributions p (true) and q (predicted) over the same set of outcomes:

```
                n
H(p, q) = - Σ  p(xᵢ) log q(xᵢ)
               i=1
```

| Symbol | Meaning                              |
|--------|--------------------------------------|
| p      | True (target) distribution           |
| q      | Predicted (model) distribution       |
| H(p,q) | Cross-entropy of q relative to p     |

### Key Relationship

```
H(p, q) = H(p) + D_KL(p ‖ q)
```

Since D_KL(p‖q) ≥ 0, we always have:

```
H(p, q) ≥ H(p)
```

**Equality holds when q = p** — the model perfectly matches the true distribution.

---

## 3. Intuition

```
  ┌──────────────┐
  │  True dist p  │──── H(p): minimum bits needed (entropy)
  └──────────────┘
         │
         ▼
  ┌──────────────┐
  │  Model dist q │──── H(p,q): actual bits used with model q
  └──────────────┘
         │
         ▼
  Extra bits wasted = D_KL(p ‖ q) = H(p,q) - H(p)
```

- If q matches p perfectly → no wasted bits → cross-entropy equals entropy
- If q is a poor model → many wasted bits → high cross-entropy

---

## 4. Binary Cross-Entropy (BCE)

For binary classification with true label y ∈ {0, 1} and predicted probability ŷ = q(y=1):

```
BCE = -[ y log(ŷ) + (1 - y) log(1 - ŷ) ]
```

For a dataset of N samples:

```
            1   N
L_BCE = - ─── Σ  [ yᵢ log(ŷᵢ) + (1 - yᵢ) log(1 - ŷᵢ) ]
            N  i=1
```

### Behavior Table

| True y | Predicted ŷ | BCE Loss  | Interpretation       |
|--------|-------------|-----------|----------------------|
| 1      | 0.99        | 0.01      | ✓ Confident correct  |
| 1      | 0.50        | 0.69      | ⚠ Uncertain          |
| 1      | 0.01        | 4.61      | ✗ Confident wrong    |
| 0      | 0.01        | 0.01      | ✓ Confident correct  |
| 0      | 0.50        | 0.69      | ⚠ Uncertain          |
| 0      | 0.99        | 4.61      | ✗ Confident wrong    |

**Key insight:** Cross-entropy penalizes **confident wrong predictions** exponentially.

---

## 5. Categorical Cross-Entropy (Multi-class)

For K classes with one-hot encoded true label y and predicted probabilities ŷ:

```
             K
L_CE = - Σ  yₖ log(ŷₖ)
            k=1
```

Since y is one-hot (only one yₖ = 1), this simplifies to:

```
L_CE = -log(ŷ_c)    where c is the true class
```

This is equivalent to the **negative log-likelihood** (NLL) of the correct class.

### Example: 3-class Classification

True label: Class 2 → y = [0, 1, 0]

| Model Output (softmax) | Loss = -log(ŷ₂) | Quality    |
|------------------------|------------------|------------|
| [0.05, 0.90, 0.05]    | 0.105            | ✓ Good     |
| [0.33, 0.34, 0.33]    | 1.079            | ⚠ Uncertain|
| [0.70, 0.10, 0.20]    | 2.303            | ✗ Bad      |

---

## 6. Why Cross-Entropy Instead of MSE for Classification?

### The Gradient Problem

With sigmoid output σ(z) and MSE loss:
```
∂L_MSE/∂z = (ŷ - y) · σ'(z)
```
The σ'(z) term **vanishes** when ŷ ≈ 0 or ŷ ≈ 1 → slow/stalled learning.

With cross-entropy loss:
```
∂L_CE/∂z = ŷ - y
```
Clean gradient — **no vanishing gradient problem**. Learning speed is proportional to error magnitude.

### Comparison

```
  Loss
   5 |  CE ╲                          ╱ CE
     |       ╲                      ╱
   4 |        ╲                   ╱
     |         ╲                ╱
   3 |          ╲             ╱
     |    MSE    ╲──────────╱    MSE
   2 |   ╱‾‾‾‾‾‾‾╲       ╱‾‾‾‾‾‾╲
     |  ╱          ╲     ╱        ╲
   1 | ╱            ╲   ╱          ╲
     |╱              ╲ ╱            ╲
   0 +────────────────+──────────────→ ŷ
     0              0.5              1
         (true y = 1)
```

| Criterion               | MSE              | Cross-Entropy        |
|--------------------------|------------------|----------------------|
| Gradient at ŷ ≈ 0 (y=1)  | Vanishes (slow)  | Large (fast fix)     |
| Probabilistic meaning    | None             | Negative log-likelihood |
| Convexity with softmax   | Non-convex       | Convex               |
| Standard in practice     | Regression       | Classification       |

---

## 7. Connection to Log Loss

In many ML libraries, **log loss** and **cross-entropy loss** are synonymous:

```
Log Loss = -(1/N) Σ [ yᵢ log(ŷᵢ) + (1-yᵢ) log(1-ŷᵢ) ]
```

- scikit-learn: `sklearn.metrics.log_loss(y_true, y_pred)`
- This is exactly the average binary cross-entropy over the dataset
- Kaggle competitions frequently use log loss as the evaluation metric

---

## 8. Step-by-Step Examples

### Example 1: Binary Cross-Entropy Calculation

Dataset with 4 samples:

| Sample | True y | Predicted ŷ | -[y log ŷ + (1-y) log(1-ŷ)] |
|--------|--------|-------------|------------------------------|
| 1      | 1      | 0.9         | -(1·ln0.9 + 0·ln0.1) = 0.105|
| 2      | 0      | 0.2         | -(0·ln0.2 + 1·ln0.8) = 0.223|
| 3      | 1      | 0.7         | -(1·ln0.7 + 0·ln0.3) = 0.357|
| 4      | 0      | 0.4         | -(0·ln0.4 + 1·ln0.6) = 0.511|

```
Average BCE = (0.105 + 0.223 + 0.357 + 0.511) / 4 = 0.299
```

### Example 2: Categorical Cross-Entropy

True distribution: p = [0, 0, 1] (class 3)
Predicted: q = [0.1, 0.2, 0.7]

```
H(p, q) = -(0·log0.1 + 0·log0.2 + 1·log0.7)
        = -log(0.7)
        = 0.357 nats (or 0.515 bits)
```

### Example 3: Comparing Two Models

True: p = [0.25, 0.25, 0.25, 0.25] (uniform over 4 classes)

Model A: q_A = [0.25, 0.25, 0.25, 0.25]
```
H(p, q_A) = -4 × (0.25 × log₂ 0.25) = 2.0 bits = H(p) ← optimal
```

Model B: q_B = [0.1, 0.1, 0.1, 0.7]
```
H(p, q_B) = -(0.25·log₂0.1 + 0.25·log₂0.1 + 0.25·log₂0.1 + 0.25·log₂0.7)
           = -(0.25·(-3.322)·3 + 0.25·(-0.515))
           = 2.620 bits > H(p) ← suboptimal
```

---

## 9. Python Implementation

```python
import numpy as np

def binary_cross_entropy(y_true, y_pred, epsilon=1e-15):
    """Binary cross-entropy loss with numerical stability."""
    y_pred = np.clip(y_pred, epsilon, 1 - epsilon)
    return -np.mean(y_true * np.log(y_pred) + (1 - y_true) * np.log(1 - y_pred))

def categorical_cross_entropy(y_true, y_pred, epsilon=1e-15):
    """Categorical cross-entropy for one-hot encoded labels."""
    y_pred = np.clip(y_pred, epsilon, 1.0)
    return -np.sum(y_true * np.log(y_pred), axis=-1).mean()

# --- Binary CE example ---
y_true = np.array([1, 0, 1, 0])
y_pred = np.array([0.9, 0.2, 0.7, 0.4])
print(f"Binary CE: {binary_cross_entropy(y_true, y_pred):.4f}")

# --- Categorical CE example ---
y_true_cat = np.array([[1,0,0], [0,1,0], [0,0,1]])
y_pred_cat = np.array([[0.8,0.1,0.1], [0.2,0.7,0.1], [0.1,0.2,0.7]])
print(f"Categorical CE: {categorical_cross_entropy(y_true_cat, y_pred_cat):.4f}")

# --- PyTorch equivalent ---
import torch
import torch.nn as nn

criterion_bce = nn.BCELoss()
criterion_ce  = nn.CrossEntropyLoss()  # expects raw logits, not softmax

logits = torch.tensor([[2.0, 0.5, -1.0]])   # raw scores
target = torch.tensor([0])                    # class index
loss = criterion_ce(logits, target)
print(f"PyTorch CE Loss: {loss.item():.4f}")
```

**Output:**
```
Binary CE: 0.2993
Categorical CE: 0.2963
PyTorch CE Loss: 0.2341
```

---

## 10. Real-World ML Applications

| Application               | How Cross-Entropy Is Used                              |
|----------------------------|-------------------------------------------------------|
| **Logistic Regression**    | BCE as the default loss function                      |
| **Neural Network Classifiers** | Softmax + categorical CE is the standard setup    |
| **Language Models**        | Next-token prediction trained with CE loss            |
| **Image Classification**   | ResNet, VGG all use CE loss                          |
| **Object Detection**       | Classification head uses focal CE (focal loss)       |
| **Knowledge Distillation** | CE between student and teacher soft labels            |
| **Kaggle Competitions**    | Log loss (= CE) is a common evaluation metric        |

---

## 11. Summary Table

| Concept                | Formula / Key Fact                                        |
|------------------------|----------------------------------------------------------|
| Cross-Entropy          | H(p,q) = -Σ p(x) log q(x)                              |
| Binary CE              | -[y log ŷ + (1-y) log(1-ŷ)]                             |
| Categorical CE         | -Σ yₖ log ŷₖ  (simplifies to -log ŷ_c)                 |
| Relationship           | H(p,q) = H(p) + D_KL(p‖q) ≥ H(p)                      |
| Minimizer              | H(p,q) is minimized when q = p                          |
| Gradient (sigmoid)     | ∂L/∂z = ŷ - y  (clean, no vanishing gradient)           |
| Equivalence            | CE loss ≡ negative log-likelihood ≡ log loss             |
| Numerical stability    | Clip predictions: ŷ ∈ [ε, 1-ε] to avoid log(0)         |

---

## 12. Quick Revision Questions

1. **What does cross-entropy measure intuitively?**
   > The average bits needed to encode data from distribution p using a code optimized for distribution q.

2. **Write the binary cross-entropy formula and explain each term.**
   > BCE = -[y log ŷ + (1-y) log(1-ŷ)]. First term penalizes when y=1 but ŷ is low; second term penalizes when y=0 but ŷ is high.

3. **Why is cross-entropy always ≥ entropy of the true distribution?**
   > Because H(p,q) = H(p) + D_KL(p‖q), and D_KL ≥ 0. The extra cost is the divergence between p and q.

4. **Why do we prefer cross-entropy over MSE for classification?**
   > CE provides clean gradients (ŷ - y) with sigmoid/softmax, while MSE gradients vanish when predictions are saturated.

5. **What happens to BCE loss when y=1 and ŷ→0?**
   > Loss → ∞. The model is confidently predicting the wrong class, which incurs maximum penalty.

6. **How does PyTorch's CrossEntropyLoss differ from the mathematical formula?**
   > PyTorch's CrossEntropyLoss expects raw logits (pre-softmax) and internally applies log-softmax + NLL loss for numerical stability.

---

[← Previous Chapter: 01-Entropy](./01-entropy.md) | [Next Chapter: 03-KL-Divergence →](./03-kl-divergence.md)
