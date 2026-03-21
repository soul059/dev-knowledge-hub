# Linear Attention Mechanisms

> **Navigation:**
> Previous: [← Sparse Attention](03-sparse-attention.md) | Next: [FlashAttention →](05-flash-attention.md)

---

## Overview

Linear attention mechanisms fundamentally rethink the attention computation by replacing the
softmax-based **O(n²d)** operation with kernel-based approximations that run in **O(nd²)** or
even **O(nd)** time. The key insight is algebraic: by decomposing the softmax into separable
feature maps, we can change the order of matrix multiplications — computing **K^T · V** first
(an O(d²) operation per token) instead of **Q · K^T** (an O(n²) operation). This family of
methods includes the Linear Transformer, Performer (with the FAVOR+ algorithm), and Random
Feature Attention, each offering different trade-offs between speed, approximation quality, and
theoretical guarantees.

---

## 1. Standard Attention Revisited

### 1.1 The Softmax Bottleneck

```
Standard attention:
    A = softmax(Q K^T / √d_k)          ← O(n² · d) to compute Q K^T
    Output = A · V                       ← O(n² · d) for matrix multiply

    Q ∈ ℝ^(n × d),  K ∈ ℝ^(n × d),  V ∈ ℝ^(n × d)

    Step-by-step cost:
    ┌─────────────────────────────────────────────────────────┐
    │  Q K^T           :  ℝ^(n×d) × ℝ^(d×n) = ℝ^(n×n)      │
    │                     Cost: O(n² · d)                     │
    │  softmax(·)      :  Applied row-wise to ℝ^(n×n)        │
    │                     Cost: O(n²)                         │
    │  A · V           :  ℝ^(n×n) × ℝ^(n×d) = ℝ^(n×d)      │
    │                     Cost: O(n² · d)                     │
    │                                                         │
    │  Total:  O(n² · d)                                      │
    │  Memory: O(n²)  for the attention matrix                │
    └─────────────────────────────────────────────────────────┘
```

### 1.2 Attention as Kernel Smoothing

Standard attention can be viewed as:

```
Output_i = Σⱼ  sim(qᵢ, kⱼ) · vⱼ  /  Σⱼ sim(qᵢ, kⱼ)

where sim(q, k) = exp(q · k / √d)   (softmax kernel)

This is a WEIGHTED AVERAGE of values, where weights come from
a similarity function (kernel) between queries and keys.
```

---

## 2. The Kernel Trick for Attention

### 2.1 Core Idea

Replace `sim(q, k) = exp(q^T k / √d)` with a **decomposable kernel**:

```
sim(q, k) ≈ φ(q)^T · φ(k)

where φ: ℝ^d → ℝ^D is a feature map

Then:
    Output_i = Σⱼ [φ(qᵢ)^T φ(kⱼ)] · vⱼ  /  Σⱼ [φ(qᵢ)^T φ(kⱼ)]

            = φ(qᵢ)^T · [Σⱼ φ(kⱼ) · vⱼ^T]  /  φ(qᵢ)^T · [Σⱼ φ(kⱼ)]
```

### 2.2 The Associativity Trick

This is where the magic happens — **change the order of multiplication**:

```
Standard (left-to-right):
    (Q K^T) · V
     ╰──╯
     n × n   → then × V → O(n²d)

Linear (right-to-left):
    Q · (K^T V)
         ╰──╯
         d × d   → then Q × → O(nd²)

When d << n (which is typical: d=64, n=4096):
    n²d = 4096² × 64  = 1.07 × 10⁹
    nd²  = 4096  × 64² = 16.8 × 10⁶

    That's a 64× speedup!
```

### 2.3 Visual Comparison

```
STANDARD ATTENTION:                    LINEAR ATTENTION:

  Q        K^T         V                Q       K^T   V
(n×d)    (d×n)       (n×d)            (n×D)   (D×n) (n×d)

  ┌──┐    ┌────────┐                   ┌──┐    ┌────────┐
  │  │    │        │                   │  │    │        │ ┌──┐
  │  │  × │        │   ┌──┐           │  │  × │        │×│  │
n │  │    │        │ × │  │         n │  │    │        │ │  │
  │  │    │        │   │  │ n         │  │    └────────┘ │  │
  │  │    └────────┘   │  │           │  │      (D×n)    │  │ n
  └──┘                 │  │           └──┘               │  │
  (n×d)    (d×n)       └──┘           (n×D)    FIRST:    └──┘
                       (n×d)                   K^T V     (n×d)
  FIRST: QK^T → (n×n)                         → (D×d)
  THEN:  (n×n)V → (n×d)                THEN: Q(K^TV) → (n×d)
  Cost: O(n²d)                          Cost: O(nDd) ≈ O(nd²)
```

---

## 3. Linear Transformer (Katharopoulos et al., 2020)

### 3.1 Feature Map: ELU + 1

The Linear Transformer uses a simple deterministic feature map:

```
φ(x) = elu(x) + 1

where elu(x) = { x           if x > 0
               { exp(x) - 1  if x ≤ 0

Properties:
    - Non-negative: φ(x) > 0  ∀x (important for valid attention weights)
    - Differentiable everywhere
    - No random features needed
    - Simple to implement
```

### 3.2 Causal Linear Attention

For autoregressive (causal) models, we need causal masking. Linear attention supports this
efficiently using **cumulative sums**:

```
Standard causal attention:
    Output_i = Σ_{j≤i} softmax(qᵢ · kⱼ / √d) · vⱼ

Linear causal attention:
    Sᵢ = Σ_{j≤i} φ(kⱼ) ⊗ vⱼ        (running sum, updated incrementally)
    zᵢ = Σ_{j≤i} φ(kⱼ)              (normalizer, running sum)

    Output_i = φ(qᵢ)^T · Sᵢ / (φ(qᵢ)^T · zᵢ)

Complexity:
    Each step: O(d²) to update Sᵢ and compute output
    Total: O(n · d²)
    Memory: O(d²) for the running state (constant in n!)
```

### 3.3 Comparison with RNNs

```
Linear causal attention is equivalent to an RNN!

Hidden state: Sᵢ ∈ ℝ^(D × d)  (the accumulated key-value outer products)
Update:       Sᵢ = S_{i-1} + φ(kᵢ) ⊗ vᵢ
Output:       oᵢ = φ(qᵢ)^T · Sᵢ / (φ(qᵢ)^T · zᵢ)

This means:
    - Training: use parallel attention formulation — O(nd²)
    - Inference: use RNN formulation — O(d²) per step (constant time!)
```

---

## 4. Performer: FAVOR+ Algorithm (Choromanski et al., 2021)

### 4.1 Random Feature Maps

Performer uses **random features** to approximate the softmax kernel:

```
exp(q^T k / √d) ≈ φ(q)^T · φ(k)

where φ(x) = (1/√m) · exp(-||x||² / 2) · [exp(ω₁^T x), ..., exp(ωₘ^T x)]

    ωᵢ ~ N(0, I_d)    (random projections from standard Gaussian)
    m = number of random features (typically m ≈ d to 4d)
```

### 4.2 FAVOR+: Fast Attention Via Positive Orthogonal Random Features

Key improvements over basic random features:

```
1. POSITIVE random features:
   φ(x) = (1/√m) · exp(-||x||²/2) · [exp(ω₁^Tx), ..., exp(ωₘ^Tx)]

   Ensures non-negative attention weights (critical for stability)

2. ORTHOGONAL random features:
   Instead of i.i.d. random ωᵢ, use orthogonal random vectors
   → Lower variance in the approximation
   → Better approximation with fewer features

3. Redrawing:
   Periodically resample random features during training
   → Reduces bias from fixed random projection
```

### 4.3 Approximation Quality

```
Approximation error of FAVOR+:

    E[||Â - A||_F²] = O(1/m)

    where:
        Â = approximate attention matrix
        A = exact softmax attention
        m = number of random features

    For m = d (e.g., 64):   moderate approximation
    For m = 4d (e.g., 256): good approximation
    For m → ∞:              exact recovery

Practical guidance:
    m = 256 features typically suffices for d = 64
```

---

## 5. Complexity Analysis

```
Method                   Time Complexity    Memory     Quality
──────────────────────────────────────────────────────────────
Standard Attention       O(n² · d)          O(n²)      Exact
Linear Transformer       O(n · d²)          O(d²)      Approximate
Performer (FAVOR+)       O(n · m · d)       O(n·m+m·d) Approximate
Random Feature Attn      O(n · m · d)       O(n·m)     Approximate

Where:
    n = sequence length
    d = head dimension (typically 64)
    m = number of random features

Break-even point (Standard vs Linear):
    n² · d  =  n · d²
    n = d

    When n > d (almost always): linear attention wins
    For n = 4096, d = 64: linear is ~64× faster
```

---

## 6. Worked Example

**Setting:** n = 2048, d = 64, m = 128 (random features for Performer).

```
Standard Attention:
    QK^T:            2048 × 2048 × 64 = 268M multiply-adds
    Attention × V:   2048 × 2048 × 64 = 268M multiply-adds
    Total:           ~536M ops
    Memory (attn):   2048² × 4 bytes = 16 MB

Linear Transformer (φ = elu+1, D = d = 64):
    K^T V:           64 × 2048 × 64 = 8.4M  (compute once)
    Q × (K^TV):      2048 × 64 × 64 = 8.4M  (per query block)
    Total:           ~16.8M ops
    Memory:          64 × 64 × 4 bytes = 16 KB (!)
    Speedup:         ~32×

Performer (FAVOR+, m = 128):
    φ(Q):            2048 × 128 ops  (apply random features)
    φ(K):            2048 × 128 ops
    φ(K)^T V:        128 × 2048 × 64 = 16.8M
    φ(Q) × result:   2048 × 128 × 64 = 16.8M
    Total:           ~33.6M ops
    Speedup:         ~16×
```

---

## 7. PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math


class LinearAttention(nn.Module):
    """Linear attention using elu+1 feature map (Katharopoulos et al.)."""

    def __init__(self, embed_dim, num_heads, causal=False):
        super().__init__()
        self.embed_dim = embed_dim
        self.num_heads = num_heads
        self.head_dim = embed_dim // num_heads
        self.causal = causal

        self.qkv = nn.Linear(embed_dim, 3 * embed_dim)
        self.out_proj = nn.Linear(embed_dim, embed_dim)

    @staticmethod
    def feature_map(x):
        """elu + 1 feature map (ensures non-negative values)."""
        return F.elu(x) + 1

    def forward(self, x):
        B, N, C = x.shape
        H, D = self.num_heads, self.head_dim

        qkv = self.qkv(x).reshape(B, N, 3, H, D).permute(2, 0, 3, 1, 4)
        q, k, v = qkv.unbind(0)  # Each: (B, H, N, D)

        # Apply feature map
        q = self.feature_map(q)
        k = self.feature_map(k)

        if self.causal:
            # Causal linear attention via cumulative sums
            output = self._causal_linear_attention(q, k, v)
        else:
            # Non-causal: use the associativity trick
            # K^T V first: (B, H, D, D) — this is the key insight!
            kv = torch.einsum('bhnd,bhnm->bhdm', k, v)  # (B, H, D, D)

            # Normalizer
            z = 1.0 / (torch.einsum('bhnd,bhd->bhn', q, k.sum(dim=2)) + 1e-6)

            # Q × (K^T V): (B, H, N, D)
            output = torch.einsum('bhnd,bhdm->bhnm', q, kv) * z.unsqueeze(-1)

        output = output.transpose(1, 2).reshape(B, N, C)
        return self.out_proj(output)

    def _causal_linear_attention(self, q, k, v):
        """Causal linear attention using cumulative sum (RNN-like)."""
        B, H, N, D = q.shape
        Dv = v.shape[-1]

        output = torch.zeros(B, H, N, Dv, device=q.device, dtype=q.dtype)
        S = torch.zeros(B, H, D, Dv, device=q.device, dtype=q.dtype)
        z = torch.zeros(B, H, D, device=q.device, dtype=q.dtype)

        for i in range(N):
            ki = k[:, :, i]     # (B, H, D)
            vi = v[:, :, i]     # (B, H, Dv)
            qi = q[:, :, i]     # (B, H, D)

            # Update running state: S += k_i ⊗ v_i
            S = S + torch.einsum('bhd,bhm->bhdm', ki, vi)
            z = z + ki

            # Compute output: q_i^T S / q_i^T z
            num = torch.einsum('bhd,bhdm->bhm', qi, S)
            den = torch.einsum('bhd,bhd->bh', qi, z).unsqueeze(-1) + 1e-6
            output[:, :, i] = num / den

        return output


class PerformerAttention(nn.Module):
    """Performer attention using FAVOR+ (random feature maps)."""

    def __init__(self, embed_dim, num_heads, num_features=256):
        super().__init__()
        self.embed_dim = embed_dim
        self.num_heads = num_heads
        self.head_dim = embed_dim // num_heads
        self.num_features = num_features

        self.qkv = nn.Linear(embed_dim, 3 * embed_dim)
        self.out_proj = nn.Linear(embed_dim, embed_dim)

        # Random projection matrix (orthogonal for FAVOR+)
        projection = torch.randn(self.head_dim, num_features)
        projection = torch.linalg.qr(projection.T)[0].T  # Orthogonalize
        self.register_buffer('projection', projection)

    def _favor_plus_features(self, x):
        """Compute FAVOR+ positive random features."""
        # x: (B, H, N, D)
        # projection: (D, m)
        projected = torch.einsum('bhnd,dm->bhnm', x, self.projection)

        # Positive random features: exp(projected - ||x||²/2)
        norm_sq = (x ** 2).sum(dim=-1, keepdim=True) / 2
        features = torch.exp(projected - norm_sq) / math.sqrt(self.num_features)
        return features  # (B, H, N, m)

    def forward(self, x):
        B, N, C = x.shape
        H, D = self.num_heads, self.head_dim

        qkv = self.qkv(x).reshape(B, N, 3, H, D).permute(2, 0, 3, 1, 4)
        q, k, v = qkv.unbind(0)

        # Apply FAVOR+ feature map
        q_prime = self._favor_plus_features(q)  # (B, H, N, m)
        k_prime = self._favor_plus_features(k)  # (B, H, N, m)

        # Linear attention with features
        kv = torch.einsum('bhnm,bhnd->bhmd', k_prime, v)  # (B, H, m, D)
        z = 1.0 / (torch.einsum('bhnm,bhm->bhn', q_prime,
                                  k_prime.sum(dim=2)) + 1e-6)
        output = torch.einsum('bhnm,bhmd->bhnd', q_prime, kv) * z.unsqueeze(-1)

        output = output.transpose(1, 2).reshape(B, N, C)
        return self.out_proj(output)


# --- Quick test ---
if __name__ == "__main__":
    B, N, D = 2, 1024, 256
    x = torch.randn(B, N, D)

    # Linear Transformer (non-causal)
    linear_attn = LinearAttention(embed_dim=256, num_heads=8)
    out1 = linear_attn(x)
    print(f"Linear Attention — Input: {x.shape}, Output: {out1.shape}")

    # Linear Transformer (causal)
    causal_attn = LinearAttention(embed_dim=256, num_heads=8, causal=True)
    out2 = causal_attn(x)
    print(f"Causal Linear   — Input: {x.shape}, Output: {out2.shape}")

    # Performer (FAVOR+)
    performer = PerformerAttention(embed_dim=256, num_heads=8, num_features=128)
    out3 = performer(x)
    print(f"Performer       — Input: {x.shape}, Output: {out3.shape}")

    # Parameter comparison
    for name, model in [("Linear", linear_attn), ("Performer", performer)]:
        params = sum(p.numel() for p in model.parameters())
        print(f"{name}: {params:,} parameters")
```

---

## 8. Trade-offs: Speed vs Quality

```
                    QUALITY vs SPEED TRADE-OFF

  Quality  │
  (vs Full │  ●  Standard Attention (exact)
  Softmax) │
     100%  │  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
           │                    ● Performer (m=4d)
      95%  │              ● Performer (m=2d)
           │
      90%  │        ● Performer (m=d)
           │
      85%  │  ● Linear Transformer (elu+1)
           │
           └──────────────────────────────────────
               1×      4×      16×     64×
                      SPEED (vs standard)

Key observations:
  - Linear Transformer: fastest but lowest quality
  - Performer: tunable quality via m (more features = better)
  - FlashAttention (next chapter): exact + fast via hardware optimization
```

---

## 9. Real-World Applications

| Application                 | Method               | Why Linear Attention Helps                |
|-----------------------------|----------------------|-------------------------------------------|
| **Protein Folding**         | Performer            | Protein sequences can be 10K+ residues    |
| **Speech Recognition**      | Linear Transformer   | Audio sequences: 10K+ frames              |
| **Reinforcement Learning**  | Linear Transformer   | Fast inference for real-time decisions     |
| **Point Cloud Processing**  | Linear attention     | 3D point clouds: 50K+ points              |
| **Time Series Forecasting** | Performer            | Long historical sequences                 |
| **Streaming / Online**      | Causal Linear        | Constant-time per-step via RNN form        |

---

## 10. Limitations and When NOT to Use

```
Situations where linear attention may underperform:

1. SHORT SEQUENCES (n < 512):
   Standard attention is already fast enough.
   FlashAttention makes it even faster with exact computation.

2. TASKS REQUIRING SHARP ATTENTION:
   Linear attention produces "softer" attention distributions.
   Tasks like copying or precise retrieval may suffer.

3. WHEN APPROXIMATION ERROR MATTERS:
   Machine translation quality can degrade with approximate attention.
   Performer with small m may miss fine-grained token interactions.

4. MODERN HARDWARE:
   FlashAttention achieves near-linear wall-clock time for exact attention
   by optimizing memory access patterns (see next chapter).
```

---

## 11. Summary Table

| Concept                  | Details                                                   |
|--------------------------|-----------------------------------------------------------|
| Core insight             | Replace softmax with decomposable kernel: φ(Q)φ(K)^T     |
| Associativity trick      | Compute K^T V first (d×d) instead of QK^T (n×n)          |
| Linear Transformer       | φ(x) = elu(x) + 1, simple, fast, moderate quality        |
| Performer (FAVOR+)       | Random orthogonal positive features, tunable quality      |
| Complexity               | O(nd²) or O(nmd) instead of O(n²d)                       |
| Causal support           | Yes — equivalent to RNN with O(d²) state                 |
| Break-even               | n > d (almost always true in practice)                    |
| Quality trade-off        | Approximate — may lose fine-grained attention patterns     |
| Key advantage            | Constant memory per step in causal/streaming mode         |

---

## 12. Revision Questions

1. **Explain the associativity trick in linear attention.** Why does computing K^T V before
   multiplying by Q reduce complexity from O(n²d) to O(nd²)?

2. **Why must the feature map φ produce non-negative outputs?** What goes wrong if φ(q)^T φ(k)
   can be negative?

3. **Compare the Linear Transformer and Performer.** What are the trade-offs in terms of
   approximation quality, computational overhead, and implementation complexity?

4. **Show that causal linear attention is equivalent to an RNN.** Write out the recurrence
   relation and identify the hidden state, update rule, and output function.

5. **When would you prefer Performer over FlashAttention?** Consider scenarios involving
   extremely long sequences, streaming inference, and hardware constraints.

6. **Calculate the break-even sequence length** where Performer with m=128 random features
   becomes faster than standard attention for head dimension d=64.

---

> **Navigation:**
> Previous: [← Sparse Attention](03-sparse-attention.md) | Next: [FlashAttention →](05-flash-attention.md)
