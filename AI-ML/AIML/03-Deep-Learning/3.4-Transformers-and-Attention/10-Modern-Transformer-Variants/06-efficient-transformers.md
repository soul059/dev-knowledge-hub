# Survey of Efficient Transformer Approaches

> **Navigation:**
> Previous: [← FlashAttention](05-flash-attention.md) | Next: [LLM Landscape →](../11-Large-Language-Models/01-llm-landscape.md)

---

## Overview

The quadratic complexity of standard self-attention has motivated a rich ecosystem of efficient
transformer methods, each trading off speed, memory, approximation quality, and implementation
complexity. This chapter provides a comprehensive survey and practical comparison of all major
approaches — sparse patterns, linear kernels, low-rank projections, memory-based methods, and
IO-aware exact algorithms — covered in the preceding chapters. We present a unified taxonomy,
empirical comparisons, and actionable recommendations for choosing the right method based on
your sequence length, task requirements, hardware constraints, and quality needs. Crucially,
FlashAttention has reshaped this landscape by making exact attention competitive in speed with
many approximate methods.

---

## 1. Taxonomy of Efficient Attention Methods

```
┌──────────────────────────────────────────────────────────────────────┐
│               EFFICIENT TRANSFORMER TAXONOMY                        │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────┐    ┌─────────────────────┐                 │
│  │   SPARSE PATTERNS   │    │   LINEAR / KERNEL   │                 │
│  │                     │    │                     │                 │
│  │ • Sliding window    │    │ • Linear Transformer│                 │
│  │ • Strided           │    │ • Performer (FAVOR+)│                 │
│  │ • Longformer        │    │ • Random Feature Attn│                │
│  │ • BigBird           │    │ • cosFormer          │                │
│  │ • Swin (vision)     │    │                     │                 │
│  └─────────────────────┘    └─────────────────────┘                 │
│                                                                      │
│  ┌─────────────────────┐    ┌─────────────────────┐                 │
│  │    LOW-RANK          │    │   MEMORY / CACHE    │                 │
│  │                     │    │                     │                 │
│  │ • Linformer         │    │ • Compressive Trans. │                │
│  │ • Nyströmformer     │    │ • Memorizing Trans.  │                │
│  │ • Set Transformer   │    │ • ∞-Former           │                │
│  │                     │    │ • Landmark Attention  │                │
│  └─────────────────────┘    └─────────────────────┘                 │
│                                                                      │
│  ┌─────────────────────┐    ┌─────────────────────┐                 │
│  │   IO-AWARE (EXACT)  │    │   ARCHITECTURAL     │                 │
│  │                     │    │                     │                 │
│  │ • FlashAttention    │    │ • Funnel Transformer │                │
│  │ • FlashAttention-2  │    │ • Pooling layers     │                │
│  │ • FlashAttention-3  │    │ • Token merging      │                │
│  │ • Memory-Efficient  │    │ • Early exit         │                │
│  │   Attention (xformers)│  │ • Mixture of Experts │                │
│  └─────────────────────┘    └─────────────────────┘                 │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 2. Method-by-Method Summary

### 2.1 Sparse Attention Methods

```
SLIDING WINDOW
    How: Each token attends to w nearest neighbors
    Complexity: O(n · w)
    Pros: Simple, good for local tasks
    Cons: Limited long-range without many layers
    Used in: Mistral, Longformer (as a component)

LONGFORMER (Beltagy et al., 2020)
    How: Sliding window + dilated + global tokens
    Complexity: O(n · w)
    Pros: Global info via [CLS], handles 4K+ tokens
    Cons: Custom CUDA kernels needed for speed
    Used in: Long document NLP tasks

BIGBIRD (Zaheer et al., 2020)
    How: Random + window + global attention
    Complexity: O(n)
    Pros: Turing complete, strong theoretical guarantees
    Cons: Random patterns reduce hardware efficiency
    Used in: Genomics, long-form QA
```

### 2.2 Linear / Kernel Methods

```
LINEAR TRANSFORMER (Katharopoulos et al., 2020)
    How: φ(x) = elu(x)+1 feature map, associativity trick
    Complexity: O(n · d²)
    Pros: Simple, fast, RNN-equivalent for causal
    Cons: Approximation quality can be poor
    Used in: Real-time / streaming applications

PERFORMER (Choromanski et al., 2021)
    How: FAVOR+ positive orthogonal random features
    Complexity: O(n · m · d)
    Pros: Tunable quality (via m), unbiased estimator
    Cons: Still approximate, overhead for feature maps
    Used in: Protein modeling, long sequences
```

### 2.3 Low-Rank Methods

```
LINFORMER (Wang et al., 2020)
    How: Project K, V to lower dimension k << n
    Formula: K̃ = E·K, Ṽ = F·V where E, F ∈ ℝ^(k × n)
    Complexity: O(n · k · d)
    Pros: Simple projection, preserves quality well
    Cons: Fixed k, projection matrices are large for very long n
    Used in: Text classification, moderate-length tasks

NYSTRÖMFORMER (Xiong et al., 2021)
    How: Nyström approximation of attention matrix
    Formula: Ã ≈ softmax(Q·K̃^T) · softmax(K̃·K̃^T)⁻¹ · softmax(K̃·V)
    Complexity: O(n · m · d)   (m = number of landmarks)
    Pros: Good approximation quality
    Cons: Matrix inversion can be unstable
```

### 2.4 IO-Aware Exact Methods

```
FLASHATTENTION (Dao et al., 2022)
    How: Tiling + fused kernels + online softmax
    Complexity: O(n²·d) FLOPs but O(n) MEMORY
    Pros: EXACT, 2-4× faster, standard in PyTorch 2.0+
    Cons: Requires CUDA, specific dtypes, no custom masks
    Used in: Everything (GPT-4, LLaMA, Stable Diffusion)

FLASHATTENTION-2 (Dao, 2023)
    How: Better parallelism, reduced non-matmul FLOPs
    Pros: ~72% GPU utilization (vs 35% for FA-1)
    Used in: State-of-the-art LLM training
```

---

## 3. Comprehensive Comparison Table

| Method             | Time         | Memory  | Exact? | Long Seq | Hardware | Quality |
|--------------------|-------------|---------|--------|----------|----------|---------|
| Standard Attention | O(n²d)      | O(n²)   | ✅ Yes  | ✗ Poor   | Any      | Best    |
| FlashAttention-2   | O(n²d)†     | O(n)    | ✅ Yes  | ✓ Good   | CUDA     | Best    |
| Sliding Window     | O(nwd)      | O(nw)   | ✅ Yes‡ | ~ Local  | Any      | Good    |
| Longformer         | O(nwd)      | O(nw)   | ✅ Yes‡ | ✓ Good   | CUDA     | V.Good  |
| BigBird            | O(n(r+w+g)d)| O(n)    | ✅ Yes‡ | ✓ Good   | CUDA     | V.Good  |
| Linformer          | O(nkd)      | O(nk)   | ✗ No   | ✓ Good   | Any      | Good    |
| Linear Transformer | O(nd²)      | O(d²)   | ✗ No   | ✓✓ Best  | Any      | Fair    |
| Performer          | O(nmd)      | O(nm)   | ✗ No   | ✓✓ Best  | Any      | Good    |
| Nyströmformer      | O(nmd)      | O(nm)   | ✗ No   | ✓ Good   | Any      | Good    |

```
† Same FLOPs as standard but 2-4× faster wall-clock (memory-bound → compute-bound)
‡ Exact within the defined attention pattern (but pattern itself is a design choice)
```

---

## 4. Decision Flowchart

```
                    WHICH ATTENTION METHOD?

                 ┌──────────────────┐
                 │ Sequence length? │
                 └────────┬─────────┘
                          │
              ┌───────────┼───────────┐
              │           │           │
         n < 2K      2K < n < 16K   n > 16K
              │           │           │
              ▼           ▼           │
         ┌────────┐  ┌──────────┐    │
         │Standard│  │ Is GPU   │    │
         │Attn w/ │  │available?│    │
         │Flash   │  └────┬─────┘    │
         │Attn-2  │       │          │
         └────────┘  ┌────┴────┐     │
                   Yes│       │No    │
                     ▼       ▼      │
              ┌──────────┐ ┌─────┐  │
              │FlashAttn │ │Long-│  ▼
              │    -2    │ │former│ ┌──────────────┐
              └──────────┘ │or   │ │Need exact    │
                           │Big  │ │attention?    │
                           │Bird │ └──────┬───────┘
                           └─────┘    Yes │     No
                                         ▼      ▼
                                   ┌──────┐ ┌──────────┐
                                   │Flash │ │Performer │
                                   │Attn-2│ │or Linear │
                                   │+slide│ │Transform.│
                                   │window│ │          │
                                   └──────┘ └──────────┘
```

---

## 5. Empirical Speed Comparison

```
WALL-CLOCK TIME vs SEQUENCE LENGTH (Forward Pass)
(Normalized to standard attention at n=512)

Time   │
(ms)   │
       │                                           ● Standard
1000   │                                         ●
       │                                       ●
       │                                     ●
       │                                   ●
 100   │                                ●
       │                  ▲ ▲ ▲ ▲ ▲ ▲ ▲    Longformer
       │              ▲ ▲
       │          ★ ★ ★ ★ ★ ★ ★ ★ ★ ★ ★    FlashAttention-2
  10   │      ★ ★
       │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○    Linear Transformer
       │○ ○
   1   │★
       └──────────────────────────────────────────
        512  1K   2K   4K   8K  16K  32K  64K
                     Sequence Length

Key observations:
  1. Standard attention: quadratic growth (unusable at 16K+)
  2. FlashAttention-2: same quadratic FLOPs but ~3× faster constant
  3. Longformer: linear growth but higher constant than Flash at short seq
  4. Linear Transformer: fastest but with quality trade-offs
```

---

## 6. Memory Comparison

```
PEAK MEMORY vs SEQUENCE LENGTH (Training, batch=1, d=64)

Memory │
(GB)   │
       │                                          ● Standard
  16   │                                        ●
       │
       │                                      ●
   8   │                                    ●
       │                                  ●
       │
   4   │                              ●
       │                          ●
       │
   2   │                      ●
       │      ▲ ▲ ▲ ▲ ▲ ▲ ▲ ▲ ▲ ▲ ▲ ▲       Longformer
   1   │  ★ ★ ★ ★ ★ ★ ★ ★ ★ ★ ★ ★ ★ ★      FlashAttention-2
       │○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○      Linear Transformer
  0.1  │
       └──────────────────────────────────────────
        512  1K   2K   4K   8K  16K  32K  64K
                     Sequence Length

Key observations:
  1. Standard: O(n²) memory — 16 GB+ at 64K
  2. FlashAttention-2: O(n) memory — manageable even at 64K
  3. Linear Transformer: O(d²) memory — nearly constant!
```

---

## 7. Quality Comparison

```
TASK PERFORMANCE vs METHOD (Relative to Standard Attention = 100%)

                    Classification  Translation  Long Doc QA  Retrieval
                    ──────────────  ───────────  ───────────  ─────────
Standard Attention      100%          100%         100%        100%
FlashAttention-2        100%          100%         100%        100%
Longformer               99%           97%         101%*        98%
BigBird                   99%           97%         102%*        97%
Linformer                 98%           95%          96%         94%
Performer                 97%           93%          95%         91%
Linear Transformer        95%           90%          92%         88%

* Better than standard on long documents because they CAN process them!
  Standard attention runs out of memory on documents >4K tokens.
```

---

## 8. Recent Trends and the FlashAttention Effect

```
THE LANDSCAPE SHIFT (2020 → 2024)

2020-2021: "We need approximate attention!"
    → Dozens of papers on sparse, linear, low-rank methods
    → Each claiming O(n) complexity
    → But: significant quality trade-offs, complex implementations

2022-2023: "FlashAttention makes exact attention fast!"
    → Exact attention with O(n) memory
    → 2-4× faster than naive implementation
    → Simple to use (just call PyTorch's SDPA)

2024+: "Best of both worlds"
    → FlashAttention for sequences up to ~32K  (sweet spot)
    → Sparse patterns (sliding window) for 32K-128K+ (Mistral, etc.)
    → FlashAttention + sliding window combined
    → Ring attention for distributed very long sequences

The key insight:
    Most "efficient attention" methods optimized for FLOP count
    but FlashAttention showed that MEMORY ACCESS PATTERNS matter more
    on modern GPUs.
```

---

## 9. Practical Recommendations

### 9.1 By Sequence Length

| Sequence Length | Recommended Approach                                         |
|-----------------|--------------------------------------------------------------|
| < 2,048         | Standard attention with FlashAttention-2                     |
| 2K – 8K        | FlashAttention-2 (exact, fast, simple)                       |
| 8K – 32K       | FlashAttention-2 + sliding window hybrid                     |
| 32K – 128K     | Sliding window + FlashAttention-2 + sparse global tokens     |
| > 128K         | Ring attention + sliding window (distributed)                |

### 9.2 By Task Type

| Task Type               | Recommended Approach                                    |
|--------------------------|--------------------------------------------------------|
| Classification           | FlashAttention-2 (quality matters, sequences short)    |
| Machine Translation      | FlashAttention-2 (quality sensitive, moderate length)  |
| Long Document QA         | Longformer or BigBird (need full document context)     |
| Code Generation          | FlashAttention-2 + sliding window (long contexts)      |
| Streaming / Real-time    | Linear Transformer (constant-time inference via RNN)   |
| Protein / Genomics       | BigBird or Performer (very long sequences, less exact) |
| Image Generation         | FlashAttention-2 (standard in diffusion models)        |

### 9.3 By Hardware

| Hardware                 | Recommended Approach                                    |
|--------------------------|--------------------------------------------------------|
| A100 / H100 GPU          | FlashAttention-2 (best utilization)                    |
| Consumer GPU (RTX 3090)  | FlashAttention-2 (enables larger models/sequences)     |
| CPU only                 | Linear Transformer or Longformer                       |
| TPU                      | Pallas FlashAttention or Performer                     |
| Edge / Mobile            | Linear Transformer (smallest memory footprint)         |

---

## 10. PyTorch Code: Benchmarking Different Approaches

```python
import torch
import torch.nn.functional as F
import time
import math


def benchmark_attention(method_name, attn_fn, q, k, v, num_runs=100):
    """Benchmark an attention function."""
    # Warmup
    for _ in range(10):
        _ = attn_fn(q, k, v)

    if q.is_cuda:
        torch.cuda.synchronize()

    start = time.perf_counter()
    for _ in range(num_runs):
        out = attn_fn(q, k, v)
    if q.is_cuda:
        torch.cuda.synchronize()
    elapsed = (time.perf_counter() - start) / num_runs * 1000

    return elapsed, out


# --- Standard Attention ---
def standard_attention(q, k, v):
    d = q.shape[-1]
    scores = torch.matmul(q, k.transpose(-2, -1)) / math.sqrt(d)
    attn = F.softmax(scores, dim=-1)
    return torch.matmul(attn, v)


# --- PyTorch SDPA (auto FlashAttention) ---
def sdpa_attention(q, k, v):
    return F.scaled_dot_product_attention(q, k, v)


# --- Linear Attention (elu+1) ---
def linear_attention(q, k, v):
    q = F.elu(q) + 1
    k = F.elu(k) + 1
    kv = torch.einsum('...nd,...ne->...de', k, v)
    z = 1.0 / (torch.einsum('...nd,...d->...n', q, k.sum(dim=-2)) + 1e-6)
    return torch.einsum('...nd,...de,...n->...ne', q, kv, z)


# --- Sliding Window Attention ---
def sliding_window_attention(q, k, v, window_size=256):
    B, H, N, D = q.shape
    d = q.shape[-1]
    scores = torch.matmul(q, k.transpose(-2, -1)) / math.sqrt(d)

    # Create window mask
    idx = torch.arange(N, device=q.device)
    mask = (idx.unsqueeze(0) - idx.unsqueeze(1)).abs() <= window_size // 2
    scores = scores.masked_fill(~mask.unsqueeze(0).unsqueeze(0), float('-inf'))

    attn = F.softmax(scores, dim=-1)
    return torch.matmul(attn, v)


# --- Run Benchmarks ---
if __name__ == "__main__":
    device = 'cuda' if torch.cuda.is_available() else 'cpu'
    dtype = torch.float16 if device == 'cuda' else torch.float32
    print(f"Device: {device}, Dtype: {dtype}\n")

    results = []

    for seq_len in [512, 1024, 2048, 4096]:
        B, H, D = 4, 8, 64
        q = torch.randn(B, H, seq_len, D, device=device, dtype=dtype)
        k = torch.randn(B, H, seq_len, D, device=device, dtype=dtype)
        v = torch.randn(B, H, seq_len, D, device=device, dtype=dtype)

        print(f"{'='*60}")
        print(f"Sequence Length: {seq_len}")
        print(f"{'='*60}")

        methods = {
            "Standard":       lambda q, k, v: standard_attention(q, k, v),
            "SDPA (Flash)":   lambda q, k, v: sdpa_attention(q, k, v),
            "Linear (elu+1)": lambda q, k, v: linear_attention(q, k, v),
            "Sliding (w=256)": lambda q, k, v: sliding_window_attention(q, k, v, 256),
        }

        for name, fn in methods.items():
            try:
                elapsed, out = benchmark_attention(name, fn, q, k, v, num_runs=50)
                mem = out.element_size() * out.nelement() / 1024
                print(f"  {name:20s}: {elapsed:8.2f} ms  |  output: {out.shape}")
                results.append((seq_len, name, elapsed))
            except Exception as e:
                print(f"  {name:20s}: FAILED ({e})")

        print()

    # Summary Table
    print(f"\n{'='*60}")
    print(f"SUMMARY: Time (ms) by Method and Sequence Length")
    print(f"{'='*60}")
    print(f"{'Method':20s} | {'512':>8s} | {'1024':>8s} | {'2048':>8s} | {'4096':>8s}")
    print(f"{'-'*20}-+-{'-'*8}-+-{'-'*8}-+-{'-'*8}-+-{'-'*8}")

    method_names = ["Standard", "SDPA (Flash)", "Linear (elu+1)", "Sliding (w=256)"]
    for method in method_names:
        row = f"{method:20s} |"
        for sl in [512, 1024, 2048, 4096]:
            matches = [r[2] for r in results if r[0] == sl and r[1] == method]
            if matches:
                row += f" {matches[0]:8.2f} |"
            else:
                row += f" {'N/A':>8s} |"
        print(row)
```

---

## 11. Summary Table

| Concept                    | Details                                                 |
|----------------------------|---------------------------------------------------------|
| Total categories           | 6: sparse, linear, low-rank, memory, IO-aware, arch.   |
| Most widely used (2024)    | FlashAttention-2 — exact, fast, built into PyTorch      |
| Best for very long seq     | Sliding window + global tokens                          |
| Best for streaming         | Linear Transformer (RNN mode)                           |
| Best quality               | FlashAttention-2 (exact, no approximation)              |
| Best memory efficiency     | Linear Transformer (O(d²) constant memory)              |
| Key insight                | Memory access patterns matter more than FLOP counts     |
| Modern trend               | Combine FlashAttention with sparse patterns              |
| Research direction         | Ring attention for distributed long context              |

---

## 12. Revision Questions

1. **Compare and contrast three categories of efficient attention.** For each, give one method,
   its complexity, and a scenario where it excels.

2. **Why did FlashAttention change the efficient attention landscape?** Explain why many
   approximate methods are now less necessary than they seemed in 2020-2021.

3. **You're building a chatbot that must handle 32K-token conversations.** Which combination of
   attention methods would you use, and why?

4. **Explain the trade-off between FLOP complexity and wall-clock speed.** Why can an O(n²)
   algorithm (FlashAttention) be faster than an O(n) algorithm (linear attention) in practice?

5. **Design an efficient attention strategy for a video understanding model** that processes
   128 frames of 224×224 images. Consider both spatial and temporal attention.

6. **What are the remaining challenges in efficient attention?** Discuss limitations of current
   methods and potential future directions (ring attention, hardware co-design, etc.).

---

> **Navigation:**
> Previous: [← FlashAttention](05-flash-attention.md) | Next: [LLM Landscape →](../11-Large-Language-Models/01-llm-landscape.md)
