# Peephole Connections

> **Unit 4, Chapter 7** — Gates that can peek at the cell state: peephole LSTM equations, implementation, and when peepholes help.

---

## 📋 Overview

Standard LSTM gates only see the hidden state h_{t-1} and current input x_t. **Peephole connections** (Gers & Schmidhuber, 2000) allow gates to also look directly at the cell state C_{t-1}, giving them more information for gating decisions. This can help in tasks requiring precise timing or counting.

---

## 🔍 Standard vs Peephole LSTM

```
STANDARD LSTM gates:                    PEEPHOLE LSTM gates:
  f_t = σ(W_f·[h_{t-1}, x_t] + b_f)    f_t = σ(W_f·[h_{t-1}, x_t] + p_f⊙C_{t-1} + b_f)
                                                                         ────────────
                                                                         peephole!
  i_t = σ(W_i·[h_{t-1}, x_t] + b_i)    i_t = σ(W_i·[h_{t-1}, x_t] + p_i⊙C_{t-1} + b_i)

  o_t = σ(W_o·[h_{t-1}, x_t] + b_o)    o_t = σ(W_o·[h_{t-1}, x_t] + p_o⊙C_t + b_o)
                                                                         ─────
                                                                         uses C_t !

┌──────────────────────────────────────────────────────────────┐
│  "Peephole" = the gate can PEEK at the cell state          │
│                                                              │
│  Standard gate input:  [h_{t-1}, x_t]                       │
│  Peephole gate input:  [h_{t-1}, x_t, C_{t-1}]             │
│                          (or C_t for output gate)            │
│                                                              │
│  Note: p_f, p_i, p_o are DIAGONAL weight vectors,           │
│  not full matrices! This keeps parameter count low.         │
└──────────────────────────────────────────────────────────────┘
```

---

## 🧮 Complete Peephole LSTM Equations

```
┌────────────────────────────────────────────────────────────────────┐
│                PEEPHOLE LSTM EQUATIONS                             │
│                                                                    │
│  f_t = σ(W_f · [h_{t-1}, x_t] + p_f ⊙ C_{t-1} + b_f)           │
│  i_t = σ(W_i · [h_{t-1}, x_t] + p_i ⊙ C_{t-1} + b_i)           │
│  C̃_t = tanh(W_C · [h_{t-1}, x_t] + b_C)                         │
│  C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t                                │
│  o_t = σ(W_o · [h_{t-1}, x_t] + p_o ⊙ C_t + b_o)                │
│  h_t = o_t ⊙ tanh(C_t)                                           │
│                                                                    │
│  Additional parameters:                                            │
│    p_f, p_i, p_o ∈ ℝⁿ  (peephole weight vectors)                 │
│    Total new params: 3n                                            │
│                                                                    │
│  Note: Forget and input gates peek at C_{t-1} (before update)     │
│        Output gate peeks at C_t (after update)                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 📐 Why Peepholes Help

```
Problem without peepholes:
   Gates only see h_{t-1}, which is a FILTERED version of C_{t-1}:
   h_{t-1} = o_{t-1} ⊙ tanh(C_{t-1})

   If o_{t-1} ≈ 0: h_{t-1} ≈ 0, even if C_{t-1} has important values!
   Gates at time t are essentially "blind" to what's stored.

With peepholes:
   Gates directly see C_{t-1}, so they know exactly what's stored.
   Even if output gate was closed (h ≈ 0), the forget and input
   gates can still make informed decisions.

Use cases where this helps:
  1. Precise TIMING: Count to N then trigger
     → Gate needs to know current count (stored in C)
  
  2. THRESHOLD detection: Act when cell exceeds threshold
     → Gate must see actual cell value
  
  3. Mutual EXCLUSION: Don't write if cell already has value
     → Input gate checks C to avoid double-writing
```

---

## 📊 Architecture Comparison

```
┌─────────────────────┬──────────────┬──────────────────┐
│                     │ Standard     │ Peephole         │
├─────────────────────┼──────────────┼──────────────────┤
│ Gate inputs         │ [h_{t-1},x_t]│ [h_{t-1},x_t]+C │
│ Extra parameters    │ 0            │ 3n               │
│ Computational cost  │ Lower        │ Slightly higher  │
│ Timing tasks        │ Good         │ Better           │
│ Standard NLP        │ Good         │ ~Same            │
│ Most frameworks     │ Default      │ Optional         │
│ Used in practice?   │ Most common  │ Sometimes        │
└─────────────────────┴──────────────┴──────────────────┘
```

---

## 💻 Implementation

```python
import torch
import torch.nn as nn

class PeepholeLSTMCell(nn.Module):
    """LSTM cell with peephole connections"""
    def __init__(self, input_size, hidden_size):
        super().__init__()
        self.hidden_size = hidden_size
        
        combined = input_size + hidden_size
        
        # Standard weights
        self.W_f = nn.Linear(combined, hidden_size)
        self.W_i = nn.Linear(combined, hidden_size)
        self.W_c = nn.Linear(combined, hidden_size)
        self.W_o = nn.Linear(combined, hidden_size)
        
        # Peephole weights (diagonal, so just vectors)
        self.p_f = nn.Parameter(torch.randn(hidden_size) * 0.1)
        self.p_i = nn.Parameter(torch.randn(hidden_size) * 0.1)
        self.p_o = nn.Parameter(torch.randn(hidden_size) * 0.1)
        
        nn.init.ones_(self.W_f.bias)
    
    def forward(self, x_t, h_prev, c_prev):
        combined = torch.cat([h_prev, x_t], dim=1)
        
        # Gates with peephole connections
        f_t = torch.sigmoid(self.W_f(combined) + self.p_f * c_prev)  # Peek at C_{t-1}
        i_t = torch.sigmoid(self.W_i(combined) + self.p_i * c_prev)  # Peek at C_{t-1}
        c_tilde = torch.tanh(self.W_c(combined))
        
        # Cell state update
        c_t = f_t * c_prev + i_t * c_tilde
        
        # Output gate peeks at C_t (AFTER update!)
        o_t = torch.sigmoid(self.W_o(combined) + self.p_o * c_t)     # Peek at C_t
        h_t = o_t * torch.tanh(c_t)
        
        return h_t, c_t

# Compare parameter counts
input_size, hidden_size = 10, 20
standard = nn.LSTMCell(input_size, hidden_size)
peephole = PeepholeLSTMCell(input_size, hidden_size)

std_params = sum(p.numel() for p in standard.parameters())
peep_params = sum(p.numel() for p in peephole.parameters())

print(f"Standard LSTM params:  {std_params}")
print(f"Peephole LSTM params:  {peep_params}")
print(f"Extra params:          {peep_params - std_params} (= 3 × {hidden_size})")
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Peephole | Gate can directly see cell state C |
| p_f, p_i, p_o | Diagonal peephole weight vectors (not full matrices) |
| f_t, i_t peek at | C_{t-1} (before cell update) |
| o_t peeks at | C_t (after cell update) |
| Extra params | Only 3n (minimal overhead) |
| When helps | Timing, counting, threshold tasks |
| When not needed | Standard NLP tasks |
| Default? | No — standard LSTM is default in most frameworks |

---

## ❓ Revision Questions

1. **What is a peephole connection? What additional information does it provide to the gates?**

2. **Write the peephole forget gate equation. How does it differ from the standard forget gate?**

3. **Why does the output gate peek at C_t (after update) while forget and input gates peek at C_{t-1} (before update)?**

4. **Peephole weights p_f, p_i, p_o are vectors, not matrices. Why? How many extra parameters does this add?**

5. **Give an example task where peephole connections would help that standard LSTM might struggle with.**

6. **In Greff et al. (2015)'s large-scale LSTM variant study, what did they conclude about peephole connections?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [How LSTM Solves Vanishing Gradient](06-how-lstm-solves-vanishing.md) |
| ➡️ Next | [GRU Architecture](../05-GRU/01-gru-architecture.md) |
| ⬆️ Unit Overview | [README](../README.md) |
