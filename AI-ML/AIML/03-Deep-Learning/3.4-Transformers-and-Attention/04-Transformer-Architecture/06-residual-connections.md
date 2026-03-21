[← Layer Normalization](05-layer-normalization.md) | [Feed-Forward Networks →](07-feed-forward-networks.md)

---

# Residual (Skip) Connections in Transformers

Residual connections — also called skip connections — are one of the simplest yet most impactful architectural ideas in deep learning. In a transformer, every sub-layer (self-attention, cross-attention, and feed-forward network) is wrapped with a residual connection that adds the sub-layer's input directly to its output: `output = x + SubLayer(x)`. This seemingly trivial addition is what makes it possible to train transformers with 6, 12, 24, or even 100+ layers. Without residual connections, gradients vanish exponentially as they propagate backward through deep networks, making training effectively impossible. This note explores the mathematics, intuition, and implementation of residual connections within the transformer architecture.

---

## 1. The Core Formula

For every sub-layer in a transformer (attention or FFN), the output is computed as:

```
output = x + SubLayer(x)
```

Combined with Layer Normalization, the two standard variants are:

```
Post-Norm:   output = LayerNorm(x + SubLayer(x))
Pre-Norm:    output = x + SubLayer(LayerNorm(x))
```

### What This Means Architecturally

```
┌─────────────────────────────────────────────────────────┐
│              RESIDUAL CONNECTION PATTERN                  │
│                                                          │
│                    ┌──────────────┐                       │
│         ┌────────►│  Sub-Layer    │────────┐              │
│         │         │  (Attention   │        │              │
│         │         │   or FFN)     │        ▼              │
│  x ─────┤         └──────────────┘      (+) ────► output │
│         │                                 ▲              │
│         │         identity path           │              │
│         └─────────────────────────────────┘              │
│                                                          │
│         output = x + SubLayer(x)                         │
└─────────────────────────────────────────────────────────┘
```

**Critical constraint:** `SubLayer(x)` must produce output with the **same dimensions** as `x`. This is why every sub-layer in a transformer outputs tensors of shape `(batch, seq_len, d_model)`, matching the input exactly.

---

## 2. Why Residual Connections Matter

### 2.1 The Vanishing Gradient Problem

In a deep network **without** residual connections, consider L stacked layers:

```
Without residual connections:
    y = f_L(f_{L-1}(...f_2(f_1(x))...))

Gradient via chain rule:
    ∂y/∂x = ∂f_L/∂f_{L-1} · ∂f_{L-1}/∂f_{L-2} · ... · ∂f_2/∂f_1 · ∂f_1/∂x

If each |∂fᵢ/∂fᵢ₋₁| < 1:
    ∂y/∂x → 0   as L → ∞     (VANISHING gradients)

If each |∂fᵢ/∂fᵢ₋₁| > 1:
    ∂y/∂x → ∞   as L → ∞     (EXPLODING gradients)
```

### 2.2 How Residual Connections Fix This

With residual connections, each layer computes `h_l = h_{l-1} + f_l(h_{l-1})`:

```
With residual connections:
    h₁ = x + f₁(x)
    h₂ = h₁ + f₂(h₁)
    ...
    h_L = h_{L-1} + f_L(h_{L-1})

Gradient of layer l with respect to layer k (where l > k):

    ∂h_l        l
    ──── = I + Σ  (terms involving ∂fᵢ/∂hᵢ₋₁)
    ∂h_k      i=k+1

The identity matrix I ensures the gradient is at LEAST 1.
Gradients can always flow directly through the skip path.
```

**Key insight:** The gradient has a direct path (the identity `I`) that doesn't pass through any nonlinearity or weight matrix. Even if `∂f/∂h` is small, the gradient never vanishes because of the additive `I` term.

### 2.3 Gradient Flow Visualization

```
WITHOUT Residual Connections (gradient vanishes):

    Loss ← Layer 6 ← Layer 5 ← Layer 4 ← Layer 3 ← Layer 2 ← Layer 1
     ▼       ▼         ▼         ▼         ▼         ▼         ▼
   grad    0.95×     0.90×     0.86×     0.81×     0.77×     0.73×
           grad      grad      grad      grad      grad      grad

    After 6 layers: 0.95⁶ ≈ 0.735 of gradient remains
    After 50 layers: 0.95⁵⁰ ≈ 0.077 of gradient remains  ← nearly gone!


WITH Residual Connections (gradient preserved):

    Loss
     │
     ▼
   Layer 6 ────────────────────────────────────────────┐
     │                                                  │
     ▼                                                  ▼
   Layer 5 ───────────────────────────────┐          gradient
     │                                     │          highway
     ▼                                     ▼            │
   Layer 4 ──────────────────┐          gradient        │
     │                        │          highway        │
     ▼                        ▼            │            │
   Layer 3 ─────┐          gradient        │            │
     │           │          highway        │            │
     ▼           ▼            │            │            │
   Layer 2    gradient        │            │            │
     │        highway         │            │            │
     ▼           │            │            │            │
   Layer 1       │            │            │            │
     │           │            │            │            │
     ▼           ▼            ▼            ▼            ▼
    Input    Receives gradient from ALL layers above!

    Gradient at Layer 1 = direct_grad + contributions from every layer
```

---

## 3. Residual Connections in the Full Transformer

### 3.1 Encoder Layer

```
┌──────────────────────────────────────────────────────┐
│                   ENCODER LAYER                       │
│                                                       │
│   Input x                                             │
│     │                                                 │
│     ├──────────────────────────────────┐              │
│     ▼                                  │              │
│   ┌──────────────────┐                 │              │
│   │  Multi-Head       │                 │ Residual    │
│   │  Self-Attention   │                 │ Connection  │
│   └────────┬─────────┘                 │ #1          │
│            ▼                           │              │
│          (+) ◄─────────────────────────┘              │
│            │                                          │
│            ▼                                          │
│       LayerNorm                                       │
│            │                                          │
│            ├──────────────────────────────┐           │
│            ▼                              │           │
│   ┌──────────────────┐                   │ Residual  │
│   │  Feed-Forward     │                   │ Connection│
│   │  Network (FFN)    │                   │ #2        │
│   └────────┬─────────┘                   │           │
│            ▼                              │           │
│          (+) ◄────────────────────────────┘           │
│            │                                          │
│            ▼                                          │
│       LayerNorm                                       │
│            │                                          │
│            ▼                                          │
│         Output                                        │
└──────────────────────────────────────────────────────┘
```

### 3.2 Decoder Layer

The decoder has **three** residual connections per layer:

```
Input x
  ├───────────────────────────────┐  Residual #1
  ▼                               │
  Masked Self-Attention           │
  ▼                               │
  (+) ◄──────────────────────────┘
  │
  LayerNorm
  ├───────────────────────────────┐  Residual #2
  ▼                               │
  Encoder-Decoder Attention       │
  ▼                               │
  (+) ◄──────────────────────────┘
  │
  LayerNorm
  ├───────────────────────────────┐  Residual #3
  ▼                               │
  Feed-Forward Network            │
  ▼                               │
  (+) ◄──────────────────────────┘
  │
  LayerNorm
  ▼
  Output
```

---

## 4. Connection to ResNet (He et al., 2015)

The residual connection idea was popularized by **ResNet** (Residual Networks) for image classification:

| Aspect              | ResNet                                   | Transformer                             |
|---------------------|------------------------------------------|-----------------------------------------|
| Year introduced     | 2015                                     | 2017                                    |
| Skip connection     | H(x) = F(x) + x                         | output = SubLayer(x) + x               |
| Sub-layer type      | Conv → BN → ReLU blocks                 | Attention / FFN blocks                  |
| Normalization       | Batch Normalization                      | Layer Normalization                     |
| Depth enabled       | Up to 152 layers (ResNet-152)            | 6-100+ layers in modern transformers    |
| Dimension matching  | 1×1 conv for dimension change            | d_model kept constant across all layers |

The transformer directly borrowed this concept: the "Attention Is All You Need" paper cites ResNet and applies the same principle to attention and FFN sub-layers.

---

## 5. Mathematical Analysis: Dimension Constraint

For the addition `x + SubLayer(x)` to be valid, both tensors must have identical shapes:

```
x:            shape (batch, seq_len, d_model)
SubLayer(x):  shape (batch, seq_len, d_model)   ← MUST MATCH

This is why:
  • Multi-Head Attention projects output back to d_model via W_O
  • FFN's second linear layer maps back to d_model
  • d_model remains constant throughout the entire model
```

### What Happens Without Dimension Matching

```
If SubLayer maps d_model → d_different:
    x:            (batch, seq_len, 512)
    SubLayer(x):  (batch, seq_len, 1024)    ← Can't add!

Solutions (if needed):
    1. Project x: x_proj = x @ W_proj     (adds parameters)
    2. Keep d_model constant (transformer's approach — simpler)
```

---

## 6. Worked Numerical Example

### Simple 2D Example

```
Input:           x = [1.0, 2.0, 3.0, 4.0]      (d_model = 4)

Suppose the sub-layer (e.g., self-attention) produces:
SubLayer(x) =       [0.3, -0.5, 0.8, -0.2]

Residual connection:
output = x + SubLayer(x)
output = [1.0 + 0.3,  2.0 + (-0.5),  3.0 + 0.8,  4.0 + (-0.2)]
output = [1.3,  1.5,  3.8,  3.8]
```

### Gradient Flow Example

```
Forward: output = x + f(x)

Backward (gradient of loss L with respect to x):
    ∂L/∂x = ∂L/∂output · ∂output/∂x
           = ∂L/∂output · (1 + ∂f(x)/∂x)
           = ∂L/∂output + ∂L/∂output · ∂f(x)/∂x
             ─────────    ────────────────────────
             direct path   through sub-layer
             (always = 1)  (may be small)

Even if ∂f(x)/∂x ≈ 0, the gradient ∂L/∂output passes through unchanged!
```

### Multi-Layer Gradient Accumulation

```
3-layer network with residual connections:

    h₀ = x
    h₁ = h₀ + f₁(h₀)     Layer 1
    h₂ = h₁ + f₂(h₁)     Layer 2
    h₃ = h₂ + f₃(h₂)     Layer 3

    ∂h₃/∂h₀ = (1 + ∂f₃/∂h₂)(1 + ∂f₂/∂h₁)(1 + ∂f₁/∂h₀)

Expanding:
    = 1 + ∂f₁/∂h₀ + ∂f₂/∂h₁ + ∂f₃/∂h₂
        + (cross terms)
        + ∂f₃/∂h₂ · ∂f₂/∂h₁ · ∂f₁/∂h₀

The "1" ensures the gradient never fully vanishes.
```

---

## 7. Implementation

### 7.1 Basic Residual Connection + LayerNorm

```python
import torch
import torch.nn as nn

class ResidualConnection(nn.Module):
    """Residual connection followed by Layer Normalization (post-norm)."""

    def __init__(self, d_model: int, dropout: float = 0.1):
        super().__init__()
        self.norm = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x: torch.Tensor, sublayer_output: torch.Tensor) -> torch.Tensor:
        """
        Args:
            x: original input (batch, seq_len, d_model)
            sublayer_output: output from attention or FFN
        """
        return self.norm(x + self.dropout(sublayer_output))
```

### 7.2 Complete Encoder Layer with Residual Connections

```python
class TransformerEncoderLayer(nn.Module):
    """Full encoder layer with two residual connections."""

    def __init__(self, d_model: int = 512, n_heads: int = 8,
                 d_ff: int = 2048, dropout: float = 0.1):
        super().__init__()
        # Sub-layers
        self.self_attention = nn.MultiheadAttention(
            d_model, n_heads, dropout=dropout, batch_first=True
        )
        self.feed_forward = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.ReLU(),
            nn.Linear(d_ff, d_model)
        )

        # Layer norms for each residual connection
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)

        # Dropout
        self.dropout1 = nn.Dropout(dropout)
        self.dropout2 = nn.Dropout(dropout)

    def forward(self, x: torch.Tensor,
                src_mask: torch.Tensor = None) -> torch.Tensor:
        # Residual connection #1: Self-Attention
        attn_output, _ = self.self_attention(x, x, x, attn_mask=src_mask)
        x = self.norm1(x + self.dropout1(attn_output))  # post-norm

        # Residual connection #2: Feed-Forward
        ff_output = self.feed_forward(x)
        x = self.norm2(x + self.dropout2(ff_output))    # post-norm

        return x


# --- Test ---
torch.manual_seed(42)
d_model = 512
encoder_layer = TransformerEncoderLayer(d_model=d_model)
x = torch.randn(2, 10, d_model)  # (batch=2, seq_len=10, d_model=512)

output = encoder_layer(x)
print(f"Input shape:  {x.shape}")       # (2, 10, 512)
print(f"Output shape: {output.shape}")   # (2, 10, 512) — same!
```

### 7.3 Pre-Norm Variant (Modern Style)

```python
class PreNormEncoderLayer(nn.Module):
    """Encoder layer with pre-norm residual connections (GPT-style)."""

    def __init__(self, d_model: int = 512, n_heads: int = 8,
                 d_ff: int = 2048, dropout: float = 0.1):
        super().__init__()
        self.self_attention = nn.MultiheadAttention(
            d_model, n_heads, dropout=dropout, batch_first=True
        )
        self.feed_forward = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.ReLU(),
            nn.Linear(d_ff, d_model)
        )
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.dropout1 = nn.Dropout(dropout)
        self.dropout2 = nn.Dropout(dropout)

    def forward(self, x: torch.Tensor,
                src_mask: torch.Tensor = None) -> torch.Tensor:
        # Pre-norm: normalize BEFORE the sub-layer
        normed = self.norm1(x)
        attn_output, _ = self.self_attention(normed, normed, normed,
                                              attn_mask=src_mask)
        x = x + self.dropout1(attn_output)   # clean residual path

        normed = self.norm2(x)
        ff_output = self.feed_forward(normed)
        x = x + self.dropout2(ff_output)     # clean residual path

        return x
```

### 7.4 Verifying Gradient Flow

```python
# Demonstrate that residual connections preserve gradient magnitude
torch.manual_seed(0)
d_model = 64

# Stack 20 layers
layers = nn.ModuleList([
    PreNormEncoderLayer(d_model=d_model, n_heads=4, d_ff=256)
    for _ in range(20)
])

x = torch.randn(1, 5, d_model, requires_grad=True)
h = x
for layer in layers:
    h = layer(h)

loss = h.sum()
loss.backward()

print(f"Gradient norm at input after 20 layers: {x.grad.norm().item():.4f}")
# With residual connections: gradient norm stays healthy (not near zero)
# Without: would be vanishingly small
```

---

## 8. Real-World Applications

| Model / System               | Residual Connection Details                                  |
|-------------------------------|--------------------------------------------------------------|
| **Original Transformer**      | Post-norm residual around each of 6 encoder + 6 decoder layers |
| **BERT-Large**                | 24 encoder layers, each with 2 residual connections (48 total) |
| **GPT-3 (175B)**              | 96 decoder layers with pre-norm residual connections          |
| **ResNet-152 (Vision)**       | 152 convolutional layers — original use of skip connections   |
| **U-Net (Segmentation)**      | Skip connections across encoder-decoder (different style)     |
| **DeepSpeed / Megatron**      | Residual connections critical for training 530B+ param models |

---

## 9. Summary Table

| Concept                       | Details                                                    |
|-------------------------------|------------------------------------------------------------|
| **Formula**                   | output = x + SubLayer(x)                                   |
| **Purpose**                   | Enable gradient flow in deep networks                      |
| **Gradient benefit**          | ∂output/∂x = I + ∂SubLayer/∂x — identity term prevents vanishing |
| **Dimension constraint**      | SubLayer output must match input shape (d_model)           |
| **Per encoder layer**         | 2 residual connections (self-attn + FFN)                   |
| **Per decoder layer**         | 3 residual connections (self-attn + cross-attn + FFN)      |
| **Post-norm**                 | LayerNorm(x + SubLayer(x)) — original transformer          |
| **Pre-norm**                  | x + SubLayer(LayerNorm(x)) — modern transformers           |
| **Origin**                    | ResNet (He et al., 2015) for image classification          |
| **Enables depth**             | 6 → 100+ layers trainable with stable gradients            |

---

## 10. Revision Questions

1. **Write the residual connection formula and explain why the identity term in the gradient prevents vanishing gradients.** Show the gradient derivation.

2. **Why must the sub-layer output dimension match the input dimension?** How does the transformer architecture ensure this for both attention and FFN sub-layers?

3. **Compare post-norm and pre-norm residual connections.** Which provides a "cleaner" gradient highway and why?

4. **A transformer encoder has 6 layers. How many total residual connections does it contain?** What about a 6-layer decoder?

5. **Explain the connection between transformer skip connections and ResNet.** What key differences exist between the two architectures?

6. **Given input x = [1.0, 2.0, 3.0] and SubLayer(x) = [0.1, -0.3, 0.5], compute the residual connection output.** Then compute ∂output/∂x and explain what happens to the gradient.

---

[← Layer Normalization](05-layer-normalization.md) | [Feed-Forward Networks →](07-feed-forward-networks.md)
