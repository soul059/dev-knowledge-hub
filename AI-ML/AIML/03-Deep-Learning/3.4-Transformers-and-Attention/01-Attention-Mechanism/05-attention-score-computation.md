# Attention Score Functions

[← Previous: Query, Key, Value](04-query-key-value.md) | [Next: Scaled Dot-Product Attention →](06-scaled-dot-product-attention.md)

---

## Overview

The attention mechanism relies on a **scoring function** that quantifies the relevance between a query and each key in a set. This score determines how much "attention" the model pays to each value when producing its output. There are three foundational scoring functions used across the literature: **dot-product**, **general (bilinear)**, and **additive (concat)**. Each offers a different trade-off between computational efficiency, expressiveness, and the number of learnable parameters. Understanding these functions is essential because the choice of scoring method directly impacts model capacity, training speed, and the types of relationships the attention mechanism can capture. This chapter provides the full mathematical formulation, numerical examples, and PyTorch implementations for all three.

---

## 1. Dot-Product Scoring

### Formula

$$
\text{score}(q, k) = q^\top k = \sum_{i=1}^{d} q_i \cdot k_i
$$

Where:
- `q` ∈ ℝ^d — the query vector
- `k` ∈ ℝ^d — the key vector
- `d` — the dimensionality of both vectors

### Properties

- **No learnable parameters** — purely geometric similarity
- Measures the cosine-like alignment between query and key
- Computationally the cheapest: O(d) per score
- Requires query and key to have the **same dimensionality**

### ASCII Diagram

```
  q = [q₁, q₂, ..., q_d]
       ↓   ↓        ↓         Element-wise multiply
  k = [k₁, k₂, ..., k_d]
       ↓   ↓        ↓
      [q₁k₁, q₂k₂, ..., q_dk_d]
              ↓
         Σ (sum all)
              ↓
        scalar score
```

### Worked Example

```
q = [2, 1, 3]      k₁ = [1, 0, 1]      k₂ = [0, 3, 0]

score(q, k₁) = (2)(1) + (1)(0) + (3)(1) = 2 + 0 + 3 = 5
score(q, k₂) = (2)(0) + (1)(3) + (3)(0) = 0 + 3 + 0 = 3

→ k₁ is more relevant to q (score 5 > 3)
```

---

## 2. General (Bilinear) Scoring

### Formula

$$
\text{score}(q, k) = q^\top W k
$$

Where:
- `q` ∈ ℝ^{d_q} — query vector
- `k` ∈ ℝ^{d_k} — key vector
- `W` ∈ ℝ^{d_q × d_k} — learnable weight matrix

### Properties

- **Learnable parameters**: d_q × d_k (the matrix W)
- Allows query and key to have **different dimensionalities**
- W learns which dimensions of q should interact with which dimensions of k
- More expressive than dot-product but more expensive

### How It Works

The matrix W acts as a learned transformation that defines how query dimensions relate to key dimensions:

```
           d_k
      ┌──────────┐
      │          │
d_q   │    W     │   ← Learnable weight matrix
      │          │
      └──────────┘

Step 1:  Wk = W · k        → produces a d_q-dimensional vector
Step 2:  q^T · (Wk)        → dot product in query space → scalar
```

### ASCII Diagram

```
  q ∈ ℝ^{d_q}          k ∈ ℝ^{d_k}
       │                     │
       │              ┌──────┴──────┐
       │              │  W (d_q×d_k)│
       │              └──────┬──────┘
       │                     │
       │               Wk ∈ ℝ^{d_q}
       │                     │
       └────── dot product ──┘
                    │
              scalar score
```

### Worked Example

```
q = [2, 1]      k = [1, 0, 1]

W = [[1, 0, 2],
     [0, 1, 1]]       (2×3 matrix)

Step 1: Compute Wk
  Wk = W · k = [1·1 + 0·0 + 2·1,  0·1 + 1·0 + 1·1]
             = [3, 1]

Step 2: Compute q^T · Wk
  score = q^T · Wk = 2·3 + 1·1 = 6 + 1 = 7
```

---

## 3. Additive (Concat) Scoring

### Formula

$$
\text{score}(q, k) = v^\top \tanh(W [q; k])
$$

Where:
- `[q; k]` ∈ ℝ^{d_q + d_k} — concatenation of q and k
- `W` ∈ ℝ^{d_h × (d_q + d_k)} — learnable weight matrix
- `v` ∈ ℝ^{d_h} — learnable weight vector
- `d_h` — hidden dimension (hyperparameter)
- `tanh` — element-wise non-linearity

### Properties

- **Learnable parameters**: d_h × (d_q + d_k) + d_h
- Most expressive — can learn non-linear relationships
- Computationally most expensive of the three
- Originally proposed by Bahdanau et al. (2014)
- The `tanh` introduces non-linearity, enabling richer score functions

### ASCII Diagram

```
  q ∈ ℝ^{d_q}     k ∈ ℝ^{d_k}
       │                │
       └──── concat ────┘
                │
         [q; k] ∈ ℝ^{d_q + d_k}
                │
       ┌────────┴────────┐
       │  W (d_h × (d_q+d_k)) │
       └────────┬────────┘
                │
          ℝ^{d_h} vector
                │
            tanh( · )
                │
          ℝ^{d_h} vector
                │
       ┌────────┴────────┐
       │   v^T (1 × d_h) │
       └────────┬────────┘
                │
          scalar score
```

### Worked Example

```
q = [2, 1]      k = [1, 3]      d_h = 3

[q; k] = [2, 1, 1, 3]    (concatenation)

W = [[1, 0, -1,  1],
     [0, 1,  1,  0],
     [1, 1,  0, -1]]     (3×4 matrix)

v = [1, -1, 1]            (3-dim vector)

Step 1: W · [q; k]
  = [1·2 + 0·1 + (-1)·1 + 1·3,
     0·2 + 1·1 + 1·1 + 0·3,
     1·2 + 1·1 + 0·1 + (-1)·3]
  = [2 + 0 - 1 + 3,
     0 + 1 + 1 + 0,
     2 + 1 + 0 - 3]
  = [4, 2, 0]

Step 2: tanh([4, 2, 0])
  = [0.9993, 0.9640, 0.0000]

Step 3: v^T · tanh(...)
  = 1·0.9993 + (-1)·0.9640 + 1·0.0000
  = 0.9993 - 0.9640 + 0.0000
  = 0.0353
```

---

## Comparison Table

| Property               | Dot-Product         | General (Bilinear)     | Additive (Concat)         |
|------------------------|---------------------|------------------------|---------------------------|
| **Formula**            | q^T k               | q^T W k                | v^T tanh(W[q;k])         |
| **Parameters**         | 0                   | d_q × d_k              | d_h(d_q + d_k) + d_h     |
| **Complexity**         | O(d)                | O(d_q · d_k)           | O(d_h · (d_q + d_k))     |
| **Non-linearity**      | No                  | No                     | Yes (tanh)                |
| **Same dim required?** | Yes                 | No                     | No                        |
| **Expressiveness**     | Low                 | Medium                 | High                      |
| **Speed**              | Fastest             | Medium                 | Slowest                   |
| **GPU parallelism**    | Excellent           | Good                   | Moderate                  |
| **Used in**            | Transformers        | Luong Attention        | Bahdanau Attention        |

---

## When to Use Which Scoring Function

### Use Dot-Product When:
- Query and key have the same dimensionality
- Speed is critical (e.g., Transformers with long sequences)
- You want maximum GPU parallelism via batched matrix multiplication
- The model has other capacity (e.g., multi-head attention, feed-forward layers)

### Use General/Bilinear When:
- Query and key have different dimensionalities
- You need a learnable interaction but want to stay linear
- Moderate computational budget is available
- Used in Luong-style attention for machine translation

### Use Additive/Concat When:
- You need the most expressive scoring function
- Working with smaller models where the score function must carry more capacity
- Historical or compatibility reasons (Bahdanau-style seq2seq)
- The non-linearity from `tanh` is beneficial for the task

---

## Computation Flow Diagram

```
                    ┌─────────────────────────────────────┐
                    │      ATTENTION SCORE COMPUTATION     │
                    └─────────────────────────────────────┘

    Query (q)                                    Keys (k₁, k₂, ..., kₙ)
       │                                              │
       │          ┌────────────────────────────────────┤
       │          │              │            │        │
       ▼          ▼              ▼            ▼        ▼
   ┌──────────────────────────────────────────────────────┐
   │              Scoring Function (choose one)           │
   │                                                      │
   │  Dot:     s_j = q^T k_j                             │
   │  General: s_j = q^T W k_j                           │
   │  Concat:  s_j = v^T tanh(W[q; k_j])                │
   └──────────────────┬───────────────────────────────────┘
                      │
              [s₁, s₂, ..., sₙ]   ← raw scores
                      │
                  softmax( · )
                      │
              [α₁, α₂, ..., αₙ]   ← attention weights (sum to 1)
                      │
              Σ αⱼ · vⱼ           ← weighted sum of values
                      │
                context vector
```

---

## PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import time


class DotProductScoring(nn.Module):
    """Dot-product scoring: score(q, k) = q^T k"""

    def forward(self, query, keys):
        """
        Args:
            query: (batch, d)
            keys:  (batch, seq_len, d)
        Returns:
            scores: (batch, seq_len)
        """
        # Einstein summation: batch dot product over dimension d
        return torch.einsum('bd,bsd->bs', query, keys)


class GeneralScoring(nn.Module):
    """General/Bilinear scoring: score(q, k) = q^T W k"""

    def __init__(self, d_query, d_key):
        super().__init__()
        self.W = nn.Linear(d_key, d_query, bias=False)

    def forward(self, query, keys):
        """
        Args:
            query: (batch, d_query)
            keys:  (batch, seq_len, d_key)
        Returns:
            scores: (batch, seq_len)
        """
        # Transform keys: (batch, seq_len, d_key) -> (batch, seq_len, d_query)
        transformed = self.W(keys)
        # Dot product with query
        return torch.einsum('bd,bsd->bs', query, transformed)


class AdditiveScoring(nn.Module):
    """Additive/Concat scoring: score(q, k) = v^T tanh(W[q; k])"""

    def __init__(self, d_query, d_key, d_hidden):
        super().__init__()
        self.W = nn.Linear(d_query + d_key, d_hidden, bias=False)
        self.v = nn.Linear(d_hidden, 1, bias=False)

    def forward(self, query, keys):
        """
        Args:
            query: (batch, d_query)
            keys:  (batch, seq_len, d_key)
        Returns:
            scores: (batch, seq_len)
        """
        seq_len = keys.size(1)

        # Expand query to match keys: (batch, seq_len, d_query)
        query_expanded = query.unsqueeze(1).expand(-1, seq_len, -1)

        # Concatenate: (batch, seq_len, d_query + d_key)
        combined = torch.cat([query_expanded, keys], dim=-1)

        # Apply W and tanh: (batch, seq_len, d_hidden)
        hidden = torch.tanh(self.W(combined))

        # Apply v to get scalar scores: (batch, seq_len, 1) -> (batch, seq_len)
        scores = self.v(hidden).squeeze(-1)
        return scores


# ── Demo and Verification ──────────────────────────────────────────

def demo_scoring_functions():
    torch.manual_seed(42)

    batch_size = 2
    seq_len = 5
    d_model = 8

    query = torch.randn(batch_size, d_model)
    keys = torch.randn(batch_size, seq_len, d_model)

    # Dot-product
    dot_scorer = DotProductScoring()
    dot_scores = dot_scorer(query, keys)
    print("Dot-Product Scores:")
    print(dot_scores)
    print(f"  Shape: {dot_scores.shape}\n")

    # General
    gen_scorer = GeneralScoring(d_query=d_model, d_key=d_model)
    gen_scores = gen_scorer(query, keys)
    print("General (Bilinear) Scores:")
    print(gen_scores)
    print(f"  Shape: {gen_scores.shape}\n")

    # Additive
    add_scorer = AdditiveScoring(d_query=d_model, d_key=d_model, d_hidden=16)
    add_scores = add_scorer(query, keys)
    print("Additive (Concat) Scores:")
    print(add_scores)
    print(f"  Shape: {add_scores.shape}\n")

    # Convert any scores to attention weights
    weights = F.softmax(dot_scores, dim=-1)
    print("Attention Weights (from dot-product):")
    print(weights)
    print(f"  Sum per row: {weights.sum(dim=-1)}")


# ── Benchmark ───────────────────────────────────────────────────────

def benchmark_scoring_functions():
    torch.manual_seed(0)
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

    batch_size = 64
    seq_len = 512
    d_model = 256
    d_hidden = 256
    n_runs = 100

    query = torch.randn(batch_size, d_model, device=device)
    keys = torch.randn(batch_size, seq_len, d_model, device=device)

    scorers = {
        'Dot-Product': DotProductScoring().to(device),
        'General':     GeneralScoring(d_model, d_model).to(device),
        'Additive':    AdditiveScoring(d_model, d_model, d_hidden).to(device),
    }

    print(f"\nBenchmark: batch={batch_size}, seq_len={seq_len}, "
          f"d={d_model}, runs={n_runs}")
    print(f"Device: {device}\n")
    print(f"{'Method':<15} {'Time (ms)':<12} {'Params':<10}")
    print("-" * 37)

    for name, scorer in scorers.items():
        n_params = sum(p.numel() for p in scorer.parameters())

        # Warm up
        for _ in range(5):
            _ = scorer(query, keys)

        if device.type == 'cuda':
            torch.cuda.synchronize()

        start = time.perf_counter()
        for _ in range(n_runs):
            _ = scorer(query, keys)
        if device.type == 'cuda':
            torch.cuda.synchronize()
        elapsed = (time.perf_counter() - start) / n_runs * 1000

        print(f"{name:<15} {elapsed:<12.3f} {n_params:<10}")


if __name__ == "__main__":
    demo_scoring_functions()
    benchmark_scoring_functions()
```

---

## Real-World Applications

### 1. Machine Translation (Luong Attention)
The general scoring function allows the decoder hidden state (query) to learn a flexible alignment with encoder outputs (keys) via the weight matrix W.

### 2. Transformer Models
Modern Transformers use the dot-product score (scaled) because it enables highly parallel computation through batched matrix multiplication — essential for processing long sequences efficiently.

### 3. Speech Recognition
Additive attention is used in end-to-end speech models where the acoustic encoder and text decoder may operate in very different representation spaces, requiring the non-linear scoring capacity.

### 4. Image Captioning
The scoring function computes relevance between the current word being generated (query) and spatial regions of the image (keys), enabling the model to "look" at relevant parts of the image for each word.

### 5. Document Summarization
Scoring functions determine which sentences in a long document are most relevant to the current summary state, directly controlling content selection.

---

## Summary Table

| Concept                    | Key Point                                                |
|----------------------------|----------------------------------------------------------|
| Scoring function purpose   | Computes relevance between query and key                 |
| Dot-product                | Simplest, no params, requires same dimensions            |
| General/Bilinear           | Adds learnable W matrix, allows different dimensions     |
| Additive/Concat            | Most expressive, uses tanh non-linearity                 |
| After scoring              | Scores → softmax → attention weights → weighted sum      |
| Dominant in practice       | Scaled dot-product (Transformers)                        |
| Choice depends on          | Speed vs. expressiveness vs. dimension compatibility     |

---

## Revision Questions

**Q1.** Write the formula for each of the three scoring functions. Which one has zero learnable parameters?

**Q2.** If the query dimension is 128 and the key dimension is 256, which scoring function(s) can be used without modification? Which requires equal dimensions?

**Q3.** Compute the dot-product attention score for q = [1, -2, 3] and k = [4, 1, -1]. Show your work.

**Q4.** In the additive scoring function, what is the role of the `tanh` non-linearity? What would happen if it were removed?

**Q5.** Why is dot-product scoring preferred in Transformers despite being less expressive than additive scoring? Discuss computational considerations.

**Q6.** Given d_query = 64, d_key = 64, and d_hidden = 128, calculate the exact number of learnable parameters for the general and additive scoring functions.

---

[← Previous: Query, Key, Value](04-query-key-value.md) | [Next: Scaled Dot-Product Attention →](06-scaled-dot-product-attention.md)
