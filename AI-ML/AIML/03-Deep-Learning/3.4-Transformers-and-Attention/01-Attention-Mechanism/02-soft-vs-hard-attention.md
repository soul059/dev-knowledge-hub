# Soft vs Hard Attention

> **Previous: [← Attention Intuition](01-attention-intuition.md) | Next: [Attention Weights →](03-attention-weights.md)**

---

## Overview

Attention mechanisms come in two fundamental flavors: **soft (deterministic) attention** and **hard (stochastic) attention**. Soft attention computes a smooth, differentiable weighted average over all input positions, making it easy to train with standard backpropagation. Hard attention, by contrast, makes a discrete selection — attending to exactly one (or a few) input positions — which is non-differentiable and requires reinforcement learning techniques like REINFORCE for training. Understanding the trade-offs between these two approaches is essential for choosing the right mechanism for a given task. This chapter lays out the mathematical formulations, compares their properties, and provides working PyTorch implementations of both.

---

## 1. Soft Attention (Deterministic)

### Intuition

Soft attention is like reading a page through a **translucent spotlight** that covers everything but shines brightest on the most relevant parts. Every input element contributes to the output, but with different weights.

```
Source tokens:    x_1     x_2     x_3     x_4     x_5
                   │       │       │       │       │
Attention wts:   0.05    0.10    0.60    0.20    0.05
                   │       │       │       │       │
                   ▼       ▼       ▼       ▼       ▼
              ┌──────────────────────────────────────┐
              │     Weighted Sum (all contribute)     │
              │     c = Σ α_i × h_i                  │
              └──────────────┬───────────────────────┘
                             │
                             ▼
                       Context Vector
```

### Mathematical Formulation

Given encoder hidden states `H = {h_1, h_2, ..., h_T}` and the current decoder state `s_t`:

**Step 1 — Compute alignment scores:**
```
e_{t,i} = score(s_t, h_i)       for i = 1, ..., T
```

**Step 2 — Normalize with softmax:**
```
α_{t,i} = exp(e_{t,i}) / Σ_{j=1}^{T} exp(e_{t,j})

Properties:  0 ≤ α_{t,i} ≤ 1   and   Σ_i α_{t,i} = 1
```

**Step 3 — Compute context vector (expected value):**
```
c_t = Σ_{i=1}^{T} α_{t,i} × h_i = E_α[h_i]
```

### Key Property: Differentiability

Because softmax and weighted summation are both continuous and differentiable operations:

```
∂c_t / ∂α_{t,i} = h_i                    (exists and is well-defined)
∂α_{t,i} / ∂e_{t,i} = α_{t,i}(1 - α_{t,i})   (softmax gradient)
```

This means we can compute `∂L/∂θ` for all parameters `θ` using standard backpropagation.

---

## 2. Hard Attention (Stochastic)

### Intuition

Hard attention is like using an **opaque spotlight** — it illuminates exactly one position and leaves everything else in the dark. The model must decide where to point the spotlight.

```
Source tokens:    x_1     x_2     x_3     x_4     x_5
                   │       │       │       │       │
Attention wts:    0       0       1       0       0
                                  │
                                  ▼
              ┌──────────────────────────────────────┐
              │     Select ONE (discrete choice)      │
              │     c = h_3                           │
              └──────────────┬───────────────────────┘
                             │
                             ▼
                       Context Vector
```

### Mathematical Formulation

Define a latent categorical variable `z_t` that selects which input to attend to:

**Step 1 — Compute selection probabilities:**
```
p(z_t = i | s_t, H) = α_{t,i} = softmax(e_{t,i})
```

**Step 2 — Sample a position:**
```
z_t ~ Categorical(α_{t,1}, α_{t,2}, ..., α_{t,T})
```

**Step 3 — Context vector is the selected hidden state:**
```
c_t = h_{z_t}
```

This is equivalent to using a **one-hot** attention vector:

```
â_{t,i} = 1  if  i = z_t
â_{t,i} = 0  otherwise

c_t = Σ_i â_{t,i} × h_i = h_{z_t}
```

### The Differentiability Problem

Sampling from a categorical distribution is **non-differentiable**:

```
∂ z_t / ∂ α_{t,i}  →  UNDEFINED  (discrete operation)
```

We cannot backpropagate through the sampling step. Instead, we optimize the **expected loss** using the REINFORCE algorithm (Williams, 1992).

### REINFORCE Gradient Estimator

The objective is to minimize the expected loss:

```
J(θ) = E_{z_t ~ p(z_t|θ)} [ L(z_t) ]
```

Using the log-derivative trick:

```
∇_θ J(θ) = E_{z_t} [ L(z_t) × ∇_θ log p(z_t | θ) ]
```

In practice, approximate with Monte Carlo sampling (N samples):

```
∇_θ J(θ) ≈ (1/N) Σ_{n=1}^{N} L(z_t^(n)) × ∇_θ log p(z_t^(n) | θ)
```

### Variance Reduction with Baseline

REINFORCE gradients have high variance. A common fix is to subtract a **baseline** `b`:

```
∇_θ J(θ) ≈ (1/N) Σ_n (L(z_t^(n)) - b) × ∇_θ log p(z_t^(n) | θ)
```

A good baseline is the **running average** of the loss.

---

## 3. Visual Comparison

```
┌────────────────────────┐    ┌────────────────────────┐
│     SOFT ATTENTION     │    │     HARD ATTENTION     │
├────────────────────────┤    ├────────────────────────┤
│                        │    │                        │
│  Weights: [.1 .2 .5 .2]│    │  Weights: [0  0  1  0] │
│           ▓░░░▓▓▓▓▓░░░ │    │           ░░░░████░░░░ │
│                        │    │                        │
│  All positions used    │    │  One position selected │
│  Deterministic         │    │  Stochastic            │
│  ✓ Differentiable      │    │  ✗ Not differentiable  │
│  ✓ Backprop works      │    │  ✗ Needs REINFORCE     │
│                        │    │                        │
│  c = 0.1h₁ + 0.2h₂    │    │  c = h₃               │
│    + 0.5h₃ + 0.2h₄    │    │                        │
│                        │    │                        │
└────────────────────────┘    └────────────────────────┘
```

---

## 4. Detailed Comparison Table

| Property | Soft Attention | Hard Attention |
|----------|---------------|----------------|
| **Type** | Deterministic | Stochastic |
| **Output** | Weighted average of all inputs | Single selected input |
| **Differentiable?** | ✅ Yes | ❌ No |
| **Training method** | Standard backpropagation | REINFORCE / policy gradient |
| **Gradient variance** | Low (exact gradients) | High (Monte Carlo estimates) |
| **Computation cost** | O(T) per step (access all inputs) | O(1) per step (access one input) |
| **Memory at inference** | Must store all hidden states | Can discard non-selected states |
| **Interpretability** | Weights show relative importance | Clear: "model looked here" |
| **Long sequences** | Expensive (all positions) | Efficient (one position) |
| **Typical use cases** | NMT, summarization, QA | Image captioning, visual QA |
| **Training stability** | Stable | Unstable (high variance) |
| **Introduced by** | Bahdanau et al., 2014 | Xu et al., 2015 |

---

## 5. Worked Example: Soft vs Hard

### Setup

Encoder hidden states (dim=3):

```
h_1 = [1.0, 0.0, 0.5]
h_2 = [0.0, 1.0, 0.3]
h_3 = [0.5, 0.5, 1.0]
h_4 = [0.2, 0.8, 0.1]
```

Alignment scores (pre-softmax): `e = [1.2, 3.1, 0.5, 2.0]`

### Soft Attention Computation

```
Softmax:   α_1 = exp(1.2) / Z = 3.32 / 33.39 = 0.099
           α_2 = exp(3.1) / Z = 22.20 / 33.39 = 0.665
           α_3 = exp(0.5) / Z = 1.65 / 33.39 = 0.049
           α_4 = exp(2.0) / Z = 7.39 / 33.39 = 0.221

   (Z = 3.32 + 22.20 + 1.65 + 7.39 = 34.56)

   Corrected:
           α_1 = 3.32 / 34.56 = 0.096
           α_2 = 22.20 / 34.56 = 0.642
           α_3 = 1.65 / 34.56 = 0.048
           α_4 = 7.39 / 34.56 = 0.214

Context:   c = 0.096 × [1.0, 0.0, 0.5]    = [0.096, 0.000, 0.048]
             + 0.642 × [0.0, 1.0, 0.3]    = [0.000, 0.642, 0.193]
             + 0.048 × [0.5, 0.5, 1.0]    = [0.024, 0.024, 0.048]
             + 0.214 × [0.2, 0.8, 0.1]    = [0.043, 0.171, 0.021]
                                           ───────────────────────
           c                               = [0.163, 0.837, 0.310]
```

### Hard Attention Computation

```
Sample z ~ Categorical(0.096, 0.642, 0.048, 0.214)

Most likely outcome: z = 2  (64.2% probability)

Context:  c = h_2 = [0.0, 1.0, 0.3]
```

Notice: hard attention produces a "sharper" context but loses information from other positions.

---

## 6. When to Use Each

### Use Soft Attention When:
- ✅ End-to-end training with backpropagation is desired
- ✅ All input positions may be relevant (e.g., translation)
- ✅ Training stability is important
- ✅ You want smooth, interpretable attention distributions
- ✅ Sequence lengths are moderate (< 1000 tokens)

### Use Hard Attention When:
- ✅ Input is very large and accessing all positions is too expensive
- ✅ Task naturally involves discrete selection (e.g., "which image region?")
- ✅ Computational efficiency at inference is critical
- ✅ The task tolerates noisy gradients and slower convergence
- ✅ Used with Gumbel-Softmax for a differentiable approximation

### Modern Trend: Sparse Attention as a Middle Ground

Techniques like **top-k attention**, **sparse transformers**, and **Gumbel-Softmax** offer compromises:

```
Full Soft                 Sparse                    Hard
[.05 .10 .60 .20 .05]    [0  0  .75 .25  0]       [0  0  1  0  0]
    All positions          Top-k positions           One position
    Differentiable         Approx. differentiable    Non-differentiable
```

---

## 7. Python/PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.distributions import Categorical


class SoftAttention(nn.Module):
    """Standard soft (deterministic) attention mechanism."""

    def __init__(self, hidden_dim):
        super().__init__()
        self.W1 = nn.Linear(hidden_dim, hidden_dim)
        self.W2 = nn.Linear(hidden_dim, hidden_dim)
        self.v = nn.Linear(hidden_dim, 1)

    def forward(self, decoder_state, encoder_outputs):
        """
        Args:
            decoder_state:   (batch, hidden_dim)
            encoder_outputs: (batch, src_len, hidden_dim)
        Returns:
            context: (batch, hidden_dim)
            weights: (batch, src_len)
        """
        # Expand decoder state for broadcasting
        query = decoder_state.unsqueeze(1)             # (batch, 1, hidden_dim)

        # Alignment scores
        energy = torch.tanh(
            self.W1(encoder_outputs) + self.W2(query)
        )                                               # (batch, src_len, hidden_dim)
        scores = self.v(energy).squeeze(-1)            # (batch, src_len)

        # Soft attention weights (differentiable)
        weights = F.softmax(scores, dim=-1)            # (batch, src_len)

        # Weighted sum → context vector
        context = torch.bmm(
            weights.unsqueeze(1), encoder_outputs
        ).squeeze(1)                                    # (batch, hidden_dim)

        return context, weights


class HardAttention(nn.Module):
    """Stochastic hard attention with REINFORCE training."""

    def __init__(self, hidden_dim):
        super().__init__()
        self.W1 = nn.Linear(hidden_dim, hidden_dim)
        self.W2 = nn.Linear(hidden_dim, hidden_dim)
        self.v = nn.Linear(hidden_dim, 1)
        self.baseline = 0.0     # running average for variance reduction
        self.baseline_decay = 0.9

    def forward(self, decoder_state, encoder_outputs):
        """
        Args:
            decoder_state:   (batch, hidden_dim)
            encoder_outputs: (batch, src_len, hidden_dim)
        Returns:
            context:   (batch, hidden_dim)
            weights:   (batch, src_len) — one-hot if hard
            log_prob:  (batch,) — for REINFORCE loss
        """
        query = decoder_state.unsqueeze(1)

        energy = torch.tanh(
            self.W1(encoder_outputs) + self.W2(query)
        )
        scores = self.v(energy).squeeze(-1)             # (batch, src_len)
        probs = F.softmax(scores, dim=-1)               # (batch, src_len)

        # Sample from categorical distribution
        dist = Categorical(probs)
        z = dist.sample()                                # (batch,) — sampled indices
        log_prob = dist.log_prob(z)                      # (batch,)

        # One-hot attention vector
        one_hot = F.one_hot(z, num_classes=encoder_outputs.size(1))
        weights = one_hot.float()                        # (batch, src_len)

        # Context = selected hidden state
        context = torch.bmm(
            weights.unsqueeze(1), encoder_outputs
        ).squeeze(1)                                     # (batch, hidden_dim)

        return context, weights, log_prob

    def reinforce_loss(self, task_loss, log_prob):
        """
        Compute REINFORCE loss for hard attention.

        Args:
            task_loss: scalar, the main task loss (e.g., cross-entropy)
            log_prob:  (batch,) log probabilities from sampling

        Returns:
            reinforce_loss: scalar to be added to the main loss
        """
        reward = -task_loss.item()   # negative loss as reward
        advantage = reward - self.baseline
        self.baseline = (
            self.baseline_decay * self.baseline
            + (1 - self.baseline_decay) * reward
        )
        return -(advantage * log_prob).mean()


# ─── Demo ───

BATCH, SRC_LEN, HIDDEN = 4, 10, 64

encoder_outputs = torch.randn(BATCH, SRC_LEN, HIDDEN)
decoder_state = torch.randn(BATCH, HIDDEN)

# Soft attention
soft_attn = SoftAttention(HIDDEN)
soft_ctx, soft_wts = soft_attn(decoder_state, encoder_outputs)
print("=== Soft Attention ===")
print(f"Context shape: {soft_ctx.shape}")          # (4, 64)
print(f"Weights shape: {soft_wts.shape}")          # (4, 10)
print(f"Weights sum:   {soft_wts.sum(dim=-1)}")    # all 1.0
print(f"Weights[0]:    {soft_wts[0].data.numpy().round(3)}")
print(f"Non-zero per sample: {(soft_wts > 0.001).sum(dim=-1)}")

print()

# Hard attention
hard_attn = HardAttention(HIDDEN)
hard_ctx, hard_wts, hard_lp = hard_attn(decoder_state, encoder_outputs)
print("=== Hard Attention ===")
print(f"Context shape: {hard_ctx.shape}")          # (4, 64)
print(f"Weights shape: {hard_wts.shape}")          # (4, 10) — one-hot
print(f"Weights sum:   {hard_wts.sum(dim=-1)}")    # all 1.0
print(f"Weights[0]:    {hard_wts[0].data.numpy()}")
print(f"Log prob:      {hard_lp.data}")
print(f"Non-zero per sample: {(hard_wts > 0).sum(dim=-1)}")  # always 1

# Simulate REINFORCE training step
criterion = nn.MSELoss()
target = torch.randn(BATCH, HIDDEN)
task_loss = criterion(hard_ctx, target)
rl_loss = hard_attn.reinforce_loss(task_loss, hard_lp)
total_loss = task_loss + 0.1 * rl_loss   # balance task + RL loss
print(f"\nTask loss:      {task_loss.item():.4f}")
print(f"REINFORCE loss: {rl_loss.item():.4f}")
print(f"Total loss:     {total_loss.item():.4f}")
```

---

## 8. Gumbel-Softmax: Best of Both Worlds

A modern technique to make hard attention approximately differentiable:

```python
def gumbel_softmax_attention(scores, tau=1.0, hard=True):
    """
    Gumbel-Softmax: differentiable approximation of hard attention.

    Args:
        scores: (batch, src_len) raw alignment scores
        tau:    temperature (lower → sharper, more like hard attention)
        hard:   if True, use straight-through estimator
    """
    # Sample Gumbel noise
    gumbels = -torch.log(-torch.log(torch.rand_like(scores) + 1e-20) + 1e-20)

    # Add noise and apply temperature-scaled softmax
    y_soft = F.softmax((scores + gumbels) / tau, dim=-1)

    if hard:
        # Straight-through: forward pass uses argmax, backward uses soft
        index = y_soft.argmax(dim=-1, keepdim=True)
        y_hard = torch.zeros_like(y_soft).scatter_(-1, index, 1.0)
        return y_hard - y_soft.detach() + y_soft  # gradient flows through y_soft
    else:
        return y_soft


# Demo: temperature effect
scores = torch.tensor([[1.0, 3.0, 0.5, 2.0]])
print("\nGumbel-Softmax at different temperatures:")
for tau in [5.0, 1.0, 0.5, 0.1]:
    result = gumbel_softmax_attention(scores, tau=tau, hard=False)
    print(f"  τ={tau:<4} → {result.data.numpy().round(3)}")
```

---

## 9. Real-World Applications

| Mechanism | Application | Why This Choice? |
|-----------|-------------|-----------------|
| **Soft** | Neural Machine Translation | All source words can be relevant; need stable training |
| **Soft** | Text Summarization | Multiple sentences contribute to each summary sentence |
| **Soft** | Transformer models (BERT, GPT) | All-to-all attention; fully differentiable |
| **Hard** | Image Captioning (Show, Attend, Tell) | Focus on one image region per word |
| **Hard** | Visual Question Answering | Select specific image patch for answer |
| **Gumbel** | Discrete latent variable models | Need differentiable discrete choices |
| **Sparse** | Long document processing | Too expensive to attend to all positions |

---

## 10. Summary Table

| Aspect | Soft | Hard | Gumbel-Softmax |
|--------|------|------|----------------|
| **Attention vector** | Dense (all > 0) | One-hot | Approx. one-hot |
| **Differentiable** | ✅ | ❌ | ✅ (approx.) |
| **Training** | Backprop | REINFORCE | Backprop + noise |
| **Gradient quality** | Exact | High variance | Low variance |
| **Compute (forward)** | O(T) | O(1) | O(T) |
| **Sharpness** | Smooth | Maximum | Controllable (τ) |
| **Implementation** | Simple | Complex | Moderate |

---

## 11. Revision Questions

1. **Define soft attention mathematically.** Write out the three steps (alignment scores, softmax, weighted sum) and explain why each is differentiable.

2. **Why can't we train hard attention with standard backpropagation?** What makes the sampling operation non-differentiable?

3. **Explain the REINFORCE gradient estimator** for hard attention. What is the role of the baseline, and why is it needed?

4. **Given alignment scores `e = [2.0, 0.5, 1.5]`**, compute the context vector under (a) soft attention and (b) hard attention (assume the sampled index is the argmax), with `h_1 = [1, 0]`, `h_2 = [0, 1]`, `h_3 = [1, 1]`.

5. **What is Gumbel-Softmax?** How does it provide a differentiable approximation to hard attention? What role does the temperature parameter τ play?

6. **In what scenarios would you choose hard attention over soft attention?** Give two real-world examples and justify your choice.

---

> **Previous: [← Attention Intuition](01-attention-intuition.md) | Next: [Attention Weights →](03-attention-weights.md)**
