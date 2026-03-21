# 2. Zero Initialization Problem

> **Unit 6 · Weight Initialization** — Why setting all weights to zero kills learning

---

## Chapter Overview

Zero initialization is the most **intuitive** but most **devastating** choice for neural network weights. When all weights in a layer are identical (especially zero), every neuron computes the exact same output, receives the exact same gradient, and updates identically. This **symmetry** is never broken during training, effectively reducing a layer with N neurons to a single neuron. This chapter provides a rigorous mathematical proof and demonstrates why biases can be zero while weights cannot.

---

## 1. The Symmetry Problem

### Intuition

```
  Layer with 3 neurons, all weights identical:
  
  x₁ ─────┐
           ├──→ [w₁₁, w₁₂] ──→ z₁ = w·x₁ + w·x₂ ──→ a₁
  x₂ ─────┤
           ├──→ [w₂₁, w₂₂] ──→ z₂ = w·x₁ + w·x₂ ──→ a₂  ← SAME as a₁!
           │
           └──→ [w₃₁, w₃₂] ──→ z₃ = w·x₁ + w·x₂ ──→ a₃  ← SAME as a₁!
  
  If w₁₁ = w₂₁ = w₃₁ and w₁₂ = w₂₂ = w₃₂:
  → z₁ = z₂ = z₃
  → a₁ = a₂ = a₃
  → All neurons are IDENTICAL copies
```

### The Vicious Cycle

```
  ┌─────────────────────────────────────────────────┐
  │  1. All weights identical (by initialization)   │
  │              ↓                                   │
  │  2. All neurons compute same output              │
  │              ↓                                   │
  │  3. All neurons receive same gradient from       │
  │     downstream layer                              │
  │              ↓                                   │
  │  4. All weight updates are identical              │
  │              ↓                                   │
  │  5. All weights remain identical                  │
  │              ↓                                   │
  │  → Back to step 2 (cycle repeats forever!)       │
  └─────────────────────────────────────────────────┘
```

---

## 2. Mathematical Proof of Symmetry Preservation

### Setup

Consider a two-layer network:

```
  Input: x ∈ ℝⁿ
  Hidden: z = W⁽¹⁾x + b⁽¹⁾,  a = σ(z)
  Output: ŷ = W⁽²⁾a + b⁽²⁾
  Loss: L(ŷ, y)
```

### Zero Initialization

Set W⁽¹⁾ = 0 (all zeros), b⁽¹⁾ = 0, W⁽²⁾ = 0, b⁽²⁾ = 0.

### Forward Pass

```
  z = W⁽¹⁾x + b⁽¹⁾ = 0·x + 0 = 0   (for ALL neurons)
  
  a = σ(0) = {
    0.5  if σ = sigmoid
    0    if σ = tanh
    0    if σ = ReLU
  }
  
  All hidden activations are IDENTICAL.
```

### Backward Pass

For the hidden layer:
```
  δ⁽¹⁾ = (W⁽²⁾)ᵀ · δ⁽²⁾ ⊙ σ'(z)
```

Since all z values are equal, σ'(z) is the same for all neurons. Since W⁽²⁾ is all zeros (or identical), every neuron in the hidden layer receives the **same** error signal:

```
  δ₁⁽¹⁾ = δ₂⁽¹⁾ = δ₃⁽¹⁾ = ... = δₕ⁽¹⁾
```

### Weight Update

```
  ΔW⁽¹⁾ᵢⱼ = -α · δᵢ⁽¹⁾ · xⱼ
```

Since all δᵢ are equal:

```
  ΔW⁽¹⁾₁ⱼ = ΔW⁽¹⁾₂ⱼ = ... = ΔW⁽¹⁾ₕⱼ  for all j
```

**All neurons get the same update → weights remain identical after update.**

### Induction

By induction on the number of gradient steps:
- **Base case**: All weights identical at t=0 (by initialization)
- **Inductive step**: If weights are identical at step t, they remain identical at step t+1

Therefore, **symmetry is preserved forever**.

---

## 3. The Zero Initialization Specifically

### With ReLU Activation

```
  z = 0·x + 0 = 0
  a = ReLU(0) = 0
  
  Output is 0. Gradient of ReLU at 0:
  ReLU'(0) is undefined (typically set to 0)
  
  → Gradient is 0 → no update → DEAD NETWORK
```

### With Sigmoid

```
  z = 0 → σ(0) = 0.5
  
  σ'(0) = 0.25 (not zero, so gradients exist)
  
  But all neurons still have the SAME gradient.
  Network trains, but every neuron learns the same feature.
  Effectively a network with 1 neuron per layer.
```

### With Tanh

```
  z = 0 → tanh(0) = 0
  
  tanh'(0) = 1 (maximum gradient!)
  
  But same symmetry issue: all neurons identical.
```

---

## 4. Constant Initialization (Non-Zero)

Setting all weights to a constant c ≠ 0 has the **same symmetry problem**:

```
  W = [[c, c, c],     All rows identical
       [c, c, c],     → all neurons compute same thing
       [c, c, c]]     → same gradients → same updates
  
  For c > 0 with sigmoid:
  z = c·(x₁ + x₂ + x₃) + b    ← same for all neurons
  
  Still can't learn diverse features!
```

### The Real Issue: Row Symmetry

The problem isn't that weights are zero — it's that all **rows** of the weight matrix are identical. This means every neuron in the layer computes the same linear combination of inputs.

---

## 5. Why Biases CAN Be Zero

### Biases Don't Break Symmetry Anyway

```
  Neuron j: z_j = Σᵢ w_ji · xᵢ + b_j
  
  If all w_j are different (random), then all z_j are different,
  regardless of whether b_j = 0 or not.
  
  Bias just shifts the activation — it doesn't determine the
  "feature" the neuron detects.
```

### What Biases Do

```
  Without bias:  z = Wx      → hyperplane through origin
  With bias:     z = Wx + b  → hyperplane can be offset
  
  b = 0 is a perfectly good starting point:
  - The hyperplane starts through the origin
  - During training, b is updated to find the right offset
```

### Standard Practice

| Parameter | Init | Reason |
|---|---|---|
| **Weights W** | Random (Xavier/He) | Must break symmetry |
| **Bias b** | Zero | Safe, doesn't cause symmetry |
| **BatchNorm γ** | One | Start as identity |
| **BatchNorm β** | Zero | Start with no shift |

---

## 6. Visualizing the Dead Network

### Network Capacity vs Effective Capacity

```
  With random init (symmetry broken):
  
  Neuron 1: detects "horizontal edge"    ┐
  Neuron 2: detects "vertical edge"      ├── 4 diverse features
  Neuron 3: detects "diagonal edge"      │
  Neuron 4: detects "corner"             ┘
  
  Effective capacity: 4 neurons
  
  
  With zero/constant init (symmetry preserved):
  
  Neuron 1: detects "average"            ┐
  Neuron 2: detects "average"            ├── 1 feature (repeated 4×)
  Neuron 3: detects "average"            │
  Neuron 4: detects "average"            ┘
  
  Effective capacity: 1 neuron!
```

### Training Curve Comparison

```
  Loss ↑
  5.0  │ ────────────────────────────────  ← Zero init (ReLU): no learning
       │
  4.0  │ ·  ·  ·  · · · · · · · · · · ·  ← Constant init: learns but limited
       │
  3.0  │  ╲
       │   ╲
  2.0  │    ╲╲
       │      ╲╲
  1.0  │        ╲╲──
       │           ──────────────────     ← Random init: full learning
  0.0  │
       └──────────────────────────────→ Epoch
```

---

## 7. Worked Example

### Two-Neuron Hidden Layer

```
  Network: 2 inputs → 2 hidden (sigmoid) → 1 output (sigmoid)
  
  Target: XOR function
  
  Data: (0,0)→0, (0,1)→1, (1,0)→1, (1,1)→0
```

**With zero init:**

```
  W⁽¹⁾ = [[0, 0],     b⁽¹⁾ = [0, 0]
           [0, 0]]
  W⁽²⁾ = [0, 0]        b⁽²⁾ = 0

  Forward (any input):
  z₁ = z₂ = 0
  a₁ = a₂ = σ(0) = 0.5
  ŷ = σ(0·0.5 + 0·0.5 + 0) = σ(0) = 0.5
  
  Every input → same output 0.5 → same loss → same gradient
  → Both hidden neurons update identically → symmetry never breaks
  → Network CAN'T learn XOR (which requires 2 different neurons)
```

**With random init:**

```
  W⁽¹⁾ = [[0.3, -0.5],     b⁽¹⁾ = [0, 0]
           [-0.2, 0.7]]
  W⁽²⁾ = [0.4, -0.6]        b⁽²⁾ = 0

  Forward for (1, 0):
  z₁ = 0.3·1 + (-0.5)·0 = 0.3  → a₁ = σ(0.3) = 0.574
  z₂ = (-0.2)·1 + 0.7·0 = -0.2 → a₂ = σ(-0.2) = 0.450
  
  Different activations! Different gradients! 
  Neurons can specialize → network CAN learn XOR ✓
```

---

## 8. Python Demonstration

```python
import torch
import torch.nn as nn

# ── XOR dataset ─────────────────────────────────────────────────
X = torch.tensor([[0,0],[0,1],[1,0],[1,1]], dtype=torch.float32)
y = torch.tensor([[0],[1],[1],[0]], dtype=torch.float32)

# ── Train with different inits ──────────────────────────────────
def train_xor(init_type, epochs=5000, lr=0.1):
    torch.manual_seed(42)
    model = nn.Sequential(
        nn.Linear(2, 4), nn.Sigmoid(),
        nn.Linear(4, 1), nn.Sigmoid()
    )
    
    # Apply initialization
    if init_type == "zero":
        for p in model.parameters():
            nn.init.zeros_(p)
    elif init_type == "constant":
        for m in model:
            if isinstance(m, nn.Linear):
                nn.init.constant_(m.weight, 0.5)
                nn.init.zeros_(m.bias)
    elif init_type == "random":
        pass  # PyTorch default is already random
    
    optimizer = torch.optim.SGD(model.parameters(), lr=lr)
    criterion = nn.BCELoss()
    
    losses = []
    for epoch in range(epochs):
        pred = model(X)
        loss = criterion(pred, y)
        optimizer.zero_grad(); loss.backward(); optimizer.step()
        if epoch % 500 == 0:
            losses.append(loss.item())
    
    # Check final predictions
    with torch.no_grad():
        preds = model(X).round().squeeze().tolist()
    
    return losses, preds

for init_type in ["zero", "constant", "random"]:
    losses, preds = train_xor(init_type)
    expected = [0, 1, 1, 0]
    correct = preds == expected
    print(f"{init_type:>10s}: loss={losses[-1]:.4f}, preds={preds}, correct={correct}")

# Output:
#       zero: loss=0.6931, preds=[0.0, 0.0, 0.0, 0.0], correct=False  ← total failure
#   constant: loss=0.6914, preds=[1.0, 1.0, 1.0, 1.0], correct=False  ← symmetry
#     random: loss=0.0023, preds=[0.0, 1.0, 1.0, 0.0], correct=True   ← success!
```

### Checking Weight Symmetry After Training

```python
def check_symmetry(model):
    """Check if hidden layer weights remain symmetric after training."""
    W = model[0].weight.data
    print(f"\nWeight matrix shape: {W.shape}")
    for i in range(W.shape[0]):
        for j in range(i+1, W.shape[0]):
            diff = (W[i] - W[j]).abs().max().item()
            status = "SYMMETRIC" if diff < 1e-6 else "broken ✓"
            print(f"  Row {i} vs Row {j}: max diff = {diff:.6f} → {status}")

# Check zero-init model
torch.manual_seed(42)
zero_model = nn.Sequential(nn.Linear(2, 4), nn.Sigmoid(), nn.Linear(4, 1), nn.Sigmoid())
for p in zero_model.parameters():
    nn.init.zeros_(p)
    
opt = torch.optim.SGD(zero_model.parameters(), lr=0.1)
for _ in range(1000):
    loss = nn.BCELoss()(zero_model(X), y)
    opt.zero_grad(); loss.backward(); opt.step()

check_symmetry(zero_model)
# All rows remain SYMMETRIC even after 1000 training steps!
```

---

## Summary Table

| Concept | Key Idea |
|---|---|
| Zero init | All weights = 0 → all neurons identical → no learning |
| Constant init | Same problem as zero init (any constant) |
| Symmetry preservation | Identical weights → identical gradients → identical updates |
| ReLU + zero init | ReLU(0)=0, ReLU'(0)=0 → completely dead |
| Sigmoid + zero init | σ(0)=0.5 → gradients exist but identical → 1 effective neuron |
| Bias = zero | Safe because weights already break symmetry |
| Effective capacity | With symmetry: N neurons = 1 neuron |
| Solution | Random initialization (next chapter) |

---

## Revision Questions

1. **Prove that weight symmetry is preserved during gradient descent.** Show that if W⁽¹⁾[i,:] = W⁽¹⁾[j,:] at step t, then the same holds at step t+1.

2. **Why does zero initialization with ReLU result in a completely dead network**, while zero initialization with sigmoid still produces *some* gradient flow (but doesn't break symmetry)?

3. **Explain why biases can safely be initialized to zero** while weights cannot. What role does bias play in the computation?

4. **In the XOR example, why does the network need at least 2 different hidden neurons?** What does each neuron learn geometrically?

5. **Does the symmetry problem apply to the output layer?** Why or why not?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Why Initialization Matters](./01-why-initialization-matters.md) | [Unit 6: Weight Init](./README.md) | [Random Initialization →](./03-random-initialization.md) |
