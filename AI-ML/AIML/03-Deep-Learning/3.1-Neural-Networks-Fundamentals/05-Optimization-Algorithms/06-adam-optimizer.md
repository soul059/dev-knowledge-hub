# 6. Adam Optimizer — Adaptive Moment Estimation

> **Unit 5 · Optimization Algorithms** — The best of Momentum + RMSProp, and the modern default

---

## Chapter Overview

**Adam** (Kingma & Ba, 2015) combines the two most successful ideas in optimization:
1. **Momentum** — exponential moving average of gradients (first moment) for direction smoothing
2. **RMSProp** — exponential moving average of squared gradients (second moment) for per-parameter adaptation

Adam adds a crucial innovation: **bias correction** to account for the zero-initialization of the moment estimates. The result is an optimizer that works well across a wide range of architectures and hyperparameter settings, making it the **default choice** for most deep learning tasks.

---

## 1. Building Blocks

### What Each Component Provides

```
  ┌─────────────┐     ┌──────────────┐
  │  Momentum   │     │   RMSProp    │
  │             │     │              │
  │ Smooths     │     │ Adapts lr    │
  │ gradient    │     │ per param    │
  │ direction   │     │ based on     │
  │ (1st moment)│     │ gradient     │
  │             │     │ scale        │
  │ mₜ = EMA   │     │ (2nd moment) │
  │ of gₜ      │     │              │
  └──────┬──────┘     │ vₜ = EMA    │
         │            │ of gₜ²      │
         │            └──────┬───────┘
         │                   │
         └───────┬───────────┘
                 │
         ┌───────▼────────┐
         │     Adam       │
         │                │
         │ m̂ₜ/(√v̂ₜ + ε) │
         │                │
         │ Direction from │
         │ momentum,      │
         │ step size from │
         │ RMSProp        │
         └────────────────┘
```

---

## 2. The Adam Algorithm

### Complete Algorithm

```
Initialize:
    m₀ = 0    (first moment estimate — mean of gradients)
    v₀ = 0    (second moment estimate — mean of squared gradients)
    β₁ = 0.9  (decay rate for first moment)
    β₂ = 0.999 (decay rate for second moment)
    ε = 1e-8   (numerical stability)
    α = 0.001  (learning rate)
    t = 0      (time step counter)

For each step:
    t = t + 1
    
    1. Compute gradient:     gₜ = ∇θ J(θₜ)
    
    2. Update first moment:  mₜ = β₁ · mₜ₋₁ + (1 - β₁) · gₜ
    
    3. Update second moment: vₜ = β₂ · vₜ₋₁ + (1 - β₂) · gₜ²
    
    4. Bias correction:      m̂ₜ = mₜ / (1 - β₁ᵗ)
                             v̂ₜ = vₜ / (1 - β₂ᵗ)
    
    5. Update parameters:    θₜ₊₁ = θₜ - α · m̂ₜ / (√v̂ₜ + ε)
```

### Per-Parameter View

For each parameter θⱼ:

```
  mₜ,ⱼ = β₁ · mₜ₋₁,ⱼ + (1 - β₁) · gₜ,ⱼ        ← EMA of gradient
  vₜ,ⱼ = β₂ · vₜ₋₁,ⱼ + (1 - β₂) · gₜ,ⱼ²        ← EMA of squared gradient
  
  m̂ₜ,ⱼ = mₜ,ⱼ / (1 - β₁ᵗ)                       ← bias-corrected mean
  v̂ₜ,ⱼ = vₜ,ⱼ / (1 - β₂ᵗ)                       ← bias-corrected variance
  
  θₜ₊₁,ⱼ = θₜ,ⱼ - α · m̂ₜ,ⱼ / (√v̂ₜ,ⱼ + ε)      ← adaptive update
```

---

## 3. Understanding First and Second Moments

### First Moment mₜ (Mean of Gradients)

```
  mₜ = β₁ · mₜ₋₁ + (1 - β₁) · gₜ

  This is the EWMA of gradients — exactly what Momentum computes.
  
  Interpretation: mₜ ≈ E[gₜ]  (approximate mean gradient)
  
  Role: Provides the UPDATE DIRECTION
        Smooths out noisy gradients
        Dampens oscillations
```

### Second Moment vₜ (Uncentered Variance)

```
  vₜ = β₂ · vₜ₋₁ + (1 - β₂) · gₜ²

  This is the EWMA of squared gradients — exactly what RMSProp computes.
  
  Interpretation: vₜ ≈ E[gₜ²]  (approximate uncentered second moment)
  
  Note: E[gₜ²] = Var(gₜ) + E[gₜ]²  (uncentered = variance + mean²)
  
  Role: Provides the STEP SIZE scaling
        Parameters with large gradient variance get smaller steps
```

### Why "Uncentered Variance"?

```
  Centered variance:    Var(g) = E[(g - E[g])²] = E[g²] - E[g]²
  Uncentered variance:  E[g²]
  
  Adam uses E[g²] (uncentered) because:
  1. Simpler to compute (no need to subtract mean²)
  2. Includes the mean term, providing a better scale estimate
  3. Works well in practice
```

---

## 4. Bias Correction

### The Problem

Both moments are initialized to zero: m₀ = 0, v₀ = 0. In early steps:

```
  t=1: m₁ = β₁·0 + (1-β₁)·g₁ = (1-β₁)·g₁ = 0.1·g₁
  
  E[m₁] = (1-β₁)·E[g₁] = 0.1·E[g]
  
  But we want E[m₁] ≈ E[g], not 0.1·E[g]!
```

The estimates are **biased toward zero** in early steps.

### The Fix: Divide by (1 - βᵗ)

At step t, the bias in mₜ is:

```
  E[mₜ] = (1-β₁ᵗ) · E[g]
  
  Therefore: m̂ₜ = mₜ / (1-β₁ᵗ) → E[m̂ₜ] = E[g]  ✓ unbiased!
```

### Derivation

Expanding mₜ:
```
  mₜ = (1-β₁) · [gₜ + β₁·gₜ₋₁ + β₁²·gₜ₋₂ + ... + β₁ᵗ⁻¹·g₁]
  
  If all gₛ have the same expected value E[g]:
  E[mₜ] = (1-β₁) · E[g] · [1 + β₁ + β₁² + ... + β₁ᵗ⁻¹]
         = (1-β₁) · E[g] · (1 - β₁ᵗ)/(1 - β₁)
         = (1 - β₁ᵗ) · E[g]
  
  Dividing by (1 - β₁ᵗ):
  E[m̂ₜ] = E[mₜ] / (1-β₁ᵗ) = E[g]  ✓
```

### Bias Correction Factor Over Time

```
  Correction factor 1/(1 - βᵗ) for β₁ = 0.9:
  
  Factor ↑
  10.0 │ ■
       │ ■
   5.0 │ ■ ■
       │     ■
   2.0 │       ■ ■
       │           ■ ■ ■
   1.0 │                   ■ ■ ■ ■ ■ ■ ■ ■ ■  ← converges to 1
       └──────────────────────────────────────→ step t
        1  2  3  4  5  6  7  8  9  10  15  20
  
  For β₂ = 0.999 (slower decay):
  Factor at t=1:   1/(1-0.999) = 1000×!
  Factor at t=10:  ~100×
  Factor at t=100: ~10×
  Factor at t=1000: ~1.6×
  Factor at t=5000: ~1.0×
```

---

## 5. Understanding the Update Rule

### Decomposing the Update

```
  θ -= α · m̂ₜ / (√v̂ₜ + ε)
       ↑   ↑        ↑
       │   │        └── Adaptive denominator (from RMSProp)
       │   └── Smoothed direction (from Momentum)
       └── Global learning rate
```

### Effective Step Size Analysis

When gradient is constant g for all steps (and bias correction has converged):

```
  m̂ₜ → g          (first moment converges to gradient)
  v̂ₜ → g²         (second moment converges to gradient²)
  
  Update: α · g / (√g² + ε) = α · g / (|g| + ε) ≈ α · sign(g)
```

**Key insight**: The step size is approximately α regardless of gradient magnitude! Adam effectively **normalizes** the gradient, making the update magnitude roughly **α** per step.

```
  Step Size Comparison:
  
  Parameter    |gradient|    SGD step       Adam step
  ──────────────────────────────────────────────────────
  θ₁           100          100α = 10      ≈ α = 0.001
  θ₂           0.01         0.01α          ≈ α = 0.001
  θ₃           1.0          1.0α = 0.1     ≈ α = 0.001
  
  Adam equalizes step sizes across parameters!
```

---

## 6. Default Hyperparameters

### Recommended Values (Kingma & Ba, 2015)

| Parameter | Default | Role |
|---|---|---|
| **α** (lr) | 0.001 | Global learning rate |
| **β₁** | 0.9 | First moment decay (gradient memory ~10 steps) |
| **β₂** | 0.999 | Second moment decay (gradient² memory ~1000 steps) |
| **ε** | 1e-8 | Numerical stability |

### Why These Values Work

```
  β₁ = 0.9: Averages ~10 recent gradients for direction
  β₂ = 0.999: Averages ~1000 recent squared gradients for scale
  
  β₂ >> β₁ because:
  - Gradient direction can change quickly → short memory
  - Gradient magnitude/scale is more stable → long memory
```

### Tuning Guidelines

| Parameter | If too small | If too large |
|---|---|---|
| α | Slow convergence | Divergence, instability |
| β₁ | Noisy updates (like no momentum) | Sluggish (ignores recent gradients) |
| β₂ | Step sizes change too fast | Step sizes too sticky (slow adaptation) |
| ε | Numerical issues with small v̂ₜ | Step sizes too uniform (less adaptive) |

---

## 7. AdamW: Decoupled Weight Decay

### The Problem with L2 Regularization in Adam

Standard L2 regularization adds λ||θ||² to the loss:

```
  Modified gradient: gₜ = ∇J(θₜ) + 2λθₜ
```

In SGD, this is equivalent to weight decay:
```
  θ -= α·(g + 2λθ) = α·g + α·2λ·θ  ← weight decay = α·2λ
```

**But in Adam**, L2 regularization ≠ weight decay because Adam **normalizes** the gradient:

```
  Adam with L2: The regularization gradient 2λθ also gets divided by √v̂ₜ
  → The decay is scaled differently for each parameter
  → Not the same as multiplying θ by (1-decay) each step!
```

### AdamW: The Fix (Loshchilov & Hutter, 2019)

AdamW **decouples** weight decay from the gradient-based update:

```
  Standard Adam with L2:                AdamW:
  gₜ = ∇J(θ) + 2λθ                    gₜ = ∇J(θ)
  mₜ = β₁·m + (1-β₁)·gₜ              mₜ = β₁·m + (1-β₁)·gₜ
  vₜ = β₂·v + (1-β₂)·gₜ²             vₜ = β₂·v + (1-β₂)·gₜ²
  θ -= α · m̂/(√v̂+ε)                  θ -= α · m̂/(√v̂+ε) + α·λ·θ
                                                               ↑
                                        Weight decay applied DIRECTLY to θ,
                                        not through the adaptive mechanism
```

### Why AdamW is Better

```
  ┌────────────────────────────────────────────────────────┐
  │  In Adam + L2:  weight decay is entangled with the    │
  │  adaptive learning rate → different parameters get    │
  │  different effective regularization strengths          │
  │                                                        │
  │  In AdamW: weight decay is clean and uniform:         │
  │  θⱼ *= (1 - α·λ) for ALL parameters equally          │
  └────────────────────────────────────────────────────────┘
```

---

## 8. Why Adam is the Default

### Advantages

| Property | Benefit |
|---|---|
| Adaptive learning rates | Handles different parameter scales automatically |
| Momentum | Smooths gradient noise |
| Bias correction | Works well from the first step |
| Robust defaults | α=0.001, β₁=0.9, β₂=0.999 work across many tasks |
| Low memory | Only 2× parameter storage (m and v) |
| Convergence speed | Often fastest in wall-clock time |

### When Adam May Not Be Optimal

| Scenario | Issue | Alternative |
|---|---|---|
| Computer vision (ResNets) | SGD+momentum can generalize better | SGD + cosine schedule |
| Very large models | Memory for states | Adafactor, 8-bit Adam |
| Theory-driven needs | Adam may not converge in some convex cases | AMSGrad |
| Fine-tuning | Adam may overfit faster | SGD with small lr |

---

## 9. Worked Example

### Problem
Minimize f(θ₁, θ₂) = 10θ₁² + θ₂² with Adam, starting at θ = (5, 5).

### Solution
Using α=0.001, β₁=0.9, β₂=0.999, ε=1e-8.

**Step 1:** t=1, θ = (5, 5), m = (0,0), v = (0,0)
```
g = (100, 10)

m₁ = 0.9·(0,0) + 0.1·(100,10) = (10, 1)
v₁ = 0.999·(0,0) + 0.001·(10000,100) = (10, 0.1)

Bias correction:
m̂₁ = (10, 1) / (1 - 0.9¹) = (10, 1) / 0.1 = (100, 10)
v̂₁ = (10, 0.1) / (1 - 0.999¹) = (10, 0.1) / 0.001 = (10000, 100)

Update:
θ₂ = (5, 5) - 0.001 · (100, 10) / (√(10000, 100) + 1e-8)
   = (5, 5) - 0.001 · (100, 10) / (100, 10)
   = (5, 5) - 0.001 · (1, 1)
   = (4.999, 4.999)
```

Notice: After bias correction, the step is approximately **α = 0.001** in each direction, regardless of gradient magnitude! This is the normalization effect.

**Step 2:** t=2, θ = (4.999, 4.999)
```
g = (99.98, 9.998)

m₂ = 0.9·(10,1) + 0.1·(99.98, 9.998) = (18.998, 1.9998)
v₂ = 0.999·(10, 0.1) + 0.001·(9996, 99.96) = (19.986, 0.1999)

m̂₂ = (18.998, 1.9998) / (1 - 0.81) = (99.99, 10.525)
v̂₂ = (19.986, 0.1999) / (1 - 0.998) = (9993, 99.95)

θ₃ = (4.999, 4.999) - 0.001 · (99.99, 10.525) / (√(9993, 99.95) + 1e-8)
   ≈ (4.999, 4.999) - 0.001 · (1.0, 1.053)
   ≈ (4.998, 4.998)
```

Consistent ~0.001 step per iteration in each direction.

---

## 10. Python Implementation (PyTorch)

### Using PyTorch's Adam and AdamW

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

# ── Data ────────────────────────────────────────────────────────
torch.manual_seed(42)
X = torch.randn(5000, 50)
y = (X @ torch.randn(50, 1) + 0.5 * torch.randn(5000, 1)).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=128, shuffle=True)

# ── Model ───────────────────────────────────────────────────────
def make_model():
    torch.manual_seed(0)
    return nn.Sequential(
        nn.Linear(50, 256), nn.ReLU(),
        nn.Linear(256, 128), nn.ReLU(),
        nn.Linear(128, 1)
    )

# ── Training function ──────────────────────────────────────────
def train(model, optimizer, epochs=50):
    criterion = nn.MSELoss()
    history = []
    for epoch in range(epochs):
        total = 0
        for xb, yb in loader:
            loss = criterion(model(xb).squeeze(), yb)
            optimizer.zero_grad(); loss.backward(); optimizer.step()
            total += loss.item() * len(xb)
        history.append(total / len(X))
    return history

# ── Compare Optimizers ──────────────────────────────────────────
optimizers = {
    "SGD":           (torch.optim.SGD,    {"lr": 0.01, "momentum": 0.9}),
    "RMSProp":       (torch.optim.RMSprop, {"lr": 0.001}),
    "Adam":          (torch.optim.Adam,    {"lr": 0.001}),
    "AdamW":         (torch.optim.AdamW,   {"lr": 0.001, "weight_decay": 0.01}),
}

results = {}
for name, (cls, kwargs) in optimizers.items():
    model = make_model()
    opt = cls(model.parameters(), **kwargs)
    results[name] = train(model, opt)
    print(f"{name:>10s}: Final loss = {results[name][-1]:.4f}")
```

### Manual Adam Implementation

```python
class ManualAdam:
    """Adam optimizer implemented from scratch."""
    
    def __init__(self, params, lr=0.001, betas=(0.9, 0.999), eps=1e-8):
        self.params = list(params)
        self.lr = lr
        self.beta1, self.beta2 = betas
        self.eps = eps
        self.t = 0
        
        self.m = [torch.zeros_like(p) for p in self.params]  # first moment
        self.v = [torch.zeros_like(p) for p in self.params]  # second moment
    
    def zero_grad(self):
        for p in self.params:
            if p.grad is not None:
                p.grad.zero_()
    
    def step(self):
        self.t += 1
        
        for i, p in enumerate(self.params):
            if p.grad is None:
                continue
            g = p.grad.data
            
            # Update biased first moment estimate
            self.m[i] = self.beta1 * self.m[i] + (1 - self.beta1) * g
            
            # Update biased second moment estimate
            self.v[i] = self.beta2 * self.v[i] + (1 - self.beta2) * g * g
            
            # Bias correction
            m_hat = self.m[i] / (1 - self.beta1 ** self.t)
            v_hat = self.v[i] / (1 - self.beta2 ** self.t)
            
            # Update parameters
            p.data -= self.lr * m_hat / (torch.sqrt(v_hat) + self.eps)

# Verify against PyTorch
model1 = nn.Linear(10, 5); model2 = nn.Linear(10, 5)
model2.load_state_dict(model1.state_dict())

opt1 = torch.optim.Adam(model1.parameters(), lr=0.001)
opt2 = ManualAdam(model2.parameters(), lr=0.001)

x = torch.randn(16, 10)
for _ in range(50):
    loss1 = model1(x).sum(); opt1.zero_grad(); loss1.backward(); opt1.step()
    loss2 = model2(x).sum(); opt2.zero_grad(); loss2.backward(); opt2.step()

print(f"Max diff: {(model1.weight.data - model2.weight.data).abs().max():.1e}")
```

### AdamW Implementation

```python
class ManualAdamW:
    """AdamW with decoupled weight decay."""
    
    def __init__(self, params, lr=0.001, betas=(0.9, 0.999), eps=1e-8,
                 weight_decay=0.01):
        self.params = list(params)
        self.lr = lr
        self.beta1, self.beta2 = betas
        self.eps = eps
        self.wd = weight_decay
        self.t = 0
        self.m = [torch.zeros_like(p) for p in self.params]
        self.v = [torch.zeros_like(p) for p in self.params]
    
    def zero_grad(self):
        for p in self.params:
            if p.grad is not None:
                p.grad.zero_()
    
    def step(self):
        self.t += 1
        for i, p in enumerate(self.params):
            if p.grad is None:
                continue
            g = p.grad.data
            
            # Decoupled weight decay (applied directly to params)
            p.data -= self.lr * self.wd * p.data
            
            # Standard Adam update (without regularization in gradient)
            self.m[i] = self.beta1 * self.m[i] + (1 - self.beta1) * g
            self.v[i] = self.beta2 * self.v[i] + (1 - self.beta2) * g * g
            
            m_hat = self.m[i] / (1 - self.beta1 ** self.t)
            v_hat = self.v[i] / (1 - self.beta2 ** self.t)
            
            p.data -= self.lr * m_hat / (torch.sqrt(v_hat) + self.eps)
```

---

## 11. Adam Variants

| Variant | Key Difference | Use Case |
|---|---|---|
| **Adam** | Standard | General purpose |
| **AdamW** | Decoupled weight decay | Best for regularization |
| **AMSGrad** | Uses max(v̂) instead of v̂ | Theoretical convergence fix |
| **RAdam** | Rectified Adam — auto warmup | Avoids needing lr warmup |
| **NAdam** | Nesterov + Adam | Slight improvement |
| **AdaBelief** | Adapts to gradient "belief" | Better generalization |
| **8-bit Adam** | Quantized states (m, v) | Large model memory savings |
| **Adafactor** | Factored second moment | Massive models (T5, etc.) |

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| First moment (mean) | mₜ = β₁·mₜ₋₁ + (1-β₁)·gₜ |
| Second moment (variance) | vₜ = β₂·vₜ₋₁ + (1-β₂)·gₜ² |
| Bias correction | m̂ₜ = mₜ/(1-β₁ᵗ), v̂ₜ = vₜ/(1-β₂ᵗ) |
| Update rule | θ -= α · m̂ₜ / (√v̂ₜ + ε) |
| Defaults | α=0.001, β₁=0.9, β₂=0.999, ε=1e-8 |
| AdamW | Decoupled weight decay: θ -= α·λ·θ separately |
| Effective step | ≈ α per parameter (gradient-normalized) |
| PyTorch | `torch.optim.Adam(params, lr=0.001)` |
| PyTorch (AdamW) | `torch.optim.AdamW(params, lr=0.001, weight_decay=0.01)` |

---

## Revision Questions

1. **Walk through the complete Adam algorithm step by step.** For each line, explain what it does and why it's needed.

2. **Derive the bias correction formula.** Show that E[mₜ] = (1-β₁ᵗ)·E[g] and explain why dividing by (1-β₁ᵗ) fixes the bias.

3. **Why does Adam approximately normalize the gradient?** Show that for constant gradients, the effective step size is ~α regardless of gradient magnitude.

4. **Explain the difference between Adam + L2 regularization and AdamW.** Why are they not equivalent, unlike in SGD?

5. **Compare Adam with SGD+Momentum for training a ResNet.** Which typically gives better test accuracy, and why?

6. **Implement Adam from scratch** including bias correction, and verify it matches `torch.optim.Adam`.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← RMSProp](./05-rmsprop.md) | [Unit 5: Optimization](./README.md) | [Learning Rate Scheduling →](./07-learning-rate-scheduling.md) |
