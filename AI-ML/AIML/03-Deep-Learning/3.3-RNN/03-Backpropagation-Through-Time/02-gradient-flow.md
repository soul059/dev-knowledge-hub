# Gradient Flow Through Time

> **Unit 3, Chapter 2** — How gradients propagate through time via repeated matrix multiplication, and the conditions that lead to vanishing or exploding gradients.

---

## 📋 Overview

When gradients flow backward through an RNN, they must pass through a chain of matrix multiplications involving W_hh and tanh derivatives. The **eigenvalues** of W_hh determine whether gradients vanish (shrink to zero) or explode (grow unboundedly).

---

## 🔗 The Gradient Chain

```
To compute how loss at time T affects parameters at time k:

∂L_T     ∂L_T   ∂o_T   T-1
───── = ────── · ──── · ∏   ∂h_{j+1}/∂h_j  · ∂h_k/∂W
∂W       ∂o_T   ∂h_T   j=k

Each factor in the product:
   ∂h_{j+1}/∂h_j = diag(1 - h_{j+1}²) · W_hh

The product of T-k such matrices:

   T-1
   ∏  [diag(1 - h_{j+1}²) · W_hh]
   j=k

This is REPEATED MATRIX MULTIPLICATION — the key issue!
```

---

## 📊 Gradient Flow Visualization

```
                    GRADIENT FLOW BACKWARD

Loss at T:    L_T
               │
               ▼
t=T:   δ_T = ∂L_T/∂h_T
               │
               │  × diag(1-h_T²) · W_hh
               ▼
t=T-1: δ_{T-1}
               │
               │  × diag(1-h_{T-1}²) · W_hh
               ▼
t=T-2: δ_{T-2}
               │
               │  × diag(1-h_{T-2}²) · W_hh
               ▼
              ...
               │
               │  × diag(1-h_{k+1}²) · W_hh
               ▼
t=k:   δ_k                    ← Gradient has been multiplied
                                  (T-k) times by similar matrices!

If each multiplication SHRINKS by factor γ < 1:
   ||δ_k|| ≈ ||δ_T|| · γ^(T-k)  → VANISHES exponentially

If each multiplication GROWS by factor γ > 1:
   ||δ_k|| ≈ ||δ_T|| · γ^(T-k)  → EXPLODES exponentially
```

---

## 🧮 Eigenvalue Analysis

```
The behavior of repeated multiplication A^n depends on eigenvalues of A:

Let λ₁, λ₂, ..., λ_n be eigenvalues of W_hh

Case 1: |λ_max| < 1  →  W_hh^n → 0 as n → ∞
         → Gradients VANISH

Case 2: |λ_max| > 1  →  ||W_hh^n|| → ∞ as n → ∞
         → Gradients EXPLODE

Case 3: |λ_max| = 1  →  Gradients preserved (ideal but unstable)

Combined with tanh derivative (always ≤ 1):
   Factor at each step: ||diag(1-h²) · W_hh|| ≤ ||W_hh||

   Since tanh'(x) = 1 - tanh²(x) ∈ (0, 1]:
   - At saturation (|h| ≈ 1): tanh' ≈ 0 → gradient KILLED
   - Near zero (|h| ≈ 0): tanh' ≈ 1 → gradient preserved

┌──────────────────────────────────────────────────┐
│ Combined condition:                              │
│   σ_max(W_hh) × max(tanh') < 1 → VANISHING     │
│   σ_max(W_hh) × max(tanh') > 1 → EXPLODING     │
│                                                  │
│ Since max(tanh') = 1, the critical boundary is: │
│   σ_max(W_hh) = 1                               │
└──────────────────────────────────────────────────┘
```

---

## 📐 Gradient Magnitude Over Time

```
Gradient magnitude ||∂L_T/∂h_k|| as a function of (T-k):

    ||grad||
    │
    │  EXPLODING (|λ|>1)
    │     /
    │    /
    │   /        IDEAL (|λ|≈1)
    │──/─────────────────────────
    │ /
    │/          VANISHING (|λ|<1)
    │ ─────────____________________
    │
    └──────────────────────────────
    k=T  k=T-5  k=T-10  k=T-50  k=1

    Distance from loss (T-k)

For T=100 and a typical shrinkage factor of 0.9:
   At k=T-10:  0.9^10  = 0.349   (65% lost)
   At k=T-50:  0.9^50  = 0.0052  (99.5% lost)
   At k=T-100: 0.9^100 = 0.0000027  (essentially zero!)
```

---

## 💻 Demonstrating Gradient Flow

```python
import torch
import torch.nn as nn
import numpy as np

def analyze_gradient_flow(hidden_size=50, seq_lengths=[10, 50, 100, 200]):
    """Show how gradient magnitudes change with sequence length"""
    results = {}
    
    for T in seq_lengths:
        rnn = nn.RNN(input_size=10, hidden_size=hidden_size, batch_first=True)
        x = torch.randn(1, T, 10, requires_grad=True)
        
        output, h_n = rnn(x)
        
        # Loss from final time step only
        loss = output[0, -1, :].sum()
        loss.backward()
        
        # Check gradient magnitude at different time steps
        grad = x.grad[0]  # (T, 10)
        grad_norms = [grad[t].norm().item() for t in range(T)]
        
        results[T] = {
            'first': grad_norms[0],
            'middle': grad_norms[T//2],
            'last': grad_norms[-1],
            'ratio': grad_norms[0] / (grad_norms[-1] + 1e-10)
        }
    
    print("=== Gradient Flow Analysis ===")
    print(f"{'Seq Len':<10} {'First':>12} {'Middle':>12} {'Last':>12} {'First/Last':>12}")
    print("-" * 58)
    for T, r in results.items():
        print(f"{T:<10} {r['first']:>12.6f} {r['middle']:>12.6f} "
              f"{r['last']:>12.6f} {r['ratio']:>12.4f}")

analyze_gradient_flow()

# Eigenvalue analysis
def check_eigenvalues():
    rnn = nn.RNN(10, 20)
    W_hh = rnn.weight_hh_l0.detach().numpy()
    eigenvalues = np.linalg.eigvals(W_hh)
    magnitudes = np.abs(eigenvalues)
    
    print(f"\n=== W_hh Eigenvalue Analysis ===")
    print(f"Max |eigenvalue|: {magnitudes.max():.4f}")
    print(f"Min |eigenvalue|: {magnitudes.min():.4f}")
    print(f"Mean |eigenvalue|: {magnitudes.mean():.4f}")
    
    if magnitudes.max() > 1:
        print("WARNING: Max eigenvalue > 1 → potential gradient explosion")
    else:
        print("Max eigenvalue < 1 → gradients will vanish over long sequences")

check_eigenvalues()
```

---

## 📊 The Two Regimes

```
┌─────────────────────────────────────────────────────────────────┐
│                    TWO GRADIENT REGIMES                         │
├──────────────────────────┬──────────────────────────────────────┤
│    VANISHING GRADIENTS   │      EXPLODING GRADIENTS             │
├──────────────────────────┼──────────────────────────────────────┤
│ |λ_max(W_hh)| < 1       │ |λ_max(W_hh)| > 1                  │
│ Gradients → 0            │ Gradients → ∞                       │
│ exponentially            │ exponentially                       │
│                          │                                      │
│ Effect: Cannot learn     │ Effect: NaN losses,                  │
│ long-range dependencies  │ training diverges                   │
│                          │                                      │
│ Solution: LSTM, GRU      │ Solution: Gradient clipping          │
│ (additive paths)         │ (cap gradient norm)                 │
│                          │                                      │
│ Harder to detect         │ Easy to detect (NaN, huge loss)     │
│ (training just stalls)   │                                      │
└──────────────────────────┴──────────────────────────────────────┘
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Gradient Chain | Product of Jacobians ∂h_{j+1}/∂h_j across time |
| Each Factor | diag(1-h²) · W_hh — tanh derivative times recurrent weight |
| Eigenvalues | Determine whether repeated multiplication grows or shrinks |
| \|λ\| < 1 | Gradients vanish exponentially with distance |
| \|λ\| > 1 | Gradients explode exponentially with distance |
| tanh Saturation | When \|h\| ≈ 1, tanh'≈ 0, gradient killed at that step |
| Critical Boundary | σ_max(W_hh) = 1 — gradients neither grow nor shrink |

---

## ❓ Revision Questions

1. **Write the product of Jacobians that determines gradient flow from time T back to time k. What are the two components of each Jacobian?**

2. **If W_hh has largest eigenvalue magnitude 0.95, what fraction of the gradient remains after 50 time steps? After 100?**

3. **Why does tanh saturation make the vanishing gradient problem worse?**

4. **How can you check whether an RNN is likely to have vanishing or exploding gradients by examining W_hh?**

5. **Explain why the gradient of the loss at t=100 w.r.t. input at t=1 involves 99 matrix multiplications.**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [BPTT Algorithm](01-bptt-algorithm.md) |
| ➡️ Next | [Vanishing Gradient Problem](03-vanishing-gradient-problem.md) |
| ⬆️ Unit Overview | [README](../README.md) |
