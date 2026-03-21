[← Relative Positional Encoding](04-relative-positional-encoding.md) | [Masked Self-Attention →](../06-Transformer-Training/01-masked-self-attention.md)

---

# Rotary Positional Encoding (RoPE)

Rotary Positional Encoding (RoPE), introduced by Su et al. (2021) in the paper "RoFormer:
Enhanced Transformer with Rotary Position Embedding," has become the dominant positional
encoding method in modern large language models. Used in LLaMA, LLaMA 2, GPT-NeoX, PaLM,
Falcon, Mistral, and most state-of-the-art LLMs, RoPE elegantly encodes position by
**rotating** query and key vectors in the attention mechanism. Its key mathematical insight
is that when both query and key are rotated by angles proportional to their absolute
positions, their dot product naturally depends only on the **relative** distance between
them—achieving relative position encoding through absolute position rotations, with zero
additional parameters.

---

## 1. Core Idea: Position as Rotation

### The Geometric Intuition

RoPE encodes position by rotating vectors in 2D subspaces. Consider a 2D vector being
rotated by an angle `θ`:

```
┌────────────────────────────────────────────────────────────┐
│          ROTATION IN 2D                                     │
│                                                             │
│          y                                                  │
│          ▲        ╱ rotated vector                          │
│          │      ╱   (x', y')                                │
│          │    ╱                                              │
│          │  ╱  θ = position × base_angle                    │
│          │╱──────────► x                                    │
│          │    original vector                                │
│          │    (x, y)                                         │
│                                                             │
│  Rotation by angle θ:                                       │
│  x' = x·cos(θ) - y·sin(θ)                                  │
│  y' = x·sin(θ) + y·cos(θ)                                  │
│                                                             │
│  Key property: rotation preserves vector magnitude          │
│  ||rotated|| = ||original||                                 │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**The brilliant insight:** If we rotate the query vector at position `m` by angle `mθ`
and the key vector at position `n` by angle `nθ`, then their dot product contains the
factor `cos((m-n)θ)` — it depends only on the **relative position** `(m - n)`!

---

## 2. Mathematical Formulation

### 2.1 Rotation in 2D

For a single pair of dimensions, the rotation matrix for position `m` is:

```
R(mθ) = ┌                    ┐
         │ cos(mθ)   -sin(mθ) │
         │ sin(mθ)    cos(mθ) │
         └                    ┘
```

### 2.2 Extension to d-Dimensional Space

For a `d`-dimensional vector (where `d` is even), RoPE partitions it into `d/2` pairs
and applies independent rotations with different frequencies to each pair:

```
θᵢ = 1 / 10000^(2i / d)     for i = 0, 1, ..., d/2 - 1
```

The full rotation matrix for position `m`:

```
R(m) = ┌                                                              ┐
        │ cos(mθ₀)  -sin(mθ₀)    0         0       ...    0        0  │
        │ sin(mθ₀)   cos(mθ₀)    0         0       ...    0        0  │
        │    0           0     cos(mθ₁)  -sin(mθ₁)  ...    0        0  │
        │    0           0     sin(mθ₁)   cos(mθ₁)  ...    0        0  │
        │   ...         ...      ...        ...     ...   ...      ... │
        │    0           0        0         0       ... cos(mθ_{d/2-1}) -sin(mθ_{d/2-1}) │
        │    0           0        0         0       ... sin(mθ_{d/2-1})  cos(mθ_{d/2-1}) │
        └                                                              ┘
```

This is a block-diagonal matrix of 2×2 rotation matrices.

### 2.3 Applying RoPE to Query and Key

```
q̃_m = R(m) · q_m        (rotate query at position m)
k̃_n = R(n) · k_n        (rotate key at position n)
```

The attention score between positions `m` and `n`:

```
q̃_m · k̃_n^T = (R(m) · q_m)^T · (R(n) · k_n)
             = q_m^T · R(m)^T · R(n) · k_n
             = q_m^T · R(m-n)^T · k_n      ← depends only on (n - m)!
             = q_m^T · R(n-m) · k_n
```

This uses the property that `R(m)^T · R(n) = R(n-m)` (rotation matrices compose).

---

## 3. The Relative Position Proof

### Why Does the Dot Product Depend Only on Relative Position?

For a single 2D pair with angle `θ`:

```
q̃_m = [q₁·cos(mθ) - q₂·sin(mθ),  q₁·sin(mθ) + q₂·cos(mθ)]
k̃_n = [k₁·cos(nθ) - k₂·sin(nθ),  k₁·sin(nθ) + k₂·cos(nθ)]

q̃_m · k̃_n = [q₁·cos(mθ) - q₂·sin(mθ)] · [k₁·cos(nθ) - k₂·sin(nθ)]
            + [q₁·sin(mθ) + q₂·cos(mθ)] · [k₁·sin(nθ) + k₂·cos(nθ)]

Expanding:
= q₁k₁·cos(mθ)cos(nθ) - q₁k₂·cos(mθ)sin(nθ) - q₂k₁·sin(mθ)cos(nθ) + q₂k₂·sin(mθ)sin(nθ)
+ q₁k₁·sin(mθ)sin(nθ) + q₁k₂·sin(mθ)cos(nθ) + q₂k₁·cos(mθ)sin(nθ) + q₂k₂·cos(mθ)cos(nθ)

Simplifying using cos(A-B) = cosA·cosB + sinA·sinB:
= q₁k₁·[cos(mθ)cos(nθ) + sin(mθ)sin(nθ)]  +  q₂k₂·[sin(mθ)sin(nθ) + cos(mθ)cos(nθ)]
+ q₁k₂·[sin(mθ)cos(nθ) - cos(mθ)sin(nθ)]  +  q₂k₁·[cos(mθ)sin(nθ) - sin(mθ)cos(nθ)]

= q₁k₁·cos((m-n)θ)  +  q₂k₂·cos((m-n)θ)
- q₁k₂·sin((m-n)θ)  +  q₂k₁·sin((m-n)θ)

= (q₁k₁ + q₂k₂)·cos((m-n)θ)  +  (q₂k₁ - q₁k₂)·sin((m-n)θ)

✓ Only depends on (m - n), NOT on absolute positions m or n!
```

---

## 4. Efficient Implementation (No Explicit Matrix Multiplication)

Instead of constructing the full rotation matrix, RoPE can be implemented efficiently
using element-wise operations:

```
┌────────────────────────────────────────────────────────────────┐
│         EFFICIENT RoPE COMPUTATION                             │
│                                                                │
│  Input vector q = [q₀, q₁, q₂, q₃, q₄, q₅, q₆, q₇]         │
│                                                                │
│  Step 1: Split into pairs                                      │
│    pairs: (q₀,q₁), (q₂,q₃), (q₄,q₅), (q₆,q₇)               │
│                                                                │
│  Step 2: Compute rotation angles for position m                │
│    angles: [mθ₀, mθ₁, mθ₂, mθ₃]                              │
│                                                                │
│  Step 3: Apply rotation to each pair                           │
│    q₀' = q₀·cos(mθ₀) - q₁·sin(mθ₀)                          │
│    q₁' = q₀·sin(mθ₀) + q₁·cos(mθ₀)                          │
│    q₂' = q₂·cos(mθ₁) - q₃·sin(mθ₁)                          │
│    q₃' = q₂·sin(mθ₁) + q₃·cos(mθ₁)                          │
│    ...                                                         │
│                                                                │
│  Vectorized:                                                   │
│    q_rotated = q * cos_cache - rotate_half(q) * sin_cache      │
│                                                                │
│  Where rotate_half([q₀,q₁,q₂,q₃,...]) = [-q₁,q₀,-q₃,q₂,...] │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 5. Worked Example: RoPE with d=4, positions 0–3

```
Setup:
  d = 4 (so 2 dimension pairs)
  θ₀ = 1 / 10000^(0/4) = 1.0
  θ₁ = 1 / 10000^(2/4) = 1/100 = 0.01

Query at position m=2: q = [1.0, 0.5, 0.8, 0.3]
Key at position n=5:   k = [0.6, 0.9, 0.4, 0.7]

Step 1: Compute angles
  For q (m=2): angles = [2×1.0, 2×0.01] = [2.0, 0.02]
  For k (n=5): angles = [5×1.0, 5×0.01] = [5.0, 0.05]

Step 2: Apply rotation to q at position 2
  Pair 0 (q₀, q₁) = (1.0, 0.5), angle = 2.0 rad
    q₀' = 1.0·cos(2.0) - 0.5·sin(2.0) = 1.0·(-0.4161) - 0.5·(0.9093) = -0.8708
    q₁' = 1.0·sin(2.0) + 0.5·cos(2.0) = 1.0·(0.9093) + 0.5·(-0.4161) = 0.7012
  
  Pair 1 (q₂, q₃) = (0.8, 0.3), angle = 0.02 rad
    q₂' = 0.8·cos(0.02) - 0.3·sin(0.02) = 0.8·(0.9998) - 0.3·(0.0200) = 0.7938
    q₃' = 0.8·sin(0.02) + 0.3·cos(0.02) = 0.8·(0.0200) + 0.3·(0.9998) = 0.3159
  
  q̃₂ = [-0.8708, 0.7012, 0.7938, 0.3159]

Step 3: Apply rotation to k at position 5
  Pair 0 (k₀, k₁) = (0.6, 0.9), angle = 5.0 rad
    k₀' = 0.6·cos(5.0) - 0.9·sin(5.0) = 0.6·(0.2837) - 0.9·(-0.9589) = 1.0332
    k₁' = 0.6·sin(5.0) + 0.9·cos(5.0) = 0.6·(-0.9589) + 0.9·(0.2837) = -0.3200
  
  Pair 1 (k₂, k₃) = (0.4, 0.7), angle = 0.05 rad
    k₂' = 0.4·cos(0.05) - 0.7·sin(0.05) = 0.4·(0.9988) - 0.7·(0.0500) = 0.3645
    k₃' = 0.4·sin(0.05) + 0.7·cos(0.05) = 0.4·(0.0500) + 0.7·(0.9988) = 0.7192
  
  k̃₅ = [1.0332, -0.3200, 0.3645, 0.7192]

Step 4: Compute attention score
  q̃₂ · k̃₅ = (-0.8708)(1.0332) + (0.7012)(-0.3200) + (0.7938)(0.3645) + (0.3159)(0.7192)
           = -0.8997 + (-0.2244) + 0.2893 + 0.2272
           = -0.6076

  This score encodes the relative distance (5 - 2) = 3, not the absolute positions!
```

---

## 6. PyTorch Implementation

```python
import torch
import torch.nn as nn
import math

class RotaryPositionalEncoding(nn.Module):
    """
    Rotary Positional Encoding (RoPE) as used in LLaMA, GPT-NeoX, etc.
    Encodes position by rotating query and key vectors.
    """
    
    def __init__(self, d_model: int, max_len: int = 4096, base: float = 10000.0):
        super().__init__()
        self.d_model = d_model
        self.max_len = max_len
        
        # Compute rotation frequencies: θ_i = 1 / base^(2i/d)
        inv_freq = 1.0 / (base ** (torch.arange(0, d_model, 2).float() / d_model))
        self.register_buffer('inv_freq', inv_freq)
        
        # Precompute sin/cos cache
        self._build_cache(max_len)
    
    def _build_cache(self, seq_len: int):
        """Precompute cos and sin for all positions."""
        positions = torch.arange(seq_len, dtype=torch.float)
        # Outer product: (seq_len, d_model/2)
        angles = torch.outer(positions, self.inv_freq)
        # Duplicate for pairs: (seq_len, d_model)
        angles = torch.cat([angles, angles], dim=-1)
        
        self.register_buffer('cos_cache', angles.cos())
        self.register_buffer('sin_cache', angles.sin())
    
    @staticmethod
    def _rotate_half(x: torch.Tensor) -> torch.Tensor:
        """
        Rotate pairs: [x0, x1, x2, x3, ...] → [-x1, x0, -x3, x2, ...]
        Equivalent to multiplying by the rotation matrix's off-diagonal part.
        """
        d_half = x.shape[-1] // 2
        x1 = x[..., :d_half]
        x2 = x[..., d_half:]
        return torch.cat([-x2, x1], dim=-1)
    
    def forward(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        position_ids: torch.Tensor = None
    ) -> tuple:
        """
        Apply RoPE to query and key tensors.
        
        Args:
            q: Query tensor, shape (batch, num_heads, seq_len, head_dim)
            k: Key tensor, shape (batch, num_heads, seq_len, head_dim)
            position_ids: Optional position indices, shape (batch, seq_len)
        
        Returns:
            Tuple of (rotated_q, rotated_k) with same shapes
        """
        seq_len = q.size(2)
        
        if position_ids is None:
            cos = self.cos_cache[:seq_len].unsqueeze(0).unsqueeze(0)
            sin = self.sin_cache[:seq_len].unsqueeze(0).unsqueeze(0)
        else:
            cos = self.cos_cache[position_ids].unsqueeze(1)
            sin = self.sin_cache[position_ids].unsqueeze(1)
        
        # Apply rotation: x_rotated = x * cos + rotate_half(x) * sin
        q_rotated = q * cos + self._rotate_half(q) * sin
        k_rotated = k * cos + self._rotate_half(k) * sin
        
        return q_rotated, k_rotated


class RoPEMultiHeadAttention(nn.Module):
    """Multi-head attention with RoPE, similar to LLaMA's implementation."""
    
    def __init__(self, d_model: int, num_heads: int, max_len: int = 4096):
        super().__init__()
        self.d_model = d_model
        self.num_heads = num_heads
        self.head_dim = d_model // num_heads
        
        self.W_Q = nn.Linear(d_model, d_model, bias=False)
        self.W_K = nn.Linear(d_model, d_model, bias=False)
        self.W_V = nn.Linear(d_model, d_model, bias=False)
        self.W_O = nn.Linear(d_model, d_model, bias=False)
        
        self.rope = RotaryPositionalEncoding(self.head_dim, max_len)
    
    def forward(self, x: torch.Tensor, mask: torch.Tensor = None) -> torch.Tensor:
        B, N, _ = x.shape
        
        Q = self.W_Q(x).view(B, N, self.num_heads, self.head_dim).transpose(1, 2)
        K = self.W_K(x).view(B, N, self.num_heads, self.head_dim).transpose(1, 2)
        V = self.W_V(x).view(B, N, self.num_heads, self.head_dim).transpose(1, 2)
        
        # Apply RoPE to Q and K (NOT to V!)
        Q, K = self.rope(Q, K)
        
        # Standard scaled dot-product attention
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.head_dim)
        
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float('-inf'))
        
        attn_weights = torch.softmax(scores, dim=-1)
        output = torch.matmul(attn_weights, V)
        
        output = output.transpose(1, 2).contiguous().view(B, N, self.d_model)
        return self.W_O(output)


# ──────────────────────────────────────────────────
# Usage and Verification
# ──────────────────────────────────────────────────

torch.manual_seed(42)

d_model = 64
num_heads = 4
head_dim = d_model // num_heads  # 16

# Create model
model = RoPEMultiHeadAttention(d_model, num_heads)

# Forward pass
x = torch.randn(2, 10, d_model)
output = model(x)
print(f"Input:  {x.shape}")      # (2, 10, 64)
print(f"Output: {output.shape}")  # (2, 10, 64)

# Verify relative position property
rope = RotaryPositionalEncoding(head_dim, max_len=100)
q = torch.randn(1, 1, 1, head_dim)  # Single query vector
k = torch.randn(1, 1, 1, head_dim)  # Single key vector

# Test: dot(q@pos_m, k@pos_n) should equal dot(q@pos_m+5, k@pos_n+5)
for m, n in [(2, 7), (10, 15), (50, 55)]:
    q_at_m = q * rope.cos_cache[m] + rope._rotate_half(q) * rope.sin_cache[m]
    k_at_n = k * rope.cos_cache[n] + rope._rotate_half(k) * rope.sin_cache[n]
    score = (q_at_m * k_at_n).sum().item()
    print(f"  dot(q@{m}, k@{n}) [rel_dist={n-m}] = {score:.6f}")
# All three scores should be identical (same relative distance of 5)
```

---

## 7. RoPE vs. Other Methods

```
┌──────────────────────────────────────────────────────────────────────┐
│              POSITIONAL ENCODING COMPARISON                          │
│                                                                      │
│  Property          │ Sinusoidal │ Learned │ Relative │    RoPE       │
│  ──────────────────┼────────────┼─────────┼──────────┼───────────    │
│  Extra parameters  │    None    │  L × d  │  varies  │    None       │
│  Position type     │  Absolute  │Absolute │ Relative │ Abs→Relative  │
│  Applied to        │  Input emb │Input emb│  Attn    │    Q, K       │
│  Rel. pos. in dot  │  Indirect  │   No    │  Direct  │   Direct      │
│  Extrapolation     │   Good     │  Poor   │  Good    │  Moderate     │
│  Implementation    │  Simple    │ Simple  │ Complex  │  Moderate     │
│  Modern LLM usage  │   Rare     │  Rare   │  Some    │  Dominant     │
│  Key models        │  Original  │BERT,GPT2│  T5, XL  │ LLaMA,Mistral│
│                    │ Transformer│         │          │ GPT-NeoX,PaLM │
│                                                                      │
│  Why RoPE wins:                                                      │
│  ✓ No extra parameters (unlike learned)                              │
│  ✓ Relative position naturally encoded (unlike sinusoidal)           │
│  ✓ Simple to implement (unlike Transformer-XL)                       │
│  ✓ Preserves vector norms (rotation is orthogonal)                   │
│  ✓ Compatible with linear attention and KV-cache                     │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 8. Long Context Extensions of RoPE

Modern LLMs extend RoPE for longer contexts via several techniques:

```
┌─────────────────────────────────────────────────────────────────┐
│              RoPE CONTEXT EXTENSION METHODS                     │
│                                                                 │
│  1. Position Interpolation (PI) — Chen et al., 2023             │
│     Instead of: θ_m = m × θ_base                                │
│     Use:        θ_m = (m × L_train / L_target) × θ_base        │
│     Compress positions to fit the trained range                  │
│                                                                 │
│  2. NTK-Aware Scaling                                           │
│     Modify the base: base' = base × α^(d/(d-2))                │
│     Spreads frequency adjustments across all dimensions          │
│                                                                 │
│  3. YaRN (Yet another RoPE extensioN) — 2023                   │
│     Combines NTK-scaling with attention temperature scaling      │
│     Different treatment for low vs. high frequency dimensions    │
│                                                                 │
│  4. Dynamic NTK                                                  │
│     Adjust base dynamically based on current sequence length     │
│                                                                 │
│  Training: 4K tokens  →  Extended: 32K, 100K, or even 1M!      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# Position Interpolation example
def rope_with_position_interpolation(
    d_model: int,
    max_trained_len: int,
    target_len: int,
    base: float = 10000.0
):
    """
    RoPE with linear position interpolation for extending context.
    """
    inv_freq = 1.0 / (base ** (torch.arange(0, d_model, 2).float() / d_model))
    
    # Scale positions down to fit within trained range
    scale = max_trained_len / target_len
    positions = torch.arange(target_len, dtype=torch.float) * scale
    
    angles = torch.outer(positions, inv_freq)
    cos_vals = angles.cos()
    sin_vals = angles.sin()
    
    return cos_vals, sin_vals

# Extend a 4096-context model to 16384
cos_ext, sin_ext = rope_with_position_interpolation(
    d_model=64, max_trained_len=4096, target_len=16384
)
print(f"Extended cos cache shape: {cos_ext.shape}")  # (16384, 32)
```

---

## 9. Real-World Applications

| Application                        | Model & RoPE Details                                       |
|------------------------------------|------------------------------------------------------------|
| **Chat / Instruction Following**   | LLaMA 2, Mistral — RoPE with 4K-32K context              |
| **Long Document QA**               | LongLLaMA — RoPE + PI for 256K context                    |
| **Code Generation**                | CodeLLaMA — RoPE extended to 100K tokens for full repos   |
| **Multilingual LLMs**              | BLOOM, Falcon — RoPE across 46+ languages                 |
| **Retrieval-Augmented Generation** | RoPE enables long contexts for retrieved documents         |
| **Scientific Literature**          | LLMs with RoPE process full papers (10K+ tokens)          |

---

## 10. Summary Table

| Aspect                    | Detail                                                          |
|--------------------------|-----------------------------------------------------------------|
| **Paper**                 | Su et al. (2021) — "RoFormer"                                  |
| **Core Mechanism**        | Rotate Q, K vectors by position-dependent angles               |
| **Frequency Formula**     | `θᵢ = 1 / 10000^(2i/d)`                                       |
| **Rotation Formula**      | `x' = x · cos(mθ) - rotate_half(x) · sin(mθ)`                |
| **Key Property**          | `dot(R(m)q, R(n)k)` depends only on `(m-n)`                   |
| **Extra Parameters**      | Zero — purely geometric transformation                         |
| **Applied To**            | Query and Key vectors (NOT Values)                             |
| **Norm Preservation**     | Yes — rotation is an orthogonal transformation                 |
| **Extrapolation**         | Moderate; improved with PI, NTK, YaRN extensions               |
| **Key Models**            | LLaMA, LLaMA 2, Mistral, GPT-NeoX, PaLM, Falcon              |
| **Why Dominant**          | Simple, parameter-free, effective, compatible with KV-cache    |

---

## 11. Revision Questions

1. **Explain the core idea of RoPE in one sentence. Why is it called "rotary"?**

2. **Derive the dot product of two RoPE-rotated vectors in 2D and show that it depends
   only on the relative position. What trigonometric identity is used?**

3. **What is the `rotate_half` operation and why is it needed for efficient implementation?
   How does it relate to the rotation matrix?**

4. **RoPE applies rotations to Q and K but NOT to V. Why? What would happen if V were
   also rotated?**

5. **Explain Position Interpolation (PI) for extending RoPE to longer contexts. Why does
   simple extrapolation (using positions beyond training range) degrade performance?**

6. **Compare RoPE with sinusoidal positional encoding. Both use sin/cos functions — what
   is the fundamental difference in how they incorporate position information?**

---

[← Relative Positional Encoding](04-relative-positional-encoding.md) | [Masked Self-Attention →](../06-Transformer-Training/01-masked-self-attention.md)
