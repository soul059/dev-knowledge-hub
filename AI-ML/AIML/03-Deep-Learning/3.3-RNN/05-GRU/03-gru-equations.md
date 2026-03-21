# GRU Equations

> **Unit 5, Chapter 3** — Complete GRU equations with step-by-step derivation and worked numerical example.

---

## 📋 Overview

This chapter presents the complete GRU equations, walks through the computation step by step, and provides a full numerical example for building calculation fluency.

---

## 🧮 Complete GRU Equations

```
┌─────────────────────────────────────────────────────────────────┐
│                      GRU EQUATIONS                              │
│                                                                 │
│  1. Update gate:                                               │
│     z_t = σ(W_z · [h_{t-1}, x_t] + b_z)                      │
│                                                                 │
│  2. Reset gate:                                                │
│     r_t = σ(W_r · [h_{t-1}, x_t] + b_r)                      │
│                                                                 │
│  3. Candidate hidden state:                                    │
│     h̃_t = tanh(W_h · [r_t ⊙ h_{t-1}, x_t] + b_h)            │
│                                                                 │
│  4. Hidden state update:                                       │
│     h_t = (1 - z_t) ⊙ h_{t-1} + z_t ⊙ h̃_t                   │
│                                                                 │
│  Dimensions (input_size=d, hidden_size=n):                     │
│    W_z, W_r, W_h ∈ ℝ^{n × (n+d)}                              │
│    b_z, b_r, b_h ∈ ℝⁿ                                         │
│    z_t, r_t, h̃_t, h_t ∈ ℝⁿ                                    │
│                                                                 │
│  Total parameters: 3[n(n+d) + n] = 3n² + 3nd + 3n            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔢 Complete Numerical Example

```
Setup: d=2 (input), n=2 (hidden)

Weights:
W_z = [[0.3, -0.1, 0.5, 0.2],   b_z = [0.0, 0.0]
       [0.1,  0.4, 0.2, 0.3]]

W_r = [[0.2,  0.3, 0.1, 0.4],   b_r = [0.0, 0.0]
       [0.4, -0.2, 0.3, 0.1]]

W_h = [[0.1,  0.5, 0.3, 0.2],   b_h = [0.0, 0.0]
       [0.3, -0.1, 0.4, 0.1]]

State: h_{t-1} = [0.6, -0.4]
Input: x_t = [1.0, 0.5]

════════════════════════════════════════════
Step 1: CONCATENATE [h_{t-1}, x_t]
   [h_{t-1}, x_t] = [0.6, -0.4, 1.0, 0.5]

Step 2: UPDATE GATE
   W_z · [h,x] + b_z
   = [0.3(0.6) + (-0.1)(-0.4) + 0.5(1.0) + 0.2(0.5),
      0.1(0.6) + 0.4(-0.4) + 0.2(1.0) + 0.3(0.5)]
   = [0.18 + 0.04 + 0.50 + 0.10,
      0.06 - 0.16 + 0.20 + 0.15]
   = [0.82, 0.25]

   z_t = σ([0.82, 0.25]) = [0.6944, 0.5622]

Step 3: RESET GATE
   W_r · [h,x] + b_r
   = [0.2(0.6) + 0.3(-0.4) + 0.1(1.0) + 0.4(0.5),
      0.4(0.6) + (-0.2)(-0.4) + 0.3(1.0) + 0.1(0.5)]
   = [0.12 - 0.12 + 0.10 + 0.20,
      0.24 + 0.08 + 0.30 + 0.05]
   = [0.30, 0.67]

   r_t = σ([0.30, 0.67]) = [0.5744, 0.6615]

Step 4: APPLY RESET GATE
   r_t ⊙ h_{t-1} = [0.5744 × 0.6, 0.6615 × (-0.4)]
                  = [0.3446, -0.2646]

Step 5: CANDIDATE
   [r_t⊙h_{t-1}, x_t] = [0.3446, -0.2646, 1.0, 0.5]

   W_h · [r⊙h, x] + b_h
   = [0.1(0.3446) + 0.5(-0.2646) + 0.3(1.0) + 0.2(0.5),
      0.3(0.3446) + (-0.1)(-0.2646) + 0.4(1.0) + 0.1(0.5)]
   = [0.0345 - 0.1323 + 0.30 + 0.10,
      0.1034 + 0.0265 + 0.40 + 0.05]
   = [0.3022, 0.5799]

   h̃_t = tanh([0.3022, 0.5799]) = [0.2931, 0.5222]

Step 6: HIDDEN STATE UPDATE
   h_t = (1 - z_t) ⊙ h_{t-1} + z_t ⊙ h̃_t
       = [(1-0.6944)×0.6, (1-0.5622)×(-0.4)]
       + [0.6944×0.2931, 0.5622×0.5222]
       = [0.3056×0.6, 0.4378×(-0.4)]
       + [0.2036, 0.2936]
       = [0.1834, -0.1751] + [0.2036, 0.2936]
       = [0.3870, 0.1185]

════════════════════════════════════════════
RESULTS:
   z_t = [0.694, 0.562]  (update gate)
   r_t = [0.574, 0.662]  (reset gate)
   h̃_t = [0.293, 0.522]  (candidate)
   h_t = [0.387, 0.119]   (new hidden state)

Interpretation:
   Dim 0: z=0.69 → mostly new info (69% candidate, 31% old)
   Dim 1: z=0.56 → balanced (56% candidate, 44% old)
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn

class GRUFromScratch(nn.Module):
    def __init__(self, input_size, hidden_size):
        super().__init__()
        self.hidden_size = hidden_size
        combined = input_size + hidden_size
        
        self.W_z = nn.Linear(combined, hidden_size)
        self.W_r = nn.Linear(combined, hidden_size)
        self.W_h = nn.Linear(combined, hidden_size)
    
    def forward_step(self, x_t, h_prev):
        combined = torch.cat([h_prev, x_t], dim=1)
        
        z_t = torch.sigmoid(self.W_z(combined))
        r_t = torch.sigmoid(self.W_r(combined))
        
        combined_reset = torch.cat([r_t * h_prev, x_t], dim=1)
        h_tilde = torch.tanh(self.W_h(combined_reset))
        
        h_t = (1 - z_t) * h_prev + z_t * h_tilde
        return h_t
    
    def forward(self, x):
        """x: (batch, seq_len, input_size)"""
        batch_size, seq_len, _ = x.shape
        h = torch.zeros(batch_size, self.hidden_size)
        outputs = []
        
        for t in range(seq_len):
            h = self.forward_step(x[:, t, :], h)
            outputs.append(h.unsqueeze(1))
        
        return torch.cat(outputs, dim=1), h

# Test
model = GRUFromScratch(10, 20)
x = torch.randn(4, 15, 10)
output, h_final = model(x)
print(f"Output: {output.shape}")      # (4, 15, 20)
print(f"Final h: {h_final.shape}")    # (4, 20)

# Verify against PyTorch built-in
gru = nn.GRU(10, 20, batch_first=True)
out_builtin, h_builtin = gru(x)
print(f"Built-in output: {out_builtin.shape}")
print(f"Built-in h_n: {h_builtin.shape}")
```

---

## 📊 Gradient Through the Update Gate

```
h_t = (1 - z_t) ⊙ h_{t-1} + z_t ⊙ h̃_t

∂h_t/∂h_{t-1} = diag(1 - z_t) + z_t terms through h̃_t

When z_t ≈ 0:
   ∂h_t/∂h_{t-1} ≈ I  (identity!)
   → Perfect gradient flow (like LSTM with f_t ≈ 1)

This is how GRU solves vanishing gradients:
   The update gate can create an identity shortcut!
```

---

## 📋 Summary Table

| Step | Equation | Output |
|------|----------|--------|
| 1. Update gate | z_t = σ(W_z·[h,x]+b_z) | (0,1)ⁿ — how much to update |
| 2. Reset gate | r_t = σ(W_r·[h,x]+b_r) | (0,1)ⁿ — how much past in candidate |
| 3. Candidate | h̃_t = tanh(W_h·[r⊙h,x]+b_h) | (-1,1)ⁿ — proposed new state |
| 4. Interpolate | (1-z)⊙h_{t-1} + z⊙h̃_t | ℝⁿ — new hidden state |

---

## ❓ Revision Questions

1. **List all GRU equations. How many weight matrices are there?**

2. **In the worked example, dimension 0 had z=0.69. How does this translate to the mix of old vs new information?**

3. **Compute one step: h_{t-1}=[1, 0], z_t=[0, 1], r_t=[1, 0], h̃_t=[0.5, -0.3]. What is h_t?**

4. **How does the GRU gradient ∂h_t/∂h_{t-1} simplify when z_t ≈ 0?**

5. **For d=20 and n=50, how many total parameters does a GRU have? Compare with LSTM.**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Reset and Update Gates](02-reset-and-update-gates.md) |
| ➡️ Next | [LSTM vs GRU Comparison](04-lstm-vs-gru-comparison.md) |
| ⬆️ Unit Overview | [README](../README.md) |
