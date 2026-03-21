# Parameter Sharing in RNNs

> **Unit 2, Chapter 5** — Why RNNs use the same weights at every time step: reduced parameters, variable-length sequences, and knowledge transfer across positions.

---

## 📋 Overview

One of the most important design decisions in RNNs is **parameter sharing**: the same weight matrices W_xh, W_hh, and W_hy are used at every time step. This is analogous to weight sharing in CNNs (convolutional filters) and is essential for handling variable-length sequences efficiently.

---

## 🔑 The Core Concept

```
┌─────────────────────────────────────────────────────────────────┐
│              WITHOUT Parameter Sharing (Bad!)                    │
│                                                                  │
│  t=1: h₁ = tanh(W_xh¹ · x₁ + W_hh¹ · h₀ + b₁)              │
│  t=2: h₂ = tanh(W_xh² · x₂ + W_hh² · h₁ + b₂)              │
│  t=3: h₃ = tanh(W_xh³ · x₃ + W_hh³ · h₂ + b₃)   DIFFERENT  │
│  ...                                                weights!   │
│  t=T: h_T = tanh(W_xhᵀ · x_T + W_hhᵀ · h_{T-1} + bᵀ)       │
│                                                                  │
│  Problems:                                                       │
│  • T sets of parameters → parameters grow with sequence length  │
│  • Cannot handle sequences longer than training max             │
│  • Position-specific: word at position 3 processed differently  │
│    than same word at position 7                                 │
│  • No generalization across time steps                          │
├─────────────────────────────────────────────────────────────────┤
│              WITH Parameter Sharing (Good! ✓)                    │
│                                                                  │
│  t=1: h₁ = tanh(W_xh · x₁ + W_hh · h₀ + b)                  │
│  t=2: h₂ = tanh(W_xh · x₂ + W_hh · h₁ + b)      SAME       │
│  t=3: h₃ = tanh(W_xh · x₃ + W_hh · h₂ + b)      weights!   │
│  ...                                                            │
│  t=T: h_T = tanh(W_xh · x_T + W_hh · h_{T-1} + b)           │
│                                                                  │
│  Benefits:                                                       │
│  ✓ Fixed number of parameters (independent of T)               │
│  ✓ Any sequence length at inference                            │
│  ✓ Generalizes across positions                                │
│  ✓ Learns universal temporal rules                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Parameter Count Comparison

```
Setup: input_dim=100, hidden_dim=256, output_dim=50

WITHOUT sharing (T time steps):
   Parameters = T × (W_xh + W_hh + W_hy + biases)
              = T × (256×100 + 256×256 + 50×256 + 256 + 50)
              = T × 104,498
   For T=100:  10,449,800 parameters
   For T=500:  52,249,000 parameters  ← 5x more!
   For T=1000: 104,498,000 parameters ← Explodes!

WITH sharing (any T):
   Parameters = 256×100 + 256×256 + 50×256 + 256 + 50
              = 25,600 + 65,536 + 12,800 + 256 + 50
              = 104,242 parameters
   SAME for T=100, T=500, T=1000!

┌─────────────────────────────────────┐
│  Sharing reduces parameters by Tx!  │
│  For T=100: 100x fewer parameters   │
└─────────────────────────────────────┘
```

---

## 🔄 Analogy: CNN Weight Sharing

```
CNN (Spatial):                          RNN (Temporal):
┌───────────────────┐                   ┌───────────────────┐
│ Same filter slides │                   │ Same RNN cell     │
│ across all spatial │                   │ applied at all    │
│ positions          │                   │ time steps        │
│                   │                   │                   │
│ ┌──┐              │                   │     t=1 t=2 t=3   │
│ │F │ → → → →      │                   │      │   │   │    │
│ └──┘              │                   │      ▼   ▼   ▼    │
│ Detects features  │                   │    ┌───┬───┬───┐  │
│ regardless of     │                   │    │RNN│RNN│RNN│  │
│ WHERE they appear │                   │    └───┴───┴───┘  │
│                   │                   │ Same cell at every │
│                   │                   │ time step          │
└───────────────────┘                   └───────────────────┘

Both achieve TRANSLATION INVARIANCE:
  CNN: spatial translation invariance (features detected anywhere)
  RNN: temporal translation invariance (patterns detected anytime)
```

---

## 🧮 Why It Works Mathematically

```
The function f(x_t, h_{t-1}) = tanh(W_xh · x_t + W_hh · h_{t-1} + b)
learns a UNIVERSAL transformation:

"Given ANY input x_t and ANY memory h_{t-1},
 how should I update my memory?"

This is the same question at every time step:
• At t=1: How do I process "The" given empty memory?
• At t=5: How do I process "cat" given memory of "The big black"?
• At t=100: How do I process "end" given memory of a long story?

The SAME function handles all cases because:
1. The input x_t provides the current content
2. The hidden state h_{t-1} provides the context
3. The weights learn HOW to combine them

Different contexts → different hidden states → different behaviors
even with the SAME weights!
```

---

## 💻 Demonstration

```python
import torch
import torch.nn as nn

# ─── Show parameter sharing explicitly ───
rnn = nn.RNN(input_size=10, hidden_size=20, batch_first=True)

# Process different sequence lengths with SAME parameters
short_seq = torch.randn(1, 5, 10)    # length 5
medium_seq = torch.randn(1, 50, 10)  # length 50
long_seq = torch.randn(1, 500, 10)   # length 500

out_short, _ = rnn(short_seq)
out_medium, _ = rnn(medium_seq)
out_long, _ = rnn(long_seq)

print("Same model, different sequence lengths:")
print(f"  Short  (T=5):   output shape = {out_short.shape}")
print(f"  Medium (T=50):  output shape = {out_medium.shape}")
print(f"  Long   (T=500): output shape = {out_long.shape}")

# Count parameters — same regardless of T
total_params = sum(p.numel() for p in rnn.parameters())
print(f"\nTotal parameters: {total_params}")
print(f"(Same for T=5, T=50, or T=500!)")

# ─── Verify weights are shared ───
# Access the weight matrices
print(f"\nWeight shapes:")
print(f"  W_ih (W_xh): {rnn.weight_ih_l0.shape}")  # (20, 10)
print(f"  W_hh:        {rnn.weight_hh_l0.shape}")  # (20, 20)
print(f"  b_ih:        {rnn.bias_ih_l0.shape}")     # (20,)
print(f"  b_hh:        {rnn.bias_hh_l0.shape}")     # (20,)

# ─── Without sharing (for comparison) ───
class NoSharingRNN(nn.Module):
    """RNN without parameter sharing (for demonstration only)"""
    def __init__(self, input_size, hidden_size, max_len):
        super().__init__()
        self.cells = nn.ModuleList([
            nn.Linear(input_size + hidden_size, hidden_size)
            for _ in range(max_len)
        ])
        self.hidden_size = hidden_size
    
    def forward(self, x):
        batch_size, seq_len, _ = x.shape
        assert seq_len <= len(self.cells), "Sequence too long!"
        h = torch.zeros(batch_size, self.hidden_size)
        for t in range(seq_len):
            combined = torch.cat([x[:, t], h], dim=1)
            h = torch.tanh(self.cells[t](combined))  # Different weights!
        return h

no_share = NoSharingRNN(10, 20, max_len=100)
no_share_params = sum(p.numel() for p in no_share.parameters())
print(f"\nNo-sharing model (max_len=100): {no_share_params} parameters")
print(f"Shared model:                    {total_params} parameters")
print(f"Ratio: {no_share_params / total_params:.1f}x more without sharing!")
```

---

## 🔑 Benefits of Parameter Sharing

```
┌────────────────────────────┬───────────────────────────────────┐
│ Benefit                    │ Explanation                       │
├────────────────────────────┼───────────────────────────────────┤
│ Variable-length input      │ Same cell works for any T         │
│ Compact model              │ Parameters independent of T       │
│ Better generalization      │ Learns universal rules            │
│ Transfer across positions  │ Pattern at t=3 helps at t=100    │
│ Data efficiency            │ All positions contribute to       │
│                            │ learning the same weights         │
│ Temporal invariance        │ Same event recognized at any time │
└────────────────────────────┴───────────────────────────────────┘
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Parameter Sharing | Same W_xh, W_hh, W_hy at every time step |
| Fixed Parameter Count | Independent of sequence length T |
| Variable-Length Support | Process any length sequence without modification |
| Temporal Invariance | Patterns detected regardless of position in sequence |
| CNN Analogy | Like convolutional filters sliding across space |
| Trade-off | Cannot learn position-specific behavior (use positional encoding) |
| Gradient Effect | Gradients from all time steps update the same weights |

---

## ❓ Revision Questions

1. **If an RNN has input_dim=50 and hidden_dim=100, how many parameters are in W_xh and W_hh? Does this number change with sequence length?**

2. **What is the analogy between parameter sharing in CNNs and RNNs?**

3. **Without parameter sharing, a model for sequences of length T=200 has T times more parameters. What practical problems does this cause?**

4. **How does parameter sharing enable an RNN trained on sequences of length ≤50 to process a sequence of length 100 at test time?**

5. **What is "temporal translation invariance"? Give an example of a pattern that should be recognized at any position in a sequence.**

6. **Is there any disadvantage to parameter sharing? When might position-specific processing be beneficial?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Forward Propagation](04-forward-propagation-rnn.md) |
| ➡️ Next | [RNN Task Variants](06-rnn-task-variants.md) |
| ⬆️ Unit Overview | [README](../README.md) |
