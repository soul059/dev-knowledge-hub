# RNN Architecture

> **Unit 2, Chapter 1** — The anatomy of a Recurrent Neural Network cell: input, hidden state, output, weight matrices, and detailed ASCII diagrams.

---

## 📋 Overview

The RNN architecture is built around a single **recurrent cell** that is applied at every time step. This cell takes the current input and the previous hidden state, produces a new hidden state, and optionally generates an output. The magic lies in the three weight matrices: **W_xh**, **W_hh**, and **W_hy**.

---

## 🏗️ RNN Cell Anatomy

### Single Cell (Detailed)

```
                         x_t ∈ ℝᵈ
                          │
                          │ W_xh ∈ ℝⁿˣᵈ
                          ▼
                    ┌───────────┐
   h_{t-1} ∈ ℝⁿ ──▶│           │──▶ h_t ∈ ℝⁿ
       │    W_hh    │   tanh    │
       │    ∈ℝⁿˣⁿ  │           │
                    └─────┬─────┘
                          │
                          │ W_hy ∈ ℝᵐˣⁿ
                          ▼
                        y_t ∈ ℝᵐ

   ┌──────────────────────────────────────────┐
   │  h_t = tanh(W_xh · x_t + W_hh · h_{t-1} + b_h)  │
   │  y_t = W_hy · h_t + b_y                           │
   └──────────────────────────────────────────┘
```

### Detailed Internal Computation

```
                    x_t (input)
                     │
         ┌───────────┤
         │           │
         │     ┌─────▼─────┐
         │     │  W_xh · x_t │──────┐
         │     └───────────┘      │
         │                        ▼
         │                   ┌─────────┐
h_{t-1}──┤                   │   (+)   │──▶ tanh ──▶ h_t
         │                   └─────────┘              │
         │                        ▲                   │
         │     ┌────────────┐     │                   │
         └────▶│W_hh·h_{t-1}│─────┘              ┌───▼───┐
               └────────────┘                     │W_hy·h_t│
                                                  └───┬───┘
                   b_h added                          │
                   before tanh                        ▼
                                                     y_t
```

---

## 📊 The Three Weight Matrices

```
┌─────────────────────────────────────────────────────────────┐
│                    WEIGHT MATRICES                          │
├─────────┬──────────────┬────────────┬──────────────────────┤
│ Matrix  │ Dimensions   │ Connects   │ Purpose              │
├─────────┼──────────────┼────────────┼──────────────────────┤
│ W_xh    │ n × d        │ Input →    │ Transform input into │
│         │              │ Hidden     │ hidden space         │
├─────────┼──────────────┼────────────┼──────────────────────┤
│ W_hh    │ n × n        │ Hidden →   │ Transform previous   │
│         │              │ Hidden     │ hidden state         │
│         │              │            │ (THE RECURRENCE!)    │
├─────────┼──────────────┼────────────┼──────────────────────┤
│ W_hy    │ m × n        │ Hidden →   │ Map hidden state to  │
│         │              │ Output     │ output space         │
└─────────┴──────────────┴────────────┴──────────────────────┘

Where: d = input dimension
       n = hidden dimension (hyperparameter)
       m = output dimension

Total trainable parameters:
   W_xh: n·d    W_hh: n·n    W_hy: m·n    b_h: n    b_y: m
   Total = n·d + n² + m·n + n + m
```

---

## 🔄 Unrolled Through Time

```
  x₁          x₂          x₃          x₄
   │           │           │           │
   │W_xh       │W_xh       │W_xh       │W_xh
   ▼           ▼           ▼           ▼
┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
│      │W_hh│      │W_hh│      │W_hh│      │
│ Cell │───▶│ Cell │───▶│ Cell │───▶│ Cell │
│      │   │      │   │      │   │      │
└──┬───┘   └──┬───┘   └──┬───┘   └──┬───┘
   │W_hy      │W_hy      │W_hy      │W_hy
   ▼           ▼           ▼           ▼
  y₁          y₂          y₃          y₄

h₀ ─────▶ h₁ ─────▶ h₂ ─────▶ h₃ ─────▶ h₄

CRITICAL: ALL cells share the SAME W_xh, W_hh, W_hy
They are the SAME cell applied at different time steps.
```

---

## 🧮 Equations Summary

```
For each time step t = 1, 2, ..., T:

┌─────────────────────────────────────────────────────┐
│                                                     │
│  Pre-activation:                                    │
│    a_t = W_xh · x_t + W_hh · h_{t-1} + b_h        │
│                                                     │
│  Hidden state:                                      │
│    h_t = tanh(a_t)                                  │
│                                                     │
│  Output:                                            │
│    o_t = W_hy · h_t + b_y                           │
│                                                     │
│  Prediction (depends on task):                      │
│    ŷ_t = softmax(o_t)   (classification)           │
│    ŷ_t = o_t             (regression)              │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn
import numpy as np

class SimpleRNNCell(nn.Module):
    """Manual RNN cell implementation"""
    def __init__(self, input_size, hidden_size, output_size):
        super().__init__()
        self.hidden_size = hidden_size
        
        # The three weight matrices
        self.W_xh = nn.Linear(input_size, hidden_size)    # W_xh · x_t + b
        self.W_hh = nn.Linear(hidden_size, hidden_size, bias=False)  # W_hh · h_{t-1}
        self.W_hy = nn.Linear(hidden_size, output_size)   # W_hy · h_t + b
    
    def forward(self, x_t, h_prev):
        """Single time step"""
        # h_t = tanh(W_xh · x_t + W_hh · h_{t-1} + b_h)
        h_t = torch.tanh(self.W_xh(x_t) + self.W_hh(h_prev))
        # y_t = W_hy · h_t + b_y
        y_t = self.W_hy(h_t)
        return y_t, h_t

class SimpleRNN(nn.Module):
    """Complete RNN using manual cell"""
    def __init__(self, input_size, hidden_size, output_size):
        super().__init__()
        self.cell = SimpleRNNCell(input_size, hidden_size, output_size)
        self.hidden_size = hidden_size
    
    def forward(self, x):
        """
        x: (batch_size, seq_len, input_size)
        """
        batch_size, seq_len, _ = x.shape
        h = torch.zeros(batch_size, self.hidden_size)  # h₀
        
        outputs = []
        for t in range(seq_len):
            y_t, h = self.cell(x[:, t, :], h)
            outputs.append(y_t.unsqueeze(1))
        
        # Stack outputs: (batch_size, seq_len, output_size)
        return torch.cat(outputs, dim=1), h

# Example usage
input_size = 10    # vocabulary / feature dimension
hidden_size = 20   # hidden state size (hyperparameter)
output_size = 5    # number of classes
seq_len = 8        # sequence length
batch_size = 4

model = SimpleRNN(input_size, hidden_size, output_size)
x = torch.randn(batch_size, seq_len, input_size)
outputs, final_hidden = model(x)

print(f"Input shape:        {x.shape}")                # (4, 8, 10)
print(f"All outputs shape:  {outputs.shape}")           # (4, 8, 5)
print(f"Final hidden shape: {final_hidden.shape}")      # (4, 20)

# Using PyTorch's built-in RNN
rnn_builtin = nn.RNN(input_size=10, hidden_size=20, batch_first=True)
out, h_n = rnn_builtin(x)
print(f"\nBuilt-in output: {out.shape}")   # (4, 8, 20)
print(f"Built-in h_n:    {h_n.shape}")     # (1, 4, 20)
```

---

## 📐 Dimension Walkthrough

```
Example: d=3 (input), n=4 (hidden), m=2 (output)

x_t = [0.5, -0.3, 0.8]             shape: (3,)

W_xh = [[w₁₁ w₁₂ w₁₃]             shape: (4, 3)
         [w₂₁ w₂₂ w₂₃]
         [w₃₁ w₃₂ w₃₃]
         [w₄₁ w₄₂ w₄₃]]

W_xh · x_t = [a₁, a₂, a₃, a₄]     shape: (4,)

h_{t-1} = [h₁, h₂, h₃, h₄]        shape: (4,)

W_hh = [[w₁₁ w₁₂ w₁₃ w₁₄]         shape: (4, 4)
         [w₂₁ w₂₂ w₂₃ w₂₄]
         [w₃₁ w₃₂ w₃₃ w₃₄]
         [w₄₁ w₄₂ w₄₃ w₄₄]]

W_hh · h_{t-1} = [b₁, b₂, b₃, b₄] shape: (4,)

h_t = tanh([a₁+b₁, a₂+b₂, a₃+b₃, a₄+b₄] + bias)  shape: (4,)

W_hy = [[w₁₁ w₁₂ w₁₃ w₁₄]         shape: (2, 4)
         [w₂₁ w₂₂ w₂₃ w₂₄]]

y_t = W_hy · h_t = [y₁, y₂]        shape: (2,)
```

---

## 📋 Summary Table

| Component | Symbol | Dimensions | Role |
|-----------|--------|-----------|------|
| Input | x_t | ℝᵈ | Current time step input |
| Hidden State | h_t | ℝⁿ | Memory / compressed history |
| Output | y_t | ℝᵐ | Prediction at time t |
| Input Weights | W_xh | ℝⁿˣᵈ | Transform input to hidden space |
| Recurrent Weights | W_hh | ℝⁿˣⁿ | Transform previous hidden state |
| Output Weights | W_hy | ℝᵐˣⁿ | Map hidden state to output |
| Activation | tanh | — | Squash to [-1, 1] |
| Parameter Count | — | nd + n² + mn + n + m | Independent of seq length |

---

## ❓ Revision Questions

1. **Draw the RNN cell showing all three weight matrices and label each connection with its dimensions.**

2. **If input_size=50 and hidden_size=100, how many parameters are in W_xh and W_hh? Which matrix is larger?**

3. **Why is W_hh square (n×n)? What would happen if hidden state dimension changed between time steps?**

4. **In the unrolled view, how many copies of W_xh exist? Explain why parameter sharing is used.**

5. **Write the equation for computing y_t from h_t. When would you apply softmax vs use raw output?**

6. **Compare the number of parameters in an RNN with hidden_size=100 processing sequences of length 10 vs length 1000. What changes?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Recurrence Concept](../01-Sequence-Modeling/04-recurrence-concept.md) |
| ➡️ Next | [Hidden State](02-hidden-state.md) |
| ⬆️ Unit Overview | [README](../README.md) |
