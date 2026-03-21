# 🔧 Custom Loss Functions

> **Chapter 3.4 — Building Your Own Loss Functions for Domain-Specific Problems**

| Topic | Details |
|---|---|
| **Subject** | Loss Functions — Custom & Advanced Losses |
| **Prerequisites** | Chapters 3.1-3.3, PyTorch basics |
| **Key Insight** | Custom losses encode domain knowledge into the optimization objective — the gradient must exist for backpropagation to work |

---

## 📌 Overview

Standard loss functions cover the majority of use cases, but real-world problems often require custom objectives. A recommendation system might need to optimize for ranking rather than classification accuracy. A medical imaging model might need to combine pixel-level accuracy with structural similarity. This chapter shows you how to design, implement, and validate custom loss functions in PyTorch.

---

## 1. Why Custom Losses?

### 1.1 When Standard Losses Fall Short

```
    ┌────────────────────────────────────────────────────────────────┐
    │                 WHY GO CUSTOM?                                  │
    ├────────────────────────────────────────────────────────────────┤
    │                                                                │
    │  1. DOMAIN-SPECIFIC REQUIREMENTS                               │
    │     • Medical: Penalize false negatives more than FP           │
    │     • Finance: Asymmetric loss (underestimate vs overestimate) │
    │     • Robotics: Physical constraint violations                 │
    │                                                                │
    │  2. MULTI-TASK LEARNING                                        │
    │     • Combine classification + regression + auxiliary losses    │
    │     • Balance multiple objectives                              │
    │                                                                │
    │  3. STRUCTURED OUTPUTS                                         │
    │     • Image segmentation: Pixel-wise + region-wise accuracy    │
    │     • Object detection: Location + class + objectness          │
    │                                                                │
    │  4. METRIC LEARNING                                            │
    │     • Triplet loss for similarity learning                     │
    │     • Contrastive loss for representation learning             │
    │                                                                │
    │  5. ADVERSARIAL TRAINING                                       │
    │     • GAN losses: Generator vs discriminator objectives        │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 2. The Fundamental Rule: Differentiability

### 2.1 Why Gradients Must Exist

```
    Backpropagation requires computing:

    ∂L/∂θ  (gradient of loss w.r.t. every parameter)

    If the loss function is NOT differentiable at some point,
    the gradient doesn't exist → backprop fails → no learning!

    ┌─────────────────────────────────────────────────────┐
    │  RULE: Every operation in your custom loss must be   │
    │  differentiable (or have a useful subgradient).      │
    │                                                      │
    │  ✅ OK:  +, -, ×, ÷, exp, log, pow, abs, clamp     │
    │  ✅ OK:  torch functions (autograd handles them)     │
    │  ⚠️ Care: argmax, argmin (not differentiable)        │
    │  ❌ Bad:  if/else on tensor values (no gradient)     │
    │  ❌ Bad:  converting to numpy mid-computation        │
    │  ❌ Bad:  integer operations, sorting (usually)      │
    └─────────────────────────────────────────────────────┘
```

### 2.2 Workarounds for Non-Differentiable Operations

| Operation | Problem | Solution |
|---|---|---|
| **argmax** | Not differentiable | Use softmax (smooth approximation) |
| **step function** | Zero gradient everywhere except origin | Use sigmoid (smooth step) |
| **sorting/ranking** | Discrete operation | Differentiable sorting (NeuralSort) |
| **L0 norm** (count nonzero) | Discrete | L1 norm (closest convex relaxation) |
| **Accuracy** | Not differentiable (discrete) | Cross-entropy (smooth proxy) |
| **IoU** (intersection over union) | Ratio of discrete sets | Soft IoU / Dice loss |

---

## 3. Building Custom Losses in PyTorch

### 3.1 Method 1: Simple Function

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

def custom_mse_with_l1_reg(y_pred, y_true, model, lambda_l1=0.001):
    """MSE + L1 regularization on weights"""
    mse = F.mse_loss(y_pred, y_true)
    l1_reg = sum(p.abs().sum() for p in model.parameters())
    return mse + lambda_l1 * l1_reg

# Usage:
# loss = custom_mse_with_l1_reg(predictions, targets, model)
# loss.backward()
```

### 3.2 Method 2: nn.Module Class (Recommended)

```python
class WeightedMSELoss(nn.Module):
    """MSE with per-sample weights."""
    
    def __init__(self):
        super().__init__()
    
    def forward(self, y_pred, y_true, weights=None):
        errors = (y_pred - y_true) ** 2
        if weights is not None:
            errors = errors * weights
        return errors.mean()

# Usage:
criterion = WeightedMSELoss()
sample_weights = torch.tensor([1.0, 1.0, 5.0, 1.0])  # Weight sample 3 more
loss = criterion(predictions, targets, sample_weights)
loss.backward()
```

---

## 4. Practical Custom Loss Implementations

### 4.1 Asymmetric Loss (Different Penalties for Over/Under Prediction)

```python
class AsymmetricLoss(nn.Module):
    """
    Different penalties for over-prediction vs under-prediction.
    Useful in finance, medicine, inventory management.
    
    Example: Predicting demand
    - Over-predict → wasted inventory (cost = α)
    - Under-predict → lost sales (cost = β, typically β > α)
    """
    
    def __init__(self, alpha=1.0, beta=2.0):
        super().__init__()
        self.alpha = alpha  # over-prediction penalty
        self.beta = beta    # under-prediction penalty
    
    def forward(self, y_pred, y_true):
        error = y_pred - y_true
        loss = torch.where(
            error >= 0,
            self.alpha * error ** 2,   # over-prediction
            self.beta * error ** 2      # under-prediction (penalize more)
        )
        return loss.mean()

# Example
criterion = AsymmetricLoss(alpha=1.0, beta=3.0)
y_pred = torch.tensor([105.0, 95.0], requires_grad=True)  # Over, Under
y_true = torch.tensor([100.0, 100.0])
loss = criterion(y_pred, y_true)
print(f"Over-predict by 5:  penalty = {1.0 * 25:.1f}")
print(f"Under-predict by 5: penalty = {3.0 * 25:.1f}")
print(f"Total loss: {loss.item():.1f}")
```

### 4.2 Weighted Cross-Entropy (Multi-Task)

```python
class MultiTaskLoss(nn.Module):
    """
    Combine multiple loss functions for multi-task learning.
    Example: Object detection = Classification + Localization
    """
    
    def __init__(self, task_weights=None, learnable=False):
        super().__init__()
        if learnable:
            # Learnable task weights (uncertainty weighting, Kendall et al. 2018)
            self.log_vars = nn.Parameter(torch.zeros(len(task_weights)))
        else:
            self.task_weights = task_weights or [1.0, 1.0]
            self.log_vars = None
        
        self.ce_loss = nn.CrossEntropyLoss()
        self.mse_loss = nn.MSELoss()
    
    def forward(self, class_logits, class_targets, bbox_pred, bbox_targets):
        loss_cls = self.ce_loss(class_logits, class_targets)
        loss_bbox = self.mse_loss(bbox_pred, bbox_targets)
        
        if self.log_vars is not None:
            # Automatic weighting: L = Σ (1/2σ²ᵢ)Lᵢ + log(σᵢ)
            precision1 = torch.exp(-self.log_vars[0])
            precision2 = torch.exp(-self.log_vars[1])
            total = precision1 * loss_cls + self.log_vars[0] + \
                    precision2 * loss_bbox + self.log_vars[1]
        else:
            total = self.task_weights[0] * loss_cls + \
                    self.task_weights[1] * loss_bbox
        
        return total, {'cls': loss_cls.item(), 'bbox': loss_bbox.item()}
```

### 4.3 Triplet Loss (Metric Learning)

```python
class TripletLoss(nn.Module):
    """
    Learn embeddings where similar items are close, dissimilar items are far.
    
    For each triplet (anchor, positive, negative):
    L = max(0, d(a,p) - d(a,n) + margin)
    
    Want: d(anchor, positive) < d(anchor, negative) by at least 'margin'
    """
    
    def __init__(self, margin=1.0):
        super().__init__()
        self.margin = margin
    
    def forward(self, anchor, positive, negative):
        dist_pos = F.pairwise_distance(anchor, positive)  # Should be small
        dist_neg = F.pairwise_distance(anchor, negative)  # Should be large
        loss = F.relu(dist_pos - dist_neg + self.margin)   # Hinge loss
        return loss.mean()

# Example
embedding_dim = 128
anchor = torch.randn(32, embedding_dim)
positive = anchor + 0.1 * torch.randn(32, embedding_dim)  # Similar
negative = torch.randn(32, embedding_dim)                  # Different

criterion = TripletLoss(margin=1.0)
loss = criterion(anchor, positive, negative)
print(f"Triplet loss: {loss.item():.4f}")
```

### 4.4 Dice Loss (Image Segmentation)

```python
class DiceLoss(nn.Module):
    """
    Dice Loss for image segmentation.
    Measures overlap between predicted and true masks.
    
    Dice = 2|A ∩ B| / (|A| + |B|)
    Dice Loss = 1 - Dice
    """
    
    def __init__(self, smooth=1.0):
        super().__init__()
        self.smooth = smooth  # Prevents division by zero
    
    def forward(self, y_pred, y_true):
        # y_pred: sigmoid output (probabilities)
        # y_true: binary mask
        y_pred = torch.sigmoid(y_pred)
        
        # Flatten
        pred_flat = y_pred.view(-1)
        true_flat = y_true.view(-1)
        
        intersection = (pred_flat * true_flat).sum()
        dice = (2. * intersection + self.smooth) / \
               (pred_flat.sum() + true_flat.sum() + self.smooth)
        
        return 1 - dice

# Often combined with BCE
class CombinedSegmentationLoss(nn.Module):
    def __init__(self, bce_weight=0.5, dice_weight=0.5):
        super().__init__()
        self.bce = nn.BCEWithLogitsLoss()
        self.dice = DiceLoss()
        self.bce_weight = bce_weight
        self.dice_weight = dice_weight
    
    def forward(self, y_pred, y_true):
        return self.bce_weight * self.bce(y_pred, y_true) + \
               self.dice_weight * self.dice(y_pred, y_true)
```

### 4.5 Perceptual Loss (Style/Content)

```python
class PerceptualLoss(nn.Module):
    """
    Compare high-level features extracted by a pre-trained network
    instead of pixel-by-pixel comparison.
    
    Used in: Style transfer, super-resolution, image generation
    """
    
    def __init__(self):
        super().__init__()
        # Use pre-trained VGG features
        from torchvision.models import vgg16
        vgg = vgg16(pretrained=True).features
        
        # Extract features at different levels
        self.blocks = nn.ModuleList([
            vgg[:4],    # relu1_2 (low-level: edges, textures)
            vgg[4:9],   # relu2_2 (mid-level: patterns)
            vgg[9:16],  # relu3_3 (high-level: objects)
        ])
        
        for param in self.parameters():
            param.requires_grad = False  # Freeze VGG weights
    
    def forward(self, pred, target):
        loss = 0
        x, y = pred, target
        for block in self.blocks:
            x = block(x)
            y = block(y)
            loss += F.mse_loss(x, y)
        return loss
```

---

## 5. Validating Custom Losses

### 5.1 Gradient Checking

```python
def check_custom_loss_gradient(loss_fn, y_pred, y_true, eps=1e-5):
    """Verify custom loss gradients via numerical differentiation."""
    y_pred.requires_grad_(True)
    
    # Analytical gradient (autograd)
    loss = loss_fn(y_pred, y_true)
    loss.backward()
    analytical_grad = y_pred.grad.clone()
    
    # Numerical gradient
    numerical_grad = torch.zeros_like(y_pred)
    for i in range(y_pred.numel()):
        y_plus = y_pred.data.clone().flatten()
        y_minus = y_pred.data.clone().flatten()
        y_plus[i] += eps
        y_minus[i] -= eps
        
        loss_plus = loss_fn(y_plus.view_as(y_pred), y_true)
        loss_minus = loss_fn(y_minus.view_as(y_pred), y_true)
        
        numerical_grad.flatten()[i] = (loss_plus - loss_minus) / (2 * eps)
    
    # Compare
    rel_error = torch.norm(analytical_grad - numerical_grad) / \
                (torch.norm(analytical_grad) + torch.norm(numerical_grad) + 1e-8)
    
    print(f"Relative error: {rel_error.item():.2e}")
    print(f"Gradient check: {'PASSED ✅' if rel_error < 1e-5 else 'FAILED ❌'}")
    return rel_error.item()

# Test
y_pred = torch.tensor([0.7, 0.2, 0.1], requires_grad=True)
y_true = torch.tensor([1.0, 0.0, 0.0])
check_custom_loss_gradient(F.mse_loss, y_pred, y_true)
```

### 5.2 Sanity Checks for Custom Losses

```python
def sanity_check_loss(loss_fn, name="Custom"):
    """Run basic sanity checks on a loss function."""
    print(f"\n{'=' * 50}")
    print(f"Sanity Checks: {name}")
    print('=' * 50)
    
    # 1. Perfect prediction should have near-zero loss
    y_true = torch.tensor([1.0, 0.0, 0.0])
    y_perfect = y_true.clone()
    loss_perfect = loss_fn(y_perfect, y_true)
    print(f"1. Perfect prediction loss: {loss_perfect.item():.6f} "
          f"{'✅' if loss_perfect.item() < 0.01 else '⚠️'}")
    
    # 2. Loss should be non-negative
    for _ in range(10):
        y_rand = torch.rand(3)
        loss = loss_fn(y_rand, y_true)
        if loss.item() < 0:
            print(f"2. Non-negative: FAILED ❌ (got {loss.item():.4f})")
            break
    else:
        print("2. Non-negative: ✅")
    
    # 3. Worse predictions should have higher loss
    y_good = torch.tensor([0.9, 0.05, 0.05])
    y_bad = torch.tensor([0.3, 0.4, 0.3])
    loss_good = loss_fn(y_good, y_true)
    loss_bad = loss_fn(y_bad, y_true)
    print(f"3. Good < Bad: {loss_good.item():.4f} < {loss_bad.item():.4f} "
          f"{'✅' if loss_good < loss_bad else '❌'}")
    
    # 4. Gradient exists
    y_test = torch.tensor([0.5, 0.3, 0.2], requires_grad=True)
    loss = loss_fn(y_test, y_true)
    loss.backward()
    has_grad = y_test.grad is not None and not torch.isnan(y_test.grad).any()
    print(f"4. Gradient exists: {'✅' if has_grad else '❌'}")

sanity_check_loss(F.mse_loss, "MSE")
```

---

## 6. Applications

| Custom Loss | Domain | Purpose |
|---|---|---|
| **Asymmetric loss** | Finance, medicine | Penalize one direction of error more |
| **Multi-task loss** | Object detection, autonomous driving | Balance multiple objectives |
| **Triplet loss** | Face recognition, retrieval | Learn embedding spaces |
| **Dice loss** | Medical image segmentation | Optimize overlap metric directly |
| **Perceptual loss** | Image generation, super-resolution | Match high-level features |
| **Contrastive loss** | Self-supervised learning (SimCLR) | Align positive pairs, push negatives |
| **CTC loss** | Speech recognition, OCR | Alignment-free sequence matching |
| **REINFORCE loss** | RL, text generation | Optimize non-differentiable rewards |

---

## 📊 Summary Table

| Concept | Key Point |
|---|---|
| **Why custom** | Standard losses don't capture domain-specific objectives |
| **Differentiability** | ALL operations must have gradients for backprop to work |
| **PyTorch approach** | Subclass `nn.Module`, implement `forward()`, autograd handles the rest |
| **Multi-task** | Combine losses with weights: L = α·L₁ + β·L₂ + ... |
| **Learnable weights** | Uncertainty weighting learns task importance automatically |
| **Validation** | Always do gradient checking and sanity checks |
| **Common pattern** | Combine standard loss + custom regularizer |
| **Non-differentiable** | Use smooth relaxations (sigmoid for step, soft for argmax) |

---

## ❓ Revision Questions

1. **Why must a custom loss function be differentiable? What happens during backpropagation if it isn't? Give an example of a non-differentiable metric and its differentiable proxy.**

2. **Design a custom loss function for a medical diagnosis model where false negatives (missing a disease) should be penalized 10x more than false positives. Write the PyTorch code.**

3. **Explain triplet loss. Draw a diagram showing anchor, positive, and negative samples in embedding space before and after training. What is the role of the margin?**

4. **What is Dice loss and why is it better than pixel-wise BCE for image segmentation when there's a class imbalance between foreground and background?**

5. **Write a multi-task loss for a self-driving car model that simultaneously predicts: (a) object class, (b) bounding box, (c) road segmentation. How would you balance the three losses?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Loss Function Selection](03-loss-function-selection.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Chain Rule Review →](../04-Backpropagation/01-chain-rule-review.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
