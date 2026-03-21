# Transformer Architecture Preview

> **Unit 10, Chapter 2** — How Transformers replaced RNNs by relying entirely on attention, and why "Attention Is All You Need" changed everything.

---

## 📋 Overview

The **Transformer** (Vaswani et al., 2017) is the architecture that dethroned RNNs as the dominant model for sequence processing. By eliminating recurrence entirely and relying solely on **self-attention** and **feed-forward layers**, Transformers achieve massive parallelism during training and superior performance on virtually all NLP benchmarks. This chapter provides a high-level preview — enough to understand why Transformers superseded RNNs and how the core ideas connect.

---

## 🏗️ Transformer Architecture Overview

### High-Level Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    TRANSFORMER ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Input                                          Output          │
│   Tokens                                         Tokens          │
│     │                                              ↑             │
│     ▼                                              │             │
│  ┌──────────┐                              ┌──────────────┐     │
│  │ Input     │                              │  Linear +    │     │
│  │ Embedding │                              │  Softmax     │     │
│  │ + PosEnc  │                              └──────────────┘     │
│  └──────────┘                                      ↑             │
│     │                                              │             │
│     ▼                                       ┌──────────────┐     │
│  ┌───────────────────┐                      │  Output      │     │
│  │                   │                      │  Embedding   │     │
│  │   ENCODER         │                      │  + PosEnc    │     │
│  │   (N=6 layers)    │                      └──────────────┘     │
│  │                   │                             ↑             │
│  │  ┌─────────────┐  │         ┌───────────────────────────┐    │
│  │  │ Self-Attn   │  │         │                           │    │
│  │  │ + Add&Norm  │  │    ┌───▶│     DECODER               │    │
│  │  ├─────────────┤  │    │    │     (N=6 layers)          │    │
│  │  │ Feed-Forward│  │    │    │                           │    │
│  │  │ + Add&Norm  │  │    │    │  ┌─────────────────────┐  │    │
│  │  └─────────────┘  │    │    │  │ Masked Self-Attn    │  │    │
│  │    × N layers     │    │    │  │ + Add&Norm          │  │    │
│  │                   │────┘    │  ├─────────────────────┤  │    │
│  └───────────────────┘         │  │ Cross-Attention     │  │    │
│                                │  │ (attend to encoder) │  │    │
│                                │  │ + Add&Norm          │  │    │
│                                │  ├─────────────────────┤  │    │
│                                │  │ Feed-Forward        │  │    │
│                                │  │ + Add&Norm          │  │    │
│                                │  └─────────────────────┘  │    │
│                                │    × N layers             │    │
│                                │                           │    │
│                                └───────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Key Components

### 1. Multi-Head Self-Attention

```
Every position attends to every other position:

Input:  "The  cat  sat  on  the  mat"
         ↓    ↓    ↓   ↓    ↓    ↓
       ┌─────────────────────────────┐
       │    Multi-Head Attention     │
       │  (8 heads, each d_k = 64)  │
       │                             │
       │  Head 1: syntactic deps     │
       │  Head 2: semantic sim       │
       │  Head 3: local context      │
       │  Head 4: long-range deps    │
       │  ...                        │
       └─────────────────────────────┘
         ↓    ↓    ↓   ↓    ↓    ↓
Output: context-aware representations
```

### 2. Position-wise Feed-Forward Network

```
FFN(x) = ReLU(x · W₁ + b₁) · W₂ + b₂

Applied independently to each position:
  Position 1: x₁ → FFN → z₁
  Position 2: x₂ → FFN → z₂     (same weights, different inputs)
  Position 3: x₃ → FFN → z₃

Typically: d_model = 512 → d_ff = 2048 → d_model = 512
(Expand, transform, compress)
```

### 3. Residual Connections + Layer Normalization

```
For each sub-layer (attention or FFN):

  Output = LayerNorm(x + SubLayer(x))

Why residual connections?
  • Enable gradient flow through deep networks
  • Each layer learns a "refinement" of the input
  • Similar to LSTM cell state highway

  x ──────────────────┐
  │                    │ (skip connection)
  ▼                    │
  [Sub-Layer]          │
  │                    │
  ▼                    ▼
  [    Add (x + out)    ]
  │
  ▼
  [    LayerNorm        ]
  │
  ▼
  output
```

### 4. Positional Encoding

```
Since there's no recurrence, position must be explicitly encoded:

PE(pos, 2i)   = sin(pos / 10000^(2i/d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))

Intuition: Each dimension oscillates at a different frequency.
Nearby positions have similar encodings.
The model can learn to attend to relative positions.

Position 0: [sin(0), cos(0), sin(0), cos(0), ...]  = [0, 1, 0, 1, ...]
Position 1: [sin(1), cos(1), sin(0.0001), cos(0.0001), ...]
Position 2: [sin(2), cos(2), sin(0.0002), cos(0.0002), ...]
```

### 5. Masked Attention (Decoder)

```
In the decoder, position i can only attend to positions ≤ i
(can't look at future tokens during generation):

Attention mask:
         pos 1  pos 2  pos 3  pos 4
pos 1  [  ✓      ✗      ✗      ✗  ]
pos 2  [  ✓      ✓      ✗      ✗  ]
pos 3  [  ✓      ✓      ✓      ✗  ]
pos 4  [  ✓      ✓      ✓      ✓  ]

✗ = -∞ before softmax → 0 attention weight
```

---

## 🔬 Why Transformers Replaced RNNs

### Training Speed

```
RNN Training:
  x₁ → h₁ → h₂ → h₃ → ... → h_n
  Must compute SEQUENTIALLY: O(n) serial steps
  GPU utilization: LOW (most cores idle)

Transformer Training:
  [x₁, x₂, x₃, ..., x_n]  →  self-attention  →  [z₁, z₂, z₃, ..., z_n]
  All positions computed IN PARALLEL: O(1) serial steps
  GPU utilization: HIGH (all cores active)

Real-world impact:
  LSTM:        ~1 day to train on 1M sentences
  Transformer: ~4 hours for same data & quality  (6× faster)
```

### Performance Comparison (Historical Results)

```
Machine Translation (WMT 2014 English-German):

  Model                  BLEU Score    Training Time
  ─────────────────────  ──────────    ─────────────
  LSTM Seq2Seq           20.7          6 days
  LSTM + Attention       25.8          4 days
  Transformer (base)     27.3          12 hours
  Transformer (big)      28.4          3.5 days

Transformers: Better quality AND faster training!
```

### The Key Advantages

```
┌────────────────────────────────────────────────────────┐
│  Why Transformers Won                                  │
├────────────────────────────────────────────────────────┤
│                                                        │
│  1. PARALLELISM                                        │
│     Self-attention: all positions computed at once      │
│     RNN: must wait for h₁ before computing h₂          │
│                                                        │
│  2. LONG-RANGE DEPENDENCIES                            │
│     Direct path between any two positions              │
│     No vanishing gradient through recurrence           │
│                                                        │
│  3. SCALABILITY                                        │
│     More parameters → better performance               │
│     RNNs hit diminishing returns with size             │
│                                                        │
│  4. FLEXIBILITY                                        │
│     Same architecture for NLP, vision, audio, code     │
│     RNN variants needed domain-specific design         │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## 💻 Minimal Transformer in PyTorch

```python
import torch
import torch.nn as nn
import math

class TransformerBlock(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        super().__init__()
        self.attention = nn.MultiheadAttention(d_model, num_heads, 
                                               dropout=dropout, batch_first=True)
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.ffn = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(d_ff, d_model),
            nn.Dropout(dropout)
        )
    
    def forward(self, x, mask=None):
        # Self-attention with residual + norm
        attn_out, _ = self.attention(x, x, x, attn_mask=mask)
        x = self.norm1(x + attn_out)
        
        # Feed-forward with residual + norm
        ffn_out = self.ffn(x)
        x = self.norm2(x + ffn_out)
        
        return x


class SimpleTransformer(nn.Module):
    def __init__(self, vocab_size, d_model=512, num_heads=8, 
                 num_layers=6, d_ff=2048, max_len=512, dropout=0.1):
        super().__init__()
        self.d_model = d_model
        self.embedding = nn.Embedding(vocab_size, d_model)
        self.pos_encoding = self._create_pos_encoding(max_len, d_model)
        self.layers = nn.ModuleList([
            TransformerBlock(d_model, num_heads, d_ff, dropout)
            for _ in range(num_layers)
        ])
        self.fc_out = nn.Linear(d_model, vocab_size)
        self.dropout = nn.Dropout(dropout)
    
    def _create_pos_encoding(self, max_len, d_model):
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * 
                            (-math.log(10000.0) / d_model))
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        return nn.Parameter(pe.unsqueeze(0), requires_grad=False)
    
    def forward(self, x, mask=None):
        seq_len = x.size(1)
        
        # Embedding + positional encoding
        x = self.embedding(x) * math.sqrt(self.d_model)
        x = x + self.pos_encoding[:, :seq_len, :]
        x = self.dropout(x)
        
        # Transformer layers
        for layer in self.layers:
            x = layer(x, mask)
        
        return self.fc_out(x)

# Usage
model = SimpleTransformer(vocab_size=30000, d_model=512, num_heads=8, num_layers=6)
x = torch.randint(0, 30000, (32, 100))  # batch=32, seq_len=100
output = model(x)
print(output.shape)  # [32, 100, 30000]
```

---

## 📈 The Transformer Family Tree

```
Transformer (2017)
    │
    ├── Encoder-only models (understanding)
    │   ├── BERT (2018) — bidirectional pre-training
    │   ├── RoBERTa (2019) — optimized BERT training
    │   └── DeBERTa (2020) — disentangled attention
    │
    ├── Decoder-only models (generation)
    │   ├── GPT (2018) — autoregressive language model
    │   ├── GPT-2 (2019) — scaling up
    │   ├── GPT-3 (2020) — few-shot learning
    │   └── GPT-4 (2023) — multimodal
    │
    ├── Encoder-Decoder models (seq2seq)
    │   ├── T5 (2019) — text-to-text framework
    │   └── BART (2019) — denoising autoencoder
    │
    └── Beyond NLP
        ├── ViT (2020) — Vision Transformer
        ├── AST (2021) — Audio Spectrogram Transformer
        ├── AlphaFold 2 (2021) — Protein structure
        └── Decision Transformer (2021) — RL
```

---

## 🔗 Connection to RNN Concepts

```
┌──────────────────────┬──────────────────────────────────────┐
│  RNN Concept         │  Transformer Equivalent              │
├──────────────────────┼──────────────────────────────────────┤
│ Hidden state         │ Attention output (context-aware)      │
│ Sequential processing│ Parallel self-attention              │
│ BPTT                 │ Standard backprop (no time unrolling)│
│ Gating (LSTM/GRU)    │ Attention weights (soft selection)   │
│ Encoder-Decoder      │ Same concept, attention-based        │
│ Teacher forcing      │ Same concept, masked attention       │
│ Bidirectional RNN    │ Bidirectional attention (BERT)       │
│ Deep RNN (stacking)  │ Stacked Transformer layers           │
│ Gradient clipping    │ Still used during training           │
│ Dropout              │ Applied in attention + FFN           │
│ Layer normalization  │ Critical component of Transformer    │
└──────────────────────┴──────────────────────────────────────┘
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Transformer | Attention-only architecture, no recurrence |
| Self-attention | Every position attends to every other |
| Multi-head | Multiple parallel attention patterns |
| Positional encoding | Sinusoidal functions encode position explicitly |
| Residual + LayerNorm | Enable deep stacking (6+ layers) |
| Encoder-Decoder | Encoder processes input, decoder generates output |
| Masked attention | Decoder can't see future tokens |
| Why it won | Parallelism + long-range deps + scalability |

---

## ❓ Revision Questions

1. **List the key components of a Transformer encoder layer.** How does information flow through each component?

2. **Why does the Transformer need positional encoding** while RNNs do not? What information would be lost without it?

3. **Explain how masked self-attention in the decoder** prevents the model from "cheating" during training.

4. **Compare the training parallelism of Transformers vs RNNs.** Why can Transformers utilize GPUs more efficiently?

5. **The Transformer uses residual connections and LayerNorm.** Relate these to LSTM's cell state highway. What similar problem do they solve?

6. **GPT is decoder-only while BERT is encoder-only.** What tasks is each suited for, and why the different architecture choices?

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Attention Mechanisms](01-attention-mechanisms.md) |
| ⬆️ Unit | [10 Beyond RNNs](../README.md) |
| ➡️ Next | [RNN vs Transformer](03-rnn-vs-transformer.md) |
| 🏠 Home | [RNN Home](../README.md) |

---

*Last updated: 2024*
