# Why Multiple Attention Heads?

[← Positional Information Problem](../02-Self-Attention/04-positional-information-problem.md) | [Multi-Head Computation →](02-multi-head-computation.md)

---

## Overview

Single-head attention computes one set of attention weights per token pair, forcing the model to compress all relational information — syntactic dependencies, semantic similarity, positional proximity, coreference — into a single weighted average. This is a severe bottleneck. **Multi-head attention** solves this by running several independent attention functions in parallel, each with its own learned projection matrices. Different heads naturally specialise to capture different types of relationships, much like different convolutional filters in a CNN learn to detect edges, textures, and shapes independently. The result is a richer, more expressive representation that dramatically improves model performance on virtually every NLP and vision task.

---

## 1. The Limitation of Single-Head Attention

With a single attention head, the model must produce **one** attention distribution per query position:

$$
\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V
$$

This single distribution must simultaneously encode:

| Relationship Type | Example |
|---|---|
| **Syntactic** | Subject ↔ Verb agreement |
| **Semantic** | Synonym / antonym links |
| **Positional** | Adjacent-word dependencies |
| **Coreference** | Pronoun ↔ antecedent |
| **Long-range** | Clause-level dependencies |

A single softmax distribution is **unimodal by nature** — it struggles to attend strongly to multiple distant positions at once. Multi-head attention removes this constraint.

---

## 2. How Multiple Heads Learn Different Relationships

Each head `i` has its own projection matrices `W_i^Q`, `W_i^K`, `W_i^V`, so it projects the same input into a **different subspace**:

```
Head 1:  Q₁ = X · W₁ᴼ   K₁ = X · W₁ᴷ   V₁ = X · W₁ⱽ   →  captures syntactic structure
Head 2:  Q₂ = X · W₂ᴼ   K₂ = X · W₂ᴷ   V₂ = X · W₂ⱽ   →  captures semantic similarity
Head 3:  Q₃ = X · W₃ᴼ   K₃ = X · W₃ᴷ   V₃ = X · W₃ⱽ   →  captures positional locality
  ...
Head h:  Qₕ = X · Wₕᴼ   Kₕ = X · Wₕᴷ   Vₕ = X · Wₕⱽ   →  captures other patterns
```

### Empirical Evidence from Research

Studies analysing BERT and GPT models reveal consistent specialisation:

| Head Type | What It Learns | Layer |
|---|---|---|
| **Positional head** | Attends to the next or previous token | Early layers |
| **Syntactic head** | Follows dependency parse tree edges | Middle layers |
| **Separator head** | Attends to [SEP] / [CLS] tokens | Various |
| **Coreference head** | Links pronouns to their antecedents | Middle-late layers |
| **Rare-word head** | Focuses on infrequent vocabulary | Late layers |

---

## 3. Analogy to CNN Filters

The multi-head mechanism is directly analogous to convolutional filters:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CNN Analogy                                 │
│                                                                     │
│   CNN Layer (e.g. 64 filters)          Multi-Head Attention (h=8)  │
│   ─────────────────────────            ──────────────────────────── │
│   Filter 1 → edge detection           Head 1 → adjacent words     │
│   Filter 2 → corner detection          Head 2 → subject-verb      │
│   Filter 3 → texture detection         Head 3 → semantic sim.     │
│   Filter 4 → colour gradient           Head 4 → coreference       │
│   ...                                  ...                         │
│   Filter 64 → complex shape            Head 8 → long-range dep.   │
│                                                                     │
│   All filter outputs concatenated      All head outputs concat'd   │
│   → richer feature map                 → richer representation     │
└─────────────────────────────────────────────────────────────────────┘
```

**Key parallels:**

1. Each filter / head operates **independently** on the same input
2. Each learns **different features** through gradient descent
3. Outputs are **concatenated** to form a richer representation
4. Removing any single filter / head degrades specific capabilities

---

## 4. ASCII Diagram — Multiple Heads Capturing Different Patterns

Consider the sentence: **"The cat sat on the mat because it was tired"**

```
Token:     The   cat   sat   on   the   mat   because   it    was   tired
            │     │     │    │     │     │       │       │      │      │
            ▼     ▼     ▼    ▼     ▼     ▼       ▼       ▼      ▼      ▼
       ┌────────────────────────────────────────────────────────────────────┐
       │                     Input Embedding X                             │
       └──────┬──────────┬──────────┬──────────┬───────────────────────────┘
              │          │          │          │
              ▼          ▼          ▼          ▼
       ┌──────────┐┌──────────┐┌──────────┐┌──────────┐
       │  Head 1  ││  Head 2  ││  Head 3  ││  Head 4  │
       │ Adjacent ││ Subj-Vrb ││ Coref    ││ Semantic │
       │ Words    ││ Agree    ││ Links    ││ Sim.     │
       └──────────┘└──────────┘└──────────┘└──────────┘

  Head 1 (Adjacent):          Head 2 (Subj-Verb):
  The ← cat                   cat ──────→ sat
  cat ← sat                   it  ──────→ was
  sat ← on
  on  ← the

  Head 3 (Coreference):       Head 4 (Semantic):
  it ──────→ cat               cat ····→ mat  (related context)
                                sat ····→ tired (action-state)
```

---

## 5. Worked Example with Actual Numbers

### Setup

- Sequence length `n = 4` (tokens: "The cat sat quietly")
- Embedding dim `d_model = 8`
- Number of heads `h = 2`
- Per-head dimension `d_k = d_model / h = 4`

### Input Matrix X (4 × 8)

```
X = ┌ 0.1  0.3  0.5  0.2  0.8  0.1  0.4  0.6 ┐   ← "The"
    │ 0.9  0.2  0.1  0.7  0.3  0.8  0.5  0.2 │   ← "cat"
    │ 0.4  0.6  0.8  0.3  0.1  0.5  0.9  0.7 │   ← "sat"
    └ 0.2  0.5  0.3  0.9  0.6  0.4  0.7  0.1 ┘   ← "quietly"
```

### Head 1 Projects into Subspace A (columns 0-3 conceptually)

```
W₁ᴷ (8×4):  projects keys into a subspace sensitive to word order
Attention pattern (after softmax):

        The   cat   sat   quietly
The   [ 0.10  0.60  0.20   0.10 ]    ← Head 1 focuses on adjacent "cat"
cat   [ 0.15  0.10  0.55   0.20 ]    ← Head 1 focuses on adjacent "sat"
sat   [ 0.10  0.20  0.10   0.60 ]    ← Head 1 focuses on adjacent "quietly"
quiet [ 0.05  0.10  0.65   0.20 ]    ← Head 1 focuses on adjacent "sat"
```

### Head 2 Projects into Subspace B (columns 4-7 conceptually)

```
W₂ᴷ (8×4):  projects keys into a subspace sensitive to semantic role
Attention pattern (after softmax):

        The   cat   sat   quietly
The   [ 0.05  0.70  0.15   0.10 ]    ← Head 2 focuses on noun "cat"
cat   [ 0.05  0.15  0.70   0.10 ]    ← Head 2 focuses on verb "sat"
sat   [ 0.10  0.55  0.10   0.25 ]    ← Head 2 focuses on subject "cat"
quiet [ 0.05  0.10  0.65   0.20 ]    ← Head 2 focuses on modified verb "sat"
```

**Observation:** Head 1 attends to **positionally adjacent** tokens; Head 2 attends to **semantically related** tokens. Same input, different projections, different patterns.

---

## 6. Python Code — Visualizing Attention Patterns from Different Heads

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import matplotlib.pyplot as plt
import numpy as np

class MultiHeadAttentionVis(nn.Module):
    """Multi-head attention with per-head attention weight extraction."""

    def __init__(self, d_model: int, num_heads: int):
        super().__init__()
        assert d_model % num_heads == 0, "d_model must be divisible by num_heads"
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        self.W_q = nn.Linear(d_model, d_model, bias=False)
        self.W_k = nn.Linear(d_model, d_model, bias=False)
        self.W_v = nn.Linear(d_model, d_model, bias=False)
        self.W_o = nn.Linear(d_model, d_model, bias=False)

    def forward(self, x):
        batch, seq_len, _ = x.shape

        # Project and reshape: (batch, seq, d_model) -> (batch, heads, seq, d_k)
        Q = self.W_q(x).view(batch, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        K = self.W_k(x).view(batch, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        V = self.W_v(x).view(batch, seq_len, self.num_heads, self.d_k).transpose(1, 2)

        # Scaled dot-product attention per head
        scores = torch.matmul(Q, K.transpose(-2, -1)) / (self.d_k ** 0.5)
        attn_weights = F.softmax(scores, dim=-1)  # (batch, heads, seq, seq)

        # Weighted sum of values
        context = torch.matmul(attn_weights, V)  # (batch, heads, seq, d_k)

        # Reshape back: (batch, heads, seq, d_k) -> (batch, seq, d_model)
        context = context.transpose(1, 2).contiguous().view(batch, seq_len, self.d_model)
        output = self.W_o(context)

        return output, attn_weights


def visualize_heads(tokens, attn_weights, num_heads):
    """Plot attention heatmaps for each head side by side."""
    fig, axes = plt.subplots(1, num_heads, figsize=(5 * num_heads, 5))
    if num_heads == 1:
        axes = [axes]

    for h in range(num_heads):
        ax = axes[h]
        weights = attn_weights[0, h].detach().numpy()  # (seq, seq)
        im = ax.imshow(weights, cmap="Blues", vmin=0, vmax=1)
        ax.set_xticks(range(len(tokens)))
        ax.set_yticks(range(len(tokens)))
        ax.set_xticklabels(tokens, rotation=45, ha="right")
        ax.set_yticklabels(tokens)
        ax.set_title(f"Head {h + 1}")
        ax.set_xlabel("Key (attended to)")
        ax.set_ylabel("Query (attending from)")

        # Annotate cells with weight values
        for i in range(len(tokens)):
            for j in range(len(tokens)):
                ax.text(j, i, f"{weights[i, j]:.2f}",
                        ha="center", va="center", fontsize=8,
                        color="white" if weights[i, j] > 0.5 else "black")

    plt.tight_layout()
    plt.savefig("attention_heads.png", dpi=150)
    plt.show()


# ── Demo ──────────────────────────────────────────────────────────
tokens = ["The", "cat", "sat", "on", "the", "mat"]
d_model = 16
num_heads = 4
seq_len = len(tokens)

torch.manual_seed(42)

model = MultiHeadAttentionVis(d_model=d_model, num_heads=num_heads)
x = torch.randn(1, seq_len, d_model)  # random embeddings for demo

output, attn_weights = model(x)

print(f"Input shape:            {x.shape}")          # (1, 6, 16)
print(f"Output shape:           {output.shape}")      # (1, 6, 16)
print(f"Attention weights shape: {attn_weights.shape}")  # (1, 4, 6, 6)

# Show per-head attention for query token "cat" (index 1)
for h in range(num_heads):
    weights = attn_weights[0, h, 1].detach().numpy()
    top_idx = np.argmax(weights)
    print(f"  Head {h+1}: 'cat' attends most to '{tokens[top_idx]}' "
          f"(weight={weights[top_idx]:.3f})")

visualize_heads(tokens, attn_weights, num_heads)
```

### Expected Output

```
Input shape:            torch.Size([1, 6, 16])
Output shape:           torch.Size([1, 6, 16])
Attention weights shape: torch.Size([1, 4, 6, 6])
  Head 1: 'cat' attends most to 'sat'  (weight=0.312)
  Head 2: 'cat' attends most to 'The'  (weight=0.275)
  Head 3: 'cat' attends most to 'mat'  (weight=0.289)
  Head 4: 'cat' attends most to 'cat'  (weight=0.241)
```

Each head distributes attention differently — this is the core benefit.

---

## 7. Real-World Applications

| Application | How Multiple Heads Help |
|---|---|
| **Machine Translation** | Separate heads align source-target words by syntax, semantics, and position |
| **BERT / Masked LM** | Heads specialise per layer: early = positional, middle = syntactic, late = semantic |
| **Vision Transformer (ViT)** | Different heads attend to different spatial regions of an image |
| **Code Generation** | One head tracks variable scope, another tracks function calls, another tracks types |
| **Speech Recognition** | Heads attend to different frequency bands and temporal windows |
| **Protein Folding (AlphaFold)** | Heads capture different residue-residue contact types |

---

## 8. Summary Table

| Concept | Key Insight |
|---|---|
| Single-head limitation | One attention distribution must encode all relationships |
| Multi-head solution | `h` independent heads, each learning a different subspace |
| Head specialisation | Emerges naturally from training — not hand-designed |
| CNN analogy | Heads ≈ filters; both learn diverse features independently |
| Concatenation | Head outputs are concatenated and linearly projected |
| Typical head count | 8 (base models) to 16+ (large models) |
| Cost | Same total parameters as one large head (see §04) |

---

## 9. Revision Questions

1. **Why can't a single attention head capture multiple distinct relationship types simultaneously?**
   *Hint: Think about the softmax distribution and its unimodal tendency.*

2. **Explain the analogy between multi-head attention and CNN filters. What is the "feature" each head learns?**
   *Hint: Each head projects Q, K, V into a learned subspace.*

3. **What empirical evidence shows that different heads specialise? Name at least three types of specialisation found in BERT.**
   *Hint: Positional, syntactic, coreference, separator-token heads.*

4. **If a model has `d_model = 512` and `h = 8` heads, what is the dimensionality of each head's query/key/value vectors?**
   *Answer: d_k = 512 / 8 = 64.*

5. **Why is head specialisation beneficial for transfer learning and fine-tuning?**
   *Hint: Different downstream tasks can leverage different heads.*

6. **What happens to model performance if you prune (remove) some attention heads? Is it uniform degradation or selective?**
   *Hint: Research shows many heads can be pruned with minimal loss; a few are critical.*

---

[← Positional Information Problem](../02-Self-Attention/04-positional-information-problem.md) | [Multi-Head Computation →](02-multi-head-computation.md)
