# 🎯 Classification Loss Functions

> **Chapter 3.2 — Measuring How Wrong Your Class Predictions Are**

| Topic | Details |
|---|---|
| **Subject** | Loss Functions — Classification Losses |
| **Prerequisites** | Probability basics, Chapter 3.1 |
| **Key Insight** | Cross-entropy measures the "distance" between two probability distributions — the true labels and the predicted probabilities |

---

## 📌 Overview

Classification losses measure how well predicted probabilities match the true class labels. Unlike regression (where error is a distance), classification loss is rooted in **information theory** — specifically, the concept of cross-entropy between distributions. This chapter covers binary and categorical cross-entropy, their mathematical foundations, numerical stability tricks, and advanced variants like focal loss for handling class imbalance.

---

## 1. Binary Cross-Entropy (BCE)

### 1.1 Formula

```
    For a single sample:
    ℓ(y, ŷ) = -[y · log(ŷ) + (1 - y) · log(1 - ŷ)]

    For m samples (averaged):
                    1   m
    L_BCE = - ─── Σ  [yᵢ · log(ŷᵢ) + (1 - yᵢ) · log(1 - ŷᵢ)]
                    m  i=1

    Where:
        y  ∈ {0, 1}     — true label
        ŷ  ∈ (0, 1)     — predicted probability (sigmoid output)
```

### 1.2 Intuition — Why This Formula?

```
    When y = 1 (positive class):
        ℓ = -log(ŷ)
        
        If ŷ → 1:  ℓ = -log(1) = 0      ← Correct! No penalty
        If ŷ → 0:  ℓ = -log(0) → ∞      ← Wrong! Infinite penalty
    
    When y = 0 (negative class):
        ℓ = -log(1 - ŷ)
        
        If ŷ → 0:  ℓ = -log(1) = 0      ← Correct! No penalty
        If ŷ → 1:  ℓ = -log(0) → ∞      ← Wrong! Infinite penalty

    The loss PUNISHES confident wrong predictions severely!
```

### 1.3 ASCII Plot

```
    Loss ℓ
    │
    │ ╲
   5┤  ╲                            When y=1:  ℓ = -log(ŷ)
    │   ╲                           When y=0:  ℓ = -log(1-ŷ)
   4┤    ╲
    │     ╲                    ╱
   3┤      ╲                 ╱
    │       ╲              ╱
   2┤        ╲           ╱
    │         ╲        ╱
   1┤          ╲╲    ╱╱
    │           ╲╲ ╱╱
   0┤ · · · · · ·╲╱· · · · · ·
    ┼──────────────●──────────── ŷ
    0             0.5           1

    Left curve:  -log(ŷ)    (y=1, want ŷ→1)
    Right curve: -log(1-ŷ)  (y=0, want ŷ→0)
```

### 1.4 Gradient

```
    ∂ℓ       y     1 - y       ŷ - y
    ── = - ─── + ──────── = ──────────
    ∂ŷ      ŷ     1 - ŷ      ŷ(1 - ŷ)

    When combined with sigmoid output (ŷ = σ(z)):

    ∂ℓ
    ── = ŷ - y = σ(z) - y
    ∂z

    ┌─────────────────────────────────────────────────────────┐
    │  This elegant simplification is why sigmoid + BCE is    │
    │  the standard for binary classification!                │
    │  The gradient is simply: (prediction - truth)           │
    └─────────────────────────────────────────────────────────┘
```

### 1.5 Worked Example

```
    Predictions for 4 samples (after sigmoid):
    
    Sample │  y  │   ŷ    │  -y·log(ŷ)  │ -(1-y)·log(1-ŷ) │ Loss ℓ
    ───────┼─────┼────────┼─────────────┼──────────────────┼────────
      1    │  1  │ 0.90   │ -1·(-0.105) │  0·(-2.303)      │ 0.105
      2    │  0  │ 0.10   │  0·(-2.303) │ -1·(-0.105)      │ 0.105
      3    │  1  │ 0.60   │ -1·(-0.511) │  0·(-0.916)      │ 0.511
      4    │  0  │ 0.80   │  0·(-0.223) │ -1·(-1.609)      │ 1.609
                                                              ──────
    BCE = (0.105 + 0.105 + 0.511 + 1.609) / 4 = 0.5825

    Note: Sample 4 has the highest loss — predicted 0.80 but true label is 0!
    The model is confidently wrong → heavily penalized.
```

---

## 2. Categorical Cross-Entropy (Multi-Class)

### 2.1 Formula

```
    For a single sample with K classes:
                    K
    ℓ(y, ŷ) = - Σ  yₖ · log(ŷₖ)
                   k=1

    Where:
        y  = one-hot encoded label: [0, 0, 1, 0, 0]  (only one yₖ = 1)
        ŷ  = softmax output: [0.05, 0.10, 0.72, 0.08, 0.05] (sums to 1)

    Since only one yₖ = 1 (say class c):
    ℓ = -log(ŷc)

    Interpretation: "How surprised are we by the true class?"
    High ŷc → low loss (not surprised)
    Low ŷc  → high loss (very surprised!)
```

### 2.2 For a Batch of m Samples

```
                       1   m    K
    L_CE = - ─── Σ   Σ  yₖ⁽ⁱ⁾ · log(ŷₖ⁽ⁱ⁾)
                       m  i=1  k=1

    Which simplifies (since each y is one-hot) to:
                       1   m
    L_CE = - ─── Σ  log(ŷc⁽ⁱ⁾)
                       m  i=1

    Where c⁽ⁱ⁾ is the true class of sample i.
```

### 2.3 Connection to Negative Log-Likelihood

```
    Cross-entropy with one-hot labels = Negative Log-Likelihood (NLL)

    If the model outputs P(class=k | x) = ŷₖ,
    then the likelihood of the correct class c is ŷc.

    NLL = -log(ŷc) = -log(P(correct class | x))

    Minimizing NLL = Maximizing likelihood of correct predictions
    
    ┌────────────────────────────────────────────────────────┐
    │  This is why PyTorch's CrossEntropyLoss actually       │
    │  computes log_softmax + NLL under the hood.            │
    └────────────────────────────────────────────────────────┘
```

### 2.4 Worked Example

```
    3-class classification, 2 samples:

    Sample 1:
        True class: 2 (one-hot: [0, 0, 1])
        Softmax output: [0.1, 0.2, 0.7]
        Loss = -log(0.7) = 0.357

    Sample 2:
        True class: 0 (one-hot: [1, 0, 0])
        Softmax output: [0.05, 0.85, 0.10]
        Loss = -log(0.05) = 2.996

    Average CE = (0.357 + 2.996) / 2 = 1.677

    Sample 2 has much higher loss — the model was very confident
    about class 1, but the true class was 0!
```

### 2.5 Gradient of Cross-Entropy with Softmax

```
    When using softmax output + cross-entropy loss:

    ∂L
    ──── = ŷₖ - yₖ     (for each class k)
    ∂zₖ

    This is the same elegant form as sigmoid + BCE!

    ∂L/∂z = ŷ - y = [ŷ₁-y₁, ŷ₂-y₂, ..., ŷₖ-yₖ]

    For the true class c: gradient = ŷc - 1  (negative → push up)
    For other classes k≠c: gradient = ŷₖ - 0 = ŷₖ  (positive → push down)
```

---

## 3. Numerical Stability — The Log-Sum-Exp Trick

### 3.1 The Problem

```
    Naive softmax:
    ŷₖ = exp(zₖ) / Σⱼ exp(zⱼ)

    If z = [1000, 1001, 999]:
    exp(1000) = Inf!  → Overflow → NaN → Training crashes

    If z = [-1000, -1001, -999]:
    exp(-1000) = 0   → log(0) = -Inf → Underflow
```

### 3.2 The Solution

```
    Log-Sum-Exp trick:
    
    log(Σ exp(zⱼ)) = max(z) + log(Σ exp(zⱼ - max(z)))

    Then: log(ŷₖ) = zₖ - max(z) - log(Σ exp(zⱼ - max(z)))

    Example with z = [1000, 1001, 999]:
    max(z) = 1001
    z - max(z) = [-1, 0, -2]
    
    exp([-1, 0, -2]) = [0.368, 1.000, 0.135]  ← No overflow!
    Σ = 1.503
    log(Σ) = 0.407
    
    log(ŷ) = z - 1001 - 0.407 = [-1.407, -0.407, -2.407]
    ŷ = [0.245, 0.665, 0.090]  ← Correct and stable!
```

### 3.3 Why PyTorch Combines Softmax + CE

```python
# ❌ NUMERICALLY UNSTABLE:
probs = torch.softmax(logits, dim=1)     # Can overflow
loss = -torch.log(probs[range(m), labels])  # Can underflow (log(0))
loss = loss.mean()

# ✅ NUMERICALLY STABLE (PyTorch does this internally):
loss = nn.CrossEntropyLoss()(logits, labels)
# Internally: log_softmax(logits) + nll_loss
# Uses log-sum-exp trick — no overflow/underflow!
```

---

## 4. Label Smoothing

### 4.1 Concept

Instead of hard labels (one-hot), use "soft" labels that don't fully commit to 100%:

```
    Hard labels:    y = [0, 0, 1, 0, 0]
    Smooth labels:  y = [0.02, 0.02, 0.92, 0.02, 0.02]

    Formula: y_smooth = y · (1 - ε) + ε/K

    Where ε = smoothing parameter (typically 0.1)
    K = number of classes
```

### 4.2 Benefits

```
    Without smoothing:              With smoothing (ε=0.1):
    Model tries to output           Model aims for
    [0, 0, 1, 0, 0]               [0.02, 0.02, 0.92, 0.02, 0.02]
    
    → Pushes logits to ±∞          → Moderate logits sufficient
    → Overly confident              → Calibrated confidence
    → Poor generalization            → Better generalization
    → Sensitive to label noise       → Robust to label noise
```

```python
# PyTorch label smoothing
criterion = nn.CrossEntropyLoss(label_smoothing=0.1)
```

---

## 5. Focal Loss (for Class Imbalance)

### 5.1 The Problem

```
    In many real-world datasets, classes are heavily imbalanced:
    
    Medical diagnosis: 99% healthy, 1% diseased
    Fraud detection:   99.9% legitimate, 0.1% fraud
    Object detection:  99% background, 1% objects
    
    With standard CE, the model learns to always predict the majority class
    (99% accuracy by always saying "healthy" — but useless!)
```

### 5.2 Focal Loss Formula

```
    Standard CE:  ℓ = -log(pₜ)
    Focal Loss:   ℓ = -(1 - pₜ)^γ · log(pₜ)

    Where:
        pₜ = predicted probability of the TRUE class
        γ  = focusing parameter (typically γ = 2)

    The (1 - pₜ)^γ term is a modulating factor:
    
    • Easy examples (pₜ high, e.g., 0.9):
      (1 - 0.9)² = 0.01 → loss reduced by 100x!
      
    • Hard examples (pₜ low, e.g., 0.1):
      (1 - 0.1)² = 0.81 → loss barely reduced
    
    Result: Model focuses on HARD examples, ignoring easy ones
```

### 5.3 ASCII Comparison

```
    Loss
    │
    │╲                     CE (γ=0)
   5┤ ╲
    │  ╲              
   4┤   ╲
    │    ╲                 Focal (γ=1)
   3┤     ╲            ╱╱
    │      ╲╲       ╱╱╱
   2┤       ╲╲   ╱╱╱
    │        ╲╲╱╱╱         Focal (γ=2)
   1┤         ●╱╱      ╱╱╱╱╱
    │       ╱╱╱╱    ╱╱╱╱╱
   0┤ ╱╱╱╱╱╱╱╱ ╱╱╱╱╱
    ┼──────────────────────── pₜ
    0        0.5           1

    Higher γ → more focus on hard (low pₜ) examples
    Easy examples (high pₜ) contribute almost nothing to loss
```

```python
class FocalLoss(nn.Module):
    def __init__(self, alpha=1.0, gamma=2.0):
        super().__init__()
        self.alpha = alpha
        self.gamma = gamma
    
    def forward(self, logits, targets):
        ce_loss = F.cross_entropy(logits, targets, reduction='none')
        pt = torch.exp(-ce_loss)  # probability of true class
        focal_loss = self.alpha * (1 - pt) ** self.gamma * ce_loss
        return focal_loss.mean()
```

---

## 6. Complete Python Implementation

```python
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F

class ClassificationLosses:
    """NumPy implementations of classification losses."""
    
    @staticmethod
    def binary_cross_entropy(y_true, y_pred, eps=1e-7):
        """Binary Cross-Entropy Loss"""
        y_pred = np.clip(y_pred, eps, 1 - eps)  # Numerical stability
        return -np.mean(
            y_true * np.log(y_pred) + (1 - y_true) * np.log(1 - y_pred)
        )
    
    @staticmethod
    def categorical_cross_entropy(y_true_onehot, y_pred, eps=1e-7):
        """Categorical Cross-Entropy Loss"""
        y_pred = np.clip(y_pred, eps, 1.0)
        return -np.mean(np.sum(y_true_onehot * np.log(y_pred), axis=1))
    
    @staticmethod
    def sparse_cross_entropy(y_true_indices, y_pred, eps=1e-7):
        """Cross-Entropy with integer labels (not one-hot)"""
        y_pred = np.clip(y_pred, eps, 1.0)
        m = len(y_true_indices)
        log_probs = np.log(y_pred[np.arange(m), y_true_indices])
        return -np.mean(log_probs)
    
    @staticmethod
    def focal_loss(y_true_indices, y_pred, gamma=2.0, eps=1e-7):
        """Focal Loss for class imbalance"""
        y_pred = np.clip(y_pred, eps, 1.0)
        m = len(y_true_indices)
        pt = y_pred[np.arange(m), y_true_indices]
        focal_weight = (1 - pt) ** gamma
        return -np.mean(focal_weight * np.log(pt))


# ──── Demo ────
losses = ClassificationLosses()

# Binary Cross-Entropy
print("=" * 60)
print("BINARY CROSS-ENTROPY")
print("=" * 60)
y_binary = np.array([1, 0, 1, 0, 1])
y_pred_good = np.array([0.9, 0.1, 0.8, 0.2, 0.95])
y_pred_bad = np.array([0.3, 0.7, 0.4, 0.6, 0.5])

print(f"Good predictions:  BCE = {losses.binary_cross_entropy(y_binary, y_pred_good):.4f}")
print(f"Bad predictions:   BCE = {losses.binary_cross_entropy(y_binary, y_pred_bad):.4f}")

# Categorical Cross-Entropy
print(f"\n{'=' * 60}")
print("CATEGORICAL CROSS-ENTROPY (3 classes)")
print("=" * 60)

# True labels (integer)
y_labels = np.array([2, 0, 1])

# Predicted probabilities (softmax outputs)
y_pred_conf = np.array([[0.05, 0.10, 0.85],   # Confident correct
                         [0.70, 0.20, 0.10],   # Confident correct
                         [0.10, 0.80, 0.10]])  # Confident correct

y_pred_unsure = np.array([[0.30, 0.35, 0.35],   # Uncertain
                           [0.35, 0.30, 0.35],   # Uncertain
                           [0.35, 0.35, 0.30]])  # Uncertain

y_pred_wrong = np.array([[0.80, 0.10, 0.10],   # Confidently WRONG
                          [0.10, 0.10, 0.80],   # Confidently WRONG
                          [0.80, 0.10, 0.10]])  # Confidently WRONG

print(f"Confident & correct:  CE = {losses.sparse_cross_entropy(y_labels, y_pred_conf):.4f}")
print(f"Uncertain:            CE = {losses.sparse_cross_entropy(y_labels, y_pred_unsure):.4f}")
print(f"Confident & WRONG:    CE = {losses.sparse_cross_entropy(y_labels, y_pred_wrong):.4f}")

# Focal Loss comparison
print(f"\n{'=' * 60}")
print("FOCAL LOSS vs STANDARD CE")
print("=" * 60)

y_labels_focal = np.array([0, 0, 0, 1])
y_pred_mixed = np.array([[0.95, 0.05],   # Easy correct
                          [0.90, 0.10],   # Easy correct
                          [0.80, 0.20],   # Easy correct
                          [0.30, 0.70]])  # Hard correct (minority)

print(f"Standard CE:     {losses.sparse_cross_entropy(y_labels_focal, y_pred_mixed):.4f}")
print(f"Focal (γ=1):     {losses.focal_loss(y_labels_focal, y_pred_mixed, gamma=1):.4f}")
print(f"Focal (γ=2):     {losses.focal_loss(y_labels_focal, y_pred_mixed, gamma=2):.4f}")
print(f"Focal (γ=5):     {losses.focal_loss(y_labels_focal, y_pred_mixed, gamma=5):.4f}")
print("→ Higher γ down-weights easy examples more aggressively")
```

---

## 7. PyTorch Loss Functions

```python
import torch
import torch.nn as nn

# ── Binary Classification ──
bce_loss = nn.BCELoss()              # Expects sigmoid output
bce_logit = nn.BCEWithLogitsLoss()   # Expects raw logits (more stable)

logits = torch.tensor([2.0, -1.0, 0.5])
targets = torch.tensor([1.0, 0.0, 1.0])

# BCEWithLogitsLoss = sigmoid + BCE (numerically stable)
loss = bce_logit(logits, targets)
print(f"BCE (from logits): {loss.item():.4f}")

# ── Multi-Class Classification ──
ce_loss = nn.CrossEntropyLoss()      # Expects raw logits, integer labels

logits = torch.tensor([[2.0, 0.5, -1.0],
                       [-0.5, 3.0, 0.5],
                       [0.5, 0.5, 2.0]])
labels = torch.tensor([0, 1, 2])

loss = ce_loss(logits, labels)
print(f"CrossEntropy: {loss.item():.4f}")

# With label smoothing
ce_smooth = nn.CrossEntropyLoss(label_smoothing=0.1)
loss_smooth = ce_smooth(logits, labels)
print(f"CE + smoothing: {loss_smooth.item():.4f}")

# With class weights (for imbalance)
weights = torch.tensor([1.0, 2.0, 3.0])  # Weight rare classes higher
ce_weighted = nn.CrossEntropyLoss(weight=weights)
loss_weighted = ce_weighted(logits, labels)
print(f"Weighted CE: {loss_weighted.item():.4f}")

# ── NLL Loss (for use with log_softmax) ──
nll_loss = nn.NLLLoss()
log_probs = F.log_softmax(logits, dim=1)
loss_nll = nll_loss(log_probs, labels)
print(f"NLL Loss: {loss_nll.item():.4f}")
print(f"CE == NLL? {torch.allclose(loss, loss_nll)}")  # True!
```

---

## 📊 Summary Table

| Loss Function | Formula | Use Case | Activation |
|---|---|---|---|
| **BCE** | -[y·log(ŷ)+(1-y)·log(1-ŷ)] | Binary classification | Sigmoid |
| **BCEWithLogits** | BCE applied to σ(z) | Binary (numerically stable) | None (built-in) |
| **Cross-Entropy** | -Σyₖ·log(ŷₖ) | Multi-class | Softmax |
| **NLL** | -log(ŷc) | Equivalent to CE with one-hot | log_softmax |
| **Focal Loss** | -(1-pₜ)^γ · log(pₜ) | Class imbalance | Softmax |
| **Label Smoothing** | CE with soft targets | Reduce overconfidence | Softmax |

---

## ❓ Revision Questions

1. **Derive the binary cross-entropy loss from first principles. Why does -log(ŷ) make sense as a penalty for y=1?**

2. **Compute the categorical cross-entropy for true class 2 and softmax output [0.1, 0.3, 0.5, 0.1]. What would the loss be if the model were perfect?**

3. **Explain the log-sum-exp trick. Why is `nn.CrossEntropyLoss` more numerically stable than manually computing softmax + log?**

4. **What is focal loss? Given γ=2 and a sample with pₜ=0.9 vs pₜ=0.1, compute the focal weight for each. How does this help with class imbalance?**

5. **Show that the gradient of cross-entropy loss with softmax output simplifies to (ŷ - y). Why is this property desirable?**

6. **What is label smoothing? Why might a model that targets [0.02, 0.02, 0.92, 0.02, 0.02] generalize better than one targeting [0, 0, 1, 0, 0]?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Regression Losses](01-regression-losses.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Loss Function Selection →](03-loss-function-selection.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
