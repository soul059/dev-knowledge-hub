# 1. L1 and L2 Regularization

> **Unit 7 · Regularization Techniques** — Constraining weights to prevent overfitting

---

## Chapter Overview

Regularization is the art of **reducing overfitting** — when a model memorizes training data instead of learning generalizable patterns. The simplest and most mathematically elegant forms of regularization are **L2** (weight decay) and **L1** (lasso), which add a penalty term to the loss function that discourages large or unnecessary weights. This chapter covers both methods in depth, their gradient effects, sparsity properties, and practical PyTorch usage.

---

## 1. The Overfitting Problem

### Bias-Variance Trade-off

```
  Model Complexity →
  
  Error ↑
       │╲                    ╱
       │ ╲                  ╱   Test Error
       │  ╲      ╱────╲   ╱    (high variance)
       │   ╲    ╱      ╲╱
       │    ╲  ╱         
       │     ╲╱  optimal         ← sweet spot
       │    ╱               
       │   ╱  Train Error
       │  ╱   (low bias)
       │╱
       └──────────────────────→ Complexity
       
  Under-     Just      Over-
  fitting    Right     fitting
```

### How Regularization Helps

By penalizing large weights, regularization:
1. **Reduces model effective complexity** — simpler models generalize better
2. **Prevents any single weight from dominating** — distributes importance
3. **Smooths the learned function** — reduces oscillations

---

## 2. L2 Regularization (Ridge / Weight Decay)

### Modified Loss Function

```
  J_reg(θ) = J(θ) + λ · Σᵢ wᵢ²
             ↑          ↑
         Original    L2 penalty
         loss        (sum of squared weights)
```

Or in vector notation:

```
  J_reg(θ) = J(θ) + λ ||w||₂²
  
  where ||w||₂² = w₁² + w₂² + w₃² + ... + wₙ²
```

### Gradient with L2

```
  ∂J_reg/∂wᵢ = ∂J/∂wᵢ + 2λwᵢ
```

The update rule becomes:

```
  wᵢ ← wᵢ - α(∂J/∂wᵢ + 2λwᵢ)
     = wᵢ(1 - 2αλ) - α · ∂J/∂wᵢ
       ↑                    ↑
   weight shrinkage    normal gradient step
```

### Key Insight: Proportional Shrinkage

```
  L2 gradient penalty = 2λw
  
  ┌─────────────────────────────────────────────────┐
  │  Large weight w = 10  →  penalty = 20λ (large)  │
  │  Small weight w = 0.1 →  penalty = 0.2λ (small) │
  │                                                  │
  │  L2 shrinks all weights PROPORTIONALLY           │
  │  toward zero, but never exactly to zero.         │
  └─────────────────────────────────────────────────┘
```

### Geometric Interpretation

```
  Without regularization:        With L2 regularization:
  
  w₂ ↑                           w₂ ↑
     │    ╱─loss contour          │    ╱─loss contour
     │   ╱ ╱                      │   ╱ ╱   ╭─╮ constraint
     │  ╱ ╱ ★ optimal             │  ╱ ╱  ╱   ╲  (circle)
     │ ╱ ╱                        │ ╱ ╱ ★╱     │ ← intersection
     │╱ ╱                         │╱ ╱  ╲     ╱    = regularized
     └──────→ w₁                  └──╰───╰─╯──→ w₁
                                        ↑
  Free to use any weights         Weights constrained to
                                  lie within circle ||w||₂ ≤ C
```

The regularized solution lies where the loss contours **first touch** the constraint circle. This pushes the solution toward **smaller weights**.

---

## 3. L1 Regularization (Lasso)

### Modified Loss Function

```
  J_reg(θ) = J(θ) + λ · Σᵢ |wᵢ|
             ↑          ↑
         Original    L1 penalty
         loss        (sum of absolute values)
```

### Gradient with L1

```
  ∂J_reg/∂wᵢ = ∂J/∂wᵢ + λ · sign(wᵢ)
  
  where sign(w) = { +1  if w > 0
                  { -1  if w < 0
                  {  0  if w = 0
```

The update rule:

```
  wᵢ ← wᵢ - α(∂J/∂wᵢ + λ · sign(wᵢ))
     = wᵢ - α · λ · sign(wᵢ) - α · ∂J/∂wᵢ
       ↑                          ↑
   constant push toward 0    normal gradient step
```

### Key Insight: Constant Shrinkage → Sparsity

```
  L1 gradient penalty = λ · sign(w)
  
  ┌──────────────────────────────────────────────────┐
  │  Large weight w = 10  →  penalty = λ  (constant) │
  │  Small weight w = 0.1 →  penalty = λ  (SAME!)    │
  │                                                   │
  │  L1 applies a CONSTANT force toward zero.         │
  │  Small weights get pushed all the way to zero!    │
  │  This creates SPARSE models (many weights = 0).   │
  └──────────────────────────────────────────────────┘
```

### Geometric Interpretation

```
  L1 constraint is a DIAMOND (not circle):
  
  w₂ ↑
     │    ╱─loss contour
     │   ╱ ╱      ╱╲
     │  ╱ ╱      ╱  ╲  ← diamond
     │ ╱ ╱   ★──╱    ╲    constraint
     │╱ ╱      ╲      │
     └──╱────────╲────╱──→ w₁
       ╱          ╲╱
  
  The diamond has CORNERS on the axes!
  Loss contours are most likely to touch at a corner.
  At corners: one or more weights = 0 → SPARSITY!
```

---

## 4. L1 vs L2: Side-by-Side Comparison

### Gradient Behavior

```
  L2 Gradient Penalty:              L1 Gradient Penalty:
  
  penalty ↑                         penalty ↑
          │        ╱                         │    ╱
          │      ╱                           │   ╱
          │    ╱                              │  ╱
          │  ╱                                │ ╱
  ────────┼╱──────→ w              ──────────┼╱────────→ w
        ╱ │                              ╱   │
      ╱   │                             ╱    │
    ╱     │                            ╱     │
  ╱       │                                  │
  Smooth, proportional                Constant magnitude,
  to weight magnitude                 discontinuous at 0
```

### Weight Distributions After Training

```
  Without Regularization:           L2 Regularization:
  
  count ↑                           count ↑
        │    ╱╲                           │   ╱╲
        │   ╱  ╲                          │  ╱  ╲
        │  ╱    ╲                         │ ╱    ╲
        │ ╱      ╲                        │╱      ╲
        │╱        ╲                       ╱        ╲
   ─────┼──────────────→ w         ──────┼──────────→ w
       -2   0    2                     -1   0    1
  
  Spread out                        Compressed toward 0
                                    (but none exactly 0)
  
  L1 Regularization:
  
  count ↑
        │ █                          ← many weights EXACTLY 0!
        │ █
        │ █  ╱╲
        │ █ ╱  ╲
        │ █╱    ╲
   ─────┼──────────→ w
       -1  0    1
  
  SPARSE: many zeros + a few large weights
```

### Comprehensive Comparison Table

| Property | L2 (Ridge) | L1 (Lasso) |
|---|---|---|
| **Penalty** | λΣwᵢ² | λΣ\|wᵢ\| |
| **Gradient** | 2λw (proportional) | λ·sign(w) (constant) |
| **Shrinkage** | Proportional to weight | Constant push |
| **Sparsity** | No (weights → 0 but never reach) | **Yes (weights become exactly 0)** |
| **Constraint shape** | Circle (L2 ball) | Diamond (L1 ball) |
| **Feature selection** | No | **Yes (zeroed weights = removed features)** |
| **Smoothness** | Smooth at w=0 | Non-smooth at w=0 |
| **Computation** | Efficient | Slightly less efficient |
| **Common name** | Weight decay, Ridge | Lasso |
| **Typical use** | Most deep learning | Feature selection, sparse models |

---

## 5. Elastic Net: Combining L1 and L2

```
  J_reg(θ) = J(θ) + λ₁ Σ|wᵢ| + λ₂ Σwᵢ²
  
  Or equivalently with mixing ratio r ∈ [0, 1]:
  
  J_reg(θ) = J(θ) + λ [r · Σ|wᵢ| + (1-r) · Σwᵢ²]
```

- r = 0: Pure L2
- r = 1: Pure L1
- r = 0.5: Equal mix

Elastic net provides **sparsity** (from L1) with **stability** (from L2, which handles correlated features better).

---

## 6. Choosing λ (Regularization Strength)

### Effect of λ

```
  λ too small:                λ just right:              λ too large:
  
  Loss ↑                      Loss ↑                     Loss ↑
       │  ╲                        │  ╲                       │ ╲
       │   ╲ . .                   │   ╲                      │  ╲╲
       │    ╲   . . .              │    ╲ ╲                   │   ╲ ╲
       │     ╲      . . .         │     ╲  ╲                 │    ╲  ╲─── val
       │      ╲            . .    │      ╲──╲── val          │     ╲───── train
       │       ╲───── val         │         ╲── train        │
       │        ╲───── train      │                          │
       └──────────→ epoch         └──────────→ epoch         └──────────→ epoch
       
  Still overfitting            Good generalization        Underfitting
  (gap between train/val)     (small gap)                 (both losses high)
```

### Typical Values

| Context | λ Range | Notes |
|---|---|---|
| Deep learning (weight decay) | 1e-5 to 1e-2 | Common: 1e-4 |
| Linear regression | 0.001 to 100 | Use cross-validation |
| NLP models | 1e-5 to 1e-3 | Lighter regularization |
| Vision models | 1e-4 to 5e-4 | Standard for ResNet |

---

## 7. Worked Example

### Problem
Linear model: ŷ = w₁x₁ + w₂x₂, current weights w = (3, 0.1), gradient ∂J/∂w = (0.5, -0.2), α = 0.1, λ = 0.01.

### L2 Update

```
  L2 gradient: ∂J/∂w + 2λw = (0.5 + 0.02·3, -0.2 + 0.02·0.1) = (0.56, -0.198)
  
  w₁: 3 - 0.1·0.56 = 2.944       (large weight shrinks more)
  w₂: 0.1 - 0.1·(-0.198) = 0.1198 (small weight barely affected)
```

### L1 Update

```
  L1 gradient: ∂J/∂w + λ·sign(w) = (0.5 + 0.01, -0.2 + 0.01) = (0.51, -0.19)
  
  w₁: 3 - 0.1·0.51 = 2.949       (same constant push regardless of magnitude)
  w₂: 0.1 - 0.1·(-0.19) = 0.119  (same push as w₁!)
```

---

## 8. Python Implementation (PyTorch)

### L2 Regularization via weight_decay

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

# ── Data ────────────────────────────────────────────────────────
torch.manual_seed(42)
X = torch.randn(500, 50)   # small dataset (overfitting likely)
y = (X[:, :5] @ torch.randn(5, 1)).squeeze()  # only 5 features matter
loader = DataLoader(TensorDataset(X, y), batch_size=32, shuffle=True)

# ── Train with different regularization ────────────────────────
def train(wd, epochs=100):
    torch.manual_seed(0)
    model = nn.Sequential(nn.Linear(50, 128), nn.ReLU(),
                          nn.Linear(128, 64), nn.ReLU(),
                          nn.Linear(64, 1))
    # weight_decay parameter IS L2 regularization
    optimizer = torch.optim.Adam(model.parameters(), lr=0.001, weight_decay=wd)
    criterion = nn.MSELoss()
    
    train_losses, sparsity = [], []
    for epoch in range(epochs):
        total = 0
        for xb, yb in loader:
            loss = criterion(model(xb).squeeze(), yb)
            optimizer.zero_grad(); loss.backward(); optimizer.step()
            total += loss.item() * len(xb)
        train_losses.append(total / len(X))
        
        # Count "near-zero" weights
        all_weights = torch.cat([p.flatten() for p in model.parameters()])
        near_zero = (all_weights.abs() < 0.01).float().mean().item()
        sparsity.append(near_zero)
    
    return train_losses, sparsity

# Compare
results = {}
for wd in [0, 1e-4, 1e-2, 1.0]:
    losses, sparse = train(wd)
    results[wd] = (losses, sparse)
    print(f"wd={wd:.0e}: final_loss={losses[-1]:.4f}, near_zero={sparse[-1]:.1%}")
```

### Manual L1 Regularization

```python
def train_l1(lambda_l1, epochs=100):
    """L1 regularization must be added manually in PyTorch."""
    torch.manual_seed(0)
    model = nn.Sequential(nn.Linear(50, 128), nn.ReLU(),
                          nn.Linear(128, 64), nn.ReLU(),
                          nn.Linear(64, 1))
    optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
    criterion = nn.MSELoss()
    
    losses = []
    for epoch in range(epochs):
        total = 0
        for xb, yb in loader:
            pred = model(xb).squeeze()
            loss = criterion(pred, yb)
            
            # Add L1 penalty manually
            l1_penalty = sum(p.abs().sum() for p in model.parameters())
            total_loss = loss + lambda_l1 * l1_penalty
            
            optimizer.zero_grad()
            total_loss.backward()
            optimizer.step()
            total += loss.item() * len(xb)
        losses.append(total / len(X))
    
    # Check sparsity (exact zeros)
    all_weights = torch.cat([p.flatten() for p in model.parameters()])
    exact_zero = (all_weights == 0).float().mean().item()
    near_zero = (all_weights.abs() < 0.01).float().mean().item()
    
    return losses, exact_zero, near_zero

for lam in [0, 1e-5, 1e-4, 1e-3]:
    losses, exact, near = train_l1(lam)
    print(f"L1 λ={lam:.0e}: loss={losses[-1]:.4f}, exact_zero={exact:.1%}, near_zero={near:.1%}")
```

### Elastic Net

```python
def train_elastic(lambda_l1, lambda_l2, epochs=100):
    """Elastic Net = L1 + L2."""
    torch.manual_seed(0)
    model = nn.Sequential(nn.Linear(50, 128), nn.ReLU(),
                          nn.Linear(128, 64), nn.ReLU(),
                          nn.Linear(64, 1))
    # L2 via weight_decay
    optimizer = torch.optim.Adam(model.parameters(), lr=0.001, weight_decay=lambda_l2)
    criterion = nn.MSELoss()
    
    for epoch in range(epochs):
        for xb, yb in loader:
            loss = criterion(model(xb).squeeze(), yb)
            # L1 manually
            l1 = sum(p.abs().sum() for p in model.parameters())
            (loss + lambda_l1 * l1).backward()
            optimizer.step(); optimizer.zero_grad()

elastic_losses = train_elastic(lambda_l1=1e-5, lambda_l2=1e-4)
```

---

## 9. Applications

| Application | Regularization | Reason |
|---|---|---|
| **Computer vision (CNNs)** | L2 (weight_decay=1e-4) | Standard, prevents large filters |
| **NLP embeddings** | L2 (small) | Prevents embedding explosion |
| **Feature selection** | L1 | Zeroes out irrelevant features |
| **Sparse autoencoders** | L1 on activations | Encourages sparse representations |
| **Transfer learning** | Small L2 | Keeps weights near pretrained values |
| **Linear models** | Elastic Net | Sparsity + stability |

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| L2 penalty | λΣwᵢ² (sum of squared weights) |
| L2 gradient | 2λw (proportional shrinkage) |
| L2 update | w ← w(1-2αλ) - α·∂J/∂w |
| L1 penalty | λΣ\|wᵢ\| (sum of absolute values) |
| L1 gradient | λ·sign(w) (constant push to zero) |
| L1 sparsity | Small weights pushed exactly to 0 |
| Elastic Net | λ₁Σ\|w\| + λ₂Σw² (combine both) |
| L2 constraint shape | Circle (L2 ball) |
| L1 constraint shape | Diamond (corners → sparsity) |
| PyTorch L2 | `optimizer = Adam(params, weight_decay=λ)` |
| PyTorch L1 | Manual: `loss += λ * Σ\|params\|` |

---

## Revision Questions

1. **Derive the gradient update rule for L2 regularization.** Show that it's equivalent to multiplying weights by (1-2αλ) each step (hence "weight decay").

2. **Explain geometrically why L1 produces sparse solutions but L2 does not.** Draw the constraint regions and show where loss contours intersect.

3. **Compare L1 and L2 gradient penalties for weights w=10 and w=0.01.** Which method applies more relative force to the small weight?

4. **Implement elastic net regularization in PyTorch** combining `weight_decay` for L2 and manual L1 penalty. Train on a dataset with 50 features where only 5 are relevant.

5. **How do you choose the regularization strength λ?** Describe a practical procedure using validation performance.

6. **Why is L2 regularization equivalent to weight decay in SGD but not in Adam?** (Hint: see Unit 5, Adam chapter.)

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Unit 6: LSUV](../06-Weight-Initialization/06-lsuv.md) | [Unit 7: Regularization](./README.md) | [Dropout →](./02-dropout.md) |
