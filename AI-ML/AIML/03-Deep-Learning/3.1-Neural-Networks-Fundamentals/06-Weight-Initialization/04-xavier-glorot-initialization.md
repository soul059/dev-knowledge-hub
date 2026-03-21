# 4. Xavier/Glorot Initialization

> **Unit 6 · Weight Initialization** — Maintaining variance for sigmoid and tanh networks

---

## Chapter Overview

**Xavier initialization** (Glorot & Bengio, 2010) derives the optimal weight variance by requiring that both **forward-pass activations** and **backward-pass gradients** maintain stable variance across layers. The resulting formula Var(W) = 2/(n_in + n_out) is a **compromise** between the forward and backward requirements, and works well with **sigmoid** and **tanh** activations. This chapter provides the full derivation, both normal and uniform variants, and practical PyTorch usage.

---

## 1. Derivation

### Assumptions

1. Weights and inputs are **independent** and **zero-mean**
2. Activations are in the **linear regime** (valid for tanh near zero)
3. Weights are i.i.d. with variance Var(w)

### Forward Pass Condition

For layer l with n_in inputs:

```
  z_j⁽ˡ⁾ = Σᵢ₌₁ⁿⁱⁿ w_ji⁽ˡ⁾ · a_i⁽ˡ⁻¹⁾

  Var(z_j⁽ˡ⁾) = n_in · Var(w⁽ˡ⁾) · Var(a⁽ˡ⁻¹⁾)
```

For stable forward propagation across L layers:

```
  Var(z⁽ᴸ⁾) = Var(x) · Πₗ₌₁ᴸ [nₗ_in · Var(w⁽ˡ⁾)]
```

We need each factor to equal 1:

```
  nₗ_in · Var(w⁽ˡ⁾) = 1   →   Var(w⁽ˡ⁾) = 1/nₗ_in     ... (1)
```

### Backward Pass Condition

For gradients flowing backward:

```
  ∂L/∂a_i⁽ˡ⁻¹⁾ = Σⱼ₌₁ⁿᵒᵘᵗ w_ji⁽ˡ⁾ · ∂L/∂z_j⁽ˡ⁾

  Var(∂L/∂a⁽ˡ⁻¹⁾) = n_out · Var(w⁽ˡ⁾) · Var(∂L/∂z⁽ˡ⁾)
```

For stable backward propagation:

```
  nₗ_out · Var(w⁽ˡ⁾) = 1   →   Var(w⁽ˡ⁾) = 1/nₗ_out   ... (2)
```

### The Compromise

Equations (1) and (2) generally can't both be satisfied (unless n_in = n_out). Xavier uses the **harmonic mean**:

```
  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │          Var(W) = 2 / (n_in + n_out)             │
  │                                                  │
  │   Xavier/Glorot Initialization                   │
  │                                                  │
  └──────────────────────────────────────────────────┘
```

### Why Harmonic Mean?

```
  Forward:   Var(w) = 1/n_in       = 2/(2·n_in)
  Backward:  Var(w) = 1/n_out      = 2/(2·n_out)
  
  Average:   Var(w) = 2/(n_in + n_out)    ← harmonic mean of 1/n_in and 1/n_out
  
  This balances both conditions equally.
```

---

## 2. Normal vs Uniform Variants

### Xavier Normal

```
  W ~ N(0, σ²)   where σ² = 2/(n_in + n_out)
  
  σ = √(2/(n_in + n_out))
```

### Xavier Uniform

For uniform distribution U(-a, a), Var = a²/3, so:

```
  a²/3 = 2/(n_in + n_out)
  
  a = √(6/(n_in + n_out))
  
  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │  W ~ U(-√(6/(n_in + n_out)), √(6/(n_in + n_out))) │
  │                                                  │
  └──────────────────────────────────────────────────┘
```

### Comparison

| Variant | Formula | Distribution Shape |
|---|---|---|
| Xavier Normal | N(0, 2/(n_in+n_out)) | Bell curve, unbounded tails |
| Xavier Uniform | U(-a, a), a=√(6/(n_in+n_out)) | Flat, bounded range |

Both have the **same variance**, so both maintain signal propagation equally well. The choice is mostly a matter of preference; uniform is slightly more predictable (bounded).

---

## 3. Why Xavier Works for Sigmoid/Tanh

### The Linear Regime Assumption

Xavier's derivation assumes the activation function behaves **linearly** near z=0:

```
  tanh(z) ≈ z         for |z| << 1      (slope = 1 at z=0)
  sigmoid(z) ≈ 0.25z + 0.5  for |z| << 1 (slope = 0.25 at z=0)
```

If weights are properly initialized so that z values stay near zero:

```
  Activations:
  
      1 │          ╱─── sigmoid
        │         ╱
   0.5 │    ╱───╱         ← linear regime
        │   ╱  ╱               used by Xavier
      0 │──╱──╱──────
        │ ╱  ╱
   -0.5│╱  ╱
        │ ╱
     -1 │╱─────────── tanh
        └──────────────→ z
       -4  -2   0   2   4
```

With Xavier init, z values are typically in [-1, 1], where tanh ≈ z, so the linear analysis holds.

### Why Xavier Doesn't Work Well for ReLU

ReLU is **not** linear — it kills half the values:

```
  ReLU(z) = max(0, z)
  
  If z ~ N(0, σ²):
  - 50% of z values are negative → ReLU outputs 0
  - Var(ReLU(z)) = Var(z)/2   (half the variance is lost!)
  
  Xavier doesn't account for this factor of 2 → signal slowly vanishes
  in deep ReLU networks.
```

```
  Signal Propagation in 20-layer ReLU network:
  
  Var(a) ↑
         │── Xavier (designed for tanh)
    1.0  │──
         │  ──
    0.5  │    ──
         │      ──
    0.25 │        ──
         │          ──
    0.1  │            ──── slowly vanishing!
         │
    0.0  └──────────────────────────────→ Layer
          1    5     10    15    20
         
  Each ReLU layer loses 50% of variance.
  After 20 layers: Var ≈ 0.5²⁰ ≈ 1e-6
```

---

## 4. Example Calculations

### Example 1: MNIST Hidden Layer

```
  Layer: 784 → 256
  n_in = 784, n_out = 256
  
  Xavier Normal: σ = √(2/(784+256)) = √(2/1040) = √0.001923 = 0.04386
  Xavier Uniform: a = √(6/1040) = √0.005769 = 0.07595
  
  Weights drawn from: N(0, 0.04386²) or U(-0.0760, 0.0760)
```

### Example 2: Convolutional Layer

For a Conv2d layer with kernel_size=3, in_channels=64, out_channels=128:

```
  n_in = in_channels × kernel_size² = 64 × 9 = 576
  n_out = out_channels × kernel_size² = 128 × 9 = 1152
  
  Xavier Normal: σ = √(2/(576+1152)) = √(2/1728) = √0.001157 = 0.03402
  Xavier Uniform: a = √(6/1728) = √0.003472 = 0.05893
```

### Example 3: Square Layer (n_in = n_out = 512)

```
  σ = √(2/(512+512)) = √(2/1024) = √0.001953 = 0.04419
  
  Notice: Var = 1/n_in = 1/n_out = 1/512 ≈ 0.001953
  Both forward and backward conditions are perfectly satisfied!
```

---

## 5. Worked Example: Variance Through 5 Layers

### Problem
Network: 100 → 200 → 150 → 200 → 100 → 10, tanh activation, Xavier init.
Show that variance stays approximately stable.

### Solution

For each layer, the variance scaling factor is:

```
  Factor = n_in · Var(w) = n_in · 2/(n_in + n_out)
```

| Layer | n_in → n_out | Var(w) | Factor = n_in · Var(w) |
|---|---|---|---|
| 1 | 100 → 200 | 2/300 = 0.00667 | 100 · 0.00667 = **0.667** |
| 2 | 200 → 150 | 2/350 = 0.00571 | 200 · 0.00571 = **1.143** |
| 3 | 150 → 200 | 2/350 = 0.00571 | 150 · 0.00571 = **0.857** |
| 4 | 200 → 100 | 2/300 = 0.00667 | 200 · 0.00667 = **1.333** |
| 5 | 100 → 10 | 2/110 = 0.01818 | 100 · 0.01818 = **1.818** |

Cumulative variance: 0.667 × 1.143 × 0.857 × 1.333 × 1.818 ≈ **1.58**

Compare with using only forward condition (Var(w) = 1/n_in):
Each factor would be exactly 1.0, cumulative = 1.0. But backward factors would vary.

Xavier's compromise keeps factors **near 1** in both directions.

---

## 6. Python Implementation (PyTorch)

### Using PyTorch's Built-in Xavier Init

```python
import torch
import torch.nn as nn

# ── Xavier Normal ───────────────────────────────────────────────
layer = nn.Linear(784, 256)
nn.init.xavier_normal_(layer.weight)
print(f"Xavier Normal: mean={layer.weight.mean():.5f}, std={layer.weight.std():.5f}")
print(f"Expected std: {(2/(784+256))**0.5:.5f}")

# ── Xavier Uniform ──────────────────────────────────────────────
nn.init.xavier_uniform_(layer.weight)
print(f"\nXavier Uniform: min={layer.weight.min():.5f}, max={layer.weight.max():.5f}")
print(f"Expected bound: ±{(6/(784+256))**0.5:.5f}")
```

### Applying Xavier to a Full Model

```python
def xavier_init_model(model):
    """Apply Xavier initialization to all linear and conv layers."""
    for m in model.modules():
        if isinstance(m, nn.Linear):
            nn.init.xavier_normal_(m.weight)
            if m.bias is not None:
                nn.init.zeros_(m.bias)
        elif isinstance(m, nn.Conv2d):
            nn.init.xavier_normal_(m.weight)
            if m.bias is not None:
                nn.init.zeros_(m.bias)

model = nn.Sequential(
    nn.Linear(784, 512), nn.Tanh(),
    nn.Linear(512, 256), nn.Tanh(),
    nn.Linear(256, 128), nn.Tanh(),
    nn.Linear(128, 10)
)
xavier_init_model(model)
```

### Verifying Signal Propagation

```python
import matplotlib.pyplot as plt

def check_propagation(model, input_dim, n_samples=1000):
    """Check if activation variance is stable across layers."""
    x = torch.randn(n_samples, input_dim)
    
    stats = []
    for layer in model:
        x = layer(x)
        if isinstance(layer, (nn.Tanh, nn.Sigmoid, nn.ReLU)):
            stats.append({
                'mean': x.mean().item(),
                'std': x.std().item(),
                'var': x.var().item()
            })
    
    return stats

# Compare Xavier vs random init
torch.manual_seed(42)

# Xavier
model_xavier = nn.Sequential(
    nn.Linear(100, 200), nn.Tanh(),
    nn.Linear(200, 200), nn.Tanh(),
    nn.Linear(200, 200), nn.Tanh(),
    nn.Linear(200, 200), nn.Tanh(),
    nn.Linear(200, 200), nn.Tanh(),
)
xavier_init_model(model_xavier)

# Bad random init
model_bad = nn.Sequential(
    nn.Linear(100, 200), nn.Tanh(),
    nn.Linear(200, 200), nn.Tanh(),
    nn.Linear(200, 200), nn.Tanh(),
    nn.Linear(200, 200), nn.Tanh(),
    nn.Linear(200, 200), nn.Tanh(),
)
for m in model_bad.modules():
    if isinstance(m, nn.Linear):
        nn.init.normal_(m.weight, 0, 1.0)  # too large!

stats_xavier = check_propagation(model_xavier, 100)
stats_bad = check_propagation(model_bad, 100)

fig, ax = plt.subplots(figsize=(8, 5))
ax.plot([s['std'] for s in stats_xavier], 'bo-', label='Xavier Init', linewidth=2)
ax.plot([s['std'] for s in stats_bad], 'ro-', label='Random (σ=1)', linewidth=2)
ax.set_xlabel('Layer (after tanh)')
ax.set_ylabel('Activation Std Dev')
ax.set_title('Signal Propagation: Xavier vs Large Random Init')
ax.legend()
ax.grid(True, alpha=0.3)
ax.set_yscale('log')
plt.tight_layout()
plt.savefig("xavier_propagation.png", dpi=150)
plt.show()
```

### Training Comparison

```python
from torch.utils.data import DataLoader, TensorDataset

torch.manual_seed(42)
X = torch.randn(3000, 100)
y = (X @ torch.randn(100, 1) + torch.randn(3000, 1) * 0.5).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

def train_with_init(init_fn, activation, epochs=40):
    torch.manual_seed(0)
    model = nn.Sequential(
        nn.Linear(100, 256), activation(),
        nn.Linear(256, 256), activation(),
        nn.Linear(256, 256), activation(),
        nn.Linear(256, 256), activation(),
        nn.Linear(256, 1)
    )
    init_fn(model)
    opt = torch.optim.SGD(model.parameters(), lr=0.01)
    
    losses = []
    for epoch in range(epochs):
        total = 0
        for xb, yb in loader:
            loss = nn.MSELoss()(model(xb).squeeze(), yb)
            if torch.isnan(loss): return [float('nan')] * epochs
            opt.zero_grad(); loss.backward(); opt.step()
            total += loss.item() * len(xb)
        losses.append(total / len(X))
    return losses

# Xavier init
xavier_losses = train_with_init(xavier_init_model, nn.Tanh)

# Large random init
def large_init(model):
    for m in model.modules():
        if isinstance(m, nn.Linear):
            nn.init.normal_(m.weight, 0, 1.0)
large_losses = train_with_init(large_init, nn.Tanh)

# Small random init
def small_init(model):
    for m in model.modules():
        if isinstance(m, nn.Linear):
            nn.init.normal_(m.weight, 0, 0.001)
small_losses = train_with_init(small_init, nn.Tanh)

print(f"Xavier:      {xavier_losses[-1]:.4f}")
print(f"Large σ=1.0: {large_losses[-1]}")
print(f"Small σ=0.001: {small_losses[-1]:.4f}")
```

---

## 7. Xavier in Different Frameworks

| Framework | Normal | Uniform |
|---|---|---|
| **PyTorch** | `nn.init.xavier_normal_(w)` | `nn.init.xavier_uniform_(w)` |
| **TensorFlow** | `GlorotNormal()` | `GlorotUniform()` (default!) |
| **JAX** | `glorot_normal()` | `glorot_uniform()` |

Note: TensorFlow uses Xavier Uniform as its **default** initialization for Dense layers.

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| Forward condition | Var(w) = 1/n_in |
| Backward condition | Var(w) = 1/n_out |
| Xavier compromise | **Var(w) = 2/(n_in + n_out)** |
| Normal variant | W ~ N(0, 2/(n_in + n_out)) |
| Uniform variant | W ~ U(-√(6/(n_in+n_out)), √(6/(n_in+n_out))) |
| Best for | Sigmoid, tanh activations |
| Not ideal for | ReLU (use He instead) |
| Key assumption | Activation is approximately linear near z=0 |
| PyTorch | `nn.init.xavier_normal_(weight)` |

---

## Revision Questions

1. **Derive the Xavier variance formula** from the forward and backward stability conditions. Show why the harmonic mean 2/(n_in + n_out) is the right compromise.

2. **Compute Xavier init parameters** for a Conv2d(64, 128, kernel_size=5). What is the std for normal variant and the bound for uniform variant?

3. **Why does Xavier not work well with ReLU?** Show mathematically that the variance decays by a factor of 2 per ReLU layer with Xavier init.

4. **Compare Xavier Normal and Xavier Uniform.** Under what circumstances might you prefer one over the other?

5. **Verify empirically** that Xavier initialization maintains activation variance across 10 layers of a tanh network. Plot the std at each layer.

6. **What happens to Xavier init when n_in >> n_out (e.g., 1000 → 10)?** Is the init variance closer to 1/n_in or 1/n_out?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Random Initialization](./03-random-initialization.md) | [Unit 6: Weight Init](./README.md) | [He/Kaiming Init →](./05-he-initialization.md) |
