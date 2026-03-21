[← Learned Positional Embeddings](03-learned-positional-embeddings.md) | [Rotary Positional Encoding →](05-rotary-positional-encoding.md)

---

# Relative Positional Encoding

Absolute positional encodings (sinusoidal or learned) assign a fixed vector to each position
in the sequence, regardless of context. **Relative positional encoding** takes a fundamentally
different approach: instead of asking "What position is this token at?", it asks "How far
apart are these two tokens?" This shift in perspective aligns more naturally with how language
works—the relationship between a verb and its subject depends on their **distance**, not their
absolute positions. Introduced by Shaw et al. (2018) and refined in Transformer-XL and T5,
relative positional encoding modifies the attention computation itself to incorporate
distance-based bias, enabling better generalization to sequences of varying lengths and
improved handling of long-range dependencies.

---

## 1. Motivation: Why Relative over Absolute?

### The Problem with Absolute Positions

```
┌───────────────────────────────────────────────────────────────┐
│        ABSOLUTE vs. RELATIVE POSITION                         │
│                                                               │
│  Sentence A: "The cat sat on the mat"                         │
│  Positions:    0    1   2   3   4   5                         │
│  "cat" is at position 1, "sat" is at position 2              │
│  Distance: |2 - 1| = 1                                        │
│                                                               │
│  Sentence B: "Yesterday, the cat sat on the mat"              │
│  Positions:      0       1   2   3   4   5   6   7            │
│  "cat" is at position 3, "sat" is at position 4              │
│  Distance: |4 - 3| = 1                                        │
│                                                               │
│  With absolute PE:                                            │
│    "cat" at pos 1 ≠ "cat" at pos 3  → different behavior!    │
│                                                               │
│  With relative PE:                                            │
│    distance("cat", "sat") = 1 in BOTH cases  → same behavior!│
│                                                               │
│  ✓ Relative PE captures the structural relationship           │
│  ✓ Independent of where the pattern appears in the sequence   │
└───────────────────────────────────────────────────────────────┘
```

---

## 2. Shaw et al. (2018): Relative Position Representations

### 2.1 Core Idea

Shaw et al. proposed adding **learnable relative position embeddings** directly into the
attention mechanism. For each pair of positions `(i, j)`, the relative distance `r = j - i`
(clipped to a range `[-k, k]`) determines the positional contribution.

Two sets of relative position embeddings are introduced:

```
a_ij^K ∈ ℝ^d_k   — relative position embedding for Keys
a_ij^V ∈ ℝ^d_v   — relative position embedding for Values
```

Where `a_ij` depends only on the relative distance `clip(j - i, -k, k)`.

### 2.2 Modified Attention Equations

**Standard attention:**
```
e_ij = (x_i · W^Q) · (x_j · W^K)^T / √d_k
```

**With relative position (Shaw et al.):**
```
e_ij = (x_i · W^Q) · (x_j · W^K + a_ij^K)^T / √d_k
                                    ▲
                                    │
                          relative position bias
                          added to the Key
```

**Output computation also modified:**
```
Standard:   z_i = Σ_j  α_ij · (x_j · W^V)
Shaw:       z_i = Σ_j  α_ij · (x_j · W^V + a_ij^V)
                                            ▲
                                 relative position bias
                                 added to the Value
```

### 2.3 Distance Clipping

```
clip(x, -k, k) = max(-k, min(k, x))

For k = 4:
  Actual distance:   -8  -5  -4  -3  -2  -1   0   1   2   3   4   5   8
  Clipped distance:  -4  -4  -4  -3  -2  -1   0   1   2   3   4   4   4
  Embedding index:    0   0   0   1   2   3   4   5   6   7   8   8   8
  
  Total unique embeddings: 2k + 1 = 9
```

The intuition: beyond a certain distance, the precise offset matters less—"far away" is
"far away" whether it's 10 or 100 tokens back.

---

## 3. Worked Example

```
Sequence: ["I", "love", "deep", "learning"]   (n=4)
Relative distances (j - i):

         j=0    j=1    j=2    j=3
i=0  [   0      1      2      3  ]
i=1  [  -1      0      1      2  ]
i=2  [  -2     -1      0      1  ]
i=3  [  -3     -2     -1      0  ]

With clip range k=2, clipped distances:

         j=0    j=1    j=2    j=3
i=0  [   0      1      2      2  ]     (3 clipped to 2)
i=1  [  -1      0      1      2  ]
i=2  [  -2     -1      0      1  ]
i=3  [  -2     -2     -1      0  ]     (-3 clipped to -2)

Unique embedding indices: {-2, -1, 0, 1, 2} → 5 embeddings

Suppose d_k = 3 and the learned relative Key embeddings are:
  a[-2] = [ 0.10, -0.05,  0.08]
  a[-1] = [ 0.20,  0.15, -0.03]
  a[ 0] = [ 0.00,  0.00,  0.00]   (self-distance, often near zero)
  a[ 1] = [-0.12,  0.22,  0.07]
  a[ 2] = [-0.18,  0.30,  0.14]

Computing attention score for i=0 ("I"), j=2 ("deep"):
  Relative distance = 2 - 0 = 2
  
  Standard term:   q_0 · k_2 = [0.5, 0.3, 0.1] · [0.2, -0.4, 0.6] = 0.1 - 0.12 + 0.06 = 0.04
  Relative term:   q_0 · a[2] = [0.5, 0.3, 0.1] · [-0.18, 0.30, 0.14] = -0.09 + 0.09 + 0.014 = 0.014
  
  Combined: e_02 = (0.04 + 0.014) / √3 = 0.054 / 1.732 = 0.0312
```

---

## 4. Transformer-XL: Extending Relative Positions

Dai et al. (2019) in Transformer-XL decomposed the attention score into four terms,
making the relative position contribution more explicit:

```
Standard absolute attention score:
  e_ij = q_i · k_j = (x_i·W^Q + p_i·W^Q) · (x_j·W^K + p_j·W^K)^T

Expanding:
  e_ij = (a) x_i·W^Q · (x_j·W^K)^T        ← content-to-content
       + (b) x_i·W^Q · (p_j·W^K)^T        ← content-to-position
       + (c) p_i·W^Q · (x_j·W^K)^T        ← position-to-content (global query bias)
       + (d) p_i·W^Q · (p_j·W^K)^T        ← position-to-position

Transformer-XL replaces absolute positions with relative:
  e_ij = (a) x_i·W^Q · (x_j·W^K)^T        ← content-to-content (unchanged)
       + (b) x_i·W^Q · (R_{i-j}·W^R)^T    ← content-to-relative-position
       + (c) u   · (x_j·W^K)^T             ← global content bias
       + (d) v   · (R_{i-j}·W^R)^T         ← global position bias

Where:
  R_{i-j} = sinusoidal encoding of relative distance (i-j)
  u, v    = learned global bias vectors (replace absolute position queries)
  W^R     = separate projection for relative position
```

```
┌─────────────────────────────────────────────────────────┐
│     TRANSFORMER-XL: FOUR-TERM ATTENTION DECOMPOSITION   │
│                                                         │
│  Term (a): "What is token j about?"                     │
│            Content × Content                            │
│                                                         │
│  Term (b): "How far is token j from token i?"           │
│            Content × Relative Position                  │
│                                                         │
│  Term (c): "How important is token j in general?"       │
│            Global Bias × Content                        │
│                                                         │
│  Term (d): "How important is distance (i-j) in general?"│
│            Global Bias × Relative Position              │
│                                                         │
│  Final: e_ij = (a) + (b) + (c) + (d)                   │
│         Then: α_ij = softmax_j(e_ij / √d_k)            │
└─────────────────────────────────────────────────────────┘
```

---

## 5. T5: Simplified Relative Position Bias

The T5 model (Raffel et al., 2020) uses a much simpler approach: a **scalar bias** added
directly to the attention logits based on the relative distance.

```
e_ij = q_i · k_j / √d_k  +  b(i - j)
                              ▲
                     scalar bias from lookup table
                     indexed by relative distance
```

```
┌───────────────────────────────────────────────────────┐
│     T5 RELATIVE POSITION BIAS                         │
│                                                       │
│  Bias table (per attention head):                     │
│                                                       │
│  Distance:   0     1     2    3-4   5-8   9-16  17+   │
│  Bucket:     0     1     2     3     4     5     6    │
│  Bias:     0.2   0.1  -0.1  -0.2  -0.4  -0.6  -0.8  │
│                                                       │
│  Key insight: uses logarithmic bucketing               │
│  Nearby positions → fine-grained distinctions          │
│  Far positions → coarse grouping                      │
│                                                       │
│  Distance 1 and 2 get separate buckets                │
│  Distance 50 and 60 share the same bucket             │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### T5's Logarithmic Bucketing

```python
import torch
import math

def t5_relative_position_bucket(
    relative_position: torch.Tensor,
    bidirectional: bool = True,
    num_buckets: int = 32,
    max_distance: int = 128
) -> torch.Tensor:
    """
    Compute relative position buckets as in T5.
    Maps relative positions to a fixed number of buckets using
    linear spacing for nearby positions and log spacing for far ones.
    """
    ret = 0
    n = -relative_position
    
    if bidirectional:
        num_buckets //= 2
        ret += (n < 0).to(torch.long) * num_buckets
        n = torch.abs(n)
    else:
        n = torch.max(n, torch.zeros_like(n))
    
    # Half the buckets are for exact small distances
    max_exact = num_buckets // 2
    is_small = n < max_exact
    
    # The other half are for log-spaced larger distances
    val_if_large = max_exact + (
        torch.log(n.float() / max_exact)
        / math.log(max_distance / max_exact)
        * (num_buckets - max_exact)
    ).to(torch.long)
    val_if_large = torch.min(
        val_if_large,
        torch.full_like(val_if_large, num_buckets - 1)
    )
    
    ret += torch.where(is_small, n, val_if_large)
    return ret


# Example: compute buckets for a 6x6 attention matrix
seq_len = 6
positions = torch.arange(seq_len)
relative_pos = positions.unsqueeze(0) - positions.unsqueeze(1)  # (6, 6)

print("Relative positions:")
print(relative_pos.numpy())

buckets = t5_relative_position_bucket(relative_pos, bidirectional=True)
print("\nBucket indices:")
print(buckets.numpy())
```

---

## 6. Implementation: Shaw et al. Relative Position

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math

class RelativePositionAttention(nn.Module):
    """
    Multi-head attention with relative position representations
    following Shaw et al. (2018).
    """
    
    def __init__(self, d_model: int, num_heads: int, max_rel_dist: int = 16):
        super().__init__()
        assert d_model % num_heads == 0
        
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads
        self.max_rel_dist = max_rel_dist
        
        # Standard Q, K, V projections
        self.W_Q = nn.Linear(d_model, d_model)
        self.W_K = nn.Linear(d_model, d_model)
        self.W_V = nn.Linear(d_model, d_model)
        self.W_O = nn.Linear(d_model, d_model)
        
        # Relative position embeddings for keys and values
        num_rel_positions = 2 * max_rel_dist + 1
        self.rel_key_embed = nn.Embedding(num_rel_positions, self.d_k)
        self.rel_val_embed = nn.Embedding(num_rel_positions, self.d_k)
    
    def _get_relative_positions(self, seq_len: int) -> torch.Tensor:
        """Compute clipped relative position indices."""
        positions = torch.arange(seq_len, dtype=torch.long)
        # (seq_len, seq_len) matrix of relative distances
        rel_pos = positions.unsqueeze(0) - positions.unsqueeze(1)
        # Clip to [-max_rel_dist, max_rel_dist]
        rel_pos = rel_pos.clamp(-self.max_rel_dist, self.max_rel_dist)
        # Shift to [0, 2*max_rel_dist] for embedding lookup
        rel_pos = rel_pos + self.max_rel_dist
        return rel_pos
    
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        """
        Args:
            x: Input tensor of shape (batch_size, seq_len, d_model)
        Returns:
            Output of shape (batch_size, seq_len, d_model)
        """
        B, N, _ = x.shape
        
        # Project to Q, K, V
        Q = self.W_Q(x).view(B, N, self.num_heads, self.d_k).transpose(1, 2)
        K = self.W_K(x).view(B, N, self.num_heads, self.d_k).transpose(1, 2)
        V = self.W_V(x).view(B, N, self.num_heads, self.d_k).transpose(1, 2)
        # Q, K, V shape: (B, num_heads, N, d_k)
        
        # Standard content-to-content attention
        content_scores = torch.matmul(Q, K.transpose(-2, -1))  # (B, H, N, N)
        
        # Relative position scores
        rel_pos_indices = self._get_relative_positions(N).to(x.device)  # (N, N)
        rel_k = self.rel_key_embed(rel_pos_indices)  # (N, N, d_k)
        
        # Compute Q · rel_k^T for each query position
        # Q: (B, H, N, d_k) → reshape for matmul with rel_k
        Q_for_rel = Q.permute(2, 0, 1, 3).reshape(N, B * self.num_heads, self.d_k)
        rel_scores = torch.bmm(Q_for_rel, rel_k.transpose(-2, -1))
        rel_scores = rel_scores.view(N, B, self.num_heads, N)
        rel_scores = rel_scores.permute(1, 2, 0, 3)  # (B, H, N, N)
        
        # Combine scores
        scores = (content_scores + rel_scores) / math.sqrt(self.d_k)
        attn_weights = F.softmax(scores, dim=-1)
        
        # Apply attention to values (with relative position contribution)
        rel_v = self.rel_val_embed(rel_pos_indices)  # (N, N, d_k)
        
        content_output = torch.matmul(attn_weights, V)  # (B, H, N, d_k)
        
        # Relative value contribution
        attn_for_rel = attn_weights.permute(2, 0, 1, 3).reshape(N, B * self.num_heads, N)
        rel_output = torch.bmm(attn_for_rel, rel_v)
        rel_output = rel_output.view(N, B, self.num_heads, self.d_k)
        rel_output = rel_output.permute(1, 2, 0, 3)  # (B, H, N, d_k)
        
        output = content_output + rel_output
        
        # Concatenate heads and project
        output = output.transpose(1, 2).contiguous().view(B, N, self.d_model)
        return self.W_O(output)


# ──────────────────────────────────────────────────
# Usage
# ──────────────────────────────────────────────────

d_model = 64
num_heads = 4
max_rel_dist = 8

model = RelativePositionAttention(d_model, num_heads, max_rel_dist)

x = torch.randn(2, 12, d_model)
output = model(x)
print(f"Input shape:  {x.shape}")       # (2, 12, 64)
print(f"Output shape: {output.shape}")   # (2, 12, 64)

# Total relative position parameters
total_rel_params = sum(
    p.numel() for n, p in model.named_parameters() if 'rel_' in n
)
print(f"Relative position params: {total_rel_params}")
# (2 * max_rel_dist + 1) * d_k * 2 = 17 * 16 * 2 = 544
```

---

## 7. Comparison of Relative Position Methods

```
┌──────────────────────────────────────────────────────────────────┐
│           RELATIVE POSITION METHODS COMPARISON                   │
│                                                                  │
│  Method         Year  Where Applied     Parameters   Complexity  │
│  ───────────────────────────────────────────────────────────────  │
│  Shaw et al.    2018  Keys + Values     O(k·d_k)     Moderate   │
│  Transformer-XL 2019  4-term decomp     O(L·d_model) Higher     │
│  T5 bias        2020  Scalar logit bias O(buckets·H) Minimal    │
│  ALiBi          2022  Linear bias       0 params     Minimal    │
│  RoPE           2021  Q,K rotation      0 params     Moderate   │
│                                                                  │
│  k = max relative distance                                       │
│  L = sequence length                                             │
│  H = number of attention heads                                   │
│  d_k = head dimension                                            │
└──────────────────────────────────────────────────────────────────┘
```

---

## 8. Real-World Applications

| Application                    | Method & Model                                                   |
|-------------------------------|------------------------------------------------------------------|
| **Language Modeling**          | Transformer-XL uses relative PE for segment-level recurrence     |
| **Text-to-Text Generation**   | T5 uses simplified relative bias for all NLP tasks               |
| **Long Document Processing**  | Relative PE generalizes to varying lengths without retraining    |
| **Music Generation**          | Relative timing between notes is more musically meaningful       |
| **Code Understanding**        | Relative positions capture structural distance in syntax trees   |
| **Protein Folding**           | Relative distances between amino acids relate to 3D structure    |

---

## 9. Summary Table

| Aspect                    | Detail                                                          |
|--------------------------|-----------------------------------------------------------------|
| **Core Idea**             | Encode distance between tokens, not absolute positions          |
| **Key Paper**             | Shaw et al. (2018) — "Self-Attention with Relative Position"   |
| **How It Works**          | Add relative position embeddings to K and V in attention        |
| **Distance Clipping**     | Clip relative distance to [-k, k], total 2k+1 embeddings       |
| **Transformer-XL**        | 4-term decomposition with sinusoidal relative encoding          |
| **T5**                    | Simple scalar bias with logarithmic bucketing                   |
| **Key Advantage**         | Generalizes to unseen sequence lengths                          |
| **Key Advantage**         | Translation-invariant: same pattern detected anywhere           |
| **Complexity**            | O(n²) relative position matrix per layer                        |
| **Modern Evolution**      | Led to RoPE, ALiBi, and other efficient methods                 |

---

## 10. Revision Questions

1. **Explain the fundamental difference between absolute and relative positional encoding.
   Give a concrete example where relative encoding provides an advantage.**

2. **In Shaw et al.'s method, why are relative position embeddings added to both Keys and
   Values? What would be lost if they were only added to Keys?**

3. **What is distance clipping in relative positional encoding? Why is it both practical
   and theoretically justifiable?**

4. **Describe the four-term decomposition in Transformer-XL. What does each term capture,
   and how does it differ from standard absolute position attention?**

5. **T5 uses logarithmic bucketing for relative positions. Why is this more efficient than
   having a separate bias for every possible distance? What information is lost?**

6. **Compare Shaw et al., Transformer-XL, and T5 approaches in terms of: (a) number of
   parameters, (b) computational overhead, (c) ability to generalize to longer sequences.**

---

[← Learned Positional Embeddings](03-learned-positional-embeddings.md) | [Rotary Positional Encoding →](05-rotary-positional-encoding.md)
