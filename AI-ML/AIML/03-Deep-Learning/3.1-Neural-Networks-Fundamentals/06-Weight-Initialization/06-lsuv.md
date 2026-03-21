# 6. LSUV — Layer-Sequential Unit-Variance Initialization

> **Unit 6 · Weight Initialization** — Data-driven initialization for reliable signal propagation

---

## Chapter Overview

Xavier and He initializations derive weight variance from **theoretical analysis** — they assume linear activations, independent weights, and specific distributions. In practice, real networks have skip connections, batch normalization, and complex architectures where these assumptions may not hold. **LSUV** (Mishkin & Matas, 2016) takes a different approach: it **empirically** adjusts weights layer-by-layer using actual data, ensuring each layer's output has **unit variance**. This data-driven approach works for any architecture without theoretical derivations.

---

## 1. Motivation: Limitations of Fixed Formulas

### When Xavier/He May Not Be Optimal

```
  ┌────────────────────────────────────────────────────────┐
  │  Xavier/He assume:                                     │
  │  ✗ Independent weights and inputs                      │
  │  ✗ No batch normalization                              │
  │  ✗ No skip connections                                 │
  │  ✗ No weight sharing                                   │
  │  ✗ Specific activation functions                       │
  │  ✗ All layers have same fan_in/fan_out pattern         │
  │                                                        │
  │  Real networks have:                                   │
  │  ✓ BatchNorm, LayerNorm                                │
  │  ✓ Skip/residual connections                           │
  │  ✓ Attention mechanisms                                │
  │  ✓ Mixed activation functions                          │
  │  ✓ Complex architectures (Inception, U-Net, etc.)      │
  └────────────────────────────────────────────────────────┘
```

### LSUV's Philosophy

Instead of deriving the right variance analytically, **measure** the actual output variance and **rescale** until it's 1.0:

```
  Theoretical approach:          Data-driven approach (LSUV):
  
  "What variance should          "What variance does this
   weights have in theory?"       layer actually produce?
                                   Let me fix it."
  
  σ² = 2/n_in (for ReLU)         σ²_actual = 0.37? → scale up!
                                   σ²_actual = 2.8?  → scale down!
```

---

## 2. The LSUV Algorithm

### Overview

```
  Step 1: Initialize weights with orthonormal matrices
  Step 2: For each layer (sequentially, from input to output):
      a. Forward pass a mini-batch through layers up to this layer
      b. Measure the variance of this layer's output
      c. Rescale weights: W = W / √(Var(output))
      d. Repeat b-c until Var(output) ≈ 1.0
```

### Detailed Algorithm

```
Input: Network f, mini-batch X, tolerance τ = 0.1, max iterations T = 10

1. Initialize all weight matrices with orthonormal initialization

2. For l = 1, 2, ..., L (each layer sequentially):
   
   For iteration = 1, 2, ..., T:
       
       a. Forward pass X through layers 1..l to get output a⁽ˡ⁾
       
       b. Compute: v = Var(a⁽ˡ⁾)
       
       c. If |v - 1.0| < τ:
              break  (variance is close enough to 1.0)
       
       d. Rescale: W⁽ˡ⁾ = W⁽ˡ⁾ / √v
       
   End for (iterations)
End for (layers)
```

### Why It Works

```
  Layer l output: a⁽ˡ⁾ = σ(W⁽ˡ⁾ · a⁽ˡ⁻¹⁾ + b⁽ˡ⁾)
  
  If Var(a⁽ˡ⁾) = v, and we scale W by 1/√v:
  
  New output: σ((W/√v) · a⁽ˡ⁻¹⁾ + b)
  
  For linear σ: Var(new output) = Var(old output) / v = v/v = 1.0 ✓
  
  For nonlinear σ: approximately 1.0 (may need a few iterations)
```

---

## 3. Step 1: Orthonormal Initialization

### What is Orthonormal Initialization?

An **orthonormal matrix** Q satisfies Q·Qᵀ = I (rows are orthogonal unit vectors):

```
  Properties:
  ✓ Preserves vector norms: ||Qx|| = ||x||
  ✓ Preserves angles between vectors
  ✓ No information loss in linear transformation
  ✓ All singular values equal 1
```

### How to Generate Orthonormal Matrices

```
  1. Generate random matrix A ~ N(0, 1)
  2. Compute QR decomposition: A = Q·R
  3. Use Q as the weight matrix
  
  For non-square matrices (n_out ≠ n_in):
  - Generate (max(n_out, n_in), max(n_out, n_in)) matrix
  - Take the appropriate submatrix of Q
```

### Why Start Orthonormal?

```
  Random Normal:                  Orthonormal:
  
  Singular values:                Singular values:
  σ ↑                             σ ↑
    │  ╱╲                           │ ─────────────
    │ ╱  ╲                          │ all = 1.0
    │╱    ╲                         │
    └──────→ index                  └──────→ index
    
  Spread of singular values       All singular values identical
  → different directions have     → all directions treated equally
    different scaling               → best starting point
```

---

## 4. Step 2: Layer-by-Layer Rescaling

### Visualization of the Process

```
  Initial state (after orthonormal init):
  
  Layer:     1        2        3        4        5
  Var(a):   [0.82]   [0.67]   [1.45]   [0.31]   [2.1]
                                                    ← not unit variance!
  
  After LSUV:
  
  Process Layer 1: Var = 0.82 → scale W by 1/√0.82 → Var ≈ 1.0 ✓
  Process Layer 2: Var = 0.71 → scale W by 1/√0.71 → Var ≈ 1.0 ✓
  Process Layer 3: Var = 1.12 → scale W by 1/√1.12 → Var ≈ 1.0 ✓
  Process Layer 4: Var = 0.93 → scale W by 1/√0.93 → Var ≈ 1.0 ✓
  Process Layer 5: Var = 1.05 → close enough (|1.05-1| < 0.1) ✓
  
  Final:
  Layer:     1        2        3        4        5
  Var(a):   [1.0]    [1.0]    [1.0]    [1.0]    [1.0]   ← all unit variance!
```

### Why Sequential (Not Parallel)?

Each layer's output depends on all previous layers. When we fix Layer 1's variance, it changes the input distribution for Layer 2. So we must process layers **in order**:

```
  ✗ Wrong: Fix all layers simultaneously
    Layer 1: Var(a₁) = 0.8 → scale → Var(a₁) ≈ 1.0
    Layer 2: Var(a₂) was computed with old a₁ → now WRONG
  
  ✓ Right: Fix layers sequentially
    Layer 1: scale → Var(a₁) ≈ 1.0
    Layer 2: recompute with new a₁ → scale → Var(a₂) ≈ 1.0
    Layer 3: recompute with new a₁, a₂ → scale → Var(a₃) ≈ 1.0
```

---

## 5. Worked Example

### Problem
3-layer ReLU network: 4 → 3 → 3 → 2, mini-batch of 8 samples.

### Step 1: Orthonormal Init

```
  W⁽¹⁾ ∈ ℝ³ˣ⁴ (orthonormal rows):
  [[ 0.50, -0.50,  0.50,  0.50],
   [-0.50,  0.50,  0.50, -0.50],
   [ 0.50,  0.50, -0.50,  0.50]]
```

### Step 2: LSUV for Layer 1

```
  Forward mini-batch X (8×4) through Layer 1:
  z = X @ W⁽¹⁾.T + b = X @ W⁽¹⁾.T
  a = ReLU(z)
  
  Suppose: Var(a) = 0.72
  
  Rescale: W⁽¹⁾ = W⁽¹⁾ / √0.72 = W⁽¹⁾ / 0.849
  
  Re-check: Forward again → Var(a) = 0.98  (close to 1.0, within τ=0.1) ✓
```

### Step 3: LSUV for Layer 2

```
  Forward mini-batch through Layer 1 (now fixed) → Layer 2:
  a⁽¹⁾ = ReLU(W_fixed⁽¹⁾ @ X.T)
  z⁽²⁾ = W⁽²⁾ @ a⁽¹⁾
  a⁽²⁾ = ReLU(z⁽²⁾)
  
  Suppose: Var(a⁽²⁾) = 1.34
  
  Rescale: W⁽²⁾ = W⁽²⁾ / √1.34
  
  Re-check: Var(a⁽²⁾) = 1.02 ✓
```

### Step 4: LSUV for Layer 3 (output)

```
  Forward through all layers to Layer 3:
  
  Suppose: Var(output) = 0.89
  
  Rescale: W⁽³⁾ = W⁽³⁾ / √0.89
  
  Re-check: Var(output) = 1.01 ✓
  
  Done! All layers now produce unit-variance outputs.
```

---

## 6. Python Implementation

### Complete LSUV Implementation

```python
import torch
import torch.nn as nn
import numpy as np

def orthonormal_init(module):
    """Initialize a layer with orthonormal weights."""
    if isinstance(module, (nn.Linear, nn.Conv2d)):
        nn.init.orthogonal_(module.weight)
        if module.bias is not None:
            nn.init.zeros_(module.bias)

def lsuv_init(model, data_batch, tol=0.1, max_iter=10, needed_std=1.0):
    """
    Layer-Sequential Unit-Variance (LSUV) initialization.
    
    Args:
        model: Neural network
        data_batch: Mini-batch of input data (e.g., 256 samples)
        tol: Tolerance for variance deviation from target
        max_iter: Maximum rescaling iterations per layer
        needed_std: Target standard deviation (default 1.0)
    """
    # Step 1: Orthonormal initialization
    model.apply(orthonormal_init)
    model.eval()
    
    # Find all layers with weights
    weight_layers = []
    for module in model.modules():
        if isinstance(module, (nn.Linear, nn.Conv2d)):
            weight_layers.append(module)
    
    # Step 2: Sequential rescaling
    with torch.no_grad():
        for i, layer in enumerate(weight_layers):
            # Hook to capture layer output
            activation = {}
            def hook_fn(module, input, output, name=i):
                activation[name] = output
            
            handle = layer.register_forward_hook(hook_fn)
            
            for iteration in range(max_iter):
                # Forward pass
                _ = model(data_batch)
                
                # Get this layer's output
                act = activation[i]
                
                # Compute std
                current_std = act.std().item()
                
                if abs(current_std - needed_std) < tol:
                    print(f"  Layer {i}: converged in {iteration+1} iters "
                          f"(std={current_std:.4f})")
                    break
                
                if current_std < 1e-8:
                    print(f"  Layer {i}: WARNING - std ≈ 0, skipping")
                    break
                
                # Rescale weights
                layer.weight.data /= (current_std / needed_std)
            else:
                print(f"  Layer {i}: max iters reached (std={current_std:.4f})")
            
            handle.remove()
    
    model.train()
    return model

# ── Usage ───────────────────────────────────────────────────────
torch.manual_seed(42)
model = nn.Sequential(
    nn.Linear(100, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 10)
)

# Generate a mini-batch for LSUV
data_batch = torch.randn(256, 100)

print("Running LSUV initialization:")
model = lsuv_init(model, data_batch)
```

### Verifying LSUV Results

```python
import matplotlib.pyplot as plt

def check_layer_stats(model, x):
    """Check activation statistics at each layer."""
    stats = []
    h = x
    model.eval()
    with torch.no_grad():
        for layer in model:
            h = layer(h)
            if isinstance(layer, nn.ReLU):
                stats.append({
                    'mean': h.mean().item(),
                    'std': h.std().item(),
                    'dead_pct': (h == 0).float().mean().item() * 100
                })
    return stats

# Compare: LSUV vs He vs Xavier
inits = {}

# LSUV
torch.manual_seed(0)
model_lsuv = nn.Sequential(
    nn.Linear(100, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
)
model_lsuv = lsuv_init(model_lsuv, torch.randn(256, 100))
inits['LSUV'] = check_layer_stats(model_lsuv, torch.randn(1000, 100))

# He
torch.manual_seed(0)
model_he = nn.Sequential(
    nn.Linear(100, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(),
)
for m in model_he.modules():
    if isinstance(m, nn.Linear):
        nn.init.kaiming_normal_(m.weight, nonlinearity='relu')
        nn.init.zeros_(m.bias)
inits['He'] = check_layer_stats(model_he, torch.randn(1000, 100))

# Plot
fig, ax = plt.subplots(figsize=(8, 5))
for name, stats in inits.items():
    stds = [s['std'] for s in stats]
    ax.plot(stds, 'o-', label=name, linewidth=2, markersize=8)

ax.axhline(y=1.0, color='gray', linestyle='--', alpha=0.5, label='Target std=1')
ax.set_xlabel('Layer (after ReLU)')
ax.set_ylabel('Activation Std')
ax.set_title('LSUV vs He: Activation Standard Deviation per Layer')
ax.legend()
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig("lsuv_vs_he.png", dpi=150)
plt.show()
```

### Training Comparison

```python
from torch.utils.data import DataLoader, TensorDataset

torch.manual_seed(42)
X = torch.randn(3000, 100)
y = (X @ torch.randn(100, 1) + torch.randn(3000, 1) * 0.3).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

def train_model(model, epochs=40):
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

# LSUV model
torch.manual_seed(0)
model_lsuv = nn.Sequential(
    nn.Linear(100, 256), nn.ReLU(), nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(), nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 1)
)
model_lsuv = lsuv_init(model_lsuv, X[:256])
lsuv_losses = train_model(model_lsuv)

# He model
torch.manual_seed(0)
model_he = nn.Sequential(
    nn.Linear(100, 256), nn.ReLU(), nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 256), nn.ReLU(), nn.Linear(256, 256), nn.ReLU(),
    nn.Linear(256, 1)
)
for m in model_he.modules():
    if isinstance(m, nn.Linear):
        nn.init.kaiming_normal_(m.weight, nonlinearity='relu')
        nn.init.zeros_(m.bias)
he_losses = train_model(model_he)

print(f"LSUV final loss: {lsuv_losses[-1]:.4f}")
print(f"He final loss:   {he_losses[-1]:.4f}")
```

---

## 7. Advantages and Limitations

### Advantages

| Advantage | Explanation |
|---|---|
| **Architecture-agnostic** | Works for any network, no derivation needed |
| **Data-driven** | Uses actual data statistics, not assumptions |
| **Handles complex architectures** | Skip connections, mixed activations, etc. |
| **Simple implementation** | Just forward passes + weight scaling |
| **Reliable** | Almost always achieves unit variance |

### Limitations

| Limitation | Explanation |
|---|---|
| **Requires data** | Need a mini-batch before training starts |
| **Sequential processing** | Can't parallelize across layers |
| **Overhead** | Several forward passes before training |
| **Not widely adopted** | Most practitioners use He + BatchNorm instead |
| **BatchNorm overlap** | If using BatchNorm, LSUV is less necessary |

### When to Use LSUV

```
  Use LSUV when:
  ✓ Novel/unusual architecture where Xavier/He may not apply
  ✓ No batch normalization
  ✓ Very deep networks (50+ layers) without residual connections
  ✓ Mixed activation functions within the same network
  ✓ You want a principled, architecture-agnostic init
  
  Don't bother with LSUV when:
  ✗ Standard architectures (ResNet, VGG, etc.) — He works fine
  ✗ Using BatchNorm (it normalizes activations anyway)
  ✗ Transfer learning (using pretrained weights)
  ✗ Shallow networks (< 5 layers)
```

---

## 8. LSUV vs Other Methods

```
  Method Comparison for 15-layer ReLU network (no BatchNorm):
  
  Activation Std ↑
  
  10.0  │                                    · Random (σ=0.1)
        │                               · ·
   5.0  │                          · · ·
        │
   1.0  │── ── ── ── ── ── ── ── ── ── ── ── ── LSUV (target)
        │ ── ──
   0.5  │      ── ──
        │           ── ── He
   0.1  │                ── ──
        │                     ── ──
  0.01  │  ·  ·  ·  ·  ·  ·       ── Xavier
        │
        └──────────────────────────────────────→ Layer
         1    3    5    7    9   11   13   15
```

| Method | Approach | Unit Variance? | Architecture-Agnostic? |
|---|---|---|---|
| Xavier | Analytical formula | ≈ for tanh | Partially |
| He | Analytical formula | ≈ for ReLU | Partially |
| **LSUV** | **Data-driven** | **Yes (by construction)** | **Yes** |
| Fixup | Architecture-specific residual scaling | ≈ for ResNets | No |
| ZerO | Data-free orthogonal | Approximately | Yes |

---

## Summary Table

| Concept | Key Idea |
|---|---|
| Full name | Layer-Sequential Unit-Variance |
| Approach | Data-driven (not formula-based) |
| Step 1 | Orthonormal initialization (QR decomposition) |
| Step 2 | Forward mini-batch, measure variance, rescale |
| Processing order | Sequential (layer by layer, input to output) |
| Rescaling | W = W / √(Var(output)) per layer |
| Target | Var(output) = 1.0 at every layer |
| Tolerance | Typically τ = 0.1 |
| Max iterations | Typically 10 per layer |
| Best for | Complex architectures without BatchNorm |

---

## Revision Questions

1. **Describe the complete LSUV algorithm step by step.** Why must layers be processed sequentially and not in parallel?

2. **Why does LSUV start with orthonormal initialization** rather than random normal? What properties of orthonormal matrices are beneficial?

3. **Show that rescaling W by 1/√v changes the output variance** from v to approximately 1.0. Under what conditions is this exact?

4. **Compare LSUV with He initialization for a 15-layer ReLU network.** When would LSUV provide a meaningful advantage over He?

5. **Implement LSUV for a convolutional neural network** (hint: variance is computed over all spatial locations and channels).

6. **Why is LSUV less necessary when using Batch Normalization?** What does BatchNorm do that overlaps with LSUV's purpose?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← He/Kaiming Init](./05-he-initialization.md) | [Unit 6: Weight Init](./README.md) | [Unit 7: Regularization →](../07-Regularization-Techniques/01-l1-l2-regularization.md) |
