# Parameter Efficiency in Multi-Head Attention

[← Head Concatenation](03-head-concatenation.md) | [Attention Is All You Need →](../04-Transformer-Architecture/01-attention-is-all-you-need.md)

---

## Overview

A remarkable property of multi-head attention is that it uses approximately the **same number of parameters** as a single-head attention mechanism with the full `d_model` dimension. Splitting into `h` heads does not multiply the parameter count — it redistributes the same budget across independent subspaces. This section provides a rigorous parameter count proof, analyses computational cost in FLOPs, explores the trade-offs between head count and per-head dimension, presents experimental evidence for choosing the optimal number of heads, and includes code to empirically verify parameter counts.

---

## 1. Parameter Count — The Proof

### Single-Head Attention Parameters

A single attention head operating on the full `d_model` dimension has:

```
W^Q:  d_model × d_model    →  d_model²
W^K:  d_model × d_model    →  d_model²
W^V:  d_model × d_model    →  d_model²

Total (single-head) = 3 × d_model²
```

No output projection needed (single head already produces `d_model`-dimensional output).

### Multi-Head Attention Parameters

With `h` heads, each projecting to `d_k = d_model / h`:

```
Per head i:
  Wᵢ^Q:  d_model × d_k     →  d_model × (d_model / h)  =  d_model² / h
  Wᵢ^K:  d_model × d_k     →  d_model² / h
  Wᵢ^V:  d_model × d_v     →  d_model² / h  (since d_v = d_k)

Per head subtotal = 3 × d_model² / h

All h heads = h × (3 × d_model² / h) = 3 × d_model²

Output projection:
  W^O:  (h × d_v) × d_model = d_model × d_model = d_model²

Total (multi-head) = 3 × d_model² + d_model² = 4 × d_model²
```

### Comparison

```
┌──────────────────────────────────────────────────────────────────┐
│                   Parameter Count Comparison                     │
│                                                                  │
│  Single-head:     3 × d_model²                                  │
│  Multi-head:      3 × d_model² + d_model²  =  4 × d_model²     │
│                                                                  │
│  The only extra cost is W^O (d_model²).                          │
│  The projection parameters are IDENTICAL in total.               │
│                                                                  │
│  For d_model = 512:                                              │
│    Single-head:   3 × 262,144  =    786,432 parameters          │
│    Multi-head:    4 × 262,144  =  1,048,576 parameters          │
│    Overhead:      262,144 parameters (+33%)                      │
│                                                                  │
│  But this +33% buys dramatically richer representations.         │
└──────────────────────────────────────────────────────────────────┘
```

### Why the Per-Head Parameters Sum to the Same

The key insight is that the packed projection matrix is equivalent:

```
Multi-head W^Q (packed) = [W₁^Q | W₂^Q | ... | Wₕ^Q]

Shape: d_model × (h × d_k) = d_model × d_model

This is exactly the same number of parameters as a single W^Q of shape
(d_model × d_model). The "splitting into heads" is just a reshape
operation — no parameters are added or removed.
```

---

## 2. Detailed Parameter Count for Standard Configurations

### Formula (without bias)

```
Total params = 3 × d_model² + d_model²  =  4 × d_model²
```

### Formula (with bias)

```
Total params = 3 × (d_model² + d_model) + (d_model² + d_model)
             = 4 × d_model² + 4 × d_model
             = 4 × d_model × (d_model + 1)
```

### Concrete Numbers

| Model | d_model | h | d_k | Params (no bias) | Params (with bias) |
|---|---|---|---|---|---|
| Transformer-Small | 256 | 4 | 64 | 262,144 | 263,168 |
| Transformer-Base | 512 | 8 | 64 | 1,048,576 | 1,050,624 |
| Transformer-Large | 1024 | 16 | 64 | 4,194,304 | 4,198,400 |
| GPT-2 Small | 768 | 12 | 64 | 2,359,296 | 2,362,368 |
| GPT-2 Medium | 1024 | 16 | 64 | 4,194,304 | 4,198,400 |
| GPT-2 Large | 1280 | 20 | 64 | 6,553,600 | 6,558,720 |
| GPT-3 175B | 12288 | 96 | 128 | 603,979,776 | 604,028,928 |
| LLaMA 7B | 4096 | 32 | 128 | 67,108,864 | 67,125,248 |

**Observation:** `d_k = 64` is remarkably common across many architectures — from the original Transformer to GPT-2. Larger models (GPT-3, LLaMA) use `d_k = 128`.

---

## 3. Computational Cost Analysis (FLOPs)

### Per-Layer FLOPs for Multi-Head Attention

For a sequence of length `n` and model dimension `d_model`:

```
Step 1: Projections (Q, K, V)
  3 × (n × d_model × d_model) multiplications
  = 3 × n × d_model²

Step 2: Attention scores  (per head: n × n × d_k,  all heads combined)
  h × (n × n × d_k) = h × n² × (d_model / h) = n² × d_model

Step 3: Weighted values  (per head: n × n × d_v,  all heads combined)
  n² × d_model

Step 4: Output projection
  n × d_model × d_model = n × d_model²

Total FLOPs ≈ 4 × n × d_model²  +  2 × n² × d_model
              \_________________/    \________________/
               Linear projections     Attention matrix
```

### ASCII Diagram: Cost Breakdown

```
  FLOPs Breakdown for d_model=512, n=512

  ┌────────────────────────────────────────────────────────┐
  │  Projections (Q,K,V,O)   ████████████████  ~69%       │
  │  4 × 512 × 512²                                       │
  │  = 536,870,912                                         │
  │                                                        │
  │  Attention (scores+ctx)  ████████         ~31%         │
  │  2 × 512² × 512                                       │
  │  = 268,435,456                                         │
  │                                                        │
  │  Total: ~805 MFLOPs per layer                          │
  └────────────────────────────────────────────────────────┘

  For longer sequences (n >> d_model), attention dominates:
  n = 2048:  Projections = 2.1 GFLOPs,  Attention = 4.3 GFLOPs
  n = 8192:  Projections = 8.6 GFLOPs,  Attention = 68.7 GFLOPs
```

**Key insight:** The number of heads `h` does **not** affect total FLOPs — only `n` and `d_model` matter. Each head does 1/h of the work, but there are `h` heads.

---

## 4. Trade-offs: More Heads vs. Larger Heads

### The Core Trade-off

```
Fixed budget: d_model = 512

Option A:  h = 4,   d_k = 128   →  4 heads, each with rich 128-dim subspace
Option B:  h = 8,   d_k = 64    →  8 heads, each with moderate 64-dim subspace
Option C:  h = 16,  d_k = 32    →  16 heads, each with small 32-dim subspace
Option D:  h = 64,  d_k = 8     →  64 heads, each with tiny 8-dim subspace
Option E:  h = 512, d_k = 1     →  512 heads, each with scalar attention (degenerate)
```

### Analysis

| Aspect | Fewer Heads (large d_k) | More Heads (small d_k) |
|---|---|---|
| **Subspace richness** | Each head has a richer representation space | Each head has a more constrained space |
| **Pattern diversity** | Fewer distinct attention patterns | More distinct attention patterns |
| **Rank of attention** | Higher rank per head | Lower rank per head (rank ≤ d_k) |
| **Risk of redundancy** | Lower (rich subspaces are less likely to overlap) | Higher (heads may learn similar patterns) |
| **Empirical optimum** | Usually not optimal | Usually h=8 or h=16 works best |

### ASCII Diagram: The Sweet Spot

```
  Model Quality (e.g. BLEU score)

  ▲
  │
  │           ●──●──●──●
  │         ●              ●
  │       ●                  ●
  │     ●                      ●
  │   ●                          ●
  │  ●                              ●
  │ ●
  │●
  └──┬───┬───┬───┬───┬───┬───┬───┬──► Number of heads (h)
     1   2   4   8  16  32  64  128

     ◄─ too few ─►◄─ sweet ─►◄── too many ──►
     heads: can't   spot:      heads: each head
     capture        diverse    too weak to capture
     diverse        yet rich   useful patterns
     patterns       enough
```

---

## 5. Experimental Evidence — Optimal Head Count

### Results from the Original Paper (Vaswani et al., 2017)

Table 3 from *"Attention Is All You Need"* — WMT 2014 EN-DE translation:

| h | d_k | d_model | BLEU | Δ from h=8 |
|---|---|---|---|---|
| 1 | 512 | 512 | 25.8 | -1.9 |
| 4 | 128 | 512 | 27.2 | -0.5 |
| 8 | 64 | 512 | **27.7** | baseline |
| 16 | 32 | 512 | 27.5 | -0.2 |
| 32 | 16 | 512 | 27.0 | -0.7 |

**Findings:**

1. Single head (`h=1`) performs worst — confirming the value of multi-head attention
2. `h=8` is optimal for `d_model=512`
3. Diminishing returns beyond `h=8`; `h=32` actually degrades performance
4. The sweet spot gives `d_k = 64`, which has become a standard

### Subsequent Research

| Study | Finding |
|---|---|
| Michel et al. (2019) | Many heads can be pruned at test time with minimal loss |
| Voita et al. (2019) | Only a few heads per layer are truly important |
| Shazeer (2019) | Multi-Query Attention: share K, V across heads to reduce memory |
| Ainslie et al. (2023) | Grouped-Query Attention (GQA): compromise between MHA and MQA |

---

## 6. Comparison Table — Head Configurations

```
┌──────────┬───────┬───────┬────────────┬────────────┬────────────────────────┐
│  Config  │   h   │  d_k  │  Proj.     │  W^O       │  Total                 │
│          │       │       │  Params    │  Params    │  Params                │
├──────────┼───────┼───────┼────────────┼────────────┼────────────────────────┤
│  A       │   1   │  512  │  786,432   │  262,144   │  1,048,576             │
│  B       │   4   │  128  │  786,432   │  262,144   │  1,048,576             │
│  C       │   8   │   64  │  786,432   │  262,144   │  1,048,576  ← default │
│  D       │  16   │   32  │  786,432   │  262,144   │  1,048,576             │
│  E       │  32   │   16  │  786,432   │  262,144   │  1,048,576             │
│  F       │  64   │    8  │  786,432   │  262,144   │  1,048,576             │
└──────────┴───────┴───────┴────────────┴────────────┴────────────────────────┘

  ALL configurations have IDENTICAL parameter counts!
  The choice of h is purely about representation quality, not parameter budget.
  (d_model = 512 for all rows)
```

---

## 7. Python Code — Counting and Verifying Parameters

```python
import torch
import torch.nn as nn

class MultiHeadAttention(nn.Module):
    """Multi-head attention for parameter counting."""

    def __init__(self, d_model, num_heads, bias=False):
        super().__init__()
        assert d_model % num_heads == 0
        self.W_q = nn.Linear(d_model, d_model, bias=bias)
        self.W_k = nn.Linear(d_model, d_model, bias=bias)
        self.W_v = nn.Linear(d_model, d_model, bias=bias)
        self.W_o = nn.Linear(d_model, d_model, bias=bias)

    def forward(self, x):
        pass  # Not needed for parameter counting


def count_mha_params(d_model, num_heads, bias=False):
    """Count parameters for multi-head attention."""
    model = MultiHeadAttention(d_model, num_heads, bias)
    total = sum(p.numel() for p in model.parameters())
    return total


def theoretical_params(d_model, bias=False):
    """Compute theoretical parameter count."""
    if bias:
        return 4 * d_model * (d_model + 1)
    return 4 * d_model * d_model


# ── Verify: parameter count is independent of h ──────────────────
print("=" * 70)
print(f"{'d_model':>8} {'h':>4} {'d_k':>5} {'Actual':>12} {'Theory':>12} {'Match':>6}")
print("=" * 70)

d_model = 512
for h in [1, 2, 4, 8, 16, 32, 64]:
    d_k = d_model // h
    actual = count_mha_params(d_model, h, bias=False)
    theory = theoretical_params(d_model, bias=False)
    match = "✓" if actual == theory else "✗"
    print(f"{d_model:>8} {h:>4} {d_k:>5} {actual:>12,} {theory:>12,} {match:>6}")

print()

# ── Compare across model sizes ───────────────────────────────────
print("=" * 70)
print(f"{'Model':>20} {'d_model':>8} {'h':>4} {'Params':>14} {'% of 175B':>10}")
print("=" * 70)

configs = [
    ("Transformer-Base",    512,   8),
    ("GPT-2 Small",         768,  12),
    ("GPT-2 Medium",       1024,  16),
    ("GPT-2 Large",        1280,  20),
    ("GPT-2 XL",           1600,  25),
    ("GPT-3 175B",        12288,  96),
    ("LLaMA-7B",           4096,  32),
    ("LLaMA-65B",          8192,  64),
]

gpt3_params = theoretical_params(12288)

for name, d, h in configs:
    params = theoretical_params(d)
    pct = (params / gpt3_params) * 100
    print(f"{name:>20} {d:>8} {h:>4} {params:>14,} {pct:>9.4f}%")
```

### Expected Output

```
======================================================================
 d_model    h   d_k       Actual       Theory  Match
======================================================================
     512    1   512    1,048,576    1,048,576      ✓
     512    2   256    1,048,576    1,048,576      ✓
     512    4   128    1,048,576    1,048,576      ✓
     512    8    64    1,048,576    1,048,576      ✓
     512   16    32    1,048,576    1,048,576      ✓
     512   32    16    1,048,576    1,048,576      ✓
     512   64     8    1,048,576    1,048,576      ✓

======================================================================
               Model  d_model    h         Params   % of 175B
======================================================================
    Transformer-Base      512    8      1,048,576    0.1736%
        GPT-2 Small      768   12      2,359,296    0.3906%
       GPT-2 Medium     1024   16      4,194,304    0.6944%
        GPT-2 Large     1280   20      6,553,600    1.0851%
          GPT-2 XL     1600   25     10,240,000    1.6954%
        GPT-3 175B    12288   96    603,979,776  100.0000%
         LLaMA-7B     4096   32     67,108,864   11.1111%
        LLaMA-65B     8192   64    268,435,456   44.4444%
```

---

## 8. FLOP Counting Code

```python
def count_flops(d_model, seq_len, num_heads):
    """Count approximate FLOPs for one multi-head attention layer.

    Counts multiply-accumulate operations (MACs) × 2 for FLOPs.
    """
    # Projections: Q, K, V, O — each is a (seq, d_model) × (d_model, d_model) matmul
    proj_flops = 4 * (2 * seq_len * d_model * d_model)

    # Attention scores: Q·Kᵀ — (seq, d_k) × (d_k, seq) per head, h heads
    # Equivalent to: 2 * seq * seq * d_model  (since h × d_k = d_model)
    score_flops = 2 * seq_len * seq_len * d_model

    # Context: attn · V — (seq, seq) × (seq, d_k) per head, h heads
    context_flops = 2 * seq_len * seq_len * d_model

    total = proj_flops + score_flops + context_flops
    return {
        "projections": proj_flops,
        "attention":   score_flops + context_flops,
        "total":       total,
    }


# ── FLOP analysis for different sequence lengths ─────────────────
print(f"\n{'seq_len':>8} {'Proj GFLOPs':>14} {'Attn GFLOPs':>14} "
      f"{'Total GFLOPs':>14} {'Attn %':>8}")
print("-" * 62)

d_model = 512
for n in [64, 128, 256, 512, 1024, 2048, 4096, 8192]:
    f = count_flops(d_model, n, 8)
    proj_g  = f["projections"] / 1e9
    attn_g  = f["attention"] / 1e9
    total_g = f["total"] / 1e9
    attn_pct = (f["attention"] / f["total"]) * 100
    print(f"{n:>8} {proj_g:>14.3f} {attn_g:>14.3f} "
          f"{total_g:>14.3f} {attn_pct:>7.1f}%")
```

### Expected Output

```
 seq_len  Proj GFLOPs   Attn GFLOPs  Total GFLOPs   Attn %
--------------------------------------------------------------
      64          0.067          0.004          0.071     5.9%
     128          0.134          0.017          0.151    11.1%
     256          0.268          0.067          0.336    20.0%
     512          0.537          0.268          0.805    33.3%
    1024          1.074          1.074          2.147    50.0%
    2048          2.147          4.295          6.442    66.7%
    4096          4.295         17.180         21.475    80.0%
    8192          8.590         68.719         77.309    88.9%
```

**Key insight:** For short sequences, projection cost dominates. For long sequences, the O(n²) attention cost takes over — this is why efficient attention variants (Flash Attention, linear attention) focus on reducing this component.

---

## 9. Multi-Query and Grouped-Query Attention

Modern architectures reduce parameters further by sharing K, V projections:

### Parameter Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                 Attention Variant Parameters                        │
│                 (d_model = 4096, h = 32)                            │
│                                                                     │
│  Standard MHA:                                                      │
│    Q: 4096² = 16M                                                   │
│    K: 4096² = 16M       (32 separate K projections)                 │
│    V: 4096² = 16M       (32 separate V projections)                 │
│    O: 4096² = 16M                                                   │
│    Total: 64M params                                                │
│                                                                     │
│  Multi-Query (MQA):                                                 │
│    Q: 4096² = 16M                                                   │
│    K: 4096 × 128 = 0.5M  (1 shared K projection)                   │
│    V: 4096 × 128 = 0.5M  (1 shared V projection)                   │
│    O: 4096² = 16M                                                   │
│    Total: 33M params     ← 48% reduction!                           │
│                                                                     │
│  Grouped-Query (GQA, g=4):                                          │
│    Q: 4096² = 16M                                                   │
│    K: 4096 × (4×128) = 2M  (4 group K projections)                 │
│    V: 4096 × (4×128) = 2M  (4 group V projections)                 │
│    O: 4096² = 16M                                                   │
│    Total: 36M params     ← 44% reduction, better quality than MQA   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 10. Real-World Applications

| Application | Parameter Efficiency Consideration |
|---|---|
| **Edge deployment (mobile)** | Fewer heads + smaller d_model to fit memory constraints |
| **LLM serving** | Multi-Query Attention reduces KV-cache memory by ~h× |
| **Knowledge distillation** | Student models use fewer heads but preserve critical ones |
| **Head pruning** | Remove redundant heads post-training for 20-40% speedup |
| **Architecture search** | h and d_k are key hyperparameters in NAS for Transformers |
| **Quantisation** | Per-head quantisation allows different precision per head |

---

## 11. Summary Table

| Concept | Detail |
|---|---|
| Total MHA params | `4 × d_model²` (without bias) |
| Params independent of h | Changing h redistributes the same parameter budget |
| Only extra cost | Output projection W^O adds `d_model²` vs single-head |
| Standard d_k | 64 (base/large models) or 128 (very large models) |
| FLOP formula | `4n·d² + 2n²·d` where d=d_model, n=seq_len |
| Attention bottleneck | O(n²) dominates for long sequences |
| Optimal h | Typically `d_model / 64` (gives d_k=64) |
| MQA savings | ~50% fewer attention parameters |
| GQA savings | ~40-45% fewer parameters, better quality than MQA |

---

## 12. Revision Questions

1. **Prove that multi-head attention with h heads has the same number of projection parameters as single-head attention. Where does the extra cost come from?**
   *Hint: Each head has d_model × d_k parameters; sum over h heads gives d_model². The extra is W^O.*

2. **For d_model=1024 and h=16, calculate the exact number of parameters (no bias). Verify with the formula 4 × d_model².**
   *Answer: 4 × 1024² = 4,194,304.*

3. **At what sequence length does the attention cost (O(n²d)) equal the projection cost (O(nd²)) for d_model=512?**
   *Hint: Solve 2n²d = 4nd² → n = 2d = 1024.*

4. **Why does increasing the number of heads beyond a certain point hurt performance, even though parameter count stays the same?**
   *Hint: Each head's d_k becomes too small to represent useful patterns. A d_k=1 head computes scalar attention.*

5. **Explain how Multi-Query Attention achieves ~50% parameter reduction. What is the trade-off?**
   *Hint: Shares K, V across all heads. Trade-off: slightly lower quality, but much faster inference.*

6. **A model has d_model=768, h=12, and includes bias terms in all projections. How many total parameters does one multi-head attention layer have?**
   *Answer: 4 × 768 × (768 + 1) = 4 × 768 × 769 = 2,362,368.*

---

[← Head Concatenation](03-head-concatenation.md) | [Attention Is All You Need →](../04-Transformer-Architecture/01-attention-is-all-you-need.md)
