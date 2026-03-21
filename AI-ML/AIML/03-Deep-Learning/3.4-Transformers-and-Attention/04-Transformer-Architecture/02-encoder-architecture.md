# The Transformer Encoder

[← Attention Is All You Need](01-attention-is-all-you-need.md) | [Decoder Architecture →](03-decoder-architecture.md)

---

## Overview

The Transformer **encoder** converts an input sequence of token embeddings into a sequence of rich,
contextualised representations. It consists of a stack of **N = 6 identical layers**, each containing
two sub-layers: (1) multi-head self-attention and (2) a position-wise feed-forward network. Every
sub-layer is wrapped with a **residual connection** followed by **layer normalisation**. The encoder
processes all positions in parallel — there is no recurrence — so a sequence of length `n` is
transformed in constant sequential depth per layer. The output of the final encoder layer is a
tensor of shape `(batch, seq_len, d_model)` that captures bidirectional context and is passed to
every decoder layer as the source of keys and values for cross-attention.

---

## 1. High-Level Encoder Stack

```
  Input Token IDs
        │
        ▼
┌───────────────────┐
│  Token Embedding   │   (vocab_size → d_model=512)
│  + Positional Enc  │   (sinusoidal, same shape)
└────────┬──────────┘
         │
         ▼
┌────────────────────────────────────────┐
│            Encoder Layer 1             │
├────────────────────────────────────────┤
│            Encoder Layer 2             │
├────────────────────────────────────────┤
│            Encoder Layer 3             │
├────────────────────────────────────────┤
│            Encoder Layer 4             │
├────────────────────────────────────────┤
│            Encoder Layer 5             │
├────────────────────────────────────────┤
│            Encoder Layer 6             │
└────────────────┬───────────────────────┘
                 │
                 ▼
     Encoder Output (batch, seq_len, 512)
     ─── passed to every decoder layer ───
```

---

## 2. Single Encoder Layer — Detailed Diagram

```
         Input x
      (batch, n, 512)
            │
            ▼
   ┌────────────────────┐
   │  Multi-Head Self-   │   Q, K, V all come from x
   │  Attention          │   8 heads, d_k = d_v = 64
   └────────┬───────────┘
            │ sublayer_out
            ▼
   ┌────────────────────┐
   │  Add & LayerNorm   │   x' = LayerNorm(x + sublayer_out)
   └────────┬───────────┘
            │ x'
            ▼
   ┌────────────────────┐
   │  Position-wise FFN  │   Linear(512→2048) → ReLU → Linear(2048→512)
   └────────┬───────────┘
            │ ffn_out
            ▼
   ┌────────────────────┐
   │  Add & LayerNorm   │   x'' = LayerNorm(x' + ffn_out)
   └────────┬───────────┘
            │
            ▼
       Output x''
      (batch, n, 512)
```

**Key observations:**
- Input and output have **exactly the same shape** `(batch, n, 512)` — every encoder layer is dimension-preserving.
- The residual connection means gradients flow directly from layer output back to input, enabling deep stacking.
- Each sub-layer applies dropout (p=0.1) to its output before the residual addition.

---

## 3. Sub-Layer 1: Multi-Head Self-Attention

In the encoder, Q, K, and V are **all derived from the same input** (the previous layer's output):

```
Q = x · W_Q          K = x · W_K          V = x · W_V
(n, 512)·(512,512)    (n, 512)·(512,512)    (n, 512)·(512,512)
= (n, 512)            = (n, 512)            = (n, 512)
```

These are split into `h = 8` heads of dimension `d_k = 64`:

```
Q_i = Q[:, i*64:(i+1)*64]    for i = 0..7   →  shape (n, 64) each
K_i = K[:, i*64:(i+1)*64]
V_i = V[:, i*64:(i+1)*64]
```

Each head computes:

$$\text{head}_i = \text{softmax}\!\left(\frac{Q_i K_i^T}{\sqrt{64}}\right) V_i$$

Heads are concatenated and projected:

$$\text{MultiHead} = [head_0; head_1; \dots; head_7] \cdot W^O$$

Where `W^O ∈ ℝ^(512×512)`.

### Parameter Count for Self-Attention (one layer)

| Matrix | Shape | Parameters |
|--------|-------|-----------|
| W_Q | 512 × 512 | 262,144 |
| W_K | 512 × 512 | 262,144 |
| W_V | 512 × 512 | 262,144 |
| W_O | 512 × 512 | 262,144 |
| Biases (×4) | 512 each | 2,048 |
| **Total** | | **1,050,624** |

---

## 4. Sub-Layer 2: Position-wise Feed-Forward Network (FFN)

$$\text{FFN}(x) = \text{ReLU}(x W_1 + b_1)\, W_2 + b_2$$

```
x ──► Linear(512 → 2048) ──► ReLU ──► Linear(2048 → 512) ──► output

Dimensions at each stage:
  x:        (batch, n, 512)
  After W₁: (batch, n, 2048)    ← expansion by 4×
  After W₂: (batch, n, 512)     ← back to d_model
```

### Parameter Count for FFN (one layer)

| Matrix | Shape | Parameters |
|--------|-------|-----------|
| W₁ | 512 × 2048 | 1,048,576 |
| b₁ | 2048 | 2,048 |
| W₂ | 2048 × 512 | 1,048,576 |
| b₂ | 512 | 512 |
| **Total** | | **2,099,712** |

---

## 5. Residual Connection + Layer Normalisation

Each sub-layer output is wrapped as:

$$\text{output} = \text{LayerNorm}(x + \text{Sublayer}(x))$$

### Layer Normalisation

For a vector `z ∈ ℝ^d_model`:

$$\text{LayerNorm}(z) = \gamma \odot \frac{z - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta$$

Where:
- `μ = (1/d) Σ z_i` — mean over the feature dimension
- `σ² = (1/d) Σ (z_i − μ)²` — variance over the feature dimension
- `γ, β ∈ ℝ^d_model` — learned scale and shift (2 × 512 = 1,024 params per LayerNorm)

---

## 6. Dimension Preservation — Complete Trace

```
Token IDs:         (batch, n)              integers
After embedding:   (batch, n, 512)         float
+ Pos encoding:    (batch, n, 512)         float (added, not concatenated)
Encoder layer in:  (batch, n, 512)
  Self-attn out:   (batch, n, 512)         ✓ same
  Add & Norm:      (batch, n, 512)         ✓ same
  FFN out:         (batch, n, 512)         ✓ same
  Add & Norm:      (batch, n, 512)         ✓ same
Encoder layer out: (batch, n, 512)
  ... ×6 layers ...
Final enc output:  (batch, n, 512)         ✓ same throughout
```

---

## 7. Worked Example: Single Encoder Layer Forward Pass

Let `d_model = 4` (tiny), `n = 3` (3 tokens), `d_ff = 8`, single head for simplicity.

### Input

```
x = [[0.5, 0.1, -0.3, 0.8],    # token 0
     [0.2, 0.7, 0.4, -0.1],    # token 1
     [-0.6, 0.3, 0.9, 0.0]]    # token 2
```

### Step 1: Self-Attention (simplified, 1 head)

```
Q = K = V = x    (identity projections for simplicity)

QKᵀ = [[0.5²+0.1²+0.09+0.64,  0.10+0.07-0.12-0.08,  -0.30+0.03-0.27+0.00],
        [0.10+0.07-0.12-0.08,  0.04+0.49+0.16+0.01,  -0.12+0.21+0.36+0.00],
        [-0.30+0.03-0.27+0.00, -0.12+0.21+0.36+0.00,  0.36+0.09+0.81+0.00]]

     = [[ 0.99, -0.03, -0.54],
        [-0.03,  0.70,  0.45],
        [-0.54,  0.45,  1.26]]
```

Scale by `√4 = 2`:

```
Scaled = [[ 0.495, -0.015, -0.270],
          [-0.015,  0.350,  0.225],
          [-0.270,  0.225,  0.630]]
```

Apply softmax row-wise → attention weights → multiply by V → self-attention output.

### Step 2: Add & Norm

```
x' = LayerNorm(x + attn_output)
```

### Step 3: FFN

```
ffn_input = x'
hidden = ReLU(x' @ W₁ + b₁)    # shape (3, 8)
ffn_out = hidden @ W₂ + b₂      # shape (3, 4)
```

### Step 4: Add & Norm

```
output = LayerNorm(x' + ffn_out)   # shape (3, 4) — same as input
```

---

## 8. Python / PyTorch Code: Full Encoder Implementation

```python
import torch
import torch.nn as nn
import math


class MultiHeadSelfAttention(nn.Module):
    """Multi-head self-attention for the encoder."""

    def __init__(self, d_model: int = 512, num_heads: int = 8, dropout: float = 0.1):
        super().__init__()
        assert d_model % num_heads == 0
        self.d_k = d_model // num_heads
        self.num_heads = num_heads

        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x: torch.Tensor, mask=None) -> torch.Tensor:
        batch, seq_len, _ = x.shape

        # Project and reshape: (batch, seq, d_model) → (batch, heads, seq, d_k)
        Q = self.W_q(x).view(batch, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        K = self.W_k(x).view(batch, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        V = self.W_v(x).view(batch, seq_len, self.num_heads, self.d_k).transpose(1, 2)

        # Scaled dot-product attention
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float("-inf"))
        attn_weights = self.dropout(torch.softmax(scores, dim=-1))
        context = torch.matmul(attn_weights, V)

        # Concatenate heads and project
        context = context.transpose(1, 2).contiguous().view(batch, seq_len, -1)
        return self.W_o(context)


class PositionWiseFFN(nn.Module):
    """Position-wise feed-forward network: two linear layers with ReLU."""

    def __init__(self, d_model: int = 512, d_ff: int = 2048, dropout: float = 0.1):
        super().__init__()
        self.linear1 = nn.Linear(d_model, d_ff)
        self.linear2 = nn.Linear(d_ff, d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.linear2(self.dropout(torch.relu(self.linear1(x))))


class TransformerEncoderLayer(nn.Module):
    """Single encoder layer: self-attention + FFN, each with Add & Norm."""

    def __init__(self, d_model: int = 512, num_heads: int = 8,
                 d_ff: int = 2048, dropout: float = 0.1):
        super().__init__()
        self.self_attn = MultiHeadSelfAttention(d_model, num_heads, dropout)
        self.ffn = PositionWiseFFN(d_model, d_ff, dropout)
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.dropout1 = nn.Dropout(dropout)
        self.dropout2 = nn.Dropout(dropout)

    def forward(self, x: torch.Tensor, mask=None) -> torch.Tensor:
        # Sub-layer 1: Multi-Head Self-Attention + Add & Norm
        attn_out = self.self_attn(x, mask)
        x = self.norm1(x + self.dropout1(attn_out))

        # Sub-layer 2: Feed-Forward Network + Add & Norm
        ffn_out = self.ffn(x)
        x = self.norm2(x + self.dropout2(ffn_out))
        return x


class TransformerEncoder(nn.Module):
    """Stack of N encoder layers with input embedding + positional encoding."""

    def __init__(self, vocab_size: int, d_model: int = 512, num_heads: int = 8,
                 d_ff: int = 2048, num_layers: int = 6, dropout: float = 0.1,
                 max_len: int = 5000):
        super().__init__()
        self.d_model = d_model
        self.embedding = nn.Embedding(vocab_size, d_model)
        self.pos_encoding = self._sinusoidal_encoding(max_len, d_model)
        self.layers = nn.ModuleList([
            TransformerEncoderLayer(d_model, num_heads, d_ff, dropout)
            for _ in range(num_layers)
        ])
        self.dropout = nn.Dropout(dropout)
        self.norm = nn.LayerNorm(d_model)  # final layer norm

    @staticmethod
    def _sinusoidal_encoding(max_len: int, d_model: int) -> torch.Tensor:
        pe = torch.zeros(max_len, d_model)
        pos = torch.arange(0, max_len).unsqueeze(1).float()
        div = torch.exp(torch.arange(0, d_model, 2).float() * -(math.log(10000.0) / d_model))
        pe[:, 0::2] = torch.sin(pos * div)
        pe[:, 1::2] = torch.cos(pos * div)
        return pe.unsqueeze(0)  # (1, max_len, d_model)

    def forward(self, src: torch.Tensor, mask=None) -> torch.Tensor:
        seq_len = src.size(1)
        x = self.embedding(src) * math.sqrt(self.d_model)
        x = x + self.pos_encoding[:, :seq_len, :].to(x.device)
        x = self.dropout(x)

        for layer in self.layers:
            x = layer(x, mask)

        return self.norm(x)  # (batch, seq_len, d_model)


# ----- Quick smoke test -----
if __name__ == "__main__":
    encoder = TransformerEncoder(vocab_size=32000)
    src_tokens = torch.randint(0, 32000, (4, 30))  # batch=4, seq_len=30
    enc_output = encoder(src_tokens)
    print(f"Encoder output shape: {enc_output.shape}")  # → (4, 30, 512)

    params = sum(p.numel() for p in encoder.parameters())
    print(f"Encoder parameters: {params:,}")
```

---

## 9. Real-World Applications of the Encoder

| Application | Model | How Encoder Is Used |
|-------------|-------|---------------------|
| Text classification | BERT | Encoder-only; CLS token output → classifier |
| Named entity recognition | BERT / RoBERTa | Per-token encoder outputs → token-level labels |
| Sentence embeddings | Sentence-BERT | Encoder output pooled into a fixed-size vector |
| Semantic search | DPR, ColBERT | Encoder maps queries and documents to vectors |
| Image classification | ViT | Image patches treated as tokens → encoder stack |
| Protein structure | ESM-2 | Amino acid sequences → encoder representations |

**Key insight:** When the task requires **understanding** an input (classification, retrieval, encoding), an encoder-only architecture is typically used. When the task requires **generation**, the encoder is paired with a decoder.

---

## 10. Summary Table

| Aspect | Detail |
|--------|--------|
| **Number of layers** | N = 6 (original paper) |
| **Sub-layers per layer** | 2: self-attention + FFN |
| **Residual connections** | Around every sub-layer |
| **Normalisation** | Layer Norm after each residual add |
| **Self-attention** | 8 heads, d_k = d_v = 64 |
| **FFN** | 512 → 2048 → 512 (ReLU) |
| **Dropout** | 0.1 on sub-layer outputs + embeddings |
| **Input** | Token embeddings + sinusoidal positional encoding |
| **Output shape** | (batch, seq_len, 512) — same d_model throughout |
| **Params per layer** | ~3.15M (attention ~1.05M + FFN ~2.10M) |
| **Total encoder params** | ~18.9M (6 layers) + embeddings |

---

## 11. Revision Questions

1. **Why does every encoder sub-layer preserve the dimension d_model = 512?**
   Hint: Residual connections require the sub-layer output to have the same shape as the input so they can be added element-wise.

2. **What is the role of layer normalisation in the encoder, and where is it applied?**
   Hint: It normalises across the feature dimension (not batch) and is applied after each residual addition.

3. **In encoder self-attention, where do Q, K, and V come from?**
   Hint: All three are linear projections of the same input — the previous layer's output (or the embedded input for layer 1).

4. **Why does the FFN expand the dimension from 512 to 2048 and then compress back?**
   Hint: The wider inner layer provides more capacity for learning complex transformations, while the compression keeps the interface dimension constant.

5. **How many parameters does a single encoder layer have (approximately)?**
   Hint: Self-attention ≈ 4 × 512² ≈ 1.05M; FFN ≈ 2 × 512 × 2048 ≈ 2.10M; total ≈ 3.15M.

6. **What would happen if we removed the residual connections from the encoder?**
   Hint: Deep networks would suffer from vanishing gradients and training instability — residuals provide a gradient highway.

---

[← Attention Is All You Need](01-attention-is-all-you-need.md) | [Decoder Architecture →](03-decoder-architecture.md)
