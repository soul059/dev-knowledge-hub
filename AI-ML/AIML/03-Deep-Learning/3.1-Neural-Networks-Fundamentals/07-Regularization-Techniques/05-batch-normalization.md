# 5. Batch Normalization

> **Unit 7 · Regularization Techniques** — Normalizing layer inputs for faster, more stable training

---

## Chapter Overview

**Batch Normalization** (Ioffe & Szegedy, 2015) is one of the most transformative innovations in deep learning. It normalizes the inputs to each layer using mini-batch statistics (mean and variance), then applies learnable scale and shift parameters. Originally motivated by reducing "internal covariate shift," recent research suggests its true benefit lies in **smoothing the loss landscape**. BatchNorm enables higher learning rates, reduces initialization sensitivity, and provides mild regularization.

---

## 1. The Problem: Internal Covariate Shift

### Original Motivation

Each layer receives inputs from the previous layer, whose distribution **shifts** as weights update:

```
  Epoch 1: Layer 2 receives inputs with mean=0.5, std=1.2
  Epoch 2: Layer 2 receives inputs with mean=0.8, std=0.9  ← shifted!
  Epoch 3: Layer 2 receives inputs with mean=-0.2, std=1.5  ← shifted again!
  
  Layer 2 must constantly re-adapt to a moving input distribution.
  This is "internal covariate shift."
```

### Modern Understanding

Recent work (Santurkar et al., 2018) suggests BatchNorm's benefit comes more from **smoothing the loss landscape** than reducing covariate shift:

```
  Without BatchNorm:                 With BatchNorm:
  
  Loss ↑                             Loss ↑
       │╱╲╱╲                              │╲
       │    ╲╱╲╱╲                          │ ╲
       │         ╲╱╲╱╲                     │  ╲╲
       │              ╲                    │    ╲╲
       │                                   │      ╲╲──
       └──────────────→ θ                  └──────────────→ θ
       
  Rough landscape                    Smooth landscape
  → small lr needed                  → large lr works!
  → optimization is hard             → optimization is easy
```

---

## 2. The BatchNorm Algorithm

### Forward Pass (Training)

For a mini-batch B = {x₁, ..., x_B}:

```
Step 1: Compute mini-batch mean
        μ_B = (1/B) Σᵢ xᵢ

Step 2: Compute mini-batch variance
        σ²_B = (1/B) Σᵢ (xᵢ - μ_B)²

Step 3: Normalize
        x̂ᵢ = (xᵢ - μ_B) / √(σ²_B + ε)

Step 4: Scale and shift (learnable parameters)
        yᵢ = γ · x̂ᵢ + β
```

### The Complete Formula

```
  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │  y = γ · (x - μ_B) / √(σ²_B + ε) + β               │
  │      ↑         ↑              ↑         ↑            │
  │    scale    center       normalize    shift           │
  │  (learnable)                        (learnable)       │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

### Parameters

| Parameter | Learnable? | Initial Value | Role |
|---|---|---|---|
| **γ** (gamma) | Yes | 1 | Scale (can undo normalization) |
| **β** (beta) | Yes | 0 | Shift (replaces bias in previous layer) |
| **μ_running** | No (statistics) | 0 | Running mean for inference |
| **σ²_running** | No (statistics) | 1 | Running variance for inference |

---

## 3. Per-Feature Normalization

### For Fully Connected Layers (BatchNorm1d)

Each feature is normalized independently across the batch:

```
  Batch of B samples, D features:
  
  Input x ∈ ℝᴮˣᴰ:
          Feature 1   Feature 2   Feature 3
  Sample 1:  [2.1,       -0.3,       1.5]
  Sample 2:  [1.8,        0.5,       0.9]
  Sample 3:  [2.4,       -0.1,       1.2]
  Sample 4:  [1.9,        0.2,       1.8]
              ↓            ↓          ↓
  μ:        [2.05,       0.075,     1.35]    ← mean per feature
  σ²:       [0.0525,     0.0919,    0.1125]  ← var per feature
  
  Normalize EACH COLUMN independently, then scale/shift
```

### For Convolutional Layers (BatchNorm2d)

Each **channel** is normalized across batch and spatial dimensions:

```
  Input: (B, C, H, W)
  
  For each channel c:
    μ_c = mean over all B samples and all H×W spatial positions
    σ²_c = variance over all B samples and all H×W spatial positions
  
  γ, β have shape (C,) — one per channel
```

```
  Batch of B images, C channels:
  
  Channel 1:  [img1_ch1, img2_ch1, img3_ch1, ...]
               Each is H×W pixels
               μ₁ = mean of ALL pixels in channel 1 across ALL images
               
  Channel 2:  [img1_ch2, img2_ch2, img3_ch2, ...]
               μ₂ = mean of ALL pixels in channel 2 across ALL images
```

---

## 4. Training vs Inference

### The Problem at Inference

During inference, we may have **a single sample** (batch size = 1), so mini-batch statistics are meaningless. We need **fixed** statistics.

### Solution: Running Statistics

During training, maintain exponential moving averages:

```
  μ_running = (1-momentum) · μ_running + momentum · μ_B
  σ²_running = (1-momentum) · σ²_running + momentum · σ²_B
  
  Default momentum = 0.1 in PyTorch
```

At inference:

```
  Training:    y = γ · (x - μ_B) / √(σ²_B + ε) + β     ← mini-batch stats
  Inference:   y = γ · (x - μ_running) / √(σ²_running + ε) + β  ← running stats
```

### Critical: model.train() vs model.eval()

```python
# ⚠️ CRITICAL: BatchNorm behaves differently in train vs eval mode!

model.train()   # Uses mini-batch statistics, updates running stats
model.eval()    # Uses running (fixed) statistics
```

---

## 5. Benefits of BatchNorm

### 1. Higher Learning Rates

```
  Without BN: lr > 0.01 → divergence
  With BN:    lr up to 0.1 works fine
  
  Result: 3-10× faster training!
```

### 2. Reduced Initialization Sensitivity

```
  Without BN:
  Init σ = 0.001 → vanishing signals
  Init σ = 1.0   → exploding signals
  
  With BN:
  Init σ = 0.001 → BN normalizes → signals OK
  Init σ = 1.0   → BN normalizes → signals OK
  
  BatchNorm makes the network robust to initialization!
```

### 3. Mild Regularization

```
  Mini-batch stats add NOISE:
  - μ_B and σ²_B are noisy estimates of true μ and σ²
  - This noise acts similarly to dropout
  - Larger batches → less noise → less regularization
  
  That's why larger batch sizes sometimes need more explicit regularization.
```

### 4. Smoother Loss Landscape

The Lipschitz continuity of the loss function is improved:

```
  Without BN: ||∇L(θ₁) - ∇L(θ₂)|| ≤ K₁ · ||θ₁ - θ₂||   (K₁ large)
  With BN:    ||∇L(θ₁) - ∇L(θ₂)|| ≤ K₂ · ||θ₁ - θ₂||   (K₂ << K₁)
  
  Smoother gradients → optimizer can take larger steps confidently
```

---

## 6. Placement: Before or After Activation?

### Two Schools of Thought

```
  Original paper (Ioffe & Szegedy):     Common practice:
  
  x → Linear → BN → ReLU → next       x → Linear → ReLU → BN → next
  
  Rationale: normalize BEFORE            Rationale: some practitioners
  the activation function                found this works equally well
                                         or better in practice
```

### Practical Recommendation

Both work well. The original placement (BN before activation) is more common:

```python
# Standard placement: Linear → BN → Activation
nn.Sequential(
    nn.Linear(256, 128),
    nn.BatchNorm1d(128),    # normalize before activation
    nn.ReLU(),
)

# Alternative: Linear → Activation → BN
nn.Sequential(
    nn.Linear(256, 128),
    nn.ReLU(),
    nn.BatchNorm1d(128),    # normalize after activation
)
```

**Note**: When using BN, the **bias in the preceding Linear/Conv layer is redundant** (BN's β replaces it):

```python
# Efficient: disable bias in Linear before BN
nn.Sequential(
    nn.Linear(256, 128, bias=False),  # bias=False (BN has its own shift)
    nn.BatchNorm1d(128),
    nn.ReLU(),
)
```

---

## 7. Worked Example

### Problem

Mini-batch of 4 samples, 3 features, γ = [1, 1, 1], β = [0, 0, 0], ε = 1e-5.

```
  Input:
  x = [[2.0,  -1.0,  3.0],
       [4.0,   1.0,  1.0],
       [6.0,  -1.0,  5.0],
       [4.0,   3.0,  3.0]]
```

### Solution

**Step 1: Mean per feature**
```
  μ = [4.0, 0.5, 3.0]
```

**Step 2: Variance per feature**
```
  σ² = [(4+0+4+0)/4, (2.25+0.25+2.25+6.25)/4, (0+4+4+0)/4]
     = [2.0, 2.75, 2.0]
```

**Step 3: Normalize**
```
  x̂₁ = (2-4)/√(2+ε) = -1.414
  x̂₅ = (1-0.5)/√(2.75+ε) = 0.302
  ... (each element independently)
```

**Step 4: Scale and shift (γ=1, β=0 → no change)**
```
  y = 1 · x̂ + 0 = x̂
```

Output has mean ≈ 0 and variance ≈ 1 per feature column. ✓

---

## 8. Python Implementation (PyTorch)

### Using PyTorch's BatchNorm

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

# ── BatchNorm1d (for fully connected layers) ───────────────────
bn = nn.BatchNorm1d(num_features=3)
print(f"γ (weight): {bn.weight.data}")   # tensor([1., 1., 1.])
print(f"β (bias):   {bn.bias.data}")     # tensor([0., 0., 0.])
print(f"running_mean: {bn.running_mean}") # tensor([0., 0., 0.])
print(f"running_var:  {bn.running_var}")  # tensor([1., 1., 1.])

# Forward pass
x = torch.tensor([[2., -1., 3.],
                   [4.,  1., 1.],
                   [6., -1., 5.],
                   [4.,  3., 3.]])
bn.train()
y = bn(x)
print(f"\nNormalized output:\n{y}")
print(f"Column means: {y.mean(dim=0)}")  # ≈ [0, 0, 0]
print(f"Column stds:  {y.std(dim=0)}")   # ≈ [1, 1, 1] (with Bessel correction)

# ── BatchNorm2d (for conv layers) ──────────────────────────────
bn2d = nn.BatchNorm2d(num_features=64)  # 64 channels
# Input shape: (B, 64, H, W) → normalizes per channel
```

### Full Model with BatchNorm

```python
class ConvNetWithBN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(3, 64, 3, padding=1, bias=False),
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.MaxPool2d(2),
            
            nn.Conv2d(64, 128, 3, padding=1, bias=False),
            nn.BatchNorm2d(128),
            nn.ReLU(),
            nn.MaxPool2d(2),
            
            nn.Conv2d(128, 256, 3, padding=1, bias=False),
            nn.BatchNorm2d(256),
            nn.ReLU(),
            nn.AdaptiveAvgPool2d(1),
        )
        self.classifier = nn.Linear(256, num_classes)
    
    def forward(self, x):
        x = self.features(x)
        x = x.view(x.size(0), -1)
        return self.classifier(x)

model = ConvNetWithBN()
print(f"Parameters: {sum(p.numel() for p in model.parameters()):,}")
```

### Comparing Training With and Without BN

```python
import matplotlib.pyplot as plt

torch.manual_seed(42)
X = torch.randn(3000, 50)
y = (X[:, :10] @ torch.randn(10, 1)).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

def train(use_bn, lr, epochs=50):
    torch.manual_seed(0)
    if use_bn:
        model = nn.Sequential(
            nn.Linear(50, 256, bias=False), nn.BatchNorm1d(256), nn.ReLU(),
            nn.Linear(256, 128, bias=False), nn.BatchNorm1d(128), nn.ReLU(),
            nn.Linear(128, 64, bias=False), nn.BatchNorm1d(64), nn.ReLU(),
            nn.Linear(64, 1)
        )
    else:
        model = nn.Sequential(
            nn.Linear(50, 256), nn.ReLU(),
            nn.Linear(256, 128), nn.ReLU(),
            nn.Linear(128, 64), nn.ReLU(),
            nn.Linear(64, 1)
        )
    
    opt = torch.optim.SGD(model.parameters(), lr=lr, momentum=0.9)
    losses = []
    for epoch in range(epochs):
        model.train()
        total = 0
        for xb, yb in loader:
            loss = nn.MSELoss()(model(xb).squeeze(), yb)
            if torch.isnan(loss): return [float('nan')] * epochs
            opt.zero_grad(); loss.backward(); opt.step()
            total += loss.item() * len(xb)
        losses.append(total / len(X))
    return losses

# Without BN: needs small lr
no_bn_loss = train(use_bn=False, lr=0.01)

# With BN: can use larger lr
bn_loss = train(use_bn=True, lr=0.1)  # 10× larger lr!

print(f"Without BN (lr=0.01): {no_bn_loss[-1]:.4f}")
print(f"With BN (lr=0.1):     {bn_loss[-1]:.4f}")
```

---

## 9. BatchNorm Gotchas

### Small Batch Sizes

```
  Batch size = 2: μ_B and σ²_B are very noisy
  → BN performance degrades
  → Use GroupNorm or LayerNorm instead (see next chapter)
```

### Batch Size Changes Between Training and Inference

```
  ┌─────────────────────────────────────────────────────┐
  │  Training (batch=32): good batch statistics         │
  │  Inference (batch=1): uses running statistics       │
  │                                                     │
  │  ⚠️ If running statistics are wrong (e.g., model   │
  │  was never properly trained), inference fails!      │
  │                                                     │
  │  Fix: After training, do a pass through training    │
  │  data with model.eval() to verify running stats.    │
  └─────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Idea |
|---|---|
| Algorithm | Normalize → Scale → Shift per mini-batch |
| Formula | y = γ · (x - μ_B)/√(σ²_B + ε) + β |
| Learnable params | γ (scale, init=1), β (shift, init=0) |
| Training | Uses mini-batch μ and σ² |
| Inference | Uses running (exponential) μ and σ² |
| BN1d | Per feature (for FC layers) |
| BN2d | Per channel (for conv layers) |
| Benefits | Higher lr, less init sensitivity, mild regularization |
| True benefit | Smoother loss landscape |
| Placement | Before activation (standard) or after |
| PyTorch | `nn.BatchNorm1d(features)`, `nn.BatchNorm2d(channels)` |

---

## Revision Questions

1. **Walk through the BatchNorm forward pass** for a mini-batch of 4 samples with 2 features. Show all intermediate values.

2. **Why does BatchNorm allow higher learning rates?** Explain in terms of the loss landscape smoothness.

3. **Explain the difference between training and inference mode** in BatchNorm. What happens if you forget `model.eval()`?

4. **Why is the bias in the preceding layer redundant** when using BatchNorm? Show mathematically.

5. **How does BatchNorm provide mild regularization?** Why does this effect decrease with larger batch sizes?

6. **When does BatchNorm perform poorly?** Describe two scenarios and suggest alternatives.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Data Augmentation](./04-data-augmentation.md) | [Unit 7: Regularization](./README.md) | [Layer Normalization →](./06-layer-normalization.md) |
