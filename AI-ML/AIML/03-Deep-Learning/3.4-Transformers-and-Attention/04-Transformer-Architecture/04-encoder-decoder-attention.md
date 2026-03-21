# Encoder-Decoder (Cross) Attention

[← Decoder Architecture](03-decoder-architecture.md) | [Layer Normalization →](05-layer-normalization.md)

---

## Overview

**Cross-attention** (also called encoder-decoder attention) is the mechanism by which the decoder
"looks at" the encoder's representations while generating each output token. Unlike self-attention
— where queries, keys, and values all come from the same sequence — cross-attention derives its
**queries (Q) from the decoder** and its **keys (K) and values (V) from the encoder output**. This
allows every decoder position to attend to every position in the input sequence, creating a dynamic,
learned alignment between source and target. Cross-attention is the bridge that makes the
encoder-decoder Transformer work for tasks like machine translation, summarisation, and
speech-to-text. Without it, the decoder would have no access to the input information.

---

## 1. How Cross-Attention Works

### Source of Q, K, V

| Signal | Source | Intuition |
|--------|--------|-----------|
| **Q** (queries) | Decoder hidden state at each position | "What information am I looking for?" |
| **K** (keys) | Encoder output (all positions) | "What information is available at each source position?" |
| **V** (values) | Encoder output (all positions) | "What is the actual content at each source position?" |

### Data Flow Diagram

```
ENCODER                                    DECODER
────────                                   ────────

  Source tokens                             Target tokens (shifted right)
       │                                         │
       ▼                                         ▼
  ┌──────────┐                             ┌──────────────────┐
  │ Encoder   │                             │ Masked Self-Attn │
  │ Stack     │                             │ + Add & Norm     │
  │ (6 layers)│                             └────────┬─────────┘
  └─────┬─────┘                                      │
        │                                            │ (decoder state = Q)
        │         ┌─────────────────────┐            │
        │         │                     │            │
        ├────────►│  K = enc_out · W_K  │            │
        │         │                     │            │
        │         │  V = enc_out · W_V  │◄───────────┤
        │         │                     │            │
        │         │  Q = dec_state · W_Q│            │
        │         │                     │            │
        │         │  Attention(Q, K, V) │            │
        │         │                     │            │
        │         └─────────┬───────────┘            │
        │                   │                        │
        │                   ▼                        │
        │         ┌─────────────────────┐            │
        │         │  Add & LayerNorm    │◄───────────┘
        │         └─────────┬───────────┘
        │                   │
        │                   ▼
        │           (to FFN sub-layer)
```

---

## 2. Mathematics

### Cross-Attention Formula

$$\text{CrossAttn}(Q, K, V) = \text{softmax}\!\left(\frac{Q K^T}{\sqrt{d_k}}\right) V$$

Where:
- `Q = decoder_hidden · W_Q` — shape `(m, d_model)` → `(m, d_k)` per head
- `K = encoder_output · W_K` — shape `(n, d_model)` → `(n, d_k)` per head
- `V = encoder_output · W_V` — shape `(n, d_model)` → `(n, d_v)` per head
- `m` = target (decoder) sequence length
- `n` = source (encoder) sequence length

### Attention Weight Matrix Shape

```
            Source positions (encoder)
            pos₀  pos₁  pos₂  ...  posₙ
         ┌─────────────────────────────┐
  tgt₀   │ a₀₀   a₀₁   a₀₂  ...  a₀ₙ │
  tgt₁   │ a₁₀   a₁₁   a₁₂  ...  a₁ₙ │   ← each row sums to 1
  tgt₂   │ a₂₀   a₂₁   a₂₂  ...  a₂ₙ │
   ...    │  .     .     .          .   │
  tgtₘ   │ aₘ₀   aₘ₁   aₘ₂  ...  aₘₙ │
         └─────────────────────────────┘

  Shape: (m, n)  — decoder length × encoder length
```

Each row tells the decoder how much to attend to each source position when generating that target token.

### Multi-Head Cross-Attention

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h)\, W^O$$

$$\text{head}_i = \text{CrossAttn}(Q W_i^Q,\; K W_i^K,\; V W_i^V)$$

Parameters per cross-attention module: same as self-attention — 4 projection matrices of size `(512 × 512)`.

---

## 3. Cross-Attention vs. Self-Attention

| Aspect | Self-Attention | Cross-Attention |
|--------|---------------|-----------------|
| **Q source** | Same sequence | Decoder hidden state |
| **K source** | Same sequence | Encoder output |
| **V source** | Same sequence | Encoder output |
| **Attention matrix** | (n, n) — square | (m, n) — rectangular |
| **Masking** | Optional (causal in decoder) | Typically no mask (attend to all source) |
| **Purpose** | Contextualise within one sequence | Bridge between two sequences |
| **Used in** | Encoder and decoder | Decoder only |

```
SELF-ATTENTION (encoder)         CROSS-ATTENTION (decoder)
─────────────────────────        ─────────────────────────

  Same sequence:                   Two sequences:
  ┌───────────┐                    ┌──────────┐    ┌──────────┐
  │  x  ──► Q │                    │ dec ──► Q│    │ enc ──► K│
  │  x  ──► K │                    │          │    │ enc ──► V│
  │  x  ──► V │                    └──────────┘    └──────────┘
  └───────────┘
  Attention(Q,K,V)                 Attention(Q_dec, K_enc, V_enc)
  shape: (n, n)                    shape: (m, n)
```

---

## 4. Worked Example: Translation with Cross-Attention

### Setup

Source (English): `"The cat sat"` → 3 encoder positions
Target (French): `"Le chat"` → 2 decoder positions (so far)

Suppose `d_k = 4` for simplicity and a single attention head.

### Encoder Output (after full encoder stack)

```
enc_out = [[0.8, 0.1, 0.5, 0.3],   # "The"  (position 0)
           [0.2, 0.9, 0.4, 0.7],   # "cat"  (position 1)
           [0.6, 0.3, 0.8, 0.1]]   # "sat"  (position 2)
```

### Decoder Hidden State (after masked self-attention + layer norm)

```
dec_state = [[0.5, 0.7, 0.2, 0.6],   # "Le"   (position 0)
             [0.3, 0.8, 0.5, 0.9]]   # "chat" (position 1)
```

### Step 1: Compute Q, K, V (identity projections for simplicity)

```
Q = dec_state    →  shape (2, 4)     ← from decoder
K = enc_out      →  shape (3, 4)     ← from encoder
V = enc_out      →  shape (3, 4)     ← from encoder
```

### Step 2: Compute QKᵀ (2×4 · 4×3 = 2×3)

```
QKᵀ[0,0] = 0.5·0.8 + 0.7·0.1 + 0.2·0.5 + 0.6·0.3 = 0.75
QKᵀ[0,1] = 0.5·0.2 + 0.7·0.9 + 0.2·0.4 + 0.6·0.7 = 1.23
QKᵀ[0,2] = 0.5·0.6 + 0.7·0.3 + 0.2·0.8 + 0.6·0.1 = 0.73

QKᵀ[1,0] = 0.3·0.8 + 0.8·0.1 + 0.5·0.5 + 0.9·0.3 = 0.84
QKᵀ[1,1] = 0.3·0.2 + 0.8·0.9 + 0.5·0.4 + 0.9·0.7 = 1.61
QKᵀ[1,2] = 0.3·0.6 + 0.8·0.3 + 0.5·0.8 + 0.9·0.1 = 0.91

QKᵀ = [[0.75, 1.23, 0.73],
        [0.84, 1.61, 0.91]]
```

### Step 3: Scale by √d_k = √4 = 2

```
Scaled = [[0.375, 0.615, 0.365],
          [0.420, 0.805, 0.455]]
```

### Step 4: Softmax (row-wise)

```
Row 0: exp([0.375, 0.615, 0.365]) = [1.455, 1.850, 1.441] → sum = 4.746
       weights = [0.307, 0.390, 0.303]

Row 1: exp([0.420, 0.805, 0.455]) = [1.522, 2.237, 1.576] → sum = 5.335
       weights = [0.285, 0.419, 0.295]
```

### Step 5: Weighted sum of V (encoder values)

```
Out[0] = 0.307·[0.8,0.1,0.5,0.3] + 0.390·[0.2,0.9,0.4,0.7] + 0.303·[0.6,0.3,0.8,0.1]
       = [0.246+0.078+0.182, 0.031+0.351+0.091, 0.154+0.156+0.242, 0.092+0.273+0.030]
       = [0.506, 0.473, 0.552, 0.395]

Out[1] = 0.285·[0.8,0.1,0.5,0.3] + 0.419·[0.2,0.9,0.4,0.7] + 0.295·[0.6,0.3,0.8,0.1]
       = [0.228+0.084+0.177, 0.029+0.377+0.089, 0.143+0.168+0.236, 0.086+0.293+0.030]
       = [0.489, 0.495, 0.547, 0.409]
```

### Interpretation

```
Attention weights:
                    "The"   "cat"   "sat"
  "Le"    →         0.307   0.390   0.303    ← "Le" attends most to "cat" (article→noun)
  "chat"  →         0.285   0.419   0.295    ← "chat" attends most to "cat" (translation!)
```

The cross-attention has learned an implicit alignment: the French word "chat" attends most
strongly to its English translation "cat". This is analogous to traditional alignment models
in statistical machine translation, but learned end-to-end.

---

## 5. Why Cross-Attention Works

1. **Full input visibility**: Unlike the causal mask in decoder self-attention, cross-attention
   has **no mask** — every decoder position can attend to every encoder position. The decoder
   needs to see the complete input to generate accurate outputs.

2. **Dynamic alignment**: The attention weights change depending on the current decoder state.
   When generating "chat", the model attends to "cat"; when generating "assis" (French for "sat"),
   it shifts attention to "sat".

3. **Shared across layers**: Every decoder layer receives the **same encoder output** as K and V.
   Different layers can learn to extract different types of information from the source.

4. **Multi-head diversity**: With 8 heads, some may focus on lexical alignment (word-to-word),
   others on syntactic structure, and others on positional patterns.

---

## 6. Information Flow: Complete Picture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FULL TRANSFORMER                                 │
│                                                                     │
│  SOURCE: "The cat sat"         TARGET: "<BOS> Le chat"              │
│       │                              │                              │
│       ▼                              ▼                              │
│  ┌──────────┐                  ┌───────────────┐                    │
│  │ Src Embed │                  │ Tgt Embed      │                   │
│  │ + PosEnc  │                  │ + PosEnc       │                   │
│  └─────┬─────┘                  └───────┬───────┘                    │
│        │                                │                            │
│        ▼                                ▼                            │
│  ┌──────────┐                  ┌────────────────┐                    │
│  │ Encoder   │─────────────────│ Decoder Layer 1 │                   │
│  │ Layer 1   │   enc_out       │  masked_self → cross → FFN         │
│  ├──────────┤   (K, V for     ├────────────────┤                    │
│  │ Encoder   │    every        │ Decoder Layer 2 │                   │
│  │ Layer 2   │    decoder      │  masked_self → cross → FFN         │
│  ├──────────┤    layer)       ├────────────────┤                    │
│  │   ...     │                  │     ...         │                   │
│  ├──────────┤                  ├────────────────┤                    │
│  │ Encoder   │                  │ Decoder Layer 6 │                   │
│  │ Layer 6   │                  │  masked_self → cross → FFN         │
│  └──────────┘                  └────────┬───────┘                    │
│                                         │                            │
│                                         ▼                            │
│                                  Linear + Softmax                    │
│                                         │                            │
│                                    "assis" (next token prediction)   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. Python / PyTorch Code: Cross-Attention Module

```python
import torch
import torch.nn as nn
import math


class CrossAttention(nn.Module):
    """
    Multi-head cross-attention: Q from decoder, K/V from encoder.
    This is the second sub-layer in each Transformer decoder layer.
    """

    def __init__(self, d_model: int = 512, num_heads: int = 8, dropout: float = 0.1):
        super().__init__()
        assert d_model % num_heads == 0
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        # Separate projections — Q from decoder, K/V from encoder
        self.W_q = nn.Linear(d_model, d_model)   # projects decoder state → queries
        self.W_k = nn.Linear(d_model, d_model)   # projects encoder output → keys
        self.W_v = nn.Linear(d_model, d_model)   # projects encoder output → values
        self.W_o = nn.Linear(d_model, d_model)   # output projection

        self.dropout = nn.Dropout(dropout)
        self.scale = math.sqrt(self.d_k)

    def forward(
        self,
        decoder_state: torch.Tensor,    # (batch, tgt_len, d_model)
        encoder_output: torch.Tensor,   # (batch, src_len, d_model)
        mask: torch.Tensor = None,      # optional padding mask
    ) -> torch.Tensor:
        batch = decoder_state.size(0)
        tgt_len = decoder_state.size(1)
        src_len = encoder_output.size(1)

        # Project Q from decoder, K and V from encoder
        Q = self.W_q(decoder_state)       # (batch, tgt_len, d_model)
        K = self.W_k(encoder_output)      # (batch, src_len, d_model)
        V = self.W_v(encoder_output)      # (batch, src_len, d_model)

        # Reshape for multi-head: (batch, heads, seq_len, d_k)
        Q = Q.view(batch, tgt_len, self.num_heads, self.d_k).transpose(1, 2)
        K = K.view(batch, src_len, self.num_heads, self.d_k).transpose(1, 2)
        V = V.view(batch, src_len, self.num_heads, self.d_k).transpose(1, 2)

        # Attention scores: (batch, heads, tgt_len, src_len)
        scores = torch.matmul(Q, K.transpose(-2, -1)) / self.scale

        # Apply mask (e.g., source padding mask)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float("-inf"))

        # Attention weights and weighted sum
        attn_weights = self.dropout(torch.softmax(scores, dim=-1))
        context = torch.matmul(attn_weights, V)   # (batch, heads, tgt_len, d_k)

        # Concatenate heads and project
        context = context.transpose(1, 2).contiguous().view(batch, tgt_len, -1)
        output = self.W_o(context)                 # (batch, tgt_len, d_model)

        return output


# ----- Demo: visualise cross-attention weights -----
if __name__ == "__main__":
    torch.manual_seed(42)

    d_model = 64      # small for demonstration
    num_heads = 4
    src_len = 5       # "The cat sat on mat"
    tgt_len = 4       # "Le chat assis sur"
    batch = 1

    cross_attn = CrossAttention(d_model=d_model, num_heads=num_heads)

    # Simulated encoder output and decoder state
    encoder_output = torch.randn(batch, src_len, d_model)
    decoder_state = torch.randn(batch, tgt_len, d_model)

    output = cross_attn(decoder_state, encoder_output)
    print(f"Cross-attention output shape: {output.shape}")   # → (1, 4, 64)

    # Inspect attention weights for one head
    with torch.no_grad():
        Q = cross_attn.W_q(decoder_state)
        K = cross_attn.W_k(encoder_output)
        d_k = d_model // num_heads
        Q = Q.view(batch, tgt_len, num_heads, d_k).transpose(1, 2)
        K = K.view(batch, src_len, num_heads, d_k).transpose(1, 2)
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(d_k)
        weights = torch.softmax(scores, dim=-1)

    print(f"\nAttention weights (head 0), shape {weights[0, 0].shape}:")
    print("         The    cat    sat    on     mat")
    src_tokens = ["The", "cat", "sat", "on", "mat"]
    tgt_tokens = ["Le", "chat", "assis", "sur"]
    for i, tok in enumerate(tgt_tokens):
        w = weights[0, 0, i].numpy()
        print(f"  {tok:6s}  " + "  ".join(f"{v:.3f}" for v in w))
```

---

## 8. Cross-Attention in Different Architectures

| Architecture | Cross-Attention? | Details |
|-------------|-----------------|---------|
| **Encoder-Decoder** (original Transformer, T5, BART) | ✅ Yes | Every decoder layer has cross-attention to encoder output |
| **Encoder-Only** (BERT, RoBERTa) | ❌ No | No decoder → no cross-attention |
| **Decoder-Only** (GPT, LLaMA) | ❌ No | No encoder → no cross-attention; uses only masked self-attention |
| **Vision-Language** (Flamingo) | ✅ Yes | Visual encoder output as K/V, language decoder as Q |
| **Speech-to-Text** (Whisper) | ✅ Yes | Audio encoder output as K/V, text decoder as Q |
| **Cross-Modal** (general) | ✅ Yes | One modality provides K/V, another provides Q |

### Note on Decoder-Only Models

GPT-style models do not use cross-attention. Instead, the input (prompt) and output (completion)
are concatenated into a single sequence, and **causal self-attention** handles both. This simplifies
the architecture but means the "encoder" and "decoder" roles are implicit rather than explicit.

---

## 9. Computational Cost

### Cross-Attention Complexity

| Operation | Complexity |
|-----------|-----------|
| QKᵀ computation | O(m · n · d_k) per head |
| Softmax | O(m · n) per head |
| Weighted sum (attn · V) | O(m · n · d_v) per head |
| Total per layer | O(m · n · d_model) |
| Memory for attention matrix | O(h · m · n) |

Where `m` = target length, `n` = source length, `h` = number of heads.

**Comparison with self-attention:**
- Encoder self-attention: O(n² · d_model)
- Decoder self-attention: O(m² · d_model)
- Cross-attention: O(m · n · d_model)

For machine translation with similar source and target lengths (m ≈ n), all three have comparable cost.

---

## 10. Real-World Applications

| Domain | Source (K, V) | Target (Q) | Example |
|--------|---------------|------------|---------|
| Machine Translation | Source language encoder | Target language decoder | EN → FR translation |
| Summarisation | Full document encoder | Summary decoder | Long article → abstract |
| Speech Recognition | Audio spectrogram encoder | Text decoder | Whisper model |
| Image Captioning | Image patch encoder (ViT) | Text decoder | "A cat sitting on a mat" |
| Video QA | Video frame encoder | Answer decoder | "What is the person doing?" |
| Document QA | Context passage encoder | Answer decoder | "What year was X founded?" |
| Code Generation | Natural language encoder | Code decoder | "Sort a list" → Python code |

---

## 11. Summary Table

| Aspect | Detail |
|--------|--------|
| **Name** | Cross-attention / encoder-decoder attention |
| **Position in model** | Second sub-layer of each decoder layer |
| **Q source** | Decoder hidden state |
| **K, V source** | Encoder output (same for all decoder layers) |
| **Attention matrix shape** | (tgt_len, src_len) — rectangular |
| **Masking** | Usually none (decoder sees full source); padding mask optional |
| **Number of parameters** | Same as self-attention: 4 × d_model² ≈ 1.05M |
| **Purpose** | Allows decoder to access input information while generating |
| **Analogy** | Similar to alignment in statistical MT, but learned end-to-end |
| **Absent in** | Encoder-only (BERT) and decoder-only (GPT) architectures |

---

## 12. Revision Questions

1. **In cross-attention, which component provides Q and which provides K and V?**
   Hint: Q comes from the decoder's current hidden state; K and V come from the encoder's output.

2. **Why is the cross-attention matrix rectangular (m × n) while self-attention is square (n × n)?**
   Hint: Cross-attention compares two sequences of different lengths — the decoder sequence (m) against the encoder sequence (n).

3. **Does cross-attention use a causal mask? Why or why not?**
   Hint: No. The decoder needs to see the entire input to generate each output token. Only decoder self-attention uses a causal mask.

4. **How does the same encoder output get used by all 6 decoder layers?**
   Hint: The encoder is run once; its final output is passed as K and V to the cross-attention sub-layer of every decoder layer.

5. **In a translation model, what would the cross-attention weights look like for a well-trained model?**
   Hint: Each target word would place high attention weight on its corresponding source word(s), forming a roughly diagonal alignment pattern (with deviations for word reordering).

6. **GPT-style models don't use cross-attention. How do they handle the "input → output" relationship instead?**
   Hint: They concatenate the prompt and completion into one sequence and use causal self-attention over the entire concatenation.

---

[← Decoder Architecture](03-decoder-architecture.md) | [Layer Normalization →](05-layer-normalization.md)
