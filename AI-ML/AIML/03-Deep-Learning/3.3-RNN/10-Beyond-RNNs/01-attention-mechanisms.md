# Attention Mechanisms: Eliminating Recurrence

> **Unit 10, Chapter 1** — How self-attention enables parallel processing with O(1) path lengths, fundamentally changing sequence modeling.

---

## 📋 Overview

While gating mechanisms (LSTM, GRU) mitigated the vanishing gradient problem, they didn't solve RNNs' fundamental limitation: **sequential computation**. Each time step must wait for the previous one, making RNNs impossible to parallelize across time. **Attention mechanisms** — particularly **self-attention** — offer a radical alternative: every position can directly attend to every other position, eliminating recurrence entirely and enabling massive parallelization.

---

## 🔄 From Seq2Seq Attention to Self-Attention

### Evolution

```
Stage 1: Vanilla Seq2Seq (2014)
  Encoder: x₁→x₂→x₃→x₄ → [context vector c] → y₁→y₂→y₃
  Problem: Single fixed-size context vector = information bottleneck

Stage 2: Attention in Seq2Seq (Bahdanau 2015)
  Decoder attends to ALL encoder hidden states
  Different focus for each output step
  Problem: Still uses RNNs — still sequential

Stage 3: Self-Attention / Transformer (Vaswani 2017)
  EVERY position attends to EVERY other position
  No recurrence at all — fully parallel
  Revolution: "Attention Is All You Need"
```

---

## 🧠 Self-Attention Mechanism

### Core Idea

For each position in the sequence, compute a weighted sum of all positions' representations, where the weights are determined by compatibility between positions.

```
Input sequence: "The cat sat on the mat"

For the word "sat":
  Attends to:  "The"(0.05) "cat"(0.40) "sat"(0.30) "on"(0.10) "the"(0.05) "mat"(0.10)
                              ↑ high attention: subject of "sat"
  
  Output for "sat" = 0.05·v("The") + 0.40·v("cat") + 0.30·v("sat") + ...

Every word gets a context-aware representation by "looking at" every other word.
```

### Query-Key-Value Framework

Each input position gets three vectors:

```
┌──────────────────────────────────────────────────────────────┐
│  For each input embedding xᵢ:                                │
│                                                              │
│  Query:  qᵢ = W_Q · xᵢ    "What am I looking for?"         │
│  Key:    kᵢ = W_K · xᵢ    "What do I contain?"             │
│  Value:  vᵢ = W_V · xᵢ    "What information do I provide?" │
│                                                              │
│  Attention weight αᵢⱼ = how much position i attends to j    │
│  = softmax(qᵢ · kⱼ / √d_k)                                 │
│                                                              │
│  Output for position i:                                      │
│  zᵢ = Σⱼ αᵢⱼ · vⱼ                                          │
└──────────────────────────────────────────────────────────────┘
```

### Scaled Dot-Product Attention

```
Attention(Q, K, V) = softmax(Q · K^T / √d_k) · V

Where:
  Q: [seq_len, d_k]    — queries for all positions
  K: [seq_len, d_k]    — keys for all positions  
  V: [seq_len, d_v]    — values for all positions
  d_k: key dimension   — scaling factor prevents large dot products

Step-by-step:
  1. Q · K^T → [seq_len, seq_len]     — all-pairs similarity scores
  2. / √d_k                            — scale to prevent softmax saturation
  3. softmax(row-wise)                  — normalize to weights summing to 1
  4. × V                               — weighted combination of values
```

### ASCII Diagram

```
                      Attention Scores Matrix
                      (who attends to whom)
                      
              Key positions →
              The   cat   sat   on   the   mat
  Query  The [ 0.8  0.05  0.05  0.02  0.06  0.02 ]
  pos.   cat [ 0.1  0.5   0.2   0.05  0.05  0.1  ]
   ↓     sat [ 0.05 0.4   0.3   0.1   0.05  0.1  ]
         on  [ 0.02 0.08  0.3   0.3   0.1   0.2  ]
         the [ 0.3  0.05  0.05  0.1   0.4   0.1  ]
         mat [ 0.05 0.1   0.1   0.2   0.15  0.4  ]
         
  Each row sums to 1.0 (softmax)
  Each row gives the attention pattern for that position
```

---

## 📐 Mathematical Details

### Complete Self-Attention Equations

```
Input: X ∈ ℝ^{n × d_model}    (n positions, d_model dimensions)

Projections:
  Q = X · W_Q    where W_Q ∈ ℝ^{d_model × d_k}
  K = X · W_K    where W_K ∈ ℝ^{d_model × d_k}
  V = X · W_V    where W_V ∈ ℝ^{d_model × d_v}

Attention scores:
  S = Q · K^T / √d_k    ∈ ℝ^{n × n}

Attention weights:
  A = softmax(S, dim=-1)  ∈ ℝ^{n × n}    (row-wise softmax)

Output:
  Z = A · V    ∈ ℝ^{n × d_v}
```

### Multi-Head Attention

Instead of one attention function, use `h` parallel attention "heads":

```
MultiHead(Q, K, V) = Concat(head₁, head₂, ..., head_h) · W_O

where headᵢ = Attention(Q · Wᵢ_Q, K · Wᵢ_K, V · Wᵢ_V)

┌──────────────────────────────────────────────────────────────┐
│  Why multiple heads?                                         │
│                                                              │
│  Head 1: might learn syntactic relationships                 │
│  Head 2: might learn semantic similarity                     │
│  Head 3: might learn positional relationships                │
│  Head 4: might learn coreference                             │
│                                                              │
│  Each head can attend to different aspects of the input!     │
└──────────────────────────────────────────────────────────────┘

If d_model = 512 and h = 8:
  Each head uses d_k = d_v = d_model / h = 64
  Total computation ≈ same as single-head with full dimensions
```

---

## ⚡ O(1) Path Length vs O(n) for RNNs

### The Key Advantage

```
RNN: Information from position 1 to position n:
  x₁ → h₁ → h₂ → h₃ → ... → h_{n-1} → h_n
  
  Path length: O(n)
  Signal must pass through n-1 transformations
  Gradient must flow back through n-1 steps
  → Vanishing gradient for large n

Self-Attention: Information from position 1 to position n:
  x₁ ─────────────────────────────────→ attends directly to x_n
  
  Path length: O(1)
  Direct connection between ANY two positions
  Gradient flows through a single attention weight
  → No vanishing gradient problem!
```

### Complexity Comparison

```
┌────────────────┬──────────────┬──────────────┬───────────────┐
│                │ Sequential   │ Max Path     │ Computation   │
│    Model       │ Operations   │ Length       │ per Layer     │
├────────────────┼──────────────┼──────────────┼───────────────┤
│ RNN            │ O(n)         │ O(n)         │ O(n · d²)     │
│ Self-Attention │ O(1)         │ O(1)         │ O(n² · d)     │
│ CNN            │ O(1)         │ O(log_k(n))  │ O(k · n · d²)│
└────────────────┴──────────────┴──────────────┴───────────────┘

n = sequence length, d = dimension, k = kernel size

Key trade-off:
  RNN:       O(n · d²) computation but O(n) sequential steps
  Attention: O(n² · d) computation but O(1) sequential steps (parallel!)
  
  For typical d ≫ n: attention wins overall
  For very long n: attention's O(n²) becomes expensive
```

---

## 💻 PyTorch Implementation

### Self-Attention from Scratch

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math

class SelfAttention(nn.Module):
    def __init__(self, d_model, d_k=None, d_v=None):
        super().__init__()
        self.d_k = d_k or d_model
        self.d_v = d_v or d_model
        
        self.W_Q = nn.Linear(d_model, self.d_k, bias=False)
        self.W_K = nn.Linear(d_model, self.d_k, bias=False)
        self.W_V = nn.Linear(d_model, self.d_v, bias=False)
    
    def forward(self, x, mask=None):
        """
        Args:
            x: [batch, seq_len, d_model]
            mask: [batch, seq_len, seq_len] or None
        Returns:
            output: [batch, seq_len, d_v]
            attention_weights: [batch, seq_len, seq_len]
        """
        Q = self.W_Q(x)  # [batch, seq_len, d_k]
        K = self.W_K(x)  # [batch, seq_len, d_k]
        V = self.W_V(x)  # [batch, seq_len, d_v]
        
        # Scaled dot-product attention
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)
        
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float('-inf'))
        
        attn_weights = F.softmax(scores, dim=-1)
        output = torch.matmul(attn_weights, V)
        
        return output, attn_weights


class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, num_heads):
        super().__init__()
        assert d_model % num_heads == 0
        
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads
        
        self.W_Q = nn.Linear(d_model, d_model)
        self.W_K = nn.Linear(d_model, d_model)
        self.W_V = nn.Linear(d_model, d_model)
        self.W_O = nn.Linear(d_model, d_model)
    
    def forward(self, x, mask=None):
        batch_size, seq_len, _ = x.size()
        
        # Project and reshape to [batch, heads, seq_len, d_k]
        Q = self.W_Q(x).view(batch_size, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        K = self.W_K(x).view(batch_size, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        V = self.W_V(x).view(batch_size, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        
        # Attention for all heads in parallel
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)
        
        if mask is not None:
            scores = scores.masked_fill(mask.unsqueeze(1) == 0, float('-inf'))
        
        attn = F.softmax(scores, dim=-1)
        context = torch.matmul(attn, V)
        
        # Concatenate heads: [batch, seq_len, d_model]
        context = context.transpose(1, 2).contiguous().view(batch_size, seq_len, self.d_model)
        
        return self.W_O(context)

# Usage
mha = MultiHeadAttention(d_model=512, num_heads=8)
x = torch.randn(32, 50, 512)  # batch=32, seq_len=50
output = mha(x)
print(output.shape)  # [32, 50, 512]
```

---

## 🧮 Worked Example: Self-Attention Computation

### Setup

```
Sequence: [x₁, x₂, x₃],  d_model = 4,  d_k = d_v = 2

Input embeddings:
  x₁ = [1, 0, 1, 0]
  x₂ = [0, 1, 0, 1]
  x₃ = [1, 1, 0, 0]

Weight matrices (simplified):
  W_Q = [[1, 0],     W_K = [[0, 1],     W_V = [[1, 1],
         [0, 1],            [1, 0],            [0, 1],
         [1, 0],            [1, 0],            [1, 0],
         [0, 1]]            [0, 1]]            [0, 1]]

Step 1: Compute Q, K, V
  q₁ = W_Q^T · x₁ = [1+1, 0+0] = [2, 0]
  q₂ = W_Q^T · x₂ = [0+0, 1+1] = [0, 2]
  q₃ = W_Q^T · x₃ = [1+0, 0+0] = [1, 1]    (simplified)

  k₁ = [1, 1],  k₂ = [1, 1],  k₃ = [1, 1]  (simplified for clarity)
  v₁ = [2, 1],  v₂ = [0, 2],  v₃ = [1, 1]

Step 2: Attention scores  (Q · K^T / √d_k)
  score(1,1) = q₁ · k₁ / √2 = (2·1 + 0·1) / 1.41 = 1.41
  score(1,2) = q₁ · k₂ / √2 = (2·1 + 0·1) / 1.41 = 1.41
  score(1,3) = q₁ · k₃ / √2 = (2·1 + 0·1) / 1.41 = 1.41

Step 3: Softmax (row-wise)
  α(1,:) = softmax([1.41, 1.41, 1.41]) = [0.33, 0.33, 0.33]
  (Uniform attention in this simplified case)

Step 4: Weighted sum
  z₁ = 0.33·[2,1] + 0.33·[0,2] + 0.33·[1,1] = [1.0, 1.33]
```

---

## 🔍 Positional Information

### The Problem

Self-attention is **permutation-invariant** — it treats the input as a set, not a sequence. We need to add positional information explicitly.

```
Without positional encoding:
  "The cat sat on the mat"  has same attention as
  "mat the on sat cat The"  ← Same bag of words!

Solution: Add positional encodings to input embeddings
  x'ᵢ = xᵢ + PEᵢ

Sinusoidal encoding:
  PE(pos, 2i)   = sin(pos / 10000^(2i/d_model))
  PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
```

---

## 📊 Attention vs RNN: When Each Wins

| Aspect | Self-Attention | RNN |
|--------|---------------|-----|
| Parallelism | ✅ Fully parallel | ❌ Sequential |
| Long-range deps | ✅ O(1) path | ❌ O(n) path |
| Memory (inference) | ❌ O(n²) | ✅ O(1) per step |
| Streaming | ❌ Needs full sequence | ✅ Online processing |
| Very long seqs | ❌ Quadratic cost | ✅ Linear cost |
| Small data | ❌ Data-hungry | ✅ Strong inductive bias |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Self-attention | Every position attends to every other — no recurrence |
| Q, K, V | Query (what I seek), Key (what I offer), Value (what I provide) |
| Scaled dot-product | scores = Q·K^T/√d_k, then softmax, then weighted V |
| Multi-head | Multiple parallel attention heads capture different patterns |
| O(1) path length | Direct connection between any two positions |
| O(n²) cost | Quadratic in sequence length — the main limitation |
| Positional encoding | Must be added explicitly since attention is permutation-invariant |

---

## ❓ Revision Questions

1. **Explain the Q, K, V framework intuitively.** Why do we need three separate projections instead of using the input directly?

2. **Why does self-attention scale by √d_k?** What happens without this scaling as d_k grows?

3. **Compare the maximum path length between position 1 and position n** for (a) RNN, (b) self-attention, (c) CNN with kernel size k. Which has the best gradient flow?

4. **Self-attention has O(n²) complexity.** For a sequence of length 10,000, how does this compare to an RNN? When does the quadratic cost become prohibitive?

5. **Why is multi-head attention better than single-head with the same total dimensionality?** What do different heads learn?

6. **Self-attention is permutation-invariant.** Explain why this is a problem for sequences and how positional encodings solve it.

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬆️ Unit | [10 Beyond RNNs](../README.md) |
| ➡️ Next | [Transformer Preview](02-transformer-preview.md) |
| 🏠 Home | [RNN Home](../README.md) |

---

*Last updated: 2024*
