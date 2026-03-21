# Cell State

> **Unit 4, Chapter 4** — C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t: the cell state update, information highway, and why additive updates solve vanishing gradients.

---

## 📋 Overview

The **cell state** C_t is the key innovation of LSTM. It runs through the entire sequence like a conveyor belt, with information being selectively added or removed by gates. The crucial insight is that updates are **additive**, creating a gradient highway that avoids the vanishing gradient problem.

---

## 🛣️ The Cell State Highway

```
C₀ ────▶ C₁ ────▶ C₂ ────▶ C₃ ────▶ C₄ ────▶ C₅
         ×f₁+i₁C̃₁  ×f₂+i₂C̃₂  ×f₃+i₃C̃₃  ×f₄+i₄C̃₄  ×f₅+i₅C̃₅

When forget gates ≈ 1 and input gates ≈ 0:
C₀ ≈ C₁ ≈ C₂ ≈ C₃ ≈ C₄ ≈ C₅
Information passes through UNCHANGED!

    ┌───────────────────────────────────────────────┐
    │       CELL STATE = INFORMATION HIGHWAY        │
    │                                               │
    │  Like a conveyor belt in a factory:           │
    │  Items (information) travel along the belt    │
    │  Workers (gates) can:                         │
    │    • Remove items (forget gate × 0)           │
    │    • Add items (input gate × candidate)       │
    │    • Leave items untouched (forget ≈ 1)       │
    │                                               │
    │  The belt itself doesn't transform anything   │
    │  — only the gates modify what's on it.        │
    └───────────────────────────────────────────────┘
```

---

## 🧮 The Cell State Update Equation

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  C_t = f_t ⊙ C_{t-1}  +  i_t ⊙ C̃_t                      │
│        ──────────────     ──────────────                    │
│        "what to keep"     "what to add"                    │
│                                                             │
│  f_t ⊙ C_{t-1}:  Element-wise multiply previous cell state│
│                   by forget gate values (0 to 1)           │
│                                                             │
│  i_t ⊙ C̃_t:     Element-wise multiply candidate values    │
│                   by input gate values (0 to 1)            │
│                                                             │
│  Result: selective erase + selective write                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📐 Why ADDITIVE Updates Matter

### Vanilla RNN: Multiplicative

```
h_t = tanh(W · h_{t-1} + ...)

Gradient: ∂h_t/∂h_{t-1} = diag(tanh') · W

After T steps: ∏ (tanh' · W)  → VANISHES or EXPLODES
```

### LSTM: Additive

```
C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t

Gradient: ∂C_t/∂C_{t-1} = diag(f_t)

After T steps: ∏(t=1 to T) diag(f_t)

If f_t ≈ 1 for all t:
   ∏ diag(f_t) ≈ I  (identity matrix!)
   Gradient = 1 → NO VANISHING!

Key difference:
┌──────────────────────────────────────────────┐
│ RNN:  gradient ∝ (W · tanh')^T              │
│       → product of MATRICES → vanishes       │
│                                              │
│ LSTM: gradient ∝ ∏ f_t                       │
│       → product of SCALARS near 1            │
│       → can stay close to 1!                 │
└──────────────────────────────────────────────┘
```

---

## 🔢 Worked Example

```
Setup: hidden_size = 3

C_{t-1} = [2.0, -1.5, 0.8]     Previous cell state

Gates and candidate:
   f_t  = [0.9, 0.1, 0.95]     Forget gate
   i_t  = [0.3, 0.8, 0.1]      Input gate
   C̃_t  = [0.5, 1.0, -0.7]     Candidate values

Step 1: What to KEEP
   f_t ⊙ C_{t-1} = [0.9×2.0, 0.1×(-1.5), 0.95×0.8]
                  = [1.80,    -0.15,        0.76]

   Interpretation:
     • Cell[0]: Keep 90% of 2.0 → 1.80 (mostly remembered)
     • Cell[1]: Keep 10% of -1.5 → -0.15 (mostly forgotten!)
     • Cell[2]: Keep 95% of 0.8 → 0.76 (mostly remembered)

Step 2: What to ADD
   i_t ⊙ C̃_t = [0.3×0.5, 0.8×1.0, 0.1×(-0.7)]
              = [0.15,    0.80,     -0.07]

   Interpretation:
     • Cell[0]: Add small amount (0.15)
     • Cell[1]: Add large amount (0.80) — this cell is being rewritten!
     • Cell[2]: Tiny subtraction (-0.07)

Step 3: COMBINE
   C_t = [1.80 + 0.15,  -0.15 + 0.80,  0.76 + (-0.07)]
       = [1.95,          0.65,           0.69]

Summary:
   Cell[0]: 2.00 → 1.95  (preserved with small update)
   Cell[1]: -1.50 → 0.65 (dramatically changed — old info forgotten, new written)
   Cell[2]: 0.80 → 0.69  (mostly preserved with tiny adjustment)
```

---

## 📊 Cell State Behavior Patterns

```
┌──────────────────────────────────────────────────────────────────┐
│ Pattern          │ f_t    │ i_t    │ Effect on C_t              │
├──────────────────┼────────┼────────┼────────────────────────────┤
│ Remember         │ ≈ 1    │ ≈ 0    │ C_t ≈ C_{t-1} (preserve)  │
│ Forget           │ ≈ 0    │ ≈ 0    │ C_t ≈ 0 (erase)           │
│ Overwrite        │ ≈ 0    │ ≈ 1    │ C_t ≈ C̃_t (replace)      │
│ Accumulate       │ ≈ 1    │ > 0    │ C_t ≈ C_{t-1} + new       │
│ Partial forget   │ (0,1)  │ (0,1)  │ Blend old and new          │
└──────────────────┴────────┴────────┴────────────────────────────┘

Most powerful capability: SELECTIVE memory per dimension
   Some dimensions remember, others forget, simultaneously!
```

---

## 💻 Visualization of Cell State Over Time

```python
import torch
import torch.nn as nn
import numpy as np

# Track cell state through a sequence
lstm = nn.LSTM(input_size=5, hidden_size=8, batch_first=True)

# Meaningful input: signal at position 0, noise elsewhere
T = 30
x = torch.randn(1, T, 5) * 0.1  # low noise
x[0, 0, :] = torch.tensor([1.0, -1.0, 0.5, -0.5, 0.8])  # strong signal at t=0

# Collect cell states at each step
h = torch.zeros(1, 1, 8)
c = torch.zeros(1, 1, 8)
cell_states = [c.squeeze().detach().numpy().copy()]

for t in range(T):
    out, (h, c) = lstm(x[:, t:t+1, :], (h, c))
    cell_states.append(c.squeeze().detach().numpy().copy())

cell_states = np.array(cell_states)

print("=== Cell State Evolution ===")
print(f"Shape: {cell_states.shape}")  # (31, 8)
print(f"\nCell state norms over time:")
for t in [0, 1, 5, 10, 20, 30]:
    norm = np.linalg.norm(cell_states[t])
    print(f"  t={t:>2}: ||C_t|| = {norm:.4f}  C_t = {cell_states[t][:4].round(3)}...")

# Check: does signal from t=0 persist?
print(f"\nCorrelation between C_1 and C_30: "
      f"{np.corrcoef(cell_states[1], cell_states[30])[0,1]:.4f}")
print("High correlation = cell state preserved information!")
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Cell State Update | C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t |
| Additive Update | Addition creates gradient highway |
| Multiplicative Gating | f_t and i_t control flow with element-wise multiply |
| Gradient Through Cell | ∂C_t/∂C_{t-1} = f_t (close to 1 → no vanishing) |
| Conveyor Belt | Cell state carries information unchanged when gates allow |
| Selective Memory | Different dimensions can remember/forget independently |
| Unbounded | Cell state values can grow (not squashed by tanh) |

---

## ❓ Revision Questions

1. **Write the cell state update equation. Why is it an addition rather than a function like tanh?**

2. **Given C_{t-1} = [3.0, -2.0], f_t = [1.0, 0.0], i_t = [0.5, 0.5], C̃_t = [1.0, 1.0], compute C_t.**

3. **Compute ∂C_t/∂C_{t-1}. Why is this so much better for gradient flow than the vanilla RNN's ∂h_t/∂h_{t-1}?**

4. **What happens to the cell state if f_t = 1 and i_t = 0 for 100 consecutive time steps?**

5. **Why can the cell state be unbounded (not limited to [-1,1])? Is this a problem?**

6. **Explain how the cell state update combines "selective forgetting" and "selective writing" in a single equation.**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Gates: Forget, Input, Output](03-gates-forget-input-output.md) |
| ➡️ Next | [Gate Equations](05-gate-equations.md) |
| ⬆️ Unit Overview | [README](../README.md) |
