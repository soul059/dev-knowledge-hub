# Transformer Inference Optimization

> **Navigation:**
> Previous: [← Training Tricks](04-training-tricks.md) | Next: [BERT Architecture →](../07-BERT-and-Encoder-Models/01-bert-architecture.md)

---

## Overview

While training a Transformer is expensive, **inference** is where cost scales with every user request and every generated token. Autoregressive generation is inherently sequential — each new token depends on all previous tokens — making it memory-bandwidth-bound rather than compute-bound. This chapter covers the essential optimization techniques that make Transformer inference practical at scale: **KV caching** eliminates redundant computation, **speculative decoding** generates multiple tokens in parallel, **quantization** (INT8/INT4) shrinks model size by 2–8×, **pruning** removes unnecessary weights and attention heads, **knowledge distillation** trains faster student models, and **batching strategies** maximize GPU utilization. These techniques are critical for deploying models like GPT-4, LLaMA, and Claude in production.

---

## 1. KV Caching

### The Problem: Redundant Computation

During autoregressive generation, to produce token at position `t`, the model must compute attention over all previous positions `1, 2, ..., t-1`. Without caching, generating the next token requires recomputing all key and value projections for **every previous token**.

```
Without KV Cache:
  Generating token 5 → compute K,V for positions 1,2,3,4,5
  Generating token 6 → compute K,V for positions 1,2,3,4,5,6  (redundant!)
  Generating token 7 → compute K,V for positions 1,2,3,4,5,6,7 (redundant!)

Wasted computation grows as O(n²) for a sequence of length n.
```

### The Solution: Cache Key-Value Pairs

Store the K and V projections from all previous positions. For each new token, only compute Q, K, V for the **current** position and append K, V to the cache:

```
┌────────────────────────────────────────────────────────────┐
│                    KV CACHE MECHANISM                       │
│                                                            │
│  Step t: Generate token at position t                      │
│                                                            │
│  ┌─────────────┐                                           │
│  │ New token xₜ │                                          │
│  └──────┬──────┘                                           │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────┐                                          │
│  │ Compute qₜ,  │  Only for the NEW token                 │
│  │ kₜ, vₜ       │  (single position)                      │
│  └──────┬───────┘                                          │
│         │                                                  │
│    ┌────┴────┐                                             │
│    │ Append  │                                             │
│    ▼         ▼                                             │
│  ┌───────────────────────────────────┐                     │
│  │ K_cache = [k₁, k₂, ..., kₜ₋₁, kₜ] │  ← append kₜ    │
│  │ V_cache = [v₁, v₂, ..., vₜ₋₁, vₜ] │  ← append vₜ    │
│  └───────────────────┬───────────────┘                     │
│                      │                                     │
│  ┌───────────────────▼───────────────┐                     │
│  │ attention = softmax(qₜ · K_cacheᵀ │                    │
│  │             / √dₖ) · V_cache       │                   │
│  └────────────────────────────────────┘                    │
│                                                            │
│  Complexity per step: O(t · d) instead of O(t² · d)       │
└────────────────────────────────────────────────────────────┘
```

### Memory Cost of KV Cache

```
KV cache size = 2 × n_layers × n_heads × d_head × seq_len × batch_size × bytes_per_value

Example (LLaMA-7B, FP16):
  = 2 × 32 × 32 × 128 × 2048 × 1 × 2 bytes
  = 2 × 32 × 32 × 128 × 2048 × 2
  = 1,073,741,824 bytes ≈ 1 GB per sequence

For batch_size=32: 32 GB just for KV cache!
```

### Speedup Analysis

| Sequence Length | Without Cache (FLOPs) | With Cache (FLOPs per token) | Speedup |
|----------------|----------------------|------------------------------|---------|
| 128 | O(128²) = 16K | O(128) | 128× |
| 1024 | O(1024²) = 1M | O(1024) | 1024× |
| 4096 | O(4096²) = 16M | O(4096) | 4096× |

---

## 2. Speculative Decoding

### The Idea

Use a small, fast **draft model** to generate candidate tokens, then verify them in parallel with the large **target model**:

```
┌───────────────────────────────────────────────────────────────┐
│                    SPECULATIVE DECODING                        │
│                                                               │
│  Step 1: Draft model generates K candidate tokens quickly     │
│                                                               │
│  Draft (small):  "The" → "cat" → "sat" → "on" → "the"       │
│                                                               │
│  Step 2: Target model verifies ALL candidates in ONE pass     │
│                                                               │
│  Target (large): ["The", "cat", "sat", "on", "the"]          │
│                     ✓      ✓      ✓     ✓    ✗ → "a"         │
│                                                               │
│  Result: Accept "cat sat on", reject "the", use "a"          │
│  Generated 4 tokens in ~1 large model forward pass!           │
│                                                               │
│  Step 3: Continue from "The cat sat on a ..."                 │
└───────────────────────────────────────────────────────────────┘
```

### Why It Works

- The draft model is **10-100× faster** than the target model
- The target model can verify K tokens **in parallel** (single forward pass)
- Acceptance rate depends on draft–target agreement (typically 70-90%)
- The output distribution is **mathematically identical** to the target model

### Expected Speedup

```
Expected tokens per step = 1 + α + α² + α³ + ... + αᴷ

where α = acceptance rate, K = draft length

For α = 0.8, K = 5:
  Expected = 1 + 0.8 + 0.64 + 0.512 + 0.41 + 0.328 = 3.69 tokens/step
  Speedup ≈ 3.69× (minus draft model overhead)

Practical speedup: typically 2-3× for well-matched draft/target pairs
```

---

## 3. Quantization

### What Is Quantization?

Reducing the numerical precision of model weights from FP16/FP32 to INT8/INT4:

```
FP16 weight: 0.3847656 (16 bits)  →  INT8: 49 (8 bits)   →  INT4: 6 (4 bits)
```

### Quantization Formula

Linear quantization maps floating-point values to integers:

```
q = round(x / scale + zero_point)

where:
  scale = (max_val - min_val) / (2^bits - 1)
  zero_point = round(-min_val / scale)
```

### Dequantization (for computation)

```
x_approx = (q - zero_point) × scale
```

### Worked Example (INT8)

```
Weight tensor: [0.12, -0.45, 0.78, -0.23, 0.56]

min_val = -0.45, max_val = 0.78
scale = (0.78 - (-0.45)) / (255) = 1.23 / 255 = 0.004824
zero_point = round(0.45 / 0.004824) = round(93.28) = 93

Quantized values:
  0.12  → round(0.12 / 0.004824 + 93) = round(24.88 + 93) = round(117.88) = 118
  -0.45 → round(-0.45 / 0.004824 + 93) = round(-93.28 + 93) = round(-0.28) = 0
  0.78  → round(0.78 / 0.004824 + 93) = round(161.69 + 93) = round(254.69) = 255
  -0.23 → round(-0.23 / 0.004824 + 93) = round(-47.68 + 93) = round(45.32) = 45
  0.56  → round(0.56 / 0.004824 + 93) = round(116.09 + 93) = round(209.09) = 209

INT8 tensor: [118, 0, 255, 45, 209]

Dequantized (approximate):
  (118 - 93) × 0.004824 = 25 × 0.004824 = 0.1206  (original: 0.12)
  (0 - 93) × 0.004824   = -93 × 0.004824 = -0.4486 (original: -0.45)
  (255 - 93) × 0.004824 = 162 × 0.004824 = 0.7815  (original: 0.78)
```

### Quantization Methods Comparison

| Method | When | Accuracy Loss | Speed |
|--------|------|--------------|-------|
| **Post-Training Quantization (PTQ)** | After training, no data needed | Small (INT8), moderate (INT4) | Fast |
| **GPTQ** | After training, needs calibration data | Small for INT4 | Medium |
| **AWQ** | After training, activation-aware | Very small for INT4 | Medium |
| **Quantization-Aware Training (QAT)** | During training | Minimal | Slow (retraining) |
| **GGUF / llama.cpp** | After training, CPU-friendly | Varies by quant level | Fast |

### Memory Savings

```
Model: LLaMA-7B (7 billion parameters)

FP32:  7B × 4 bytes = 28 GB
FP16:  7B × 2 bytes = 14 GB
INT8:  7B × 1 byte  =  7 GB
INT4:  7B × 0.5 bytes = 3.5 GB

        FP32    FP16    INT8    INT4
        ████    ██      █       ▌
        28GB    14GB    7GB     3.5GB
```

---

## 4. Pruning

### Types of Pruning

```
┌─────────────────────────────────────────────────────────┐
│                    PRUNING STRATEGIES                     │
│                                                          │
│  1. Weight Pruning (Unstructured)                        │
│     Before: [0.5, 0.01, -0.8, 0.02, 0.6, -0.03]       │
│     After:  [0.5, 0.0,  -0.8, 0.0,  0.6,  0.0 ]       │
│     (Remove small-magnitude weights)                     │
│                                                          │
│  2. Head Pruning (Structured)                            │
│     Before: [Head1, Head2, Head3, Head4, Head5, Head6]  │
│     After:  [Head1, Head3, Head5, Head6]                │
│     (Remove entire attention heads)                      │
│                                                          │
│  3. Layer Pruning                                        │
│     Before: [L1, L2, L3, L4, L5, L6, ..., L32]         │
│     After:  [L1, L2, L4, L6, ..., L32]                  │
│     (Remove entire transformer layers)                   │
│                                                          │
│  4. Width Pruning                                        │
│     Before: FFN dim = 4096                               │
│     After:  FFN dim = 2048                               │
│     (Reduce hidden dimensions)                           │
└─────────────────────────────────────────────────────────-┘
```

### Attention Head Importance

Michel et al. (2019) showed that many attention heads can be removed with minimal accuracy loss:

```
  Accuracy
  100% │ ████████████████████████████████████
       │ ████████████████████████████████
   95% │ ████████████████████████████
       │ ██████████████████████████
   90% │ ██████████████████████
       │ ██████████████████
   85% │ ██████████████
       │
   80% │
       └──────────────────────────────────────► Heads Pruned
         0%   10%   20%   30%   40%   50%

  Many models retain >95% accuracy with 20-30% of heads pruned
```

---

## 5. Knowledge Distillation

### Overview

Train a smaller **student** model to mimic a larger **teacher** model:

```
┌──────────────┐         ┌──────────────────┐
│   Teacher     │         │   Student         │
│   (Large)     │         │   (Small)         │
│   e.g., 7B   │         │   e.g., 1.3B      │
│              │         │                   │
│  Input x ───►│logits_T │  Input x ─────►  │logits_S
│              │    │    │                   │   │
└──────────────┘    │    └──────────────────-┘   │
                    │                            │
                    ▼                            ▼
              soft targets ◄── KL Divergence ── soft predictions
                    │                            │
                    └───────── L_distill ────────┘
                                  +
                           L_student (CE with hard labels)
```

### Distillation Loss

```
L_total = α · L_distill + (1 - α) · L_student

L_distill = KL(softmax(logits_T / τ) ‖ softmax(logits_S / τ)) · τ²

where:
  τ = temperature (typically 2-20, higher = softer distributions)
  α = interpolation weight (typically 0.5-0.9)
```

### Why Temperature Matters

```
Teacher logits: [5.0, 2.0, 0.5, -1.0]

τ = 1:   softmax → [0.842, 0.042, 0.009, 0.002]  (peaked — hard to learn from)
τ = 3:   softmax → [0.447, 0.175, 0.117, 0.058]  (softer — more information)
τ = 10:  softmax → [0.296, 0.255, 0.241, 0.207]  (very soft — reveals structure)
```

Higher temperature exposes the teacher's "dark knowledge" — relative rankings among incorrect classes that contain useful information.

---

## 6. Batching Strategies

### Static Batching (Naive)

```
  Request 1: [████████████████████]  (long)
  Request 2: [████████]              (short)  ░░░░░░░░░░░ (padding — wasted)
  Request 3: [██████████████]        (medium) ░░░░░░░ (padding — wasted)

  All requests must wait for the longest one to finish.
  Padding wastes compute.
```

### Continuous Batching (vLLM, TGI)

```
  Time ──────────────────────────────────────────────────►
  Slot 1: [Req1 ████████████████████]
  Slot 2: [Req2 ████████][Req4 ██████████████]
  Slot 3: [Req3 ██████████████][Req5 ████████████]

  New requests fill slots as they become available.
  No padding waste. Much higher throughput.
```

### Comparison

| Strategy | GPU Utilization | Latency | Throughput |
|----------|-----------------|---------|------------|
| Static batching | 40-60% | High (wait for longest) | Low |
| Dynamic batching | 60-80% | Medium | Medium |
| **Continuous batching** | **80-95%** | **Low** | **High** |
| PagedAttention (vLLM) | 90-98% | Low | Very High |

---

## 7. PyTorch Implementation

### KV Cache Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math
from typing import Optional, Tuple


class CachedSelfAttention(nn.Module):
    """Self-attention with KV caching for efficient autoregressive generation."""

    def __init__(self, d_model: int, n_heads: int, max_seq_len: int = 4096):
        super().__init__()
        self.n_heads = n_heads
        self.d_head = d_model // n_heads

        self.W_q = nn.Linear(d_model, d_model, bias=False)
        self.W_k = nn.Linear(d_model, d_model, bias=False)
        self.W_v = nn.Linear(d_model, d_model, bias=False)
        self.W_o = nn.Linear(d_model, d_model, bias=False)

    def forward(
        self,
        x: torch.Tensor,
        kv_cache: Optional[Tuple[torch.Tensor, torch.Tensor]] = None,
        use_cache: bool = False,
    ) -> Tuple[torch.Tensor, Optional[Tuple[torch.Tensor, torch.Tensor]]]:
        """
        Args:
            x: Input tensor (B, T, D) where T=1 during generation with cache
            kv_cache: Tuple of (cached_K, cached_V), each (B, n_heads, T_prev, d_head)
            use_cache: Whether to return updated cache
        Returns:
            output: (B, T, D)
            new_kv_cache: Updated (K_cache, V_cache) if use_cache=True
        """
        B, T, D = x.shape

        # Project current input to Q, K, V
        Q = self.W_q(x).view(B, T, self.n_heads, self.d_head).transpose(1, 2)
        K = self.W_k(x).view(B, T, self.n_heads, self.d_head).transpose(1, 2)
        V = self.W_v(x).view(B, T, self.n_heads, self.d_head).transpose(1, 2)
        # Q, K, V: (B, n_heads, T, d_head)

        # Append to cache if available
        if kv_cache is not None:
            K_prev, V_prev = kv_cache
            K = torch.cat([K_prev, K], dim=2)  # (B, n_heads, T_prev + T, d_head)
            V = torch.cat([V_prev, V], dim=2)

        # Compute attention
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_head)

        # Apply causal mask (only needed during prefill, not single-token generation)
        if T > 1:
            mask = torch.triu(torch.ones(T, K.size(2), device=x.device), diagonal=K.size(2) - T + 1)
            scores = scores.masked_fill(mask.bool().unsqueeze(0).unsqueeze(0), float('-inf'))

        attn_weights = F.softmax(scores, dim=-1)
        output = torch.matmul(attn_weights, V)

        # Reshape and project
        output = output.transpose(1, 2).contiguous().view(B, T, D)
        output = self.W_o(output)

        new_cache = (K, V) if use_cache else None
        return output, new_cache


# --- Generation with KV Cache ---
@torch.no_grad()
def generate_with_kv_cache(model_layers, embedding, lm_head, input_ids,
                            max_new_tokens=50, temperature=1.0):
    """
    Autoregressive generation using KV caching.
    Each step only processes the LAST token through the model.
    """
    # Phase 1: Prefill — process entire prompt at once
    x = embedding(input_ids)  # (1, prompt_len, d_model)
    caches = [None] * len(model_layers)

    for i, layer in enumerate(model_layers):
        x, caches[i] = layer(x, kv_cache=None, use_cache=True)

    # Get first predicted token
    logits = lm_head(x[:, -1:, :])  # Only last position
    next_token = torch.multinomial(
        F.softmax(logits[:, -1, :] / temperature, dim=-1), 1
    )
    generated = [next_token.item()]

    # Phase 2: Decode — process one token at a time using cache
    for step in range(max_new_tokens - 1):
        x = embedding(next_token)  # (1, 1, d_model) — single token!

        for i, layer in enumerate(model_layers):
            x, caches[i] = layer(x, kv_cache=caches[i], use_cache=True)

        logits = lm_head(x)
        next_token = torch.multinomial(
            F.softmax(logits[:, -1, :] / temperature, dim=-1), 1
        )
        generated.append(next_token.item())

    return generated
```

### Simple Quantization Example

```python
import torch


def quantize_tensor_int8(tensor: torch.Tensor):
    """Quantize a floating-point tensor to INT8 with per-tensor scaling."""
    # Compute scale and zero point
    min_val = tensor.min().item()
    max_val = tensor.max().item()

    scale = (max_val - min_val) / 255.0  # INT8 unsigned: 0-255
    zero_point = round(-min_val / scale) if scale != 0 else 0
    zero_point = max(0, min(255, zero_point))

    # Quantize
    quantized = torch.clamp(
        torch.round(tensor / scale + zero_point), 0, 255
    ).to(torch.uint8)

    return quantized, scale, zero_point


def dequantize_tensor_int8(quantized, scale, zero_point):
    """Dequantize INT8 tensor back to floating point."""
    return (quantized.float() - zero_point) * scale


# --- Demo ---
if __name__ == "__main__":
    torch.manual_seed(42)

    # Create a sample weight tensor
    weights = torch.randn(4, 4) * 0.5
    print("Original weights:")
    print(weights)

    # Quantize
    q_weights, scale, zp = quantize_tensor_int8(weights)
    print(f"\nQuantized (INT8), scale={scale:.6f}, zero_point={zp}:")
    print(q_weights)

    # Dequantize
    deq_weights = dequantize_tensor_int8(q_weights, scale, zp)
    print(f"\nDequantized weights:")
    print(deq_weights)

    # Error analysis
    error = (weights - deq_weights).abs()
    print(f"\nMax quantization error:  {error.max().item():.6f}")
    print(f"Mean quantization error: {error.mean().item():.6f}")
    print(f"Memory: {weights.element_size() * weights.nelement()} bytes (FP32)")
    print(f"Memory: {q_weights.element_size() * q_weights.nelement()} bytes (INT8)")
    print(f"Compression: {weights.element_size() / q_weights.element_size():.0f}×")
```

---

## 8. Real-World Applications

| Technique | Used By | Impact |
|-----------|---------|--------|
| **KV Caching** | All production LLMs (GPT-4, Claude, LLaMA) | Reduces generation cost from O(n²) to O(n) per token |
| **Speculative Decoding** | Anthropic (Claude), Google (PaLM) | 2-3× faster generation with identical output quality |
| **INT4 Quantization (GPTQ)** | LLaMA.cpp, GGUF models, ExLlamaV2 | Run 7B models on consumer GPUs (6GB VRAM) |
| **Knowledge Distillation** | DistilBERT, TinyLLaMA, Phi-2 | 60% smaller models with 95%+ accuracy |
| **Continuous Batching** | vLLM, TGI (HuggingFace) | 2-4× throughput improvement in production |
| **PagedAttention** | vLLM | Near-zero memory waste for KV cache management |
| **Head Pruning** | Research → production pipelines | 20-30% heads removed with <1% accuracy loss |

---

## 9. Summary Table

| Technique | What It Does | Speedup | Memory Savings | Quality Impact |
|-----------|-------------|---------|----------------|----------------|
| KV Caching | Cache key-value pairs | 10-1000× per token | Costs memory | None |
| Speculative Decoding | Draft + verify | 2-3× | Minimal | None (exact) |
| INT8 Quantization | Reduce weight precision | 1.5-2× | 2× (vs FP16) | Minimal |
| INT4 Quantization | Aggressive precision reduction | 2-3× | 4× (vs FP16) | Small (1-3% accuracy) |
| Pruning (unstructured) | Zero out small weights | 1-1.5× | Up to 2× | Small |
| Pruning (structured) | Remove heads/layers | 1.5-3× | Proportional | Moderate |
| Knowledge Distillation | Train smaller model | Model-dependent | Model-dependent | 1-5% accuracy loss |
| Continuous Batching | Fill GPU slots dynamically | 2-4× throughput | Better utilization | None |

---

## 10. Revision Questions

1. **For a 13B parameter model with 40 layers, 40 heads, d_head=128, generating a 2048-token sequence in FP16 — how much memory does the KV cache require? Is this more or less than the model weights?**
   *Hint: KV cache = 2 × layers × heads × d_head × seq_len × 2 bytes.*

2. **In speculative decoding, why is it important that the draft model's token distribution overlaps significantly with the target model? What happens to the acceptance rate if they disagree frequently?**
   *Hint: Think about the rejection sampling step and how many draft tokens get wasted.*

3. **Explain why INT4 quantization works surprisingly well for large models (>7B parameters) but may degrade quality significantly for smaller models (<1B parameters).**
   *Hint: Consider redundancy in weight matrices and the relationship between model capacity and quantization error.*

4. **Compare the trade-offs between knowledge distillation and quantization as strategies for model compression. When would you prefer one over the other?**
   *Hint: Consider training cost, deployment constraints, and accuracy requirements.*

5. **Why is autoregressive Transformer inference memory-bandwidth-bound rather than compute-bound? How does this affect which optimizations are most impactful?**
   *Hint: Think about the arithmetic intensity (FLOPs per byte loaded) of generating one token.*

6. **PagedAttention (used in vLLM) applies the concept of virtual memory paging to KV cache management. Why does this matter for serving multiple concurrent requests with different sequence lengths?**
   *Hint: Consider memory fragmentation when different requests have different KV cache sizes.*

---

> **Navigation:**
> Previous: [← Training Tricks](04-training-tricks.md) | Next: [BERT Architecture →](../07-BERT-and-Encoder-Models/01-bert-architecture.md)
