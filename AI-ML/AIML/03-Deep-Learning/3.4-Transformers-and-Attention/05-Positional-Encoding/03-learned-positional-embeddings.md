[← Sinusoidal Encoding](02-sinusoidal-encoding.md) | [Relative Positional Encoding →](04-relative-positional-encoding.md)

---

# Learned Positional Embeddings

Rather than using fixed mathematical functions to encode position, **learned positional
embeddings** treat each position as a token in its own right and learn a dense vector
representation for it during training. This approach is conceptually identical to how word
embeddings work: just as each word in the vocabulary maps to a trainable vector, each
position in the sequence (0, 1, 2, ..., max_len - 1) maps to its own trainable vector.
Learned positional embeddings are used in some of the most influential transformer models,
including BERT, GPT-2, and ViT (Vision Transformer). Despite their simplicity, they perform
comparably to sinusoidal encodings in practice, and their ability to adapt to the data
distribution makes them a compelling choice for fixed-length tasks.

---

## 1. How Learned Positional Embeddings Work

### Core Mechanism

```
┌────────────────────────────────────────────────────────────────┐
│            LEARNED POSITIONAL EMBEDDING                        │
│                                                                │
│  Position Index        Embedding Lookup Table                  │
│  ┌───┐                 ┌──────────────────────────┐            │
│  │ 0 │ ─────────────►  │ [0.12, -0.34, 0.56, ...] │  d_model  │
│  │ 1 │ ─────────────►  │ [0.78,  0.91, -0.23,...]│  d_model  │
│  │ 2 │ ─────────────►  │ [-0.45, 0.67, 0.89,...] │  d_model  │
│  │ 3 │ ─────────────►  │ [0.33, -0.11, 0.44,...] │  d_model  │
│  │...│                  │ ...                      │            │
│  │ N │ ─────────────►  │ [0.55,  0.22, -0.77,...]│  d_model  │
│  └───┘                 └──────────────────────────┘            │
│                        max_len × d_model parameters            │
│                        (ALL trainable via backprop)            │
│                                                                │
│  N = max_seq_len - 1 (e.g., 511 for BERT, 1023 for GPT-2)    │
└────────────────────────────────────────────────────────────────┘
```

The mechanism is straightforward:

1. Define a learnable embedding matrix `W_pos ∈ ℝ^(max_len × d_model)`
2. For each position `p` in the input sequence, look up row `p` from `W_pos`
3. Add the position embedding to the token embedding element-wise

```
output(p) = TokenEmbedding(token_p) + W_pos[p]
```

### Mathematical Formulation

```
Given:
  - Token embedding matrix:    E_token ∈ ℝ^(V × d_model)     (V = vocab size)
  - Position embedding matrix: E_pos   ∈ ℝ^(L × d_model)     (L = max sequence length)
  - Input token IDs:           [t₀, t₁, ..., tₙ₋₁]
  
The combined input representation:

  x_p = E_token[t_p] + E_pos[p]     for p = 0, 1, ..., n-1

Total positional parameters: L × d_model
  - BERT:   512 × 768   = 393,216 parameters
  - GPT-2:  1024 × 768  = 786,432 parameters
  - GPT-2 Large: 1024 × 1280 = 1,310,720 parameters
```

---

## 2. Architecture Diagram: Where Learned PE Fits

```
┌──────────────────────────────────────────────────────────────┐
│            BERT/GPT-2 INPUT PROCESSING                       │
│                                                              │
│  Input:  "The cat sat on the mat"                            │
│                                                              │
│  Token IDs:     [1996, 4937, 2938, 2006, 1996, 13523]       │
│  Position IDs:  [   0,    1,    2,    3,    4,     5]        │
│                                                              │
│        │                    │                                 │
│        ▼                    ▼                                 │
│  ┌────────────┐     ┌──────────────┐                         │
│  │   Token     │     │  Position    │                         │
│  │ Embedding   │     │  Embedding   │                         │
│  │ (nn.Embed)  │     │ (nn.Embed)   │                         │
│  │ V × d_model │     │ L × d_model  │                         │
│  └──────┬─────┘     └──────┬───────┘                         │
│         │                  │                                  │
│         └────────┬─────────┘                                  │
│                  │                                            │
│                  ▼                                            │
│           ┌────────────┐                                      │
│           │  Add (+)   │                                      │
│           └──────┬─────┘                                      │
│                  │                                            │
│                  ▼                                            │
│           ┌────────────┐   (BERT also adds segment embeddings │
│           │ LayerNorm  │    before this step)                 │
│           └──────┬─────┘                                      │
│                  │                                            │
│                  ▼                                            │
│           ┌────────────┐                                      │
│           │ Dropout    │                                      │
│           └──────┬─────┘                                      │
│                  │                                            │
│                  ▼                                            │
│         Transformer Layers                                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Implementation in PyTorch

```python
import torch
import torch.nn as nn
import math

class LearnedPositionalEmbedding(nn.Module):
    """
    Learned positional embeddings as used in BERT and GPT-2.
    Each position (0 to max_len-1) has its own trainable d_model-dim vector.
    """
    
    def __init__(self, d_model: int, max_len: int = 512, dropout: float = 0.1):
        super().__init__()
        self.position_embedding = nn.Embedding(max_len, d_model)
        self.dropout = nn.Dropout(p=dropout)
        
        # Register position indices as a buffer
        self.register_buffer(
            'position_ids',
            torch.arange(max_len).unsqueeze(0)  # shape: (1, max_len)
        )
    
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        """
        Args:
            x: Token embeddings, shape (batch_size, seq_len, d_model)
        Returns:
            Position-encoded embeddings, same shape
        """
        seq_len = x.size(1)
        position_ids = self.position_ids[:, :seq_len]  # (1, seq_len)
        
        # Look up position embeddings
        pos_emb = self.position_embedding(position_ids)  # (1, seq_len, d_model)
        
        # Add to token embeddings (broadcasts across batch)
        x = x + pos_emb
        return self.dropout(x)


# ──────────────────────────────────────────────────
# Usage Example
# ──────────────────────────────────────────────────

d_model = 128
max_len = 512
seq_len = 20
batch_size = 4

torch.manual_seed(42)

# Token embedding layer
vocab_size = 30000
token_embedding = nn.Embedding(vocab_size, d_model)

# Position embedding layer
pos_embedding = LearnedPositionalEmbedding(d_model=d_model, max_len=max_len, dropout=0.0)

# Forward pass
token_ids = torch.randint(0, vocab_size, (batch_size, seq_len))
token_emb = token_embedding(token_ids)          # (4, 20, 128)
encoded = pos_embedding(token_emb)              # (4, 20, 128)

print(f"Token embedding shape:  {token_emb.shape}")
print(f"Encoded output shape:   {encoded.shape}")
print(f"Position parameters:    {max_len * d_model:,}")
```

---

## 4. Worked Example with Actual Numbers

```
Setup: d_model = 4, max_len = 4, vocab_size = 5

Learned position embedding table (randomly initialized, then trained):
  W_pos = [
    pos 0: [ 0.15, -0.22,  0.08,  0.31]
    pos 1: [-0.07,  0.44,  0.19, -0.12]
    pos 2: [ 0.33, -0.15, -0.28,  0.06]
    pos 3: [ 0.21,  0.09,  0.37, -0.45]
  ]

Token embeddings:
  "the"  (id=0): [ 0.50,  0.30, -0.10,  0.20]
  "cat"  (id=1): [-0.40,  0.60,  0.80,  0.10]
  "sat"  (id=2): [ 0.20, -0.50,  0.30,  0.70]

Input: "the cat sat"  →  token IDs: [0, 1, 2], positions: [0, 1, 2]

Computing combined embeddings:

  Position 0 ("the"):
    token_emb + pos_emb = [0.50, 0.30, -0.10, 0.20] + [0.15, -0.22, 0.08, 0.31]
                        = [0.65, 0.08, -0.02, 0.51]

  Position 1 ("cat"):
    token_emb + pos_emb = [-0.40, 0.60, 0.80, 0.10] + [-0.07, 0.44, 0.19, -0.12]
                        = [-0.47, 1.04, 0.99, -0.02]

  Position 2 ("sat"):
    token_emb + pos_emb = [0.20, -0.50, 0.30, 0.70] + [0.33, -0.15, -0.28, 0.06]
                        = [0.53, -0.65, 0.02, 0.76]

Now consider reversed input: "sat cat the"  →  token IDs: [2, 1, 0], positions: [0, 1, 2]

  Position 0 ("sat"):
    token_emb + pos_emb = [0.20, -0.50, 0.30, 0.70] + [0.15, -0.22, 0.08, 0.31]
                        = [0.35, -0.72, 0.38, 1.01]    ← DIFFERENT from pos 2 above!

  The same token "sat" gets DIFFERENT representations at different positions!
```

---

## 5. Comparison: Learned vs. Sinusoidal

```python
import torch
import torch.nn as nn
import math

# ──────────────────────────────────────────────────
# Side-by-side comparison
# ──────────────────────────────────────────────────

class SinusoidalPE(nn.Module):
    def __init__(self, d_model, max_len=512):
        super().__init__()
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(
            torch.arange(0, d_model, 2).float() * -(math.log(10000.0) / d_model)
        )
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        self.register_buffer('pe', pe.unsqueeze(0))
    
    def forward(self, x):
        return x + self.pe[:, :x.size(1), :]


class LearnedPE(nn.Module):
    def __init__(self, d_model, max_len=512):
        super().__init__()
        self.pe = nn.Embedding(max_len, d_model)
        self.register_buffer('pos_ids', torch.arange(max_len).unsqueeze(0))
    
    def forward(self, x):
        return x + self.pe(self.pos_ids[:, :x.size(1)])


# Compare parameter counts
d_model = 768
max_len = 512

sinusoidal = SinusoidalPE(d_model, max_len)
learned = LearnedPE(d_model, max_len)

sin_params = sum(p.numel() for p in sinusoidal.parameters())
learned_params = sum(p.numel() for p in learned.parameters())

print(f"Sinusoidal PE parameters: {sin_params:>10,}")       # 0
print(f"Learned PE parameters:    {learned_params:>10,}")    # 393,216

# Both produce identical output shapes
x = torch.randn(2, 20, d_model)
print(f"\nSinusoidal output shape: {sinusoidal(x).shape}")
print(f"Learned output shape:    {learned(x).shape}")

# Extrapolation test
x_long = torch.randn(1, 600, d_model)
try:
    out_sin = sinusoidal(x_long)  # Works if max_len >= 600
    print(f"\nSinusoidal handles length 600: {out_sin.shape}")
except:
    print("\nSinusoidal failed for length 600")

try:
    out_learned = learned(x_long)
    print(f"Learned handles length 600: {out_learned.shape}")
except Exception as e:
    print(f"Learned FAILS for length 600: {type(e).__name__}")
```

---

## 6. Models That Use Learned Positional Embeddings

```
┌─────────────────────────────────────────────────────────────┐
│           MODELS USING LEARNED POSITIONAL EMBEDDINGS        │
│                                                             │
│  ┌─────────┐   max_len = 512    d_model = 768              │
│  │  BERT   │   Params: 393,216  Bidirectional               │
│  └─────────┘                                                │
│                                                             │
│  ┌─────────┐   max_len = 1024   d_model = 768/1280         │
│  │  GPT-2  │   Params: 786K/1.3M  Autoregressive            │
│  └─────────┘                                                │
│                                                             │
│  ┌─────────┐   max_len = 197    d_model = 768              │
│  │   ViT   │   196 patches + 1 [CLS] token                  │
│  └─────────┘   Params: 151,296                              │
│                                                             │
│  ┌─────────┐   max_len = 514    d_model = 768              │
│  │ RoBERTa │   Same as BERT but better training             │
│  └─────────┘   Params: 394,752                              │
│                                                             │
│  ┌──────────┐  max_len = 2048   d_model = 2048             │
│  │ GPT-3    │  Params: 4,194,304  (4M positional params!)  │
│  └──────────┘  (Before shift to RoPE in later models)      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. The Maximum Sequence Length Limitation

The most significant disadvantage of learned positional embeddings is the **fixed maximum
sequence length**. Since positions are a lookup table, the model cannot handle positions
it has never seen during training.

```
┌───────────────────────────────────────────────────────────┐
│         THE FIXED-LENGTH PROBLEM                          │
│                                                           │
│  Training:   max_len = 512                                │
│  Position table: [pos_0, pos_1, ..., pos_511]             │
│                                                           │
│  Inference with seq_len = 300:  ✅ Works                  │
│  Inference with seq_len = 512:  ✅ Works (boundary)       │
│  Inference with seq_len = 513:  ❌ Index out of bounds!   │
│  Inference with seq_len = 1024: ❌ Cannot handle          │
│                                                           │
│  Workarounds:                                             │
│  1. Set max_len very large (wastes parameters)            │
│  2. Interpolate positions (degrades quality)              │
│  3. Use relative/rotary encoding instead                  │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Position Interpolation (used in extending context windows)

```python
def interpolate_pos_embeddings(pos_embed, orig_len, new_len):
    """
    Linearly interpolate position embeddings to handle longer sequences.
    Used in LongLoRA, YaRN, and other context extension methods.
    """
    # pos_embed shape: (1, orig_len, d_model)
    pos_embed = pos_embed.permute(0, 2, 1)  # (1, d_model, orig_len)
    pos_embed = torch.nn.functional.interpolate(
        pos_embed.float(),
        size=new_len,
        mode='linear',
        align_corners=False
    )
    pos_embed = pos_embed.permute(0, 2, 1)  # (1, new_len, d_model)
    return pos_embed

# Example: extend 512-position embeddings to handle 1024 tokens
# original = torch.randn(1, 512, 768)
# extended = interpolate_pos_embeddings(original, 512, 1024)
```

---

## 8. Real-World Applications

| Application                     | Model & PE Details                                              |
|---------------------------------|-----------------------------------------------------------------|
| **Text Classification (BERT)**  | 512-position learned PE; fine-tuned for sentiment, NER, QA      |
| **Text Generation (GPT-2)**     | 1024-position learned PE; autoregressive language modeling       |
| **Image Classification (ViT)**  | 197-position learned PE (196 patches + CLS); 2D position info   |
| **Code Completion (CodeBERT)**  | 512-position learned PE; applied to source code tokens          |
| **Speech Recognition**          | Learned PE for spectrogram frame positions in ASR transformers  |
| **Protein Structure (ESM)**     | Learned PE for amino acid positions in protein sequences        |

---

## 9. Summary Table

| Aspect                    | Detail                                                          |
|--------------------------|-----------------------------------------------------------------|
| **Mechanism**             | Trainable lookup table: `nn.Embedding(max_len, d_model)`       |
| **Parameters**            | `max_len × d_model` (e.g., 393K for BERT)                     |
| **Learnable?**            | Yes — trained end-to-end via backpropagation                   |
| **Value Range**           | Unconstrained initially; regularized by training               |
| **Max Length**             | Fixed at initialization; cannot handle longer sequences        |
| **Extrapolation**         | Poor — no mechanism for unseen positions                       |
| **Performance**           | Comparable to sinusoidal in practice (Vaswani et al., 2017)    |
| **Key Models**            | BERT, GPT-2, GPT-3, ViT, RoBERTa                              |
| **Advantage**             | Data-adaptive; can learn task-specific position patterns        |
| **Disadvantage**          | Fixed max length; adds parameters; no theoretical guarantees   |

---

## 10. Revision Questions

1. **How does a learned positional embedding work? Describe the lookup process for a
   sequence of 3 tokens at positions 0, 1, 2.**

2. **Calculate the number of trainable parameters added by learned positional embeddings
   for a model with d_model = 1024 and max_len = 2048.**

3. **Why can't a model with learned positional embeddings handle sequences longer than
   max_len? What happens if you try?**

4. **Compare learned and sinusoidal positional embeddings in terms of: (a) parameter count,
   (b) extrapolation ability, (c) empirical performance. Which would you choose for a model
   that must handle variable-length documents?**

5. **In ViT (Vision Transformer), positions correspond to image patches rather than words.
   How does this change the interpretation of learned positional embeddings?**

6. **Describe the position interpolation technique. Why might linearly interpolating
   position embeddings degrade model quality for longer sequences?**

---

[← Sinusoidal Encoding](02-sinusoidal-encoding.md) | [Relative Positional Encoding →](04-relative-positional-encoding.md)
