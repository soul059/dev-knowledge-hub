# 1. Why Initialization Matters

> **Unit 6 · Weight Initialization** — How starting conditions determine training success or failure

---

## Chapter Overview

Weight initialization is the **first decision** that shapes the entire training trajectory of a neural network. Before any data is processed, before any gradient is computed, the initial values of weights determine:
- Whether gradients **flow** or **vanish/explode** through the network
- Whether the network can **break symmetry** and learn diverse features
- How **fast** training converges (or whether it converges at all)
- Which **region of the loss landscape** the optimizer begins exploring

A poor initialization can render a perfectly designed architecture untrainable. Understanding initialization is fundamental to deep learning practice.

---

## 1. What Gets Initialized?

### Trainable Parameters

```
  Neural Network Layer:
  
  Input x ──→ [ W·x + b ] ──→ activation ──→ output
                ↑     ↑
              weights  bias
              (W)      (b)
              
  Both W and b need initial values before training begins.
```

| Parameter | Shape | Initialization | Notes |
|---|---|---|---|
| **Weights W** | (n_out, n_in) | **Critical** — must break symmetry | Various strategies |
| **Biases b** | (n_out,) | Usually zeros | Safe default |
| **BatchNorm γ** | (n_features,) | Ones | Scale parameter |
| **BatchNorm β** | (n_features,) | Zeros | Shift parameter |

---

## 2. How Initialization Affects Training

### Effect 1: Convergence Speed

```
  Loss ↑
       │╲
       │ ╲ ·  ·  ·  ·                     · · · Bad initialization
       │  ╲·                               ───── Good initialization
       │   ──╲ ·  ·  ·  ·  ·
       │      ──╲·  ·  ·  ·  ·  ·
       │         ──╲──·──·──·──·──·──·──
       │              ────────────
       └────────────────────────────────→ Epoch
        0     20     40     60     80
        
  Good init: converges in ~30 epochs
  Bad init:  takes 60+ epochs (if it converges at all)
```

### Effect 2: Final Performance

Different initializations land in **different regions** of the loss landscape:

```
  Loss Landscape:
  
  Loss ↑
       │     ╱╲          ╱╲
       │    ╱  ╲    ╱╲  ╱  ╲
       │   ╱    ╲  ╱  ╲╱    ╲
       │  ╱  ●A  ╲╱  ●B      ╲
       │ ╱        ★            ╲     ★ = global minimum
       │╱                       ╲    ●A = init A lands here (ok)
       │                              ●B = init B lands here (better)
       └──────────────────────────→ θ
       
  Init A → converges to suboptimal minimum
  Init B → converges to better minimum
```

### Effect 3: Gradient Flow

```
  10-layer network with different init scales:

  Too Small Weights (σ = 0.001):
  Layer:  1    2    3    4    5    6    7    8    9    10
  Signal: ██  ▓▓  ░░   ·    ·    ·    ·    ·    ·    ·     ← VANISHES
  
  Just Right (Xavier/He):
  Layer:  1    2    3    4    5    6    7    8    9    10
  Signal: ██  ██  ██  ██  ██  ██  ██  ██  ██  ██           ← STABLE ✓
  
  Too Large Weights (σ = 2.0):
  Layer:  1    2    3    4    5    6    7    8    9    10
  Signal: ██  ███ ████ █████ ██████ → → → ∞               ← EXPLODES
```

---

## 3. The Signal Propagation Problem

### Forward Pass Variance Analysis

For a single layer: **z = Wx + b** where x has n_in elements:

```
  z_j = Σᵢ w_ji · x_i + b_j
```

If weights and inputs are independent with zero mean:

```
  Var(z_j) = n_in · Var(w) · Var(x) + Var(b)
```

For the signal to neither grow nor shrink across layers:

```
  Var(z) = Var(x)   →   n_in · Var(w) = 1   →   Var(w) = 1/n_in
```

### Backward Pass Variance Analysis

Similarly, for gradients flowing backward:

```
  Var(∂L/∂x) = n_out · Var(w) · Var(∂L/∂z)
```

For gradients to neither grow nor shrink:

```
  n_out · Var(w) = 1   →   Var(w) = 1/n_out
```

### The Compromise

We can't satisfy both n_in·Var(w)=1 and n_out·Var(w)=1 simultaneously (unless n_in=n_out). This leads to the Xavier/Glorot compromise:

```
  Var(w) = 2 / (n_in + n_out)    ← harmonic mean compromise
```

---

## 4. Symmetry Breaking

### Why Identical Initialization Fails

If all weights in a layer are identical:

```
  w₁ = w₂ = w₃ = ... = wₙ = c
  
  Then all neurons compute the SAME output:
  z₁ = z₂ = z₃ = ... = zₙ = c·Σx_i + b
  
  And all receive the SAME gradient:
  ∂L/∂w₁ = ∂L/∂w₂ = ∂L/∂w₃ = ... = ∂L/∂wₙ
  
  After update: w₁ = w₂ = w₃ = ... = wₙ  (still identical!)
  
  The symmetry is NEVER broken. The network has effectively 1 neuron per layer.
```

This is explored in detail in the next chapter.

---

## 5. Loss Landscape Starting Point

### Initialization Determines the Exploration Region

```
  2D Loss Landscape (bird's eye view):
  
    ╭──────────────────────────────────╮
    │     ★                            │
    │    ╱ ╲     Basin A               │
    │   ╱   ╲                          │
    │──╱     ╲──────────╮              │
    │                    │   Basin B    │
    │          ●init1    │    ★★       │
    │                    │   ╱  ╲      │
    │                    │──╱    ╲──   │
    │                    │             │
    │     ●init2         │             │
    │                    │  ●init3     │
    ╰──────────────────────────────────╯
    
  init1 → converges to Basin A (local min)
  init2 → converges to Basin A (global min ★)
  init3 → converges to Basin B (another local min ★★)
```

The optimizer follows the **local gradient** from wherever initialization places it. The initialization determines which **basin of attraction** the training falls into.

---

## 6. Visualizing Different Initializations

### Training Curves with Different Init Strategies

```
  Loss ↑
  5.0  │ · ·  ·  · · · · · · · · · · ·    ← All zeros (doesn't learn)
       │ ╲
  3.0  │  ╲ ╲
       │   ╲  ╲
  2.0  │    ╲  · · · · · · · · ·           ← Too large σ (unstable)
       │     ╲   ╲
  1.0  │      ╲    ╲╲
       │       ╲     ╲╲──                  ← Xavier (good)
  0.5  │        ╲       ──────             ← He (good for ReLU)
       │         ╲╲
  0.1  │           ╲╲──────────            ← Optimal for this net
       └──────────────────────────────→ Epoch
        0     10    20    30    40
```

---

## 7. Python Implementation: Observing the Effect

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset
import matplotlib.pyplot as plt

# ── Data ────────────────────────────────────────────────────────
torch.manual_seed(42)
X = torch.randn(2000, 20)
y = (X @ torch.randn(20, 1) + torch.randn(2000, 1) * 0.3).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

# ── Different initialization strategies ────────────────────────
def make_model_with_init(init_fn, depth=5, width=128):
    """Create a deep network with a specific initialization."""
    layers = []
    in_dim = 20
    for _ in range(depth):
        linear = nn.Linear(in_dim, width)
        init_fn(linear.weight)
        nn.init.zeros_(linear.bias)
        layers.extend([linear, nn.ReLU()])
        in_dim = width
    layers.append(nn.Linear(width, 1))
    return nn.Sequential(*layers)

init_strategies = {
    "Zeros":       lambda w: nn.init.zeros_(w),
    "Too Small":   lambda w: nn.init.normal_(w, 0, 0.001),
    "Too Large":   lambda w: nn.init.normal_(w, 0, 2.0),
    "Xavier":      lambda w: nn.init.xavier_normal_(w),
    "He/Kaiming":  lambda w: nn.init.kaiming_normal_(w, nonlinearity='relu'),
}

# ── Train and compare ──────────────────────────────────────────
results = {}
for name, init_fn in init_strategies.items():
    torch.manual_seed(0)
    try:
        model = make_model_with_init(init_fn)
        optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
        criterion = nn.MSELoss()
        
        losses = []
        for epoch in range(40):
            total = 0
            for xb, yb in loader:
                pred = model(xb).squeeze()
                loss = criterion(pred, yb)
                if torch.isnan(loss):
                    losses.append(float('nan'))
                    break
                optimizer.zero_grad(); loss.backward(); optimizer.step()
                total += loss.item() * len(xb)
            else:
                losses.append(total / len(X))
                continue
            break
        
        results[name] = losses
        print(f"{name:>12s}: Final loss = {losses[-1]:.4f}" 
              if not (losses[-1] != losses[-1]) else f"{name:>12s}: NaN!")
    except Exception as e:
        print(f"{name:>12s}: Failed ({e})")

# ── Visualize ──────────────────────────────────────────────────
fig, ax = plt.subplots(figsize=(10, 6))
for name, losses in results.items():
    ax.plot(losses, label=name, linewidth=2)
ax.set_xlabel("Epoch"); ax.set_ylabel("Loss"); ax.set_yscale("log")
ax.set_title("Effect of Weight Initialization on Training")
ax.legend(); ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig("init_comparison.png", dpi=150)
plt.show()
```

### Monitoring Activation Statistics

```python
def check_activations(model, x, label=""):
    """Monitor activation statistics layer by layer."""
    print(f"\n{'='*60}")
    print(f"Activation statistics: {label}")
    print(f"{'Layer':<15} {'Mean':>10} {'Std':>10} {'% Dead':>10}")
    print(f"{'='*60}")
    
    h = x
    for i, layer in enumerate(model):
        h = layer(h)
        if isinstance(layer, nn.ReLU):
            dead_pct = (h == 0).float().mean().item() * 100
            print(f"After ReLU {i//2:<4} {h.mean():>10.4f} {h.std():>10.4f} {dead_pct:>9.1f}%")
    
    return h

x_sample = X[:256]
for name, init_fn in [("He", nn.init.kaiming_normal_), 
                       ("Too Small", lambda w: nn.init.normal_(w, 0, 0.001))]:
    torch.manual_seed(0)
    model = make_model_with_init(init_fn)
    check_activations(model, x_sample, label=name)
```

---

## 8. Applications and Practical Impact

| Scenario | Initialization Impact |
|---|---|
| **Deep CNNs (50+ layers)** | Critical — wrong init → no training |
| **Transformers** | Important — affects warmup phase stability |
| **Transfer learning** | Less critical — pretrained weights are already good |
| **Small networks (2–3 layers)** | Less sensitive — most inits work |
| **ResNets with skip connections** | Skip connections reduce sensitivity |
| **RNNs/LSTMs** | Very important — affects long-term memory |

---

## Summary Table

| Concept | Key Idea |
|---|---|
| Why it matters | Initialization determines convergence, speed, and final quality |
| Signal propagation | Var(w) must be calibrated to maintain signal across layers |
| Symmetry breaking | Identical weights → identical neurons → no learning |
| Too small init | Signals/gradients vanish → dead network |
| Too large init | Signals/gradients explode → NaN/instability |
| Loss landscape | Init determines starting basin of attraction |
| Bias init | Usually zeros (safe default) |
| Goal | Var(output) ≈ Var(input) at every layer |

---

## Revision Questions

1. **Why can't we just initialize all weights to the same constant value?** What would happen during training?

2. **Derive the variance condition for stable signal propagation** through a linear layer z = Wx. What must Var(w) be?

3. **Explain the conflict between forward and backward variance conditions.** Why can't both be perfectly satisfied simultaneously?

4. **How does initialization affect the loss landscape exploration?** Can two different initializations lead to different final models?

5. **Run the activation monitoring code for a 10-layer network** with σ=0.01 vs σ=1.0 initialization. What do you observe about the mean and std at each layer?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Unit 5: Warm Restarts](../05-Optimization-Algorithms/08-warm-restarts.md) | [Unit 6: Weight Init](./README.md) | [Zero Initialization Problem →](./02-zero-initialization-problem.md) |
