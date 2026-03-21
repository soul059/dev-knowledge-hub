# Reset and Update Gates

> **Unit 5, Chapter 2** — Detailed analysis of the reset gate r_t and update gate z_t: formulas, intuition, and how they control information flow.

---

## 📋 Overview

The GRU uses just two gates — **reset** and **update** — to achieve functionality similar to LSTM's three gates. The update gate controls the balance between old and new information, while the reset gate determines how much past state influences new candidate values.

---

## 🔄 Update Gate (z_t)

```
z_t = σ(W_z · [h_{t-1}, x_t] + b_z)

┌──────────────────────────────────────────────────────────────┐
│                    UPDATE GATE                                │
│                                                              │
│  z_t ∈ (0,1)ⁿ — one value per hidden dimension             │
│                                                              │
│  Controls the interpolation between old and new:            │
│    h_t = (1 - z_t) ⊙ h_{t-1} + z_t ⊙ h̃_t                 │
│           ─────────────────    ─────────────                │
│           "keep old"           "use new"                    │
│                                                              │
│  z_t ≈ 0:  h_t ≈ h_{t-1}     (KEEP old, ignore new)       │
│  z_t ≈ 1:  h_t ≈ h̃_t         (USE new, forget old)        │
│  z_t = 0.5: h_t = average of old and new                   │
│                                                              │
│  Key insight: (1-z_t) + z_t = 1 ALWAYS                     │
│  This is a CONVEX COMBINATION — output is bounded           │
│  between h_{t-1} and h̃_t                                   │
└──────────────────────────────────────────────────────────────┘

Analogy:
   z_t is like a MIXING KNOB:
   
   Keep old ◀──────────●──────────▶ Use new
   z_t=0                               z_t=1
```

### How Update Gate Replaces LSTM's Forget + Input

```
LSTM:
   C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t
   f_t and i_t are INDEPENDENT
   Can have f_t=1, i_t=1 (keep AND add)
   Can have f_t=0, i_t=0 (erase AND don't add)

GRU:
   h_t = (1-z_t) ⊙ h_{t-1} + z_t ⊙ h̃_t
   (1-z_t) and z_t are COUPLED
   ALWAYS: keep_amount + new_amount = 1
   Cannot keep AND add simultaneously (trade-off enforced)
```

---

## 🔁 Reset Gate (r_t)

```
r_t = σ(W_r · [h_{t-1}, x_t] + b_r)

┌──────────────────────────────────────────────────────────────┐
│                    RESET GATE                                │
│                                                              │
│  r_t ∈ (0,1)ⁿ — one value per hidden dimension             │
│                                                              │
│  Controls how much past state influences the CANDIDATE:     │
│    h̃_t = tanh(W_h · [r_t ⊙ h_{t-1}, x_t] + b_h)           │
│                       ──────────────                        │
│                       reset applied HERE                    │
│                                                              │
│  r_t ≈ 1:  h̃_t sees full h_{t-1}  (normal operation)       │
│  r_t ≈ 0:  h̃_t ignores h_{t-1}    (fresh start!)           │
│                                                              │
│  When r_t ≈ 0, the candidate is computed as if there       │
│  is NO history — like starting from scratch.               │
│  This enables HARD RESETS of the hidden state.             │
│                                                              │
│  The reset gate allows the GRU to "drop" irrelevant        │
│  past information when computing what to potentially       │
│  write next.                                                │
└──────────────────────────────────────────────────────────────┘
```

---

## 📐 Gate Interaction Diagram

```
Step 1: Compute BOTH gates from same input
   [h_{t-1}, x_t] ──┬──▶ σ ──▶ z_t (update gate)
                     └──▶ σ ──▶ r_t (reset gate)

Step 2: Reset gate modifies past for candidate
   r_t ⊙ h_{t-1} ──▶ combined with x_t ──▶ tanh ──▶ h̃_t

Step 3: Update gate interpolates
   h_t = (1-z_t) ⊙ h_{t-1} + z_t ⊙ h̃_t

Four behavioral modes:
┌──────────────────────────────────────────────────────────────┐
│ z_t ≈ 0, r_t ≈ any:  h_t ≈ h_{t-1}  (COPY — ignore input) │
│ z_t ≈ 1, r_t ≈ 1:    h_t ≈ tanh(W·[h_{t-1},x_t])  (RNN)  │
│ z_t ≈ 1, r_t ≈ 0:    h_t ≈ tanh(W·[0,x_t])  (RESET)       │
│ z_t ≈ 0.5, r_t ≈ 1:  h_t = avg(old, new)  (BLEND)         │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔢 Worked Example

```
Setup: input_size=2, hidden_size=2

h_{t-1} = [0.8, -0.5]     Previous hidden state
x_t     = [1.0, 0.3]      Current input

Suppose gates compute to:
   z_t = [0.3, 0.9]       Update gate
   r_t = [0.1, 0.8]       Reset gate

Step 1: Apply reset gate
   r_t ⊙ h_{t-1} = [0.1 × 0.8, 0.8 × (-0.5)]
                  = [0.08, -0.40]
   
   Dim 0: Past almost IGNORED (r=0.1, only 10% of h visible)
   Dim 1: Past mostly VISIBLE (r=0.8, 80% of h visible)

Step 2: Compute candidate (assume result after tanh)
   h̃_t = tanh(W · [0.08, -0.40, 1.0, 0.3] + b)
        ≈ [0.6, 0.2]  (hypothetical)

Step 3: Interpolate with update gate
   h_t = (1-z_t) ⊙ h_{t-1} + z_t ⊙ h̃_t
       = [0.7 × 0.8, 0.1 × (-0.5)] + [0.3 × 0.6, 0.9 × 0.2]
       = [0.56, -0.05] + [0.18, 0.18]
       = [0.74, 0.13]

Interpretation:
   Dim 0: Mostly kept old (z=0.3) → 0.74 close to old 0.8
   Dim 1: Mostly used new (z=0.9) → 0.13 close to new 0.2
```

---

## 💻 Gate Visualization

```python
import torch
import torch.nn as nn
import numpy as np

# Observe gate values during sequence processing
gru = nn.GRU(input_size=5, hidden_size=8, batch_first=True)

# Input with a pattern change
T = 20
x = torch.zeros(1, T, 5)
x[0, :10, 0] = 1.0   # Signal in first half
x[0, 10:, 1] = 1.0   # Different signal in second half

# Hook to capture gate values
gate_values = {'update': [], 'reset': []}

h = torch.zeros(1, 1, 8)
for t in range(T):
    out, h = gru(x[:, t:t+1, :], h)
    
    # Extract gate values (approximate from weight analysis)
    h_val = h.squeeze().detach().numpy()
    gate_values['update'].append(h_val[:4].mean())
    gate_values['reset'].append(h_val[4:].mean())

print("=== GRU Gate Behavior Over Time ===")
print("Pattern changes at t=10")
print(f"{'Time':>4} {'Hidden State Mean':>20}")
for t in range(T):
    with torch.no_grad():
        out, _ = gru(x[:, :t+1, :])
        h_mean = out[0, -1].mean().item()
    marker = " ← pattern change" if t == 10 else ""
    print(f"t={t:>2}  h_mean = {h_mean:>8.4f}{marker}")
```

---

## 📋 Summary Table

| Gate | Formula | z/r ≈ 0 | z/r ≈ 1 |
|------|---------|---------|---------|
| Update (z_t) | σ(W_z·[h,x]+b_z) | Keep old h_{t-1} | Use new h̃_t |
| Reset (r_t) | σ(W_r·[h,x]+b_r) | Ignore h_{t-1} in candidate | Use full h_{t-1} |

---

## ❓ Revision Questions

1. **In the update equation h_t = (1-z_t)⊙h_{t-1} + z_t⊙h̃_t, prove that each element of h_t is always between the corresponding elements of h_{t-1} and h̃_t.**

2. **Why does the GRU's coupled gating (1-z and z) mean it can't independently forget AND not add new information?**

3. **What is the effect of r_t ≈ 0 on the candidate h̃_t? When would this be useful?**

4. **Given h_{t-1}=[1.0, -0.5], z_t=[0.0, 1.0], h̃_t=[0.3, 0.8], compute h_t.**

5. **Compare the update gate's interpolation to LSTM's f_t and i_t. What capability does LSTM have that GRU doesn't?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [GRU Architecture](01-gru-architecture.md) |
| ➡️ Next | [GRU Equations](03-gru-equations.md) |
| ⬆️ Unit Overview | [README](../README.md) |
