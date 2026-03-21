# Attention Weights and Alignment

> **Previous: [← Soft vs Hard Attention](02-soft-vs-hard-attention.md) | Next: [Query, Key, Value →](04-query-key-value.md)**

---

## Overview

Attention weights are the numeric scores that determine how much each input position contributes to the current output. They are computed from raw alignment (energy) scores via softmax normalization, producing a valid probability distribution over the source sequence. These weights can be visualized as an **alignment matrix** (or attention heatmap), offering powerful interpretability — we can literally see which source words the model considers most important when generating each target word. This chapter dives into the mechanics of computing attention weights, walks through a full numerical example, and demonstrates how to build and visualize attention heatmaps in Python.

---

## 1. Alignment Scores (Energy Scores)

The alignment score `e_{t,i}` measures the **compatibility** between the decoder state at time `t` and the encoder hidden state at position `i`.

### Common Scoring Functions

| Name | Formula | Introduced By |
|------|---------|---------------|
| **Additive (Bahdanau)** | `e_{t,i} = v^T tanh(W_1 h_i + W_2 s_t)` | Bahdanau et al., 2014 |
| **Dot-Product** | `e_{t,i} = s_t^T h_i` | Luong et al., 2015 |
| **Scaled Dot-Product** | `e_{t,i} = (s_t^T h_i) / √d_k` | Vaswani et al., 2017 |
| **General (Bilinear)** | `e_{t,i} = s_t^T W h_i` | Luong et al., 2015 |
| **Location-Based** | `e_{t,i} = W s_t` | Luong et al., 2015 |

### Why Scaling Matters (Scaled Dot-Product)

For high-dimensional vectors, the dot product grows in magnitude proportional to `√d_k`, pushing softmax into saturation regions where gradients vanish:

```
If s_t, h_i are d_k-dimensional with components ~ N(0,1):

    E[s_t^T h_i] = 0
    Var[s_t^T h_i] = d_k

Without scaling:  For d_k = 512, std ≈ 22.6 → softmax saturates
With scaling:     (s_t^T h_i) / √512 → std ≈ 1.0 → softmax behaves well
```

---

## 2. From Scores to Weights: Softmax Normalization

### Formula

```
α_{t,i} = softmax(e_{t,i}) = exp(e_{t,i}) / Σ_{j=1}^{T} exp(e_{t,j})
```

### Properties

```
1. Non-negativity:    α_{t,i} ≥ 0        ∀ i
2. Normalization:     Σ_i α_{t,i} = 1
3. Differentiable:    ∂α_{t,i}/∂e_{t,j} = α_{t,i}(δ_{ij} - α_{t,j})
```

Where `δ_{ij}` is the Kronecker delta (1 if i=j, 0 otherwise).

### Numerical Stability

In practice, we subtract the maximum score before computing softmax to prevent overflow:

```
α_{t,i} = exp(e_{t,i} - max_j(e_{t,j})) / Σ_j exp(e_{t,j} - max_j(e_{t,j}))
```

---

## 3. Attention Distribution: What the Weights Mean

The attention weight `α_{t,i}` represents the **probability** that source position `i` is the most relevant input for generating output at time step `t`.

```
┌───────────────────────────────────────────────────────┐
│              Attention Weight Distribution             │
│                                                       │
│  α_{t,i}                                              │
│  1.0 │                                                │
│      │                                                │
│  0.8 │        █                                       │
│      │        █                                       │
│  0.6 │        █                                       │
│      │        █                                       │
│  0.4 │        █                                       │
│      │        █                                       │
│  0.2 │   █    █    █                                  │
│      │   █    █    █    █                             │
│  0.0 ├───┴────┴────┴────┴────┬──── Source positions   │
│      h_1  h_2  h_3  h_4  h_5                         │
│                                                       │
│  Decoder focuses mostly on h_2 at this time step      │
└───────────────────────────────────────────────────────┘
```

### Interpreting Attention Weights

- **Peaked distribution** (one weight dominates): Model is very confident about alignment — common for well-aligned language pairs
- **Flat/uniform distribution**: Model is uncertain or information is spread across positions
- **Multi-modal distribution** (multiple peaks): Multiple source positions are relevant (e.g., compound words, idioms)

---

## 4. Attention Heatmap (Alignment Matrix)

The full set of attention weights across all decoder time steps forms an **alignment matrix** `A`:

```
A[t][i] = α_{t,i}

Rows: decoder time steps (target tokens)
Cols: encoder positions (source tokens)
```

### ASCII Attention Heatmap Example

Translation: "I love cats" → "J'aime les chats"

```
                    Source (Encoder)
                 I     love   cats
              ┌──────┬──────┬──────┐
   J'aime     │ ░░▓░ │ ████ │ ░░░░ │   α = [0.15, 0.75, 0.10]
              ├──────┼──────┼──────┤
   les        │ ░░░░ │ ░▓░░ │ ░▓▓░ │   α = [0.05, 0.25, 0.70]
   Target     ├──────┼──────┼──────┤
   (Decoder)  │ ░░░░ │ ░░░░ │ ████ │   α = [0.03, 0.07, 0.90]
   chats      └──────┴──────┴──────┘

   Legend:  ░░░░ = low weight (< 0.2)
            ░▓░░ = medium weight (0.2 - 0.5)
            ░▓▓░ = high weight (0.5 - 0.7)
            ████ = very high weight (> 0.7)
```

### What the Heatmap Reveals

- **Diagonal patterns**: Monotonic alignment (common in similar-structure language pairs like English-French)
- **Off-diagonal entries**: Reordering (e.g., adjective-noun order differences between English and Japanese)
- **One-to-many**: One source word maps to multiple target words
- **Many-to-one**: Multiple source words map to one target word

---

## 5. Worked Example with Full Numbers

### Problem Setup

**Task**: Translate "The cat sat" to "Le chat assis"

**Encoder hidden states** (dimension = 3):
```
h_1 ("The") = [ 0.5,  0.1, -0.3]
h_2 ("cat") = [-0.2,  0.9,  0.4]
h_3 ("sat") = [ 0.3,  0.0,  0.8]
```

**Decoder state** at step t=1 (generating "Le"):
```
s_1 = [0.4, -0.1, 0.2]
```

**Scoring function**: Dot-product `e_{t,i} = s_t^T h_i`

### Step 1: Compute Alignment Scores

```
e_{1,1} = s_1^T h_1 = (0.4)(0.5) + (-0.1)(0.1) + (0.2)(-0.3)
        = 0.20 - 0.01 - 0.06 = 0.13

e_{1,2} = s_1^T h_2 = (0.4)(-0.2) + (-0.1)(0.9) + (0.2)(0.4)
        = -0.08 - 0.09 + 0.08 = -0.09

e_{1,3} = s_1^T h_3 = (0.4)(0.3) + (-0.1)(0.0) + (0.2)(0.8)
        = 0.12 + 0.00 + 0.16 = 0.28

Scores: e = [0.13, -0.09, 0.28]
```

### Step 2: Softmax Normalization

```
exp(0.13)  = 1.139
exp(-0.09) = 0.914
exp(0.28)  = 1.323

Z = 1.139 + 0.914 + 1.323 = 3.376

α_{1,1} = 1.139 / 3.376 = 0.337
α_{1,2} = 0.914 / 3.376 = 0.271
α_{1,3} = 1.323 / 3.376 = 0.392

Attention weights: α = [0.337, 0.271, 0.392]
Check: 0.337 + 0.271 + 0.392 = 1.000 ✓
```

### Step 3: Compute Context Vector

```
c_1 = α_{1,1} × h_1 + α_{1,2} × h_2 + α_{1,3} × h_3

    = 0.337 × [ 0.5,  0.1, -0.3]    = [ 0.169,  0.034, -0.101]
    + 0.271 × [-0.2,  0.9,  0.4]    = [-0.054,  0.244,  0.108]
    + 0.392 × [ 0.3,  0.0,  0.8]    = [ 0.118,  0.000,  0.314]
                                     ──────────────────────────
    c_1                              = [ 0.233,  0.278,  0.321]
```

### Interpretation

The weights `[0.337, 0.271, 0.392]` are relatively uniform, which makes sense: "Le" (French for "The") draws information from the article "The" (0.337) but also needs context about what follows (the noun "cat" gets 0.271 and "sat" gets 0.392).

### Decoder Step t=2 (generating "chat")

Now assume `s_2 = [-0.3, 0.7, 0.1]`:

```
e_{2,1} = (-0.3)(0.5) + (0.7)(0.1) + (0.1)(-0.3) = -0.15 + 0.07 - 0.03 = -0.11
e_{2,2} = (-0.3)(-0.2) + (0.7)(0.9) + (0.1)(0.4) = 0.06 + 0.63 + 0.04 = 0.73
e_{2,3} = (-0.3)(0.3) + (0.7)(0.0) + (0.1)(0.8)  = -0.09 + 0.00 + 0.08 = -0.01

exp(-0.11) = 0.896,  exp(0.73) = 2.075,  exp(-0.01) = 0.990
Z = 0.896 + 2.075 + 0.990 = 3.961

α = [0.226, 0.524, 0.250]
```

Now `h_2` ("cat") dominates, which aligns with generating "chat" (French for "cat"). ✓

---

## 6. Python/PyTorch: Computing and Visualizing Attention Weights

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import numpy as np


class AttentionScorer(nn.Module):
    """Implements multiple attention scoring functions."""

    def __init__(self, hidden_dim, method="dot"):
        super().__init__()
        self.method = method
        self.hidden_dim = hidden_dim

        if method == "general":
            self.W = nn.Linear(hidden_dim, hidden_dim, bias=False)
        elif method == "additive":
            self.W1 = nn.Linear(hidden_dim, hidden_dim)
            self.W2 = nn.Linear(hidden_dim, hidden_dim)
            self.v = nn.Linear(hidden_dim, 1, bias=False)

    def forward(self, decoder_state, encoder_outputs):
        """
        Args:
            decoder_state:   (batch, hidden_dim)
            encoder_outputs: (batch, src_len, hidden_dim)
        Returns:
            weights: (batch, src_len)
        """
        query = decoder_state.unsqueeze(1)  # (batch, 1, hidden_dim)

        if self.method == "dot":
            # e_{t,i} = s_t^T h_i
            scores = torch.bmm(query, encoder_outputs.transpose(1, 2))
            scores = scores.squeeze(1)  # (batch, src_len)

        elif self.method == "scaled_dot":
            # e_{t,i} = (s_t^T h_i) / sqrt(d_k)
            scores = torch.bmm(query, encoder_outputs.transpose(1, 2))
            scores = scores.squeeze(1) / (self.hidden_dim ** 0.5)

        elif self.method == "general":
            # e_{t,i} = s_t^T W h_i
            transformed = self.W(encoder_outputs)  # (batch, src_len, hidden_dim)
            scores = torch.bmm(query, transformed.transpose(1, 2))
            scores = scores.squeeze(1)

        elif self.method == "additive":
            # e_{t,i} = v^T tanh(W1 h_i + W2 s_t)
            energy = torch.tanh(
                self.W1(encoder_outputs) + self.W2(query)
            )
            scores = self.v(energy).squeeze(-1)  # (batch, src_len)

        # Softmax normalization
        weights = F.softmax(scores, dim=-1)  # (batch, src_len)
        return weights, scores


# ─── Compute Attention Weights ───

HIDDEN_DIM = 3

# Manually set encoder hidden states from our worked example
encoder_outputs = torch.tensor([[
    [ 0.5,  0.1, -0.3],   # h_1 ("The")
    [-0.2,  0.9,  0.4],   # h_2 ("cat")
    [ 0.3,  0.0,  0.8],   # h_3 ("sat")
]])  # (1, 3, 3)

decoder_states = torch.tensor([
    [ 0.4, -0.1,  0.2],   # s_1 → generating "Le"
    [-0.3,  0.7,  0.1],   # s_2 → generating "chat"
    [ 0.1, -0.2,  0.9],   # s_3 → generating "assis"
])  # (3, 3)

scorer = AttentionScorer(HIDDEN_DIM, method="dot")

source_tokens = ["The", "cat", "sat"]
target_tokens = ["Le", "chat", "assis"]

print("=" * 50)
print("Attention Weights (Dot-Product Scoring)")
print("=" * 50)

alignment_matrix = []
for t in range(len(target_tokens)):
    state = decoder_states[t].unsqueeze(0)  # (1, 3)
    weights, scores = scorer(state, encoder_outputs)
    weights_np = weights.detach().numpy()[0]
    scores_np = scores.detach().numpy()[0]
    alignment_matrix.append(weights_np)

    print(f"\nDecoding step {t+1} → '{target_tokens[t]}'")
    print(f"  Raw scores:  {scores_np.round(4)}")
    print(f"  Weights:     {weights_np.round(4)}")
    print(f"  Focuses on:  '{source_tokens[np.argmax(weights_np)]}'")

alignment_matrix = np.array(alignment_matrix)


# ─── Visualization with Matplotlib ───

def plot_attention_heatmap(alignment, src_tokens, tgt_tokens, save_path=None):
    """
    Plot attention alignment heatmap.

    Args:
        alignment:  (tgt_len, src_len) numpy array
        src_tokens: list of source token strings
        tgt_tokens: list of target token strings
        save_path:  optional path to save figure
    """
    try:
        import matplotlib
        matplotlib.use('Agg')  # non-interactive backend
        import matplotlib.pyplot as plt

        fig, ax = plt.subplots(figsize=(6, 5))

        # Plot heatmap
        im = ax.imshow(alignment, cmap='YlOrRd', aspect='auto',
                        vmin=0, vmax=1)

        # Labels
        ax.set_xticks(range(len(src_tokens)))
        ax.set_yticks(range(len(tgt_tokens)))
        ax.set_xticklabels(src_tokens, fontsize=12)
        ax.set_yticklabels(tgt_tokens, fontsize=12)
        ax.set_xlabel("Source Tokens", fontsize=13)
        ax.set_ylabel("Target Tokens", fontsize=13)
        ax.set_title("Attention Alignment Heatmap", fontsize=14)

        # Add weight values in cells
        for i in range(len(tgt_tokens)):
            for j in range(len(src_tokens)):
                color = "white" if alignment[i, j] > 0.5 else "black"
                ax.text(j, i, f"{alignment[i, j]:.3f}",
                       ha="center", va="center", color=color, fontsize=11)

        plt.colorbar(im, ax=ax, label="Attention Weight")
        plt.tight_layout()

        if save_path:
            plt.savefig(save_path, dpi=150, bbox_inches='tight')
            print(f"\nHeatmap saved to: {save_path}")
        plt.close()

    except ImportError:
        print("\nmatplotlib not available — printing ASCII heatmap instead")
        print_ascii_heatmap(alignment, src_tokens, tgt_tokens)


def print_ascii_heatmap(alignment, src_tokens, tgt_tokens):
    """Fallback ASCII heatmap when matplotlib is unavailable."""
    chars = " ░▒▓█"

    # Header
    col_width = max(len(s) for s in src_tokens) + 2
    header = " " * (max(len(t) for t in tgt_tokens) + 3)
    header += "".join(s.center(col_width) for s in src_tokens)
    print(header)
    print(" " * (max(len(t) for t in tgt_tokens) + 3) + "─" * (col_width * len(src_tokens)))

    for i, tgt in enumerate(tgt_tokens):
        row = f"{tgt:>{max(len(t) for t in tgt_tokens)}} │ "
        for j in range(len(src_tokens)):
            w = alignment[i, j]
            bar_len = int(w * (col_width - 1))
            char_idx = min(int(w * 4), 4)
            cell = chars[char_idx] * bar_len + " " * (col_width - 1 - bar_len)
            row += cell + " "
        row += f"  [{', '.join(f'{alignment[i,j]:.3f}' for j in range(len(src_tokens)))}]"
        print(row)


# Plot (or print ASCII fallback)
print("\n" + "=" * 50)
print("Attention Heatmap")
print("=" * 50)
plot_attention_heatmap(alignment_matrix, source_tokens, target_tokens,
                       save_path="attention_heatmap.png")

# Also print ASCII version
print("\nASCII Heatmap:")
print_ascii_heatmap(alignment_matrix, source_tokens, target_tokens)
```

---

## 7. Attention Weight Patterns in Practice

### Common Patterns

```
1. MONOTONIC ALIGNMENT          2. REORDERING
   (English → French)             (English → Japanese)

   ┌─────────────┐               ┌─────────────┐
   │ █ ░ ░ ░ ░ ░ │               │ ░ ░ ░ ░ ░ █ │
   │ ░ █ ░ ░ ░ ░ │               │ ░ ░ ░ ░ █ ░ │
   │ ░ ░ █ ░ ░ ░ │               │ ░ ░ ░ █ ░ ░ │
   │ ░ ░ ░ █ ░ ░ │               │ ░ ░ █ ░ ░ ░ │
   │ ░ ░ ░ ░ █ ░ │               │ ░ █ ░ ░ ░ ░ │
   │ ░ ░ ░ ░ ░ █ │               │ █ ░ ░ ░ ░ ░ │
   └─────────────┘               └─────────────┘
   Diagonal = ordered             Anti-diagonal = reversed

3. ONE-TO-MANY                  4. MANY-TO-ONE
   (Compound expansion)           (Phrase compression)

   ┌─────────────┐               ┌─────────────┐
   │ █ ░ ░ ░ ░ ░ │               │ ▓ ▓ ░ ░ ░ ░ │
   │ █ ░ ░ ░ ░ ░ │               │ ░ ░ █ ░ ░ ░ │
   │ ░ █ ░ ░ ░ ░ │               │ ░ ░ ░ ▓ ▓ ▓ │
   │ ░ ░ █ ░ ░ ░ │               │ ░ ░ ░ ░ ░ █ │
   └─────────────┘               └─────────────┘
   One source → many target       Many source → one target
```

---

## 8. Attention Weights ≠ Explanation

### Important Caveat

While attention weights are often used for interpretability, **attention weights are not always faithful explanations** of model behavior (Jain & Wallace, 2019):

```
Concerns:
├── Alternative weight distributions can produce similar outputs
├── Attention may reflect input redundancy, not true importance
├── Gradient-based methods may be more reliable for explanations
└── In multi-head attention, individual heads can be redundant
```

Use attention heatmaps as a **debugging and exploration tool**, but be cautious about treating them as ground-truth explanations.

---

## 9. Real-World Applications

| Application | How Attention Weights Are Used |
|-------------|-------------------------------|
| **Machine Translation** | Alignment visualization between source and target languages |
| **Document Summarization** | Identifying key sentences that contribute to the summary |
| **Clinical NLP** | Highlighting which medical terms triggered a diagnosis prediction |
| **Sentiment Analysis** | Showing which words drove positive/negative classification |
| **Code Generation** | Mapping natural language descriptions to relevant code tokens |
| **Legal AI** | Highlighting contract clauses relevant to a specific query |

---

## 10. Summary Table

| Concept | Description |
|---------|-------------|
| **Alignment score `e_{t,i}`** | Raw compatibility between decoder state `s_t` and encoder state `h_i` |
| **Scoring functions** | Dot-product, scaled dot-product, additive (Bahdanau), general (Luong) |
| **Softmax normalization** | Converts raw scores to probabilities: `α = softmax(e)` |
| **Attention weights `α_{t,i}`** | Probability that source position `i` is relevant at decoder step `t` |
| **Context vector `c_t`** | Weighted sum: `c_t = Σ α_{t,i} h_i` |
| **Alignment matrix** | Full `T_target × T_source` matrix of attention weights |
| **Heatmap patterns** | Monotonic, reordering, one-to-many, many-to-one |
| **Scaling factor `√d_k`** | Prevents softmax saturation in high dimensions |
| **Interpretability caveat** | Weights ≠ faithful explanations (Jain & Wallace, 2019) |

---

## 11. Revision Questions

1. **Name four different scoring functions** for computing alignment scores. Write the formula for each and state who introduced them.

2. **Why do we divide dot-product scores by `√d_k`?** Derive the variance of the dot product for random vectors and explain the saturation problem.

3. **Given encoder states and decoder state**, manually compute attention weights using dot-product scoring:
   - `h_1 = [1, 2]`, `h_2 = [3, 1]`, `h_3 = [0, 4]`
   - `s_t = [2, 1]`

4. **Interpret the following alignment matrix pattern**: a strong diagonal with occasional off-diagonal peaks. What type of translation task might produce this?

5. **Why should we be cautious about using attention weights as explanations?** Cite at least two reasons from recent research.

6. **Describe two practical applications** where visualizing attention weights provides actionable insights. How would a practitioner use these heatmaps?

---

> **Previous: [← Soft vs Hard Attention](02-soft-vs-hard-attention.md) | Next: [Query, Key, Value →](04-query-key-value.md)**
