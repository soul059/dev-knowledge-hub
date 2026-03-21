# Masked Self-Attention for Autoregressive Generation

> **Navigation:**
> Previous: [← Rotary Positional Encoding](../05-Positional-Encoding/05-rotary-positional-encoding.md) | Next: [Label Smoothing →](02-label-smoothing.md)

---

## Overview

Masked self-attention is the cornerstone mechanism that enables Transformers to perform **autoregressive generation** — producing one token at a time, left to right, without ever "peeking" at tokens that haven't been generated yet. During training the entire target sequence is available, so we must artificially prevent each position from attending to future positions using a **causal mask** (an upper-triangular matrix of negative infinity values applied before the softmax). This single constraint turns the standard bidirectional self-attention into a unidirectional mechanism that mirrors the sequential nature of text generation while still allowing the model to train on all positions in parallel. Masked self-attention is used in every decoder layer of the original Transformer, in GPT-family models, and in the decoder side of encoder–decoder architectures.

---

## 1. Why Masking Is Needed

### The Autoregressive Property

An autoregressive language model factorizes the probability of a sequence as:

```
P(x₁, x₂, ..., xₙ) = ∏ᵢ₌₁ⁿ P(xᵢ | x₁, x₂, ..., xᵢ₋₁)
```

Each token `xᵢ` may only depend on **previous** tokens `x₁ … xᵢ₋₁`.

### The Training Paradox

During inference the model generates one token at a time, so future tokens literally do not exist. During training, however, the full target sequence is fed in at once (teacher forcing). Without masking, position `i` could attend to positions `i+1, i+2, …, n` — effectively cheating by reading the answer.

### What the Mask Does

The causal mask ensures that, even though all tokens are processed in parallel during training, each position **behaves as if** future tokens are absent. This gives us the best of both worlds: parallel training speed with autoregressive correctness.

---

## 2. The Mathematics of Causal Masking

### Standard Self-Attention (Unmasked)

```
Attention(Q, K, V) = softmax(QKᵀ / √dₖ) · V
```

### Masked Self-Attention

We add a mask matrix **M** before the softmax:

```
MaskedAttention(Q, K, V) = softmax((QKᵀ / √dₖ) + M) · V
```

where the mask matrix **M** ∈ ℝⁿˣⁿ is defined as:

```
M[i][j] = 0       if j ≤ i   (current or past positions — allowed)
M[i][j] = -∞      if j > i   (future positions — blocked)
```

### Why -∞ Works

When we add -∞ to a logit before softmax:

```
softmax(zⱼ) = exp(zⱼ) / Σₖ exp(zₖ)

If zⱼ = -∞:  exp(-∞) = 0  →  attention weight = 0
```

The future positions receive **exactly zero** attention weight after softmax — they are completely invisible.

### Upper-Triangular Mask Matrix (for n = 5)

```
M = |  0   -∞   -∞   -∞   -∞ |
    |  0    0   -∞   -∞   -∞ |
    |  0    0    0   -∞   -∞ |
    |  0    0    0    0   -∞ |
    |  0    0    0    0    0  |
```

The mask is **upper-triangular** with -∞ above the diagonal and 0 on/below the diagonal.

---

## 3. ASCII Diagram of Masked Attention

```
 ┌─────────────────────────────────────────────────────────────────┐
 │                  MASKED SELF-ATTENTION FLOW                     │
 └─────────────────────────────────────────────────────────────────┘

  Input Embeddings:  [x₁]  [x₂]  [x₃]  [x₄]
                       │     │     │     │
              ┌────────┴─────┴─────┴─────┴────────┐
              │   Linear Projections (Wq, Wk, Wv)  │
              └────────┬─────┬─────┬─────┬────────┘
                       │     │     │     │
                      Q,K,V for each position
                       │     │     │     │
              ┌────────┴─────┴─────┴─────┴────────┐
              │      Compute QKᵀ / √dₖ             │
              │                                     │
              │  Raw Scores:                        │
              │   ┌──────────────────────┐          │
              │   │ 1.2   0.8   1.5  0.3 │  pos 1  │
              │   │ 0.5   1.1   0.7  1.4 │  pos 2  │
              │   │ 0.9   0.6   1.3  0.8 │  pos 3  │
              │   │ 1.0   0.4   0.2  1.6 │  pos 4  │
              │   └──────────────────────┘          │
              └────────────────┬──────────────────-─┘
                               │
              ┌────────────────▼──────────────────-─┐
              │       ADD CAUSAL MASK                │
              │                                      │
              │   ┌──────────────────────────┐       │
              │   │ 1.2   -∞    -∞    -∞    │ pos 1 │
              │   │ 0.5   1.1   -∞    -∞    │ pos 2 │
              │   │ 0.9   0.6   1.3   -∞    │ pos 3 │
              │   │ 1.0   0.4   0.2   1.6   │ pos 4 │
              │   └──────────────────────────┘       │
              └────────────────┬─────────────────────┘
                               │
              ┌────────────────▼─────────────────────┐
              │         SOFTMAX (row-wise)            │
              │                                       │
              │   ┌──────────────────────────┐        │
              │   │ 1.00  0.00  0.00  0.00   │ pos 1 │
              │   │ 0.35  0.65  0.00  0.00   │ pos 2 │
              │   │ 0.30  0.22  0.48  0.00   │ pos 3 │
              │   │ 0.23  0.13  0.10  0.54   │ pos 4 │
              │   └──────────────────────────┘        │
              └────────────────┬─────────────────────-┘
                               │
              ┌────────────────▼──────────────────────┐
              │     Multiply by V → Output             │
              └────────────────────────────────────────┘
```

---

## 4. Step-by-Step Worked Example

### Setup

Consider a tiny sequence of **3 tokens** with `dₖ = 2`.

```
Q = | 1.0   0.5 |      K = | 0.8   0.2 |      V = | 1.0   0.0 |
    | 0.3   1.2 |          | 0.5   1.0 |          | 0.0   1.0 |
    | 0.7   0.4 |          | 0.1   0.6 |          | 0.5   0.5 |
```

### Step 1: Compute Raw Scores  QKᵀ / √dₖ

```
QKᵀ = | 1.0·0.8+0.5·0.2   1.0·0.5+0.5·1.0   1.0·0.1+0.5·0.6 |
      | 0.3·0.8+1.2·0.2   0.3·0.5+1.2·1.0   0.3·0.1+1.2·0.6 |
      | 0.7·0.8+0.4·0.2   0.7·0.5+0.4·1.0   0.7·0.1+0.4·0.6 |

    = | 0.90   1.00   0.40 |
      | 0.48   1.35   0.75 |
      | 0.64   0.75   0.31 |

Divide by √2 ≈ 1.414:

S = | 0.636   0.707   0.283 |
    | 0.339   0.955   0.530 |
    | 0.453   0.530   0.219 |
```

### Step 2: Apply Causal Mask

```
M = |  0    -∞    -∞  |
    |  0     0    -∞  |
    |  0     0     0  |

S + M = | 0.636    -∞      -∞   |
        | 0.339   0.955    -∞   |
        | 0.453   0.530   0.219 |
```

### Step 3: Apply Softmax (Row-wise)

**Row 0:** Only one valid entry → softmax = [1.000, 0.000, 0.000]

**Row 1:** exp(0.339) = 1.403, exp(0.955) = 2.599
- Sum = 4.002
- softmax = [1.403/4.002, 2.599/4.002, 0] = [**0.351**, **0.649**, **0.000**]

**Row 2:** exp(0.453) = 1.573, exp(0.530) = 1.699, exp(0.219) = 1.245
- Sum = 4.517
- softmax = [0.348, 0.376, 0.276]

```
Attention Weights A:

A = | 1.000   0.000   0.000 |
    | 0.351   0.649   0.000 |
    | 0.348   0.376   0.276 |
```

### Step 4: Compute Output  A · V

```
Output = A · V

Row 0: 1.000·[1.0, 0.0] + 0·[0.0, 1.0] + 0·[0.5, 0.5]
      = [1.000, 0.000]

Row 1: 0.351·[1.0, 0.0] + 0.649·[0.0, 1.0] + 0·[0.5, 0.5]
      = [0.351, 0.649]

Row 2: 0.348·[1.0, 0.0] + 0.376·[0.0, 1.0] + 0.276·[0.5, 0.5]
      = [0.486, 0.514]
```

**Key observation:** Position 0 sees only itself. Position 1 sees positions 0–1. Position 2 sees all positions. No position can attend to future tokens.

---

## 5. Implementation Details

### Mask Creation Strategies

| Strategy | Method | Notes |
|----------|--------|-------|
| `torch.triu` | Upper triangular of ones, then fill with -∞ | Most explicit |
| `torch.ones().tril()` | Lower triangular boolean mask | Memory efficient |
| `torch.masked_fill` | Fill positions where mask is False/True | PyTorch idiomatic |
| Additive mask | Add -∞ directly to scores | Numerically stable |
| Causal attention flag | `F.scaled_dot_product_attention(is_causal=True)` | PyTorch 2.0+ optimized |

### When to Apply the Mask

```
1. Compute Q, K, V projections
2. Compute attention scores: QKᵀ / √dₖ
3. ──► APPLY MASK HERE ◄── (add -∞ to future positions)
4. Apply softmax
5. Multiply by V
```

The mask must be applied **after** computing raw scores and **before** softmax.

---

## 6. PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math


class MaskedSelfAttention(nn.Module):
    """
    Masked (causal) self-attention for autoregressive models.
    Prevents each position from attending to future positions.
    """

    def __init__(self, d_model: int, n_heads: int, max_seq_len: int = 2048, dropout: float = 0.1):
        super().__init__()
        assert d_model % n_heads == 0, "d_model must be divisible by n_heads"

        self.d_model = d_model
        self.n_heads = n_heads
        self.d_k = d_model // n_heads

        # Linear projections for Q, K, V and output
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)

        self.dropout = nn.Dropout(dropout)

        # Pre-compute causal mask and register as buffer (not a parameter)
        # Lower-triangular matrix: mask[i][j] = 1 if j <= i, else 0
        causal_mask = torch.tril(torch.ones(max_seq_len, max_seq_len))
        self.register_buffer("causal_mask", causal_mask.view(1, 1, max_seq_len, max_seq_len))

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        """
        Args:
            x: Input tensor of shape (batch_size, seq_len, d_model)
        Returns:
            Output tensor of shape (batch_size, seq_len, d_model)
        """
        B, T, C = x.shape

        # Step 1: Project to Q, K, V
        Q = self.W_q(x)  # (B, T, d_model)
        K = self.W_k(x)
        V = self.W_v(x)

        # Step 2: Reshape for multi-head attention
        # (B, T, d_model) -> (B, T, n_heads, d_k) -> (B, n_heads, T, d_k)
        Q = Q.view(B, T, self.n_heads, self.d_k).transpose(1, 2)
        K = K.view(B, T, self.n_heads, self.d_k).transpose(1, 2)
        V = V.view(B, T, self.n_heads, self.d_k).transpose(1, 2)

        # Step 3: Compute scaled dot-product attention scores
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)
        # scores shape: (B, n_heads, T, T)

        # Step 4: Apply causal mask — set future positions to -inf
        mask = self.causal_mask[:, :, :T, :T]  # Slice to current sequence length
        scores = scores.masked_fill(mask == 0, float('-inf'))

        # Step 5: Softmax → attention weights
        attn_weights = F.softmax(scores, dim=-1)
        attn_weights = self.dropout(attn_weights)

        # Step 6: Weighted sum of values
        output = torch.matmul(attn_weights, V)  # (B, n_heads, T, d_k)

        # Step 7: Concatenate heads and project
        output = output.transpose(1, 2).contiguous().view(B, T, self.d_model)
        output = self.W_o(output)

        return output


# --- Demo ---
if __name__ == "__main__":
    torch.manual_seed(42)

    batch_size, seq_len, d_model, n_heads = 2, 8, 64, 4
    x = torch.randn(batch_size, seq_len, d_model)

    attn = MaskedSelfAttention(d_model=d_model, n_heads=n_heads, max_seq_len=128)
    output = attn(x)

    print(f"Input shape:  {x.shape}")      # (2, 8, 64)
    print(f"Output shape: {output.shape}")  # (2, 8, 64)

    # Visualize the causal mask for seq_len=5
    mask = torch.tril(torch.ones(5, 5))
    print("\nCausal mask (5x5):")
    print(mask)

    # Show that masked scores produce zeros after softmax
    raw_scores = torch.randn(1, 1, 5, 5)
    masked_scores = raw_scores.masked_fill(mask.unsqueeze(0).unsqueeze(0) == 0, float('-inf'))
    weights = F.softmax(masked_scores, dim=-1)
    print("\nAttention weights (notice upper triangle is 0.0):")
    print(weights.squeeze())
```

### PyTorch 2.0+ Optimized Version

```python
# Using the built-in optimized implementation (Flash Attention compatible)
def causal_attention_pytorch2(Q, K, V, dropout_p=0.0):
    """Uses PyTorch 2.0 scaled_dot_product_attention with is_causal flag."""
    return F.scaled_dot_product_attention(
        Q, K, V,
        attn_mask=None,
        dropout_p=dropout_p,
        is_causal=True  # Automatically applies causal mask with Flash Attention
    )
```

---

## 7. Real-World Applications

| Application | How Masked Attention Is Used |
|-------------|------------------------------|
| **GPT / ChatGPT** | Every layer uses masked self-attention for left-to-right language modeling |
| **Machine Translation (Decoder)** | Decoder generates target tokens one at a time using masked attention |
| **Code Generation (Codex, Copilot)** | Autoregressive code completion — each token depends only on prior code |
| **Text Summarization** | Decoder generates summary autoregressively with causal masking |
| **Music / Audio Generation** | WaveNet, Jukebox use causal masking for sequential audio generation |
| **Protein Sequence Design** | Autoregressive protein generators use masked attention for amino acid sequences |

### Why Not Use Bidirectional Attention Everywhere?

- **Generation requires autoregressive modeling** — you can't attend to tokens that don't exist yet
- **Training must mirror inference** — masking during training ensures the model learns the same conditional distributions it will use at test time
- **Encoder models (BERT)** use bidirectional attention because they don't generate — they only encode

---

## 8. Summary Table

| Concept | Details |
|---------|---------|
| **Purpose** | Prevent future token attention in autoregressive models |
| **Mask type** | Upper-triangular matrix of -∞ values |
| **Applied where** | After QKᵀ/√dₖ scores, before softmax |
| **Effect** | Future positions get attention weight = 0 |
| **Used in** | Transformer decoder, GPT, all autoregressive LLMs |
| **PyTorch function** | `torch.tril()`, `masked_fill()`, or `is_causal=True` |
| **Complexity** | O(n²) for mask creation; O(1) extra per attention computation |
| **Key insight** | Enables parallel training while maintaining autoregressive property |

---

## 9. Revision Questions

1. **Why can't we simply remove future tokens from the input during training instead of using a mask?**
   *Hint: Think about parallelism and computational efficiency.*

2. **What would happen if we used 0 instead of -∞ in the mask positions? Would the model still work correctly?**
   *Hint: Consider what softmax does with zero vs. negative infinity inputs.*

3. **In a Transformer with both an encoder and decoder, which attention layers use causal masking and which don't?**
   *Hint: There are three types of attention in encoder-decoder models.*

4. **How does the causal mask interact with multi-head attention? Is one mask shared across all heads?**
   *Hint: Think about the mask dimensions and broadcasting.*

5. **Explain why Flash Attention can implement causal masking more efficiently than the standard approach.**
   *Hint: Consider memory access patterns and the tiling strategy of Flash Attention.*

6. **During inference (generation), is the causal mask still needed? Why or why not?**
   *Hint: Think about what tokens are available at each generation step when using KV caching.*

---

> **Navigation:**
> Previous: [← Rotary Positional Encoding](../05-Positional-Encoding/05-rotary-positional-encoding.md) | Next: [Label Smoothing →](02-label-smoothing.md)
