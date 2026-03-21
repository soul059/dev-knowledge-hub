# Head Concatenation and Output Projection

[← Multi-Head Computation](02-multi-head-computation.md) | [Parameter Efficiency →](04-parameter-efficiency.md)

---

## Overview

After each attention head independently computes its context vector in a reduced-dimension subspace, the results must be reassembled into a single representation of the original model dimension. This is done in two steps: **concatenation** along the feature dimension, which simply stacks the head outputs side by side, and **output projection** via a learned matrix `W^O`, which mixes information across heads to produce the final output. Understanding this pipeline — and especially the role of `W^O` — is essential to grasping why multi-head attention works as a cohesive unit rather than just `h` isolated attention modules.

---

## 1. The Concatenation Operation

### What Happens

Each head produces an output of shape `(n × d_v)`, where `n` is the sequence length and `d_v = d_model / h`. Concatenation stacks all `h` head outputs along the last (feature) dimension:

```
Concat(head₁, head₂, ..., headₕ) = [head₁ ; head₂ ; ... ; headₕ]
```

### Dimension Arithmetic

```
head₁ ∈ ℝ^(n × d_v)       d_v = d_model / h
head₂ ∈ ℝ^(n × d_v)
  ...
headₕ ∈ ℝ^(n × d_v)

Concat result ∈ ℝ^(n × (h · d_v))  =  ℝ^(n × d_model)
```

The critical identity:

```
h × d_v  =  h × (d_model / h)  =  d_model
```

This guarantees the concatenated output has the **same feature dimension** as the original input.

---

## 2. ASCII Diagram — Concatenation Along the Feature Axis

```
  Head 1 output        Head 2 output        Head 3 output        Head 4 output
  (n × d_v)            (n × d_v)            (n × d_v)            (n × d_v)
  d_v = 128            d_v = 128            d_v = 128            d_v = 128

  ┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
  │ tok₁ ... │         │ tok₁ ... │         │ tok₁ ... │         │ tok₁ ... │
  │ tok₂ ... │         │ tok₂ ... │         │ tok₂ ... │         │ tok₂ ... │
  │ tok₃ ... │         │ tok₃ ... │         │ tok₃ ... │         │ tok₃ ... │
  │  ...     │         │  ...     │         │  ...     │         │  ...     │
  │ tokₙ ... │         │ tokₙ ... │         │ tokₙ ... │         │ tokₙ ... │
  └──────────┘         └──────────┘         └──────────┘         └──────────┘
       │                    │                    │                    │
       └────────────────────┼────────────────────┼────────────────────┘
                            │
                            ▼
               ┌─────────────────────────────────────────────┐
               │              Concatenated Output            │
               │            (n × d_model)                    │
               │          d_model = 4 × 128 = 512            │
               │                                             │
               │  tok₁: [head1_feats | head2 | head3 | head4]│
               │  tok₂: [head1_feats | head2 | head3 | head4]│
               │  tok₃: [head1_feats | head2 | head3 | head4]│
               │   ...                                       │
               └─────────────────────────────────────────────┘
                            │
                            ▼
               ┌─────────────────────────────────────────────┐
               │          Output Projection (W^O)            │
               │     (d_model × d_model) = (512 × 512)       │
               │                                             │
               │  Mixes information across all heads          │
               └─────────────────────────────────────────────┘
                            │
                            ▼
               ┌─────────────────────────────────────────────┐
               │              Final Output                   │
               │            (n × d_model)                    │
               └─────────────────────────────────────────────┘
```

---

## 3. The Output Projection Matrix W^O

### Definition

```
W^O ∈ ℝ^(d_model × d_model)
```

Applied as:

```
Output = Concat(head₁, ..., headₕ) · W^O
       (n × d_model)   ·  (d_model × d_model)  =  (n × d_model)
```

### Why W^O Is Necessary

Without `W^O`, each position in the output would simply be a concatenation of independent head results — features from Head 1 would never interact with features from Head 2. The output projection serves three critical purposes:

| Purpose | Explanation |
|---|---|
| **Cross-head mixing** | Allows the model to combine information captured by different heads |
| **Dimensionality mapping** | Ensures the output matches `d_model` regardless of `h × d_v` changes |
| **Learned re-weighting** | Some heads may be more important for certain positions; `W^O` learns this |

### What Happens Without W^O

```
Without W^O:
  Output[pos] = [head₁[pos] | head₂[pos] | ... | headₕ[pos]]

  → Head outputs are simply stacked
  → No interaction between head features
  → Downstream layers must do all the mixing → less efficient

With W^O:
  Output[pos] = W^O · [head₁[pos] | head₂[pos] | ... | headₕ[pos]]

  → W^O row j takes a weighted sum of ALL head features
  → Each output feature is a learned combination of all heads
  → Much richer representations immediately available
```

---

## 4. Worked Example: Concatenation and Projection

### Setup

```
d_model = 4,  h = 2,  d_v = 2
Sequence length n = 3
```

### Head Outputs (from the previous section's computation)

```
head₁ = ┌ 0.75   0.73 ┐       head₂ = ┌ 0.51   0.62 ┐
         │ 0.75   0.73 │                │ 0.49   0.60 │
         └ 0.75   0.73 ┘                └ 0.51   0.62 ┘
        (3 × 2)                        (3 × 2)
```

### Step 1: Concatenation

Concatenate along the feature dimension (axis=-1):

```
Concat = ┌ 0.75   0.73   0.51   0.62 ┐
          │ 0.75   0.73   0.49   0.60 │
          └ 0.75   0.73   0.51   0.62 ┘
         (3 × 4) = (n × d_model)  ✓
```

Each row now has `d_model = 4` features: the first 2 from Head 1, the last 2 from Head 2.

### Step 2: Output Projection

```
W^O = ┌  0.3   0.1   0.4   0.2 ┐
      │  0.5   0.2   0.1   0.3 │
      │  0.2   0.6   0.3   0.1 │
      └  0.1   0.4   0.5   0.4 ┘
     (4 × 4)
```

Compute `Output = Concat · W^O`:

```
Row 0 of Output:
  [0.75, 0.73, 0.51, 0.62] · W^O

  Feature 0: 0.75×0.3 + 0.73×0.5 + 0.51×0.2 + 0.62×0.1
           = 0.225 + 0.365 + 0.102 + 0.062
           = 0.754

  Feature 1: 0.75×0.1 + 0.73×0.2 + 0.51×0.6 + 0.62×0.4
           = 0.075 + 0.146 + 0.306 + 0.248
           = 0.775

  Feature 2: 0.75×0.4 + 0.73×0.1 + 0.51×0.3 + 0.62×0.5
           = 0.300 + 0.073 + 0.153 + 0.310
           = 0.836

  Feature 3: 0.75×0.2 + 0.73×0.3 + 0.51×0.1 + 0.62×0.4
           = 0.150 + 0.219 + 0.051 + 0.248
           = 0.668

Row 0 = [0.754, 0.775, 0.836, 0.668]
```

Similarly for rows 1 and 2:

```
Output = ┌ 0.754   0.775   0.836   0.668 ┐
          │ 0.746   0.767   0.828   0.660 │
          └ 0.754   0.775   0.836   0.668 ┘
         (3 × 4) = (n × d_model)  ✓
```

**Notice:** Output feature 0 (`0.754`) combines information from both heads — `W^O` mixes Head 1's syntactic features with Head 2's semantic features into a unified representation.

---

## 5. Implementation in Code — Reshape View

The efficient implementation avoids explicit concatenation by using `view` / `reshape`:

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class ConcatAndProject(nn.Module):
    """Demonstrates the concat + project step in isolation."""

    def __init__(self, d_model: int, num_heads: int):
        super().__init__()
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_v = d_model // num_heads
        self.W_o = nn.Linear(d_model, d_model, bias=False)

    def forward(self, head_outputs):
        """
        Args:
            head_outputs: (batch, num_heads, seq_len, d_v)
        Returns:
            output: (batch, seq_len, d_model)
        """
        batch_size, h, seq_len, d_v = head_outputs.shape

        # Transpose: (batch, heads, seq, d_v) → (batch, seq, heads, d_v)
        x = head_outputs.transpose(1, 2)

        # Reshape (implicit concat): (batch, seq, heads, d_v) → (batch, seq, d_model)
        x = x.contiguous().view(batch_size, seq_len, self.d_model)

        # Output projection
        output = self.W_o(x)

        return output


# Demo
d_model = 512
num_heads = 8
d_v = 64
seq_len = 10
batch_size = 2

torch.manual_seed(42)
module = ConcatAndProject(d_model, num_heads)

# Simulated head outputs
head_outputs = torch.randn(batch_size, num_heads, seq_len, d_v)

output = module(head_outputs)
print(f"Head outputs shape: {head_outputs.shape}")  # (2, 8, 10, 64)
print(f"Final output shape: {output.shape}")         # (2, 10, 512)
```

### Step-by-Step Shape Transformations

```python
# Starting shape: (batch, heads, seq, d_v) = (2, 8, 10, 64)

# After transpose(1, 2):
#   (batch, seq, heads, d_v) = (2, 10, 8, 64)
#   Explanation: bring heads next to d_v for the contiguous concat

# After view(batch, seq, d_model):
#   (batch, seq, d_model) = (2, 10, 512)
#   Explanation: 8 × 64 = 512 features merged into one dimension

# After W_o linear:
#   (batch, seq, d_model) = (2, 10, 512)
#   Explanation: learned linear combination across all 512 features
```

---

## 6. Explicit vs. Implicit Concatenation

```python
# Method 1: Explicit concatenation (educational, slower)
heads = [torch.randn(2, 10, 64) for _ in range(8)]
concat_explicit = torch.cat(heads, dim=-1)          # (2, 10, 512)

# Method 2: Implicit via reshape (production, faster)
stacked = torch.stack(heads, dim=1)                  # (2, 8, 10, 64)
concat_implicit = stacked.transpose(1, 2).contiguous().view(2, 10, 512)

# Verify equivalence
print(torch.allclose(concat_explicit, concat_implicit))  # True
```

**Why implicit is faster:** No memory copy — `view()` just reinterprets the same memory buffer with a new shape (assuming contiguity). `torch.cat` allocates a new tensor and copies data from each head.

---

## 7. Dimension Management Deep Dive

### The d_model = h × d_k Relationship

This is not arbitrary — it is **engineered** so that concatenation perfectly reconstructs the original dimension:

```
┌──────────────────────────────────────────────────────────────────┐
│                    Dimension Budget                              │
│                                                                  │
│  Total feature budget:  d_model = 512                            │
│                                                                  │
│  Configuration A:  h=8,  d_k=64   →  8 × 64  = 512  ✓          │
│  Configuration B:  h=16, d_k=32   →  16 × 32 = 512  ✓          │
│  Configuration C:  h=4,  d_k=128  →  4 × 128 = 512  ✓          │
│  Configuration D:  h=1,  d_k=512  →  1 × 512 = 512  ✓ (single) │
│                                                                  │
│  In every case:  h × d_k = d_model                               │
│  Concat output always has d_model features                        │
│  W^O always maps d_model → d_model                                │
└──────────────────────────────────────────────────────────────────┘
```

### What If d_model Is Not Divisible by h?

```python
d_model = 512
h = 6  # 512 / 6 = 85.33... → NOT an integer!

# This will fail:
assert d_model % h == 0, f"d_model ({d_model}) must be divisible by h ({h})"
# AssertionError: d_model (512) must be divisible by h (6)

# Solution: choose h that divides d_model evenly
# Valid h values for d_model=512: 1, 2, 4, 8, 16, 32, 64, 128, 256, 512
```

---

## 8. Full Integration — End-to-End Multi-Head Attention

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class MultiHeadAttentionFull(nn.Module):
    """Complete multi-head attention with clear concat + project steps."""

    def __init__(self, d_model: int, num_heads: int, dropout: float = 0.0):
        super().__init__()
        assert d_model % num_heads == 0
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        self.W_q = nn.Linear(d_model, d_model, bias=False)
        self.W_k = nn.Linear(d_model, d_model, bias=False)
        self.W_v = nn.Linear(d_model, d_model, bias=False)
        self.W_o = nn.Linear(d_model, d_model, bias=False)
        self.dropout = nn.Dropout(dropout)

    def forward(self, Q, K, V, mask=None):
        B, Sq, _ = Q.shape
        _, Sk, _ = K.shape

        # ── Step 1: Project ──────────────────────────────────────
        Q = self.W_q(Q)  # (B, Sq, d_model)
        K = self.W_k(K)  # (B, Sk, d_model)
        V = self.W_v(V)  # (B, Sk, d_model)

        # ── Step 2: Split heads ──────────────────────────────────
        Q = Q.view(B, Sq, self.num_heads, self.d_k).transpose(1, 2)
        K = K.view(B, Sk, self.num_heads, self.d_k).transpose(1, 2)
        V = V.view(B, Sk, self.num_heads, self.d_k).transpose(1, 2)
        # Shape: (B, h, S, d_k)

        # ── Step 3: Attention scores ─────────────────────────────
        scores = torch.matmul(Q, K.transpose(-2, -1)) / (self.d_k ** 0.5)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float('-inf'))
        attn = self.dropout(F.softmax(scores, dim=-1))

        # ── Step 4: Weighted values ──────────────────────────────
        context = torch.matmul(attn, V)  # (B, h, Sq, d_k)

        # ── Step 5: CONCATENATE heads ────────────────────────────
        context = context.transpose(1, 2).contiguous()  # (B, Sq, h, d_k)
        concat = context.view(B, Sq, self.d_model)       # (B, Sq, d_model)

        # ── Step 6: OUTPUT PROJECTION ────────────────────────────
        output = self.W_o(concat)  # (B, Sq, d_model)

        return output, attn


# ── Verification ──────────────────────────────────────────────────
torch.manual_seed(0)
mha = MultiHeadAttentionFull(d_model=512, num_heads=8)

x = torch.randn(2, 10, 512)
out, attn = mha(x, x, x)

print(f"Input:         {x.shape}")       # (2, 10, 512)
print(f"Output:        {out.shape}")      # (2, 10, 512)
print(f"Attn weights:  {attn.shape}")     # (2, 8, 10, 10)
print(f"W_o params:    {mha.W_o.weight.shape}")  # (512, 512)
print(f"Attn sum (should be 1.0): {attn[0, 0, 0].sum().item():.4f}")
```

---

## 9. Real-World Applications

| Application | Concatenation & Projection Role |
|---|---|
| **GPT-series** | W^O mixes head outputs before residual connection and layer norm |
| **BERT** | Same architecture; each Transformer layer applies concat + W^O |
| **Multi-Query Attention (MQA)** | Shares K, V across heads but still uses per-head Q; concatenation unchanged |
| **Grouped-Query Attention (GQA)** | Groups of heads share K, V; concat merges group outputs |
| **Mixture of Experts (MoE)** | Analogous gating: expert outputs are combined via learned weights |

---

## 10. Summary Table

| Concept | Detail |
|---|---|
| Concatenation | Stacks `h` head outputs along feature axis: `(n, d_v) × h → (n, d_model)` |
| Key identity | `h × d_v = d_model` (always holds by design) |
| W^O shape | `(d_model × d_model)` |
| W^O purpose | Cross-head information mixing; learned reweighting |
| Without W^O | Heads are isolated; no inter-head interaction |
| Efficient concat | `transpose(1,2).contiguous().view(B, S, d_model)` — no copy |
| Divisibility | `d_model % h == 0` is required; otherwise concat doesn't fill d_model |

---

## 11. Revision Questions

1. **What is the shape of the concatenated output and why does it equal `(n, d_model)`?**
   *Hint: h × d_v = h × (d_model / h) = d_model.*

2. **Explain why the output projection W^O is necessary. What would happen if we removed it?**
   *Hint: Without W^O, features from different heads never interact.*

3. **In the efficient PyTorch implementation, how is concatenation performed without calling `torch.cat`?**
   *Hint: `transpose(1, 2).contiguous().view(B, seq, d_model)` reinterprets memory layout.*

4. **If `d_model = 768` and `h = 12`, what is `d_k`? Verify that `h × d_k = d_model`.**
   *Answer: d_k = 768/12 = 64. Check: 12 × 64 = 768. ✓*

5. **How does Grouped-Query Attention (GQA) modify the standard concatenation step?**
   *Hint: Multiple query heads share the same K, V; the concat step remains the same, but fewer K, V projections are needed.*

6. **Why does `.contiguous()` need to be called before `.view()` in the efficient implementation?**
   *Hint: `transpose` makes the tensor non-contiguous in memory; `view` requires contiguous memory layout.*

---

[← Multi-Head Computation](02-multi-head-computation.md) | [Parameter Efficiency →](04-parameter-efficiency.md)
