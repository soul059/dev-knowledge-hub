# Attention Mechanism Basics

> **Unit 6, Chapter 4** — Attention as the solution to the bottleneck: alignment scores, attention weights, context vector as weighted sum, Bahdanau vs Luong attention.

---

## 📋 Overview

**Attention** allows the decoder to look at ALL encoder hidden states when generating each output token, instead of relying on a single context vector. This solves the information bottleneck by creating a **dynamic, weighted connection** between decoder and encoder positions.

---

## 🔑 Core Idea

```
WITHOUT Attention (bottleneck):
   Encoder: h₁, h₂, h₃, h₄, h₅ ──▶ c (single vector) ──▶ Decoder

WITH Attention:
   Encoder: h₁, h₂, h₃, h₄, h₅
                ↑    ↑    ↑    ↑    ↑
                └────┴────┴────┴────┘
                         │
              Decoder ATTENDS to all encoder
              states with different weights
              at each decoding step

   Step 1: α = [0.1, 0.1, 0.6, 0.1, 0.1]  Focus on h₃
   Step 2: α = [0.5, 0.3, 0.1, 0.05, 0.05] Focus on h₁, h₂
   Step 3: α = [0.05, 0.05, 0.1, 0.3, 0.5] Focus on h₅
```

---

## 🧮 Attention Mechanism

```
┌─────────────────────────────────────────────────────────────────┐
│                    ATTENTION (at decoder step t)                 │
│                                                                 │
│  Step 1: Compute ALIGNMENT SCORES                              │
│     e_{t,j} = score(s_t, h_j)   for j = 1, ..., T_x           │
│                                                                 │
│  Step 2: Compute ATTENTION WEIGHTS (normalize)                  │
│     α_{t,j} = softmax(e_{t,j}) = exp(e_{t,j}) / Σ_k exp(e_{t,k})│
│                                                                 │
│  Step 3: Compute CONTEXT VECTOR (weighted sum)                  │
│     c_t = Σ_j α_{t,j} · h_j                                    │
│                                                                 │
│  Step 4: Combine with decoder state                             │
│     ỹ_t = f(s_t, c_t)                                          │
│                                                                 │
│  Key: α_{t,j} tells the decoder HOW MUCH to focus on          │
│  encoder position j when generating output at step t           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📐 Scoring Functions

### Bahdanau Attention (Additive, 2014)

```
e_{t,j} = v^T · tanh(W_a · s_{t-1} + U_a · h_j)

Parameters: v ∈ ℝⁿ, W_a ∈ ℝ^{n×n}, U_a ∈ ℝ^{n×n}

Uses s_{t-1} (PREVIOUS decoder state)
More expressive but slower
```

### Luong Attention (Multiplicative, 2015)

```
Three variants:

Dot:     e_{t,j} = s_t^T · h_j              (no parameters!)
General: e_{t,j} = s_t^T · W_a · h_j         (W_a ∈ ℝ^{n×n})
Concat:  e_{t,j} = v^T · tanh(W_a · [s_t; h_j])  (similar to Bahdanau)

Uses s_t (CURRENT decoder state)
Dot product is fastest and often works well
```

---

## 📊 Attention Visualization

```
Translating: "I love cats" → "J'aime les chats"

Encoder states: h₁(I), h₂(love), h₃(cats)

Decoding "J'":
   α = [0.8, 0.1, 0.1]     ← focuses on "I"
   c₁ = 0.8·h₁ + 0.1·h₂ + 0.1·h₃

Decoding "aime":
   α = [0.1, 0.8, 0.1]     ← focuses on "love"
   c₂ = 0.1·h₁ + 0.8·h₂ + 0.1·h₃

Decoding "les":
   α = [0.1, 0.1, 0.8]     ← focuses on "cats" (article agrees)
   c₃ = 0.1·h₁ + 0.1·h₂ + 0.8·h₃

Attention matrix (heatmap):
              I    love  cats
   J'      [0.8   0.1   0.1]
   aime    [0.1   0.8   0.1]
   les     [0.1   0.1   0.8]
   chats   [0.0   0.1   0.9]

This matrix shows the ALIGNMENT between source and target words!
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class BahdanauAttention(nn.Module):
    """Additive (Bahdanau) attention"""
    def __init__(self, hidden_dim):
        super().__init__()
        self.W_a = nn.Linear(hidden_dim, hidden_dim, bias=False)
        self.U_a = nn.Linear(hidden_dim, hidden_dim, bias=False)
        self.v = nn.Linear(hidden_dim, 1, bias=False)
    
    def forward(self, decoder_state, encoder_outputs):
        """
        decoder_state: (batch, hidden_dim)
        encoder_outputs: (batch, src_len, hidden_dim)
        Returns: context (batch, hidden_dim), weights (batch, src_len)
        """
        # decoder_state: (batch, 1, hidden) for broadcasting
        score = self.v(torch.tanh(
            self.W_a(decoder_state.unsqueeze(1)) + 
            self.U_a(encoder_outputs)
        ))  # (batch, src_len, 1)
        
        weights = F.softmax(score.squeeze(2), dim=1)  # (batch, src_len)
        context = torch.bmm(weights.unsqueeze(1), encoder_outputs)  # (batch, 1, hidden)
        
        return context.squeeze(1), weights

class LuongAttention(nn.Module):
    """Multiplicative (Luong) dot attention"""
    def forward(self, decoder_state, encoder_outputs):
        """
        decoder_state: (batch, hidden_dim)
        encoder_outputs: (batch, src_len, hidden_dim)
        """
        # Dot product: s_t^T · h_j
        score = torch.bmm(
            encoder_outputs, 
            decoder_state.unsqueeze(2)
        ).squeeze(2)  # (batch, src_len)
        
        weights = F.softmax(score, dim=1)
        context = torch.bmm(weights.unsqueeze(1), encoder_outputs).squeeze(1)
        
        return context, weights

# Demo
batch, src_len, hidden = 4, 10, 64
decoder_state = torch.randn(batch, hidden)
encoder_outputs = torch.randn(batch, src_len, hidden)

attn = BahdanauAttention(hidden)
context, weights = attn(decoder_state, encoder_outputs)
print(f"Context: {context.shape}")    # (4, 64)
print(f"Weights: {weights.shape}")    # (4, 10)
print(f"Weights sum: {weights.sum(dim=1)}")  # All 1.0
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Alignment Score | How well decoder position t matches encoder position j |
| Attention Weights | Softmax of scores → probability distribution over encoder |
| Context Vector | Weighted sum of encoder states → dynamic per decoder step |
| Bahdanau (Additive) | score = v·tanh(W·s + U·h), uses s_{t-1} |
| Luong (Dot) | score = s_t^T · h_j, fastest, no extra parameters |
| Bottleneck Solution | Decoder accesses ALL encoder states, not just last one |

---

## ❓ Revision Questions

1. **Explain how attention solves the information bottleneck problem of encoder-decoder models.**

2. **What do the attention weights α_{t,j} represent? Why must they sum to 1?**

3. **Compare Bahdanau and Luong attention: scoring function, which decoder state is used, and computational cost.**

4. **Given encoder states h₁=[1,0], h₂=[0,1] and weights α=[0.3, 0.7], compute the context vector.**

5. **Why is the attention matrix useful for interpreting translation quality?**

6. **What happens if attention weights are uniform (α=[1/T, 1/T, ..., 1/T])? How does this compare to no attention?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Encoder-Decoder](03-encoder-decoder.md) |
| ➡️ Next | [Teacher Forcing](05-teacher-forcing.md) |
| ⬆️ Unit Overview | [README](../README.md) |
