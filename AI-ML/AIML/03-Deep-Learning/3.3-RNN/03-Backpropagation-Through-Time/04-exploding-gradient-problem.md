# Exploding Gradient Problem

> **Unit 3, Chapter 4** — Why gradients explode when W_hh eigenvalues > 1, causing NaN losses, training instability, and divergence.

---

## 📋 Overview

The **exploding gradient problem** is the opposite of vanishing: gradients grow exponentially during BPTT, causing weights to receive enormous updates that destabilize training. While easier to detect than vanishing gradients (NaN values, huge losses), it requires explicit mitigation through **gradient clipping**.

---

## 💥 The Problem

```
Gradient flow backward through time:

   δ_k = ∏(j=k+1 to T) [diag(1-h_j²) · W_hh] · δ_T

If |λ_max(W_hh)| > 1:

   ||δ_k|| ∝ |λ_max|^(T-k) · ||δ_T||

   For |λ_max| = 1.1, T-k = 100:
   1.1^100 = 13,780.6    ← gradient is 14,000x larger!

   For |λ_max| = 1.5, T-k = 50:
   1.5^50 = 637,621,500  ← gradient is 6×10⁸ x larger!

┌────────────────────────────────────────────────────┐
│  Even slightly above 1 → CATASTROPHIC explosion   │
│  over long sequences                               │
└────────────────────────────────────────────────────┘
```

---

## 📊 Symptoms of Exploding Gradients

```
┌──────────────────────────────────────────────────────────────┐
│                SYMPTOMS                                       │
│                                                              │
│  1. Loss suddenly becomes NaN or Inf                         │
│     Epoch 1: loss=2.34, Epoch 2: loss=1.87, Epoch 3: NaN!   │
│                                                              │
│  2. Loss oscillates wildly                                   │
│     Epoch 1: 2.3, Epoch 2: 0.5, Epoch 3: 15.7, Epoch 4: 0.1│
│                                                              │
│  3. Weights become very large                                │
│     ||W|| jumps from 5.0 to 500.0 in one step               │
│                                                              │
│  4. Gradient norms are huge                                  │
│     ||∂L/∂W|| > 1000 (should be ~1-10)                      │
│                                                              │
│  5. Model predictions are nonsensical after a few epochs     │
│                                                              │
│  Loss curve:                                                 │
│     │                    ╱                                    │
│     │                   ╱                                     │
│     │          ╱╲      ╱                                      │
│     │     ╱╲  ╱  ╲    ╱                                       │
│     │    ╱  ╲╱    ╲  ╱     → NaN                             │
│     │   ╱         ╲╱                                         │
│     │──╱─────────────────                                    │
│     └────────────────────                                    │
│      Normal         Exploding                                │
└──────────────────────────────────────────────────────────────┘
```

---

## 🧮 Mathematical Analysis

```
Consider the scalar case for clarity:
   h_t = tanh(w_xh · x_t + w_hh · h_{t-1})

   ∂h_t/∂h_{t-1} = (1 - h_t²) · w_hh

For gradient NOT to explode, we need:
   |1 - h_t²| · |w_hh| ≤ 1 for all t

Since 0 < (1-h_t²) ≤ 1:
   The condition reduces to |w_hh| ≤ 1

If |w_hh| = 1.2:
   • When h_t ≈ 0 (near zero): factor ≈ 1.0 × 1.2 = 1.2
   • Over 50 steps: 1.2^50 = 9,100
   • Over 100 steps: 1.2^100 = 82,817,974

Matrix case (multi-dimensional):
   The spectral radius ρ(W_hh) = max|λ_i| determines behavior
   If ρ(W_hh) > 1 → EXPLOSION is guaranteed for long enough T
```

---

## 📐 Comparison: Vanishing vs Exploding

```
    Gradient
    Magnitude
       │                     EXPLODING
       │                    ╱
       │                   ╱
  100  │                  ╱
       │                 ╱
       │                ╱
   10  │               ╱
       │              ╱
       │             ╱
    1  │────────────────────── IDEAL (constant)
       │            
  0.1  │             ╲
       │              ╲         VANISHING
  0.01 │               ╲
       │                ╲
 0.001 │                 ╲
       └──────────────────────────
       T    T-10   T-20   T-50

Both problems get WORSE with longer sequences.
```

---

## 💻 Demonstration

```python
import torch
import torch.nn as nn
import numpy as np

# ─── Demonstrating gradient explosion ───
def demonstrate_explosion():
    """Show gradient explosion with specific W_hh initialization"""
    hidden_size = 10
    seq_len = 50
    
    rnn = nn.RNN(input_size=5, hidden_size=hidden_size, batch_first=True)
    
    # Force large eigenvalues by scaling W_hh
    with torch.no_grad():
        rnn.weight_hh_l0.copy_(
            torch.eye(hidden_size) * 1.5 + torch.randn(hidden_size, hidden_size) * 0.1
        )
    
    x = torch.randn(1, seq_len, 5, requires_grad=True)
    output, _ = rnn(x)
    loss = output[0, -1, :].sum()
    loss.backward()
    
    grad = x.grad[0]
    print("=== Gradient Explosion Demo ===")
    print(f"{'Time':>6} {'Grad Norm':>15}")
    print("-" * 25)
    for t in [seq_len-1, seq_len-5, seq_len-10, seq_len-20, seq_len-30, 0]:
        if t >= 0:
            print(f"{t:>6} {grad[t].norm().item():>15.4f}")
    
    # Check eigenvalues
    W_hh = rnn.weight_hh_l0.detach().numpy()
    eigenvalues = np.abs(np.linalg.eigvals(W_hh))
    print(f"\nMax |eigenvalue| of W_hh: {eigenvalues.max():.4f}")
    print(f"Gradients at early time steps are HUGE → explosion!")

demonstrate_explosion()

# ─── Show NaN from explosion ───
def show_nan_training():
    """Training that diverges due to exploding gradients"""
    model = nn.RNN(10, 20, batch_first=True)
    
    # Sabotage: set large W_hh
    with torch.no_grad():
        model.weight_hh_l0.mul_(3.0)
    
    optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
    
    print("\n=== Training Without Gradient Clipping ===")
    for epoch in range(10):
        x = torch.randn(4, 30, 10)
        output, _ = model(x)
        loss = output.sum()
        
        optimizer.zero_grad()
        loss.backward()
        
        grad_norm = model.weight_hh_l0.grad.norm().item()
        
        optimizer.step()
        print(f"Epoch {epoch+1}: loss={loss.item():>15.4f}, "
              f"grad_norm={grad_norm:>15.4f}")
        
        if torch.isnan(loss) or torch.isinf(loss):
            print("  TRAINING DIVERGED! (NaN/Inf)")
            break

show_nan_training()
```

---

## 🛡️ Solutions

```
┌─────────────────────────────────────────────────────────────────┐
│               SOLUTIONS TO EXPLODING GRADIENTS                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. GRADIENT CLIPPING (most common)                            │
│     if ||g|| > threshold:                                      │
│         g = g × (threshold / ||g||)                            │
│     Simple, effective, widely used                             │
│                                                                 │
│  2. Weight Initialization                                      │
│     Initialize W_hh as orthogonal matrix (eigenvalues = 1)    │
│     Delays explosion but doesn't prevent it                   │
│                                                                 │
│  3. Regularization                                             │
│     L2 penalty on W_hh keeps eigenvalues bounded              │
│                                                                 │
│  4. Architecture Change                                        │
│     LSTM/GRU have gating that controls gradient flow          │
│                                                                 │
│  5. Truncated BPTT                                             │
│     Only backpropagate through k < T time steps               │
│     Limits gradient path length                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Exploding Gradients | Gradient norms grow exponentially with time distance |
| Cause | \|λ_max(W_hh)\| > 1 → repeated multiplication amplifies |
| Symptoms | NaN loss, huge weight updates, oscillating loss |
| Detection | Monitor gradient norms, check for NaN |
| Primary Solution | Gradient clipping (next chapter) |
| Initialization | Orthogonal W_hh helps delay explosion |
| vs Vanishing | Exploding is easier to detect but equally harmful |

---

## ❓ Revision Questions

1. **If the largest eigenvalue of W_hh is 1.3, how large will the gradient become after 30 time steps of backpropagation?**

2. **List three observable symptoms of exploding gradients during training.**

3. **Why is exploding gradient easier to detect than vanishing gradient?**

4. **What is orthogonal initialization? Why does it help with exploding gradients?**

5. **Explain why gradient clipping solves exploding gradients but NOT vanishing gradients.**

6. **A model trains fine on sequences of length 20 but produces NaN on sequences of length 100. What is the likely cause, and how would you fix it?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Vanishing Gradient Problem](03-vanishing-gradient-problem.md) |
| ➡️ Next | [Gradient Clipping](05-gradient-clipping.md) |
| ⬆️ Unit Overview | [README](../README.md) |
