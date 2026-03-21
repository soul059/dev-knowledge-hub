# The Transformer Decoder

[← Encoder Architecture](02-encoder-architecture.md) | [Encoder-Decoder Attention →](04-encoder-decoder-attention.md)

---

## Overview

The Transformer **decoder** generates the output sequence one token at a time in an autoregressive
fashion. It consists of a stack of **N = 6 identical layers**, each containing three sub-layers:
(1) **masked** multi-head self-attention, (2) multi-head **cross-attention** over the encoder output,
and (3) a position-wise feed-forward network. Each sub-layer is wrapped with a residual connection
and layer normalisation. The critical difference from the encoder is the **causal mask** in the
self-attention sub-layer: it prevents each position from attending to subsequent positions, ensuring
that predictions for position `t` depend only on the known outputs at positions `< t`. This
autoregressive property is what makes the decoder suitable for generation tasks such as translation,
summarisation, and language modelling.

---

## 1. High-Level Decoder Stack

```
  Target Token IDs (shifted right)
         │
         ▼
┌────────────────────┐
│  Token Embedding    │   (tgt_vocab_size → d_model=512)
│  + Positional Enc   │   (sinusoidal, same shape)
└────────┬───────────┘
         │
         ▼
┌──────────────────────────────────────────────┐
│              Decoder Layer 1                  │
│  ┌────────────────────────────────────────┐   │
│  │ 1. Masked Multi-Head Self-Attention    │   │
│  │ 2. Add & LayerNorm                     │   │◄── Encoder
│  │ 3. Multi-Head Cross-Attention          │   │    Output
│  │ 4. Add & LayerNorm                     │   │    (K, V)
│  │ 5. Position-wise FFN                   │   │
│  │ 6. Add & LayerNorm                     │   │
│  └────────────────────────────────────────┘   │
├──────────────────────────────────────────────┤
│              Decoder Layer 2                  │
├──────────────────────────────────────────────┤
│              Decoder Layer 3                  │
├──────────────────────────────────────────────┤
│              Decoder Layer 4                  │
├──────────────────────────────────────────────┤
│              Decoder Layer 5                  │
├──────────────────────────────────────────────┤
│              Decoder Layer 6                  │
└──────────────────┬───────────────────────────┘
                   │
                   ▼
          ┌────────────────┐
          │  Linear Layer   │   (512 → tgt_vocab_size)
          └────────┬───────┘
                   ▼
          ┌────────────────┐
          │    Softmax      │   → output probabilities
          └────────────────┘
```

---

## 2. Single Decoder Layer — Detailed Diagram

```
          Input y
       (batch, m, 512)
             │
             ▼
    ┌─────────────────────┐
    │  Masked Multi-Head   │   Q, K, V from y; future positions masked
    │  Self-Attention      │   (causal / look-ahead mask)
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │  Add & LayerNorm     │   y' = LN(y + masked_attn_out)
    └─────────┬───────────┘
              │ y'
              ▼
    ┌─────────────────────┐
    │  Multi-Head Cross-   │   Q from y', K and V from encoder output
    │  Attention           │   (encoder-decoder attention)
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │  Add & LayerNorm     │   y'' = LN(y' + cross_attn_out)
    └─────────┬───────────┘
              │ y''
              ▼
    ┌─────────────────────┐
    │  Position-wise FFN   │   Linear(512→2048) → ReLU → Linear(2048→512)
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │  Add & LayerNorm     │   y''' = LN(y'' + ffn_out)
    └─────────┬───────────┘
              │
              ▼
         Output y'''
       (batch, m, 512)
```

---

## 3. Why Causal Masking Is Needed

### The Autoregressive Property

During generation, the decoder produces tokens one at a time:

```
Step 0:  <BOS>                          → predict token₁
Step 1:  <BOS> token₁                   → predict token₂
Step 2:  <BOS> token₁ token₂            → predict token₃
  ...
Step t:  <BOS> token₁ ... tokenₜ        → predict tokenₜ₊₁
```

If position `t` could attend to positions `t+1, t+2, ...`, the model would be **cheating** — it
would see the answer it is supposed to predict. The causal mask prevents this.

### The Mask Matrix

For a sequence of length `m = 5`, the mask is an upper-triangular matrix of `-∞`:

```
Mask =  ┌                          ┐
        │  0    -∞   -∞   -∞   -∞  │   position 0 sees only itself
        │  0     0   -∞   -∞   -∞  │   position 1 sees 0, 1
        │  0     0    0   -∞   -∞  │   position 2 sees 0, 1, 2
        │  0     0    0    0   -∞  │   position 3 sees 0, 1, 2, 3
        │  0     0    0    0    0  │   position 4 sees all
        └                          ┘
```

This is **added** to the attention scores before softmax:

$$\text{MaskedAttn} = \text{softmax}\!\left(\frac{QK^T}{\sqrt{d_k}} + \text{Mask}\right) V$$

Since `softmax(-∞) → 0`, the masked positions contribute zero weight.

### Generating the Mask in Code

```python
import torch

def causal_mask(seq_len: int) -> torch.Tensor:
    """Upper-triangular mask filled with -inf."""
    return torch.triu(torch.full((seq_len, seq_len), float("-inf")), diagonal=1)

print(causal_mask(5))
# tensor([[  0., -inf, -inf, -inf, -inf],
#         [  0.,   0., -inf, -inf, -inf],
#         [  0.,   0.,   0., -inf, -inf],
#         [  0.,   0.,   0.,   0., -inf],
#         [  0.,   0.,   0.,   0.,   0.]])
```

---

## 4. Worked Example: Masked Self-Attention

Suppose `d_k = 4`, sequence length `m = 3`.

```
Q = K = V = [[1.0, 0.5, 0.2, 0.8],    # position 0
             [0.3, 0.9, 0.1, 0.4],    # position 1
             [0.7, 0.2, 0.6, 0.3]]    # position 2
```

**Step 1 — Raw scores (QKᵀ):**

```
QKᵀ[0,0] = 1.0·1.0 + 0.5·0.5 + 0.2·0.2 + 0.8·0.8 = 1.93
QKᵀ[0,1] = 1.0·0.3 + 0.5·0.9 + 0.2·0.1 + 0.8·0.4 = 1.09
QKᵀ[0,2] = 1.0·0.7 + 0.5·0.2 + 0.2·0.6 + 0.8·0.3 = 1.16

QKᵀ[1,0] = 0.3·1.0 + 0.9·0.5 + 0.1·0.2 + 0.4·0.8 = 1.09
QKᵀ[1,1] = 0.3·0.3 + 0.9·0.9 + 0.1·0.1 + 0.4·0.4 = 1.07
QKᵀ[1,2] = 0.3·0.7 + 0.9·0.2 + 0.1·0.6 + 0.4·0.3 = 0.57

QKᵀ[2,0] = 0.7·1.0 + 0.2·0.5 + 0.6·0.2 + 0.3·0.8 = 1.16
QKᵀ[2,1] = 0.7·0.3 + 0.2·0.9 + 0.6·0.1 + 0.3·0.4 = 0.57
QKᵀ[2,2] = 0.7·0.7 + 0.2·0.2 + 0.6·0.6 + 0.3·0.3 = 0.98

QKᵀ = [[1.93, 1.09, 1.16],
        [1.09, 1.07, 0.57],
        [1.16, 0.57, 0.98]]
```

**Step 2 — Scale by √4 = 2:**

```
Scaled = [[0.965, 0.545, 0.580],
          [0.545, 0.535, 0.285],
          [0.580, 0.285, 0.490]]
```

**Step 3 — Apply causal mask (add -∞ to upper triangle):**

```
Masked = [[0.965,  -∞,    -∞  ],
          [0.545, 0.535,  -∞  ],
          [0.580, 0.285, 0.490]]
```

**Step 4 — Softmax (row-wise):**

```
Row 0: softmax([0.965]) = [1.000]              → [1.000, 0.000, 0.000]
Row 1: softmax([0.545, 0.535]) → exp: [1.725, 1.707] → sum=3.432
                                      → [0.503, 0.497, 0.000]
Row 2: softmax([0.580, 0.285, 0.490]) → exp: [1.786, 1.330, 1.632] → sum=4.748
                                             → [0.376, 0.280, 0.344]
```

**Step 5 — Multiply by V:**

```
Out[0] = 1.000·[1.0,0.5,0.2,0.8] = [1.000, 0.500, 0.200, 0.800]
Out[1] = 0.503·[1.0,0.5,0.2,0.8] + 0.497·[0.3,0.9,0.1,0.4]
       = [0.652, 0.699, 0.150, 0.601]
Out[2] = 0.376·[1.0,0.5,0.2,0.8] + 0.280·[0.3,0.9,0.1,0.4] + 0.344·[0.7,0.2,0.6,0.3]
       = [0.701, 0.509, 0.310, 0.516]
```

Notice: position 0 only sees itself, position 1 sees 0 and 1, position 2 sees all three — exactly the autoregressive pattern.

---

## 5. Mathematics of the Decoder Layer

### Sub-layer 1: Masked Self-Attention

$$y' = \text{LayerNorm}\!\left(y + \text{MaskedMultiHead}(y, y, y)\right)$$

### Sub-layer 2: Cross-Attention

$$y'' = \text{LayerNorm}\!\left(y' + \text{MultiHead}(y', \text{enc\_out}, \text{enc\_out})\right)$$

- **Q** comes from the decoder (y')
- **K** and **V** come from the encoder output

### Sub-layer 3: Feed-Forward Network

$$y''' = \text{LayerNorm}\!\left(y'' + \text{FFN}(y'')\right)$$

### Parameter Count per Decoder Layer

| Component | Parameters |
|-----------|-----------|
| Masked self-attention (4 × 512² + biases) | ~1,050,624 |
| Cross-attention (4 × 512² + biases) | ~1,050,624 |
| FFN (512×2048 + 2048×512 + biases) | ~2,099,712 |
| 3 × LayerNorm (2 × 512 each) | 3,072 |
| **Total per decoder layer** | **~4,204,032** |
| **Total decoder (6 layers)** | **~25,224,192** |

---

## 6. Python / PyTorch Code: Full Decoder Implementation

```python
import torch
import torch.nn as nn
import math


class MultiHeadAttention(nn.Module):
    """General multi-head attention (works for self-attn and cross-attn)."""

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

    def forward(self, query, key, value, mask=None):
        batch = query.size(0)

        Q = self.W_q(query).view(batch, -1, self.num_heads, self.d_k).transpose(1, 2)
        K = self.W_k(key).view(batch, -1, self.num_heads, self.d_k).transpose(1, 2)
        V = self.W_v(value).view(batch, -1, self.num_heads, self.d_k).transpose(1, 2)

        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float("-inf"))
        attn = self.dropout(torch.softmax(scores, dim=-1))
        context = torch.matmul(attn, V)

        context = context.transpose(1, 2).contiguous().view(batch, -1, self.num_heads * self.d_k)
        return self.W_o(context)


class PositionWiseFFN(nn.Module):
    def __init__(self, d_model: int = 512, d_ff: int = 2048, dropout: float = 0.1):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(d_ff, d_model),
        )

    def forward(self, x):
        return self.net(x)


class TransformerDecoderLayer(nn.Module):
    """Single decoder layer with masked self-attn, cross-attn, and FFN."""

    def __init__(self, d_model: int = 512, num_heads: int = 8,
                 d_ff: int = 2048, dropout: float = 0.1):
        super().__init__()
        self.masked_self_attn = MultiHeadAttention(d_model, num_heads, dropout)
        self.cross_attn = MultiHeadAttention(d_model, num_heads, dropout)
        self.ffn = PositionWiseFFN(d_model, d_ff, dropout)

        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.norm3 = nn.LayerNorm(d_model)

        self.dropout1 = nn.Dropout(dropout)
        self.dropout2 = nn.Dropout(dropout)
        self.dropout3 = nn.Dropout(dropout)

    def forward(self, tgt, encoder_output, tgt_mask=None, memory_mask=None):
        # Sub-layer 1: Masked multi-head self-attention
        attn_out = self.masked_self_attn(tgt, tgt, tgt, mask=tgt_mask)
        tgt = self.norm1(tgt + self.dropout1(attn_out))

        # Sub-layer 2: Multi-head cross-attention (Q=decoder, K=V=encoder)
        cross_out = self.cross_attn(tgt, encoder_output, encoder_output, mask=memory_mask)
        tgt = self.norm2(tgt + self.dropout2(cross_out))

        # Sub-layer 3: Position-wise FFN
        ffn_out = self.ffn(tgt)
        tgt = self.norm3(tgt + self.dropout3(ffn_out))

        return tgt


class TransformerDecoder(nn.Module):
    """Stack of N decoder layers with embedding + positional encoding."""

    def __init__(self, vocab_size: int, d_model: int = 512, num_heads: int = 8,
                 d_ff: int = 2048, num_layers: int = 6, dropout: float = 0.1,
                 max_len: int = 5000):
        super().__init__()
        self.d_model = d_model
        self.embedding = nn.Embedding(vocab_size, d_model)
        self.pos_encoding = self._sinusoidal_encoding(max_len, d_model)
        self.layers = nn.ModuleList([
            TransformerDecoderLayer(d_model, num_heads, d_ff, dropout)
            for _ in range(num_layers)
        ])
        self.dropout = nn.Dropout(dropout)
        self.norm = nn.LayerNorm(d_model)

    @staticmethod
    def _sinusoidal_encoding(max_len, d_model):
        pe = torch.zeros(max_len, d_model)
        pos = torch.arange(0, max_len).unsqueeze(1).float()
        div = torch.exp(torch.arange(0, d_model, 2).float() * -(math.log(10000.0) / d_model))
        pe[:, 0::2] = torch.sin(pos * div)
        pe[:, 1::2] = torch.cos(pos * div)
        return pe.unsqueeze(0)

    @staticmethod
    def _causal_mask(seq_len: int) -> torch.Tensor:
        return torch.triu(torch.ones(seq_len, seq_len), diagonal=1).bool()

    def forward(self, tgt, encoder_output, memory_mask=None):
        seq_len = tgt.size(1)
        tgt_mask = self._causal_mask(seq_len).to(tgt.device)
        # Convert bool mask: True = masked → use masked_fill convention
        tgt_mask = tgt_mask.unsqueeze(0).unsqueeze(0)  # (1, 1, m, m)
        tgt_mask = (~tgt_mask).float()  # 1 = attend, 0 = mask

        x = self.embedding(tgt) * math.sqrt(self.d_model)
        x = x + self.pos_encoding[:, :seq_len, :].to(x.device)
        x = self.dropout(x)

        for layer in self.layers:
            x = layer(x, encoder_output, tgt_mask=tgt_mask, memory_mask=memory_mask)

        return self.norm(x)


# ----- Quick smoke test -----
if __name__ == "__main__":
    decoder = TransformerDecoder(vocab_size=32000)
    tgt_tokens = torch.randint(0, 32000, (2, 15))            # batch=2, tgt_len=15
    fake_enc_output = torch.randn(2, 20, 512)                 # batch=2, src_len=20, d=512
    dec_output = decoder(tgt_tokens, fake_enc_output)
    print(f"Decoder output shape: {dec_output.shape}")         # → (2, 15, 512)

    params = sum(p.numel() for p in decoder.parameters())
    print(f"Decoder parameters: {params:,}")
```

---

## 7. Training vs. Inference

### Training (Teacher Forcing)

During training, the decoder receives the **entire target sequence** (shifted right) at once.
The causal mask ensures each position only attends to previous positions, so all positions can
be computed **in parallel**:

```
Input:  <BOS>  The   cat   sat         (shifted right ground truth)
Target:  The   cat   sat   <EOS>       (what we want to predict)
                ↑     ↑     ↑
         positions computed in parallel with causal mask
```

### Inference (Autoregressive)

At inference, tokens are generated **one at a time**:

```
Step 0: Input: <BOS>              → Model predicts: "The"
Step 1: Input: <BOS> The          → Model predicts: "cat"
Step 2: Input: <BOS> The cat      → Model predicts: "sat"
Step 3: Input: <BOS> The cat sat  → Model predicts: <EOS>
```

This is inherently sequential and is a major bottleneck for Transformer generation speed.

---

## 8. Real-World Applications

| Application | Architecture | Masking |
|-------------|-------------|---------|
| Machine translation | Full encoder-decoder | Causal mask in decoder |
| Language modelling (GPT) | Decoder-only | Causal mask throughout |
| Text summarisation | Encoder-decoder (T5, BART) | Causal mask in decoder |
| Code generation (Copilot) | Decoder-only (Codex) | Causal mask throughout |
| Music generation | Decoder-only or enc-dec | Causal mask |
| Image generation (DALL·E v1) | Decoder (autoregressive) | Causal mask on image tokens |

---

## 9. Summary Table

| Aspect | Detail |
|--------|--------|
| **Number of layers** | N = 6 (original paper) |
| **Sub-layers per layer** | 3: masked self-attn + cross-attn + FFN |
| **Causal mask** | Upper-triangular -∞ matrix prevents attending to future |
| **Cross-attention** | Q from decoder, K/V from encoder output |
| **FFN** | Same as encoder: 512 → 2048 → 512 |
| **Params per layer** | ~4.2M (self-attn ~1.05M + cross-attn ~1.05M + FFN ~2.10M) |
| **Total decoder params** | ~25.2M (6 layers) + embeddings |
| **Training** | Teacher forcing (parallel with mask) |
| **Inference** | Autoregressive (sequential, one token at a time) |
| **Output** | (batch, tgt_len, 512) → Linear → Softmax → probabilities |

---

## 10. Revision Questions

1. **Why does the decoder need a causal mask but the encoder does not?**
   Hint: The encoder reads the full input bidirectionally. The decoder must generate autoregressively — seeing future tokens would leak information.

2. **What are the three sub-layers in each decoder layer and how do they differ?**
   Hint: (1) Masked self-attention on decoder input, (2) cross-attention with Q from decoder and K/V from encoder, (3) position-wise FFN.

3. **How does teacher forcing make training more efficient?**
   Hint: All target positions can be computed in parallel because the ground-truth sequence is provided; the causal mask prevents cheating.

4. **If the target sequence has m = 10 tokens, what is the shape of the causal mask?**
   Hint: (10, 10) with -∞ above the main diagonal and 0 on/below it.

5. **Why does the decoder have more parameters per layer than the encoder?**
   Hint: The decoder has an extra multi-head attention sub-layer (cross-attention) compared to the encoder.

6. **Explain why autoregressive inference is slower than training for the decoder.**
   Hint: During training, all positions are computed in a single forward pass. During inference, each new token requires a separate forward pass.

---

[← Encoder Architecture](02-encoder-architecture.md) | [Encoder-Decoder Attention →](04-encoder-decoder-attention.md)
