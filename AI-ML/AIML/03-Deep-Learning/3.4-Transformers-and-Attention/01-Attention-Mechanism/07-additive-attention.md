# Additive (Bahdanau) Attention

[← Previous: Scaled Dot-Product Attention](06-scaled-dot-product-attention.md) | [Next: Self-Attention Concept →](../02-Self-Attention/01-self-attention-concept.md)

---

## Overview

Additive attention, introduced by Dzmitry Bahdanau, KyungHyun Cho, and Yoshua Bengio in their landmark 2014 paper *"Neural Machine Translation by Jointly Learning to Align and Translate"*, was the **first attention mechanism** applied to neural machine translation. Before this work, encoder-decoder models compressed an entire input sentence into a single fixed-length context vector — a severe information bottleneck. Bahdanau attention solved this by allowing the decoder to dynamically focus on different parts of the encoder output at each decoding step. The mechanism uses a small feed-forward neural network to compute alignment scores, making it an "additive" approach (as opposed to the "multiplicative" dot-product methods). This paper fundamentally changed the trajectory of NLP and directly paved the way for the Transformer architecture.

---

## Historical Significance

```
Timeline:
  2014 ─── Bahdanau et al. ──► First attention for NMT
     │                          "Neural Machine Translation by Jointly
     │                           Learning to Align and Translate"
     │
  2015 ─── Luong et al. ────► Simplified multiplicative attention
     │                          "Effective Approaches to Attention-based
     │                           Neural Machine Translation"
     │
  2017 ─── Vaswani et al. ──► Transformer (self-attention, multi-head)
     │                          "Attention Is All You Need"
     │
  2018+ ── BERT, GPT, etc. ─► Attention becomes universal
```

Before Bahdanau attention, seq2seq models used a single context vector:

```
Without Attention:
  Encoder: [h₁, h₂, h₃, h₄] ──compress──► single vector c ──► Decoder

  Problem: c must encode EVERYTHING about the input.
           Long sentences lose information ("bottleneck problem").

With Bahdanau Attention:
  Encoder: [h₁, h₂, h₃, h₄] ──all available──► Decoder picks
                                                  relevant hⱼ at
                                                  each step
```

---

## The Bahdanau Attention Formula

### Core Equation

$$
\text{score}(s_i, h_j) = v_a^\top \tanh(W_1 s_i + W_2 h_j)
$$

Where:
- `s_i` ∈ ℝ^{d_s} — decoder hidden state at time step i
- `h_j` ∈ ℝ^{d_h} — encoder hidden state at position j
- `W_1` ∈ ℝ^{d_a × d_s} — learnable weight matrix for decoder state
- `W_2` ∈ ℝ^{d_a × d_h} — learnable weight matrix for encoder state
- `v_a` ∈ ℝ^{d_a} — learnable weight vector
- `d_a` — attention hidden dimension (hyperparameter)

### Full Attention Computation

```
Step 1: Compute alignment scores
  e_{i,j} = v_a^T tanh(W_1 s_i + W_2 h_j)    for all encoder positions j

Step 2: Normalize with softmax
  α_{i,j} = exp(e_{i,j}) / Σ_k exp(e_{i,k})   (attention weights)

Step 3: Compute context vector
  c_i = Σ_j α_{i,j} · h_j                     (weighted sum of encoder states)

Step 4: Use context in decoder
  s_{i+1} = f(s_i, y_i, c_i)                   (update decoder state)
```

---

## Derivation and Component Analysis

### Why Two Separate Weight Matrices?

Instead of concatenating [s_i; h_j] and using a single matrix (like the concat scoring in the previous chapter), Bahdanau separates the transformations:

```
Concat version:    W · [s_i; h_j]       ← single combined projection
Bahdanau version:  W_1 · s_i + W_2 · h_j  ← separate projections, then add
```

These are mathematically equivalent for the linear part:

```
W · [s_i; h_j] = [W_left | W_right] · [s_i; h_j]
               = W_left · s_i + W_right · h_j
               = W_1 · s_i + W_2 · h_j
```

**Computational advantage**: W_2 · h_j can be **precomputed once** for all encoder states and reused at every decoder step, since encoder states don't change during decoding.

### Role of Each Component

```
┌─────────────────────────────────────────────────────────────────┐
│  Component    │  Role                                           │
├─────────────────────────────────────────────────────────────────┤
│  W_1 · s_i   │  Projects decoder state into attention space     │
│  W_2 · h_j   │  Projects encoder state into attention space     │
│  + (addition) │  Combines both representations (additive)       │
│  tanh         │  Non-linear activation, bounds values to [-1,1] │
│  v_a^T        │  Collapses d_a-dim vector to scalar score       │
└─────────────────────────────────────────────────────────────────┘
```

### Why tanh and Not ReLU?

- **tanh** outputs values in [-1, 1], which provides both positive and negative signals
- **ReLU** kills negative values, losing half the information
- In the original paper, tanh was standard for recurrent networks
- The bounded output keeps the score in a reasonable range for softmax

---

## ASCII Diagram: Bahdanau Attention in Encoder-Decoder

```
                         ENCODER (Bidirectional GRU/LSTM)
    ┌──────────────────────────────────────────────────────────┐
    │                                                          │
    │  x₁ ──► [h₁] ──► x₂ ──► [h₂] ──► x₃ ──► [h₃] ──► x₄ ──► [h₄]  │
    │  x₁ ◄── [h₁] ◄── x₂ ◄── [h₂] ◄── x₃ ◄── [h₃] ◄── x₄ ◄── [h₄]  │
    │         ↓              ↓              ↓              ↓           │
    │    h₁ = [→h₁;←h₁]  h₂ = [→h₂;←h₂]  h₃ = [→h₃;←h₃]  h₄ = [→h₄;←h₄]  │
    └──────────────────────────────────────────────────────────┘
                  │              │              │              │
                  ▼              ▼              ▼              ▼
            ┌─────────────────────────────────────────────┐
            │           ATTENTION MECHANISM                │
            │                                             │
            │  For decoder state s_i:                     │
            │                                             │
            │  e_{i,1} = v^T tanh(W₁s_i + W₂h₁)         │
            │  e_{i,2} = v^T tanh(W₁s_i + W₂h₂)         │
            │  e_{i,3} = v^T tanh(W₁s_i + W₂h₃)         │
            │  e_{i,4} = v^T tanh(W₁s_i + W₂h₄)         │
            │                                             │
            │  [α₁, α₂, α₃, α₄] = softmax([e₁,e₂,e₃,e₄])│
            │                                             │
            │  c_i = α₁h₁ + α₂h₂ + α₃h₃ + α₄h₄         │
            └──────────────────┬──────────────────────────┘
                               │
                          c_i (context)
                               │
            ┌──────────────────┴──────────────────────────┐
            │              DECODER (GRU/LSTM)              │
            │                                             │
            │    ┌─────┐   ┌─────┐   ┌─────┐             │
            │    │ s₁  │──►│ s₂  │──►│ s₃  │──► ...      │
            │    └──┬──┘   └──┬──┘   └──┬──┘             │
            │       │         │         │                 │
            │       ▼         ▼         ▼                 │
            │      y₁        y₂        y₃                │
            └─────────────────────────────────────────────┘
```

---

## Worked Example with Small Vectors

### Setup

```
Decoder state:   s_i = [0.5, -0.3, 0.8]        (d_s = 3)
Encoder states:  h₁  = [1.0, 0.2]               (d_h = 2)
                 h₂  = [0.1, 0.9]
                 h₃  = [-0.5, 0.7]
Attention dim:   d_a = 4

W₁ (4×3):                    W₂ (4×2):               v_a (4):
┌                ┐           ┌           ┐           ┌      ┐
│  0.3  -0.1  0.2│           │  0.5  0.1 │           │  0.4 │
│  0.1   0.4  0.0│           │ -0.2  0.3 │           │ -0.3 │
│ -0.2   0.1  0.3│           │  0.1  0.4 │           │  0.5 │
│  0.0   0.2 -0.1│           │  0.3 -0.1 │           │  0.2 │
└                ┘           └           ┘           └      ┘
```

### Step 1: Compute W₁ · s_i (done once per decoder step)

```
W₁ · s_i = W₁ · [0.5, -0.3, 0.8]

Row 0: 0.3(0.5) + (-0.1)(-0.3) + 0.2(0.8) = 0.15 + 0.03 + 0.16 = 0.34
Row 1: 0.1(0.5) + 0.4(-0.3) + 0.0(0.8)    = 0.05 - 0.12 + 0.00 = -0.07
Row 2: -0.2(0.5) + 0.1(-0.3) + 0.3(0.8)   = -0.10 - 0.03 + 0.24 = 0.11
Row 3: 0.0(0.5) + 0.2(-0.3) + (-0.1)(0.8) = 0.00 - 0.06 - 0.08 = -0.14

W₁s_i = [0.34, -0.07, 0.11, -0.14]
```

### Step 2: Compute W₂ · h_j for each encoder state (precomputable)

```
W₂ · h₁ = W₂ · [1.0, 0.2]
  = [0.5(1)+0.1(0.2), -0.2(1)+0.3(0.2), 0.1(1)+0.4(0.2), 0.3(1)+(-0.1)(0.2)]
  = [0.52, -0.14, 0.18, 0.28]

W₂ · h₂ = W₂ · [0.1, 0.9]
  = [0.5(0.1)+0.1(0.9), -0.2(0.1)+0.3(0.9), 0.1(0.1)+0.4(0.9), 0.3(0.1)+(-0.1)(0.9)]
  = [0.14, 0.25, 0.37, -0.06]

W₂ · h₃ = W₂ · [-0.5, 0.7]
  = [0.5(-0.5)+0.1(0.7), -0.2(-0.5)+0.3(0.7), 0.1(-0.5)+0.4(0.7), 0.3(-0.5)+(-0.1)(0.7)]
  = [-0.18, 0.31, 0.23, -0.22]
```

### Step 3: Add and apply tanh

```
For h₁: tanh(W₁s_i + W₂h₁)
  = tanh([0.34+0.52, -0.07-0.14, 0.11+0.18, -0.14+0.28])
  = tanh([0.86, -0.21, 0.29, 0.14])
  = [0.6974, -0.2070, 0.2826, 0.1391]

For h₂: tanh(W₁s_i + W₂h₂)
  = tanh([0.34+0.14, -0.07+0.25, 0.11+0.37, -0.14-0.06])
  = tanh([0.48, 0.18, 0.48, -0.20])
  = [0.4462, 0.1781, 0.4462, -0.1974]

For h₃: tanh(W₁s_i + W₂h₃)
  = tanh([0.34-0.18, -0.07+0.31, 0.11+0.23, -0.14-0.22])
  = tanh([0.16, 0.24, 0.34, -0.36])
  = [0.1586, 0.2355, 0.3275, -0.3452]
```

### Step 4: Compute scores with v_a

```
v_a = [0.4, -0.3, 0.5, 0.2]

e₁ = v^T · [0.6974, -0.2070, 0.2826, 0.1391]
   = 0.4(0.6974) + (-0.3)(-0.2070) + 0.5(0.2826) + 0.2(0.1391)
   = 0.2790 + 0.0621 + 0.1413 + 0.0278
   = 0.5102

e₂ = v^T · [0.4462, 0.1781, 0.4462, -0.1974]
   = 0.4(0.4462) + (-0.3)(0.1781) + 0.5(0.4462) + 0.2(-0.1974)
   = 0.1785 - 0.0534 + 0.2231 - 0.0395
   = 0.3087

e₃ = v^T · [0.1586, 0.2355, 0.3275, -0.3452]
   = 0.4(0.1586) + (-0.3)(0.2355) + 0.5(0.3275) + 0.2(-0.3452)
   = 0.0634 - 0.0707 + 0.1638 - 0.0690
   = 0.0875
```

### Step 5: Softmax to get attention weights

```
Scores: [0.5102, 0.3087, 0.0875]

exp values:
  exp(0.5102) = 1.6658
  exp(0.3087) = 1.3617
  exp(0.0875) = 1.0914

Sum = 4.1189

α₁ = 1.6658 / 4.1189 = 0.4044
α₂ = 1.3617 / 4.1189 = 0.3306
α₃ = 1.0914 / 4.1189 = 0.2650

Attention weights: [0.4044, 0.3306, 0.2650]
  (sum = 1.0000 ✓)
```

### Step 6: Compute context vector

```
c_i = α₁·h₁ + α₂·h₂ + α₃·h₃
    = 0.4044·[1.0, 0.2] + 0.3306·[0.1, 0.9] + 0.2650·[-0.5, 0.7]
    = [0.4044, 0.0809] + [0.0331, 0.2975] + [-0.1325, 0.1855]
    = [0.3050, 0.5639]

→ The decoder attends most to h₁ (α=0.40) and least to h₃ (α=0.27)
```

---

## Comparison: Additive vs. Multiplicative Attention

| Aspect                    | Additive (Bahdanau)                   | Multiplicative (Dot-Product)       |
|---------------------------|----------------------------------------|------------------------------------|
| **Score formula**         | v^T tanh(W₁s + W₂h)                  | q^T k  or  q^T k / √d_k           |
| **Non-linearity**         | Yes (tanh)                             | No                                 |
| **Parameters**            | d_a(d_s + d_h) + d_a                  | 0 (or d² for general)             |
| **Computational cost**    | Higher (two matmuls + tanh + dot)     | Lower (single matmul)             |
| **Parallelism**           | Moderate                               | Excellent (batched matmul)         |
| **Small d performance**   | Better (more capacity per score)      | Weaker                             |
| **Large d performance**   | Comparable (but slower)               | Strong (with scaling)              |
| **Precomputation**        | W₂h_j can be cached                   | Not applicable                     |
| **Original context**      | RNN encoder-decoder (2014)            | Transformer (2017)                 |

### When Additive Attention Is Preferred

1. **Different dimensionalities**: When encoder and decoder hidden states have different sizes, additive attention naturally handles this via separate projection matrices.

2. **Small models**: When the overall model is small, the additive score function's extra capacity (non-linearity + parameters) can be beneficial.

3. **RNN-based architectures**: In seq2seq models with GRU/LSTM, the sequential nature already limits parallelism, so the additive scoring overhead is less impactful.

4. **Interpretability**: The two-step projection (W₁, W₂) followed by tanh provides a more interpretable intermediate representation in the attention space.

---

## PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F


class BahdanauAttention(nn.Module):
    """
    Additive (Bahdanau) Attention Mechanism.

    From: "Neural Machine Translation by Jointly Learning to Align
           and Translate" (Bahdanau et al., 2014)

    score(s_i, h_j) = v^T tanh(W1 @ s_i + W2 @ h_j)
    """

    def __init__(self, decoder_dim: int, encoder_dim: int, attention_dim: int):
        """
        Args:
            decoder_dim:   Dimensionality of decoder hidden state (d_s)
            encoder_dim:   Dimensionality of encoder hidden states (d_h)
            attention_dim: Hidden dimension of the attention network (d_a)
        """
        super().__init__()

        self.W1 = nn.Linear(decoder_dim, attention_dim, bias=False)
        self.W2 = nn.Linear(encoder_dim, attention_dim, bias=False)
        self.v = nn.Linear(attention_dim, 1, bias=False)

    def forward(
        self,
        decoder_state: torch.Tensor,
        encoder_outputs: torch.Tensor,
        mask: torch.Tensor = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Args:
            decoder_state:   (batch, decoder_dim)
            encoder_outputs: (batch, src_len, encoder_dim)
            mask:            (batch, src_len) — True for padding positions

        Returns:
            context:  (batch, encoder_dim)
            weights:  (batch, src_len)
        """
        # Project decoder state: (batch, 1, attention_dim)
        query = self.W1(decoder_state).unsqueeze(1)

        # Project encoder outputs: (batch, src_len, attention_dim)
        keys = self.W2(encoder_outputs)

        # Additive combination + tanh: (batch, src_len, attention_dim)
        energy = torch.tanh(query + keys)

        # Collapse to scalar scores: (batch, src_len)
        scores = self.v(energy).squeeze(-1)

        # Apply mask if provided
        if mask is not None:
            scores = scores.masked_fill(mask, float('-inf'))

        # Attention weights via softmax
        weights = F.softmax(scores, dim=-1)

        # Context vector: weighted sum of encoder outputs
        context = torch.bmm(weights.unsqueeze(1), encoder_outputs).squeeze(1)

        return context, weights


class BahdanauSeq2SeqDecoder(nn.Module):
    """
    Example GRU decoder with Bahdanau attention for machine translation.
    """

    def __init__(
        self,
        vocab_size: int,
        embed_dim: int,
        decoder_dim: int,
        encoder_dim: int,
        attention_dim: int,
        dropout: float = 0.1,
    ):
        super().__init__()

        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.attention = BahdanauAttention(decoder_dim, encoder_dim, attention_dim)
        self.gru = nn.GRUCell(embed_dim + encoder_dim, decoder_dim)
        self.output_proj = nn.Linear(decoder_dim + encoder_dim + embed_dim, vocab_size)
        self.dropout = nn.Dropout(dropout)

    def forward_step(
        self,
        prev_token: torch.Tensor,
        decoder_state: torch.Tensor,
        encoder_outputs: torch.Tensor,
        mask: torch.Tensor = None,
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        """
        Single decoding step.

        Args:
            prev_token:      (batch,) — previous output token indices
            decoder_state:   (batch, decoder_dim)
            encoder_outputs: (batch, src_len, encoder_dim)
            mask:            (batch, src_len)

        Returns:
            logits:        (batch, vocab_size)
            new_state:     (batch, decoder_dim)
            attn_weights:  (batch, src_len)
        """
        # Embed previous token
        embedded = self.dropout(self.embedding(prev_token))  # (batch, embed_dim)

        # Compute attention
        context, attn_weights = self.attention(decoder_state, encoder_outputs, mask)

        # GRU input: concatenate embedding and context
        gru_input = torch.cat([embedded, context], dim=-1)

        # Update decoder state
        new_state = self.gru(gru_input, decoder_state)

        # Predict next token
        output = torch.cat([new_state, context, embedded], dim=-1)
        logits = self.output_proj(output)

        return logits, new_state, attn_weights


# ── Demo ────────────────────────────────────────────────────────────

def demo():
    torch.manual_seed(42)

    # Dimensions
    batch_size = 2
    src_len = 6
    encoder_dim = 32
    decoder_dim = 24
    attention_dim = 16

    # Simulated encoder outputs
    encoder_outputs = torch.randn(batch_size, src_len, encoder_dim)
    decoder_state = torch.randn(batch_size, decoder_dim)

    # Create attention module
    attention = BahdanauAttention(decoder_dim, encoder_dim, attention_dim)

    # Count parameters
    n_params = sum(p.numel() for p in attention.parameters())
    print(f"BahdanauAttention Parameters: {n_params}")
    print(f"  W1: {decoder_dim} × {attention_dim} = {decoder_dim * attention_dim}")
    print(f"  W2: {encoder_dim} × {attention_dim} = {encoder_dim * attention_dim}")
    print(f"  v:  {attention_dim} × 1 = {attention_dim}")
    print(f"  Total: {decoder_dim * attention_dim + encoder_dim * attention_dim + attention_dim}\n")

    # Forward pass
    context, weights = attention(decoder_state, encoder_outputs)
    print(f"Context shape: {context.shape}")
    print(f"Weights shape: {weights.shape}")
    print(f"Weights sum:   {weights.sum(dim=-1)}")
    print(f"\nAttention weights (batch 0):")
    print(f"  {weights[0].detach().numpy().round(4)}")
    print(f"  Most attended position: {weights[0].argmax().item()}\n")

    # With padding mask (last 2 positions are padding)
    mask = torch.zeros(batch_size, src_len, dtype=torch.bool)
    mask[:, 4:] = True  # positions 4 and 5 are padding
    context_masked, weights_masked = attention(decoder_state, encoder_outputs, mask)
    print(f"With padding mask (positions 4,5 masked):")
    print(f"  Weights: {weights_masked[0].detach().numpy().round(4)}")
    print(f"  Masked positions have weight ≈ 0: "
          f"{weights_masked[0, 4:].sum().item():.6f}")

    # Demo full decoder step
    print("\n--- Full Decoder Step ---")
    vocab_size = 100
    embed_dim = 16

    decoder = BahdanauSeq2SeqDecoder(
        vocab_size=vocab_size,
        embed_dim=embed_dim,
        decoder_dim=decoder_dim,
        encoder_dim=encoder_dim,
        attention_dim=attention_dim,
    )

    prev_token = torch.tensor([5, 12])  # batch of 2 previous tokens
    logits, new_state, attn_weights = decoder.forward_step(
        prev_token, decoder_state, encoder_outputs
    )
    print(f"Logits shape:      {logits.shape}")
    print(f"New state shape:   {new_state.shape}")
    print(f"Attn weights shape: {attn_weights.shape}")

    total_params = sum(p.numel() for p in decoder.parameters())
    print(f"\nTotal decoder parameters: {total_params:,}")


if __name__ == "__main__":
    demo()
```

---

## Attention Visualization Example

```
Source:     "Le   chat  est  sur  le   tapis"
             ↓     ↓     ↓    ↓    ↓     ↓
            h₁    h₂    h₃   h₄   h₅    h₆

Decoding "The":    α = [0.45, 0.05, 0.05, 0.05, 0.35, 0.05]
                         ^^^^                      ^^^^
                        "Le" (the)              "le" (the)

Decoding "cat":    α = [0.02, 0.88, 0.03, 0.02, 0.02, 0.03]
                               ^^^^
                             "chat" (cat)

Decoding "is":     α = [0.05, 0.05, 0.75, 0.10, 0.02, 0.03]
                                     ^^^^
                                   "est" (is)

Decoding "on":     α = [0.03, 0.02, 0.10, 0.78, 0.04, 0.03]
                                           ^^^^
                                         "sur" (on)

Decoding "the":    α = [0.30, 0.02, 0.03, 0.05, 0.55, 0.05]
                         ^^^^                      ^^^^
                        "Le"                     "le"

Decoding "mat":    α = [0.02, 0.03, 0.02, 0.03, 0.05, 0.85]
                                                        ^^^^
                                                      "tapis" (mat)

→ The attention weights reveal the soft alignment between source and target!
```

---

## Real-World Applications

### 1. Neural Machine Translation
The original and most impactful application. Bahdanau attention allowed NMT systems to handle long sentences by attending to relevant source words at each decoding step, dramatically improving BLEU scores.

### 2. Text Summarization
Abstractive summarization models use additive attention to identify which parts of the source document are relevant for generating each word of the summary.

### 3. Speech Recognition (Listen, Attend and Spell)
The LAS model (Chan et al., 2016) uses Bahdanau-style attention to align acoustic features with text output, enabling end-to-end speech recognition without explicit alignment.

### 4. Image Captioning (Show, Attend and Tell)
Xu et al. (2015) applied additive attention to image captioning, where the model attends to different spatial regions of the image when generating each word of the caption.

### 5. Question Answering
Additive attention is used to compute the relevance between question tokens and passage tokens, helping the model locate the answer span in reading comprehension tasks.

---

## Summary Table

| Concept                      | Key Point                                                    |
|------------------------------|--------------------------------------------------------------|
| Paper                        | Bahdanau et al. (2014) — first attention for NMT             |
| Score formula                | v^T tanh(W₁s_i + W₂h_j)                                    |
| Why "additive"               | Decoder and encoder projections are added, not multiplied    |
| Key advantage                | Non-linear scoring, handles different dimensions             |
| Key disadvantage             | Slower than dot-product due to tanh and extra parameters     |
| Precomputation trick         | W₂h_j computed once for all encoder states                   |
| Parameters count             | d_a(d_s + d_h) + d_a                                        |
| Historical impact            | Solved the bottleneck problem in seq2seq models              |
| Superseded by                | Scaled dot-product attention in Transformers (2017)          |

---

## Revision Questions

**Q1.** Write the Bahdanau attention score formula and explain the role of each component (W₁, W₂, v_a, tanh).

**Q2.** Why does Bahdanau attention use two separate weight matrices (W₁ and W₂) instead of a single matrix applied to the concatenation [s_i; h_j]? What computational advantage does this provide?

**Q3.** Given s = [1, 0], h₁ = [0.5, -0.5], h₂ = [1, 1], W₁ = [[0.5, 0.2]], W₂ = [[0.3, -0.1]], and v = [1], compute the attention weights α₁ and α₂. Show all steps.

**Q4.** Explain the "bottleneck problem" in encoder-decoder models and how Bahdanau attention solves it.

**Q5.** Compare additive and multiplicative attention in terms of computational cost, expressiveness, and parallelism. In what scenarios would you still prefer additive attention today?

**Q6.** In the attention visualization example, the model attends to both "Le" and "le" when translating "The"/"the". What does this reveal about how attention handles determiners in French-to-English translation?

---

[← Previous: Scaled Dot-Product Attention](06-scaled-dot-product-attention.md) | [Next: Self-Attention Concept →](../02-Self-Attention/01-self-attention-concept.md)
