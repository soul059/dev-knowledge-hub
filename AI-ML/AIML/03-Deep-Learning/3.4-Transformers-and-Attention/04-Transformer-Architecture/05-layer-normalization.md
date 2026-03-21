[← Encoder-Decoder Attention](04-encoder-decoder-attention.md) | [Residual Connections →](06-residual-connections.md)

---

# Layer Normalization in Transformers

Layer Normalization (LayerNorm) is a critical component in every transformer sub-layer. Unlike Batch Normalization, which computes statistics across the batch dimension, LayerNorm computes the mean and variance across all features within a single training example. This makes it perfectly suited for transformer architectures where sequences have variable lengths and batch sizes may be small. Every encoder and decoder sub-layer in the original "Attention Is All You Need" architecture wraps its output with LayerNorm, ensuring stable activations and faster convergence throughout the network.

---

## 1. The LayerNorm Formula

Given an input vector **x** of dimension `d` (the feature/hidden dimension), Layer Normalization is defined as:

```
LN(x) = γ · (x - μ) / √(σ² + ε) + β
```

Where:

| Symbol | Meaning                                          |
|--------|--------------------------------------------------|
| x      | Input vector of shape (d,)                       |
| μ      | Mean of x across the feature dimension           |
| σ²     | Variance of x across the feature dimension       |
| ε      | Small constant for numerical stability (e.g. 1e-5) |
| γ      | Learnable scale parameter (shape d,)             |
| β      | Learnable shift parameter (shape d,)             |

### Mathematical Derivation of Statistics

For a single token's hidden representation x = [x₁, x₂, ..., x_d]:

```
         1   d
    μ = ─── Σ  xᵢ           (mean across features)
         d  i=1

         1   d
   σ² = ─── Σ  (xᵢ - μ)²    (variance across features)
         d  i=1

              xᵢ - μ
   x̂ᵢ = ─────────────       (normalize)
           √(σ² + ε)

   yᵢ = γᵢ · x̂ᵢ + βᵢ       (scale and shift)
```

The parameters γ and β are **per-feature** learnable parameters, initialized to ones and zeros respectively, giving the network the ability to undo the normalization if needed.

---

## 2. BatchNorm vs LayerNorm

### 2.1 Which Dimensions Are Normalized

```
Input tensor shape: (Batch, Sequence Length, Features)
                     B       T                d_model

┌──────────────────────────────────────────────────────┐
│                  BATCH NORMALIZATION                  │
│                                                      │
│  Normalizes across Batch (and Sequence) for EACH     │
│  feature independently.                              │
│                                                      │
│     Feature 1    Feature 2    Feature 3   ...        │
│    ┌─────────┐  ┌─────────┐  ┌─────────┐            │
│    │ ███████ │  │ ███████ │  │ ███████ │            │
│    │ ███████ │  │ ███████ │  │ ███████ │  ← Batch   │
│    │ ███████ │  │ ███████ │  │ ███████ │    samples  │
│    └─────────┘  └─────────┘  └─────────┘            │
│    Compute μ,σ  Compute μ,σ  Compute μ,σ            │
│    for this     for this     for this                │
│    feature      feature      feature                 │
│                                                      │
│  → Statistics depend on OTHER samples in the batch   │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│                  LAYER NORMALIZATION                  │
│                                                      │
│  Normalizes across ALL features for EACH sample      │
│  (each token position) independently.                │
│                                                      │
│  Sample 1: ┌─────────────────────────────────┐       │
│            │ feat1  feat2  feat3  ...  feat_d │       │
│            │  ████████████████████████████████ │  ←   │
│            └─────────────────────────────────┘  one  │
│            Compute μ,σ across ALL features      μ,σ  │
│                                                      │
│  Sample 2: ┌─────────────────────────────────┐       │
│            │ feat1  feat2  feat3  ...  feat_d │       │
│            │  ████████████████████████████████ │  ←   │
│            └─────────────────────────────────┘  own  │
│            Compute μ,σ across ALL features      μ,σ  │
│                                                      │
│  → Statistics are INDEPENDENT of other samples       │
└──────────────────────────────────────────────────────┘
```

### 2.2 Why LayerNorm Wins for Transformers

| Criterion                    | BatchNorm                       | LayerNorm                      |
|------------------------------|---------------------------------|--------------------------------|
| Depends on batch size        | Yes — unstable with small batch | No — per-sample normalization  |
| Variable-length sequences    | Problematic (padding affects μ) | No issue (per-token stats)     |
| Inference behavior           | Needs running mean/var          | Same as training               |
| Works with batch size = 1    | No (undefined statistics)       | Yes                            |
| Parallelism across sequences | Requires synchronization        | Fully independent              |
| Used in Transformers         | Rarely                          | Always (standard component)    |

**Key insight:** In NLP, sequences have variable lengths. BatchNorm would mix statistics from padded and real tokens across the batch, leading to noisy and unreliable normalization. LayerNorm avoids this entirely by normalizing each token vector independently.

---

## 3. Pre-Norm vs Post-Norm

The placement of LayerNorm relative to the sub-layer function differs between the original transformer and modern architectures.

### 3.1 Post-Norm (Original Transformer — Vaswani et al., 2017)

```
           ┌──────────────┐
    x ────►│  Sub-Layer    │────► x + SubLayer(x) ────► LayerNorm ────► output
    │      │  (Attention   │              ▲
    │      │   or FFN)     │              │
    │      └──────────────┘              │
    └────────────────────────────────────┘
                 residual connection

    output = LayerNorm(x + SubLayer(x))
```

### 3.2 Pre-Norm (GPT-2, GPT-3, LLaMA, Modern Transformers)

```
           ┌──────────────┐
    x ──► LayerNorm ────►│  Sub-Layer    │────► x + SubLayer(LN(x)) ────► output
    │                     │  (Attention   │              ▲
    │                     │   or FFN)     │              │
    │                     └──────────────┘              │
    └───────────────────────────────────────────────────┘
                      residual connection

    output = x + SubLayer(LayerNorm(x))
```

### 3.3 Comparison

| Aspect                 | Post-Norm                            | Pre-Norm                           |
|------------------------|--------------------------------------|------------------------------------|
| Training stability     | Can be unstable without warmup       | More stable, easier to train       |
| Learning rate warmup   | Usually required                     | Often not needed                   |
| Final performance      | Slightly higher with careful tuning  | Competitive, sometimes slightly lower |
| Gradient flow          | Gradients pass through LN            | Gradients bypass LN via residual   |
| Used in                | Original Transformer, BERT           | GPT-2, GPT-3, LLaMA, PaLM         |
| Depth scalability      | Harder to scale beyond 12 layers     | Scales to 100+ layers easily       |

**Why Pre-Norm is preferred in modern models:** The residual path is "clean" — gradients flow through the identity mapping without passing through normalization, making optimization easier for very deep networks.

---

## 4. Worked Numerical Example

### Setup

Let d_model = 4 and ε = 1e-5 (for simplicity we show with ε ≈ 0).

```
Input token vector:   x = [2.0, 4.0, 6.0, 8.0]

Step 1 — Compute mean:
    μ = (2.0 + 4.0 + 6.0 + 8.0) / 4
    μ = 20.0 / 4 = 5.0

Step 2 — Compute variance:
    σ² = [(2-5)² + (4-5)² + (6-5)² + (8-5)²] / 4
    σ² = [9 + 1 + 1 + 9] / 4
    σ² = 20 / 4 = 5.0

Step 3 — Normalize:
    x̂₁ = (2.0 - 5.0) / √(5.0 + 1e-5) = -3.0 / 2.2361 ≈ -1.3416
    x̂₂ = (4.0 - 5.0) / √5.0            = -1.0 / 2.2361 ≈ -0.4472
    x̂₃ = (6.0 - 5.0) / √5.0            =  1.0 / 2.2361 ≈  0.4472
    x̂₄ = (8.0 - 5.0) / √5.0            =  3.0 / 2.2361 ≈  1.3416

    Normalized: x̂ = [-1.3416, -0.4472, 0.4472, 1.3416]

Step 4 — Scale and shift (assume γ = [1,1,1,1], β = [0,0,0,0]):
    y = γ · x̂ + β = [-1.3416, -0.4472, 0.4472, 1.3416]

Verification:
    mean(y) = (-1.3416 - 0.4472 + 0.4472 + 1.3416) / 4 ≈ 0.0  ✓
    var(y)  ≈ 1.0  ✓
```

### With Learned Parameters

```
After training, suppose:   γ = [0.5, 1.0, 1.5, 2.0],  β = [0.1, 0.0, -0.1, 0.2]

    y₁ = 0.5 × (-1.3416) + 0.1  = -0.5708
    y₂ = 1.0 × (-0.4472) + 0.0  = -0.4472
    y₃ = 1.5 × ( 0.4472) - 0.1  =  0.5708
    y₄ = 2.0 × ( 1.3416) + 0.2  =  2.8832

    Output: y = [-0.5708, -0.4472, 0.5708, 2.8832]
```

The learned γ and β allow the model to control the distribution of each feature after normalization.

---

## 5. Implementation

### 5.1 LayerNorm from Scratch

```python
import torch
import torch.nn as nn

class LayerNormFromScratch(nn.Module):
    """Layer Normalization implemented from first principles."""

    def __init__(self, d_model: int, eps: float = 1e-5):
        super().__init__()
        self.eps = eps
        # Learnable parameters
        self.gamma = nn.Parameter(torch.ones(d_model))   # scale
        self.beta = nn.Parameter(torch.zeros(d_model))   # shift

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # x shape: (batch, seq_len, d_model)
        # Compute statistics across the last dimension (features)
        mean = x.mean(dim=-1, keepdim=True)          # (batch, seq_len, 1)
        var = x.var(dim=-1, keepdim=True, unbiased=False)  # (batch, seq_len, 1)

        # Normalize
        x_norm = (x - mean) / torch.sqrt(var + self.eps)

        # Scale and shift
        return self.gamma * x_norm + self.beta


# --- Verification against PyTorch built-in ---
torch.manual_seed(42)
batch, seq_len, d_model = 2, 3, 4
x = torch.randn(batch, seq_len, d_model)

# Our implementation
custom_ln = LayerNormFromScratch(d_model)
out_custom = custom_ln(x)

# PyTorch built-in
builtin_ln = nn.LayerNorm(d_model)
# Copy our parameters to built-in for fair comparison
with torch.no_grad():
    builtin_ln.weight.copy_(custom_ln.gamma)
    builtin_ln.bias.copy_(custom_ln.beta)
out_builtin = builtin_ln(x)

print("Custom output:\n", out_custom)
print("\nBuiltin output:\n", out_builtin)
print("\nMax difference:", (out_custom - out_builtin).abs().max().item())
# Expected: Max difference ≈ 0.0 (floating-point precision)
```

### 5.2 Using PyTorch nn.LayerNorm

```python
import torch
import torch.nn as nn

# Basic usage — normalize across last dimension
layer_norm = nn.LayerNorm(normalized_shape=512)  # d_model = 512

x = torch.randn(8, 50, 512)  # (batch=8, seq_len=50, d_model=512)
output = layer_norm(x)

print(f"Input  mean: {x.mean(dim=-1)[0, 0]:.4f}, var: {x.var(dim=-1)[0, 0]:.4f}")
print(f"Output mean: {output.mean(dim=-1)[0, 0]:.4f}, var: {output.var(dim=-1, unbiased=False)[0, 0]:.4f}")
# Output mean ≈ 0.0, var ≈ 1.0
```

### 5.3 Pre-Norm vs Post-Norm Sub-Layer Wrappers

```python
class PostNormSubLayer(nn.Module):
    """Post-norm: output = LayerNorm(x + SubLayer(x))"""

    def __init__(self, d_model: int, sublayer: nn.Module, dropout: float = 0.1):
        super().__init__()
        self.sublayer = sublayer
        self.norm = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, **kwargs):
        return self.norm(x + self.dropout(self.sublayer(x, **kwargs)))


class PreNormSubLayer(nn.Module):
    """Pre-norm: output = x + SubLayer(LayerNorm(x))"""

    def __init__(self, d_model: int, sublayer: nn.Module, dropout: float = 0.1):
        super().__init__()
        self.sublayer = sublayer
        self.norm = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, **kwargs):
        return x + self.dropout(self.sublayer(self.norm(x), **kwargs))


# Example usage
d_model = 512
ffn = nn.Sequential(
    nn.Linear(d_model, 2048),
    nn.ReLU(),
    nn.Linear(2048, d_model)
)

post_norm_block = PostNormSubLayer(d_model, ffn)
pre_norm_block = PreNormSubLayer(d_model, ffn)

x = torch.randn(2, 10, d_model)
print("Post-norm output shape:", post_norm_block(x).shape)  # (2, 10, 512)
print("Pre-norm output shape:", pre_norm_block(x).shape)     # (2, 10, 512)
```

---

## 6. Real-World Applications

| Application                        | How LayerNorm Is Used                                          |
|------------------------------------|----------------------------------------------------------------|
| **BERT / RoBERTa**                 | Post-norm after each self-attention and FFN sub-layer          |
| **GPT-2 / GPT-3**                  | Pre-norm before each sub-layer for training stability          |
| **LLaMA / Llama 2**               | RMSNorm (simplified LayerNorm without mean subtraction)        |
| **Vision Transformers (ViT)**      | Pre-norm applied to patch embeddings                           |
| **Speech Recognition (Whisper)**   | Pre-norm in encoder and decoder for audio features             |
| **Protein Folding (AlphaFold 2)**  | LayerNorm throughout the Evoformer attention blocks            |

### RMSNorm — A Modern Simplification

Many recent LLMs (LLaMA, Mistral, Gemma) use **RMSNorm** instead of LayerNorm:

```
RMSNorm(x) = γ · x / RMS(x)

where RMS(x) = √(1/d · Σ xᵢ²)
```

This removes the mean-centering step, reducing computation by ~10% with similar performance.

```python
class RMSNorm(nn.Module):
    """Root Mean Square Layer Normalization (used in LLaMA)."""

    def __init__(self, d_model: int, eps: float = 1e-6):
        super().__init__()
        self.eps = eps
        self.weight = nn.Parameter(torch.ones(d_model))

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        rms = torch.sqrt(x.pow(2).mean(dim=-1, keepdim=True) + self.eps)
        return self.weight * (x / rms)
```

---

## 7. Summary Table

| Concept                      | Details                                                        |
|------------------------------|----------------------------------------------------------------|
| **Formula**                  | LN(x) = γ · (x − μ) / √(σ² + ε) + β                         |
| **Statistics computed over** | Feature dimension (last dim) for each token independently      |
| **Learnable parameters**     | γ (scale, init=1) and β (shift, init=0), both shape (d_model,)|
| **Post-Norm**                | LN(x + SubLayer(x)) — original transformer, BERT              |
| **Pre-Norm**                 | x + SubLayer(LN(x)) — GPT-2/3, LLaMA, modern models          |
| **Why not BatchNorm**        | BN depends on batch stats; fails with variable-length seqs     |
| **RMSNorm variant**          | Drops mean centering: γ · x / RMS(x) — used in LLaMA          |
| **ε value**                  | Typically 1e-5 or 1e-6 for numerical stability                 |
| **Total LN layers**          | 2 per encoder layer, 3 per decoder layer in standard transformer|

---

## 8. Revision Questions

1. **Write the full LayerNorm formula and explain each component.** What role do γ and β play, and what are they initialized to?

2. **Why is LayerNorm preferred over BatchNorm in transformers?** Give at least three specific reasons related to NLP workloads.

3. **Compare pre-norm and post-norm architectures.** Which modern models use each variant, and why has pre-norm become the default?

4. **Given x = [1.0, 3.0, 5.0, 7.0], compute the LayerNorm output step by step** (assume γ = [1,1,1,1], β = [0,0,0,0], ε = 0).

5. **What is RMSNorm and how does it differ from standard LayerNorm?** Why might it be preferred in very large language models?

6. **Explain why the gradient flow is "cleaner" in pre-norm compared to post-norm.** Draw or describe the computational graph for each.

---

[← Encoder-Decoder Attention](04-encoder-decoder-attention.md) | [Residual Connections →](06-residual-connections.md)
