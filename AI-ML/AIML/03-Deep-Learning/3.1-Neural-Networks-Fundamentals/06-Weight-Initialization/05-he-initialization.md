# 5. He/Kaiming Initialization

> **Unit 6 · Weight Initialization** — Designed for ReLU: accounting for the half-zeroing effect

---

## Chapter Overview

Xavier initialization assumes activations behave linearly, which works for sigmoid/tanh but **fails for ReLU**. ReLU zeroes out all negative values, cutting the variance in half at every layer. **He initialization** (He et al., 2015) corrects for this by doubling the variance, using **Var(W) = 2/n_in**. This simple change enables stable training of very deep networks (50+ layers) with ReLU activations and has become the **standard initialization for modern deep learning**.

---

## 1. Why Xavier Fails for ReLU

### The Factor-of-2 Problem

For a linear layer followed by ReLU:

```
  z = W·a + b           (pre-activation)
  a' = ReLU(z)           (post-activation)
```

If z ~ N(0, σ²), then ReLU(z) zeroes out all negative values:

```
  Before ReLU:           After ReLU:
  
  │     ╱╲               │        ╱╲
  │   ╱╱  ╲╲             │       ╱╱  ╲╲
  │  ╱╱    ╲╲            │      ╱╱    ╲╲
  │╱╱        ╲╲          │     ╱╱        ╲╲
  ├───────────────→ z    │ ███╱╱            ╲╲
  │                      │ ███                ╲╲
  │ 50% of values        │ ████████████████████→ z
  │ are negative         │ zero'd out!
  
  Var(z) = σ²            Var(ReLU(z)) = σ²/2
```

### Proof: Var(ReLU(z)) = Var(z)/2

For z ~ N(0, σ²):

```
  E[ReLU(z)] = E[z · 1(z>0)] = ∫₀^∞ z · p(z) dz = σ/√(2π)
  
  E[ReLU(z)²] = E[z² · 1(z>0)] = ∫₀^∞ z² · p(z) dz = σ²/2
  
  Var(ReLU(z)) = E[ReLU(z)²] - E[ReLU(z)]²
               = σ²/2 - σ²/(2π)
               ≈ σ²/2   (dominant term)
```

More precisely: Var(ReLU(z)) = σ²(1/2 - 1/(2π)) ≈ 0.341σ², but the factor 1/2 is the key scaling.

### Cascade Through L Layers

With Xavier init (Var(w) = 2/(n_in + n_out) ≈ 1/n for square layers):

```
  Forward: each layer multiplies variance by n·Var(w)·(1/2) = 1·(1/2) = 0.5
  
  After L layers: Var(a⁽ᴸ⁾) ≈ (1/2)ᴸ · Var(x)
  
  L=10:  Var ≈ 1/1024 ≈ 0.001  ← signals have mostly vanished!
  L=20:  Var ≈ 1/10⁶            ← practically zero
  L=50:  Var ≈ 10⁻¹⁵            ← numerical zero
```

---

## 2. He Initialization Derivation

### Forward Pass

For layer l with ReLU:

```
  z⁽ˡ⁾ = W⁽ˡ⁾ · a⁽ˡ⁻¹⁾
  a⁽ˡ⁾ = ReLU(z⁽ˡ⁾)
  
  Var(z⁽ˡ⁾) = n_in · Var(w) · Var(a⁽ˡ⁻¹⁾)
  Var(a⁽ˡ⁾) = Var(z⁽ˡ⁾) / 2                    ← ReLU halves variance
  
  Combined:
  Var(a⁽ˡ⁾) = (n_in · Var(w) / 2) · Var(a⁽ˡ⁻¹⁾)
```

For stable forward propagation, we need the scaling factor to equal 1:

```
  n_in · Var(w) / 2 = 1
  
  ┌──────────────────────────────────────────────┐
  │                                              │
  │         Var(W) = 2 / n_in                    │
  │                                              │
  │   He/Kaiming Initialization                  │
  │                                              │
  └──────────────────────────────────────────────┘
```

### Backward Pass

He et al. also analyzed the backward pass:

```
  For stable backward gradient propagation:
  
  Var(w) = 2 / n_out
```

In practice, the **forward-pass version** (2/n_in) is used by default, as it ensures proper signal propagation, which is more critical for training deep networks.

---

## 3. Normal and Uniform Variants

### He Normal

```
  W ~ N(0, σ²)   where σ² = 2/n_in
  
  σ = √(2/n_in)
```

### He Uniform

For uniform U(-a, a), Var = a²/3:

```
  a²/3 = 2/n_in
  
  a = √(6/n_in)
  
  W ~ U(-√(6/n_in), √(6/n_in))
```

### Comparison with Xavier

| Init | Var(W) | σ (for n_in=n_out=n) | Designed For |
|---|---|---|---|
| Xavier Normal | 2/(n_in+n_out) = 1/n | 1/√n | Sigmoid, Tanh |
| **He Normal** | **2/n_in** | **√(2/n)** | **ReLU, Leaky ReLU** |
| Xavier Uniform | U(±√(6/(n_in+n_out))) | — | Sigmoid, Tanh |
| **He Uniform** | **U(±√(6/n_in))** | — | **ReLU, Leaky ReLU** |

Key difference: He has **2× the variance** of Xavier (for square layers), compensating for ReLU's halving effect.

---

## 4. Signal Propagation: Xavier vs He on ReLU Networks

### Activation Variance Through Layers

```
  20-layer ReLU network, n=256 per layer:
  
  Var(a) ↑ (log scale)
         │
  10.0   │                         ·  · ← Random (σ=0.1): EXPLODES
         │                    ·  ·
   1.0   │── ── ── ── ── ── ── ── ── ──── He: STABLE ✓
         │──
   0.1   │  ──
         │    ── ──
  0.01   │        ── ──
         │             ── ──
  0.001  │                  ── ──── Xavier: VANISHES
         │
  0.0001 │
         └──────────────────────────────→ Layer
          1    5     10    15    20
```

### Mathematical Confirmation

```
  Xavier on ReLU (n=256):
  Factor per layer = n_in · Var(w) · 1/2 = 256 · (1/256) · 0.5 = 0.5
  After 20 layers: 0.5²⁰ = 9.5 × 10⁻⁷ ← vanished
  
  He on ReLU (n=256):
  Factor per layer = n_in · Var(w) · 1/2 = 256 · (2/256) · 0.5 = 1.0
  After 20 layers: 1.0²⁰ = 1.0 ← perfect! ✓
```

---

## 5. Variants for Other Activations

### Leaky ReLU

Leaky ReLU with negative slope α doesn't fully zero out negatives:

```
  LeakyReLU(z) = {  z      if z > 0
                 {  α·z    if z ≤ 0    (α typically 0.01 or 0.2)
  
  Var(LeakyReLU(z)) = σ² · (1 + α²) / 2
```

He init for Leaky ReLU:

```
  Var(w) = 2 / (n_in · (1 + α²))
```

### PReLU / ELU / SELU

| Activation | Var(W) | PyTorch `nonlinearity` arg |
|---|---|---|
| ReLU | 2/n_in | `'relu'` |
| Leaky ReLU (α=0.01) | 2/(n_in · 1.0001) ≈ 2/n_in | `'leaky_relu'` |
| Leaky ReLU (α=0.2) | 2/(n_in · 1.04) | `'leaky_relu'` |
| SELU | 1/n_in (LeCun init) | — |

---

## 6. Worked Example

### Problem
Initialize a 5-layer ReLU network: 784 → 512 → 256 → 128 → 64 → 10.

### Solution (He Normal)

| Layer | n_in | Var(w) = 2/n_in | σ = √Var | Weight Range (±2σ) |
|---|---|---|---|---|
| 1 | 784 | 0.002551 | 0.0505 | ±0.101 |
| 2 | 512 | 0.003906 | 0.0625 | ±0.125 |
| 3 | 256 | 0.007813 | 0.0884 | ±0.177 |
| 4 | 128 | 0.015625 | 0.1250 | ±0.250 |
| 5 | 64 | 0.031250 | 0.1768 | ±0.354 |

Notice: **smaller layers get larger init values** (more variance per weight to compensate for fewer weights contributing to each neuron's output).

### Verify Forward Propagation

Starting with Var(x) = 1:

```
  Layer 1: Var(a) = n_in · Var(w) · Var(x) / 2 = 784 · (2/784) · 1 / 2 = 1.0 ✓
  Layer 2: Var(a) = 512 · (2/512) · 1.0 / 2 = 1.0 ✓
  Layer 3: Var(a) = 256 · (2/256) · 1.0 / 2 = 1.0 ✓
  ... every layer: 1.0 ✓
```

---

## 7. Python Implementation (PyTorch)

### Using PyTorch's Built-in He Init

```python
import torch
import torch.nn as nn

# ── Kaiming Normal (default for ReLU) ──────────────────────────
layer = nn.Linear(512, 256)

nn.init.kaiming_normal_(layer.weight, mode='fan_in', nonlinearity='relu')
print(f"He Normal: std = {layer.weight.std():.4f}")
print(f"Expected:  std = {(2/512)**0.5:.4f}")

# ── Kaiming Uniform ────────────────────────────────────────────
nn.init.kaiming_uniform_(layer.weight, mode='fan_in', nonlinearity='relu')
bound = (6/512)**0.5
print(f"\nHe Uniform: range = [{layer.weight.min():.4f}, {layer.weight.max():.4f}]")
print(f"Expected:   range = [{-bound:.4f}, {bound:.4f}]")

# ── Fan modes ──────────────────────────────────────────────────
# mode='fan_in'  → Var = 2/n_in  (preserves forward-pass variance)
# mode='fan_out' → Var = 2/n_out (preserves backward-pass variance)
```

### Applying He Init to a Complete Model

```python
def he_init_model(model):
    """Apply He/Kaiming initialization to all layers."""
    for m in model.modules():
        if isinstance(m, nn.Linear):
            nn.init.kaiming_normal_(m.weight, mode='fan_in', nonlinearity='relu')
            if m.bias is not None:
                nn.init.zeros_(m.bias)
        elif isinstance(m, nn.Conv2d):
            nn.init.kaiming_normal_(m.weight, mode='fan_out', nonlinearity='relu')
            if m.bias is not None:
                nn.init.zeros_(m.bias)
        elif isinstance(m, nn.BatchNorm2d):
            nn.init.ones_(m.weight)
            nn.init.zeros_(m.bias)

# Example: Deep ReLU network
model = nn.Sequential(
    nn.Linear(784, 512), nn.ReLU(),
    nn.Linear(512, 256), nn.ReLU(),
    nn.Linear(256, 128), nn.ReLU(),
    nn.Linear(128, 64), nn.ReLU(),
    nn.Linear(64, 10)
)
he_init_model(model)
```

### Comparing Xavier vs He on Deep ReLU Network

```python
import matplotlib.pyplot as plt
from torch.utils.data import DataLoader, TensorDataset

torch.manual_seed(42)
X = torch.randn(3000, 100)
y = (X @ torch.randn(100, 1) + torch.randn(3000, 1) * 0.3).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

def train(init_fn, depth=10, epochs=40):
    torch.manual_seed(0)
    layers = []
    in_d = 100
    for _ in range(depth):
        layers.extend([nn.Linear(in_d, 256), nn.ReLU()])
        in_d = 256
    layers.append(nn.Linear(256, 1))
    model = nn.Sequential(*layers)
    init_fn(model)
    
    opt = torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
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

def xavier_init(model):
    for m in model.modules():
        if isinstance(m, nn.Linear):
            nn.init.xavier_normal_(m.weight)
            nn.init.zeros_(m.bias)

xavier_losses = train(xavier_init)
he_losses = train(he_init_model)

fig, ax = plt.subplots(figsize=(10, 5))
ax.plot(xavier_losses, label='Xavier (designed for tanh)', linewidth=2)
ax.plot(he_losses, label='He/Kaiming (designed for ReLU)', linewidth=2)
ax.set_xlabel('Epoch'); ax.set_ylabel('Loss'); ax.set_yscale('log')
ax.set_title('Xavier vs He Init on 10-Layer ReLU Network')
ax.legend(); ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig("xavier_vs_he.png", dpi=150)
plt.show()
```

### Visualizing Activation Distributions

```python
def visualize_activations(model, x, title):
    """Plot activation distributions at each layer."""
    activations = []
    h = x
    for layer in model:
        h = layer(h)
        if isinstance(layer, nn.ReLU):
            activations.append(h.detach().clone())
    
    fig, axes = plt.subplots(1, len(activations), figsize=(3*len(activations), 3))
    for i, (act, ax) in enumerate(zip(activations, axes)):
        ax.hist(act.flatten().numpy(), bins=50, density=True, alpha=0.7)
        ax.set_title(f'Layer {i+1}\nμ={act.mean():.3f}, σ={act.std():.3f}')
        ax.set_xlim(-0.5, 3)
    fig.suptitle(title, fontsize=14)
    plt.tight_layout()
    plt.savefig(f"activations_{title.replace(' ', '_')}.png", dpi=150)
    plt.show()

x_sample = torch.randn(1000, 100)

# He init
torch.manual_seed(0)
model_he = nn.Sequential(*[nn.Linear(100 if i==0 else 256, 256) if j==0 
                           else nn.ReLU() for i in range(5) for j in range(2)])
he_init_model(model_he)
visualize_activations(model_he, x_sample, "He Init")

# Xavier init
torch.manual_seed(0)
model_xav = nn.Sequential(*[nn.Linear(100 if i==0 else 256, 256) if j==0 
                            else nn.ReLU() for i in range(5) for j in range(2)])
xavier_init(model_xav)
visualize_activations(model_xav, x_sample, "Xavier Init")
```

---

## 8. PyTorch Default Initialization

Note: PyTorch's default initialization for `nn.Linear` is actually **Kaiming Uniform** (fan_in mode):

```python
# PyTorch default (see source code):
# nn.Linear.__init__ uses:
#   init.kaiming_uniform_(self.weight, a=math.sqrt(5))
#   fan_in, _ = init._calculate_fan_in_and_fan_out(self.weight)
#   bound = 1 / math.sqrt(fan_in) if fan_in > 0 else 0
#   init.uniform_(self.bias, -bound, bound)
```

So PyTorch already uses a variant of He init by default! But explicitly setting it is still recommended for:
- Control over the mode (fan_in vs fan_out)
- Choosing the nonlinearity (relu vs leaky_relu)
- Ensuring biases are zeroed (PyTorch default bias init is not zero)

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| ReLU's effect | Var(ReLU(z)) = Var(z)/2 |
| He forward | **Var(W) = 2/n_in** |
| He backward | Var(W) = 2/n_out |
| He Normal | W ~ N(0, 2/n_in) |
| He Uniform | W ~ U(-√(6/n_in), √(6/n_in)) |
| vs Xavier | He has 2× the variance (compensates ReLU halving) |
| For Leaky ReLU | Var(W) = 2/(n_in · (1+α²)) |
| PyTorch | `nn.init.kaiming_normal_(w, mode='fan_in', nonlinearity='relu')` |
| PyTorch default | Already uses Kaiming Uniform (approximately) |

---

## Revision Questions

1. **Derive He initialization from first principles.** Start with the variance of ReLU(z) and show why Var(W) = 2/n_in maintains stable forward propagation.

2. **Why does Xavier init cause signal vanishing in deep ReLU networks?** Calculate the signal variance after 20 layers with Xavier vs He init (n=256).

3. **What is the difference between `mode='fan_in'` and `mode='fan_out'`** in PyTorch's Kaiming init? When would you use each?

4. **Derive the He init formula for Leaky ReLU** with negative slope α. Show that it reduces to standard He when α=0.

5. **Compare the weight initialization scales** for layers with n_in=100 vs n_in=10000. Why do wider layers need smaller weight magnitudes?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Xavier/Glorot Init](./04-xavier-glorot-initialization.md) | [Unit 6: Weight Init](./README.md) | [LSUV →](./06-lsuv.md) |
