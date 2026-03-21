# 7. Weight Decay vs L2 Regularization

> **Unit 7 · Regularization Techniques** — Understanding the subtle but important distinction

---

## Chapter Overview

**Weight decay** and **L2 regularization** are often used interchangeably, but they are **only equivalent for vanilla SGD**. For adaptive optimizers like Adam, they produce fundamentally different behavior. This chapter clarifies the distinction, explains **decoupled weight decay** (AdamW), and provides practical guidance on choosing and tuning the weight decay parameter.

---

## 1. The Two Concepts

### L2 Regularization

Add a penalty term to the **loss function**:

```
  J_reg(θ) = J(θ) + (λ/2) ||θ||²
  
  Gradient: ∇J_reg = ∇J + λθ
```

The penalty term modifies the **gradient** used by the optimizer.

### Weight Decay

Directly shrink weights toward zero at each step, **separate from** the gradient:

```
  Update: θ ← (1 - αλ) · θ - α · ∇J(θ)
              ↑                  ↑
        decay term         gradient term
```

The shrinkage is applied **directly to θ**, not through the loss function.

### In SGD: They Are Equivalent

```
  L2 Regularization with SGD:
  θ ← θ - α(∇J + λθ)
    = θ - α·∇J - αλθ
    = (1-αλ)θ - α·∇J       ← same as weight decay!
  
  Weight Decay with SGD:
  θ ← (1-αλ)θ - α·∇J       ← identical!
  
  ∴ For SGD: L2 ≡ Weight Decay (with appropriate λ scaling)
```

---

## 2. Why They Differ in Adam

### L2 Regularization in Adam

When we add L2 to the loss, the gradient becomes g̃ = ∇J + λθ. Adam processes this:

```
  g̃ = ∇J + λθ                        ← modified gradient
  mₜ = β₁mₜ₋₁ + (1-β₁)g̃              ← first moment of modified gradient
  vₜ = β₂vₜ₋₁ + (1-β₂)g̃²             ← second moment of modified gradient
  θ ← θ - α · m̂ₜ/√(v̂ₜ + ε)
```

**Problem**: The regularization gradient λθ is:
1. **Included in mₜ** — averaged with other gradients (diluted)
2. **Included in vₜ** — affects the adaptive learning rate
3. **Divided by √vₜ** — the decay is scaled non-uniformly per parameter

```
  Result: The effective weight decay strength is DIFFERENT for each
  parameter, depending on its gradient history!
  
  Parameter with large gradients: √vₜ is large → decay is weakened
  Parameter with small gradients: √vₜ is small → decay is amplified
  
  This is NOT what we want! We want UNIFORM decay.
```

### Weight Decay in Adam (AdamW)

**Decoupled** weight decay applies the shrinkage **after** the Adam update:

```
  gₜ = ∇J(θₜ)                         ← clean gradient (no λθ)
  mₜ = β₁mₜ₋₁ + (1-β₁)gₜ
  vₜ = β₂vₜ₋₁ + (1-β₂)gₜ²
  θ ← θ - α · m̂ₜ/√(v̂ₜ + ε) - αλθ    ← decay applied SEPARATELY
  
  equivalently:
  θ ← (1-αλ)θ - α · m̂ₜ/√(v̂ₜ + ε)
```

### Side-by-Side Comparison

```
  Adam + L2:                          AdamW (decoupled):
  
  gradient includes λθ                gradient is clean
         ↓                                  ↓
  Adam normalizes the                 Adam normalizes clean gradient
  COMBINED gradient                         ↓
         ↓                            Weight decay applied
  Decay is non-uniformly              UNIFORMLY to all parameters
  scaled by √vₜ                            ↓
         ↓                            Each param decays by
  Some params decay too                same fraction (1-αλ)
  much, others too little
```

### Visualization

```
  Parameter θ₁ (large gradients):    Parameter θ₂ (small gradients):
  
  Adam + L2:                          Adam + L2:
  √v₁ = 10 → decay ≈ λ/10           √v₂ = 0.01 → decay ≈ λ/0.01 = 100λ
  (weak decay)                        (excessive decay!)
  
  AdamW:                              AdamW:
  decay = αλ (uniform)                decay = αλ (same!)
  
  ┌────────────────────────────────────────────────────────────┐
  │  Adam + L2: Decay strength depends on gradient history    │
  │  AdamW:     Decay strength is UNIFORM for all parameters  │
  │                                                           │
  │  This matters in practice! AdamW generalizes better.      │
  └────────────────────────────────────────────────────────────┘
```

---

## 3. Mathematical Proof of Inequivalence

### Setup

Consider a single parameter θ with gradient g and regularization λ.

### Adam + L2

```
  Modified gradient: g̃ = g + λθ
  
  After convergence of moment estimates:
  m̂ ≈ g̃ = g + λθ
  v̂ ≈ g̃² = (g + λθ)²
  
  Update: Δθ = -α · (g + λθ) / √((g + λθ)² + ε)
             ≈ -α · sign(g + λθ)    (for large |g + λθ|)
  
  The decay (λθ part) is normalized by the same √v̂ as the gradient.
```

### AdamW

```
  Clean gradient: g
  
  m̂ ≈ g
  v̂ ≈ g²
  
  Update: Δθ = -α · g/√(g² + ε) - αλθ
             ≈ -α · sign(g) - αλθ
  
  The decay (αλθ) is NOT normalized — it's applied directly.
```

### The Difference

```
  Adam + L2: effective_decay ∝ λθ / √(v̂)    ← varies per parameter
  AdamW:     effective_decay = αλθ            ← uniform
  
  Only when √v̂ = constant for all params are they equivalent.
  Since v̂ varies widely across parameters, they are NOT equivalent.
```

---

## 4. Practical Impact

### Experiment: Adam+L2 vs AdamW

```
  Test Accuracy on CIFAR-10:
  
  Accuracy ↑
  94%      │              ● AdamW (0.01)
  93%      │          ●
  92%      │      ● Adam+L2 (best-tuned)
  91%      │  ●
  90%      │●
           └──────────────────────────→ Weight Decay / L2 Strength
            0     0.001   0.01   0.1
  
  AdamW consistently outperforms Adam+L2 by 0.5-1.5%
  across a range of regularization strengths.
```

| Method | Generalization | Tuning | Recommendation |
|---|---|---|---|
| SGD + L2 | Good | Easy (equivalent to WD) | Standard for CNNs |
| SGD + WD | Good | Easy (equivalent to L2) | Same as above |
| Adam + L2 | Suboptimal | Hard (entangled) | **Avoid** |
| **AdamW** | **Best** | **Easy (decoupled)** | **Use this** |

---

## 5. Choosing Weight Decay Values

### Typical Ranges

| Architecture | Optimizer | Weight Decay | Notes |
|---|---|---|---|
| ResNet-50 (ImageNet) | SGD | 1e-4 | Standard |
| ViT (Vision Transformer) | AdamW | 0.05–0.3 | Much larger! |
| BERT fine-tuning | AdamW | 0.01 | Standard |
| GPT-style training | AdamW | 0.1 | Large models |
| Small models | Any | 1e-5 to 1e-3 | Light regularization |

### Interaction with Learning Rate

```
  Effective decay per step = α × λ
  
  If you increase lr by 2×, you should halve λ to maintain
  the same effective decay rate (or use AdamW where they're decoupled).
  
  With AdamW:
  - lr controls optimization step size
  - weight_decay controls regularization strength
  - They're INDEPENDENT (by design!)
```

### Tuning Strategy

```
  Step 1: Fix weight_decay = 0.01 (default)
  Step 2: Tune learning rate first
  Step 3: Tune weight_decay:
          - If overfitting: increase (0.01 → 0.05 → 0.1)
          - If underfitting: decrease (0.01 → 0.001 → 0)
  Step 4: Fine-tune both together
  
  Grid search space:
  lr:           [1e-4, 3e-4, 1e-3, 3e-3]
  weight_decay: [0, 1e-4, 1e-3, 0.01, 0.1]
```

---

## 6. What NOT to Decay

Not all parameters should have weight decay applied:

```python
# Common pattern: no weight decay on biases and normalization params
def get_param_groups(model, wd=0.01):
    """Separate parameters into decay and no-decay groups."""
    decay_params = []
    no_decay_params = []
    
    for name, param in model.named_parameters():
        if not param.requires_grad:
            continue
        
        # Don't decay biases, LayerNorm/BatchNorm params
        if param.dim() == 1 or 'bias' in name:
            no_decay_params.append(param)
        else:
            decay_params.append(param)
    
    return [
        {'params': decay_params, 'weight_decay': wd},
        {'params': no_decay_params, 'weight_decay': 0.0},
    ]

# Usage
param_groups = get_param_groups(model, wd=0.01)
optimizer = torch.optim.AdamW(param_groups, lr=1e-3)
```

### Why No Decay on These Parameters?

| Parameter | Why No Decay |
|---|---|
| **Biases** | Biases are not multiplied by input → no overfitting risk |
| **LayerNorm γ, β** | Normalization params should be free to scale |
| **Embeddings** | Some practitioners decay, others don't (debated) |
| **Batch norm params** | Already regularized by normalization |

---

## 7. Worked Example

### Problem
Parameter θ = 5.0, gradient ∇J = 2.0, lr α = 0.01, decay λ = 0.1.

### SGD + L2

```
  Modified gradient: ∇J + λθ = 2.0 + 0.1×5.0 = 2.5
  Update: θ = 5.0 - 0.01×2.5 = 4.975
  
  Equivalently (weight decay form):
  θ = (1 - 0.01×0.1)×5.0 - 0.01×2.0
    = 0.999×5.0 - 0.02
    = 4.995 - 0.02
    = 4.975  ✓ same!
```

### Adam + L2 (not recommended)

```
  Modified gradient: g̃ = 2.0 + 0.5 = 2.5
  Suppose v̂ = 10 (accumulated from large past gradients)
  
  Update: θ = 5.0 - 0.01 × 2.5/√(10 + 1e-8)
            = 5.0 - 0.01 × 0.791
            = 5.0 - 0.00791
            = 4.992
  
  Effective decay: (5.0 - 4.992) / 5.0 = 0.16%
  The decay is DILUTED by the large √v̂!
```

### AdamW (recommended)

```
  Clean gradient: g = 2.0
  Suppose v̂ = 10 (from gradient history)
  
  Adam update:  -0.01 × 2.0/√(10 + 1e-8) = -0.00632
  Decay:        -0.01 × 0.1 × 5.0 = -0.005
  
  θ = 5.0 - 0.00632 - 0.005 = 4.989
  
  Effective decay: 0.005/5.0 = 0.1% (exactly αλ = 0.001, as intended!)
```

---

## 8. Python Implementation (PyTorch)

### Adam vs AdamW

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

torch.manual_seed(42)
X = torch.randn(1000, 30)
y = (X[:, :5] @ torch.randn(5, 1)).squeeze() + torch.randn(1000) * 0.3
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

def train(opt_cls, opt_kwargs, epochs=60):
    torch.manual_seed(0)
    model = nn.Sequential(
        nn.Linear(30, 256), nn.ReLU(),
        nn.Linear(256, 128), nn.ReLU(),
        nn.Linear(128, 1)
    )
    optimizer = opt_cls(model.parameters(), **opt_kwargs)
    losses = []
    for epoch in range(epochs):
        total = 0
        for xb, yb in loader:
            loss = nn.MSELoss()(model(xb).squeeze(), yb)
            optimizer.zero_grad(); loss.backward(); optimizer.step()
            total += loss.item() * len(xb)
        losses.append(total / len(X))
    return losses

# Compare
adam_l2 = train(torch.optim.Adam, {"lr": 0.001, "weight_decay": 0.01})
adamw = train(torch.optim.AdamW, {"lr": 0.001, "weight_decay": 0.01})
sgd_wd = train(torch.optim.SGD, {"lr": 0.01, "momentum": 0.9, "weight_decay": 1e-4})

print(f"Adam + L2 (weight_decay): {adam_l2[-1]:.4f}")
print(f"AdamW (decoupled):        {adamw[-1]:.4f}")
print(f"SGD + weight_decay:       {sgd_wd[-1]:.4f}")
```

### Parameter Groups (Selective Weight Decay)

```python
class MyModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.features = nn.Sequential(
            nn.Linear(30, 128), nn.LayerNorm(128), nn.ReLU(),
            nn.Linear(128, 64), nn.LayerNorm(64), nn.ReLU(),
        )
        self.head = nn.Linear(64, 1)
    
    def forward(self, x):
        return self.head(self.features(x))

model = MyModel()

# Separate parameters for weight decay
decay_params = []
no_decay_params = []
for name, param in model.named_parameters():
    if 'weight' in name and param.dim() > 1:
        decay_params.append(param)
    else:
        no_decay_params.append(param)

print(f"Params WITH decay:    {sum(p.numel() for p in decay_params):,}")
print(f"Params WITHOUT decay: {sum(p.numel() for p in no_decay_params):,}")

optimizer = torch.optim.AdamW([
    {'params': decay_params, 'weight_decay': 0.01},
    {'params': no_decay_params, 'weight_decay': 0.0},
], lr=1e-3)
```

### Manual AdamW vs Adam+L2 Verification

```python
def verify_difference():
    """Show that Adam+L2 and AdamW produce different results."""
    torch.manual_seed(42)
    model_a = nn.Linear(10, 5)
    model_b = nn.Linear(10, 5)
    model_b.load_state_dict(model_a.state_dict())
    
    opt_a = torch.optim.Adam(model_a.parameters(), lr=0.01, weight_decay=0.1)
    opt_b = torch.optim.AdamW(model_b.parameters(), lr=0.01, weight_decay=0.1)
    
    x = torch.randn(16, 10)
    
    for step in range(20):
        loss_a = model_a(x).sum()
        opt_a.zero_grad(); loss_a.backward(); opt_a.step()
        
        loss_b = model_b(x).sum()
        opt_b.zero_grad(); loss_b.backward(); opt_b.step()
    
    diff = (model_a.weight.data - model_b.weight.data).abs()
    print(f"After 20 steps:")
    print(f"  Adam+L2 weight norm:  {model_a.weight.data.norm():.4f}")
    print(f"  AdamW weight norm:    {model_b.weight.data.norm():.4f}")
    print(f"  Max weight difference: {diff.max():.4f}")
    print(f"  Mean weight difference: {diff.mean():.4f}")

verify_difference()
# Output shows they are NOT the same!
```

---

## 9. Summary of Key Points

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  For SGD:   weight_decay = L2 regularization (equivalent)   │
  │                                                              │
  │  For Adam:  weight_decay ≠ L2 regularization (different!)    │
  │             → Use AdamW for proper weight decay              │
  │             → torch.optim.Adam(weight_decay=...) is L2      │
  │             → torch.optim.AdamW(weight_decay=...) is WD     │
  │                                                              │
  │  Rule of thumb:                                              │
  │  - If using SGD: weight_decay = L2, both fine                │
  │  - If using Adam: ALWAYS use AdamW instead                   │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Idea |
|---|---|
| L2 regularization | Penalty in loss: J + λ/2 · \|\|w\|\|² |
| Weight decay | Direct shrinkage: θ ← (1-αλ)θ |
| SGD equivalence | L2 ≡ weight decay for SGD (same update) |
| Adam inequivalence | L2 ≠ weight decay for Adam (different!) |
| Adam + L2 problem | Decay is scaled by 1/√vₜ → non-uniform |
| AdamW | Decoupled: decay applied after Adam update |
| What to decay | Weight matrices only (not biases, norms) |
| Typical values | 1e-4 (SGD), 0.01 (AdamW), 0.05-0.3 (ViT) |
| PyTorch | `Adam(weight_decay=)` = L2, `AdamW(weight_decay=)` = WD |

---

## Revision Questions

1. **Prove that L2 regularization and weight decay are equivalent for SGD** by expanding both update rules and showing they produce the same result.

2. **Explain why L2 and weight decay are NOT equivalent for Adam.** What role does the second moment vₜ play in making them different?

3. **In the worked example, why does Adam+L2 produce weaker effective decay** than AdamW? Trace through the computation.

4. **Why should biases and normalization parameters NOT have weight decay?** What would happen if you applied decay to LayerNorm's γ?

5. **Design a hyperparameter search** for weight decay in AdamW. What range would you search, and how would you evaluate each setting?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Layer Normalization](./06-layer-normalization.md) | [Unit 7: Regularization](./README.md) | [Unit 8: Practical Training →](../08-Practical-Training/) |
