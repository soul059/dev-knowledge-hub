# BPTT Algorithm

> **Unit 3, Chapter 1** — Backpropagation Through Time as chain rule applied through the unrolled computational graph, with gradient computation across all time steps.

---

## 📋 Overview

**Backpropagation Through Time (BPTT)** is the algorithm for computing gradients in RNNs. It works by "unrolling" the RNN across time steps and applying the standard backpropagation chain rule through the resulting computational graph. Since weights are shared across time steps, gradients from every time step must be **accumulated**.

---

## 🔄 The Unrolled Computational Graph

```
Forward Pass:
   x₁         x₂         x₃         x₄
    │          │          │          │
    ▼          ▼          ▼          ▼
h₀→[RNN]→h₁→[RNN]→h₂→[RNN]→h₃→[RNN]→h₄
    │          │          │          │
    ▼          ▼          ▼          ▼
   y₁         y₂         y₃         y₄
    │          │          │          │
    ▼          ▼          ▼          ▼
   L₁         L₂         L₃         L₄

Backward Pass (BPTT):
   ∂L₁        ∂L₂        ∂L₃        ∂L₄
    │          │          │          │
    ▼          ▼          ▼          ▼
   [RNN]←────[RNN]←────[RNN]←────[RNN]
    │          │          │          │
    ▼          ▼          ▼          ▼
   ∂W         ∂W         ∂W         ∂W
   (accumulated from all time steps)
```

---

## 🧮 Mathematical Derivation

### Setup

```
Equations at each time step t:
   h_t = tanh(W_xh · x_t + W_hh · h_{t-1} + b_h)    ... (1)
   o_t = W_hy · h_t + b_y                              ... (2)
   L_t = loss(o_t, target_t)                            ... (3)

Total loss:
   L = Σ(t=1 to T) L_t
```

### Gradient of L w.r.t. W_hy (Simple Case)

```
∂L/∂W_hy = Σ(t=1 to T) ∂L_t/∂W_hy

Each L_t only depends on W_hy through o_t:
   ∂L_t/∂W_hy = ∂L_t/∂o_t · ∂o_t/∂W_hy
              = δ_t^o · h_t^T

where δ_t^o = ∂L_t/∂o_t (output error)

So: ∂L/∂W_hy = Σ(t=1 to T) δ_t^o · h_t^T
```

### Gradient of L w.r.t. W_hh (The Hard Part!)

```
W_hh affects L_t through h_t, BUT h_t depends on h_{t-1},
which depends on h_{t-2}, ..., which ALL depend on W_hh!

∂L_t/∂W_hh = Σ(k=1 to t) ∂L_t/∂h_t · (∂h_t/∂h_k) · ∂h_k/∂W_hh

The chain ∂h_t/∂h_k requires multiplying through all intermediate steps:

∂h_t/∂h_k = ∏(j=k+1 to t) ∂h_j/∂h_{j-1}

Each factor: ∂h_j/∂h_{j-1} = diag(1 - h_j²) · W_hh
                                ──────────────   ─────
                                tanh derivative   recurrent weight
```

### Complete Gradient

```
                    T     t
∂L/∂W_hh = Σ     Σ    ∂L_t/∂o_t · ∂o_t/∂h_t · (∏ ∂h_j/∂h_{j-1}) · ∂h_k/∂W_hh
               t=1   k=1                           j=k+1

This is why it's called "through time" — gradients propagate
backward through EVERY time step in the sequence.
```

---

## 📐 BPTT Algorithm Step-by-Step

```
┌─────────────────────────────────────────────────────────────────┐
│                    BPTT ALGORITHM                                │
│                                                                  │
│  1. FORWARD PASS: Compute and store h₁, h₂, ..., h_T           │
│                                                                  │
│  2. COMPUTE OUTPUT GRADIENTS:                                    │
│     For t = T down to 1:                                        │
│       δ_t^o = ∂L_t/∂o_t          (output gradient)             │
│                                                                  │
│  3. BACKPROPAGATE THROUGH TIME:                                  │
│     δ_T^h = W_hy^T · δ_T^o      (initialize from last step)   │
│                                                                  │
│     For t = T-1 down to 1:                                      │
│       δ_t^h = W_hy^T · δ_t^o                                   │
│             + W_hh^T · diag(1 - h_{t+1}²) · δ_{t+1}^h         │
│             ─────────────────────────────────────────            │
│             gradient from future + gradient from current output  │
│                                                                  │
│  4. ACCUMULATE PARAMETER GRADIENTS:                              │
│     ∂L/∂W_hy = Σ_t  δ_t^o · h_t^T                             │
│     ∂L/∂W_hh = Σ_t  (diag(1-h_t²) · δ_t^h) · h_{t-1}^T      │
│     ∂L/∂W_xh = Σ_t  (diag(1-h_t²) · δ_t^h) · x_t^T          │
│                                                                  │
│  5. UPDATE PARAMETERS: W ← W - η · ∂L/∂W                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔢 Numerical Example

```
Setup: d=1, n=1, m=1 (scalar case for clarity)
W_xh = 0.5,  W_hh = 0.8,  W_hy = 1.0
b_h = 0,  b_y = 0,  h₀ = 0

Inputs: x₁ = 1.0, x₂ = 0.5, x₃ = -0.3
Targets: y₁* = 0.5, y₂* = 0.3, y₃* = -0.1
Loss: L_t = 0.5(o_t - y_t*)²

── Forward Pass ──
t=1: a₁ = 0.5·1.0 + 0.8·0 = 0.5
     h₁ = tanh(0.5) = 0.4621
     o₁ = 1.0·0.4621 = 0.4621
     L₁ = 0.5·(0.4621-0.5)² = 0.000072

t=2: a₂ = 0.5·0.5 + 0.8·0.4621 = 0.6197
     h₂ = tanh(0.6197) = 0.5511
     o₂ = 0.5511
     L₂ = 0.5·(0.5511-0.3)² = 0.03156

t=3: a₃ = 0.5·(-0.3) + 0.8·0.5511 = 0.2909
     h₃ = tanh(0.2909) = 0.2826
     o₃ = 0.2826
     L₃ = 0.5·(0.2826-(-0.1))² = 0.07325

── Backward Pass ──
Output gradients: δ_t^o = o_t - y_t*
  δ₃^o = 0.2826 - (-0.1) = 0.3826
  δ₂^o = 0.5511 - 0.3 = 0.2511
  δ₁^o = 0.4621 - 0.5 = -0.0379

Hidden gradients (backward through time):
  δ₃^h = W_hy · δ₃^o = 1.0 · 0.3826 = 0.3826
  δ₂^h = W_hy · δ₂^o + W_hh · (1-h₃²) · δ₃^h
       = 0.2511 + 0.8 · (1-0.2826²) · 0.3826
       = 0.2511 + 0.8 · 0.9202 · 0.3826
       = 0.2511 + 0.2817 = 0.5328
  δ₁^h = W_hy · δ₁^o + W_hh · (1-h₂²) · δ₂^h
       = -0.0379 + 0.8 · (1-0.5511²) · 0.5328
       = -0.0379 + 0.8 · 0.6963 · 0.5328
       = -0.0379 + 0.2968 = 0.2589

Gradient accumulation for W_hh:
  ∂L/∂W_hh = Σ_t (1-h_t²) · δ_t^h · h_{t-1}
  = (1-h₁²)·δ₁^h·h₀ + (1-h₂²)·δ₂^h·h₁ + (1-h₃²)·δ₃^h·h₂
  = 0.7865·0.2589·0 + 0.6963·0.5328·0.4621 + 0.9202·0.3826·0.5511
  = 0 + 0.1714 + 0.1940
  = 0.3654
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn

class RNNWithManualBPTT(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super().__init__()
        self.hidden_size = hidden_size
        self.W_xh = nn.Parameter(torch.randn(hidden_size, input_size) * 0.1)
        self.W_hh = nn.Parameter(torch.randn(hidden_size, hidden_size) * 0.1)
        self.W_hy = nn.Parameter(torch.randn(output_size, hidden_size) * 0.1)
        self.b_h = nn.Parameter(torch.zeros(hidden_size))
        self.b_y = nn.Parameter(torch.zeros(output_size))
    
    def forward(self, x):
        """x: (seq_len, input_size)"""
        T = x.shape[0]
        h = torch.zeros(self.hidden_size)
        outputs = []
        
        for t in range(T):
            h = torch.tanh(self.W_xh @ x[t] + self.W_hh @ h + self.b_h)
            o = self.W_hy @ h + self.b_y
            outputs.append(o)
        
        return torch.stack(outputs)

# PyTorch autograd handles BPTT automatically!
model = RNNWithManualBPTT(input_size=2, hidden_size=4, output_size=1)
x = torch.randn(5, 2)  # 5 time steps
targets = torch.randn(5, 1)

# Forward
outputs = model(x)
loss = ((outputs - targets) ** 2).mean()

# Backward (BPTT happens automatically!)
loss.backward()

print("=== BPTT Gradients ===")
print(f"∂L/∂W_xh shape: {model.W_xh.grad.shape}, norm: {model.W_xh.grad.norm():.4f}")
print(f"∂L/∂W_hh shape: {model.W_hh.grad.shape}, norm: {model.W_hh.grad.norm():.4f}")
print(f"∂L/∂W_hy shape: {model.W_hy.grad.shape}, norm: {model.W_hy.grad.norm():.4f}")
print("Gradients accumulated from ALL time steps automatically!")
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| BPTT | Backpropagation applied to unrolled RNN graph |
| Gradient Accumulation | Same weights used at every step → sum gradients from all steps |
| ∂L/∂W_hy | Sum of output gradients (simple — no temporal dependency) |
| ∂L/∂W_hh | Complex — must chain through all intermediate hidden states |
| ∂h_t/∂h_{t-1} | diag(1-h_t²) · W_hh (the temporal Jacobian) |
| Memory Cost | Must store all h₁..h_T from forward pass |
| Time Complexity | O(T) forward + O(T) backward = O(T) total |

---

## ❓ Revision Questions

1. **Why is BPTT called "through time"? How does it differ from standard backpropagation?**

2. **Write the expression for ∂L₃/∂W_hh. How many terms does the sum have, and why?**

3. **What is the Jacobian ∂h_t/∂h_{t-1}? Why does it involve both tanh derivative and W_hh?**

4. **In the numerical example, why is the gradient contribution from t=1 zero for ∂L/∂W_hh? (Hint: look at h₀.)**

5. **Why must we store all intermediate hidden states during the forward pass?**

6. **If a sequence has T=1000 time steps, how does the computational cost of BPTT compare to training a 1000-layer feedforward network?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [RNN Task Variants](../02-Basic-RNN/06-rnn-task-variants.md) |
| ➡️ Next | [Gradient Flow](02-gradient-flow.md) |
| ⬆️ Unit Overview | [README](../README.md) |
