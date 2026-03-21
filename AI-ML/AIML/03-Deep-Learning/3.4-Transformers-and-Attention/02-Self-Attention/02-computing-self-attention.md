# Computing Self-Attention Step by Step

[← Self-Attention Concept](01-self-attention-concept.md) | [Why Self-Attention Works →](03-why-self-attention-works.md)

---

## Overview

This note walks through the **complete computation** of scaled dot-product self-attention with actual numbers. We will start with raw input embeddings, project them into queries, keys, and values, compute attention scores, apply scaling and softmax, and finally produce the context-enriched output. Every single matrix multiplication is shown so you can verify each step with pen and paper. By the end, you will have an intuitive and mathematical understanding of exactly what happens inside one self-attention layer.

---

## 1. The Six Steps of Self-Attention

```
┌──────────────┐
│ Input X      │   (n × d_model)
└──────┬───────┘
       │
       ├───────────────────┬───────────────────┐
       ▼                   ▼                   ▼
  ┌─────────┐        ┌─────────┐        ┌─────────┐
  │ Q = XW_Q│        │ K = XW_K│        │ V = XW_V│
  │ (n×d_k) │        │ (n×d_k) │        │ (n×d_v) │
  └────┬────┘        └────┬────┘        └────┬────┘
       │                   │                  │
       ▼                   ▼                  │
  ┌────────────────────────────┐              │
  │  Scores = Q @ K^T         │              │
  │  (n × n)                  │              │
  └────────────┬───────────────┘              │
               │                              │
               ▼                              │
  ┌────────────────────────────┐              │
  │  Scaled = Scores / √d_k   │              │
  └────────────┬───────────────┘              │
               │                              │
               ▼                              │
  ┌────────────────────────────┐              │
  │  Weights = softmax(Scaled) │              │
  │  (n × n)                  │              │
  └────────────┬───────────────┘              │
               │                              │
               ▼                              ▼
  ┌─────────────────────────────────────────────┐
  │  Output = Weights @ V                       │
  │  (n × d_v)                                  │
  └─────────────────────────────────────────────┘
```

---

## 2. Setup: Defining Dimensions

For this worked example, we use small dimensions to keep the math tractable:

| Symbol | Value | Meaning |
|--------|-------|---------|
| n | 3 | Number of tokens |
| d_model | 4 | Input embedding dimension |
| d_k | 3 | Query/Key dimension |
| d_v | 3 | Value dimension |

**Tokens**: `["I", "love", "AI"]`

---

## 3. Step 1 — Input Embeddings X

Each token is represented as a 4-dimensional vector (d_model = 4):

```
         d1    d2    d3    d4
X = ┌                          ┐
    │  1.0   0.0   1.0   0.0  │   ← "I"
    │  0.0   1.0   1.0   0.0  │   ← "love"
    │  1.0   1.0   0.0   1.0  │   ← "AI"
    └                          ┘
Shape: (3 × 4)
```

---

## 4. Step 2 — Compute Q, K, V via Linear Projections

### Weight Matrices

```
         d_k1  d_k2  d_k3                d_k1  d_k2  d_k3
W_Q = ┌                    ┐     W_K = ┌                    ┐
      │  1     0     1     │           │  0     1     0     │
      │  0     1     0     │           │  1     0     1     │
      │  1     0     0     │           │  0     1     1     │
      │  0     1     1     │           │  1     0     0     │
      └                    ┘           └                    ┘
Shape: (4 × 3)                   Shape: (4 × 3)

         d_v1  d_v2  d_v3
W_V = ┌                    ┐
      │  1     0     0     │
      │  0     1     0     │
      │  0     0     1     │
      │  1     1     0     │
      └                    ┘
Shape: (4 × 3)
```

### Computing Q = X @ W_Q

```
Q[0] = [1,0,1,0] @ W_Q = [1*1+0*0+1*1+0*0, 1*0+0*1+1*0+0*1, 1*1+0*0+1*0+0*1] = [2, 0, 1]
Q[1] = [0,1,1,0] @ W_Q = [0*1+1*0+1*1+0*0, 0*0+1*1+1*0+0*1, 0*1+1*0+1*0+0*1] = [1, 1, 0]
Q[2] = [1,1,0,1] @ W_Q = [1*1+1*0+0*1+1*0, 1*0+1*1+0*0+1*1, 1*1+1*0+0*0+1*1] = [1, 2, 2]

      ┌             ┐
Q =   │  2   0   1  │   ← "I"
      │  1   1   0  │   ← "love"
      │  1   2   2  │   ← "AI"
      └             ┘
```

### Computing K = X @ W_K

```
K[0] = [1,0,1,0] @ W_K = [0+0+0+0, 1+0+1+0, 0+0+1+0] = [0, 2, 1]
K[1] = [0,1,1,0] @ W_K = [0+1+0+0, 0+0+1+0, 0+1+1+0] = [1, 1, 2]
K[2] = [1,1,0,1] @ W_K = [0+1+0+1, 1+0+0+0, 0+1+0+0] = [2, 1, 1]

      ┌             ┐
K =   │  0   2   1  │   ← "I"
      │  1   1   2  │   ← "love"
      │  2   1   1  │   ← "AI"
      └             ┘
```

### Computing V = X @ W_V

```
V[0] = [1,0,1,0] @ W_V = [1+0+0+0, 0+0+0+0, 0+0+1+0] = [1, 0, 1]
V[1] = [0,1,1,0] @ W_V = [0+0+0+0, 0+1+0+0, 0+0+1+0] = [0, 1, 1]
V[2] = [1,1,0,1] @ W_V = [1+0+0+1, 0+1+0+1, 0+0+0+0] = [2, 2, 0]

      ┌             ┐
V =   │  1   0   1  │   ← "I"
      │  0   1   1  │   ← "love"
      │  2   2   0  │   ← "AI"
      └             ┘
```

---

## 5. Step 3 — Compute Raw Attention Scores (QKᵀ)

Scores matrix S = Q @ Kᵀ tells us how much each query "matches" each key.

```
K^T = ┌             ┐
      │  0   1   2  │
      │  2   1   1  │
      │  1   2   1  │
      └             ┘

S = Q @ K^T:

S[0][0] = [2,0,1] · [0,2,1] = 0+0+1 = 1
S[0][1] = [2,0,1] · [1,1,2] = 2+0+2 = 4
S[0][2] = [2,0,1] · [2,1,1] = 4+0+1 = 5

S[1][0] = [1,1,0] · [0,2,1] = 0+2+0 = 2
S[1][1] = [1,1,0] · [1,1,2] = 1+1+0 = 2
S[1][2] = [1,1,0] · [2,1,1] = 2+1+0 = 3

S[2][0] = [1,2,2] · [0,2,1] = 0+4+2 = 6
S[2][1] = [1,2,2] · [1,1,2] = 1+2+4 = 7
S[2][2] = [1,2,2] · [2,1,1] = 2+2+2 = 6

            "I"   "love"  "AI"     ← Keys
S =   ┌                        ┐
"I"   │   1      4       5     │   "I" has highest score with "AI"
"love"│   2      2       3     │   "love" has highest score with "AI"
"AI"  │   6      7       6     │   "AI" has highest score with "love"
      └                        ┘
              ↑ Queries
```

---

## 6. Step 4 — Scale by √d_k

```
√d_k = √3 ≈ 1.732

Scaled = S / 1.732:

            "I"     "love"   "AI"
      ┌                              ┐
"I"   │  0.577    2.309    2.887     │
"love"│  1.155    1.155    1.732     │
"AI"  │  3.464    4.041    3.464     │
      └                              ┘
```

**Why scale?** Without scaling, the large dot products (like 7) would push softmax to near-one-hot outputs, killing gradients. Scaling keeps the values moderate.

---

## 7. Step 5 — Apply Softmax (Row-wise)

Softmax formula: softmax(z_i) = exp(z_i) / Σ_j exp(z_j)

### Row 0: "I" → [0.577, 2.309, 2.887]

```
exp(0.577) = 1.781
exp(2.309) = 10.063
exp(2.887) = 17.940
Sum = 29.784

softmax = [1.781/29.784, 10.063/29.784, 17.940/29.784]
        = [0.060, 0.338, 0.602]
```

### Row 1: "love" → [1.155, 1.155, 1.732]

```
exp(1.155) = 3.174
exp(1.155) = 3.174
exp(1.732) = 5.651
Sum = 11.999

softmax = [3.174/11.999, 3.174/11.999, 5.651/11.999]
        = [0.264, 0.264, 0.471]
```

### Row 2: "AI" → [3.464, 4.041, 3.464]

```
exp(3.464) = 31.956
exp(4.041) = 56.877
exp(3.464) = 31.956
Sum = 120.789

softmax = [31.956/120.789, 56.877/120.789, 31.956/120.789]
        = [0.265, 0.471, 0.265]
```

### Final Attention Weight Matrix

```
              "I"     "love"   "AI"     ← Which tokens are being attended TO
        ┌                              ┐
  "I"   │  0.060    0.338    0.602     │  "I" mostly attends to "AI"
  "love"│  0.264    0.264    0.471     │  "love" mostly attends to "AI"
  "AI"  │  0.265    0.471    0.265     │  "AI" mostly attends to "love"
        └                              ┘
  ↑ Which token is doing the attending (Query)

Each row sums to 1.0 ✓
```

---

## 8. Step 6 — Compute Output (Weights @ V)

The final output for each token is a weighted combination of all value vectors.

```
V = ┌             ┐
    │  1   0   1  │   ← V["I"]
    │  0   1   1  │   ← V["love"]
    │  2   2   0  │   ← V["AI"]
    └             ┘

Output = Weights @ V:

Out["I"]   = 0.060*[1,0,1] + 0.338*[0,1,1] + 0.602*[2,2,0]
           = [0.060,0.000,0.060] + [0.000,0.338,0.338] + [1.204,1.204,0.000]
           = [1.264, 1.542, 0.398]

Out["love"]= 0.264*[1,0,1] + 0.264*[0,1,1] + 0.471*[2,2,0]
           = [0.264,0.000,0.264] + [0.000,0.264,0.264] + [0.942,0.942,0.000]
           = [1.206, 1.206, 0.528]

Out["AI"]  = 0.265*[1,0,1] + 0.471*[0,1,1] + 0.265*[2,2,0]
           = [0.265,0.000,0.265] + [0.000,0.471,0.471] + [0.530,0.530,0.000]
           = [0.795, 1.001, 0.736]
```

### Final Output Matrix

```
              d1      d2      d3
        ┌                          ┐
  "I"   │  1.264   1.542   0.398  │   Now "I" contains info from all tokens
  "love"│  1.206   1.206   0.528  │   weighted by relevance
  "AI"  │  0.795   1.001   0.736  │
        └                          ┘
Shape: (3 × 3)  =  (n × d_v)
```

### Interpretation

Each output vector is no longer a "pure" representation of a single token. It's a **context-enriched blend**:
- `Out["I"]` is mostly influenced by `V["AI"]` (weight 0.602) — the model found "AI" most relevant to "I" with these weights.
- `Out["AI"]` is mostly influenced by `V["love"]` (weight 0.471) — "AI" attended most to "love".

---

## 9. Complete Computation Flow — Summary Diagram

```
X (3×4) ──┬── @W_Q(4×3) ──► Q (3×3)──┐
           │                           │
           ├── @W_K(4×3) ──► K (3×3)──┤── Q@K^T ──► S (3×3)
           │                           │               │
           └── @W_V(4×3) ──► V (3×3)  │          ÷ √d_k
                                │      │               │
                                │      │          softmax
                                │      │               │
                                │      │          W (3×3)
                                │      │               │
                                └──────┴───── W@V ──► Out (3×3)
```

---

## 10. PyTorch Implementation — Step by Step with Intermediate Output

```python
import torch
import torch.nn.functional as F

# --- Setup (matching our worked example) ---
X = torch.tensor([
    [1.0, 0.0, 1.0, 0.0],   # "I"
    [0.0, 1.0, 1.0, 0.0],   # "love"
    [1.0, 1.0, 0.0, 1.0],   # "AI"
])

W_Q = torch.tensor([
    [1.0, 0.0, 1.0],
    [0.0, 1.0, 0.0],
    [1.0, 0.0, 0.0],
    [0.0, 1.0, 1.0],
])

W_K = torch.tensor([
    [0.0, 1.0, 0.0],
    [1.0, 0.0, 1.0],
    [0.0, 1.0, 1.0],
    [1.0, 0.0, 0.0],
])

W_V = torch.tensor([
    [1.0, 0.0, 0.0],
    [0.0, 1.0, 0.0],
    [0.0, 0.0, 1.0],
    [1.0, 1.0, 0.0],
])

d_k = 3

# --- Step 2: Linear projections ---
Q = X @ W_Q
K = X @ W_K
V = X @ W_V
print("Q =\n", Q)
print("K =\n", K)
print("V =\n", V)

# --- Step 3: Raw scores ---
scores = Q @ K.T
print("\nRaw Scores (Q @ K^T) =\n", scores)

# --- Step 4: Scale ---
scaled = scores / (d_k ** 0.5)
print("\nScaled Scores =\n", scaled)

# --- Step 5: Softmax ---
weights = F.softmax(scaled, dim=-1)
print("\nAttention Weights =\n", weights)
print("Row sums:", weights.sum(dim=-1))  # Should all be 1.0

# --- Step 6: Output ---
output = weights @ V
print("\nOutput =\n", output)
```

### Expected Output

```
Q =
 tensor([[2., 0., 1.],
         [1., 1., 0.],
         [1., 2., 2.]])
K =
 tensor([[0., 2., 1.],
         [1., 1., 2.],
         [2., 1., 1.]])
V =
 tensor([[1., 0., 1.],
         [0., 1., 1.],
         [2., 2., 0.]])

Raw Scores (Q @ K^T) =
 tensor([[1., 4., 5.],
         [2., 2., 3.],
         [6., 7., 6.]])

Scaled Scores =
 tensor([[0.5774, 2.3094, 2.8868],
         [1.1547, 1.1547, 1.7321],
         [3.4641, 4.0415, 3.4641]])

Attention Weights =
 tensor([[0.0598, 0.3378, 0.6024],
         [0.2644, 0.2644, 0.4711],
         [0.2645, 0.4709, 0.2645]])
Row sums: tensor([1.0000, 1.0000, 1.0000])

Output =
 tensor([[1.2648, 1.5426, 0.3976],
         [1.2067, 1.2067, 0.5289],
         [0.7936, 1.0000, 0.7355]])
```

---

## 11. Using PyTorch's Built-in Function

```python
import torch
import torch.nn.functional as F

# PyTorch 2.0+ provides F.scaled_dot_product_attention
X = torch.tensor([
    [1.0, 0.0, 1.0, 0.0],
    [0.0, 1.0, 1.0, 0.0],
    [1.0, 1.0, 0.0, 1.0],
]).unsqueeze(0)  # Add batch dim: (1, 3, 4)

d_k = 3
W_Q = torch.randn(4, d_k)
W_K = torch.randn(4, d_k)
W_V = torch.randn(4, d_k)

Q = X @ W_Q  # (1, 3, 3)
K = X @ W_K
V = X @ W_V

# Built-in scaled dot-product attention (PyTorch 2.0+)
output = F.scaled_dot_product_attention(Q, K, V)
print("Output shape:", output.shape)  # (1, 3, 3)
print("Output:\n", output)
```

---

## 12. Common Pitfalls and Debugging Tips

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Forgot to scale by √d_k | Attention weights are nearly one-hot | Divide scores by `d_k ** 0.5` |
| Softmax on wrong dimension | Rows don't sum to 1 | Use `dim=-1` (last dimension) |
| Transposed wrong matrix | Shape mismatch error | Transpose K, not Q: `Q @ K.T` |
| Missing batch dimension | Matrix multiply fails | Unsqueeze: `X.unsqueeze(0)` |
| Using same weights for Q, K, V | Model can't differentiate roles | Use separate W_Q, W_K, W_V |

---

## 13. Dimension Tracking Cheat Sheet

```
Given:
  X ∈ ℝ^(n × d_model)         Input embeddings
  W_Q ∈ ℝ^(d_model × d_k)     Query projection
  W_K ∈ ℝ^(d_model × d_k)     Key projection
  W_V ∈ ℝ^(d_model × d_v)     Value projection

Computation:
  Q = X @ W_Q      ∈ ℝ^(n × d_k)
  K = X @ W_K      ∈ ℝ^(n × d_k)
  V = X @ W_V      ∈ ℝ^(n × d_v)

  Scores = Q @ K^T  ∈ ℝ^(n × n)         ← n² attention scores
  Weights = softmax(Scores / √d_k)  ∈ ℝ^(n × n)
  Output = Weights @ V  ∈ ℝ^(n × d_v)   ← same seq length, value dim

With batch dimension (B):
  X:       (B, n, d_model)
  Q, K:    (B, n, d_k)
  V:       (B, n, d_v)
  Scores:  (B, n, n)
  Weights: (B, n, n)
  Output:  (B, n, d_v)
```

---

## 14. Real-World Applications

- **Transformer Encoder**: Each encoder layer applies self-attention to its input, producing context-enriched representations. Stacking 6-12 such layers builds increasingly abstract representations.
- **BERT Pre-training**: Masked tokens are predicted using self-attention outputs, forcing the model to learn bidirectional context.
- **Code Understanding**: Models like CodeBERT use self-attention to understand code, where a variable at line 100 may depend on a declaration at line 5.
- **Document Summarization**: Self-attention identifies the most important sentences by measuring how each sentence relates to all others.

---

## 15. Summary Table

| Step | Operation | Shape Change | Purpose |
|------|-----------|-------------|---------|
| 1 | Input X | — → (n, d_model) | Token embeddings |
| 2 | Q = XW_Q | (n, d_model) → (n, d_k) | "What am I looking for?" |
| 2 | K = XW_K | (n, d_model) → (n, d_k) | "What do I contain?" |
| 2 | V = XW_V | (n, d_model) → (n, d_v) | "What info do I carry?" |
| 3 | S = QKᵀ | (n, d_k) × (d_k, n) → (n, n) | Compatibility scores |
| 4 | S / √d_k | (n, n) → (n, n) | Stabilize gradients |
| 5 | softmax(S) | (n, n) → (n, n) | Normalize to probabilities |
| 6 | Output = WV | (n, n) × (n, d_v) → (n, d_v) | Weighted value aggregation |

---

## 16. Revision Questions

1. **Verify**: Manually compute the attention weight for "love" attending to "I" using the matrices defined in this note. Show every multiplication.

2. **Shape Check**: If d_model = 512, d_k = 64, and n = 100, what are the shapes of Q, K, V, the score matrix, and the output? How many floating-point operations does Q @ Kᵀ require?

3. **Scaling Effect**: Recompute the softmax for Row 0 of our example WITHOUT dividing by √d_k (use raw scores [1, 4, 5]). Compare the resulting distribution to the scaled version. Which is more "peaked"?

4. **Interpretation**: In our example, "AI" attends most to "love" (weight 0.471). What does this mean in terms of the output representation of "AI"?

5. **Generalization**: If we set W_Q = W_K (tied query and key weights), what symmetry would the attention matrix have? Would this be desirable?

6. **Edge Case**: What happens to the attention weights if d_k is very large (say, 10000) and we forget to scale? Compute softmax([100, 101, 100]) vs softmax([0.1, 0.101, 0.1]) to illustrate.

---

[← Self-Attention Concept](01-self-attention-concept.md) | [Why Self-Attention Works →](03-why-self-attention-works.md)
