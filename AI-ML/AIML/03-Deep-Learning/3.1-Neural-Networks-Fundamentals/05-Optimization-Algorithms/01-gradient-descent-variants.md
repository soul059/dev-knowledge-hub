# 1. Gradient Descent Variants

> **Unit 5 · Optimization Algorithms** — Understanding Batch GD, Stochastic GD, and Mini-Batch GD

---

## Chapter Overview

Gradient Descent is the backbone of neural network training. Given a loss function **J(θ)**, we iteratively update parameters **θ** in the direction that reduces J most steeply. The three classical variants differ in *how much data* they use to compute each gradient estimate, creating a fundamental trade-off between **accuracy of the gradient**, **computational cost per step**, and **convergence behavior**.

| Variant | Data per Step | Gradient Quality | Update Speed | Memory |
|---|---|---|---|---|
| Batch GD | Full dataset (N) | Exact (true gradient) | Slow per step | High |
| Stochastic GD (SGD) | 1 sample | Very noisy | Very fast per step | Minimal |
| Mini-Batch GD | B samples (16–512) | Moderate noise | Fast per step | Moderate |

---

## 1. Batch Gradient Descent (Full-Batch)

### Algorithm

Compute the gradient over the **entire** training set of N samples, then take a single step:

```
θₜ₊₁ = θₜ − α · (1/N) Σᵢ₌₁ᴺ ∇θ L(xᵢ, yᵢ; θₜ)
```

where:
- **α** = learning rate (step size)
- **L(xᵢ, yᵢ; θ)** = loss for sample i
- **J(θ) = (1/N) Σ L** = empirical risk (mean loss)

### Characteristics

| Property | Detail |
|---|---|
| Gradient | True gradient (no noise) |
| Convergence | Smooth, deterministic path |
| Per-step cost | O(N) — must scan entire dataset |
| Memory | Must fit all N samples (or accumulate gradients) |
| Parallelism | Easily parallelizable within one step |

### When Batch GD Works

- Small datasets (N < 10,000)
- Convex problems where exact gradients help
- When you need deterministic, reproducible training

### Limitations

- **Slow**: For ImageNet (1.2M images), one step requires a full pass
- **Redundancy**: Many samples contribute similar gradient info
- **Saddle points**: Deterministic path can get stuck at saddle points
- **No escape from local minima** in non-convex landscapes

---

## 2. Stochastic Gradient Descent (SGD)

### Algorithm

Use a **single randomly chosen sample** to estimate the gradient:

```
θₜ₊₁ = θₜ − α · ∇θ L(xᵢ, yᵢ; θₜ)     where i ~ Uniform{1, ..., N}
```

### Key Insight: Unbiased Estimator

The single-sample gradient is an **unbiased estimate** of the true gradient:

```
E[∇θ L(xᵢ, yᵢ; θ)] = (1/N) Σᵢ₌₁ᴺ ∇θ L(xᵢ, yᵢ; θ) = ∇θ J(θ)
```

So on average, SGD moves in the right direction, but with **high variance**.

### SGD Noise as Implicit Regularization

The noise in SGD has been shown to:

1. **Help escape sharp minima** — noisy updates can bounce out of narrow loss valleys
2. **Prefer flat minima** — flat minima are more robust to perturbation → better generalization
3. **Act as an implicit regularizer** — noise proportional to learning rate prevents overfitting

```
Noise scale ≈ α / B × (Gradient variance)
```

Where B is the batch size (B=1 for pure SGD).

### Characteristics

| Property | Detail |
|---|---|
| Gradient | Very noisy (high variance) |
| Convergence | Oscillatory, non-monotonic |
| Per-step cost | O(1) |
| Memory | Single sample |
| Benefit | Can escape local minima |

---

## 3. Mini-Batch Gradient Descent

### Algorithm

Sample a mini-batch **B** of size B from the dataset:

```
θₜ₊₁ = θₜ − α · (1/B) Σⱼ∈B ∇θ L(xⱼ, yⱼ; θₜ)
```

### Why Mini-Batch is the Sweet Spot

```
         Batch GD                Mini-Batch GD              SGD
    ┌─────────────────┐    ┌─────────────────────┐    ┌────────────┐
    │  All N samples  │    │   B samples (32-512) │    │  1 sample  │
    │  per update     │    │   per update          │    │ per update │
    │                 │    │                       │    │            │
    │ ✓ True gradient │    │ ✓ Reduced variance    │    │ ✗ Very     │
    │ ✗ Very slow     │    │ ✓ GPU parallelism     │    │   noisy    │
    │ ✗ No noise      │    │ ✓ Some noise (good!)  │    │ ✓ Fast     │
    │   benefit       │    │ ✓ Practical speed     │    │ ✗ No GPU   │
    └─────────────────┘    └─────────────────────┘    │   benefit  │
                                                       └────────────┘
```

### Variance Reduction

For a mini-batch of size B, the variance of the gradient estimate is:

```
Var(ĝ_minibatch) = Var(ĝ_single) / B
```

So doubling the batch size **halves** the gradient variance (standard error goes down as 1/√B).

### Common Batch Sizes

| Batch Size | Use Case |
|---|---|
| 16–32 | Small models, limited GPU memory |
| 64–128 | General purpose, good default |
| 256–512 | Large models with ample GPU memory |
| 1024+ | Distributed training, requires LR scaling |

### Linear Scaling Rule

When increasing batch size by k, increase learning rate by k:

```
α_new = k · α_original
```

This approximately preserves the signal-to-noise ratio of the gradient update.

---

## Convergence Behavior Comparison

### ASCII Convergence Paths

```
Loss ↑
  │
  │  ╲                          Legend:
  │   ╲  . . .                  ──── Batch GD (smooth)
  │    ╲. .   . .               · · · SGD (very noisy)
  │     ──╲ .    .  .           - - - Mini-Batch (moderate)
  │        ──╲.    . .
  │      - -  ──╲ .  .  .
  │          - - ──╲.  .  .
  │              - -──╲  . .
  │                  - ──╲. .
  │                    - -──╲.
  │                        -──── ← Batch GD converges smoothly
  │                         - - - ← Mini-batch: some oscillation
  │                           .  . ← SGD: lots of oscillation
  └──────────────────────────────────── Iterations →
```

### Convergence in the Loss Landscape (2D View)

```
  Contour plot of loss function J(θ₁, θ₂):

      ╭─────────────────────────╮
      │          ╭───╮          │
      │        ╭─┤   ├─╮       │
      │       ╭┤ │ ★ │ ├╮      │   ★ = optimum
      │       │╰─┤   ├─╯│      │
      │        ╰─┤   ├─╯       │   ──── Batch GD: direct path
      │   S───────┼───╯         │   ····· SGD: erratic path
      │    ╲·.·.·.│             │   - - - Mini-batch
      │     ╲- - -│             │
      │      ╲    │             │   S = start
      ╰─────────────────────────╯

  Batch GD: S ──────────────────→ ★ (straight, slow steps)
  SGD:      S ·╲·╱·╲·╱·╲·╱·╲·→ ★ (zigzag, fast steps)
  Mini-Batch: S - -╲- -╱- - -→ ★ (mild zigzag)
```

---

## Vectorized Mini-Batch Implementation

### Efficient Epoch Structure

```
One Epoch = One full pass through the dataset

  Dataset: [x₁, x₂, x₃, ..., x_N]
                                          Shuffle at start of each epoch
  After shuffle: [x₇, x₂, x₉, ..., x₄]

  Split into mini-batches of size B:
  ┌─────────┐ ┌─────────┐ ┌─────────┐     ┌─────────┐
  │ Batch 1  │ │ Batch 2  │ │ Batch 3  │ ... │Batch N/B│
  │ B samples│ │ B samples│ │ B samples│     │≤B smpls │
  └─────────┘ └─────────┘ └─────────┘     └─────────┘

  Number of updates per epoch = ⌈N/B⌉
```

### Vectorized Computation

For a mini-batch **X** of shape (B, d):

```
Forward pass:    Z = X @ W + b          # (B, d) @ (d, h) → (B, h)
                 A = σ(Z)               # element-wise activation

Loss:            L = (1/B) Σᵢ loss(aᵢ, yᵢ)

Backward pass:   dW = (1/B) X.T @ δ    # (d, B) @ (B, h) → (d, h)
                 db = (1/B) Σᵢ δᵢ      # sum over batch

Update:          W = W - α · dW
                 b = b - α · db
```

GPU parallelism makes the matrix multiply **X @ W** extremely fast for batch sizes that fit in GPU memory.

---

## Loss Landscape Traversal

### Types of Critical Points

```
     Local        Saddle         Global
     Minimum      Point          Minimum
       ╲╱           ╲╱╲           ╲╱
      ╱  ╲         ╱    ╲        ╱  ╲
     ╱    ╲       ╱      ╲      ╱    ╲
    ╱      ╲     ╱ (↕ goes╲    ╱      ╲
                   both ways)
```

In high-dimensional spaces:
- **Saddle points** are far more common than local minima
- SGD noise helps escape saddle points (gradient noise pushes in escape directions)
- Batch GD can get trapped at saddle points (zero gradient)

### Effect of Learning Rate

```
  Small α:             Medium α:            Large α:
  ╲                    ╲                    ╲
   ╲                    ╲                    ╲   ╱╲
    ╲                    ╲  ╱                 ╲ ╱  ╲
     ╲                    ╲╱                   ╲    → diverges!
      ╲╱ ← slow            ← just right
       (many steps)      (few steps)
```

---

## Worked Example: Mini-Batch GD on a Simple Function

### Problem

Minimize **J(θ) = θ²** using mini-batch SGD with B=2 samples from {x₁=1, x₂=3, x₃=2, x₄=4} where the loss per sample is L(xᵢ, θ) = (θ - xᵢ)².

### Solution

True gradient: ∇J(θ) = (1/4) Σ 2(θ - xᵢ) = 2θ - 2·(mean of xᵢ) = 2θ - 5

Starting at θ₀ = 0, α = 0.1:

**Step 1**: Mini-batch = {x₁=1, x₃=2}
```
ĝ = (1/2)[2(0-1) + 2(0-2)] = (1/2)[-2 + -4] = -3.0
θ₁ = 0 - 0.1·(-3.0) = 0.3
```

**Step 2**: Mini-batch = {x₂=3, x₄=4}
```
ĝ = (1/2)[2(0.3-3) + 2(0.3-4)] = (1/2)[-5.4 + -7.4] = -6.4
θ₂ = 0.3 - 0.1·(-6.4) = 0.94
```

The true optimum is θ* = mean(xᵢ) = 2.5. We're converging (0 → 0.3 → 0.94 → ...).

Compare: Batch GD would compute ĝ = 2(0) - 5 = -5, giving θ₁ = 0.5.

---

## Python Implementation (PyTorch)

### Manual Mini-Batch Training Loop

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

# ── Synthetic Data ──────────────────────────────────────────────
torch.manual_seed(42)
X = torch.randn(1000, 10)
y = (X @ torch.randn(10, 1) + 0.5 * torch.randn(1000, 1)).squeeze()

dataset = TensorDataset(X, y)

# ── Model ───────────────────────────────────────────────────────
model = nn.Sequential(
    nn.Linear(10, 64),
    nn.ReLU(),
    nn.Linear(64, 1)
)
criterion = nn.MSELoss()

# ── Comparing Batch Sizes ──────────────────────────────────────
results = {}

for batch_size in [1, 32, 128, 1000]:
    # Reset model
    torch.manual_seed(0)
    model = nn.Sequential(nn.Linear(10, 64), nn.ReLU(), nn.Linear(64, 1))
    optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
    loader = DataLoader(dataset, batch_size=batch_size, shuffle=True)
    
    losses = []
    for epoch in range(20):
        epoch_loss = 0.0
        for X_batch, y_batch in loader:
            optimizer.zero_grad()
            pred = model(X_batch).squeeze()
            loss = criterion(pred, y_batch)
            loss.backward()
            optimizer.step()
            epoch_loss += loss.item() * len(X_batch)
        
        avg_loss = epoch_loss / len(dataset)
        losses.append(avg_loss)
    
    results[batch_size] = losses
    name = {1: "SGD", 32: "Mini-32", 128: "Mini-128", 1000: "Batch"}
    print(f"{name[batch_size]:>8s}: Final loss = {losses[-1]:.4f}")

# Output (typical):
# SGD:      Final loss = 0.2891
# Mini-32:  Final loss = 0.2634
# Mini-128: Final loss = 0.2718
# Batch:    Final loss = 0.2845
```

### Visualizing Convergence

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(10, 6))
styles = {1: (':', 'red'), 32: ('-', 'blue'), 128: ('--', 'green'), 1000: ('-.', 'purple')}
names = {1: "SGD (B=1)", 32: "Mini-batch (B=32)", 128: "Mini-batch (B=128)", 1000: "Batch GD (B=N)"}

for bs, losses in results.items():
    ls, color = styles[bs]
    ax.plot(losses, linestyle=ls, color=color, label=names[bs], linewidth=2)

ax.set_xlabel("Epoch", fontsize=12)
ax.set_ylabel("Loss", fontsize=12)
ax.set_title("Convergence: Batch vs Mini-Batch vs SGD", fontsize=14)
ax.legend(fontsize=11)
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig("gd_variants_comparison.png", dpi=150)
plt.show()
```

### Custom SGD with Gradient Accumulation

```python
# Simulating large batch with gradient accumulation
# Useful when full batch doesn't fit in GPU memory

accumulation_steps = 8   # effective batch = 32 * 8 = 256
physical_batch = 32

model = nn.Sequential(nn.Linear(10, 64), nn.ReLU(), nn.Linear(64, 1))
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
loader = DataLoader(dataset, batch_size=physical_batch, shuffle=True)

optimizer.zero_grad()
for i, (X_batch, y_batch) in enumerate(loader):
    pred = model(X_batch).squeeze()
    loss = criterion(pred, y_batch) / accumulation_steps  # scale loss
    loss.backward()                                        # accumulate gradients
    
    if (i + 1) % accumulation_steps == 0:
        optimizer.step()    # update with accumulated gradient
        optimizer.zero_grad()
```

---

## Applications

| Domain | Preferred Variant | Reason |
|---|---|---|
| Small tabular data | Batch GD | Few samples, exact gradient helpful |
| Image classification | Mini-batch (32–256) | GPU parallelism, moderate noise |
| NLP / Transformers | Mini-batch (8–64) | Memory constraints, long sequences |
| Reinforcement Learning | SGD / small mini-batch | Online learning, streaming data |
| Distributed training | Large mini-batch (1K–64K) | Communication efficiency |

---

## Summary Table

| Aspect | Batch GD | Mini-Batch GD | SGD |
|---|---|---|---|
| **Samples per update** | N (all) | B (subset) | 1 |
| **Updates per epoch** | 1 | ⌈N/B⌉ | N |
| **Gradient quality** | True gradient | Low variance estimate | High variance estimate |
| **GPU utilization** | Good | Excellent | Poor |
| **Convergence path** | Smooth | Mild oscillation | Very noisy |
| **Escapes local minima** | No | Sometimes | Yes (via noise) |
| **Memory** | O(N) | O(B) | O(1) |
| **Practical use** | Rare | **Default choice** | Rare (pure form) |

---

## Revision Questions

1. **Why is the single-sample gradient in SGD an unbiased estimator of the true gradient?** Explain mathematically and discuss what "unbiased" means in this context.

2. **Derive the relationship between batch size B and gradient variance.** Why does doubling the batch size only reduce the standard error by a factor of √2, not 2?

3. **Explain how SGD noise acts as an implicit regularizer.** What is the relationship between learning rate, batch size, and the noise scale?

4. **What is gradient accumulation, and when would you use it?** Write the key code pattern for accumulating gradients over K mini-batches.

5. **Describe the linear scaling rule for learning rates when increasing batch size.** Why does it work, and when does it break down?

6. **Compare the loss landscape traversal of Batch GD and SGD near a saddle point.** Which method is more likely to escape, and why?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| ← [Unit 4: Backpropagation](../04-Backpropagation/) | [Unit 5: Optimization](./README.md) | [Momentum →](./02-momentum.md) |
