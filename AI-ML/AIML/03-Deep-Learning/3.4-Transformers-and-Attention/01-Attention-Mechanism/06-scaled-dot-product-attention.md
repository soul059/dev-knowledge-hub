# Scaled Dot-Product Attention

[← Previous: Attention Score Functions](05-attention-score-computation.md) | [Next: Additive Attention →](07-additive-attention.md)

---

## Overview

Scaled dot-product attention is the **core computational primitive** of the Transformer architecture. Introduced in "Attention Is All You Need" (Vaswani et al., 2017), it computes attention weights by taking the dot product of queries with keys, dividing by the square root of the key dimension, applying softmax, and then using the resulting weights to combine the values. The scaling factor √d_k is not merely a cosmetic choice — it is a mathematically motivated correction that prevents the dot products from growing too large in magnitude, which would push the softmax into regions of extremely small gradients. This chapter provides the full mathematical derivation of why scaling is necessary, a complete numerical walkthrough, and a production-quality PyTorch implementation with masking support.

---

## The Core Formula

$$
\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V
$$

Where:
- `Q` ∈ ℝ^{n × d_k} — matrix of n query vectors
- `K` ∈ ℝ^{m × d_k} — matrix of m key vectors
- `V` ∈ ℝ^{m × d_v} — matrix of m value vectors
- `d_k` — dimensionality of queries and keys
- The softmax is applied **row-wise** (over the key dimension)

### Dimensions at Each Step

```
Q:      (n × d_k)
K^T:    (d_k × m)
QK^T:   (n × m)       ← raw score matrix
÷√d_k:  (n × m)       ← scaled scores
softmax: (n × m)       ← attention weights (each row sums to 1)
V:       (m × d_v)
Output:  (n × d_v)     ← weighted combination of values
```

---

## Why Scaling by √d_k Is Needed

### The Variance Problem

Assume that the components of q and k are **independent random variables** with mean 0 and variance 1:

```
q_i ~ (μ=0, σ²=1)    for i = 1, ..., d_k
k_i ~ (μ=0, σ²=1)    for i = 1, ..., d_k
```

The dot product is:

```
q · k = Σᵢ qᵢ kᵢ
```

Each term qᵢkᵢ has:
- E[qᵢkᵢ] = E[qᵢ]·E[kᵢ] = 0·0 = 0
- Var(qᵢkᵢ) = E[qᵢ²kᵢ²] - (E[qᵢkᵢ])² = E[qᵢ²]·E[kᵢ²] - 0 = 1·1 = 1

Since the d_k terms are independent:

```
E[q · k] = 0
Var(q · k) = d_k · 1 = d_k
```

**The variance of the dot product grows linearly with d_k!**

### The Softmax Saturation Problem

When d_k is large (e.g., 512), dot products can reach magnitudes of ±30 or more:

```
d_k = 512  →  std(q·k) = √512 ≈ 22.6

Typical dot products range: roughly [-45, +45]
```

When large values enter the softmax:

```
softmax([30, 1, -1, 2]) ≈ [1.0000, 0.0000, 0.0000, 0.0000]

Almost all probability mass on one element!
```

This causes:
1. **Vanishing gradients**: ∂softmax/∂x ≈ 0 when output is near 0 or 1
2. **Loss of expressiveness**: the model can't express "partial attention"
3. **Slow or stalled training**: gradients too small to update weights

### The Fix: Divide by √d_k

After scaling:

```
Var(q · k / √d_k) = Var(q · k) / d_k = d_k / d_k = 1
```

The scaled dot products have **unit variance** regardless of d_k, keeping softmax inputs in a healthy range where gradients flow properly.

### Visualization of the Effect

```
Without scaling (d_k = 512):
  Scores:  [-23.1,  18.7,  2.3,  -5.1, ...]    ← huge spread
  Softmax: [ 0.00,  1.00, 0.00,  0.00, ...]    ← saturated!
  Gradient:  ≈ 0 everywhere                      ← training stalls

With scaling (÷ √512 ≈ 22.6):
  Scores:  [-1.02,  0.83, 0.10, -0.23, ...]     ← moderate spread
  Softmax: [ 0.09,  0.56, 0.27,  0.19, ...]     ← spread out
  Gradient:  healthy non-zero values              ← training works!
```

---

## Full Computation Pipeline

```
            Q (n × d_k)              K (m × d_k)             V (m × d_v)
               │                        │                        │
               │                   Transpose                     │
               │                        │                        │
               │                   K^T (d_k × m)                │
               │                        │                        │
               └──── MatMul ────────────┘                        │
                        │                                        │
                   QK^T (n × m)                                  │
                        │                                        │
                  ÷ √d_k (scale)                                 │
                        │                                        │
                ┌───────┴───────┐                                │
                │  Optional:    │                                │
                │  Apply Mask   │  (set masked positions to -∞)  │
                │  (causal or   │                                │
                │   padding)    │                                │
                └───────┬───────┘                                │
                        │                                        │
                   Softmax (row-wise)                            │
                        │                                        │
                  Weights (n × m)                                │
                        │                                        │
                  ┌─────┴──── MatMul ────────────────────────────┘
                  │
            Output (n × d_v)
```

---

## Step-by-Step Worked Example

### Setup

```
d_k = 3,  d_v = 2,  n = 4 (queries),  m = 4 (keys/values)

Q (4×3):              K (4×3):              V (4×2):
┌           ┐         ┌           ┐         ┌        ┐
│  1   0   1 │         │  1   1   0 │         │  1   0 │
│  0   1   0 │         │  0   1   1 │         │  0   1 │
│  1   1   0 │         │  1   0   1 │         │  1   1 │
│  0   0   1 │         │  0   0   1 │         │  0   0 │
└           ┘         └           ┘         └        ┘
```

### Step 1: Compute QK^T

```
QK^T = Q · K^T

K^T (3×4):
┌              ┐
│  1   0   1   0 │
│  1   1   0   0 │
│  0   1   1   1 │
└              ┘

QK^T (4×4):
Row 0: [1·1+0·1+1·0, 1·0+0·1+1·1, 1·1+0·0+1·1, 1·0+0·0+1·1] = [1, 1, 2, 1]
Row 1: [0·1+1·1+0·0, 0·0+1·1+0·1, 0·1+1·0+0·1, 0·0+1·0+0·1] = [1, 1, 0, 0]
Row 2: [1·1+1·1+0·0, 1·0+1·1+0·1, 1·1+1·0+0·1, 1·0+1·0+0·1] = [2, 1, 1, 0]
Row 3: [0·1+0·1+1·0, 0·0+0·1+1·1, 0·1+0·0+1·1, 0·0+0·0+1·1] = [0, 1, 1, 1]

QK^T =
┌              ┐
│  1   1   2   1 │
│  1   1   0   0 │
│  2   1   1   0 │
│  0   1   1   1 │
└              ┘
```

### Step 2: Scale by √d_k

```
√d_k = √3 ≈ 1.732

Scaled = QK^T / 1.732 =
┌                          ┐
│  0.577   0.577   1.155   0.577 │
│  0.577   0.577   0.000   0.000 │
│  1.155   0.577   0.577   0.000 │
│  0.000   0.577   0.577   0.577 │
└                          ┘
```

### Step 3: Apply Softmax (Row-wise)

```
Row 0: softmax([0.577, 0.577, 1.155, 0.577])
  exp values: [1.781, 1.781, 3.174, 1.781]   sum = 8.517
  weights:    [0.209, 0.209, 0.373, 0.209]

Row 1: softmax([0.577, 0.577, 0.000, 0.000])
  exp values: [1.781, 1.781, 1.000, 1.000]   sum = 5.562
  weights:    [0.320, 0.320, 0.180, 0.180]

Row 2: softmax([1.155, 0.577, 0.577, 0.000])
  exp values: [3.174, 1.781, 1.781, 1.000]   sum = 7.736
  weights:    [0.410, 0.230, 0.230, 0.129]

Row 3: softmax([0.000, 0.577, 0.577, 0.577])
  exp values: [1.000, 1.781, 1.781, 1.781]   sum = 6.343
  weights:    [0.158, 0.281, 0.281, 0.281]

Attention Weights ≈
┌                          ┐
│  0.209   0.209   0.373   0.209 │
│  0.320   0.320   0.180   0.180 │
│  0.410   0.230   0.230   0.129 │
│  0.158   0.281   0.281   0.281 │
└                          ┘
```

### Step 4: Multiply by V

```
Output = Weights · V    (4×4) · (4×2) = (4×2)

Row 0: [0.209·1+0.209·0+0.373·1+0.209·0,  0.209·0+0.209·1+0.373·1+0.209·0]
      = [0.582, 0.582]

Row 1: [0.320·1+0.320·0+0.180·1+0.180·0,  0.320·0+0.320·1+0.180·1+0.180·0]
      = [0.500, 0.500]

Row 2: [0.410·1+0.230·0+0.230·1+0.129·0,  0.410·0+0.230·1+0.230·1+0.129·0]
      = [0.641, 0.461]

Row 3: [0.158·1+0.281·0+0.281·1+0.281·0,  0.158·0+0.281·1+0.281·1+0.281·0]
      = [0.438, 0.562]

Final Output ≈
┌              ┐
│  0.582  0.582 │
│  0.500  0.500 │
│  0.641  0.461 │
│  0.438  0.562 │
└              ┘
```

---

## Numerical Stability Considerations

### Problem: Softmax Overflow

For very large scores, `exp(x)` can overflow to infinity:

```
exp(1000) = Inf    (float32 max ≈ 3.4 × 10^38)
```

### Solution: Subtract the Maximum

The log-sum-exp trick makes softmax numerically stable:

```
softmax(x_i) = exp(x_i - max(x)) / Σ exp(x_j - max(x))
```

This is mathematically equivalent but prevents overflow since `x_i - max(x) ≤ 0`.

### Problem: Underflow in Half-Precision

In float16 (used for efficiency), the smallest positive number is ~6 × 10⁻⁸. Very small attention weights can underflow to zero.

### Masking: Padding and Causal

**Padding mask**: Set scores to -∞ for padding tokens so softmax gives them zero weight.

**Causal mask**: Prevent position i from attending to positions j > i (future tokens). Essential for autoregressive generation.

```
Causal mask for seq_len=4:

     k₁   k₂   k₃   k₄
q₁ [  0   -∞   -∞   -∞ ]    ← can only see itself
q₂ [  0    0   -∞   -∞ ]    ← can see positions 1-2
q₃ [  0    0    0   -∞ ]    ← can see positions 1-3
q₄ [  0    0    0    0 ]    ← can see all positions
```

---

## PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math


class ScaledDotProductAttention(nn.Module):
    """
    Scaled Dot-Product Attention as described in
    'Attention Is All You Need' (Vaswani et al., 2017).

    Attention(Q, K, V) = softmax(QK^T / sqrt(d_k)) V
    """

    def __init__(self, dropout: float = 0.0):
        super().__init__()
        self.dropout = nn.Dropout(p=dropout)

    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        mask: torch.Tensor = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Args:
            query: (batch, n_heads, seq_q, d_k)
            key:   (batch, n_heads, seq_k, d_k)
            value: (batch, n_heads, seq_k, d_v)
            mask:  (batch, 1, 1, seq_k) or (batch, 1, seq_q, seq_k)
                   True/1 for positions to MASK (set to -inf)

        Returns:
            output:  (batch, n_heads, seq_q, d_v)
            weights: (batch, n_heads, seq_q, seq_k)
        """
        d_k = query.size(-1)

        # Step 1: Compute raw scores via matmul
        # (batch, heads, seq_q, d_k) × (batch, heads, d_k, seq_k)
        scores = torch.matmul(query, key.transpose(-2, -1))

        # Step 2: Scale by sqrt(d_k)
        scores = scores / math.sqrt(d_k)

        # Step 3: Apply mask (optional)
        if mask is not None:
            scores = scores.masked_fill(mask == 1, float('-inf'))

        # Step 4: Softmax over key dimension
        weights = F.softmax(scores, dim=-1)

        # Step 5: Apply dropout to attention weights
        weights = self.dropout(weights)

        # Step 6: Weighted sum of values
        output = torch.matmul(weights, value)

        return output, weights


def create_causal_mask(seq_len: int) -> torch.Tensor:
    """Creates an upper-triangular causal mask.
    Returns a mask where future positions are True (to be masked)."""
    mask = torch.triu(torch.ones(seq_len, seq_len), diagonal=1).bool()
    return mask.unsqueeze(0).unsqueeze(0)  # (1, 1, seq_len, seq_len)


def create_padding_mask(lengths: torch.Tensor, max_len: int) -> torch.Tensor:
    """Creates a padding mask from sequence lengths.
    Returns a mask where padding positions are True."""
    batch_size = lengths.size(0)
    positions = torch.arange(max_len).unsqueeze(0).expand(batch_size, -1)
    mask = positions >= lengths.unsqueeze(1)
    return mask.unsqueeze(1).unsqueeze(1)  # (batch, 1, 1, max_len)


# ── Demo ────────────────────────────────────────────────────────────

def demo():
    torch.manual_seed(42)

    batch, heads, seq_len, d_k, d_v = 1, 1, 4, 3, 2

    Q = torch.tensor([[[[1,0,1],[0,1,0],[1,1,0],[0,0,1]]]], dtype=torch.float)
    K = torch.tensor([[[[1,1,0],[0,1,1],[1,0,1],[0,0,1]]]], dtype=torch.float)
    V = torch.tensor([[[[1,0],[0,1],[1,1],[0,0]]]], dtype=torch.float)

    attn = ScaledDotProductAttention(dropout=0.0)

    # Without mask
    output, weights = attn(Q, K, V)
    print("=== Without Mask ===")
    print(f"Attention Weights:\n{weights.squeeze()}")
    print(f"Output:\n{output.squeeze()}\n")

    # With causal mask
    causal_mask = create_causal_mask(seq_len)
    output_causal, weights_causal = attn(Q, K, V, mask=causal_mask)
    print("=== With Causal Mask ===")
    print(f"Attention Weights:\n{weights_causal.squeeze()}")
    print(f"Output:\n{output_causal.squeeze()}\n")

    # With padding mask (last token is padding)
    lengths = torch.tensor([3])  # actual length is 3, position 4 is padding
    pad_mask = create_padding_mask(lengths, seq_len)
    output_pad, weights_pad = attn(Q, K, V, mask=pad_mask)
    print("=== With Padding Mask (last position masked) ===")
    print(f"Attention Weights:\n{weights_pad.squeeze()}")
    print(f"Output:\n{output_pad.squeeze()}\n")

    # Demonstrate the effect of scaling
    print("=== Effect of Scaling ===")
    for d in [4, 64, 512, 2048]:
        q = torch.randn(1, 1, 1, d)
        k = torch.randn(1, 1, 100, d)
        raw_scores = torch.matmul(q, k.transpose(-2, -1))
        scaled_scores = raw_scores / math.sqrt(d)
        print(f"  d_k={d:>5}  |  raw std={raw_scores.std():.2f}  "
              f"|  scaled std={scaled_scores.std():.2f}")


if __name__ == "__main__":
    demo()
```

### Expected Output Highlights

```
=== Effect of Scaling ===
  d_k=    4  |  raw std=1.87  |  scaled std=0.94
  d_k=   64  |  raw std=7.93  |  scaled std=0.99
  d_k=  512  |  raw std=22.12 |  scaled std=0.98
  d_k= 2048  |  raw std=44.89 |  scaled std=0.99
                                        ↑
                    Scaling keeps std ≈ 1 regardless of d_k!
```

---

## Real-World Applications

### 1. Large Language Models (GPT, LLaMA, etc.)
Every token prediction in GPT-style models uses scaled dot-product attention with a causal mask to ensure autoregressive generation — each token can only attend to previous tokens.

### 2. BERT and Bidirectional Models
BERT uses scaled dot-product attention without causal masking, allowing each token to attend to all other tokens. Padding masks are used to handle variable-length inputs in batched processing.

### 3. Vision Transformers (ViT)
Images are split into patches and treated as a sequence. Scaled dot-product attention computes relationships between all pairs of patches, enabling the model to capture global spatial dependencies.

### 4. Protein Structure Prediction (AlphaFold)
AlphaFold uses attention to model pairwise amino acid interactions. The scaling ensures stable gradient flow across very long protein sequences (hundreds to thousands of residues).

### 5. Music Generation
Transformer-based music models use masked scaled dot-product attention to generate notes sequentially, with the attention mechanism capturing harmonic and temporal patterns.

---

## Summary Table

| Concept                     | Key Point                                                    |
|-----------------------------|--------------------------------------------------------------|
| Core formula                | softmax(QK^T / √d_k) · V                                    |
| Why scale by √d_k           | Dot product variance = d_k; scaling normalizes to variance 1 |
| Without scaling             | Softmax saturates → vanishing gradients → training fails     |
| Causal mask                 | Upper-triangle set to -∞; prevents attending to future       |
| Padding mask                | Padding positions set to -∞; prevents attending to padding   |
| Numerical stability         | Softmax uses max-subtraction trick internally                |
| Output shape                | (n × d_v) — same sequence length as queries                  |
| Computational complexity    | O(n · m · d_k) for score computation                        |

---

## Revision Questions

**Q1.** Write the full formula for scaled dot-product attention. What is the purpose of each component (Q, K, V, √d_k, softmax)?

**Q2.** Prove mathematically that the variance of the dot product q·k equals d_k when each component has zero mean and unit variance.

**Q3.** Given Q = [[1,0],[0,1]], K = [[1,1],[0,1]], V = [[1,0],[0,1]], compute the full attention output step by step (d_k = 2).

**Q4.** Explain why a causal mask is necessary for autoregressive language models but not for BERT. How is the mask implemented in the score matrix?

**Q5.** What happens to the gradient of softmax when one input is much larger than the others? How does this relate to the need for scaling?

**Q6.** In half-precision (float16) training, what additional numerical stability concerns arise in attention computation? How can they be addressed?

---

[← Previous: Attention Score Functions](05-attention-score-computation.md) | [Next: Additive Attention →](07-additive-attention.md)
