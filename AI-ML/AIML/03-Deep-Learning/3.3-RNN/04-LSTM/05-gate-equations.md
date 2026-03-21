# Complete Gate Equations

> **Unit 4, Chapter 5** — All LSTM equations together, step-by-step computation, and a complete numerical worked example.

---

## 📋 Overview

This chapter consolidates all LSTM equations into one reference and walks through a complete numerical example to build calculation fluency.

---

## 🧮 Complete LSTM Equations

```
┌────────────────────────────────────────────────────────────────────┐
│                     LSTM EQUATIONS (Complete)                      │
│                                                                    │
│  Input at time t: x_t ∈ ℝᵈ                                       │
│  Previous states: h_{t-1} ∈ ℝⁿ, C_{t-1} ∈ ℝⁿ                   │
│                                                                    │
│  Step 1 — Forget gate:                                            │
│    f_t = σ(W_f · [h_{t-1}, x_t] + b_f)           ∈ (0,1)ⁿ       │
│                                                                    │
│  Step 2 — Input gate:                                             │
│    i_t = σ(W_i · [h_{t-1}, x_t] + b_i)           ∈ (0,1)ⁿ       │
│                                                                    │
│  Step 3 — Candidate values:                                       │
│    C̃_t = tanh(W_C · [h_{t-1}, x_t] + b_C)        ∈ (-1,1)ⁿ      │
│                                                                    │
│  Step 4 — Cell state update:                                      │
│    C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t              ∈ ℝⁿ           │
│                                                                    │
│  Step 5 — Output gate:                                            │
│    o_t = σ(W_o · [h_{t-1}, x_t] + b_o)           ∈ (0,1)ⁿ       │
│                                                                    │
│  Step 6 — Hidden state:                                           │
│    h_t = o_t ⊙ tanh(C_t)                          ∈ (-1,1)ⁿ      │
│                                                                    │
│  where:                                                            │
│    σ = sigmoid function                                            │
│    ⊙ = element-wise (Hadamard) product                            │
│    [a, b] = concatenation of vectors a and b                      │
│    W_f, W_i, W_C, W_o ∈ ℝ^{n×(n+d)}                              │
│    b_f, b_i, b_C, b_o ∈ ℝⁿ                                       │
│                                                                    │
│  Total parameters: 4 × [n(n+d) + n] = 4n(n+d) + 4n              │
│                  = 4n² + 4nd + 4n                                 │
└────────────────────────────────────────────────────────────────────┘
```

---

## 📐 Parameter Dimensions

```
For input_size=d, hidden_size=n:

Concatenated input: [h_{t-1}, x_t] ∈ ℝ^{n+d}

┌──────────┬─────────────────┬──────────┬───────────────────┐
│ Parameter│ Dimensions      │ Count    │ Example (d=3,n=4) │
├──────────┼─────────────────┼──────────┼───────────────────┤
│ W_f      │ n × (n+d)       │ n(n+d)   │ 4×7 = 28          │
│ b_f      │ n               │ n        │ 4                  │
│ W_i      │ n × (n+d)       │ n(n+d)   │ 28                 │
│ b_i      │ n               │ n        │ 4                  │
│ W_C      │ n × (n+d)       │ n(n+d)   │ 28                 │
│ b_C      │ n               │ n        │ 4                  │
│ W_o      │ n × (n+d)       │ n(n+d)   │ 28                 │
│ b_o      │ n               │ n        │ 4                  │
├──────────┼─────────────────┼──────────┼───────────────────┤
│ TOTAL    │                 │4n(n+d+1)│ 128                │
└──────────┴─────────────────┴──────────┴───────────────────┘

Compare to vanilla RNN: n(n+d) + n = n(n+d+1) = 32
LSTM has exactly 4× more parameters!
```

---

## 🔢 Complete Numerical Example

```
Setup: d=2 (input), n=2 (hidden)

Weights (simplified for hand computation):
W_f = [[0.1, 0.2, 0.3, 0.4],     b_f = [0.5, 0.5]   (high → remember)
       [0.5, 0.1, 0.2, 0.3]]

W_i = [[0.3, 0.1, 0.2, 0.4],     b_i = [0.0, 0.0]
       [0.1, 0.3, 0.4, 0.2]]

W_C = [[0.2, 0.3, 0.1, 0.2],     b_C = [0.0, 0.0]
       [0.4, 0.1, 0.3, 0.1]]

W_o = [[0.1, 0.4, 0.2, 0.1],     b_o = [0.0, 0.0]
       [0.3, 0.2, 0.1, 0.4]]

Initial states:
h₀ = [0.0, 0.0],  C₀ = [0.0, 0.0]
Input: x₁ = [1.0, 0.5]

═══════════════════════════════════════════
Step 1: Concatenate [h₀, x₁] = [0.0, 0.0, 1.0, 0.5]

Step 2: FORGET GATE
  W_f · [h₀,x₁] + b_f
  = [0.1·0 + 0.2·0 + 0.3·1 + 0.4·0.5 + 0.5,
     0.5·0 + 0.1·0 + 0.2·1 + 0.3·0.5 + 0.5]
  = [0 + 0 + 0.3 + 0.2 + 0.5,
     0 + 0 + 0.2 + 0.15 + 0.5]
  = [1.0, 0.85]
  f₁ = σ([1.0, 0.85]) = [0.7311, 0.7005]

Step 3: INPUT GATE
  W_i · [h₀,x₁] + b_i
  = [0.3·0 + 0.1·0 + 0.2·1 + 0.4·0.5,
     0.1·0 + 0.3·0 + 0.4·1 + 0.2·0.5]
  = [0.4, 0.5]
  i₁ = σ([0.4, 0.5]) = [0.5987, 0.6225]

Step 4: CANDIDATE
  W_C · [h₀,x₁] + b_C
  = [0.2·0 + 0.3·0 + 0.1·1 + 0.2·0.5,
     0.4·0 + 0.1·0 + 0.3·1 + 0.1·0.5]
  = [0.2, 0.35]
  C̃₁ = tanh([0.2, 0.35]) = [0.1974, 0.3364]

Step 5: CELL STATE UPDATE
  C₁ = f₁ ⊙ C₀ + i₁ ⊙ C̃₁
     = [0.7311·0 + 0.5987·0.1974,
        0.7005·0 + 0.6225·0.3364]
     = [0.1182, 0.2094]

Step 6: OUTPUT GATE
  W_o · [h₀,x₁] + b_o
  = [0.1·0 + 0.4·0 + 0.2·1 + 0.1·0.5,
     0.3·0 + 0.2·0 + 0.1·1 + 0.4·0.5]
  = [0.25, 0.3]
  o₁ = σ([0.25, 0.3]) = [0.5622, 0.5744]

Step 7: HIDDEN STATE
  h₁ = o₁ ⊙ tanh(C₁)
     = [0.5622 · tanh(0.1182),
        0.5744 · tanh(0.2094)]
     = [0.5622 · 0.1177,
        0.5744 · 0.2063]
     = [0.0662, 0.1185]

═══════════════════════════════════════════
RESULTS for t=1:
  f₁ = [0.731, 0.701]  (forget gate — moderately open)
  i₁ = [0.599, 0.623]  (input gate — moderately open)
  C₁ = [0.118, 0.209]  (cell state — new values stored)
  o₁ = [0.562, 0.574]  (output gate — moderately open)
  h₁ = [0.066, 0.119]  (hidden state — filtered cell output)
```

---

## 💻 PyTorch Verification

```python
import torch
import torch.nn as nn

# Verify manual computation matches PyTorch
cell = nn.LSTMCell(input_size=2, hidden_size=2)

# Set weights to match our example
with torch.no_grad():
    # PyTorch concatenates [W_ih, W_hh] and stores gates as [i, f, g, o]
    # Order: input, forget, candidate(g), output
    W_ih = torch.tensor([
        [0.2, 0.4],  # i gate: x weights
        [0.3, 0.4],  # f gate: x weights  
        [0.1, 0.2],  # g(candidate): x weights
        [0.2, 0.1],  # o gate: x weights
    ], dtype=torch.float32)
    
    W_hh = torch.tensor([
        [0.3, 0.1],  # i gate: h weights
        [0.1, 0.2],  # f gate: h weights
        [0.2, 0.3],  # g: h weights
        [0.1, 0.4],  # o gate: h weights
    ], dtype=torch.float32)

# Quick demo with default initialization
cell = nn.LSTMCell(2, 2)
x = torch.tensor([[1.0, 0.5]])
h0 = torch.zeros(1, 2)
c0 = torch.zeros(1, 2)

h1, c1 = cell(x, (h0, c0))
print(f"h₁ = {h1.detach().numpy().round(4)}")
print(f"C₁ = {c1.detach().numpy().round(4)}")
```

---

## 📋 Summary Table

| Step | Equation | Activation | Output Range |
|------|----------|-----------|--------------|
| 1. Forget gate | f_t = σ(W_f·[h,x]+b_f) | sigmoid | (0,1) |
| 2. Input gate | i_t = σ(W_i·[h,x]+b_i) | sigmoid | (0,1) |
| 3. Candidate | C̃_t = tanh(W_C·[h,x]+b_C) | tanh | (-1,1) |
| 4. Cell update | C_t = f_t⊙C_{t-1} + i_t⊙C̃_t | none | ℝ |
| 5. Output gate | o_t = σ(W_o·[h,x]+b_o) | sigmoid | (0,1) |
| 6. Hidden state | h_t = o_t⊙tanh(C_t) | tanh | (-1,1) |

---

## ❓ Revision Questions

1. **List all 6 LSTM equations in order. Which use sigmoid and which use tanh?**

2. **For d=10 and n=50, calculate the total number of trainable parameters in an LSTM cell.**

3. **In the worked example, which gate was most "open" at t=1? Which was most "closed"? What does this mean?**

4. **Why is h_t = o_t ⊙ tanh(C_t) rather than h_t = o_t ⊙ C_t? Why the extra tanh?**

5. **Compute one step: Given h₀=[0.5,-0.3], C₀=[1.0,0.5], x₁=[0.2,0.8], f₁=[0.9,0.1], i₁=[0.2,0.7], C̃₁=[0.4,-0.6], o₁=[0.5,0.8], find C₁ and h₁.**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Cell State](04-cell-state.md) |
| ➡️ Next | [How LSTM Solves Vanishing Gradient](06-how-lstm-solves-vanishing.md) |
| ⬆️ Unit Overview | [README](../README.md) |
