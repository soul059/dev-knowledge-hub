# 3. Random Initialization

> **Unit 6 · Weight Initialization** — Breaking symmetry with random values, and why scale matters

---

## Chapter Overview

Random initialization solves the symmetry problem by ensuring each neuron starts with **unique** weights, allowing them to learn different features. However, **not all random initializations are equal** — the **scale** (standard deviation) of the random values critically determines whether signals and gradients propagate properly through deep networks. Too small and signals vanish; too large and they explode. This chapter analyzes why scale matters and sets the stage for principled initialization methods (Xavier, He).

---

## 1. Breaking Symmetry with Randomness

### Basic Idea

```
  Zero Init (broken):              Random Init (works):
  
  W = [[0, 0, 0],                 W = [[ 0.3, -0.5,  0.1],
       [0, 0, 0],                      [-0.2,  0.7, -0.4],
       [0, 0, 0]]                      [ 0.6, -0.1,  0.8]]
       ↑                                ↑
  All rows identical               All rows DIFFERENT
  → All neurons same               → Each neuron unique
  → 1 effective neuron             → Full capacity utilized
```

### Random Distributions

Two common choices:

**Normal (Gaussian):**
```
  W ~ N(0, σ²)     (mean=0, variance=σ²)
  
  PDF: p(w) = (1/√(2πσ²)) · exp(-w²/(2σ²))
  
  Most values within [-2σ, 2σ]
```

**Uniform:**
```
  W ~ U(-a, a)     (uniform between -a and a)
  
  Var(W) = a²/3    (for uniform on [-a, a])
  
  To match variance σ²: a = σ√3
```

### Why Mean = 0?

```
  Mean ≠ 0 creates a bias in the pre-activations:
  
  z_j = Σᵢ w_ji · x_i
  
  If E[w] = μ ≠ 0:
  E[z] = μ · Σᵢ E[x_i] ≠ 0  (systematic bias)
  
  With sigmoid/tanh, this pushes activations toward saturation.
  
  Mean = 0 ensures:
  E[z] = 0  → activations start centered → healthy gradient flow
```

---

## 2. The Scale Problem

### Signal Propagation Through Layers

For a single layer z = Wx (ignoring bias, assuming E[x] = 0, E[w] = 0):

```
  Var(z_j) = Var(Σᵢ w_ji · x_i)
           = Σᵢ Var(w_ji · x_i)          (assuming independence)
           = Σᵢ Var(w_ji) · Var(x_i)     (since E[w]=E[x]=0)
           = n_in · Var(w) · Var(x)       (all weights same variance)
```

So the variance **scales** by a factor of n_in · Var(w) at each layer.

### Through L Layers

```
  Var(z⁽ᴸ⁾) = [n · Var(w)]ᴸ · Var(x)
```

where n is the layer width (assuming all layers have width n).

| n · Var(w) | Effect | Result |
|---|---|---|
| **< 1** | Variance shrinks exponentially | **Signal vanishes** |
| **= 1** | Variance preserved | **Stable** ✓ |
| **> 1** | Variance grows exponentially | **Signal explodes** |

### ASCII Visualization

```
  10-layer network, n=100 neurons per layer:
  
  σ = 0.01 (too small): n·σ² = 100·0.0001 = 0.01
  ──────────────────────────────────────────────────
  Layer:    1      2      3      4      5     ...   10
  Var:     0.01   0.0001  1e-6   1e-8   1e-10 ... 1e-20
  Status:  small  tiny    ≈0     ≈0     ≈0         VANISHED
  
  σ = 0.1 (just right): n·σ² = 100·0.01 = 1.0
  ──────────────────────────────────────────────────
  Layer:    1      2      3      4      5     ...   10
  Var:     1.0    1.0    1.0    1.0    1.0    ...   1.0
  Status:  ✓      ✓      ✓      ✓      ✓           STABLE
  
  σ = 1.0 (too large): n·σ² = 100·1.0 = 100
  ──────────────────────────────────────────────────
  Layer:    1      2      3      4      5     ...   10
  Var:     100    10⁴    10⁶    10⁸    10¹⁰  ...   10²⁰
  Status:  large  huge   sat.   sat.   sat.        EXPLODED
```

---

## 3. Effect of Scale on Activations

### With Sigmoid Activation

```
  sigmoid(z) = 1/(1 + e⁻ᶻ)
  
  If |z| is large: sigmoid ≈ 0 or 1 → SATURATED → gradient ≈ 0
  If |z| is small: sigmoid ≈ 0.5 → LINEAR REGIME → gradient ≈ 0.25
  
  σ too large:
  z values: [-50, 42, -31, ...]
  activations: [0.000, 1.000, 0.000, ...]  ← all saturated!
  gradients: [≈0, ≈0, ≈0, ...]  ← vanishing!
  
  σ too small:
  z values: [-0.001, 0.002, -0.001, ...]
  activations: [0.4997, 0.5005, 0.4997, ...]  ← all ≈ 0.5
  network behaves like a LINEAR model (no nonlinearity utilized)
  
  σ just right:
  z values: [-1.2, 0.8, -0.5, ...]
  activations: [0.231, 0.690, 0.378, ...]  ← diverse, non-saturated ✓
```

### With ReLU Activation

```
  ReLU(z) = max(0, z)
  
  σ too small:
  z values: [-0.001, 0.002, -0.001, ...]
  activations: [0, 0.002, 0, ...]  ← tiny values, ~50% dead
  gradients: tiny → learning is very slow
  
  σ too large:
  z values: [-50, 42, -31, 65, ...]
  activations: [0, 42, 0, 65, ...]  ← large values → exploding
  Next layer's z values: even larger → EXPLOSION
  
  σ just right:
  z values: [-1.2, 0.8, -0.5, 1.5, ...]
  activations: [0, 0.8, 0, 1.5, ...]  ← healthy range ✓
```

---

## 4. Variance Analysis: Finding the Right Scale

### Goal

We want the variance of activations to remain **constant** across layers.

### For Linear Layers (No Activation)

```
  z = Wx,  where x ∈ ℝⁿⁱⁿ, W ∈ ℝⁿᵒᵘᵗ ˣ ⁿⁱⁿ
  
  Var(z_j) = n_in · Var(w) · Var(x)
  
  For Var(z) = Var(x):
  
  n_in · Var(w) = 1
  
  ∴ Var(w) = 1/n_in       →  σ = 1/√n_in
```

### For Sigmoid/Tanh

Near z=0, tanh(z) ≈ z (linear), so the linear analysis approximately holds:

```
  Var(w) ≈ 1/n_in   (works well for tanh in practice)
```

For sigmoid: σ(z) ≈ 0.25z + 0.5 near z=0, so the effective variance is scaled by 0.25²:

```
  Need to compensate: Var(w) ≈ 4/n_in for sigmoid (to account for the 0.25 slope)
```

### For ReLU

ReLU kills half the values (those < 0), reducing variance by factor 2:

```
  Var(ReLU(z)) = Var(z) / 2     (since half the values become 0)
  
  To compensate:
  n_in · Var(w) · (1/2) = 1
  
  ∴ Var(w) = 2/n_in        ← This is He initialization!
```

---

## 5. Practical Rules of Thumb

### Quick Reference

| Activation | Var(w) | σ (std dev) | Name |
|---|---|---|---|
| Linear/tanh | 1/n_in | 1/√n_in | LeCun |
| Sigmoid/tanh | 2/(n_in+n_out) | √(2/(n_in+n_out)) | Xavier/Glorot |
| ReLU | 2/n_in | √(2/n_in) | He/Kaiming |

### Example: Layer with n_in=784 (MNIST input)

```
  LeCun:    σ = 1/√784 = 0.0357
  Xavier:   σ = √(2/(784+256)) ≈ 0.0439  (assuming n_out=256)
  He:       σ = √(2/784) = 0.0505
  
  Typical weight range (within 2σ):
  LeCun:  [-0.071, 0.071]
  Xavier: [-0.088, 0.088]
  He:     [-0.101, 0.101]
```

---

## 6. Worked Example: Signal Propagation

### Problem
A 5-layer network with 100 neurons per layer, ReLU activation, input with Var(x) = 1.
Compare σ = 0.01, σ = 0.1, and σ = √(2/100) = 0.1414 (He init).

### Solution

Variance propagation formula with ReLU: Var(z⁽ˡ⁺¹⁾) = n · Var(w) · Var(z⁽ˡ⁾) / 2

```
σ = 0.01:  n·Var(w)/2 = 100·0.0001/2 = 0.005

Layer    Var(z)
1        0.005
2        2.5e-5
3        1.25e-7
4        6.25e-10
5        3.13e-12  ← VANISHED

σ = 0.1:  n·Var(w)/2 = 100·0.01/2 = 0.5

Layer    Var(z)
1        0.5
2        0.25
3        0.125
4        0.0625
5        0.03125  ← slowly vanishing

σ = √(2/100) = 0.1414:  n·Var(w)/2 = 100·(2/100)/2 = 1.0

Layer    Var(z)
1        1.0
2        1.0
3        1.0
4        1.0
5        1.0  ← PERFECT ✓
```

---

## 7. Python Implementation

### Comparing Random Init Scales

```python
import torch
import torch.nn as nn
import matplotlib.pyplot as plt

# ── Measure activation statistics through a deep network ────────
def measure_activation_stats(sigma, n_layers=10, width=256, n_samples=1000):
    """Forward pass through a deep network, recording activation stats."""
    x = torch.randn(n_samples, width)
    
    means, stds = [], []
    for layer in range(n_layers):
        W = torch.randn(width, width) * sigma
        z = x @ W.T
        x = torch.relu(z)  # ReLU activation
        
        means.append(x.mean().item())
        stds.append(x.std().item())
    
    return means, stds

# ── Test different scales ───────────────────────────────────────
n_in = 256
scales = {
    "σ=0.01 (too small)":  0.01,
    "σ=0.1":               0.1,
    f"σ=√(2/{n_in}) (He)": (2/n_in)**0.5,
    "σ=1.0 (too large)":   1.0,
}

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))

for label, sigma in scales.items():
    means, stds = measure_activation_stats(sigma, n_layers=15, width=n_in)
    ax1.plot(stds, label=label, linewidth=2)

ax1.set_xlabel("Layer")
ax1.set_ylabel("Activation Std Dev")
ax1.set_title("Activation Scale Through Layers (ReLU)")
ax1.set_yscale("log")
ax1.legend()
ax1.grid(True, alpha=0.3)
ax1.axhline(y=1.0, color='gray', linestyle='--', alpha=0.5)

# ── Also show distributions at specific layers ──────────────────
for i, sigma in enumerate([0.01, (2/256)**0.5, 1.0]):
    x = torch.randn(5000, 256)
    for _ in range(5):
        W = torch.randn(256, 256) * sigma
        x = torch.relu(x @ W.T)
    ax2.hist(x.flatten().numpy(), bins=50, alpha=0.5, density=True,
             label=f"σ={sigma:.3f}")

ax2.set_xlabel("Activation Value")
ax2.set_ylabel("Density")
ax2.set_title("Activation Distribution at Layer 5")
ax2.legend()

plt.tight_layout()
plt.savefig("random_init_scales.png", dpi=150)
plt.show()
```

### Training Comparison

```python
from torch.utils.data import DataLoader, TensorDataset

torch.manual_seed(42)
X = torch.randn(3000, 50)
y = (X @ torch.randn(50, 1) + torch.randn(3000, 1) * 0.5).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

def train_with_sigma(sigma, depth=8, width=128, epochs=30):
    torch.manual_seed(0)
    layers = []
    in_d = 50
    for _ in range(depth):
        lin = nn.Linear(in_d, width)
        nn.init.normal_(lin.weight, 0, sigma)
        nn.init.zeros_(lin.bias)
        layers.extend([lin, nn.ReLU()])
        in_d = width
    layers.append(nn.Linear(width, 1))
    model = nn.Sequential(*layers)
    
    opt = torch.optim.Adam(model.parameters(), lr=0.001)
    losses = []
    for epoch in range(epochs):
        total = 0
        for xb, yb in loader:
            loss = nn.MSELoss()(model(xb).squeeze(), yb)
            if torch.isnan(loss):
                return [float('nan')] * epochs
            opt.zero_grad(); loss.backward(); opt.step()
            total += loss.item() * len(xb)
        losses.append(total / len(X))
    return losses

sigmas = {"0.001": 0.001, "0.01": 0.01, "He": (2/128)**0.5, "0.5": 0.5, "2.0": 2.0}
fig, ax = plt.subplots(figsize=(10, 6))
for label, sigma in sigmas.items():
    losses = train_with_sigma(sigma)
    ax.plot(losses, label=f"σ={label}", linewidth=2)
ax.set_xlabel("Epoch"); ax.set_ylabel("Loss"); ax.set_yscale("log")
ax.set_title("Training Loss for Different Init Scales (8-layer ReLU network)")
ax.legend(); ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig("init_scale_training.png", dpi=150)
plt.show()
```

---

## 8. Key Takeaways

### The Goldilocks Zone

```
  Init Scale Spectrum:
  
  ← Too Small ──────────── Just Right ──────────── Too Large →
  
  Vanishing           Stable propagation          Exploding
  signals/gradients   across all layers           signals/gradients
  
  Slow/no learning    Fast convergence            NaN / instability
  
  σ ≈ 0.001           σ ≈ 1/√n_in (tanh)        σ ≈ 1.0
  for n=100            σ ≈ √(2/n_in) (ReLU)       for n=100
```

---

## Summary Table

| Concept | Key Idea |
|---|---|
| Random init | Breaks symmetry — each neuron starts different |
| Normal W ~ N(0, σ²) | Most common random distribution |
| Uniform W ~ U(-a, a) | Alternative with Var = a²/3 |
| Mean = 0 | Prevents systematic bias in pre-activations |
| Scale matters | Too small → vanishing, too large → exploding |
| Variance propagation | Var(z⁽ˡ⁺¹⁾) = n_in · Var(w) · Var(z⁽ˡ⁾) |
| Stability condition | n_in · Var(w) = 1 (linear/tanh) |
| ReLU correction | n_in · Var(w) = 2 (accounts for half being zeroed) |
| Next step | Principled formulas: Xavier (ch.4) and He (ch.5) |

---

## Revision Questions

1. **Derive the variance propagation formula** Var(z_j) = n_in · Var(w) · Var(x) for a linear layer, stating all assumptions.

2. **Why must the mean of initialization be zero?** What happens to sigmoid activations if weights are initialized with positive mean?

3. **Calculate the init scale** for a ReLU network with layers of width [784, 512, 256, 128, 10]. What should σ be for each layer?

4. **Explain the difference between vanishing signals and vanishing gradients.** Are they the same problem?

5. **Why does ReLU require a different scale than tanh?** Show the factor-of-2 difference mathematically.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Zero Init Problem](./02-zero-initialization-problem.md) | [Unit 6: Weight Init](./README.md) | [Xavier/Glorot Init →](./04-xavier-glorot-initialization.md) |
