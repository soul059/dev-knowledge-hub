# Attention Is All You Need: The Transformer Paper

[← Parameter Efficiency](../03-Multi-Head-Attention/04-parameter-efficiency.md) | [Encoder Architecture →](02-encoder-architecture.md)

---

## Overview

The 2017 paper *"Attention Is All You Need"* by Vaswani et al. (Google Brain / Google Research) introduced
the **Transformer** — a sequence-to-sequence architecture that dispenses with recurrence and convolutions
entirely and relies **solely on attention mechanisms**. Before this paper, dominant models for machine
translation and language modelling were RNNs (LSTMs, GRUs) that processed tokens one-by-one in sequence.
The Transformer's key insight is that self-attention can capture dependencies across an entire sequence
in a **single parallel step**, enabling far greater training efficiency on modern GPUs/TPUs. This single
paper catalysed an explosion of progress: BERT, GPT, T5, PaLM, LLaMA, and virtually every modern large
language model traces its lineage directly to the Transformer architecture described here.

---

## 1. Key Contributions

| # | Contribution | Why It Matters |
|---|---|---|
| 1 | Removed recurrence entirely | Enables full parallelism during training |
| 2 | Scaled dot-product attention as the core primitive | Simple, efficient, hardware-friendly |
| 3 | Multi-head attention | Allows the model to attend to information from different representation sub-spaces |
| 4 | Positional encoding | Injects order information without recurrence |
| 5 | Encoder-decoder with residual connections + layer norm | Deep, stable, trainable architecture |

---

## 2. The Paradigm Shift: Sequential → Parallel

### RNN / LSTM (Before)

```
x₁ → [h₁] → x₂ → [h₂] → x₃ → [h₃] → ... → xₙ → [hₙ]
        ↑           ↑           ↑                     ↑
    (must wait  (must wait  (must wait            (sequential
     for h₀)    for h₁)    for h₂)               bottleneck)
```

- **Time complexity per layer**: O(n) sequential steps
- Long-range dependencies decay through many multiplicative steps

### Transformer (After)

```
x₁  x₂  x₃  ...  xₙ          (all tokens processed simultaneously)
 ↓   ↓   ↓        ↓
[======= Self-Attention =======]   ← O(1) sequential steps, O(n²) parallel
 ↓   ↓   ↓        ↓
y₁  y₂  y₃  ...  yₙ
```

- **Time complexity per layer**: O(1) sequential steps (all positions attend in parallel)
- Any token can attend to any other token directly — no vanishing gradient over distance

---

## 3. Original Architecture Summary

```
                        ┌──────────────────────────────┐
                        │        OUTPUT PROBABILITIES   │
                        │         (Softmax Layer)       │
                        └──────────────┬───────────────┘
                                       │
                        ┌──────────────▼───────────────┐
                        │          Linear Layer         │
                        └──────────────┬───────────────┘
                                       │
              ┌────────────────────────▼────────────────────────┐
              │                   DECODER (×6)                  │
              │  ┌──────────────────────────────────────────┐   │
              │  │  Masked Multi-Head Self-Attention        │   │
              │  │  Add & Norm                              │   │
              │  │  Multi-Head Cross-Attention (Enc→Dec)    │   │
              │  │  Add & Norm                              │   │
              │  │  Feed-Forward Network                    │   │
              │  │  Add & Norm                              │   │
              │  └──────────────────────────────────────────┘   │
              └────────────────────────┬────────────────────────┘
                   ▲                   │
                   │ (encoder output   │
                   │  as K, V)         │
              ┌────┴───────────────────┴────────────────────────┐
              │                   ENCODER (×6)                  │
              │  ┌──────────────────────────────────────────┐   │
              │  │  Multi-Head Self-Attention               │   │
              │  │  Add & Norm                              │   │
              │  │  Feed-Forward Network                    │   │
              │  │  Add & Norm                              │   │
              │  └──────────────────────────────────────────┘   │
              └────────────────────────┬────────────────────────┘
                                       │
              ┌────────────────────────▼────────────────────────┐
              │     Input Embeddings + Positional Encoding      │
              └─────────────────────────────────────────────────┘
```

---

## 4. Key Hyperparameters from the Paper

| Symbol | Name | Value | Description |
|--------|------|-------|-------------|
| d_model | Model dimension | 512 | Embedding and hidden size throughout the model |
| h | Number of heads | 8 | Parallel attention heads |
| d_k | Key dimension | 64 | d_model / h = 512 / 8 |
| d_v | Value dimension | 64 | d_model / h = 512 / 8 |
| d_ff | Feed-forward dimension | 2048 | Inner dimension of the FFN (4 × d_model) |
| N | Number of layers | 6 | Stacked encoder and decoder layers |
| P_drop | Dropout rate | 0.1 | Applied to sub-layer outputs and embeddings |
| Vocab | Vocabulary size | ~37,000 | BPE tokens (shared source-target vocabulary) |

---

## 5. Core Mathematics

### Scaled Dot-Product Attention

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

- **Q** (queries): shape `(seq_len_q, d_k)`
- **K** (keys): shape `(seq_len_k, d_k)`
- **V** (values): shape `(seq_len_k, d_v)`
- The scaling factor `√d_k` prevents the dot products from growing too large as `d_k` increases.

### Multi-Head Attention

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h) W^O$$

where each head:

$$\text{head}_i = \text{Attention}(Q W_i^Q, K W_i^K, V W_i^V)$$

### Position-wise Feed-Forward Network

$$\text{FFN}(x) = \max(0,\; x W_1 + b_1)\, W_2 + b_2$$

- W₁ ∈ ℝ^(512×2048), W₂ ∈ ℝ^(2048×512) — the inner dimension expands 4×.

### Positional Encoding (Sinusoidal)

$$PE_{(pos, 2i)} = \sin\!\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)$$
$$PE_{(pos, 2i+1)} = \cos\!\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)$$

---

## 6. Worked Example: Attention Score Computation

Suppose `d_k = 4` (small for illustration) and we have 3 tokens.

```
Q = [[1, 0, 1, 0],      K = [[1, 1, 0, 0],      V = [[1, 0, 1, 0],
     [0, 1, 0, 1],           [0, 0, 1, 1],           [0, 1, 0, 1],
     [1, 1, 0, 0]]           [1, 0, 1, 0]]           [1, 1, 0, 0]]
```

**Step 1 — Compute QKᵀ:**

```
QKᵀ = [[1·1+0·1+1·0+0·0, 1·0+0·0+1·1+0·1, 1·1+0·0+1·1+0·0],   [[1, 1, 2],
        [0·1+1·1+0·0+1·0, 0·0+1·0+0·1+1·1, 0·1+1·0+0·1+1·0],  = [1, 1, 0],
        [1·1+1·1+0·0+0·0, 1·0+1·0+0·1+0·1, 1·1+1·0+0·1+0·0]]    [2, 0, 1]]
```

**Step 2 — Scale by √d_k = √4 = 2:**

```
Scaled = [[0.50, 0.50, 1.00],
          [0.50, 0.50, 0.00],
          [1.00, 0.00, 0.50]]
```

**Step 3 — Softmax (row-wise):**

```
Row 0: exp([0.50, 0.50, 1.00]) = [1.649, 1.649, 2.718] → sum=6.016
        → [0.274, 0.274, 0.452]

Row 1: exp([0.50, 0.50, 0.00]) = [1.649, 1.649, 1.000] → sum=4.298
        → [0.384, 0.384, 0.233]

Row 2: exp([1.00, 0.00, 0.50]) = [2.718, 1.000, 1.649] → sum=5.367
        → [0.506, 0.186, 0.307]
```

**Step 4 — Multiply by V:**

```
Output[0] = 0.274·[1,0,1,0] + 0.274·[0,1,0,1] + 0.452·[1,1,0,0]
          = [0.726, 0.726, 0.274, 0.274]

Output[1] = 0.384·[1,0,1,0] + 0.384·[0,1,0,1] + 0.233·[1,1,0,0]
          = [0.617, 0.617, 0.384, 0.384]

Output[2] = 0.506·[1,0,1,0] + 0.186·[0,1,0,1] + 0.307·[1,1,0,0]
          = [0.813, 0.493, 0.506, 0.186]
```

---

## 7. Python / PyTorch Code: Basic Transformer Skeleton

```python
import torch
import torch.nn as nn
import math


class PositionalEncoding(nn.Module):
    """Sinusoidal positional encoding from the original paper."""

    def __init__(self, d_model: int, max_len: int = 5000, dropout: float = 0.1):
        super().__init__()
        self.dropout = nn.Dropout(p=dropout)

        pe = torch.zeros(max_len, d_model)                       # (max_len, d_model)
        position = torch.arange(0, max_len).unsqueeze(1).float() # (max_len, 1)
        div_term = torch.exp(
            torch.arange(0, d_model, 2).float() * -(math.log(10000.0) / d_model)
        )
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        pe = pe.unsqueeze(0)  # (1, max_len, d_model)
        self.register_buffer("pe", pe)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # x: (batch, seq_len, d_model)
        x = x + self.pe[:, :x.size(1)]
        return self.dropout(x)


class TransformerModel(nn.Module):
    """Minimal Transformer (encoder-decoder) matching original paper layout."""

    def __init__(
        self,
        src_vocab_size: int,
        tgt_vocab_size: int,
        d_model: int = 512,
        nhead: int = 8,
        num_encoder_layers: int = 6,
        num_decoder_layers: int = 6,
        d_ff: int = 2048,
        dropout: float = 0.1,
    ):
        super().__init__()
        self.d_model = d_model

        # Embedding layers
        self.src_embed = nn.Embedding(src_vocab_size, d_model)
        self.tgt_embed = nn.Embedding(tgt_vocab_size, d_model)
        self.pos_enc = PositionalEncoding(d_model, dropout=dropout)

        # Encoder & Decoder stacks (PyTorch built-in mirrors the paper)
        encoder_layer = nn.TransformerEncoderLayer(
            d_model=d_model, nhead=nhead, dim_feedforward=d_ff,
            dropout=dropout, batch_first=True
        )
        self.encoder = nn.TransformerEncoder(encoder_layer, num_layers=num_encoder_layers)

        decoder_layer = nn.TransformerDecoderLayer(
            d_model=d_model, nhead=nhead, dim_feedforward=d_ff,
            dropout=dropout, batch_first=True
        )
        self.decoder = nn.TransformerDecoder(decoder_layer, num_layers=num_decoder_layers)

        # Final linear projection to vocabulary
        self.output_proj = nn.Linear(d_model, tgt_vocab_size)

    def _generate_square_subsequent_mask(self, sz: int) -> torch.Tensor:
        """Causal mask for autoregressive decoding."""
        return torch.triu(torch.full((sz, sz), float("-inf")), diagonal=1)

    def forward(
        self,
        src: torch.Tensor,      # (batch, src_len)
        tgt: torch.Tensor,      # (batch, tgt_len)
        src_mask=None,
        tgt_mask=None,
    ) -> torch.Tensor:
        # Embed + scale + positional encoding
        src_emb = self.pos_enc(self.src_embed(src) * math.sqrt(self.d_model))
        tgt_emb = self.pos_enc(self.tgt_embed(tgt) * math.sqrt(self.d_model))

        # Create causal mask if not provided
        if tgt_mask is None:
            tgt_mask = self._generate_square_subsequent_mask(tgt.size(1)).to(tgt.device)

        # Encoder → Decoder → Linear
        memory = self.encoder(src_emb, mask=src_mask)
        dec_out = self.decoder(tgt_emb, memory, tgt_mask=tgt_mask)
        return self.output_proj(dec_out)   # (batch, tgt_len, tgt_vocab_size)


# ----- Quick smoke test -----
if __name__ == "__main__":
    model = TransformerModel(src_vocab_size=10000, tgt_vocab_size=10000)
    src = torch.randint(0, 10000, (2, 20))   # batch=2, src_len=20
    tgt = torch.randint(0, 10000, (2, 15))   # batch=2, tgt_len=15
    logits = model(src, tgt)
    print(f"Output shape: {logits.shape}")    # → torch.Size([2, 15, 10000])
    total_params = sum(p.numel() for p in model.parameters())
    print(f"Total parameters: {total_params:,}")
```

---

## 8. Impact on NLP and AI

### Timeline of Developments After the Paper

| Year | Model / Paper | Built On |
|------|---------------|----------|
| 2017 | Transformer (Vaswani et al.) | — (this paper) |
| 2018 | GPT-1 (OpenAI) | Decoder-only Transformer |
| 2018 | BERT (Google) | Encoder-only Transformer |
| 2019 | GPT-2 (OpenAI) | Scaled-up decoder Transformer |
| 2019 | T5 (Google) | Full encoder-decoder Transformer |
| 2020 | GPT-3 (OpenAI) | 175B-param decoder Transformer |
| 2020 | Vision Transformer (ViT) | Transformer applied to images |
| 2021 | DALL·E (OpenAI) | Transformer for text → image |
| 2022 | ChatGPT / InstructGPT | RLHF-tuned decoder Transformer |
| 2023 | GPT-4, LLaMA, PaLM 2 | Continued scaling & refinement |
| 2024 | Gemini, Mixtral, Claude 3 | Multi-modal Transformers |

### Why the Transformer Won

1. **Parallelism** — Training is orders of magnitude faster than RNNs.
2. **Scalability** — Performance improves predictably with more data and parameters.
3. **Flexibility** — Works for text, images, audio, code, proteins, and more.
4. **Simplicity** — The architecture is conceptually straightforward (attention + FFN + residuals).

---

## 9. Real-World Applications

| Domain | Application | Transformer Variant |
|--------|-------------|---------------------|
| Machine Translation | Google Translate | Encoder-Decoder Transformer |
| Search & Retrieval | Google Search (BERT) | Encoder-only |
| Text Generation | ChatGPT, Claude | Decoder-only |
| Code Generation | GitHub Copilot | Decoder-only |
| Image Recognition | ViT, DINOv2 | Encoder-only (patched images) |
| Speech Recognition | Whisper (OpenAI) | Encoder-Decoder |
| Protein Folding | AlphaFold 2 | Attention-based architecture |
| Music Generation | MusicLM | Encoder-Decoder |

---

## 10. Summary Table

| Aspect | Detail |
|--------|--------|
| **Paper** | "Attention Is All You Need" — Vaswani et al., NeurIPS 2017 |
| **Core Idea** | Replace recurrence with self-attention for sequence modelling |
| **Architecture** | Encoder-decoder, each with 6 identical layers |
| **Attention Type** | Scaled dot-product, multi-head (h=8) |
| **Key Dimensions** | d_model=512, d_k=d_v=64, d_ff=2048 |
| **Position Info** | Sinusoidal positional encodings |
| **Training** | WMT 2014 EN→DE and EN→FR, label smoothing, Adam with warm-up |
| **Result** | New SOTA on EN→DE (28.4 BLEU) and EN→FR (41.0 BLEU) |
| **Legacy** | Foundation of BERT, GPT, T5, ViT, and all modern LLMs |

---

## 11. Revision Questions

1. **Why does the Transformer scale the dot products by `1/√d_k`?**
   Hint: Think about what happens to the magnitude of dot products as `d_k` grows and the effect on softmax gradients.

2. **How does the Transformer handle word order without recurrence?**
   Hint: Positional encodings — sinusoidal functions of position and dimension index.

3. **What are the three types of attention used in the original Transformer?**
   Hint: Self-attention in encoder, masked self-attention in decoder, and encoder-decoder (cross) attention.

4. **Why is multi-head attention preferable to single-head attention with the same total dimension?**
   Hint: Multiple heads allow the model to jointly attend to information from different representation sub-spaces.

5. **Calculate the total parameter count for the multi-head attention in one encoder layer with d_model=512, h=8.**
   Hint: Four projection matrices (Wᵠ, Wᴷ, Wⱽ, Wᴼ), each of size 512×512, plus biases.

6. **What training schedule (learning rate) did the paper use, and why?**
   Hint: Linear warm-up for the first 4000 steps, then decay proportional to the inverse square root of the step number.

---

[← Parameter Efficiency](../03-Multi-Head-Attention/04-parameter-efficiency.md) | [Encoder Architecture →](02-encoder-architecture.md)
