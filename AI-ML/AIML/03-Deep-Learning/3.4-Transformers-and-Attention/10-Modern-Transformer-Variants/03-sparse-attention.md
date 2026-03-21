# Sparse Attention Mechanisms

> **Navigation:**
> Previous: [← Swin Transformer](02-swin-transformer.md) | Next: [Linear Attention →](04-linear-attention.md)

---

## Overview

Standard transformer self-attention computes pairwise interactions between all tokens, resulting
in **O(n²)** time and memory complexity. For long sequences (documents, high-resolution images,
genomics), this quadratic cost becomes prohibitive. Sparse attention mechanisms address this by
restricting each token to attend to only a **subset** of other tokens, defined by carefully
designed sparsity patterns. This section covers the major sparse attention variants — local/sliding
window, strided, Longformer, and BigBird — analyzing their patterns, complexity, and the
theoretical properties that make them effective replacements for full attention.

---

## 1. The Quadratic Bottleneck

```
Standard Self-Attention:
    Attention(Q, K, V) = softmax(Q K^T / √d_k) · V

    Q, K, V ∈ ℝ^(n × d)

    Q K^T  →  ℝ^(n × n)    ← This is the bottleneck!

    Time:   O(n² · d)
    Memory: O(n²)  (storing the full attention matrix)

For different sequence lengths:
    n = 512    →  n² = 262K       ✓ feasible
    n = 4,096  →  n² = 16.8M      ⚠ expensive
    n = 16,384 →  n² = 268M       ✗ very expensive
    n = 65,536 →  n² = 4.3B       ✗ impractical
```

---

## 2. Local / Sliding Window Attention

### 2.1 Concept

Each token attends only to its **w nearest neighbors** (w/2 on each side):

```
Window size w = 4 (attend to 2 left + self + 2 right - 1 = 5 tokens):

Token:    1  2  3  4  5  6  7  8  9  10 11 12
          ─  ─  ─  ─  ─  ─  ─  ─  ─  ──  ── ──
Token 1:  ■  ■  ■  .  .  .  .  .  .  .   .  .
Token 2:  ■  ■  ■  ■  .  .  .  .  .  .   .  .
Token 3:  ■  ■  ■  ■  ■  .  .  .  .  .   .  .
Token 4:  .  ■  ■  ■  ■  ■  .  .  .  .   .  .
Token 5:  .  .  ■  ■  ■  ■  ■  .  .  .   .  .
Token 6:  .  .  .  ■  ■  ■  ■  ■  .  .   .  .
Token 7:  .  .  .  .  ■  ■  ■  ■  ■  .   .  .
Token 8:  .  .  .  .  .  ■  ■  ■  ■  ■   .  .
Token 9:  .  .  .  .  .  .  ■  ■  ■  ■   ■  .
Token 10: .  .  .  .  .  .  .  ■  ■  ■   ■  ■
Token 11: .  .  .  .  .  .  .  .  ■  ■   ■  ■
Token 12: .  .  .  .  .  .  .  .  .  ■   ■  ■

■ = attends to    . = does not attend to
```

### 2.2 Complexity

```
Each token attends to w tokens (instead of n):
    Time:   O(n · w · d)    — linear in n
    Memory: O(n · w)

Receptive field after L layers:
    L layers × w window → effective receptive field = L × w
    For L=12, w=256: receptive field = 3,072 tokens
```

---

## 3. Strided (Dilated) Attention

### 3.1 Concept

Each token attends to every **k-th** token, covering a wider range with fewer connections:

```
Stride k = 3:

Token:    1  2  3  4  5  6  7  8  9  10  11  12
          ─  ─  ─  ─  ─  ─  ─  ─  ─  ──  ──  ──
Token 1:  ■  .  .  ■  .  .  ■  .  .  ■   .   .
Token 2:  .  ■  .  .  ■  .  .  ■  .  .   ■   .
Token 3:  .  .  ■  .  .  ■  .  .  ■  .   .   ■
Token 4:  ■  .  .  ■  .  .  ■  .  .  ■   .   .
  ...

Each token attends to n/k tokens
Time:  O(n · (n/k) · d) = O(n²·d/k)
```

### 3.2 Combining Local + Strided (Sparse Transformer, Child et al. 2019)

```
Head 1: Local attention (nearby tokens)
Head 2: Strided attention (distant tokens)

Combined pattern for token i:
    Local:    {i-w/2, ..., i, ..., i+w/2}
    Strided:  {i mod k, i mod k + k, i mod k + 2k, ...}

Effective complexity: O(n · √n)  (with stride k = √n)
```

---

## 4. Longformer (Beltagy et al., 2020)

### 4.1 Three Attention Patterns

Longformer combines **three** types of attention:

```
┌───────────────────────────────────────────────────────┐
│                 LONGFORMER ATTENTION                  │
├───────────────────────────────────────────────────────┤
│                                                       │
│  1. SLIDING WINDOW (Local)                           │
│     Each token attends to w neighbors                │
│     ┌─────────────────────────────────────┐          │
│     │ . . ■ ■ ■ . . . . . . . . . . . . │          │
│     │ . . . ■ ■ ■ . . . . . . . . . . . │          │
│     │ . . . . ■ ■ ■ . . . . . . . . . . │          │
│     └─────────────────────────────────────┘          │
│                                                       │
│  2. DILATED SLIDING WINDOW                           │
│     Window with gaps of size d (dilation)            │
│     ┌─────────────────────────────────────┐          │
│     │ ■ . ■ . ■ . ■ . . . . . . . . . . │          │
│     │ . ■ . ■ . ■ . ■ . . . . . . . . . │          │
│     │ . . ■ . ■ . ■ . ■ . . . . . . . . │          │
│     └─────────────────────────────────────┘          │
│                                                       │
│  3. GLOBAL ATTENTION (on select tokens)              │
│     Special tokens ([CLS]) attend to ALL tokens      │
│     ┌─────────────────────────────────────┐          │
│     │ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ │ ← [CLS] │
│     │ ■ . . ■ ■ ■ . . . . . . . . . . . │          │
│     │ ■ . . . ■ ■ ■ . . . . . . . . . . │          │
│     └─────────────────────────────────────┘          │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### 4.2 Complexity

```
Sliding window:    O(n · w)       — w = window size
Dilated window:    O(n · w)       — same count, wider coverage
Global attention:  O(n · g)       — g = number of global tokens (small)

Total: O(n · (w + g))  ≈  O(n · w)   since g << n

Example: n = 4096, w = 512, g = 2
    Full attention: 4096² = 16.8M
    Longformer:     4096 × (512 + 2) = 2.1M
    Speedup:        ~8×
```

### 4.3 Multi-Head Strategy

```
Different heads use different configurations:
    Heads 1-4:  window size 256, no dilation
    Heads 5-8:  window size 256, dilation = 2  (effective window = 512)
    Heads 9-12: window size 256, dilation = 4  (effective window = 1024)

This gives each layer a mix of local and broader attention.
```

---

## 5. BigBird (Zaheer et al., 2020)

### 5.1 Three Components

BigBird combines **random**, **window**, and **global** attention:

```
┌────────────────────────────────────────────────────────┐
│                   BIGBIRD ATTENTION                    │
├────────────────────────────────────────────────────────┤
│                                                        │
│  RANDOM ATTENTION        WINDOW ATTENTION              │
│  (r random tokens)       (w neighbors)                 │
│                                                        │
│  . . . ■ . . ■ . .      . . ■ ■ ■ . . . .            │
│  ■ . . . . ■ . . .      . . . ■ ■ ■ . . .            │
│  . ■ . . . . . ■ .      . . . . ■ ■ ■ . .            │
│                                                        │
│      +                                                 │
│                                                        │
│  GLOBAL ATTENTION                                      │
│  (g tokens attend all)                                 │
│                                                        │
│  ■ ■ ■ ■ ■ ■ ■ ■ ■   ← global token                  │
│  ■ . . . . . . . .                                    │
│  ■ . . . . . . . .                                    │
│                                                        │
│  COMBINED:                                             │
│  ■ ■ ■ ■ ■ ■ ■ ■ ■   ← global                        │
│  ■ . . ■ ■ ■ ■ . .   ← window + random               │
│  ■ ■ . . ■ ■ ■ ■ .   ← window + random               │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 5.2 Theoretical Guarantee: Turing Completeness

```
Theorem (Zaheer et al.):
    BigBird's sparse attention (random + window + global) is a
    UNIVERSAL APPROXIMATOR of sequence-to-sequence functions.

    Specifically, BigBird with:
      - g ≥ 1 global tokens
      - w ≥ 1 window size
      - r ≥ 1 random connections
    can simulate any Turing machine.

This means BigBird loses NO expressive power compared to full attention,
despite using O(n) connections instead of O(n²).
```

### 5.3 Complexity

```
Random:  O(n · r)     — r random connections per token
Window:  O(n · w)     — w neighboring tokens
Global:  O(n · g)     — g global tokens

Total:   O(n · (r + w + g))  =  O(n)   (all constants)

Typical values: r = 3, w = 3, g = 2
    Each token attends to ~8 tokens instead of n
    For n = 4096: 8/4096 = 0.2% of full attention
```

---

## 6. Comparison of Sparse Patterns

```
Full Attention          Sliding Window         Strided
■ ■ ■ ■ ■ ■ ■ ■        ■ ■ ■ . . . . .        ■ . . ■ . . ■ .
■ ■ ■ ■ ■ ■ ■ ■        ■ ■ ■ ■ . . . .        . ■ . . ■ . . ■
■ ■ ■ ■ ■ ■ ■ ■        . ■ ■ ■ ■ . . .        . . ■ . . ■ . .
■ ■ ■ ■ ■ ■ ■ ■        . . ■ ■ ■ ■ . .        ■ . . ■ . . ■ .
■ ■ ■ ■ ■ ■ ■ ■        . . . ■ ■ ■ ■ .        . ■ . . ■ . . ■
■ ■ ■ ■ ■ ■ ■ ■        . . . . ■ ■ ■ ■        . . ■ . . ■ . .
■ ■ ■ ■ ■ ■ ■ ■        . . . . . ■ ■ ■        ■ . . ■ . . ■ .
■ ■ ■ ■ ■ ■ ■ ■        . . . . . . ■ ■        . ■ . . ■ . . ■

Longformer              BigBird
■ ■ ■ ■ ■ ■ ■ ■ ←CLS   ■ ■ ■ ■ ■ ■ ■ ■ ←GBL
■ ■ ■ ■ . . . .        ■ ■ ■ ■ . ■ . .
■ ■ ■ ■ ■ . . .        ■ ■ ■ ■ ■ . . ■
■ . ■ ■ ■ ■ . .        ■ . ■ ■ ■ ■ . .
■ . . ■ ■ ■ ■ .        ■ . ■ ■ ■ ■ . .
■ . . . ■ ■ ■ ■        ■ ■ . . ■ ■ ■ .
■ . . . . ■ ■ ■        ■ . . . . ■ ■ ■
■ . . . . . ■ ■        ■ . ■ . . . ■ ■
```

---

## 7. Complexity Comparison Table

| Method           | Attention Complexity | Memory     | Receptive Field    | Quality      |
|------------------|---------------------|------------|--------------------|--------------| 
| Full Attention   | O(n²·d)             | O(n²)      | Global (1 layer)   | Best         |
| Sliding Window   | O(n·w·d)            | O(n·w)     | L·w (L layers)     | Good (local) |
| Strided          | O(n·(n/k)·d)        | O(n·n/k)   | Global (1 layer)   | Good         |
| Local + Strided  | O(n·√n·d)           | O(n·√n)    | Global (1 layer)   | Good         |
| Longformer       | O(n·w·d)            | O(n·w)     | Global (via [CLS]) | Very Good    |
| BigBird          | O(n·(r+w+g)·d)      | O(n)       | Global (theory)    | Very Good    |

---

## 8. Worked Example

**Setting:** Sequence length n = 4096, embedding d = 64, using Longformer with w = 256, g = 1.

```
Full Attention:
    Attention matrix: 4096 × 4096 = 16,777,216 entries
    FLOPs (QK^T):     2 × 4096 × 4096 × 64 = 2.15 × 10⁹
    Memory:            16,777,216 × 4 bytes = 64 MB

Longformer (sliding window w=256 + 1 global):
    Per-token attention: 256 (window) + 1 (global) = 257
    Total entries:       4096 × 257 = 1,052,672
    FLOPs (QK^T):        2 × 4096 × 257 × 64 = 1.35 × 10⁸
    Memory:              1,052,672 × 4 bytes = 4 MB

Speedup: ~16× fewer FLOPs, ~16× less memory
```

---

## 9. PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F


class SlidingWindowAttention(nn.Module):
    """Sliding window self-attention with optional global tokens."""

    def __init__(self, embed_dim, num_heads, window_size, num_global_tokens=0):
        super().__init__()
        self.embed_dim = embed_dim
        self.num_heads = num_heads
        self.head_dim = embed_dim // num_heads
        self.window_size = window_size
        self.num_global = num_global_tokens
        self.scale = self.head_dim ** -0.5

        self.qkv = nn.Linear(embed_dim, 3 * embed_dim)
        self.out_proj = nn.Linear(embed_dim, embed_dim)

    def _sliding_window_attention(self, q, k, v):
        """Compute sliding window attention (simplified implementation)."""
        B, H, N, D = q.shape
        w = self.window_size

        # Build the sliding window attention mask
        # Each token attends to w tokens centered around it
        mask = torch.zeros(N, N, dtype=torch.bool, device=q.device)
        for i in range(N):
            start = max(0, i - w // 2)
            end = min(N, i + w // 2 + 1)
            mask[i, start:end] = True

        # Standard attention with mask
        attn_weights = torch.matmul(q, k.transpose(-2, -1)) * self.scale

        # Apply mask: set non-attended positions to -inf
        attn_weights = attn_weights.masked_fill(~mask.unsqueeze(0).unsqueeze(0), float('-inf'))

        attn_weights = F.softmax(attn_weights, dim=-1)
        output = torch.matmul(attn_weights, v)
        return output

    def _global_attention(self, q, k, v, global_mask):
        """Global tokens attend to all, all attend to global tokens."""
        B, H, N, D = q.shape

        # Global tokens: full attention
        global_q = q[:, :, :self.num_global]           # (B, H, g, D)
        global_attn = torch.matmul(global_q, k.transpose(-2, -1)) * self.scale
        global_attn = F.softmax(global_attn, dim=-1)
        global_out = torch.matmul(global_attn, v)      # (B, H, g, D)

        return global_out

    def forward(self, x):
        B, N, C = x.shape

        # Compute Q, K, V
        qkv = self.qkv(x).reshape(B, N, 3, self.num_heads, self.head_dim)
        qkv = qkv.permute(2, 0, 3, 1, 4)  # (3, B, H, N, D)
        q, k, v = qkv.unbind(0)

        if self.num_global > 0:
            # Local tokens: sliding window
            local_out = self._sliding_window_attention(
                q[:, :, self.num_global:],
                k, v
            )

            # Global tokens: full attention
            global_out = self._global_attention(q, k, v, None)

            # Combine
            output = torch.cat([global_out, local_out], dim=2)
        else:
            output = self._sliding_window_attention(q, k, v)

        # Reshape and project
        output = output.transpose(1, 2).reshape(B, N, C)
        output = self.out_proj(output)
        return output


# --- Quick test ---
if __name__ == "__main__":
    B, N, D = 2, 128, 256
    x = torch.randn(B, N, D)

    # Pure sliding window
    attn_local = SlidingWindowAttention(
        embed_dim=256, num_heads=8, window_size=16
    )
    out = attn_local(x)
    print(f"Sliding window — Input: {x.shape}, Output: {out.shape}")

    # Sliding window + global tokens (Longformer-style)
    attn_longformer = SlidingWindowAttention(
        embed_dim=256, num_heads=8, window_size=16, num_global_tokens=1
    )
    out = attn_longformer(x)
    print(f"Longformer-style — Input: {x.shape}, Output: {out.shape}")

    # Compare FLOPs (approximate)
    full_attn_ops = N * N * D
    window_attn_ops = N * 16 * D
    print(f"\nFull attention ops:    {full_attn_ops:,}")
    print(f"Window attention ops:  {window_attn_ops:,}")
    print(f"Reduction:             {full_attn_ops / window_attn_ops:.1f}×")
```

---

## 10. Real-World Applications

| Application                   | Method     | Details                                  |
|-------------------------------|-----------|------------------------------------------|
| **Long Document QA**          | Longformer | Handles 4,096+ token documents           |
| **Genomics / DNA sequences**  | BigBird    | Sequences of 10K+ nucleotides            |
| **Long-form Summarization**   | Longformer | Summarize entire research papers          |
| **Code Understanding**        | BigBird    | Full source files (thousands of lines)    |
| **Legal Document Analysis**   | Longformer | Contracts and filings (10K+ tokens)       |
| **Image Generation**          | Sparse Tx  | High-resolution image synthesis (OpenAI)  |
| **Music Generation**          | Sparse Tx  | Long audio sequences (MuseNet)            |

---

## 11. Summary Table

| Concept                | Details                                                     |
|------------------------|-------------------------------------------------------------|
| Core problem           | O(n²) attention is impractical for long sequences           |
| Sliding window         | Attend to w nearest neighbors — O(n·w)                     |
| Strided attention      | Attend to every k-th token — O(n²/k)                       |
| Longformer             | Sliding window + dilated + global attention                 |
| BigBird                | Random + window + global — Turing complete                  |
| Key trade-off          | Speed vs. ability to capture long-range dependencies        |
| Receptive field        | Grows linearly with depth for local patterns                |
| Best for               | Sequences >2,048 tokens where full attention is too costly  |
| Modern alternative     | FlashAttention makes full attention fast for moderate n     |

---

## 12. Revision Questions

1. **Why is O(n²) attention problematic?** Calculate the memory required for full attention on a
   sequence of length 16,384 with float32 precision.

2. **How does the receptive field grow with depth for sliding window attention?** If w = 256 and
   L = 12, what is the maximum distance two tokens can interact through?

3. **Explain why BigBird is Turing complete despite sparse attention.** What role does each
   component (random, window, global) play in achieving this?

4. **Compare Longformer's dilated attention with standard strided attention.** What advantage
   does dilation provide over simple striding?

5. **Design a sparse attention pattern for a specific task:** You're building a chatbot that
   processes conversations up to 8,192 tokens. What combination of attention patterns would you
   use and why?

6. **What are the limitations of sparse attention methods?** In what scenarios might they
   underperform full attention, and how does FlashAttention change this landscape?

---

> **Navigation:**
> Previous: [← Swin Transformer](02-swin-transformer.md) | Next: [Linear Attention →](04-linear-attention.md)
