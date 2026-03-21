# 2. Dropout

> **Unit 7 · Regularization Techniques** — Randomly silencing neurons to prevent co-adaptation

---

## Chapter Overview

**Dropout** (Srivastava et al., 2014) is one of the most effective regularization techniques in deep learning. During training, each neuron is **randomly zeroed out** (dropped) with probability p at every forward pass. This prevents neurons from developing excessive dependencies on each other (**co-adaptation**) and can be interpreted as training an **exponential ensemble** of sub-networks. At inference, all neurons are active but their outputs are scaled to match the expected training-time output.

---

## 1. The Co-Adaptation Problem

### What is Co-Adaptation?

```
  Without Dropout:
  
  Neuron A ──→ ┐
               ├──→ Neuron D ──→ output
  Neuron B ──→ ┘
  
  Neuron C ──→ (learned to be ignored)
  
  Problem: A and B become a "team" — they rely on each other.
  If either is perturbed at test time (new data), the whole
  prediction breaks. Neuron C is wasted capacity.
  
  
  With Dropout:
  
  Sometimes A is dropped:    Neuron B ──→ Neuron D (B must work alone)
  Sometimes B is dropped:    Neuron A ──→ Neuron D (A must work alone)
  Sometimes C gets a chance:  Neuron C ──→ Neuron D (C learns too!)
  
  Result: Each neuron learns a robust, independent feature.
  No neuron can "depend" on any specific other neuron.
```

---

## 2. Dropout Algorithm

### Training Phase

```
For each training step:
    For each layer with dropout:
        1. Generate random mask: mᵢ ~ Bernoulli(1-p)  for each neuron i
           (mᵢ = 1 with probability 1-p, mᵢ = 0 with probability p)
        
        2. Apply mask: â = a ⊙ m   (element-wise multiply)
        
        3. Use â as input to next layer
```

### Visualization

```
  Full Network:                   With Dropout (p=0.5):
  
  ● ─── ● ─── ● ─── ●           ● ─── ● ─── ● ─── ●
  │ ╲ ╱ │ ╲ ╱ │ ╲ ╱ │           │     │ ╲     ╲ ╱ │
  ● ─── ● ─── ● ─── ●           ○     ●     ○     ●
  │ ╲ ╱ │ ╲ ╱ │ ╲ ╱ │               ╱ │       │
  ● ─── ● ─── ● ─── ●           ●     ○ ─── ● ─── ○
  
  All neurons active              ○ = dropped (output zeroed)
  4,096 neurons                   ~2,048 neurons active
```

### A New Sub-Network Every Step

```
  Step 1: ● ○ ● ○ ● ● ○ ●    Sub-network A
  Step 2: ○ ● ● ● ○ ● ● ○    Sub-network B
  Step 3: ● ● ○ ○ ● ○ ● ●    Sub-network C
  ...
  
  For n neurons with dropout p=0.5:
  Number of possible sub-networks = 2ⁿ
  
  n=1000: 2¹⁰⁰⁰ sub-networks (more than atoms in universe!)
```

---

## 3. Training vs Inference

### The Scaling Problem

During training with dropout rate p:
- Each neuron's output is zeroed with probability p
- Expected output: E[â_i] = (1-p) · a_i

At inference (all neurons active):
- Each neuron outputs a_i (no masking)
- This is **larger** than training by factor 1/(1-p)

### Two Solutions

**Option 1: Scale at Test Time**
```
  Training: â = a ⊙ mask          (mask has p fraction of zeros)
  Inference: â = (1-p) · a        (multiply output by keep probability)
```

**Option 2: Inverted Dropout (Standard)**
```
  Training: â = (a ⊙ mask) / (1-p)   (scale UP during training)
  Inference: â = a                     (no change needed!)
```

### Why Inverted Dropout is Preferred

```
  ┌────────────────────────────────────────────────────┐
  │  Inverted Dropout:                                 │
  │  • Training: divide by (1-p) to compensate         │
  │  • Inference: use weights directly (no modification)│
  │                                                    │
  │  Advantages:                                       │
  │  ✓ Inference code is simpler (no scaling needed)   │
  │  ✓ Model can be exported without dropout logic     │
  │  ✓ PyTorch default behavior                        │
  └────────────────────────────────────────────────────┘
```

### Mathematical Verification

```
  With inverted dropout during training:
  
  E[â_i] = E[(a_i · m_i) / (1-p)]
         = a_i · E[m_i] / (1-p)
         = a_i · (1-p) / (1-p)
         = a_i                    ← matches inference output!
```

---

## 4. Ensemble Interpretation

### Implicit Ensemble of Sub-Networks

```
  Dropout is approximately equivalent to:
  
  1. Training 2ⁿ different sub-networks
     (each with a different dropout pattern)
  
  2. Averaging all their predictions at test time
     (weight sharing makes this tractable)
  
  ┌─────────────────────────────────────┐
  │  Ensemble of M models:             │
  │  ŷ = (1/M) Σₘ fₘ(x)              │
  │                                     │
  │  Dropout (approximately):           │
  │  ŷ = (1/2ⁿ) Σ_masks f_mask(x)     │
  │    ≈ f_all(x) with scaled weights  │
  └─────────────────────────────────────┘
```

This ensemble interpretation explains why dropout **reduces variance** in predictions and improves generalization.

---

## 5. Typical Dropout Rates

| Layer Type | Typical p (drop rate) | Notes |
|---|---|---|
| **Input layer** | 0.0–0.2 | Light or no dropout |
| **Hidden layers** | 0.2–0.5 | Standard range |
| **Large hidden layers** | 0.5 | Classic Hinton recommendation |
| **Before final layer** | 0.3–0.5 | Common in classifiers |
| **Conv layers (standard)** | 0.0–0.2 | Less dropout for convs |
| **Conv layers (spatial)** | 0.1–0.3 | Use spatial dropout |
| **After attention** | 0.1 | Transformers |

### Guidelines

```
  ┌──────────────────────────────────────────────────┐
  │  More dropout (p → 0.5):                        │
  │  • Model is very large relative to data          │
  │  • Seeing significant overfitting                │
  │  • Fully connected layers                        │
  │                                                  │
  │  Less dropout (p → 0.0):                        │
  │  • Model is small or appropriately sized         │
  │  • Already using other regularization            │
  │  • Convolutional layers (spatial correlations)   │
  │  • Batch normalization present                   │
  └──────────────────────────────────────────────────┘
```

---

## 6. Spatial Dropout for CNNs

### Problem with Standard Dropout in CNNs

```
  Feature map (6×6, 1 channel):
  
  Standard dropout (p=0.3):
  ┌─────────────┐     ┌─────────────┐
  │ 2 1 3 0 1 4 │     │ 2 0 3 0 1 4 │
  │ 0 2 1 3 2 1 │     │ 0 2 0 3 2 0 │  ← individual pixels dropped
  │ 3 1 0 2 4 2 │ →   │ 3 1 0 0 4 2 │     (spatial correlations intact)
  │ 1 4 2 1 0 3 │     │ 1 4 2 1 0 3 │
  │ 2 0 3 4 1 2 │     │ 0 0 3 4 0 2 │
  │ 1 3 2 0 2 1 │     │ 1 3 2 0 2 1 │
  └─────────────┘     └─────────────┘
  
  Problem: neighboring pixels are highly correlated,
  so dropping individual pixels has little effect.
```

### Spatial Dropout

Drop entire **feature channels** instead:

```
  Before spatial dropout:          After spatial dropout (p=0.3):
  
  Channel 1: [activated]           Channel 1: [activated]  ✓
  Channel 2: [activated]           Channel 2: [ALL ZEROS]  ✗ dropped
  Channel 3: [activated]           Channel 3: [activated]  ✓
  Channel 4: [activated]           Channel 4: [ALL ZEROS]  ✗ dropped
  Channel 5: [activated]           Channel 5: [activated]  ✓
```

```python
# PyTorch spatial dropout for Conv2d
# Drop entire channels (dimension 1)
dropout_2d = nn.Dropout2d(p=0.2)  # drops entire feature maps
```

---

## 7. Worked Example

### Problem

Layer with 4 neurons, outputs a = [2.0, 0.5, 1.5, 3.0], dropout rate p = 0.5.

### Training (Inverted Dropout)

```
  Step 1: Generate mask (Bernoulli with prob 1-p = 0.5)
  mask = [1, 0, 1, 0]  (randomly generated)
  
  Step 2: Apply mask and scale by 1/(1-p) = 2
  â = [2.0, 0.5, 1.5, 3.0] ⊙ [1, 0, 1, 0] / 0.5
    = [2.0, 0.0, 1.5, 0.0] / 0.5
    = [4.0, 0.0, 3.0, 0.0]
  
  Note: Active neurons are DOUBLED to compensate for dropped ones.
```

### Inference (No Dropout)

```
  â = [2.0, 0.5, 1.5, 3.0]  (used directly, no mask)
  
  Expected training output:
  E[â] = [2.0·1·2, 0.5·0.5·2, 1.5·1·2, 3.0·0.5·2]
       = [4.0·0.5, 0.5·1, 3.0·0.5, 3.0·1]  ... actually:
  E[â_i] = a_i (by design of inverted dropout) ✓
```

---

## 8. Python Implementation (PyTorch)

### Using PyTorch's Dropout

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

# ── Data (small — to demonstrate overfitting) ──────────────────
torch.manual_seed(42)
X = torch.randn(300, 50)
y = (X[:, :5] @ torch.randn(5, 1)).squeeze() + torch.randn(300) * 0.1
X_val = torch.randn(200, 50)
y_val = (X_val[:, :5] @ torch.randn(5, 1)).squeeze()

train_loader = DataLoader(TensorDataset(X, y), batch_size=32, shuffle=True)

# ── Model WITH Dropout ─────────────────────────────────────────
class ModelWithDropout(nn.Module):
    def __init__(self, p=0.5):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(50, 256), nn.ReLU(), nn.Dropout(p),
            nn.Linear(256, 128), nn.ReLU(), nn.Dropout(p),
            nn.Linear(128, 64), nn.ReLU(), nn.Dropout(p),
            nn.Linear(64, 1)
        )
    
    def forward(self, x):
        return self.net(x).squeeze()

# ── Model WITHOUT Dropout ──────────────────────────────────────
class ModelNoDropout(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(50, 256), nn.ReLU(),
            nn.Linear(256, 128), nn.ReLU(),
            nn.Linear(128, 64), nn.ReLU(),
            nn.Linear(64, 1)
        )
    
    def forward(self, x):
        return self.net(x).squeeze()

# ── Training function ──────────────────────────────────────────
def train_model(model, epochs=100):
    optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
    criterion = nn.MSELoss()
    
    train_hist, val_hist = [], []
    for epoch in range(epochs):
        # Training
        model.train()  # ← enables dropout
        total = 0
        for xb, yb in train_loader:
            loss = criterion(model(xb), yb)
            optimizer.zero_grad(); loss.backward(); optimizer.step()
            total += loss.item() * len(xb)
        train_hist.append(total / len(X))
        
        # Validation
        model.eval()   # ← disables dropout!
        with torch.no_grad():
            val_loss = criterion(model(X_val), y_val).item()
        val_hist.append(val_loss)
    
    return train_hist, val_hist

# ── Compare ────────────────────────────────────────────────────
torch.manual_seed(0)
no_drop_train, no_drop_val = train_model(ModelNoDropout())
torch.manual_seed(0)
drop_train, drop_val = train_model(ModelWithDropout(p=0.5))

print(f"No Dropout - Train: {no_drop_train[-1]:.4f}, Val: {no_drop_val[-1]:.4f}")
print(f"Dropout    - Train: {drop_train[-1]:.4f}, Val: {drop_val[-1]:.4f}")
```

### Manual Dropout Implementation

```python
class ManualDropout(nn.Module):
    """Inverted dropout implemented from scratch."""
    
    def __init__(self, p=0.5):
        super().__init__()
        self.p = p
    
    def forward(self, x):
        if not self.training or self.p == 0:
            return x
        
        # Generate binary mask
        mask = (torch.rand_like(x) > self.p).float()
        
        # Apply mask and scale (inverted dropout)
        return x * mask / (1 - self.p)

# Verify behavior
manual_drop = ManualDropout(p=0.5)

x = torch.ones(1, 10) * 2.0  # all values = 2.0

manual_drop.train()
print("Training mode (several forward passes):")
for i in range(5):
    out = manual_drop(x)
    print(f"  Pass {i}: {out.numpy().flatten()}")
    # Some zeros, others = 4.0 (2.0 / 0.5)

manual_drop.eval()
print(f"\nEval mode: {manual_drop(x).numpy().flatten()}")
# All values = 2.0 (no dropout)
```

### Dropout with Different Rates per Layer

```python
class FlexibleDropoutModel(nn.Module):
    """Different dropout rates for different layers."""
    
    def __init__(self):
        super().__init__()
        self.layers = nn.Sequential(
            nn.Linear(50, 512), nn.ReLU(),
            nn.Dropout(0.2),    # light dropout early
            nn.Linear(512, 256), nn.ReLU(),
            nn.Dropout(0.3),    # moderate
            nn.Linear(256, 128), nn.ReLU(),
            nn.Dropout(0.5),    # heavy dropout late (more parameters)
            nn.Linear(128, 10)  # no dropout on output
        )
    
    def forward(self, x):
        return self.layers(x)
```

---

## 9. Important: model.train() vs model.eval()

```python
# ⚠️ CRITICAL: Dropout behavior depends on mode!

model.train()   # Dropout is ACTIVE
                 # Must use during training

model.eval()    # Dropout is DISABLED
                 # Must use during inference/validation

# Common mistake:
# Forgetting model.eval() before inference → predictions are noisy and wrong!
# Forgetting model.train() before training → no regularization benefit!
```

---

## Summary Table

| Concept | Key Idea |
|---|---|
| Dropout rate p | Probability of zeroing each neuron (0.2–0.5 typical) |
| Training | Randomly zero neurons, scale by 1/(1-p) (inverted) |
| Inference | All neurons active, no scaling needed |
| Co-adaptation | Dropout prevents neurons from relying on specific partners |
| Ensemble view | Trains exponentially many sub-networks simultaneously |
| Spatial dropout | Drop entire feature channels for CNNs |
| PyTorch | `nn.Dropout(p)`, `nn.Dropout2d(p)` |
| Critical | `model.train()` for training, `model.eval()` for inference |

---

## Revision Questions

1. **Explain the co-adaptation problem** and how dropout prevents it. Give a concrete example with 3 neurons.

2. **Derive the expected output of inverted dropout** and show it equals the neuron's activation a_i.

3. **Why is inverted dropout preferred** over scaling at test time? What practical advantage does it provide?

4. **How many distinct sub-networks** can dropout produce in a layer with 1000 neurons and p=0.5? How does this relate to ensembling?

5. **Why is spatial dropout more appropriate for CNNs** than standard dropout? What would happen if you applied standard dropout to a 7×7 feature map?

6. **What happens if you forget `model.eval()` at inference time?** How would predictions be affected?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← L1/L2 Regularization](./01-l1-l2-regularization.md) | [Unit 7: Regularization](./README.md) | [Early Stopping →](./03-early-stopping.md) |
