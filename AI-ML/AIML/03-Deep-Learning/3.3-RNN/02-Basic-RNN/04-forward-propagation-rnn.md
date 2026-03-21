# Forward Propagation in RNN

> **Unit 2, Chapter 4** — Step-by-step forward pass with numerical examples: computing h_t and y_t at each time step.

---

## 📋 Overview

Forward propagation in an RNN processes the input sequence one time step at a time, updating the hidden state and computing outputs. This chapter walks through the complete forward pass with a **detailed numerical example**.

---

## 🧮 Forward Pass Algorithm

```
┌─────────────────────────────────────────────────────────────┐
│                  RNN FORWARD PASS                           │
│                                                             │
│  Input:  X = (x₁, x₂, ..., x_T), initial state h₀        │
│  Output: Y = (y₁, y₂, ..., y_T), final state h_T          │
│                                                             │
│  for t = 1 to T:                                           │
│      a_t = W_xh · x_t + W_hh · h_{t-1} + b_h             │
│      h_t = tanh(a_t)                                       │
│      o_t = W_hy · h_t + b_y                                │
│      ŷ_t = softmax(o_t)    [for classification]           │
│      L_t = -log(ŷ_t[target_t])  [cross-entropy loss]      │
│                                                             │
│  Total Loss: L = (1/T) Σ L_t                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔢 Complete Numerical Example

### Setup

```
Dimensions: d=2 (input), n=3 (hidden), m=2 (output)

Weight matrices:
W_xh = [[ 0.1,  0.2],     W_hh = [[ 0.3,  0.1, -0.1],
         [ 0.3, -0.1],             [-0.2,  0.4,  0.2],
         [-0.2,  0.4]]             [ 0.1, -0.3,  0.5]]

W_hy = [[ 0.2, -0.1,  0.3],    b_h = [0, 0, 0]    b_y = [0, 0]
         [-0.3,  0.4,  0.1]]

Input sequence (3 time steps):
x₁ = [1.0, 0.5]    x₂ = [0.0, 1.0]    x₃ = [0.5, 0.5]

Initial hidden state:
h₀ = [0.0, 0.0, 0.0]
```

### Step 1: t = 1

```
── Compute pre-activation ──
W_xh · x₁ = [0.1·1.0 + 0.2·0.5,   = [0.20,
              0.3·1.0 + (-0.1)·0.5,    0.25,
              (-0.2)·1.0 + 0.4·0.5]    0.00]

W_hh · h₀ = [0, 0, 0]  (h₀ is zero)

a₁ = [0.20, 0.25, 0.00] + [0, 0, 0] + [0, 0, 0]
   = [0.20, 0.25, 0.00]

── Compute hidden state ──
h₁ = tanh(a₁) = [tanh(0.20), tanh(0.25), tanh(0.00)]
   = [0.1974, 0.2449, 0.0000]

── Compute output ──
o₁ = W_hy · h₁ + b_y
   = [0.2·0.1974 + (-0.1)·0.2449 + 0.3·0.0000,
      (-0.3)·0.1974 + 0.4·0.2449 + 0.1·0.0000]
   = [0.0395 - 0.0245 + 0.0000,
      -0.0592 + 0.0980 + 0.0000]
   = [0.0150, 0.0388]

── Compute prediction (softmax) ──
ŷ₁ = softmax([0.0150, 0.0388])
   = [e^0.0150 / (e^0.0150 + e^0.0388),
      e^0.0388 / (e^0.0150 + e^0.0388)]
   = [1.0151 / 2.0547, 1.0396 / 2.0547]
   = [0.4941, 0.5059]
```

### Step 2: t = 2

```
── Compute pre-activation ──
W_xh · x₂ = [0.1·0.0 + 0.2·1.0,   = [0.2000,
              0.3·0.0 + (-0.1)·1.0,   -0.1000,
              (-0.2)·0.0 + 0.4·1.0]    0.4000]

W_hh · h₁ = [0.3·0.1974 + 0.1·0.2449 + (-0.1)·0.0000,   = [0.0837,
              (-0.2)·0.1974 + 0.4·0.2449 + 0.2·0.0000,      0.0585,
              0.1·0.1974 + (-0.3)·0.2449 + 0.5·0.0000]     -0.0538]

a₂ = [0.2000 + 0.0837, -0.1000 + 0.0585, 0.4000 + (-0.0538)]
   = [0.2837, -0.0415, 0.3462]

── Compute hidden state ──
h₂ = tanh(a₂) = [tanh(0.2837), tanh(-0.0415), tanh(0.3462)]
   = [0.2767, -0.0415, 0.3330]

── Compute output ──
o₂ = W_hy · h₂ + b_y
   = [0.2·0.2767 + (-0.1)·(-0.0415) + 0.3·0.3330,
      (-0.3)·0.2767 + 0.4·(-0.0415) + 0.1·0.3330]
   = [0.0553 + 0.0042 + 0.0999,
      -0.0830 + (-0.0166) + 0.0333]
   = [0.1594, -0.0663]
```

### Step 3: t = 3

```
── Compute pre-activation ──
W_xh · x₃ = [0.1·0.5 + 0.2·0.5,   = [0.1500,
              0.3·0.5 + (-0.1)·0.5,    0.1000,
              (-0.2)·0.5 + 0.4·0.5]    0.1000]

W_hh · h₂ = [0.3·0.2767 + 0.1·(-0.0415) + (-0.1)·0.3330,  = [0.0456,
              (-0.2)·0.2767 + 0.4·(-0.0415) + 0.2·0.3330,     -0.0054,
              0.1·0.2767 + (-0.3)·(-0.0415) + 0.5·0.3330]      0.2066]

a₃ = [0.1500 + 0.0456, 0.1000 + (-0.0054), 0.1000 + 0.2066]
   = [0.1956, 0.0946, 0.3066]

── Compute hidden state ──
h₃ = tanh(a₃) = [0.1937, 0.0943, 0.2975]

── Compute output ──
o₃ = W_hy · h₃ + b_y
   = [0.2·0.1937 + (-0.1)·0.0943 + 0.3·0.2975,
      (-0.3)·0.1937 + 0.4·0.0943 + 0.1·0.2975]
   = [0.0387 - 0.0094 + 0.0893,
      -0.0581 + 0.0377 + 0.0298]
   = [0.1186, 0.0094]
```

### Summary of Forward Pass

```
┌──────┬─────────────────────┬──────────────────────┬────────────────────┐
│ Step │ Hidden State h_t    │ Pre-softmax o_t     │ Prediction ŷ_t     │
├──────┼─────────────────────┼──────────────────────┼────────────────────┤
│ t=1  │ [0.197, 0.245, 0.000] │ [0.015, 0.039]    │ [0.494, 0.506]    │
│ t=2  │ [0.277,-0.042, 0.333] │ [0.159,-0.066]    │ [0.556, 0.444]    │
│ t=3  │ [0.194, 0.094, 0.298] │ [0.119, 0.009]    │ [0.527, 0.473]    │
└──────┴─────────────────────┴──────────────────────┴────────────────────┘
```

---

## 📊 Computing the Loss

```
Targets: y₁ = 0, y₂ = 1, y₃ = 0  (class indices)

Cross-entropy loss at each step:
   L₁ = -log(ŷ₁[0]) = -log(0.494) = 0.706
   L₂ = -log(ŷ₂[1]) = -log(0.444) = 0.812
   L₃ = -log(ŷ₃[0]) = -log(0.527) = 0.641

Total loss:
   L = (L₁ + L₂ + L₃) / 3 = (0.706 + 0.812 + 0.641) / 3 = 0.720
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class RNNForwardDemo(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super().__init__()
        self.hidden_size = hidden_size
        self.W_xh = nn.Parameter(torch.randn(hidden_size, input_size) * 0.1)
        self.W_hh = nn.Parameter(torch.randn(hidden_size, hidden_size) * 0.1)
        self.W_hy = nn.Parameter(torch.randn(output_size, hidden_size) * 0.1)
        self.b_h  = nn.Parameter(torch.zeros(hidden_size))
        self.b_y  = nn.Parameter(torch.zeros(output_size))
    
    def forward_step(self, x_t, h_prev):
        """Single time step forward pass"""
        a_t = self.W_xh @ x_t + self.W_hh @ h_prev + self.b_h
        h_t = torch.tanh(a_t)
        o_t = self.W_hy @ h_t + self.b_y
        return o_t, h_t
    
    def forward(self, X):
        """Complete forward pass through sequence
        X: (seq_len, input_size)
        """
        T = X.shape[0]
        h = torch.zeros(self.hidden_size)
        
        outputs = []
        hidden_states = [h.clone()]
        
        for t in range(T):
            o_t, h = self.forward_step(X[t], h)
            outputs.append(o_t)
            hidden_states.append(h.clone())
        
        return torch.stack(outputs), torch.stack(hidden_states)

# Example
model = RNNForwardDemo(input_size=2, hidden_size=3, output_size=2)
X = torch.tensor([[1.0, 0.5], [0.0, 1.0], [0.5, 0.5]])
targets = torch.tensor([0, 1, 0])

outputs, hidden_states = model(X)

# Compute loss
loss = F.cross_entropy(outputs, targets)

print("=== Forward Pass Results ===")
for t in range(3):
    probs = F.softmax(outputs[t], dim=0)
    print(f"t={t+1}: h={hidden_states[t+1].detach().numpy().round(4)}, "
          f"ŷ={probs.detach().numpy().round(4)}")
print(f"\nTotal Loss: {loss.item():.4f}")

# Compare with PyTorch built-in
rnn = nn.RNN(input_size=2, hidden_size=3, batch_first=False)
out, h_n = rnn(X.unsqueeze(1))  # Add batch dim
print(f"\nPyTorch RNN output: {out.squeeze().shape}")
```

---

## 📐 Flow Diagram with Dimensions

```
Input x_t          Weights           Operations           Output
─────────          ───────           ──────────           ──────

x_t ∈ ℝ²  ──▶  W_xh ∈ ℝ³ˣ²  ──▶  W_xh·x_t ∈ ℝ³  ─┐
                                                       │  (+)  ──▶ tanh ──▶ h_t ∈ ℝ³
h_{t-1}∈ℝ³ ──▶  W_hh ∈ ℝ³ˣ³  ──▶  W_hh·h  ∈ ℝ³   ─┘                      │
                                                                              │
                                                           W_hy ∈ ℝ²ˣ³ ──▶ o_t ∈ ℝ²
                                                                              │
                                                                         softmax
                                                                              │
                                                                         ŷ_t ∈ ℝ²
```

---

## 📋 Summary Table

| Step | Computation | Result |
|------|-------------|--------|
| 1. Linear transform input | W_xh · x_t | Maps input to hidden space |
| 2. Linear transform hidden | W_hh · h_{t-1} | Processes past memory |
| 3. Add + bias | Sum + b_h | Combine current and past |
| 4. Activation | tanh(·) | Non-linear, bounded h_t |
| 5. Output projection | W_hy · h_t + b_y | Map to output space |
| 6. Prediction | softmax(o_t) | Probability distribution |
| 7. Loss | -log(ŷ_t[target]) | Cross-entropy per step |
| 8. Total loss | Average over T | Training objective |

---

## ❓ Revision Questions

1. **Given W_xh = [[1, 0], [0, 1]], W_hh = [[0.5, 0], [0, 0.5]], b_h = [0, 0], x₁ = [1, 0], h₀ = [0, 0], compute h₁ and h₂ if x₂ = [0, 1].**

2. **Why is tanh applied after the linear combination? What would happen without it?**

3. **In a sequence of length T with output at every step, how many times is W_xh used in the forward pass?**

4. **If the cross-entropy loss at each step is L₁=0.5, L₂=0.3, L₃=0.8, L₄=0.2, what is the average loss?**

5. **Why must we store all intermediate hidden states h₁, h₂, ..., h_T during the forward pass? What are they used for?**

6. **Compare the forward pass of a 5-layer feedforward network vs a 5-step RNN. What are the key structural differences?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Unrolling Through Time](03-unrolling-through-time.md) |
| ➡️ Next | [Parameter Sharing](05-parameter-sharing.md) |
| ⬆️ Unit Overview | [README](../README.md) |
