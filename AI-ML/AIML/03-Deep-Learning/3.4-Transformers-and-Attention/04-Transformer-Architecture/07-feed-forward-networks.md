[← Residual Connections](06-residual-connections.md) | [Why Positional Encoding →](../05-Positional-Encoding/01-why-positional-encoding.md)

---

# Position-wise Feed-Forward Networks

The Position-wise Feed-Forward Network (FFN) is the second major sub-layer in every transformer encoder and decoder block. After the multi-head attention mechanism gathers contextual information from other positions, the FFN processes each position's representation independently through two linear transformations with a nonlinear activation in between. Despite its simplicity — it is essentially two matrix multiplications with a ReLU — the FFN accounts for roughly two-thirds of a transformer's total parameters and plays a crucial role: it introduces the non-linearity needed for the model to learn complex functions, and recent research suggests it acts as a "key-value memory" that stores factual knowledge learned during training.

---

## 1. The FFN Formula

From "Attention Is All You Need" (Vaswani et al., 2017):

```
FFN(x) = max(0, xW₁ + b₁)W₂ + b₂
```

Breaking this down:

```
Step 1 — Expand:     h = xW₁ + b₁         (d_model → d_ff)
Step 2 — Activate:   h = max(0, h)         (ReLU element-wise)
Step 3 — Compress:   y = hW₂ + b₂         (d_ff → d_model)
```

| Symbol | Shape                  | Description                        |
|--------|------------------------|------------------------------------|
| x      | (seq_len, d_model)     | Input from attention sub-layer     |
| W₁     | (d_model, d_ff)        | First weight matrix (expand)       |
| b₁     | (d_ff,)                | First bias                         |
| W₂     | (d_ff, d_model)        | Second weight matrix (compress)    |
| b₂     | (d_model,)             | Second bias                        |
| d_model| 512 (base transformer) | Model hidden dimension             |
| d_ff   | 2048 (base transformer)| Inner FFN dimension (4 × d_model)  |

---

## 2. "Position-wise" — What Does It Mean?

The FFN is applied to **each position independently and identically**. The same weight matrices W₁, W₂ are used for every token position, but there is no interaction between positions:

```
Sequence: ["The", "cat", "sat", "on", "the", "mat"]
                ↓      ↓      ↓     ↓     ↓      ↓
              FFN    FFN    FFN   FFN   FFN    FFN
              (same weights for all positions)

Token 1: FFN(x₁) = ReLU(x₁W₁ + b₁)W₂ + b₂
Token 2: FFN(x₂) = ReLU(x₂W₁ + b₁)W₂ + b₂
Token 3: FFN(x₃) = ReLU(x₃W₁ + b₁)W₂ + b₂
  ...
No interaction between tokens — that's attention's job!
```

This is equivalent to applying two **1×1 convolutions** across the sequence:
- Conv1D with kernel_size=1, in_channels=d_model, out_channels=d_ff
- Conv1D with kernel_size=1, in_channels=d_ff, out_channels=d_model

---

## 3. The Expand-Compress Architecture

### 3.1 Dimension Flow (ASCII Diagram)

```
┌──────────────────────────────────────────────────────────────┐
│               FEED-FORWARD NETWORK (FFN)                      │
│                                                               │
│  Input x                                                      │
│  ┌─────────────────────┐                                      │
│  │  d_model = 512      │                                      │
│  └──────────┬──────────┘                                      │
│             │                                                 │
│             ▼  × W₁ (512 × 2048) + b₁                        │
│  ┌─────────────────────────────────────────────────┐          │
│  │                 d_ff = 2048                      │          │
│  │    EXPANDED REPRESENTATION (4× wider)            │          │
│  └─────────────────────┬───────────────────────────┘          │
│                        │                                      │
│                        ▼  ReLU (element-wise)                 │
│  ┌─────────────────────────────────────────────────┐          │
│  │                 d_ff = 2048                      │          │
│  │    ACTIVATED (sparse — ~50% zeros from ReLU)     │          │
│  └─────────────────────┬───────────────────────────┘          │
│                        │                                      │
│                        ▼  × W₂ (2048 × 512) + b₂             │
│  ┌─────────────────────┐                                      │
│  │  d_model = 512      │                                      │
│  └──────────┬──────────┘                                      │
│             │                                                 │
│             ▼                                                 │
│         Output y                                              │
│                                                               │
│  Parameters: 512×2048 + 2048 + 2048×512 + 512                │
│            = 1,048,576 + 2,048 + 1,048,576 + 512              │
│            = 2,099,712 parameters per FFN layer               │
└──────────────────────────────────────────────────────────────┘
```

### 3.2 Why d_ff = 4 × d_model?

The 4× expansion ratio is a design choice from the original paper:

| Configuration       | d_model | d_ff   | Ratio | Model            |
|---------------------|---------|--------|-------|------------------|
| Transformer Base    | 512     | 2048   | 4×    | Original paper   |
| Transformer Big     | 1024    | 4096   | 4×    | Original paper   |
| BERT-Base           | 768     | 3072   | 4×    | Devlin et al.    |
| BERT-Large          | 1024    | 4096   | 4×    | Devlin et al.    |
| GPT-3 (175B)        | 12288   | 49152  | 4×    | Brown et al.     |
| LLaMA-7B            | 4096    | 11008  | 2.68× | Meta (with SwiGLU)|

**Intuition:** The expansion to a higher-dimensional space allows the network to learn more complex features. The compression back to d_model keeps the residual connection compatible and controls parameter count.

---

## 4. Role of FFN in the Transformer

### 4.1 Attention vs FFN — Division of Labor

```
┌─────────────────────────────────────────────────────┐
│                                                      │
│   Multi-Head Attention          Feed-Forward Network │
│   ──────────────────           ──────────────────── │
│                                                      │
│   • Mixes information          • Processes each      │
│     ACROSS positions             position ALONE      │
│                                                      │
│   • "Which tokens are          • "What to do with    │
│     relevant to each              the gathered        │
│     other?"                       information?"       │
│                                                      │
│   • Linear operation           • Non-linear: adds    │
│     (weighted sum)               expressive power    │
│                                                      │
│   • ~1/3 of parameters        • ~2/3 of parameters  │
│                                                      │
│   • Contextualizes tokens      • Stores factual      │
│                                  knowledge (recent   │
│                                  research suggests)  │
└─────────────────────────────────────────────────────┘
```

### 4.2 FFN as Key-Value Memory

Recent work (Geva et al., 2021) suggests each FFN layer acts as a memory:
- **W₁ rows** are "keys" that match input patterns
- **W₂ columns** are "values" that produce corresponding outputs
- **ReLU** selects which memories to activate

```
Input x matches row i of W₁ strongly → ReLU activates neuron i
→ Column i of W₂ is added to the output
→ This "recalls" the knowledge stored in that neuron
```

---

## 5. Modern Activation Variants

### 5.1 Activation Functions Compared

The original transformer uses ReLU, but modern models have moved to smoother alternatives:

```
ReLU(x)  = max(0, x)                    (Original Transformer)

GELU(x)  = x · Φ(x)                     (BERT, GPT-2)
         ≈ 0.5x(1 + tanh[√(2/π)(x + 0.044715x³)])

SiLU(x)  = x · σ(x)     (also called Swish)     (Some vision models)

SwiGLU(x, W₁, V, W₂):                  (LLaMA, PaLM, Mistral)
         = (xW₁ ⊙ SiLU(xV)) W₂
         (Gated Linear Unit with Swish activation)
```

### 5.2 Activation Comparison Chart

```
         ▲ output
     2.0 │                          ╱ ReLU
         │                        ╱
     1.0 │                      ╱          ·· GELU
         │                    ╱        ··
     0.0 │──────────────────╱·····─────────── → input
         │              ··╱
    -0.5 │          ··  ╱
         │       ·    ╱
    -1.0 │─────·────╱─────────────────────
        -3   -2   -1    0    1    2    3

   Key difference: GELU has a smooth curve near 0,
   allowing small negative gradients (not hard cutoff).
```

### 5.3 GLU Variants (Gated Linear Units)

Modern LLMs often use gated variants that add a multiplicative gate:

```
Standard FFN:     FFN(x) = Activation(xW₁)W₂

GLU FFN:          FFN(x) = (xW₁ ⊙ Activation(xV)) W₂
                            ───    ────────────────
                           linear      gate
                           path        path

  ⊙ = element-wise multiplication

  This requires THREE weight matrices (W₁, V, W₂) instead of two.
  To keep parameter count similar, d_ff is reduced by 2/3:
      d_ff = (2/3) × 4 × d_model ≈ 2.67 × d_model
```

---

## 6. Worked Example with Small Matrices

### Setup

```
d_model = 3,  d_ff = 4

Input (single token):
    x = [1.0, -0.5, 2.0]    shape: (1, 3)

W₁ = [[ 0.5,  0.3, -0.2,  0.8],
      [-0.1,  0.6,  0.4, -0.3],
      [ 0.7, -0.4,  0.1,  0.5]]    shape: (3, 4)

b₁ = [0.1, -0.1, 0.2, 0.0]        shape: (4,)

W₂ = [[ 0.3, -0.2,  0.5],
      [ 0.6,  0.1, -0.3],
      [-0.4,  0.7,  0.2],
      [ 0.1,  0.3, -0.6]]          shape: (4, 3)

b₂ = [0.0, 0.1, -0.1]             shape: (3,)
```

### Step 1 — Linear Transform #1: h = xW₁ + b₁

```
xW₁:
  h₁ = 1.0×0.5 + (-0.5)×(-0.1) + 2.0×0.7 = 0.5 + 0.05 + 1.4 = 1.95
  h₂ = 1.0×0.3 + (-0.5)×0.6 + 2.0×(-0.4) = 0.3 - 0.3 - 0.8 = -0.80
  h₃ = 1.0×(-0.2) + (-0.5)×0.4 + 2.0×0.1 = -0.2 - 0.2 + 0.2 = -0.20
  h₄ = 1.0×0.8 + (-0.5)×(-0.3) + 2.0×0.5 = 0.8 + 0.15 + 1.0 = 1.95

xW₁ = [1.95, -0.80, -0.20, 1.95]

h = xW₁ + b₁ = [1.95+0.1, -0.80-0.1, -0.20+0.2, 1.95+0.0]
h = [2.05, -0.90, 0.00, 1.95]
```

### Step 2 — ReLU Activation

```
h_relu = max(0, h) = [max(0, 2.05), max(0, -0.90), max(0, 0.00), max(0, 1.95)]
h_relu = [2.05, 0.00, 0.00, 1.95]

Note: 2 out of 4 neurons are zeroed out (~50% sparsity, typical for ReLU)
```

### Step 3 — Linear Transform #2: y = h_relu · W₂ + b₂

```
h_relu W₂:
  y₁ = 2.05×0.3 + 0.00×0.6 + 0.00×(-0.4) + 1.95×0.1
     = 0.615 + 0.0 + 0.0 + 0.195 = 0.810
  y₂ = 2.05×(-0.2) + 0.00×0.1 + 0.00×0.7 + 1.95×0.3
     = -0.410 + 0.0 + 0.0 + 0.585 = 0.175
  y₃ = 2.05×0.5 + 0.00×(-0.3) + 0.00×0.2 + 1.95×(-0.6)
     = 1.025 + 0.0 + 0.0 - 1.170 = -0.145

y = h_relu W₂ + b₂ = [0.810+0.0, 0.175+0.1, -0.145-0.1]
y = [0.810, 0.275, -0.245]
```

### Final Result

```
Input:  x = [1.000, -0.500,  2.000]   (d_model = 3)
Output: y = [0.810,  0.275, -0.245]   (d_model = 3)  ← same dimension!

With residual connection:
    final = x + y = [1.810, -0.225, 1.755]
```

---

## 7. Implementation

### 7.1 Standard FFN with ReLU

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class PositionwiseFFN(nn.Module):
    """Standard position-wise feed-forward network with ReLU."""

    def __init__(self, d_model: int = 512, d_ff: int = 2048,
                 dropout: float = 0.1):
        super().__init__()
        self.linear1 = nn.Linear(d_model, d_ff)
        self.linear2 = nn.Linear(d_ff, d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # x: (batch, seq_len, d_model)
        return self.linear2(self.dropout(F.relu(self.linear1(x))))


# Test
ffn = PositionwiseFFN(d_model=512, d_ff=2048)
x = torch.randn(2, 10, 512)
output = ffn(x)
print(f"Input:  {x.shape}")      # (2, 10, 512)
print(f"Output: {output.shape}")  # (2, 10, 512)
print(f"Params: {sum(p.numel() for p in ffn.parameters()):,}")  # 2,099,712
```

### 7.2 FFN with Multiple Activation Options

```python
class FlexibleFFN(nn.Module):
    """FFN supporting ReLU, GELU, and SiLU activations."""

    def __init__(self, d_model: int = 512, d_ff: int = 2048,
                 activation: str = "relu", dropout: float = 0.1):
        super().__init__()
        self.linear1 = nn.Linear(d_model, d_ff)
        self.linear2 = nn.Linear(d_ff, d_model)
        self.dropout = nn.Dropout(dropout)

        activations = {
            "relu": nn.ReLU(),
            "gelu": nn.GELU(),
            "silu": nn.SiLU(),  # same as Swish
        }
        if activation not in activations:
            raise ValueError(f"Unknown activation: {activation}")
        self.activation = activations[activation]

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.linear2(self.dropout(self.activation(self.linear1(x))))


# Compare activations
for act in ["relu", "gelu", "silu"]:
    ffn = FlexibleFFN(d_model=512, d_ff=2048, activation=act)
    out = ffn(torch.randn(1, 5, 512))
    print(f"{act:>4s} → output mean: {out.mean():.4f}, std: {out.std():.4f}")
```

### 7.3 SwiGLU FFN (LLaMA / PaLM Style)

```python
class SwiGLUFFN(nn.Module):
    """
    SwiGLU Feed-Forward Network used in LLaMA, PaLM, Mistral.
    FFN(x) = (SiLU(xW_gate) ⊙ xW_up) W_down
    
    Uses 3 weight matrices instead of 2, so d_ff is reduced
    to keep parameter count similar: d_ff ≈ (2/3) × 4 × d_model
    """

    def __init__(self, d_model: int = 4096, d_ff: int = 11008,
                 dropout: float = 0.0):
        super().__init__()
        self.w_gate = nn.Linear(d_model, d_ff, bias=False)
        self.w_up = nn.Linear(d_model, d_ff, bias=False)
        self.w_down = nn.Linear(d_ff, d_model, bias=False)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # Gate path with SiLU activation
        gate = F.silu(self.w_gate(x))
        # Up-projection (no activation)
        up = self.w_up(x)
        # Element-wise product then down-project
        return self.w_down(self.dropout(gate * up))


# Test with LLaMA-7B dimensions
swiglu = SwiGLUFFN(d_model=4096, d_ff=11008)
x = torch.randn(1, 5, 4096)
output = swiglu(x)
print(f"SwiGLU output shape: {output.shape}")        # (1, 5, 4096)
print(f"SwiGLU params: {sum(p.numel() for p in swiglu.parameters()):,}")
# 3 × 4096 × 11008 = 135,266,304 parameters
```

### 7.4 FFN as 1×1 Convolution (Equivalent View)

```python
class FFNasConv1d(nn.Module):
    """FFN as two 1×1 convolutions — mathematically equivalent to linear FFN."""

    def __init__(self, d_model: int = 512, d_ff: int = 2048):
        super().__init__()
        self.conv1 = nn.Conv1d(d_model, d_ff, kernel_size=1)
        self.conv2 = nn.Conv1d(d_ff, d_model, kernel_size=1)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = x.transpose(1, 2)            # (batch, d_model, seq_len)
        x = F.relu(self.conv1(x))         # (batch, d_ff, seq_len)
        x = self.conv2(x)                 # (batch, d_model, seq_len)
        return x.transpose(1, 2)          # (batch, seq_len, d_model)
```

---

## 8. Parameter Count Analysis

```
For Transformer Base (d_model=512, d_ff=2048, 6 layers):

Per FFN layer:
    W₁: 512 × 2048    = 1,048,576
    b₁: 2048           =     2,048
    W₂: 2048 × 512     = 1,048,576
    b₂: 512             =       512
    ─────────────────────────────
    Total per FFN:        2,099,712

Per Attention layer (h=8, d_k=64):
    W_Q: 512 × 512     =   262,144
    W_K: 512 × 512     =   262,144
    W_V: 512 × 512     =   262,144
    W_O: 512 × 512     =   262,144
    ─────────────────────────────
    Total per Attention:  1,048,576

Ratio: FFN / Attention = 2,099,712 / 1,048,576 ≈ 2:1

FFN accounts for ~2/3 of each layer's parameters!
```

---

## 9. Real-World Applications

| Model / System          | FFN Configuration                                                |
|-------------------------|------------------------------------------------------------------|
| **Original Transformer**| ReLU, d_ff = 4 × d_model = 2048                                 |
| **BERT**                | GELU activation, d_ff = 4 × 768 = 3072                          |
| **GPT-2 / GPT-3**       | GELU activation, d_ff = 4 × d_model                             |
| **LLaMA / Llama 2**     | SwiGLU, d_ff ≈ 2.67 × d_model (11008 for 4096)                  |
| **PaLM**                | SwiGLU with no bias terms                                        |
| **Mistral / Mixtral**   | SwiGLU; Mixtral uses Mixture-of-Experts (8 FFN experts per layer)|
| **Vision Transformer**  | GELU, d_ff = 4 × d_model = 3072 (ViT-Base)                      |

### Mixture of Experts (MoE)

In MoE models (Mixtral, Switch Transformer), the FFN is replaced by multiple expert FFNs with a router selecting top-k experts per token, scaling capacity without proportional compute.

---

## 10. Summary Table

| Concept                    | Details                                                        |
|----------------------------|----------------------------------------------------------------|
| **Formula**                | FFN(x) = max(0, xW₁ + b₁)W₂ + b₂                             |
| **Architecture**           | Linear → Activation → Linear (expand then compress)            |
| **Standard dimensions**    | d_model → d_ff (4×) → d_model                                 |
| **"Position-wise"**        | Same weights, applied independently to each token              |
| **Equivalent to**          | Two 1×1 convolutions                                           |
| **Parameters per layer**   | ~2 × d_model × d_ff (≈ 2/3 of layer total)                    |
| **Original activation**    | ReLU — max(0, x)                                               |
| **Modern activations**     | GELU (BERT/GPT), SwiGLU (LLaMA/PaLM), GeGLU                  |
| **Role**                   | Non-linearity + knowledge storage + feature transformation     |
| **GLU variants**           | 3 weight matrices, d_ff reduced to ≈ 2.67 × d_model           |

---

## 11. Revision Questions

1. **Write the FFN formula and explain what "position-wise" means.** Why doesn't the FFN allow tokens to interact with each other?

2. **Why is d_ff = 4 × d_model the standard ratio?** Calculate the total FFN parameter count for BERT-Base (d_model=768, d_ff=3072).

3. **Explain how the FFN can be viewed as two 1×1 convolutions.** What would the kernel_size, in_channels, and out_channels be for each convolution?

4. **Compare ReLU, GELU, and SwiGLU activations in FFNs.** Which models use each, and what are the tradeoffs?

5. **Given x = [1.0, -0.5, 2.0], W₁ (3×4), and W₂ (4×3), trace through the complete FFN computation.** Show the intermediate values after each step.

6. **Why does the FFN account for approximately 2/3 of a transformer layer's parameters?** Show the calculation for Transformer Base. How do GLU variants change this ratio?

---

[← Residual Connections](06-residual-connections.md) | [Why Positional Encoding →](../05-Positional-Encoding/01-why-positional-encoding.md)
