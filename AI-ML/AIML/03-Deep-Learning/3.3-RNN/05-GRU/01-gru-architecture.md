# GRU Architecture

> **Unit 5, Chapter 1** — The Gated Recurrent Unit: simplified gating with 2 gates instead of 3, reset and update gates, and ASCII cell diagram.

---

## 📋 Overview

The **Gated Recurrent Unit (GRU)**, introduced by Cho et al. (2014), is a simplified alternative to LSTM. It combines the forget and input gates into a single **update gate** and merges the cell state and hidden state into one. Despite having fewer parameters, GRU often achieves comparable performance to LSTM.

---

## 🏗️ GRU Cell Diagram

```
┌────────────────────────────────────────────────────────────┐
│                    GRU CELL at time t                      │
│                                                            │
│  h_{t-1} ──┬──────────[×]──────────[+]──────── h_t        │
│             │           ↑   (1-z_t)  ↑ z_t                │
│             │           │            │                     │
│             │        h_{t-1}       h̃_t                    │
│             │                    (candidate)               │
│             │                        ↑                     │
│             │                     ┌──┴──┐                  │
│             │                     │tanh │                  │
│             │                     └──┬──┘                  │
│             │                        │                     │
│             │              ┌─────────┴──────────┐          │
│             │              │  W · [r_t⊙h_{t-1}, x_t]  │  │
│             │              └────────────────────────┘      │
│             │                        ↑                     │
│             │                      r_t (reset gate)        │
│             │                        ↑                     │
│             │                     ┌──┴──┐                  │
│             │                     │  σ  │                  │
│             │                     └──┬──┘                  │
│             │                        │                     │
│             ├────────────────────────┤                     │
│             │                        │                     │
│             │              z_t (update gate)               │
│             │                        ↑                     │
│             │                     ┌──┴──┐                  │
│             │                     │  σ  │                  │
│             │                     └──┬──┘                  │
│             │                        │                     │
│  x_t ───────┴────────────────────────┘                     │
│                                                            │
└────────────────────────────────────────────────────────────┘

Simplified flow:
                 (1 - z_t)
   h_{t-1} ────────[×]─────┐
       │                    │
       │               ┌────[+]────▶ h_t
       │               │
       └──▶ [candidate] ──[× z_t]──┘
              h̃_t
```

---

## 📊 GRU vs LSTM: Structural Comparison

```
┌─────────────────────┬──────────────────┬──────────────────┐
│                     │ LSTM             │ GRU              │
├─────────────────────┼──────────────────┼──────────────────┤
│ State vectors       │ 2 (C_t and h_t)  │ 1 (h_t only)     │
│ Gates               │ 3 (f, i, o)      │ 2 (z, r)         │
│ Cell state          │ Yes (separate)   │ No (merged)      │
│ Output gate         │ Yes              │ No               │
│ Parameters (d=n)    │ 4n(2n+1)         │ 3n(2n+1)         │
│ Parameter ratio     │ 1.0x             │ 0.75x            │
│ Gating mechanism    │ Separate forget  │ Coupled update   │
│                     │ and input gates  │ gate (z and 1-z) │
└─────────────────────┴──────────────────┴──────────────────┘
```

---

## 🔑 Key Design Decisions

```
1. MERGE cell state and hidden state:
   LSTM: C_t (memory) + h_t (output) → 2 state vectors
   GRU:  h_t serves both roles → 1 state vector

2. COUPLE forget and input:
   LSTM: f_t and i_t are independent
         Can simultaneously forget AND not add (f=0, i=0)
         Can simultaneously keep AND add (f=1, i=1)
   
   GRU:  z_t controls both
         If z_t ≈ 1 → keep old (like f=1, i=0)
         If z_t ≈ 0 → use new (like f=0, i=1)
         ALWAYS: keep_fraction + new_fraction = 1

3. REMOVE output gate:
   LSTM: o_t filters what's exposed from C_t
   GRU:  Full h_t is exposed (no filtering)

4. ADD reset gate:
   r_t controls how much of h_{t-1} influences the candidate
   When r_t ≈ 0: candidate ignores past → fresh start
   When r_t ≈ 1: candidate sees full past → standard update
```

---

## 🧮 GRU Equations

```
┌─────────────────────────────────────────────────────────────┐
│                    GRU EQUATIONS                            │
│                                                             │
│  1. Update gate:                                           │
│     z_t = σ(W_z · [h_{t-1}, x_t] + b_z)                  │
│                                                             │
│  2. Reset gate:                                            │
│     r_t = σ(W_r · [h_{t-1}, x_t] + b_r)                  │
│                                                             │
│  3. Candidate hidden state:                                │
│     h̃_t = tanh(W_h · [r_t ⊙ h_{t-1}, x_t] + b_h)        │
│                        ──────────────                      │
│                        reset gate applied HERE             │
│                                                             │
│  4. Hidden state update:                                   │
│     h_t = (1 - z_t) ⊙ h_{t-1} + z_t ⊙ h̃_t               │
│           ────────────────────   ──────────                │
│           keep from past          add from candidate       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn

class GRUCellManual(nn.Module):
    """GRU cell from scratch"""
    def __init__(self, input_size, hidden_size):
        super().__init__()
        combined = input_size + hidden_size
        self.W_z = nn.Linear(combined, hidden_size)  # Update gate
        self.W_r = nn.Linear(combined, hidden_size)  # Reset gate
        self.W_h = nn.Linear(combined, hidden_size)  # Candidate
        self.hidden_size = hidden_size
    
    def forward(self, x_t, h_prev):
        combined = torch.cat([h_prev, x_t], dim=1)
        
        z_t = torch.sigmoid(self.W_z(combined))       # Update gate
        r_t = torch.sigmoid(self.W_r(combined))       # Reset gate
        
        # Reset gate applied to h_prev before computing candidate
        combined_reset = torch.cat([r_t * h_prev, x_t], dim=1)
        h_tilde = torch.tanh(self.W_h(combined_reset))  # Candidate
        
        # Interpolate between old and new
        h_t = (1 - z_t) * h_prev + z_t * h_tilde
        
        return h_t

# Compare with PyTorch built-in
gru_manual = GRUCellManual(10, 20)
gru_builtin = nn.GRUCell(10, 20)

x = torch.randn(4, 10)
h = torch.zeros(4, 20)

h_manual = gru_manual(x, h)
h_builtin = gru_builtin(x, h)

print(f"Manual GRU output:  {h_manual.shape}")
print(f"PyTorch GRU output: {h_builtin.shape}")

# Parameter counts
manual_params = sum(p.numel() for p in gru_manual.parameters())
builtin_params = sum(p.numel() for p in gru_builtin.parameters())
lstm_params = sum(p.numel() for p in nn.LSTMCell(10, 20).parameters())
print(f"\nGRU parameters:  {builtin_params}")
print(f"LSTM parameters: {lstm_params}")
print(f"GRU is {100*builtin_params/lstm_params:.0f}% of LSTM parameters")
```

---

## 📋 Summary Table

| Component | Formula | Purpose |
|-----------|---------|---------|
| Update gate z_t | σ(W_z·[h_{t-1},x_t]+b_z) | How much to update vs keep |
| Reset gate r_t | σ(W_r·[h_{t-1},x_t]+b_r) | How much past to use in candidate |
| Candidate h̃_t | tanh(W_h·[r_t⊙h_{t-1},x_t]+b_h) | Proposed new state |
| State update | (1-z_t)⊙h_{t-1} + z_t⊙h̃_t | Interpolation of old and new |

---

## ❓ Revision Questions

1. **Draw the GRU cell diagram showing both gates and the candidate computation.**

2. **How does the GRU's update gate combine the roles of LSTM's forget and input gates?**

3. **What happens when z_t = 0 everywhere? When z_t = 1 everywhere?**

4. **Why does the GRU need only one state vector while LSTM needs two?**

5. **How many parameters does a GRU with input_size=50, hidden_size=100 have?**

6. **Explain the role of the reset gate. Why is it applied BEFORE the candidate computation?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Peephole Connections](../04-LSTM/07-peephole-connections.md) |
| ➡️ Next | [Reset and Update Gates](02-reset-and-update-gates.md) |
| ⬆️ Unit Overview | [README](../README.md) |
