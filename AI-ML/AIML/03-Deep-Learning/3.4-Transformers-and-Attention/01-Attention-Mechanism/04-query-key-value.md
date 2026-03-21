# Query, Key, Value Abstraction

> **Previous: [← Attention Weights](03-attention-weights.md) | Next: [Attention Score Computation →](05-attention-score-computation.md)**

---

## Overview

The Query-Key-Value (QKV) framework is the modern formulation of attention that powers Transformer architectures. Instead of computing attention directly between raw hidden states, we project inputs into three separate vector spaces — **Queries** (what am I looking for?), **Keys** (what do I contain?), and **Values** (what information do I provide?) — using learned weight matrices. This abstraction, introduced by Vaswani et al. in "Attention Is All You Need" (2017), is elegant, parallelizable, and forms the backbone of models like BERT, GPT, and all modern large language models. This chapter explains the QKV concept through intuitive analogies, derives the math step-by-step, and implements it in PyTorch with a worked numerical example.

---

## 1. The Database / Dictionary Analogy

The QKV abstraction maps perfectly onto how databases and dictionaries work:

### Dictionary Lookup

```
┌─────────────────────────────────────────────────────────┐
│                   DICTIONARY ANALOGY                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  You have a QUERY:   "What is the capital of France?"   │
│                                                         │
│  Dictionary has KEYS:                                    │
│    Key 1: "Capital of France"      → Value: "Paris"     │
│    Key 2: "Capital of Germany"     → Value: "Berlin"    │
│    Key 3: "Population of France"   → Value: "67M"       │
│                                                         │
│  Process:                                                │
│    1. Compare QUERY against each KEY                     │
│    2. Find best match: Key 1 (highest similarity)        │
│    3. Return corresponding VALUE: "Paris"                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Soft Dictionary Lookup (Attention)

In attention, the lookup is **soft** — instead of returning one value, we return a weighted combination of ALL values, with weights based on query-key similarity:

```
Query: "What is the capital of France?"

  Key 1 ("Capital of France")     → similarity: 0.85  → Value: "Paris"
  Key 2 ("Capital of Germany")    → similarity: 0.10  → Value: "Berlin"
  Key 3 ("Population of France")  → similarity: 0.05  → Value: "67M"

  Result = 0.85 × "Paris" + 0.10 × "Berlin" + 0.05 × "67M"
         ≈ "Paris" (dominated by the best match)
```

### Mapping to Attention

| Concept | Database | Neural Attention |
|---------|----------|-----------------|
| **Query** | Search term / question | Decoder state or token being processed |
| **Key** | Index entries | Projections of encoder/input states |
| **Value** | Stored data | Projections of encoder/input states (can differ from keys) |
| **Similarity** | Exact match | Dot product / scaled dot product |
| **Retrieval** | Return one record | Weighted sum of all values |

---

## 2. How Q, K, V Are Computed

### From Input to QKV

Given an input sequence `X` (e.g., embeddings + positional encodings), we compute Q, K, V using three separate learned weight matrices:

```
Q = X W_Q          (Queries)
K = X W_K          (Keys)
V = X W_V          (Values)
```

### Diagram: QKV Projection Flow

```
                        Input Sequence X
                    (batch, seq_len, d_model)
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
         ┌─────────┐   ┌─────────┐   ┌─────────┐
         │  × W_Q  │   │  × W_K  │   │  × W_V  │
         │         │   │         │   │         │
         │ (d_model│   │ (d_model│   │ (d_model│
         │  × d_k) │   │  × d_k) │   │  × d_v) │
         └────┬────┘   └────┬────┘   └────┬────┘
              │              │              │
              ▼              ▼              ▼
         ┌─────────┐   ┌─────────┐   ┌─────────┐
         │ Queries │   │  Keys   │   │ Values  │
         │  Q      │   │   K     │   │   V     │
         │(batch,  │   │(batch,  │   │(batch,  │
         │ seq_len,│   │ seq_len,│   │ seq_len,│
         │ d_k)    │   │ d_k)    │   │ d_v)    │
         └────┬────┘   └────┬────┘   └────┬────┘
              │              │              │
              │    ┌─────────┘              │
              │    │                        │
              ▼    ▼                        │
         ┌──────────────┐                  │
         │   Q × K^T    │                  │
         │  ─────────   │                  │
         │    √d_k      │                  │
         └──────┬───────┘                  │
                │                          │
                ▼                          │
         ┌──────────────┐                  │
         │   Softmax    │                  │
         └──────┬───────┘                  │
                │                          │
                ▼                          ▼
         ┌──────────────────────────────────┐
         │      Attention Weights × V       │
         │                                  │
         │   Output = softmax(QK^T/√d_k)V  │
         └──────────────┬───────────────────┘
                        │
                        ▼
                   Output (batch, seq_len, d_v)
```

---

## 3. Mathematical Formulation

### Dimensions

```
X:   (n, d_model)     — n tokens, each of dimension d_model
W_Q: (d_model, d_k)   — query projection matrix
W_K: (d_model, d_k)   — key projection matrix
W_V: (d_model, d_v)   — value projection matrix

Q:   (n, d_k)         — query vectors
K:   (n, d_k)         — key vectors
V:   (n, d_v)         — value vectors
```

### Projection

```
Q = X × W_Q    →  (n, d_model) × (d_model, d_k)  = (n, d_k)
K = X × W_K    →  (n, d_model) × (d_model, d_k)  = (n, d_k)
V = X × W_V    →  (n, d_model) × (d_model, d_v)  = (n, d_v)
```

### Scaled Dot-Product Attention

```
Attention(Q, K, V) = softmax( Q K^T / √d_k ) × V
```

Breaking it down:

```
Step 1:  S = Q × K^T              (n, d_k) × (d_k, n) = (n, n)
         S[i][j] = similarity between query_i and key_j

Step 2:  S_scaled = S / √d_k      Scale to prevent softmax saturation

Step 3:  A = softmax(S_scaled)     (n, n) — attention weight matrix
         A[i][j] = how much token i attends to token j

Step 4:  Output = A × V            (n, n) × (n, d_v) = (n, d_v)
         Output[i] = weighted sum of all value vectors for token i
```

### Why Separate Q, K, V?

Having three separate projections gives the model flexibility:

```
┌─────────────────────────────────────────────────────────┐
│  Why not just use X directly (i.e., Q = K = V = X)?    │
│                                                         │
│  1. DIFFERENT ROLES: What you're searching for (Q)      │
│     may differ from what you're indexed by (K)          │
│     which may differ from what you return (V).          │
│                                                         │
│  2. ASYMMETRY: In cross-attention, Q comes from the     │
│     decoder but K, V come from the encoder.             │
│                                                         │
│  3. LEARNED SUBSPACES: W_Q, W_K, W_V project into      │
│     optimized subspaces where similarity is meaningful. │
│                                                         │
│  4. DIMENSION FLEXIBILITY: d_k and d_v can differ       │
│     from d_model, enabling efficient computation.       │
└─────────────────────────────────────────────────────────┘
```

---

## 4. Self-Attention vs Cross-Attention

### Self-Attention

All three (Q, K, V) come from the **same** input sequence:

```
Q = X W_Q
K = X W_K       ← same X for all three
V = X W_V
```

Used in: Transformer encoder, GPT decoder (masked).

### Cross-Attention

Q comes from one sequence, K and V from another:

```
Q = X_decoder × W_Q     ← from decoder
K = X_encoder × W_K     ← from encoder
V = X_encoder × W_V     ← from encoder
```

Used in: Transformer decoder (encoder-decoder attention), machine translation.

```
            Self-Attention                Cross-Attention
     ┌───────────────────────┐    ┌───────────────────────┐
     │   Sentence: "I am"    │    │  Encoder: "I love AI" │
     │                       │    │  Decoder: "J'aime"    │
     │  Q ← "I am"          │    │                       │
     │  K ← "I am"          │    │  Q ← "J'aime"        │
     │  V ← "I am"          │    │  K ← "I love AI"     │
     │                       │    │  V ← "I love AI"     │
     │  Each token attends   │    │                       │
     │  to every other token │    │  Decoder queries      │
     │  in the same sequence │    │  attend to encoder    │
     └───────────────────────┘    └───────────────────────┘
```

---

## 5. Worked Example with Small Matrices

### Setup

Input: 3 tokens, `d_model = 4`, `d_k = d_v = 3`

```
X (input embeddings):

     Token 1  → [1.0,  0.0,  1.0,  0.0]
     Token 2  → [0.0,  1.0,  0.0,  1.0]
     Token 3  → [1.0,  1.0,  0.0,  0.0]

     X = ┌ 1.0  0.0  1.0  0.0 ┐
         │ 0.0  1.0  0.0  1.0 │    (3 × 4)
         └ 1.0  1.0  0.0  0.0 ┘
```

Weight matrices (small values for clarity):

```
W_Q = ┌ 0.1  0.2  0.0 ┐        W_K = ┌ 0.2  0.0  0.1 ┐        W_V = ┌ 1.0  0.0  0.0 ┐
      │ 0.3  0.0  0.1 │              │ 0.0  0.3  0.0 │              │ 0.0  1.0  0.0 │
      │ 0.0  0.1  0.2 │              │ 0.1  0.0  0.2 │              │ 0.0  0.0  1.0 │
      └ 0.2  0.0  0.3 ┘              └ 0.0  0.1  0.0 ┘              └ 0.5  0.5  0.5 ┘
        (4 × 3)                         (4 × 3)                       (4 × 3)
```

### Step 1: Compute Q, K, V

**Q = X × W_Q:**
```
Row 1: [1,0,1,0] × W_Q = [1×0.1+0×0.3+1×0.0+0×0.2, 1×0.2+0×0.0+1×0.1+0×0.0, 1×0.0+0×0.1+1×0.2+0×0.3]
                        = [0.1, 0.3, 0.2]

Row 2: [0,1,0,1] × W_Q = [0×0.1+1×0.3+0×0.0+1×0.2, 0×0.2+1×0.0+0×0.1+1×0.0, 0×0.0+1×0.1+0×0.2+1×0.3]
                        = [0.5, 0.0, 0.4]

Row 3: [1,1,0,0] × W_Q = [1×0.1+1×0.3+0×0.0+0×0.2, 1×0.2+1×0.0+0×0.1+0×0.0, 1×0.0+1×0.1+0×0.2+0×0.3]
                        = [0.4, 0.2, 0.1]

Q = ┌ 0.1  0.3  0.2 ┐
    │ 0.5  0.0  0.4 │    (3 × 3)
    └ 0.4  0.2  0.1 ┘
```

**K = X × W_K:**
```
Row 1: [1,0,1,0] × W_K = [0.2+0+0.1+0, 0+0+0+0, 0.1+0+0.2+0] = [0.3, 0.0, 0.3]
Row 2: [0,1,0,1] × W_K = [0+0+0+0, 0+0.3+0+0.1, 0+0+0+0]     = [0.0, 0.4, 0.0]
Row 3: [1,1,0,0] × W_K = [0.2+0+0+0, 0+0.3+0+0, 0.1+0+0+0]   = [0.2, 0.3, 0.1]

K = ┌ 0.3  0.0  0.3 ┐
    │ 0.0  0.4  0.0 │    (3 × 3)
    └ 0.2  0.3  0.1 ┘
```

**V = X × W_V:**
```
Row 1: [1,0,1,0] × W_V = [1+0+0+0, 0+0+0+0, 0+0+1+0]     = [1.0, 0.0, 1.0]
Row 2: [0,1,0,1] × W_V = [0+0+0+0.5, 0+1+0+0.5, 0+0+0+0.5] = [0.5, 1.5, 0.5]
Row 3: [1,1,0,0] × W_V = [1+0+0+0, 0+1+0+0, 0+0+0+0]     = [1.0, 1.0, 0.0]

V = ┌ 1.0  0.0  1.0 ┐
    │ 0.5  1.5  0.5 │    (3 × 3)
    └ 1.0  1.0  0.0 ┘
```

### Step 2: Compute Attention Scores `Q × K^T`

```
S = Q × K^T     (3×3) × (3×3) = (3×3)

S[1][1] = 0.1×0.3 + 0.3×0.0 + 0.2×0.3 = 0.03 + 0 + 0.06 = 0.09
S[1][2] = 0.1×0.0 + 0.3×0.4 + 0.2×0.0 = 0 + 0.12 + 0     = 0.12
S[1][3] = 0.1×0.2 + 0.3×0.3 + 0.2×0.1 = 0.02 + 0.09 + 0.02 = 0.13

S[2][1] = 0.5×0.3 + 0.0×0.0 + 0.4×0.3 = 0.15 + 0 + 0.12 = 0.27
S[2][2] = 0.5×0.0 + 0.0×0.4 + 0.4×0.0 = 0                = 0.00
S[2][3] = 0.5×0.2 + 0.0×0.3 + 0.4×0.1 = 0.10 + 0 + 0.04 = 0.14

S[3][1] = 0.4×0.3 + 0.2×0.0 + 0.1×0.3 = 0.12 + 0 + 0.03 = 0.15
S[3][2] = 0.4×0.0 + 0.2×0.4 + 0.1×0.0 = 0 + 0.08 + 0    = 0.08
S[3][3] = 0.4×0.2 + 0.2×0.3 + 0.1×0.1 = 0.08 + 0.06 + 0.01 = 0.15

S = ┌ 0.09  0.12  0.13 ┐
    │ 0.27  0.00  0.14 │    (3 × 3)
    └ 0.15  0.08  0.15 ┘
```

### Step 3: Scale by `√d_k`

```
√d_k = √3 ≈ 1.732

S_scaled = S / 1.732

S_scaled = ┌ 0.052  0.069  0.075 ┐
           │ 0.156  0.000  0.081 │
           └ 0.087  0.046  0.087 ┘
```

### Step 4: Softmax (Row-wise)

```
Row 1: softmax([0.052, 0.069, 0.075])
  exp: [1.053, 1.071, 1.078] → sum = 3.202
  α_1 = [0.329, 0.335, 0.337]

Row 2: softmax([0.156, 0.000, 0.081])
  exp: [1.169, 1.000, 1.084] → sum = 3.253
  α_2 = [0.359, 0.307, 0.333]

Row 3: softmax([0.087, 0.046, 0.087])
  exp: [1.091, 1.047, 1.091] → sum = 3.229
  α_3 = [0.338, 0.324, 0.338]
```

### Step 5: Compute Output = A × V

```
Output[1] = 0.329 × [1.0, 0.0, 1.0]
          + 0.335 × [0.5, 1.5, 0.5]
          + 0.337 × [1.0, 1.0, 0.0]
          = [0.329, 0.000, 0.329]
          + [0.168, 0.503, 0.168]
          + [0.337, 0.337, 0.000]
          = [0.834, 0.840, 0.497]

Output[2] = 0.359 × [1.0, 0.0, 1.0]
          + 0.307 × [0.5, 1.5, 0.5]
          + 0.333 × [1.0, 1.0, 0.0]
          = [0.359, 0.000, 0.359]
          + [0.154, 0.461, 0.154]
          + [0.333, 0.333, 0.000]
          = [0.846, 0.794, 0.513]

Output[3] = 0.338 × [1.0, 0.0, 1.0]
          + 0.324 × [0.5, 1.5, 0.5]
          + 0.338 × [1.0, 1.0, 0.0]
          = [0.338, 0.000, 0.338]
          + [0.162, 0.486, 0.162]
          + [0.338, 0.338, 0.000]
          = [0.838, 0.824, 0.500]
```

---

## 6. Python/PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math


class QKVAttention(nn.Module):
    """
    Query-Key-Value attention with scaled dot-product.

    Implements: Attention(Q, K, V) = softmax(Q K^T / √d_k) V
    """

    def __init__(self, d_model, d_k, d_v):
        """
        Args:
            d_model: input embedding dimension
            d_k:     query and key dimension
            d_v:     value dimension
        """
        super().__init__()
        self.d_k = d_k

        # Learned projection matrices
        self.W_Q = nn.Linear(d_model, d_k, bias=False)
        self.W_K = nn.Linear(d_model, d_k, bias=False)
        self.W_V = nn.Linear(d_model, d_v, bias=False)

    def forward(self, x, mask=None):
        """
        Self-attention: Q, K, V all derived from x.

        Args:
            x:    (batch, seq_len, d_model)
            mask: (batch, seq_len, seq_len) optional attention mask

        Returns:
            output:  (batch, seq_len, d_v)
            weights: (batch, seq_len, seq_len)
        """
        # Step 1: Project to Q, K, V
        Q = self.W_Q(x)    # (batch, seq_len, d_k)
        K = self.W_K(x)    # (batch, seq_len, d_k)
        V = self.W_V(x)    # (batch, seq_len, d_v)

        # Step 2: Compute scaled dot-product scores
        scores = torch.bmm(Q, K.transpose(1, 2))   # (batch, seq_len, seq_len)
        scores = scores / math.sqrt(self.d_k)

        # Step 3: Apply mask (optional, e.g., for causal attention)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float('-inf'))

        # Step 4: Softmax → attention weights
        weights = F.softmax(scores, dim=-1)   # (batch, seq_len, seq_len)

        # Step 5: Weighted sum of values
        output = torch.bmm(weights, V)   # (batch, seq_len, d_v)

        return output, weights


class CrossAttention(nn.Module):
    """
    Cross-attention: Q from one source, K/V from another.
    Used in Transformer decoder for encoder-decoder attention.
    """

    def __init__(self, d_model, d_k, d_v):
        super().__init__()
        self.d_k = d_k
        self.W_Q = nn.Linear(d_model, d_k, bias=False)
        self.W_K = nn.Linear(d_model, d_k, bias=False)
        self.W_V = nn.Linear(d_model, d_v, bias=False)

    def forward(self, x_query, x_kv, mask=None):
        """
        Args:
            x_query: (batch, tgt_len, d_model) — decoder input
            x_kv:    (batch, src_len, d_model) — encoder output
        """
        Q = self.W_Q(x_query)    # (batch, tgt_len, d_k)
        K = self.W_K(x_kv)      # (batch, src_len, d_k)
        V = self.W_V(x_kv)      # (batch, src_len, d_v)

        scores = torch.bmm(Q, K.transpose(1, 2)) / math.sqrt(self.d_k)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float('-inf'))
        weights = F.softmax(scores, dim=-1)
        output = torch.bmm(weights, V)

        return output, weights


# ═══════════════════════════════════════════
#  Demo: Self-Attention with QKV Projection
# ═══════════════════════════════════════════

D_MODEL = 4
D_K = 3
D_V = 3
SEQ_LEN = 3
BATCH = 1

# Input (same as worked example)
X = torch.tensor([[[1.0, 0.0, 1.0, 0.0],
                    [0.0, 1.0, 0.0, 1.0],
                    [1.0, 1.0, 0.0, 0.0]]])  # (1, 3, 4)

# Initialize attention
attn = QKVAttention(D_MODEL, D_K, D_V)

# Manually set weights to match worked example
with torch.no_grad():
    attn.W_Q.weight.copy_(torch.tensor([
        [0.1, 0.3, 0.0, 0.2],
        [0.2, 0.0, 0.1, 0.0],
        [0.0, 0.1, 0.2, 0.3],
    ]))
    attn.W_K.weight.copy_(torch.tensor([
        [0.2, 0.0, 0.1, 0.0],
        [0.0, 0.3, 0.0, 0.1],
        [0.1, 0.0, 0.2, 0.0],
    ]))
    attn.W_V.weight.copy_(torch.tensor([
        [1.0, 0.0, 0.0, 0.5],
        [0.0, 1.0, 0.0, 0.5],
        [0.0, 0.0, 1.0, 0.5],
    ]))

output, weights = attn(X)

print("=" * 55)
print(" QKV Self-Attention — Worked Example Verification")
print("=" * 55)

# Show intermediate values
Q = attn.W_Q(X)
K = attn.W_K(X)
V = attn.W_V(X)

print(f"\nQ (queries):\n{Q.data.squeeze().numpy().round(3)}")
print(f"\nK (keys):\n{K.data.squeeze().numpy().round(3)}")
print(f"\nV (values):\n{V.data.squeeze().numpy().round(3)}")

scores = torch.bmm(Q, K.transpose(1, 2)) / math.sqrt(D_K)
print(f"\nScaled scores (QK^T / √d_k):\n{scores.data.squeeze().numpy().round(4)}")
print(f"\nAttention weights (softmax):\n{weights.data.squeeze().numpy().round(4)}")
print(f"\nOutput:\n{output.data.squeeze().numpy().round(4)}")

# ═══════════════════════════════════════════
#  Demo: Cross-Attention
# ═══════════════════════════════════════════

print("\n" + "=" * 55)
print(" Cross-Attention Demo")
print("=" * 55)

encoder_out = torch.randn(2, 5, D_MODEL)   # batch=2, src_len=5
decoder_in = torch.randn(2, 3, D_MODEL)    # batch=2, tgt_len=3

cross_attn = CrossAttention(D_MODEL, D_K, D_V)
cross_out, cross_wts = cross_attn(decoder_in, encoder_out)

print(f"\nEncoder output shape: {encoder_out.shape}")
print(f"Decoder input shape:  {decoder_in.shape}")
print(f"Cross-attention output shape: {cross_out.shape}")
print(f"Cross-attention weights shape: {cross_wts.shape}")
print(f"Weights sum per query: {cross_wts.sum(dim=-1).data}")

# ═══════════════════════════════════════════
#  Demo: Causal (Masked) Self-Attention
# ═══════════════════════════════════════════

print("\n" + "=" * 55)
print(" Causal (Masked) Self-Attention")
print("=" * 55)

# Create causal mask (lower triangular)
causal_mask = torch.tril(torch.ones(SEQ_LEN, SEQ_LEN)).unsqueeze(0)
print(f"\nCausal mask:\n{causal_mask.squeeze().numpy().astype(int)}")

masked_output, masked_weights = attn(X, mask=causal_mask)
print(f"\nMasked attention weights:\n{masked_weights.data.squeeze().numpy().round(4)}")
print(f"(Each row can only attend to current and previous positions)")
```

---

## 7. Dimensions and Shapes — Complete Reference

```
┌──────────────────────────────────────────────────────────┐
│              DIMENSION REFERENCE CARD                     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Typical Transformer-Base values:                        │
│    d_model = 512     (input/output dimension)            │
│    d_k = d_v = 64    (per-head dimension)                │
│    n_heads = 8       (d_model / d_k = 512 / 64 = 8)     │
│    n = seq_len       (number of tokens)                  │
│                                                          │
│  Shape flow:                                             │
│    X:       (batch, n, d_model)    = (B, n, 512)         │
│    W_Q:     (d_model, d_k)        = (512, 64)           │
│    Q:       (batch, n, d_k)       = (B, n, 64)          │
│    K:       (batch, n, d_k)       = (B, n, 64)          │
│    V:       (batch, n, d_v)       = (B, n, 64)          │
│    QK^T:    (batch, n, n)         = (B, n, n)            │
│    weights: (batch, n, n)         = (B, n, n)            │
│    output:  (batch, n, d_v)       = (B, n, 64)          │
│                                                          │
│  Parameter count per attention head:                     │
│    W_Q: d_model × d_k = 512 × 64 = 32,768              │
│    W_K: d_model × d_k = 512 × 64 = 32,768              │
│    W_V: d_model × d_v = 512 × 64 = 32,768              │
│    Total per head: 98,304                                │
│    Total all heads: 8 × 98,304 = 786,432                │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 8. Real-World Applications

| Application | Q Source | K, V Source | Purpose |
|-------------|---------|-------------|---------|
| **BERT** (encoder) | Same sentence | Same sentence | Bidirectional self-attention for understanding |
| **GPT** (decoder) | Previous tokens | Previous tokens | Causal self-attention for generation |
| **T5 decoder** | Target tokens | Encoder output | Cross-attention for seq2seq tasks |
| **Vision Transformer** | Image patches | Image patches | Self-attention over flattened image patches |
| **DALL-E** | Text tokens | Image tokens | Cross-modal attention for image generation |
| **AlphaFold** | Amino acid residues | Amino acid residues | Structural attention for protein folding |
| **Whisper** | Decoder tokens | Audio features | Cross-attention for speech-to-text |

---

## 9. Summary Table

| Concept | Description |
|---------|-------------|
| **Query (Q)** | "What am I looking for?" — derived from current position |
| **Key (K)** | "What do I contain?" — index for matching with queries |
| **Value (V)** | "What information do I return?" — content to aggregate |
| **W_Q, W_K, W_V** | Learned projection matrices mapping input to Q, K, V spaces |
| **Scaled dot-product** | `softmax(QK^T / √d_k) V` — the core attention formula |
| **√d_k scaling** | Prevents dot products from growing too large with dimension |
| **Self-attention** | Q, K, V all from the same sequence |
| **Cross-attention** | Q from decoder; K, V from encoder |
| **Causal mask** | Lower-triangular mask to prevent attending to future tokens |
| **Output shape** | Same sequence length as input, dimension `d_v` |

---

## 10. Revision Questions

1. **Explain the QKV abstraction using the dictionary/database analogy.** What role does each of Q, K, and V play?

2. **Given `d_model = 512` and `d_k = 64`**, what are the shapes of `W_Q`, `Q`, and the score matrix `QK^T` for a sequence of 10 tokens?

3. **Why do we use separate projection matrices** `W_Q`, `W_K`, `W_V` instead of using the input `X` directly as Q, K, and V?

4. **What is the difference between self-attention and cross-attention?** Give one real-world example of each.

5. **Manually compute** `Q × K^T` for `Q = [[1, 0], [0, 1]]` and `K = [[1, 1], [0, 1]]`. Then apply softmax row-wise and multiply by `V = [[2, 0], [0, 3]]`.

6. **How does a causal mask work** in autoregressive models like GPT? Why is it necessary, and what values does it set in the attention score matrix?

---

> **Previous: [← Attention Weights](03-attention-weights.md) | Next: [Attention Score Computation →](05-attention-score-computation.md)**
