[← Why Positional Encoding](01-why-positional-encoding.md) | [Learned Positional Embeddings →](03-learned-positional-embeddings.md)

---

# Sinusoidal Positional Encoding

The original transformer paper "Attention Is All You Need" (Vaswani et al., 2017) introduced
sinusoidal positional encoding as a deterministic, parameter-free method to inject position
information into the model. The core insight is brilliant in its simplicity: use sine and
cosine functions of varying frequencies to create a unique "fingerprint" for each position.
Different dimensions of the encoding oscillate at different wavelengths, ranging from `2π`
to `10000 · 2π`, so that each position maps to a distinct point in the high-dimensional
encoding space. This approach requires no training, works for any sequence length, and
has an elegant mathematical property—relative positions can be expressed as linear
transformations of absolute positions.

---

## 1. The Sinusoidal Encoding Formulas

For a token at position `pos` in the sequence, the positional encoding vector
`PE(pos) ∈ ℝ^d_model` is defined dimension-by-dimension:

```
PE(pos, 2i)     = sin(pos / 10000^(2i / d_model))
PE(pos, 2i + 1) = cos(pos / 10000^(2i / d_model))
```

Where:
- `pos` = position in the sequence (0, 1, 2, ...)
- `i` = dimension index (0, 1, 2, ..., d_model/2 - 1)
- `d_model` = embedding dimension (e.g., 512)
- Even dimensions (`2i`) use **sine**
- Odd dimensions (`2i+1`) use **cosine**

### Equivalent Formulation with Wavelength

Each dimension pair `(2i, 2i+1)` operates at a specific **wavelength**:

```
λᵢ = 2π · 10000^(2i / d_model)
```

```
┌──────────────────────────────────────────────────────────┐
│          WAVELENGTH ACROSS DIMENSIONS                    │
│                                                          │
│  Dimension 0,1:  λ = 2π          (shortest wavelength)  │
│  Dimension 2,3:  λ = 2π · 10000^(2/512) ≈ 2π · 1.036   │
│  Dimension 4,5:  λ = 2π · 10000^(4/512) ≈ 2π · 1.073   │
│     ...                                                  │
│  Dimension 510,511: λ = 2π · 10000  (longest wavelength) │
│                                                          │
│  Low dims  ──────────────────────── High dims            │
│  Fast oscillation                   Slow oscillation     │
│  Fine position detail               Coarse position info │
│                                                          │
│  Think of it like a clock:                               │
│  • Low dimensions  = seconds hand  (fast, fine)          │
│  • Mid dimensions  = minutes hand  (medium)              │
│  • High dimensions = hours hand    (slow, coarse)        │
└──────────────────────────────────────────────────────────┘
```

---

## 2. Why Sine and Cosine? The Relative Position Property

The most important property of sinusoidal encoding is that **relative positions can be
represented as linear transformations** of absolute positions.

For any fixed offset `k`, there exists a linear transformation matrix `M` such that:

```
PE(pos + k) = M_k · PE(pos)
```

This works because of the trigonometric addition formulas:

```
sin(pos · ω + k · ω) = sin(pos · ω) · cos(k · ω) + cos(pos · ω) · sin(k · ω)
cos(pos · ω + k · ω) = cos(pos · ω) · cos(k · ω) - sin(pos · ω) · sin(k · ω)
```

In matrix form for each dimension pair:

```
┌                          ┐   ┌                    ┐   ┌              ┐
│ PE(pos + k, 2i)          │   │ cos(kω)   sin(kω)  │   │ PE(pos, 2i)  │
│                          │ = │                     │ × │              │
│ PE(pos + k, 2i+1)        │   │ -sin(kω)  cos(kω)  │   │ PE(pos,2i+1) │
└                          ┘   └                    ┘   └              ┘
```

This is a **rotation matrix**! The encoding at position `pos + k` is obtained by rotating
the encoding at position `pos` — the model can learn to attend to relative offsets.

---

## 3. Worked Example: Computing PE for Positions 0–4, d_model = 8

With `d_model = 8`, we have `i ∈ {0, 1, 2, 3}` (4 dimension pairs).

**Step 1: Compute the divisor terms**

```
div_term(i) = 10000^(2i / 8) = 10000^(i/4)

i = 0: 10000^(0/4) = 10000^0.00  =     1.000
i = 1: 10000^(1/4) = 10000^0.25  =    10.000
i = 2: 10000^(2/4) = 10000^0.50  =   100.000
i = 3: 10000^(3/4) = 10000^0.75  =  1000.000
```

**Step 2: Compute angular frequencies**

```
ω(i) = 1 / div_term(i)

ω₀ = 1.0000    ω₁ = 0.1000    ω₂ = 0.0100    ω₃ = 0.0010
```

**Step 3: Compute PE matrix**

```
PE(pos, 2i) = sin(pos × ωᵢ),  PE(pos, 2i+1) = cos(pos × ωᵢ)
```

```
         dim 0    dim 1    dim 2    dim 3    dim 4    dim 5    dim 6    dim 7
         sin(ω₀)  cos(ω₀)  sin(ω₁)  cos(ω₁)  sin(ω₂)  cos(ω₂)  sin(ω₃)  cos(ω₃)
pos 0: [ 0.0000,  1.0000,  0.0000,  1.0000,  0.0000,  1.0000,  0.0000,  1.0000]
pos 1: [ 0.8415,  0.5403,  0.0998,  0.9950,  0.0100,  1.0000,  0.0010,  1.0000]
pos 2: [ 0.9093, -0.4161,  0.1987,  0.9801,  0.0200,  0.9998,  0.0020,  1.0000]
pos 3: [ 0.1411, -0.9900,  0.2955,  0.9553,  0.0300,  0.9996,  0.0030,  1.0000]
pos 4: [-0.7568, -0.6536,  0.3894,  0.9211,  0.0400,  0.9992,  0.0040,  1.0000]
```

### Observations

- **dim 0–1** (high frequency): oscillate rapidly — change significantly each position
- **dim 6–7** (low frequency): barely change — encode coarse position over long ranges
- **Position 0**: always `[0, 1, 0, 1, 0, 1, 0, 1]` — alternating 0s and 1s
- Each row is **unique** — the combination of frequencies creates a distinct signature

---

## 4. ASCII Heatmap Visualization

```
┌──────────────────────────────────────────────────────────────┐
│     SINUSOIDAL PE HEATMAP (d_model=16, positions 0-15)      │
│     + = positive, - = negative, · = near zero               │
│                                                              │
│     Dimension →                                              │
│  P  0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15         │
│  o  ─────────────────────────────────────────────            │
│  s                                                           │
│  0: ·  +  ·  +  ·  +  ·  +  ·  +  ·  +  ·  +  ·  +         │
│  1: +  +  +  +  ·  +  ·  +  ·  +  ·  +  ·  +  ·  +         │
│  2: +  -  +  +  ·  +  ·  +  ·  +  ·  +  ·  +  ·  +         │
│  3: +  -  +  +  ·  +  ·  +  ·  +  ·  +  ·  +  ·  +         │
│  4: -  -  +  +  ·  +  ·  +  ·  +  ·  +  ·  +  ·  +         │
│  5: -  +  +  +  +  +  ·  +  ·  +  ·  +  ·  +  ·  +         │
│  6: -  +  +  -  +  +  ·  +  ·  +  ·  +  ·  +  ·  +         │
│  7: +  +  +  -  +  +  ·  +  ·  +  ·  +  ·  +  ·  +         │
│  8: +  -  +  -  +  +  ·  +  ·  +  ·  +  ·  +  ·  +         │
│  9: +  -  +  -  +  +  +  +  ·  +  ·  +  ·  +  ·  +         │
│ 10: -  -  -  -  +  +  +  +  ·  +  ·  +  ·  +  ·  +         │
│ 11: -  +  -  -  +  -  +  +  ·  +  ·  +  ·  +  ·  +         │
│ 12: -  +  -  +  +  -  +  +  ·  +  ·  +  ·  +  ·  +         │
│ 13: +  +  -  +  +  -  +  +  ·  +  ·  +  ·  +  ·  +         │
│ 14: +  -  -  +  +  -  +  +  +  +  ·  +  ·  +  ·  +         │
│ 15: +  -  +  +  +  -  +  +  +  +  ·  +  ·  +  ·  +         │
│                                                              │
│  ◄── Fast oscillation              Slow oscillation ──►      │
│      (fine position)                (coarse position)         │
└──────────────────────────────────────────────────────────────┘
```

Notice how the leftmost dimensions (low `i`) flip sign frequently (high frequency),
while the rightmost dimensions (high `i`) stay positive across many positions (low
frequency). This multi-scale representation is analogous to binary counting — each
"bit" flips at a different rate.

---

## 5. Full Implementation with Visualization

```python
import torch
import torch.nn as nn
import math
import numpy as np

class SinusoidalPositionalEncoding(nn.Module):
    """
    Sinusoidal positional encoding from 'Attention Is All You Need'.
    
    This module generates fixed (non-learnable) positional encodings
    using sine and cosine functions at different frequencies.
    """
    
    def __init__(self, d_model: int, max_len: int = 5000, dropout: float = 0.1):
        super().__init__()
        self.dropout = nn.Dropout(p=dropout)
        
        # Create positional encoding matrix: (max_len, d_model)
        pe = torch.zeros(max_len, d_model)
        
        # Position indices: (max_len, 1)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        
        # Divisor term: computes 10000^(2i/d_model) via log-space for stability
        # log(10000^(2i/d_model)) = (2i/d_model) * log(10000)
        div_term = torch.exp(
            torch.arange(0, d_model, 2).float() * -(math.log(10000.0) / d_model)
        )
        
        # Apply sin to even indices, cos to odd indices
        pe[:, 0::2] = torch.sin(position * div_term)  # dim 0, 2, 4, ...
        pe[:, 1::2] = torch.cos(position * div_term)  # dim 1, 3, 5, ...
        
        # Register as buffer (not a parameter, but saved in state_dict)
        # Shape: (1, max_len, d_model) for broadcasting with batch dimension
        pe = pe.unsqueeze(0)
        self.register_buffer('pe', pe)
    
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        """
        Args:
            x: Token embeddings of shape (batch_size, seq_len, d_model)
        Returns:
            Position-encoded embeddings of same shape
        """
        x = x + self.pe[:, :x.size(1), :]
        return self.dropout(x)


# ──────────────────────────────────────────────────
# Usage Example
# ──────────────────────────────────────────────────

d_model = 64
max_len = 100
seq_len = 10
batch_size = 2

# Create the PE module
pe_module = SinusoidalPositionalEncoding(d_model=d_model, max_len=max_len, dropout=0.0)

# Simulate token embeddings
token_embeddings = torch.randn(batch_size, seq_len, d_model)

# Add positional encoding
encoded = pe_module(token_embeddings)

print(f"Input shape:  {token_embeddings.shape}")  # (2, 10, 64)
print(f"Output shape: {encoded.shape}")            # (2, 10, 64)

# ──────────────────────────────────────────────────
# Verify the worked example from Section 3
# ──────────────────────────────────────────────────

pe_small = SinusoidalPositionalEncoding(d_model=8, max_len=5, dropout=0.0)
pe_matrix = pe_small.pe[0].numpy()

print("\nPE matrix (positions 0-4, d_model=8):")
print("        dim0    dim1    dim2    dim3    dim4    dim5    dim6    dim7")
for pos in range(5):
    values = "  ".join(f"{v:7.4f}" for v in pe_matrix[pos])
    print(f"pos {pos}: {values}")

# ──────────────────────────────────────────────────
# Verify relative position property
# ──────────────────────────────────────────────────

print("\n--- Verifying PE(pos+k) = M_k · PE(pos) ---")
pe_check = SinusoidalPositionalEncoding(d_model=4, max_len=20, dropout=0.0)
pe_vals = pe_check.pe[0]  # (20, 4)

pos, k = 3, 5
omega_0 = 1.0 / (10000 ** (0.0 / 4))  # frequency for i=0
omega_1 = 1.0 / (10000 ** (2.0 / 4))  # frequency for i=1

# Rotation matrix for dim pair (0, 1)
cos_k = math.cos(k * omega_0)
sin_k = math.sin(k * omega_0)
M = torch.tensor([[cos_k, sin_k], [-sin_k, cos_k]])

pe_pos = pe_vals[pos, :2]
pe_pos_k = pe_vals[pos + k, :2]
pe_rotated = M @ pe_pos

print(f"PE(pos={pos+k}) actual:  {pe_pos_k.numpy().round(4)}")
print(f"M_k · PE(pos={pos}):     {pe_rotated.numpy().round(4)}")
print(f"Match: {torch.allclose(pe_pos_k, pe_rotated, atol=1e-5)}")

# ──────────────────────────────────────────────────
# Distance between position encodings
# ──────────────────────────────────────────────────

print("\n--- Euclidean distances between position encodings ---")
pe_dist = SinusoidalPositionalEncoding(d_model=64, max_len=50, dropout=0.0)
pe_vecs = pe_dist.pe[0]  # (50, 64)

for ref_pos in [0, 10, 20]:
    dists = []
    for p in range(50):
        d = torch.norm(pe_vecs[p] - pe_vecs[ref_pos]).item()
        dists.append(d)
    nearest = sorted(range(50), key=lambda x: dists[x])[1:4]
    print(f"Nearest to pos {ref_pos}: positions {nearest} "
          f"(distances: {[round(dists[n], 2) for n in nearest]})")
```

---

## 6. Key Properties and Analysis

### 6.1 Uniqueness

Every position gets a unique encoding because the combination of sine and cosine values
across all dimensions at different frequencies creates a unique "fingerprint." This is
analogous to how every number has a unique binary representation.

### 6.2 Bounded Values

All values lie in `[-1, 1]` since sine and cosine are bounded functions. This prevents
magnitude issues when added to token embeddings (which are typically scaled by `√d_model`).

### 6.3 Dot Product Encodes Distance

```
PE(pos₁) · PE(pos₂) = Σᵢ [sin(pos₁·ωᵢ)·sin(pos₂·ωᵢ) + cos(pos₁·ωᵢ)·cos(pos₂·ωᵢ)]
                     = Σᵢ cos((pos₁ - pos₂)·ωᵢ)
```

The dot product depends only on the **difference** `(pos₁ - pos₂)`, which means the
model can learn to use this signal for relative position reasoning.

---

## 7. Real-World Applications

| Application                   | How Sinusoidal PE Helps                                         |
|-------------------------------|-----------------------------------------------------------------|
| **Original Transformer**       | Used in both encoder and decoder for NMT                       |
| **Long Document Processing**   | Generalizes beyond training length (no fixed max)              |
| **Cross-lingual Transfer**     | Language-agnostic position signal aids multilingual models      |
| **Audio Processing**           | Natural fit for waveform-like data with periodic patterns      |
| **Scientific Computing**       | Fixed encoding avoids overfitting on small datasets             |
| **Edge Deployment**            | No learned parameters → smaller model size                     |

---

## 8. Summary Table

| Aspect                    | Detail                                                          |
|--------------------------|-----------------------------------------------------------------|
| **Introduced In**         | "Attention Is All You Need" (Vaswani et al., 2017)             |
| **Formula (even dims)**   | `PE(pos, 2i) = sin(pos / 10000^(2i/d_model))`                 |
| **Formula (odd dims)**    | `PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))`               |
| **Learnable?**            | No — fixed, deterministic, parameter-free                      |
| **Value Range**           | `[-1, 1]` (bounded by sin/cos)                                 |
| **Key Property**          | PE(pos+k) is a linear transformation of PE(pos)                |
| **Extrapolation**         | Can generalize to sequence lengths beyond training              |
| **Wavelength Range**      | `2π` (dim 0) to `2π × 10000` (dim d_model - 1)                |
| **Space Complexity**      | O(max_len × d_model), stored as buffer                         |
| **Modern Usage**          | Largely replaced by learned/relative/rotary in LLMs            |

---

## 9. Revision Questions

1. **Write out the sinusoidal positional encoding formulas and explain what each variable
   represents. Why are both sine and cosine used?**

2. **Compute PE(pos=3) for dimensions 0 and 1 when `d_model = 8`. Show your work step
   by step.**

3. **Explain the relative position property: how can PE(pos + k) be expressed as a function
   of PE(pos)? Why is this useful for the self-attention mechanism?**

4. **Why does sinusoidal PE use frequencies that span a geometric progression from `1` to
   `1/10000`? What would happen if all dimensions used the same frequency?**

5. **What are the advantages and disadvantages of sinusoidal PE compared to learned
   positional embeddings? When might you prefer one over the other?**

6. **The dot product `PE(pos₁) · PE(pos₂)` depends only on the difference `pos₁ - pos₂`.
   Derive this result using the cosine addition formula and explain its significance.**

---

[← Why Positional Encoding](01-why-positional-encoding.md) | [Learned Positional Embeddings →](03-learned-positional-embeddings.md)
