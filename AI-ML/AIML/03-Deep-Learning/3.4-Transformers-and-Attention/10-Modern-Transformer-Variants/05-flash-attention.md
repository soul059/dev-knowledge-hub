# FlashAttention: Fast and Memory-Efficient Attention

> **Navigation:**
> Previous: [← Linear Attention](04-linear-attention.md) | Next: [Efficient Transformers →](06-efficient-transformers.md)

---

## Overview

FlashAttention, introduced by Dao et al. (2022), is an **IO-aware** algorithm that computes
**exact** standard attention in **O(N)** memory instead of O(N²), while being **2–4× faster**
than standard PyTorch attention on modern GPUs. Unlike sparse or linear attention methods,
FlashAttention does not approximate — it produces bit-for-bit identical results to standard
softmax attention. The key insight is that standard attention is **memory-bound** (limited by
reads/writes to GPU memory), not compute-bound. By restructuring the computation into
**tiles** that fit in fast GPU SRAM and **fusing** all operations into a single kernel,
FlashAttention dramatically reduces memory traffic and avoids ever materializing the full
N×N attention matrix. FlashAttention-2 further improves upon this with better parallelism
and work partitioning.

---

## 1. The Memory Bottleneck Problem

### 1.1 GPU Memory Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│                    GPU MEMORY HIERARCHY                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────┐                                   │
│  │   SRAM (on-chip)  │  ~20 MB     ~19 TB/s bandwidth  │
│  │   (Shared Memory) │  FAST!      (A100 GPU)          │
│  └────────┬─────────┘                                   │
│           │                                             │
│           │  ← bottleneck: data must travel here        │
│           │                                             │
│  ┌────────▼─────────┐                                   │
│  │   HBM (off-chip)  │  40-80 GB   ~2 TB/s bandwidth   │
│  │   (Global Memory) │  10× slower than SRAM           │
│  └──────────────────┘                                   │
│                                                         │
│  The gap: SRAM is ~10× faster but ~1000× smaller       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1.2 Standard Attention Memory Access Pattern

```
Standard attention (naive PyTorch implementation):

Step 1: S = Q K^T / √d     → Write S (N×N) to HBM        ← SLOW
Step 2: P = softmax(S)      → Read S from HBM, write P    ← SLOW
Step 3: O = P · V           → Read P from HBM             ← SLOW

Total HBM reads/writes: O(N² + N·d)

For N = 4096, d = 64:
    Attention matrix S: 4096² × 4 bytes = 64 MB
    HBM bandwidth: ~2 TB/s
    Just reading/writing S: ~64 μs (memory access alone!)
    Actual compute (multiply-adds): much faster than memory transfer

→ Attention is MEMORY-BOUND, not COMPUTE-BOUND
```

---

## 2. FlashAttention: The Tiling Strategy

### 2.1 Core Idea

Instead of computing the full N×N attention matrix, compute attention **block by block**,
keeping everything in fast SRAM:

```
┌──────────────────────────────────────────────────────────┐
│              FLASHATTENTION TILING STRATEGY               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Standard Attention:                                     │
│  ┌──────────────────────┐                                │
│  │                      │  ← Full N×N attention matrix   │
│  │    Materialized in   │     stored in HBM (slow)       │
│  │    HBM (64 MB+)      │                                │
│  │                      │                                │
│  └──────────────────────┘                                │
│                                                          │
│  FlashAttention:                                         │
│  ┌──────────────────────┐                                │
│  │ ┌────┐               │  ← Only one TILE (Br × Bc)    │
│  │ │TILE│               │     lives in SRAM (fast)       │
│  │ └────┘               │     Never store full matrix!   │
│  │          ┌────┐      │                                │
│  │          │TILE│      │  ← Process tiles sequentially  │
│  │          └────┘      │                                │
│  └──────────────────────┘                                │
│                                                          │
│  Tile sizes chosen to fit in SRAM:                       │
│     Br × Bc ≈ √(SRAM_size / d)                          │
│     For A100: SRAM ≈ 192 KB per SM                       │
│     Typical: Br = Bc = 128                               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### 2.2 The Online Softmax Trick

The challenge with tiling: softmax requires the **full row** to normalize. FlashAttention
uses an **online softmax** algorithm:

```
Online softmax (Milakov & Gimelshein, 2018):

Processing tile j for row i:

    m_new = max(m_old, max(S_ij))              ← running maximum
    l_new = l_old · exp(m_old - m_new)         ← rescale old normalizer
          + Σ exp(S_ij - m_new)                ← add new contributions

    O_new = O_old · (l_old · exp(m_old - m_new) / l_new)    ← rescale old output
          + exp(S_ij - m_new) · V_j / l_new                 ← add new contribution

After processing ALL tiles:
    m = true maximum (for numerical stability)
    l = true normalizer (Σ exp)
    O = exact softmax attention output!
```

---

## 3. FlashAttention Algorithm (Detailed)

### 3.1 Pseudocode

```
FLASHATTENTION(Q, K, V):
    Input:  Q, K, V ∈ ℝ^(N × d) in HBM
    Output: O ∈ ℝ^(N × d) in HBM

    1. Set block sizes:  Bc = ⌈SRAM_size / (4d)⌉,  Br = min(Bc, d)
    2. Initialize:       O = 0,  l = 0,  m = -∞   (all in HBM, size N)
    3. Divide Q into Tr = ⌈N/Br⌉ blocks,  K, V into Tc = ⌈N/Bc⌉ blocks

    4. FOR j = 1, ..., Tc:                         ← outer loop over K, V blocks
         Load K_j, V_j from HBM to SRAM            (size: Bc × d each)

       5. FOR i = 1, ..., Tr:                      ← inner loop over Q blocks
            Load Q_i, O_i, l_i, m_i from HBM to SRAM

            a. Compute S_ij = Q_i · K_j^T / √d     ← (Br × Bc) tile in SRAM
            b. m̃_ij = rowmax(S_ij)
            c. P̃_ij = exp(S_ij - m̃_ij)
            d. l̃_ij = rowsum(P̃_ij)

            e. m_new = max(m_i, m̃_ij)
            f. l_new = exp(m_i - m_new) · l_i + exp(m̃_ij - m_new) · l̃_ij

            g. O_i ← diag(l_i · exp(m_i - m_new))⁻¹ ·
                      (diag(l_i · exp(m_i - m_new)) · O_i + exp(m̃_ij - m_new) · P̃_ij · V_j)
               ─── simplifies to ───
               O_i ← (l_i · exp(m_i - m_new) · O_i + exp(m̃_ij - m_new) · P̃_ij · V_j) / l_new

            h. Update: m_i ← m_new,  l_i ← l_new
            i. Write O_i, l_i, m_i back to HBM

    6. Return O, l, m
```

### 3.2 ASCII Diagram of Tiling

```
        K₁   K₂   K₃   K₄        V₁   V₂   V₃   V₄
       ┌────┬────┬────┬────┐     ┌────┬────┬────┬────┐
       │    │    │    │    │     │    │    │    │    │
       │    │    │    │    │     │    │    │    │    │
       └────┴────┴────┴────┘     └────┴────┴────┴────┘

   Q₁ ┌────┐  S₁₁  S₁₂  S₁₃  S₁₄     ┌────┐
      │    │ ┌────┐                    │ O₁ │  ← accumulated
      │    │ │TILE│ computed in SRAM   │    │     in HBM
   Q₂ ├────┤ └────┘                    ├────┤
      │    │        ┌────┐             │ O₂ │
      │    │        │TILE│             │    │
   Q₃ ├────┤        └────┘             ├────┤
      │    │              ┌────┐       │ O₃ │
      │    │              │TILE│       │    │
   Q₄ ├────┤              └────┘       ├────┤
      │    │                    ┌────┐ │ O₄ │
      │    │                    │TILE│ │    │
      └────┘                    └────┘ └────┘

  Each TILE (Br × Bc) fits in SRAM.
  Process tiles left-to-right, updating O incrementally.
  NEVER materialize the full N×N attention matrix!
```

---

## 4. Complexity Analysis

### 4.1 Memory

```
Standard Attention:
    O(N²)  — must store full attention matrix

FlashAttention:
    O(N)   — only store O, l, m (each size N)
             plus one tile Br × Bc in SRAM at a time

Memory savings:
    N = 2048:   4M  → 2K       = 2000× reduction
    N = 4096:   16M → 4K       = 4000× reduction
    N = 16384:  256M → 16K     = 16000× reduction
```

### 4.2 HBM Accesses

```
Standard attention:
    HBM accesses = O(N²·d + N²)   (read/write attention matrix)

FlashAttention:
    HBM accesses = O(N²·d² / SRAM_size)

    Since SRAM_size >> d²:
        FlashAttention makes FEWER HBM accesses
        Typically 4-8× fewer memory transfers

    This is why FlashAttention is faster despite doing the SAME computation!
```

### 4.3 FLOPs (Unchanged)

```
FlashAttention performs the EXACT SAME FLOPs as standard attention:
    O(N² · d) multiply-adds

It's not faster because it does less work.
It's faster because it accesses slow memory less often.

Analogy:
    Standard: chef walks to pantry for each ingredient separately
    Flash:    chef plans meals, makes one trip, cooks in kitchen
    Same food (FLOPs), fewer trips (HBM accesses)
```

---

## 5. FlashAttention-2 Improvements

```
FlashAttention-2 (Dao, 2023):

1. REDUCED NON-MATMUL FLOPs:
   Rescaling operations reduced from O(N²) to fewer per-tile ops
   More time spent on matmuls (which GPUs excel at)

2. BETTER PARALLELISM:
   FlashAttention-1: parallelize over batch & heads
   FlashAttention-2: also parallelize over sequence length
   → Better utilization of GPU SMs

3. WORK PARTITIONING:
   Swap inner/outer loops:
     FA-1: outer over K/V blocks, inner over Q blocks
     FA-2: outer over Q blocks, inner over K/V blocks
   → Reduces shared memory reads/writes

Performance:
   FlashAttention-2 achieves ~72% of theoretical max FLOPs on A100
   (vs ~35% for FlashAttention-1 and ~25% for standard PyTorch)

┌─────────────────────────────────────────────────────┐
│         GPU FLOP UTILIZATION COMPARISON              │
│                                                     │
│  Theoretical Max ─────────────────────────── 100%   │
│                                                     │
│  FlashAttention-2  ████████████████████████▒  72%   │
│  FlashAttention-1  ██████████████▒            35%   │
│  PyTorch Attention █████████▒                 25%   │
│  Megatron          ████████████████▒          40%   │
│                                                     │
│  ▒ = utilized compute                               │
└─────────────────────────────────────────────────────┘
```

---

## 6. Backward Pass (Training)

### 6.1 The Recomputation Strategy

```
Standard backward pass:
    Save the N×N attention matrix P during forward pass → O(N²) memory
    Use P in backward to compute gradients

FlashAttention backward:
    DON'T save P during forward (saves O(N²) memory!)
    Instead, save only O, l, m (O(N) total)
    RECOMPUTE P tile-by-tile during backward pass

Trade-off:
    Extra FLOPs: ~1 extra matmul per tile (recomputation)
    Memory saved: O(N²) → O(N)

    In practice, the recomputation is FASTER than reading P from HBM!
    (Because SRAM computation > HBM read speed)
```

---

## 7. Worked Example

**Setting:** N = 2048, d = 64, SRAM = 192 KB (A100 SM), float16.

```
Step 1: Determine tile sizes
    Each element = 2 bytes (float16)
    Need to fit: Q_tile (Br × d), K_tile (Bc × d), V_tile (Bc × d),
                 S_tile (Br × Bc), O_tile (Br × d)

    Memory per tile ≈ Br·d + 2·Bc·d + Br·Bc + Br·d
                    = 2·Br·64 + 2·Bc·64 + Br·Bc
                    = 128·Br + 128·Bc + Br·Bc

    For Br = Bc = 128:
        Memory = 128·128 + 128·128 + 128·128 = 49,152 values
                = 49,152 × 2 bytes = 96 KB  ✓ fits in 192 KB SRAM

Step 2: Number of tiles
    Tr = 2048 / 128 = 16 (Q blocks)
    Tc = 2048 / 128 = 16 (K/V blocks)
    Total tiles: 16 × 16 = 256

Step 3: HBM accesses
    Standard:  Read/write S (2048×2048×2 bytes) = 8 MB  ×2 = 16 MB
    Flash:     Read Q, K, V once per outer loop pass
               Q: 2048 × 64 × 2 = 256 KB  (read Tc=16 times) = 4 MB
               K, V: 256 KB each (read Tr=16 times) = 4+4 = 8 MB
               O: 256 KB (write once per inner iter) = 4 MB
               Total: ~16 MB  (similar total, but more sequential/cacheable)

Step 4: Memory for attention matrix
    Standard:  2048 × 2048 × 2 bytes = 8 MB
    Flash:     128 × 128 × 2 bytes = 32 KB  (just one tile!)
    Savings:   256× less memory
```

---

## 8. PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math


# Method 1: Using PyTorch's built-in FlashAttention (recommended)
class FlashAttentionLayer(nn.Module):
    """Multi-head attention using PyTorch's scaled_dot_product_attention.

    As of PyTorch 2.0+, this automatically dispatches to FlashAttention-2
    when conditions are met (CUDA, float16/bfloat16, no custom mask).
    """

    def __init__(self, embed_dim, num_heads, dropout=0.0):
        super().__init__()
        self.embed_dim = embed_dim
        self.num_heads = num_heads
        self.head_dim = embed_dim // num_heads
        self.scale = self.head_dim ** -0.5

        self.qkv = nn.Linear(embed_dim, 3 * embed_dim)
        self.out_proj = nn.Linear(embed_dim, embed_dim)
        self.dropout = dropout

    def forward(self, x, is_causal=False):
        B, N, C = x.shape
        H, D = self.num_heads, self.head_dim

        qkv = self.qkv(x).reshape(B, N, 3, H, D).permute(2, 0, 3, 1, 4)
        q, k, v = qkv.unbind(0)  # Each: (B, H, N, D)

        # PyTorch 2.0+ automatically uses FlashAttention when possible
        out = F.scaled_dot_product_attention(
            q, k, v,
            dropout_p=self.dropout if self.training else 0.0,
            is_causal=is_causal,
        )

        out = out.transpose(1, 2).reshape(B, N, C)
        return self.out_proj(out)


# Method 2: Manual tiled attention (educational implementation)
def tiled_attention_forward(Q, K, V, block_size=128):
    """Educational implementation of FlashAttention's tiling strategy.

    NOTE: This is a simplified CPU/Python version for understanding.
    Real FlashAttention uses custom CUDA kernels for GPU SRAM access.
    """
    N, d = Q.shape
    O = torch.zeros_like(Q)
    l = torch.zeros(N, 1)      # normalizer
    m = torch.full((N, 1), float('-inf'))  # running max

    Tr = math.ceil(N / block_size)  # number of Q blocks
    Tc = math.ceil(N / block_size)  # number of K/V blocks

    for j in range(Tc):
        # Load K_j, V_j block
        kj_start = j * block_size
        kj_end = min(kj_start + block_size, N)
        Kj = K[kj_start:kj_end]        # (Bc, d)
        Vj = V[kj_start:kj_end]        # (Bc, d)

        for i in range(Tr):
            # Load Q_i block
            qi_start = i * block_size
            qi_end = min(qi_start + block_size, N)
            Qi = Q[qi_start:qi_end]     # (Br, d)

            # Compute attention scores for this tile
            Sij = Qi @ Kj.T / math.sqrt(d)     # (Br, Bc)

            # Online softmax update
            m_old = m[qi_start:qi_end]
            l_old = l[qi_start:qi_end]
            O_old = O[qi_start:qi_end]

            m_tile = Sij.max(dim=-1, keepdim=True).values
            m_new = torch.maximum(m_old, m_tile)

            # Rescale factors
            exp_old = torch.exp(m_old - m_new)
            exp_new = torch.exp(m_tile - m_new)

            P_tile = torch.exp(Sij - m_tile)    # (Br, Bc) — unnormalized
            l_tile = P_tile.sum(dim=-1, keepdim=True)

            l_new = exp_old * l_old + exp_new * l_tile

            # Update output
            O[qi_start:qi_end] = (
                exp_old * l_old * O_old + exp_new * P_tile @ Vj
            ) / l_new

            # Update running stats
            m[qi_start:qi_end] = m_new
            l[qi_start:qi_end] = l_new

    return O


# --- Verification: compare standard vs tiled attention ---
if __name__ == "__main__":
    torch.manual_seed(42)
    N, d = 256, 64
    Q = torch.randn(N, d)
    K = torch.randn(N, d)
    V = torch.randn(N, d)

    # Standard attention
    scores = Q @ K.T / math.sqrt(d)
    attn = F.softmax(scores, dim=-1)
    standard_out = attn @ V

    # Tiled (FlashAttention-style)
    tiled_out = tiled_attention_forward(Q, K, V, block_size=64)

    # Compare
    max_diff = (standard_out - tiled_out).abs().max().item()
    print(f"Standard attention output shape: {standard_out.shape}")
    print(f"Tiled attention output shape:    {tiled_out.shape}")
    print(f"Maximum difference:              {max_diff:.2e}")
    print(f"Are they equal (within tol)?     {torch.allclose(standard_out, tiled_out, atol=1e-5)}")

    # Using PyTorch's built-in (if CUDA available)
    layer = FlashAttentionLayer(embed_dim=256, num_heads=4)
    x = torch.randn(2, 128, 256)
    out = layer(x)
    print(f"\nFlashAttentionLayer — Input: {x.shape}, Output: {out.shape}")
```

---

## 9. When FlashAttention Is Used Automatically

```
PyTorch 2.0+ (torch.nn.functional.scaled_dot_product_attention):

Automatically selects the best backend:

┌─────────────────────────────────────────────────────────────┐
│  Backend Selection (in order of preference):                │
│                                                             │
│  1. FlashAttention-2                                        │
│     ✓ CUDA GPU                                              │
│     ✓ float16 or bfloat16                                   │
│     ✓ head_dim ≤ 256                                        │
│     ✓ No custom attention mask (use is_causal=True instead) │
│                                                             │
│  2. Memory-Efficient Attention (xformers)                   │
│     ✓ CUDA GPU                                              │
│     ✓ Supports custom masks                                 │
│     ✓ Supports float32                                      │
│                                                             │
│  3. Standard Math Attention (fallback)                      │
│     ✓ Any device (CPU, GPU)                                 │
│     ✓ Any dtype                                             │
│     ✓ Any mask                                              │
│     ✗ O(N²) memory                                          │
└─────────────────────────────────────────────────────────────┘

Check which backend is used:
    torch.backends.cuda.flash_sdp_enabled()
    torch.backends.cuda.mem_efficient_sdp_enabled()
```

---

## 10. Real-World Applications

| Application                | Impact                                                  |
|----------------------------|---------------------------------------------------------|
| **LLM Training**          | GPT-4, LLaMA 2 — 2-4× faster attention during training |
| **Long-Context Models**   | Enables 32K-128K context lengths (GPT-4, Claude)        |
| **Diffusion Models**      | Stable Diffusion, DALL-E — faster image generation      |
| **Video Models**          | Process longer video sequences in memory                |
| **Fine-Tuning**           | Enables fine-tuning larger models on consumer GPUs       |
| **Inference Optimization**| vLLM, TGI — use FlashAttention for fast serving         |
| **Protein Language Models**| ESMFold — long protein sequences with exact attention   |

---

## 11. Summary Table

| Concept                    | Details                                                 |
|----------------------------|---------------------------------------------------------|
| Paper                      | Dao et al., *FlashAttention* (NeurIPS 2022)             |
| Core insight               | Attention is memory-bound, not compute-bound            |
| Strategy                   | Tiling + fused kernel to minimize HBM access            |
| Exactness                  | **Exact** — produces identical results to standard attn |
| Memory complexity          | O(N) instead of O(N²)                                  |
| Speed                      | 2-4× faster than standard PyTorch attention             |
| Online softmax             | Enables correct softmax without materializing full row  |
| FlashAttention-2           | ~72% GPU utilization (vs 25% standard)                  |
| Backward pass              | Recomputes P from O, l, m — saves O(N²) memory         |
| PyTorch integration        | `F.scaled_dot_product_attention` (auto-dispatches)      |
| Impact                     | Made approximate attention less necessary               |

---

## 12. Revision Questions

1. **Why is standard attention memory-bound?** Explain the GPU memory hierarchy and why the N×N
   attention matrix is the bottleneck, not the floating-point computations.

2. **How does the online softmax trick work?** Given three tiles of attention scores, walk through
   the running maximum and normalizer updates step by step.

3. **Why does FlashAttention use MORE FLOPs but run FASTER?** Explain the relationship between
   arithmetic intensity, memory bandwidth, and the roofline model.

4. **Compare FlashAttention with sparse attention methods.** When would you still prefer sparse
   attention over FlashAttention, and vice versa?

5. **How does FlashAttention-2 improve upon FlashAttention-1?** Discuss the changes in loop
   ordering, parallelism, and non-matmul FLOP reduction.

6. **Explain FlashAttention's backward pass recomputation strategy.** Why is recomputing the
   attention matrix faster than loading it from HBM? Under what conditions might this trade-off
   not hold?

---

> **Navigation:**
> Previous: [← Linear Attention](04-linear-attention.md) | Next: [Efficient Transformers →](06-efficient-transformers.md)
