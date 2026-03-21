# Label Smoothing Regularization

> **Navigation:**
> Previous: [← Masked Self-Attention](01-masked-self-attention.md) | Next: [Warmup Schedules →](03-warmup-schedules.md)

---

## Overview

Label smoothing is a regularization technique that replaces **hard one-hot target distributions** with **soft target distributions** during training. Instead of assigning probability 1.0 to the correct class and 0.0 to all others, label smoothing distributes a small fraction ε of probability mass uniformly across all classes. Introduced by Szegedy et al. (2016) for image classification, it became a critical component of the original Transformer (Vaswani et al., 2017) with ε = 0.1. Label smoothing prevents the model from becoming overconfident in its predictions, improves generalization to unseen data, and produces better-calibrated probability estimates. It is now standard practice in training large language models, machine translation systems, and vision transformers.

---

## 1. The Problem: Overconfident Hard Targets

### One-Hot Encoding

For a vocabulary of size K = 5, if the correct class is index 2:

```
y_hard = [0, 0, 1, 0, 0]
```

### Why This Is Problematic

The model is trained to maximize `log P(correct class)`, pushing logits toward:

```
Ideal logits for hard targets:  [-∞, -∞, +∞, -∞, -∞]
```

This creates several issues:

| Problem | Explanation |
|---------|-------------|
| **Overconfidence** | Model outputs near-100% probability for one class |
| **Poor generalization** | Extreme logits mean model "memorizes" rather than learns soft patterns |
| **Sensitivity to noise** | Mislabeled examples get infinite loss gradient |
| **Large logit magnitudes** | Drives weights to extreme values, hurting generalization |
| **Poor calibration** | Model's confidence doesn't match actual accuracy |

---

## 2. The Label Smoothing Formula

### Smoothed Target Distribution

Given smoothing parameter ε (epsilon), the smoothed label is:

```
y_smooth(k) = (1 - ε) · y_hot(k) + ε / K
```

Where:
- `y_hot(k)` = 1 if k is the correct class, 0 otherwise
- `K` = total number of classes (vocabulary size)
- `ε` = smoothing parameter (typically 0.1)

### Expanded Form

For the **correct class** (k = c):
```
y_smooth(c) = (1 - ε) + ε/K = 1 - ε + ε/K = 1 - ε(1 - 1/K) = 1 - ε(K-1)/K
```

For **incorrect classes** (k ≠ c):
```
y_smooth(k) = 0 + ε/K = ε/K
```

### Verification: Probabilities Sum to 1

```
Sum = y_smooth(c) + (K-1) · y_smooth(k≠c)
    = [1 - ε(K-1)/K] + (K-1) · [ε/K]
    = 1 - ε(K-1)/K + ε(K-1)/K
    = 1  ✓
```

---

## 3. Worked Numerical Example

### Setup

- Vocabulary size: **K = 5**
- Correct class: **index 2**
- Smoothing: **ε = 0.1**

### Step 1: Hard Target

```
y_hard = [0.0, 0.0, 1.0, 0.0, 0.0]
```

### Step 2: Compute Smoothed Target

For correct class (index 2):
```
y_smooth(2) = (1 - 0.1) · 1.0 + 0.1 / 5 = 0.9 + 0.02 = 0.92
```

For incorrect classes (indices 0, 1, 3, 4):
```
y_smooth(k) = (1 - 0.1) · 0.0 + 0.1 / 5 = 0.0 + 0.02 = 0.02
```

### Result

```
y_hard   = [0.00, 0.00, 1.00, 0.00, 0.00]
y_smooth = [0.02, 0.02, 0.92, 0.02, 0.02]

Sum check: 0.02 + 0.02 + 0.92 + 0.02 + 0.02 = 1.00  ✓
```

### Step 3: Compare Cross-Entropy Losses

Suppose the model predicts: `p = [0.05, 0.05, 0.80, 0.05, 0.05]`

**With hard targets:**
```
L_hard = -log(0.80) = 0.2231
```

**With smoothed targets:**
```
L_smooth = -(0.02·log(0.05) + 0.02·log(0.05) + 0.92·log(0.80) 
           + 0.02·log(0.05) + 0.02·log(0.05))
         = -(0.02·(-2.996) + 0.02·(-2.996) + 0.92·(-0.223) 
           + 0.02·(-2.996) + 0.02·(-2.996))
         = -((-0.0599) + (-0.0599) + (-0.2052) + (-0.0599) + (-0.0599))
         = -(-0.4448)
         = 0.4448
```

The smoothed loss is higher because the model is also penalized for assigning low probability (0.05) to non-target classes — encouraging it to be less extreme.

---

## 4. ASCII Diagram: Hard vs. Smoothed Targets

```
  Hard Target (ε = 0)                   Smoothed Target (ε = 0.1, K = 5)

  Probability                            Probability
  1.0 │       ██                         1.0 │
      │       ██                             │
  0.8 │       ██                         0.8 │       ██  (0.92)
      │       ██                             │       ██
  0.6 │       ██                         0.6 │       ██
      │       ██                             │       ██
  0.4 │       ██                         0.4 │       ██
      │       ██                             │       ██
  0.2 │       ██                         0.2 │       ██
      │       ██                             │  ██   ██   ██   ██  (0.02 each)
  0.0 └──██───██───██───██───██──        0.0 └──██───██───██───██──
         c0   c1   c2   c3   c4                c0   c1   c2   c3   c4
                   ▲                                      ▲
              correct class                          correct class
```

---

## 5. Connection to KL Divergence and Cross-Entropy

### Cross-Entropy with Soft Targets

The label-smoothing loss is simply the cross-entropy between the smoothed distribution `q` and the model's predicted distribution `p`:

```
L_LS = H(q, p) = -Σₖ q(k) · log p(k)
```

### Decomposition

This can be decomposed as:

```
L_LS = (1 - ε) · H(y_hot, p) + ε · H(u, p)
```

Where:
- `H(y_hot, p)` = standard cross-entropy with hard targets
- `H(u, p)` = cross-entropy with uniform distribution `u = 1/K`
- `H(u, p) = -1/K · Σₖ log p(k)` = negative mean log-probability

### Interpretation

```
L_LS = (1 - ε) · [standard CE loss] + ε · [penalty for non-uniform predictions]
```

The second term acts as a **regularizer** that pushes the model toward a uniform distribution, preventing any single logit from becoming too dominant.

### KL Divergence View

Equivalently, since `H(q, p) = H(q) + KL(q ‖ p)` and `H(q)` is constant:

```
Minimizing L_LS  ⟺  Minimizing KL(q_smooth ‖ p_model)
```

The model learns to match the **smooth target distribution**, not a point mass.

---

## 6. Effect on Model Calibration

### What Is Calibration?

A model is **well-calibrated** if when it says "80% confident," it is correct 80% of the time.

### How Label Smoothing Helps

```
                  Without Label Smoothing       With Label Smoothing
                  ┌─────────────────────┐       ┌─────────────────────┐
  Confidence      │ Model says 99%      │       │ Model says 85%      │
  on correct      │ but correct only    │       │ and correct about   │
  class           │ 85% of the time     │       │ 85% of the time     │
                  │                     │       │                     │
  Calibration     │ OVERCONFIDENT       │       │ WELL-CALIBRATED     │
                  │ (reliability gap)   │       │ (gap is small)      │
                  └─────────────────────┘       └─────────────────────┘
```

### Expected Calibration Error (ECE)

```
ECE = Σ_b (|B_b| / N) · |accuracy(B_b) - confidence(B_b)|
```

Label smoothing typically reduces ECE by 30-50% compared to hard targets.

---

## 7. Practical Considerations

### Choosing ε

| ε Value | Effect | Use Case |
|---------|--------|----------|
| 0.0 | No smoothing (standard CE) | Baseline |
| 0.01 | Very mild smoothing | Simple classification tasks |
| **0.1** | **Standard choice** | **Transformers, NMT, most tasks** |
| 0.2 | Aggressive smoothing | Very noisy labels |
| 0.3+ | Too aggressive | Rarely used — hurts accuracy |

### Interaction with Other Techniques

| Technique | Interaction with Label Smoothing |
|-----------|----------------------------------|
| **Temperature scaling** | Both affect calibration; may be redundant |
| **Mixup / CutMix** | Complementary — smooths in input space |
| **Dropout** | Complementary — different regularization axes |
| **Weight decay** | Complementary — regularizes weights, not targets |
| **Knowledge distillation** | Soft targets from teacher act similarly to label smoothing |

---

## 8. PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F


class LabelSmoothingCrossEntropy(nn.Module):
    """
    Cross-entropy loss with label smoothing.
    
    Instead of targeting 100% on the correct class, distributes
    ε probability mass uniformly across all classes.
    """

    def __init__(self, epsilon: float = 0.1, reduction: str = "mean"):
        super().__init__()
        self.epsilon = epsilon
        self.reduction = reduction

    def forward(self, logits: torch.Tensor, targets: torch.Tensor) -> torch.Tensor:
        """
        Args:
            logits: Model output logits, shape (batch_size, num_classes)
            targets: Ground truth class indices, shape (batch_size,)
        Returns:
            Scalar loss value
        """
        K = logits.size(-1)  # Number of classes

        # Compute log-softmax for numerical stability
        log_probs = F.log_softmax(logits, dim=-1)

        # Standard NLL loss component: -log p(correct class)
        nll_loss = -log_probs.gather(dim=-1, index=targets.unsqueeze(-1)).squeeze(-1)

        # Uniform distribution component: -mean of all log probabilities
        smooth_loss = -log_probs.mean(dim=-1)

        # Combined loss
        loss = (1.0 - self.epsilon) * nll_loss + self.epsilon * smooth_loss

        if self.reduction == "mean":
            return loss.mean()
        elif self.reduction == "sum":
            return loss.sum()
        return loss


class LabelSmoothingCrossEntropyExplicit(nn.Module):
    """
    Alternative implementation that explicitly constructs smooth targets.
    More readable but slightly less memory-efficient.
    """

    def __init__(self, epsilon: float = 0.1):
        super().__init__()
        self.epsilon = epsilon

    def forward(self, logits: torch.Tensor, targets: torch.Tensor) -> torch.Tensor:
        K = logits.size(-1)

        # Build smoothed target distribution
        smooth_targets = torch.full_like(logits, self.epsilon / K)
        smooth_targets.scatter_(1, targets.unsqueeze(1), 1.0 - self.epsilon + self.epsilon / K)

        # Cross-entropy with the smooth distribution
        log_probs = F.log_softmax(logits, dim=-1)
        loss = -(smooth_targets * log_probs).sum(dim=-1)

        return loss.mean()


# --- Demo ---
if __name__ == "__main__":
    torch.manual_seed(42)

    batch_size = 4
    num_classes = 10
    logits = torch.randn(batch_size, num_classes)
    targets = torch.randint(0, num_classes, (batch_size,))

    # Standard cross-entropy
    ce_loss = F.cross_entropy(logits, targets)

    # Label smoothing (our implementation)
    ls_criterion = LabelSmoothingCrossEntropy(epsilon=0.1)
    ls_loss = ls_criterion(logits, targets)

    # PyTorch built-in (v1.10+)
    builtin_ls_loss = F.cross_entropy(logits, targets, label_smoothing=0.1)

    print(f"Standard CE loss:          {ce_loss.item():.4f}")
    print(f"Label smoothing loss:      {ls_loss.item():.4f}")
    print(f"PyTorch built-in LS loss:  {builtin_ls_loss.item():.4f}")

    # Verify our implementation matches PyTorch's
    print(f"\nDifference from built-in:  {abs(ls_loss.item() - builtin_ls_loss.item()):.6f}")

    # Show the smooth targets
    print(f"\nTargets: {targets}")
    smooth = torch.full((batch_size, num_classes), 0.1 / num_classes)
    smooth.scatter_(1, targets.unsqueeze(1), 1.0 - 0.1 + 0.1 / num_classes)
    print(f"Smooth target for sample 0: {smooth[0].tolist()}")
    print(f"Sum of smooth target:       {smooth[0].sum().item():.4f}")
```

---

## 9. Real-World Applications

| Application | Details |
|-------------|---------|
| **Original Transformer** | Used ε = 0.1 for English–German and English–French translation; improved BLEU by ~0.5 |
| **Inception v3** | Introduced label smoothing for ImageNet classification; reduced top-1 error |
| **BERT pre-training** | Some BERT variants use label smoothing for MLM objective |
| **Vision Transformers (ViT)** | Standard recipe uses ε = 0.1 for ImageNet training |
| **Speech Recognition** | Whisper and other ASR models use label smoothing to handle ambiguous transcriptions |
| **Recommender Systems** | Smooths click/no-click labels to handle noisy implicit feedback |

---

## 10. Summary Table

| Concept | Details |
|---------|---------|
| **Formula** | `y_smooth = (1-ε)·y_hot + ε/K` |
| **Typical ε** | 0.1 (Transformer default) |
| **Correct class target** | `1 - ε(K-1)/K ≈ 0.9` for ε=0.1 |
| **Incorrect class target** | `ε/K ≈ 0.01` for ε=0.1, K=10 |
| **Effect** | Reduces overconfidence, improves calibration |
| **Loss decomposition** | `(1-ε)·CE_loss + ε·uniform_penalty` |
| **PyTorch built-in** | `F.cross_entropy(logits, targets, label_smoothing=0.1)` |
| **Computational cost** | Negligible — only modifies target distribution |

---

## 11. Revision Questions

1. **If you have a vocabulary of K = 32,000 and use ε = 0.1, what is the target probability for the correct token and for each incorrect token?**
   *Hint: Apply the formula y_smooth(c) = 1 - ε(K-1)/K.*

2. **Why does label smoothing improve model calibration? Explain the mechanism in terms of logit magnitudes.**
   *Hint: Think about what happens to logits when the model tries to output probability 1.0 vs. 0.92.*

3. **Can label smoothing hurt performance? In what scenarios might you want to avoid it?**
   *Hint: Consider tasks where the model must be very confident, like safety-critical systems.*

4. **How does label smoothing compare to knowledge distillation with soft targets? What are the similarities and differences?**
   *Hint: In distillation, the soft targets come from a teacher model, not a uniform distribution.*

5. **Derive the gradient of the label-smoothing loss with respect to a logit zₖ. How does it differ from standard cross-entropy?**
   *Hint: Start from L = -(1-ε)·log p(c) - (ε/K)·Σₖ log p(k).*

6. **The original Transformer paper reports that label smoothing "hurts perplexity but improves BLEU score." Explain why these two metrics could move in opposite directions.**
   *Hint: Perplexity measures exact probability of the correct token; BLEU measures translation quality.*

---

> **Navigation:**
> Previous: [← Masked Self-Attention](01-masked-self-attention.md) | Next: [Warmup Schedules →](03-warmup-schedules.md)
