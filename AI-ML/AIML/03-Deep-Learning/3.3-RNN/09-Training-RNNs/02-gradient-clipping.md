# Gradient Clipping

> **Unit 9, Chapter 2** — Taming exploding gradients with norm clipping and value clipping techniques for stable RNN training.

---

## 📋 Overview

Recurrent Neural Networks are notorious for **exploding gradients** during training — gradients that grow exponentially as they propagate backward through time. This causes weights to update wildly, leading to NaN losses and training collapse. **Gradient clipping** is the primary defense: it caps gradient magnitudes before they can destabilize training, acting as a safety valve on the optimization process.

---

## 💥 Why Gradients Explode in RNNs

### The Root Cause

During BPTT, gradients involve products of the recurrent weight matrix:

```
∂hₜ/∂hₖ = ∏_{i=k+1}^{t} ∂hᵢ/∂hᵢ₋₁ = ∏_{i=k+1}^{t} diag(tanh'(zᵢ)) · W_hh

If largest eigenvalue of W_hh > 1:
  ‖∂hₜ/∂hₖ‖ ≈ λ_max^(t-k) → ∞  as t-k grows
```

### Visualizing the Problem

```
Gradient magnitude over time steps:

    ‖∇‖
     │
10⁶  │                                          ╱
     │                                        ╱
10⁴  │                                     ╱
     │                                   ╱
10²  │                               ╱
     │                           ╱
10⁰  │  ─ ─ ─ ─ ─ ─ ─ ─ ─ ╱
     │                  ╱          ← Exponential growth
10⁻² │              ╱
     │          ╱
     └──────────────────────────────── time steps
      1    5   10   15   20   25   30

Without clipping: loss → NaN after a few updates
With clipping:    gradients bounded, stable training
```

---

## ✂️ Gradient Norm Clipping

### Algorithm

The most common approach. Rescale the **entire gradient vector** if its norm exceeds a threshold:

```
Given: gradient vector g, threshold τ

if ‖g‖ > τ:
    g ← τ · (g / ‖g‖)       ← Scale down, preserve direction

else:
    g ← g                    ← No change
```

### Geometric Interpretation

```
Before clipping:                After clipping:
                                
    ↗ g (‖g‖ = 100)                ↗ g' (‖g‖ = 5)
   /                               /
  /   (same direction,             / (same direction,
 /     huge magnitude)            /   bounded magnitude)

Clipping preserves the DIRECTION of the gradient
but limits its MAGNITUDE to at most τ.

Think of it as: "Move in the right direction,
                 but don't take a huge step."
```

### Mathematical Details

Using L2 norm:

```
g_clipped = g · min(1, τ / ‖g‖₂)

where ‖g‖₂ = √(Σᵢ gᵢ²)
```

This is equivalent to projecting onto a ball of radius τ:

```
                    ┌─────────┐
              ╱─────│  τ-ball │─────╲
           ╱        └─────────┘        ╲
         ╱     Gradients inside:         ╲
        │       unchanged                 │
        │                                 │
        │     Gradients outside:          │
         ╲     projected to surface     ╱
           ╲                          ╱
              ╲─────────────────────╱
```

---

## 📐 Gradient Value Clipping

### Algorithm

Clip each gradient component independently:

```
For each element gᵢ of gradient vector g:
    gᵢ = max(min(gᵢ, τ), -τ)

    i.e., clamp each gᵢ to [-τ, τ]
```

### Comparison with Norm Clipping

```
Norm Clipping:                 Value Clipping:
┌─────────────────────┐        ┌─────────────────────┐
│ Preserves direction │        │ May change direction │
│ Scales uniformly    │        │ Clips independently  │
│ Preferred in most   │        │ Can distort gradient │
│ RNN applications    │        │ Simpler to implement │
└─────────────────────┘        └─────────────────────┘

Example:
  g = [100, 1], τ = 5

  Norm clipping:  g' = 5 · [100, 1] / √10001 ≈ [5.00, 0.05]
                  Direction preserved: still mostly along dim 0

  Value clipping: g' = [5, 1]
                  Direction changed: dim 1 now has more relative weight
```

---

## 💻 PyTorch Implementation

### Gradient Norm Clipping (Recommended)

```python
import torch
import torch.nn as nn

model = nn.LSTM(input_size=100, hidden_size=256, num_layers=2, batch_first=True)
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
criterion = nn.CrossEntropyLoss()

# Training loop
for batch_x, batch_y in dataloader:
    optimizer.zero_grad()
    output, _ = model(batch_x)
    loss = criterion(output.reshape(-1, vocab_size), batch_y.reshape(-1))
    loss.backward()
    
    # ── Gradient Norm Clipping ──
    # Clips so that total gradient norm ≤ max_norm
    total_norm = torch.nn.utils.clip_grad_norm_(
        model.parameters(), 
        max_norm=5.0        # threshold τ
    )
    # total_norm: the original gradient norm (before clipping)
    
    optimizer.step()
```

### Gradient Value Clipping

```python
# Clips each gradient element to [-clip_value, clip_value]
torch.nn.utils.clip_grad_value_(
    model.parameters(),
    clip_value=1.0      # threshold τ
)
```

### Monitoring Gradient Norms

```python
def train_with_monitoring(model, dataloader, epochs, max_norm=5.0):
    optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
    criterion = nn.CrossEntropyLoss()
    
    for epoch in range(epochs):
        grad_norms = []
        clipped_count = 0
        
        for batch_x, batch_y in dataloader:
            optimizer.zero_grad()
            output, _ = model(batch_x)
            loss = criterion(output.reshape(-1, -1), batch_y.reshape(-1))
            loss.backward()
            
            # Record gradient norm BEFORE clipping
            total_norm = torch.nn.utils.clip_grad_norm_(
                model.parameters(), max_norm=max_norm
            )
            grad_norms.append(total_norm.item())
            
            if total_norm > max_norm:
                clipped_count += 1
            
            optimizer.step()
        
        avg_norm = sum(grad_norms) / len(grad_norms)
        max_observed = max(grad_norms)
        clip_pct = 100 * clipped_count / len(grad_norms)
        
        print(f"Epoch {epoch+1}: "
              f"Avg grad norm: {avg_norm:.2f}, "
              f"Max: {max_observed:.2f}, "
              f"Clipped: {clip_pct:.1f}%")
```

---

## 🎯 Choosing the Clipping Threshold

### Strategy 1: Empirical Monitoring

```python
# Run a few batches without clipping, observe gradient norms
norms = []
for batch_x, batch_y in dataloader:
    optimizer.zero_grad()
    output, _ = model(batch_x)
    loss = criterion(output, batch_y)
    loss.backward()
    
    total_norm = 0
    for p in model.parameters():
        if p.grad is not None:
            total_norm += p.grad.data.norm(2).item() ** 2
    total_norm = total_norm ** 0.5
    norms.append(total_norm)

# Set threshold slightly above typical norm
import numpy as np
threshold = np.percentile(norms, 95)  # clip top 5%
print(f"Suggested threshold: {threshold:.2f}")
```

### Strategy 2: Common Defaults

| Model Type | Typical max_norm |
|-----------|-----------------|
| Vanilla RNN | 1.0 – 5.0 |
| LSTM | 5.0 – 10.0 |
| GRU | 5.0 – 10.0 |
| Deep RNN (3+ layers) | 1.0 – 5.0 |
| Seq2Seq with Attention | 5.0 – 50.0 |

### Strategy 3: Adaptive Approach

```
Start with max_norm = 5.0

If training is unstable (loss spikes):
  → Decrease to 1.0

If training is too slow (gradients always clipped):
  → Increase to 10.0

Monitor the percentage of clipped batches:
  Ideal: 5-20% of batches are clipped
  Too high (>50%): threshold too low, slowing learning
  Too low (<1%): threshold too high, not protecting against spikes
```

---

## 🧮 Worked Example: Norm Clipping

### Setup

```
Parameter gradients after backward pass:

  ∂L/∂W_xh = [3.0, 4.0]      (2D for simplicity)
  ∂L/∂W_hh = [6.0, 8.0]
  ∂L/∂b    = [1.0]

Global gradient vector: g = [3.0, 4.0, 6.0, 8.0, 1.0]

Step 1: Compute L2 norm
  ‖g‖₂ = √(9 + 16 + 36 + 64 + 1) = √126 ≈ 11.22

Step 2: Compare with threshold τ = 5.0
  11.22 > 5.0 → Need to clip!

Step 3: Compute scaling factor
  scale = τ / ‖g‖₂ = 5.0 / 11.22 ≈ 0.446

Step 4: Scale all gradients
  ∂L/∂W_xh = 0.446 · [3.0, 4.0] = [1.34, 1.78]
  ∂L/∂W_hh = 0.446 · [6.0, 8.0] = [2.67, 3.57]
  ∂L/∂b    = 0.446 · [1.0]       = [0.446]

Verify: ‖g_clipped‖₂ = √(1.34² + 1.78² + 2.67² + 3.57² + 0.446²)
                      ≈ √(1.80 + 3.17 + 7.13 + 12.74 + 0.20)
                      = √25.04 ≈ 5.0 ✓
```

---

## 🔍 Clipping and Optimization Interaction

### Effect on Different Optimizers

```
┌──────────────────────────────────────────────────────────────┐
│ SGD + Clipping:                                              │
│   Simple and effective. Clipping directly limits step size.  │
│                                                              │
│ Adam + Clipping:                                             │
│   Adam already normalizes by second moment.                  │
│   Clipping provides additional safety for extreme spikes.    │
│   May need higher threshold since Adam's updates are         │
│   already normalized.                                        │
│                                                              │
│ SGD with Momentum + Clipping:                                │
│   Momentum can accumulate clipped gradients.                 │
│   May need slightly lower threshold.                         │
└──────────────────────────────────────────────────────────────┘
```

### When Clipping Fires Too Often

```
If gradients are ALWAYS being clipped:
  1. Learning rate may be too high → reduce LR
  2. Model may be too deep → reduce layers
  3. Sequence may be too long → use truncated BPTT
  4. Weight initialization may be bad → use orthogonal init
  5. Threshold may be too conservative → increase τ
```

---

## 📋 Summary Table

| Technique | How It Works | Preserves Direction? | Best For |
|-----------|-------------|---------------------|----------|
| Norm clipping | Scale entire gradient if ‖g‖ > τ | ✅ Yes | Most RNN tasks |
| Value clipping | Clamp each element to [-τ, τ] | ❌ May distort | Simple cases |
| No clipping | Raw gradients | ✅ Yes | Stable architectures |

| Parameter | Recommendation |
|-----------|---------------|
| Method | Norm clipping (`clip_grad_norm_`) |
| Threshold (LSTM/GRU) | 5.0 (start here, tune) |
| Threshold (Vanilla RNN) | 1.0 – 5.0 |
| Monitoring | Track % of clipped batches |
| Target clip rate | 5–20% of batches |

---

## ❓ Revision Questions

1. **Explain the difference between gradient norm clipping and gradient value clipping.** Which preserves the gradient direction, and why does that matter?

2. **Given a gradient vector g = [12, 5, 0, -4] and threshold τ = 10**, compute the clipped gradient using (a) norm clipping and (b) value clipping.

3. **Why is gradient clipping especially important for RNNs** but less critical for feedforward networks?

4. **You observe that 80% of your training batches trigger gradient clipping.** What might be wrong, and what would you adjust?

5. **How does gradient clipping interact with Adam optimizer?** Is clipping still necessary when using Adam?

6. **Describe a scenario where gradient clipping masks an underlying architecture problem** rather than solving the root cause.

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Truncated BPTT](01-truncated-bptt.md) |
| ⬆️ Unit | [09 Training RNNs](../README.md) |
| ➡️ Next | [Learning Rate Scheduling](03-learning-rate-scheduling.md) |
| 🏠 Home | [RNN Home](../README.md) |

---

*Last updated: 2024*
