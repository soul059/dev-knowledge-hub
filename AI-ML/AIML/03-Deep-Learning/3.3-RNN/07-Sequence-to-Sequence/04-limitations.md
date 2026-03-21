# Seq2Seq Limitations

> **Unit 7, Chapter 4** — Fixed context vector bottleneck, long sequence problems, computational cost, and comparison with attention-based models.

---

## 📋 Overview

Despite their versatility, vanilla seq2seq models have significant limitations. Understanding these motivates the development of attention mechanisms and ultimately Transformer architectures.

---

## ❌ Key Limitations

### 1. Fixed Context Vector Bottleneck

```
┌──────────────────────────────────────────────────────────────┐
│  ALL source information must fit in ONE vector of size n    │
│                                                              │
│  Source sentence (variable length):                          │
│  "The quick brown fox jumps over the lazy dog near the river"│
│   │    │      │    │    │     │    │    │   │    │    │      │
│   ▼    ▼      ▼    ▼    ▼     ▼    ▼    ▼   ▼    ▼    ▼      │
│  ┌────────────────────────────────────────────────────┐      │
│  │              ENCODER                                │      │
│  └──────────────────────┬─────────────────────────────┘      │
│                         ▼                                    │
│                    c ∈ ℝ²⁵⁶                                  │
│                  (256 numbers!)                               │
│                                                              │
│  12 words of meaning → 256 floating-point numbers           │
│  For longer texts (100+ words) → severe information loss    │
│                                                              │
│  Result: Performance DEGRADES as input length increases     │
└──────────────────────────────────────────────────────────────┘
```

### 2. Long Sequence Degradation

```
BLEU Score vs Source Sentence Length:

BLEU
  40│ ████
    │ █████
  30│ ██████
    │ ████████
  20│ █████████░░░
    │ █████████░░░░░░
  10│ █████████░░░░░░░░░░░░
    │ █████████░░░░░░░░░░░░░░░░░░
   0└────────────────────────────────
    0   10   20   30   40   50   60
         Source Sentence Length

Performance drops sharply after ~20-30 tokens!
(This was the key finding that motivated attention)
```

### 3. Sequential Processing (No Parallelism)

```
RNN encoding/decoding is inherently sequential:

   h₁ must be computed before h₂
   h₂ must be computed before h₃
   ...

   Time to encode: O(T_src) sequential steps
   Time to decode: O(T_trg) sequential steps
   Total: O(T_src + T_trg)

   CANNOT use GPU parallelism effectively!
   Contrast with Transformers: O(1) parallel depth for encoding
```

### 4. Exposure Bias

```
Training:  Decoder sees GROUND TRUTH inputs
Inference: Decoder sees ITS OWN predictions

At inference, one wrong prediction → all subsequent predictions
may be wrong because they condition on an incorrect prefix.
```

### 5. One-Directional Encoding

```
Standard encoder: left-to-right only
   "The bank by the river" → encoder processes left-to-right
   At "bank": only sees "The" → ambiguous!

Partial solution: Bidirectional encoder
   But decoder is still left-to-right (autoregressive)
```

---

## 📊 Attention Solves Most Bottleneck Issues

```
┌──────────────────┬──────────────────┬──────────────────────┐
│ Problem          │ Without Attention│ With Attention        │
├──────────────────┼──────────────────┼──────────────────────┤
│ Bottleneck       │ Single vector    │ Access ALL h_j       │
│ Long sequences   │ Severe degradation│ Graceful handling   │
│ Alignment        │ Implicit only    │ Explicit alignment   │
│ Interpretability │ Black box        │ Attention heatmaps   │
│ Gradient flow    │ Through c only   │ Direct connections   │
├──────────────────┼──────────────────┼──────────────────────┤
│ Still sequential │ Yes              │ Yes ← NOT solved!    │
│ Exposure bias    │ Yes              │ Yes ← NOT solved!    │
└──────────────────┴──────────────────┴──────────────────────┘

What DOES solve sequential processing → TRANSFORMERS
What helps exposure bias → Scheduled sampling, RL training
```

---

## 💻 Demonstrating the Bottleneck

```python
import torch
import torch.nn as nn

# Show how information compression increases with length
hidden_size = 256

# Short sequence (easy to compress)
short_info = 5 * hidden_size   # 5 tokens × 256 dims of info

# Long sequence (hard to compress)
long_info = 100 * hidden_size  # 100 tokens × 256 dims of info

compression_short = short_info / hidden_size  # 5:1 ratio
compression_long = long_info / hidden_size    # 100:1 ratio

print(f"=== Information Bottleneck ===")
print(f"Context vector size: {hidden_size}")
print(f"Short seq (5 tokens):  compression = {compression_short:.0f}:1")
print(f"Long seq (100 tokens): compression = {compression_long:.0f}:1")
print(f"\n100x more compression for 20x longer sequence!")
print(f"Information MUST be lost in the long case.")

# With attention: decoder accesses ALL hidden states
# Effective "context" at each step = weighted sum of ALL h_j
# No single bottleneck — information distributed
print(f"\nWith attention:")
print(f"  Short seq: accesses 5 × {hidden_size} = {5*hidden_size} values")
print(f"  Long seq:  accesses 100 × {hidden_size} = {100*hidden_size} values")
print(f"  No compression needed — scales linearly!")
```

---

## 📋 Summary Table

| Limitation | Impact | Solution |
|-----------|--------|----------|
| Context bottleneck | Information loss for long inputs | Attention mechanism |
| Long sequence degradation | BLEU drops with length | Attention, chunking |
| Sequential processing | Cannot parallelize | Transformers |
| Exposure bias | Error accumulation at inference | Scheduled sampling |
| Unidirectional encoding | Missing future context | Bidirectional encoder |
| Fixed vocabulary | Can't handle rare/new words | Subword tokenization (BPE) |

---

## ❓ Revision Questions

1. **Explain the information bottleneck. Why does a context vector of size 256 struggle with a 100-word sentence?**

2. **How does attention address the bottleneck? Does it completely solve it?**

3. **Why does seq2seq performance degrade with longer source sentences? Show with a diagram.**

4. **List three limitations that attention solves and two that it doesn't.**

5. **Why can't RNN-based seq2seq models be easily parallelized? How do Transformers solve this?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Beam Search Decoding](03-beam-search-decoding.md) |
| ➡️ Next | [Language Modeling](../08-RNN-Applications/01-language-modeling.md) |
| ⬆️ Unit Overview | [README](../README.md) |
