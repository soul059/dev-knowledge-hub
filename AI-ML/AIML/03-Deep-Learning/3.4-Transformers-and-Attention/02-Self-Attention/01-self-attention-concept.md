# Self-Attention: Attending to Your Own Sequence

[← Additive Attention](../01-Attention-Mechanism/07-additive-attention.md) | [Computing Self-Attention →](02-computing-self-attention.md)

---

## Overview

Self-attention is the mechanism that allows every token in a sequence to directly attend to every other token **in the same sequence**. Unlike traditional attention (where a decoder attends to an encoder), self-attention operates *within* a single sequence, enabling each word to gather context from all surrounding words regardless of their distance. This single idea — letting a sequence "look at itself" — is the foundation of the Transformer architecture and has revolutionized natural language processing, computer vision, and beyond. Self-attention captures long-range dependencies in O(1) path length, making it far more effective than RNNs, which must propagate information step by step through potentially hundreds of time steps.

---

## 1. Core Concept: Every Token Attends to All Tokens

In a traditional RNN, the hidden state at position *t* depends only on position *t-1*. Information from position 0 must travel through every intermediate step to reach position *t*. Self-attention removes this bottleneck entirely.

### ASCII Diagram — Token-to-Token Attention

```
Input Sequence:  [The]   [cat]   [sat]   [on]   [the]   [mat]

                   │       │       │       │       │       │
                   ▼       ▼       ▼       ▼       ▼       ▼
              ┌─────────────────────────────────────────────────┐
              │            SELF-ATTENTION LAYER                 │
              │                                                 │
              │  Every token computes attention to ALL tokens:  │
              │                                                 │
              │  "The" ──► attends to [The,cat,sat,on,the,mat]  │
              │  "cat" ──► attends to [The,cat,sat,on,the,mat]  │
              │  "sat" ──► attends to [The,cat,sat,on,the,mat]  │
              │  "on"  ──► attends to [The,cat,sat,on,the,mat]  │
              │  "the" ──► attends to [The,cat,sat,on,the,mat]  │
              │  "mat" ──► attends to [The,cat,sat,on,the,mat]  │
              └─────────────────────────────────────────────────┘
                   │       │       │       │       │       │
                   ▼       ▼       ▼       ▼       ▼       ▼
Output:       [The']  [cat']  [sat']  [on']  [the']  [mat']

Each output token is a *weighted combination* of ALL input tokens,
where the weights are learned attention scores.
```

### Key Insight

Each output representation (e.g., `cat'`) is a **context-enriched** version of the input. The word "cat" is no longer just a static embedding — it now carries information from the entire sentence.

---

## 2. Motivating Example: Coreference Resolution

Consider the sentence:

> **"The cat sat on the mat because it was tired."**

What does **"it"** refer to? A human immediately knows it refers to **"cat"**. But how does a model figure this out?

### RNN Approach (Problematic)

```
The → cat → sat → on → the → mat → because → it → was → tired
 ──────────────────────────────────────────────►
 Information about "cat" must survive 6 time steps to reach "it"
 Signal degrades with each step (vanishing gradient problem)
```

### Self-Attention Approach (Direct)

```
Attention scores for "it" (token 8):

  The   cat   sat   on   the   mat  because  it   was  tired
  0.03  0.72  0.04  0.01 0.02  0.05  0.01   0.08 0.02  0.02
        ^^^^
        Highest attention!

"it" directly attends to "cat" with weight 0.72
No information needs to travel through intermediate tokens!
```

The model learns that **"it"** and **"cat"** are semantically related (both animate nouns, coreference pattern), and assigns a high attention weight directly — regardless of the 6-word gap between them.

---

## 3. Self-Attention vs Cross-Attention

| Feature | Self-Attention | Cross-Attention |
|---------|---------------|-----------------|
| **Q, K, V source** | All from the **same** sequence | Q from one sequence, K/V from **another** |
| **Use case** | Encoding context within a sentence | Decoder attending to encoder output |
| **Example** | Encoder in BERT | Decoder in machine translation |
| **Sequence** | Single sequence X | Two sequences: X (query) and Y (key/value) |

### ASCII Diagram — Self vs Cross Attention

```
SELF-ATTENTION                      CROSS-ATTENTION
(within one sequence)               (between two sequences)

Sequence X: [x1, x2, x3]           Query Seq:  [q1, q2]
                                    Key/Val Seq: [k1, k2, k3]
  Q ← X                             Q ← Query Seq
  K ← X    (same source!)           K ← Key/Val Seq  (different!)
  V ← X                             V ← Key/Val Seq

  x1 attends to [x1,x2,x3]         q1 attends to [k1,k2,k3]
  x2 attends to [x1,x2,x3]         q2 attends to [k1,k2,k3]
  x3 attends to [x1,x2,x3]
```

---

## 4. Mathematical Formulation

Given an input sequence **X ∈ ℝ^(n × d_model)** (n tokens, each of dimension d_model):

### Step 1: Linear Projections

$$Q = XW_Q, \quad K = XW_K, \quad V = XW_V$$

Where:
- **W_Q ∈ ℝ^(d_model × d_k)** — query weight matrix
- **W_K ∈ ℝ^(d_model × d_k)** — key weight matrix
- **W_V ∈ ℝ^(d_model × d_v)** — value weight matrix

### Step 2: Scaled Dot-Product Attention

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

### What Each Component Represents

| Component | Role | Analogy |
|-----------|------|---------|
| **Q** (Query) | "What am I looking for?" | A search query |
| **K** (Key) | "What do I contain?" | An index/tag |
| **V** (Value) | "What information do I provide?" | The actual content |
| **QKᵀ** | Compatibility score | Search relevance ranking |
| **softmax(·)** | Normalized weights | Probability distribution |
| **√d_k** | Temperature scaling | Prevents extreme softmax outputs |

### Why Scale by √d_k?

For large d_k, the dot products grow in magnitude (variance ≈ d_k), pushing softmax into regions with tiny gradients. Dividing by √d_k keeps the variance ≈ 1, ensuring healthy gradient flow.

If q and k are independent random variables with mean 0 and variance 1:
```
Var(q · k) = Var(Σ q_i * k_i) = d_k * Var(q_i) * Var(k_i) = d_k

After scaling:  Var(q · k / √d_k) = d_k / d_k = 1  ✓
```

---

## 5. Worked Example (Conceptual)

Consider a 3-token sentence: **["I", "love", "transformers"]**

```
Step 1: Embed each token (d_model = 4)
  X = [[0.1, 0.9, 0.3, 0.2],    ← "I"
       [0.4, 0.2, 0.8, 0.1],    ← "love"
       [0.7, 0.5, 0.1, 0.6]]   ← "transformers"

Step 2: Project to Q, K, V (d_k = 3)
  Q = X @ W_Q    →  3×3 matrix  (each token gets a query vector)
  K = X @ W_K    →  3×3 matrix  (each token gets a key vector)
  V = X @ W_V    →  3×3 matrix  (each token gets a value vector)

Step 3: Compute attention scores = Q @ K^T  →  3×3 matrix
  scores[i][j] = "how much should token i attend to token j"

       "I"    "love"  "trans"
  "I"  [0.52   0.31    0.44 ]
  "love"[0.38   0.67    0.55 ]
  "trans"[0.41   0.53    0.71 ]

Step 4: Scale by √d_k = √3 ≈ 1.73
  scaled = scores / 1.73

Step 5: Softmax across each row
  weights = softmax(scaled)

       "I"    "love"  "trans"
  "I"  [0.38   0.28    0.34 ]   ← "I" attends most to itself
  "love"[0.26   0.40    0.34 ]   ← "love" attends most to itself
  "trans"[0.27   0.33    0.40 ]   ← "transformers" attends most to itself

Step 6: Output = weights @ V
  Each row of the output is a weighted combination of all value vectors
```

> Full numerical detail with actual matrix multiplications is in [Computing Self-Attention →](02-computing-self-attention.md).

---

## 6. Python/PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class SelfAttention(nn.Module):
    """
    Basic single-head self-attention from scratch.
    """
    def __init__(self, d_model, d_k):
        super().__init__()
        self.d_k = d_k
        # Linear projections for Q, K, V
        self.W_Q = nn.Linear(d_model, d_k, bias=False)
        self.W_K = nn.Linear(d_model, d_k, bias=False)
        self.W_V = nn.Linear(d_model, d_k, bias=False)

    def forward(self, X):
        """
        Args:
            X: Input tensor of shape (batch, seq_len, d_model)
        Returns:
            output: Context-enriched representations (batch, seq_len, d_k)
            weights: Attention weights (batch, seq_len, seq_len)
        """
        Q = self.W_Q(X)   # (batch, seq_len, d_k)
        K = self.W_K(X)   # (batch, seq_len, d_k)
        V = self.W_V(X)   # (batch, seq_len, d_k)

        # Compute scaled dot-product attention
        scores = torch.matmul(Q, K.transpose(-2, -1))  # (batch, seq_len, seq_len)
        scores = scores / (self.d_k ** 0.5)            # Scale

        weights = F.softmax(scores, dim=-1)             # Normalize
        output = torch.matmul(weights, V)               # Weighted sum

        return output, weights


# --- Demo ---
torch.manual_seed(42)

d_model = 4
d_k = 3
seq_len = 5
batch_size = 1

# Simulate input embeddings for 5 tokens
X = torch.randn(batch_size, seq_len, d_model)

attn = SelfAttention(d_model, d_k)
output, weights = attn(X)

print("Input shape: ", X.shape)       # [1, 5, 4]
print("Output shape:", output.shape)   # [1, 5, 3]
print("Attention weights shape:", weights.shape)  # [1, 5, 5]
print("\nAttention weight matrix (how much each token attends to others):")
print(weights.squeeze().detach().numpy().round(3))
```

### Visualizing Attention Weights

```python
import matplotlib.pyplot as plt
import numpy as np

tokens = ["The", "cat", "sat", "on", "mat"]
attn_weights = weights.squeeze().detach().numpy()

fig, ax = plt.subplots(figsize=(6, 5))
im = ax.imshow(attn_weights, cmap='Blues', vmin=0, vmax=1)

ax.set_xticks(range(len(tokens)))
ax.set_yticks(range(len(tokens)))
ax.set_xticklabels(tokens)
ax.set_yticklabels(tokens)
ax.set_xlabel("Attending TO (Key)")
ax.set_ylabel("Attending FROM (Query)")
ax.set_title("Self-Attention Weights")

# Add text annotations
for i in range(len(tokens)):
    for j in range(len(tokens)):
        ax.text(j, i, f'{attn_weights[i, j]:.2f}',
                ha='center', va='center', fontsize=10)

plt.colorbar(im)
plt.tight_layout()
plt.savefig("self_attention_heatmap.png", dpi=150)
plt.show()
```

---

## 7. Real-World Applications

### Natural Language Processing
- **BERT**: Uses self-attention in the encoder to build deep bidirectional representations. Each word simultaneously sees all other words, enabling rich contextual embeddings.
- **GPT**: Uses *masked* self-attention in the decoder, where each token can only attend to previous tokens (causal/autoregressive setting).
- **Machine Translation**: Self-attention in both encoder and decoder captures within-language dependencies before cross-attention handles between-language alignment.

### Computer Vision
- **Vision Transformer (ViT)**: Splits images into patches and applies self-attention, allowing each patch to attend to all other patches — capturing global context that CNNs struggle with.
- **DETR**: Uses self-attention for object detection, relating objects to each other within an image.

### Biology & Chemistry
- **AlphaFold 2**: Self-attention over amino acid sequences captures residue-residue interactions critical for protein structure prediction.
- **Drug Discovery**: Self-attention on molecular graphs captures atom-atom interactions for property prediction.

### Audio & Speech
- **Whisper**: Self-attention on audio spectrograms for speech recognition, capturing temporal dependencies across the utterance.

---

## 8. Summary Table

| Aspect | Detail |
|--------|--------|
| **What** | Each token computes a weighted sum of all token representations in the same sequence |
| **Why** | Captures long-range dependencies in O(1) path length |
| **Q, K, V** | Three learned linear projections of the input |
| **Formula** | Attention(Q,K,V) = softmax(QKᵀ/√d_k) · V |
| **vs RNN** | Parallel, O(1) path length, no vanishing gradient for distant tokens |
| **vs Cross-Attn** | Q, K, V all come from the same sequence (not two different ones) |
| **Key Advantage** | Global context in a single layer |
| **Key Limitation** | O(n²) memory and compute in sequence length |
| **Used In** | BERT, GPT, ViT, AlphaFold, Whisper, and many more |

---

## 9. Revision Questions

1. **Conceptual**: Explain in your own words why self-attention can capture long-range dependencies more effectively than an RNN. What is the maximum path length for information to travel between any two tokens?

2. **Compare & Contrast**: What is the difference between self-attention and cross-attention? When would you use each one?

3. **Mathematical**: Why do we scale the dot product scores by √d_k? What would happen if we didn't scale? Describe the effect on the softmax distribution.

4. **Application**: In the sentence "The animal didn't cross the street because it was too wide", what does "it" refer to? How would self-attention help a model resolve this ambiguity?

5. **Implementation**: Given an input X of shape (batch=2, seq_len=10, d_model=512) and d_k=64, what are the shapes of Q, K, V, the attention score matrix, and the final output?

6. **Critical Thinking**: Self-attention has O(n²) complexity in sequence length. For a document with 10,000 tokens, how many attention scores must be computed? Why might this be problematic, and what alternatives exist?

---

[← Additive Attention](../01-Attention-Mechanism/07-additive-attention.md) | [Computing Self-Attention →](02-computing-self-attention.md)
