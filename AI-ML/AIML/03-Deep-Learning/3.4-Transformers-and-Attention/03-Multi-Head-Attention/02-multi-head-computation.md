# Multi-Head Attention Computation

[← Why Multiple Heads](01-why-multiple-heads.md) | [Head Concatenation →](03-head-concatenation.md)

---

## Overview

Multi-head attention is the mechanism that gives Transformers their representational power. Instead of performing a single attention function over `d_model`-dimensional keys, values, and queries, the model linearly projects them `h` times into smaller `d_k`-dimensional subspaces, runs scaled dot-product attention in parallel across all heads, concatenates the results, and applies a final linear projection. This section derives every formula step-by-step, tracks tensor dimensions meticulously, walks through a full numerical example, and provides a production-quality PyTorch implementation.

---

## 1. The Master Formula

The complete multi-head attention equation from *"Attention Is All You Need"* (Vaswani et al., 2017):

```
MultiHead(Q, K, V) = Concat(head₁, head₂, ..., headₕ) · W^O
```

where each head is:

```
headᵢ = Attention(Q · Wᵢ^Q,  K · Wᵢ^K,  V · Wᵢ^V)
```

and Attention is scaled dot-product attention:

```
Attention(Qᵢ, Kᵢ, Vᵢ) = softmax( Qᵢ · Kᵢᵀ / √d_k ) · Vᵢ
```

### Parameter Matrices and Their Dimensions

| Matrix | Shape | Description |
|---|---|---|
| `Wᵢ^Q` | `(d_model × d_k)` | Query projection for head `i` |
| `Wᵢ^K` | `(d_model × d_k)` | Key projection for head `i` |
| `Wᵢ^V` | `(d_model × d_v)` | Value projection for head `i` |
| `W^O` | `(h·d_v × d_model)` | Output projection |

In practice, `d_k = d_v = d_model / h`.

---

## 2. Dimension Tracking (Standard Configuration)

```
Given:  d_model = 512,  h = 8

Then:   d_k = d_v = 512 / 8 = 64

Step-by-step dimensions for a sequence of length n:

Input:
  Q, K, V ∈ ℝ^(n × 512)

Per-head projection:
  Qᵢ = Q · Wᵢ^Q    →  (n × 512) · (512 × 64)  =  (n × 64)
  Kᵢ = K · Wᵢ^K    →  (n × 512) · (512 × 64)  =  (n × 64)
  Vᵢ = V · Wᵢ^V    →  (n × 512) · (512 × 64)  =  (n × 64)

Attention scores:
  Qᵢ · Kᵢᵀ          →  (n × 64) · (64 × n)     =  (n × n)

After softmax:
  attn_weights       →  (n × n)

Weighted values:
  headᵢ = attn · Vᵢ →  (n × n) · (n × 64)      =  (n × 64)

Concatenation (all h heads):
  Concat(head₁..headₕ) →  (n × (8 × 64))        =  (n × 512)

Output projection:
  output = Concat · W^O →  (n × 512) · (512 × 512) = (n × 512)
```

**Key insight:** The output has the **same shape** as the input (`n × d_model`), making multi-head attention a drop-in replacement for single-head attention.

---

## 3. ASCII Diagram — Parallel Head Computation

```
                            Input X ∈ ℝ^(n × d_model)
                                      │
                  ┌───────────────────┼───────────────────┐
                  │                   │                   │
                  ▼                   ▼                   ▼
              Q = X·W^Q          K = X·W^K          V = X·W^V
            (n × d_model)      (n × d_model)      (n × d_model)
                  │                   │                   │
          ┌───────┼───────┐   ┌───────┼───────┐   ┌───────┼───────┐
          │       │       │   │       │       │   │       │       │
      ┌───┴──┐┌──┴──┐┌───┴┐ ┌┴────┐┌─┴───┐┌──┴┐ ┌┴────┐┌─┴───┐┌──┴┐
      │Q₁    ││Q₂   ││Q₃  │ │K₁   ││K₂   ││K₃ │ │V₁   ││V₂   ││V₃ │
      │(n,dk)││(n,dk)││... │ │(n,dk)││(n,dk)││...│ │(n,dv)││(n,dv)││...│
      └──┬───┘└──┬───┘└──┬┘ └──┬───┘└──┬───┘└──┬┘ └──┬───┘└──┬───┘└──┬┘
         │       │       │     │       │       │     │       │       │
         ▼       ▼       ▼     ▼       ▼       ▼     ▼       ▼       ▼
      ┌──────────────────────────────────────────────────────────────────┐
      │              Scaled Dot-Product Attention (per head)            │
      │                                                                  │
      │  Head i:  softmax( Qᵢ · Kᵢᵀ / √d_k ) · Vᵢ   →  (n × d_v)    │
      └──────────────────────────────────────────────────────────────────┘
         │       │       │
         ▼       ▼       ▼
      ┌──────┐┌──────┐┌──────┐
      │head₁ ││head₂ ││head₃ │  ...  headₕ
      │(n,dv)││(n,dv)││(n,dv)│
      └──┬───┘└──┬───┘└──┬───┘
         │       │       │
         └───────┼───────┘
                 ▼
        ┌────────────────┐
        │   Concatenate  │   →  (n × h·d_v) = (n × d_model)
        └───────┬────────┘
                ▼
        ┌────────────────┐
        │  Linear (W^O)  │   →  (n × d_model)
        └───────┬────────┘
                ▼
         Output ∈ ℝ^(n × d_model)
```

---

## 4. Step-by-Step Worked Example

### Setup

```
d_model = 4,  h = 2,  d_k = d_v = 2
Sequence length n = 3  (tokens: "I", "love", "AI")
```

### 4.1 Input Matrix

```
X = ┌ 1.0   0.5   0.3   0.8 ┐   ← "I"
    │ 0.2   0.9   0.7   0.1 │   ← "love"
    └ 0.6   0.4   1.0   0.5 ┘   ← "AI"
```

For self-attention: `Q = K = V = X` (before projection).

### 4.2 Projection Matrices (Head 1)

```
W₁^Q = ┌ 0.1   0.3 ┐       W₁^K = ┌ 0.2   0.1 ┐       W₁^V = ┌ 0.5   0.2 ┐
        │ 0.4   0.2 │               │ 0.3   0.5 │               │ 0.1   0.6 │
        │ 0.5   0.1 │               │ 0.1   0.4 │               │ 0.3   0.4 │
        └ 0.2   0.6 ┘               └ 0.6   0.2 ┘               └ 0.4   0.3 ┘
       (4 × 2)                     (4 × 2)                     (4 × 2)
```

### 4.3 Compute Q₁, K₁, V₁

```
Q₁ = X · W₁^Q

Row "I":    [1.0, 0.5, 0.3, 0.8] · W₁^Q
          = [1.0×0.1 + 0.5×0.4 + 0.3×0.5 + 0.8×0.2,
             1.0×0.3 + 0.5×0.2 + 0.3×0.1 + 0.8×0.6]
          = [0.10 + 0.20 + 0.15 + 0.16,  0.30 + 0.10 + 0.03 + 0.48]
          = [0.61, 0.91]

Row "love": [0.2, 0.9, 0.7, 0.1] · W₁^Q
          = [0.02 + 0.36 + 0.35 + 0.02,  0.06 + 0.18 + 0.07 + 0.06]
          = [0.75, 0.37]

Row "AI":   [0.6, 0.4, 1.0, 0.5] · W₁^Q
          = [0.06 + 0.16 + 0.50 + 0.10,  0.18 + 0.08 + 0.10 + 0.30]
          = [0.82, 0.66]

Q₁ = ┌ 0.61   0.91 ┐
     │ 0.75   0.37 │
     └ 0.82   0.66 ┘
```

Similarly compute K₁ and V₁ (matrix multiplications omitted for brevity):

```
K₁ = ┌ 0.86   0.52 ┐       V₁ = ┌ 0.96   0.74 ┐
     │ 0.80   0.75 │            │ 0.48   0.78 │
     └ 0.72   0.56 ┘            └ 0.84   0.67 ┘
```

### 4.4 Attention Scores (Head 1)

```
scores₁ = Q₁ · K₁ᵀ / √d_k     (d_k = 2, so √d_k ≈ 1.414)

Q₁ · K₁ᵀ:

         ┌ 0.61×0.86 + 0.91×0.52   0.61×0.80 + 0.91×0.75   0.61×0.72 + 0.91×0.56 ┐
         │ 0.75×0.86 + 0.37×0.52   0.75×0.80 + 0.37×0.75   0.75×0.72 + 0.37×0.56 │
         └ 0.82×0.86 + 0.66×0.52   0.82×0.80 + 0.66×0.75   0.82×0.72 + 0.66×0.56 ┘

       = ┌ 0.525 + 0.473   0.488 + 0.683   0.439 + 0.510 ┐
         │ 0.645 + 0.192   0.600 + 0.278   0.540 + 0.207 │
         └ 0.705 + 0.343   0.656 + 0.495   0.590 + 0.370 ┘

       = ┌ 0.998   1.171   0.949 ┐
         │ 0.837   0.878   0.747 │
         └ 1.048   1.151   0.960 ┘

After dividing by √2 ≈ 1.414:

scores₁ = ┌ 0.706   0.828   0.671 ┐
           │ 0.592   0.621   0.528 │
           └ 0.741   0.814   0.679 ┘
```

### 4.5 Softmax (Head 1)

```
Row 0: softmax([0.706, 0.828, 0.671])
       exp: [2.026, 2.289, 1.956]  sum = 6.271
       = [0.323, 0.365, 0.312]

Row 1: softmax([0.592, 0.621, 0.528])
       exp: [1.807, 1.861, 1.695]  sum = 5.363
       = [0.337, 0.347, 0.316]

Row 2: softmax([0.741, 0.814, 0.679])
       exp: [2.098, 2.257, 1.972]  sum = 6.327
       = [0.332, 0.357, 0.312]

attn₁ = ┌ 0.323   0.365   0.312 ┐
         │ 0.337   0.347   0.316 │
         └ 0.332   0.357   0.312 ┘
```

### 4.6 Weighted Values (Head 1)

```
head₁ = attn₁ · V₁

Row 0: [0.323×0.96 + 0.365×0.48 + 0.312×0.84,
        0.323×0.74 + 0.365×0.78 + 0.312×0.67]
     = [0.310 + 0.175 + 0.262,  0.239 + 0.285 + 0.209]
     = [0.747, 0.733]

(Similarly for rows 1 and 2)

head₁ = ┌ 0.747   0.733 ┐
         │ 0.752   0.731 │
         └ 0.749   0.732 ┘
```

### 4.7 Repeat for Head 2

Head 2 uses different projection matrices `W₂^Q, W₂^K, W₂^V`, producing:

```
head₂ = ┌ 0.513   0.624 ┐
         │ 0.491   0.598 │      (example values)
         └ 0.508   0.617 ┘
```

### 4.8 Concatenate and Project

```
Concat(head₁, head₂) = ┌ 0.747  0.733  0.513  0.624 ┐
                         │ 0.752  0.731  0.491  0.598 │   shape: (3 × 4)
                         └ 0.749  0.732  0.508  0.617 ┘

Output = Concat · W^O    where W^O ∈ ℝ^(4 × 4)

→ shape: (3 × 4) = (n × d_model)  ✓
```

---

## 5. Implementation Approaches

### 5.1 Naive Implementation (Loop Over Heads)

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class MultiHeadAttentionNaive(nn.Module):
    """Educational multi-head attention using explicit loop over heads."""

    def __init__(self, d_model: int, num_heads: int):
        super().__init__()
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        # Separate projection matrices per head
        self.W_q = nn.ModuleList([nn.Linear(d_model, self.d_k, bias=False) for _ in range(num_heads)])
        self.W_k = nn.ModuleList([nn.Linear(d_model, self.d_k, bias=False) for _ in range(num_heads)])
        self.W_v = nn.ModuleList([nn.Linear(d_model, self.d_k, bias=False) for _ in range(num_heads)])
        self.W_o = nn.Linear(d_model, d_model, bias=False)

    def forward(self, Q, K, V, mask=None):
        heads = []
        for i in range(self.num_heads):
            Qi = self.W_q[i](Q)   # (batch, seq, d_k)
            Ki = self.W_k[i](K)
            Vi = self.W_v[i](V)

            scores = torch.matmul(Qi, Ki.transpose(-2, -1)) / (self.d_k ** 0.5)
            if mask is not None:
                scores = scores.masked_fill(mask == 0, float('-inf'))
            attn = F.softmax(scores, dim=-1)
            head_i = torch.matmul(attn, Vi)  # (batch, seq, d_k)
            heads.append(head_i)

        concat = torch.cat(heads, dim=-1)  # (batch, seq, d_model)
        return self.W_o(concat)
```

### 5.2 Efficient Implementation (Batched — No Loop)

```python
class MultiHeadAttention(nn.Module):
    """Production-quality multi-head attention with batched projections."""

    def __init__(self, d_model: int, num_heads: int, dropout: float = 0.1):
        super().__init__()
        assert d_model % num_heads == 0
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        # Single large linear layers (all heads packed together)
        self.W_q = nn.Linear(d_model, d_model, bias=False)
        self.W_k = nn.Linear(d_model, d_model, bias=False)
        self.W_v = nn.Linear(d_model, d_model, bias=False)
        self.W_o = nn.Linear(d_model, d_model, bias=False)
        self.dropout = nn.Dropout(dropout)

    def split_heads(self, x, batch_size):
        """Reshape (batch, seq, d_model) → (batch, heads, seq, d_k)."""
        x = x.view(batch_size, -1, self.num_heads, self.d_k)
        return x.transpose(1, 2)

    def forward(self, Q, K, V, mask=None):
        batch_size = Q.size(0)

        # Step 1: Project all heads at once
        Q = self.W_q(Q)  # (batch, seq_q, d_model)
        K = self.W_k(K)  # (batch, seq_k, d_model)
        V = self.W_v(V)  # (batch, seq_k, d_model)

        # Step 2: Split into heads
        Q = self.split_heads(Q, batch_size)  # (batch, heads, seq_q, d_k)
        K = self.split_heads(K, batch_size)  # (batch, heads, seq_k, d_k)
        V = self.split_heads(V, batch_size)  # (batch, heads, seq_k, d_k)

        # Step 3: Scaled dot-product attention
        scores = torch.matmul(Q, K.transpose(-2, -1)) / (self.d_k ** 0.5)
        # scores shape: (batch, heads, seq_q, seq_k)

        if mask is not None:
            # mask shape: (batch, 1, 1, seq_k) or (batch, 1, seq_q, seq_k)
            scores = scores.masked_fill(mask == 0, float('-inf'))

        attn_weights = F.softmax(scores, dim=-1)
        attn_weights = self.dropout(attn_weights)

        # Step 4: Compute weighted values
        context = torch.matmul(attn_weights, V)
        # context shape: (batch, heads, seq_q, d_k)

        # Step 5: Concatenate heads
        context = context.transpose(1, 2).contiguous()
        # context shape: (batch, seq_q, heads, d_k)
        context = context.view(batch_size, -1, self.d_model)
        # context shape: (batch, seq_q, d_model)

        # Step 6: Output projection
        output = self.W_o(context)
        # output shape: (batch, seq_q, d_model)

        return output, attn_weights
```

### 5.3 Usage Example

```python
# Standard Transformer Base configuration
d_model = 512
num_heads = 8
seq_len = 10
batch_size = 2

mha = MultiHeadAttention(d_model=d_model, num_heads=num_heads)

# Self-attention: Q = K = V = encoder output
x = torch.randn(batch_size, seq_len, d_model)
output, attn = mha(x, x, x)

print(f"Input:   {x.shape}")       # torch.Size([2, 10, 512])
print(f"Output:  {output.shape}")   # torch.Size([2, 10, 512])
print(f"Weights: {attn.shape}")     # torch.Size([2, 8, 10, 10])

# Cross-attention: Q from decoder, K & V from encoder
enc_out = torch.randn(batch_size, 20, d_model)  # encoder output (seq=20)
dec_in  = torch.randn(batch_size, 15, d_model)  # decoder input  (seq=15)

cross_out, cross_attn = mha(dec_in, enc_out, enc_out)
print(f"Cross-attention output: {cross_out.shape}")  # torch.Size([2, 15, 512])
print(f"Cross-attention weights: {cross_attn.shape}") # torch.Size([2, 8, 15, 20])
```

---

## 6. Dimension Flow Summary Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                    Dimension Tracking Cheat-Sheet                    │
│                                                                      │
│  Config: d_model=512, h=8, d_k=d_v=64, batch=B, seq_q=Sq, seq_k=Sk│
│                                                                      │
│  Input Q:          (B, Sq, 512)                                      │
│  Input K, V:       (B, Sk, 512)                                      │
│                         │                                            │
│  After W_q/W_k/W_v:    │  still (B, S, 512) — packed all heads      │
│                         │                                            │
│  After split_heads:     │  (B, 8, S, 64)  — h separate d_k dims     │
│                         │                                            │
│  Scores Q·Kᵀ/√d_k:     │  (B, 8, Sq, Sk) — per-head attention map   │
│                         │                                            │
│  After softmax:         │  (B, 8, Sq, Sk) — normalised weights       │
│                         │                                            │
│  Context attn·V:        │  (B, 8, Sq, 64) — per-head output          │
│                         │                                            │
│  After concat:          │  (B, Sq, 512)   — heads merged              │
│                         │                                            │
│  After W_o:             │  (B, Sq, 512)   — final output              │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 7. Real-World Applications

| Application | Multi-Head Role |
|---|---|
| **GPT-4 / LLM inference** | Causal multi-head attention with KV-cache for efficient autoregressive generation |
| **BERT pre-training** | Bidirectional multi-head attention captures context from both directions |
| **Vision Transformer (ViT)** | Patches attend to all other patches; each head captures different spatial relationships |
| **Whisper (speech)** | Cross-attention between audio features and text tokens for transcription |
| **AlphaFold 2** | Evoformer uses multi-head attention over MSA rows and columns |
| **Stable Diffusion** | Cross-attention between image latents and text embeddings for conditional generation |

---

## 8. Summary Table

| Aspect | Detail |
|---|---|
| Formula | `MultiHead(Q,K,V) = Concat(head₁..headₕ)W^O` |
| Per head | `headᵢ = Attention(QWᵢ^Q, KWᵢ^K, VWᵢ^V)` |
| Key dimensions | `d_k = d_v = d_model / h` |
| Standard config | `d_model=512, h=8, d_k=64` |
| Output shape | Same as input: `(batch, seq, d_model)` |
| Efficient trick | Pack all head projections into one large linear layer |
| Masking | Applied to scores before softmax; `-inf` for masked positions |

---

## 9. Revision Questions

1. **Write out the full multi-head attention formula and explain each component.**
   *Hint: Start with the outer Concat · W^O structure, then define each headᵢ.*

2. **Why do we divide by √d_k in the attention scores? What happens without it?**
   *Hint: Variance of dot products grows with d_k, pushing softmax into saturation.*

3. **In the efficient implementation, how is the "loop over heads" eliminated?**
   *Hint: All head projections are packed into a single (d_model × d_model) linear layer, then reshaped.*

4. **Trace the tensor shapes through multi-head attention for `d_model=256, h=4, seq=20, batch=8`.**
   *Answer: (8,20,256) → split → (8,4,20,64) → scores → (8,4,20,20) → context → (8,4,20,64) → concat → (8,20,256) → W_o → (8,20,256).*

5. **What is the difference between self-attention and cross-attention in terms of Q, K, V inputs?**
   *Hint: Self-attention uses Q=K=V=X; cross-attention uses Q from decoder, K=V from encoder.*

6. **Why is the output projection W^O necessary? What would happen without it?**
   *Hint: Without W^O, the model cannot learn to mix information across heads.*

---

[← Why Multiple Heads](01-why-multiple-heads.md) | [Head Concatenation →](03-head-concatenation.md)
