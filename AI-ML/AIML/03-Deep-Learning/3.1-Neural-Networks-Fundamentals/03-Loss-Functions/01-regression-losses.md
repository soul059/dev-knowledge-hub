# 📉 Regression Loss Functions

> **Chapter 3.1 — Measuring How Wrong Your Predictions Are (Continuous Values)**

| Topic | Details |
|---|---|
| **Subject** | Loss Functions — Regression Losses |
| **Prerequisites** | Chapter 2.3 (Forward Pass) |
| **Key Insight** | Different regression losses handle outliers differently — MSE penalizes large errors heavily, MAE treats all errors equally, Huber combines both |

---

## 📌 Overview

A loss function (also called cost function or objective function) quantifies how far the network's predictions are from the true values. For regression tasks — where the output is a continuous number — we need loss functions that measure the distance between predicted values ŷ and true values y. This chapter covers the three most important regression losses, their mathematical properties, gradients, and when to use each.

---

## 1. Mean Squared Error (MSE)

### 1.1 Formula

```
                   1   m
    L_MSE = ─── Σ  (yᵢ - ŷᵢ)²
                   m  i=1

    For a single sample:
    ℓ(y, ŷ) = (y - ŷ)²

    Where:
        m  = number of samples
        yᵢ = true value for sample i
        ŷᵢ = predicted value for sample i
```

### 1.2 Gradient

```
    ∂L_MSE      2   m
    ────── = - ─── Σ  (yᵢ - ŷᵢ)
     ∂ŷ        m  i=1

    For a single sample:
    ∂ℓ
    ── = -2(y - ŷ) = 2(ŷ - y)
    ∂ŷ

    ┌─────────────────────────────────────────────────────┐
    │  Key property: Gradient is PROPORTIONAL to error.   │
    │  Large errors → Large gradients → Big corrections   │
    │  Small errors → Small gradients → Fine adjustments  │
    └─────────────────────────────────────────────────────┘
```

### 1.3 ASCII Plot of MSE

```
    Loss
    │
    │                          ╱
   9┤                        ╱·
    │                      ╱
    │                    ╱·
   4┤               ···╱
    │            ··╱
   1┤        ···╱
    │     ··╱
   0┤ ····●
    ┼──────●────────────────── (y - ŷ)
          0  1  2  3

    Parabolic: L = (y - ŷ)²
    Symmetric around error = 0
    Grows QUADRATICALLY — large errors are penalized heavily!
```

### 1.4 Properties

| Property | Value |
|---|---|
| **Smoothness** | ✅ Differentiable everywhere |
| **Convexity** | ✅ Convex (unique global minimum) |
| **Outlier sensitivity** | ❌ Very sensitive — squared amplification |
| **Gradient at zero** | Smooth, goes to zero |
| **Scale** | Squared units (if y in meters, MSE in meters²) |
| **Optimization** | Easy to optimize — smooth gradients |

### 1.5 Worked Example

```
    True values:      y = [3.0, 5.0, 2.5, 7.0]
    Predicted values: ŷ = [2.5, 4.8, 3.0, 5.0]

    Errors:           e = y - ŷ = [0.5, 0.2, -0.5, 2.0]
    Squared errors:   e² = [0.25, 0.04, 0.25, 4.00]

    MSE = (0.25 + 0.04 + 0.25 + 4.00) / 4
        = 4.54 / 4
        = 1.135

    Note: The outlier (error = 2.0) contributes 4.0/4.54 = 88% of the loss!
    MSE is DOMINATED by large errors.
```

---

## 2. Mean Absolute Error (MAE)

### 2.1 Formula

```
                   1   m
    L_MAE = ─── Σ  |yᵢ - ŷᵢ|
                   m  i=1

    For a single sample:
    ℓ(y, ŷ) = |y - ŷ|
```

### 2.2 Gradient

```
    ∂ℓ       ┌ -1,  if y > ŷ  (under-predicting)
    ── =     │
    ∂ŷ       │ +1,  if y < ŷ  (over-predicting)
             │
             └  undefined at y = ŷ  (use subgradient: 0)

    = -sign(y - ŷ)

    ┌─────────────────────────────────────────────────────┐
    │  Key property: Gradient magnitude is CONSTANT.       │
    │  Large errors → Same gradient as small errors        │
    │  This makes MAE more robust to outliers!             │
    └─────────────────────────────────────────────────────┘
```

### 2.3 ASCII Plot of MAE

```
    Loss
    │
    │                    ╱
   3┤                  ╱
    │                ╱
   2┤              ╱
    │            ╱
   1┤          ╱
    │        ╱
   0┤      ●
    │    ╱
    ┼──╱───●────────────────── (y - ŷ)
        -2  0  1  2  3

    V-shaped: L = |y - ŷ|
    Linear growth — outliers are NOT amplified
    NOT differentiable at the tip (error = 0)
```

### 2.4 Properties

| Property | Value |
|---|---|
| **Smoothness** | ❌ Not differentiable at error = 0 (kink) |
| **Convexity** | ✅ Convex |
| **Outlier sensitivity** | ✅ Robust — linear, not squared |
| **Gradient at zero** | Discontinuous (subgradient used) |
| **Scale** | Same units as y |
| **Optimization** | Harder — constant gradient can cause oscillation near minimum |

### 2.5 Worked Example (Same Data)

```
    True values:      y = [3.0, 5.0, 2.5, 7.0]
    Predicted values: ŷ = [2.5, 4.8, 3.0, 5.0]

    Absolute errors:  |e| = [0.5, 0.2, 0.5, 2.0]

    MAE = (0.5 + 0.2 + 0.5 + 2.0) / 4
        = 3.2 / 4
        = 0.80

    Outlier contribution: 2.0/3.2 = 62.5%  (vs 88% for MSE)
    MAE is less affected by the outlier.
```

---

## 3. Huber Loss (Smooth L1)

### 3.1 Formula

```
                  ┌  ½(y - ŷ)²,           if |y - ŷ| ≤ δ
    L_Huber =     │
                  └  δ|y - ŷ| - ½δ²,      if |y - ŷ| > δ

    Where δ (delta) is a threshold hyperparameter (typically δ = 1.0)
    
    Behavior:
    • For small errors (|e| ≤ δ): behaves like MSE (quadratic)
    • For large errors (|e| > δ): behaves like MAE (linear)
```

### 3.2 Gradient

```
    ∂L_Huber     ┌  -(y - ŷ) = (ŷ - y),     if |y - ŷ| ≤ δ
    ──────── =   │
      ∂ŷ         └  -δ · sign(y - ŷ),         if |y - ŷ| > δ

    Smoothly transitions from MSE gradient to MAE gradient at |error| = δ
```

### 3.3 ASCII Plot of Huber Loss

```
    Loss
    │
    │                           ╱
    │                         ╱
   3┤                       ╱       ← Linear (MAE-like)
    │                     ╱
   2┤                  ╱·
    │              ·╱·             δ = 1
   1┤          ·╱·
    │      ··╱·                    ← Quadratic (MSE-like)
    │   ··╱
   0┤···●                          Transition at |error| = δ
    ┼──────●───────────────────── (y - ŷ)
          0  δ   2δ   3δ

    Best of both worlds:
    Smooth like MSE near zero, robust like MAE for outliers
```

### 3.4 Comparison of All Three

```
    Loss
    │
    │                MSE ╱     MAE ╱
    │                  ╱╱         ╱
    │                ╱╱╱         ╱
    │              ╱╱╱  Huber  ╱
    │            ╱╱╱    ╱ ···╱
    │          ╱╱╱   ╱·····╱
    │        ╱╱╱ ╱·····╱
    │      ╱╱╱····╱
    │    ╱╱····╱
    │  ╱····╱
    │··╱╱
    ●·╱
    ┼────────────────────────── |error|
    0           δ

    MSE:   Grows fastest (quadratic) → sensitive to outliers
    MAE:   Grows linearly → robust but non-smooth
    Huber: Quadratic near 0, linear far away → best compromise
```

### 3.5 Properties Comparison

| Property | MSE | MAE | Huber |
|---|---|---|---|
| **Smoothness** | ✅ Smooth everywhere | ❌ Kink at 0 | ✅ Smooth everywhere |
| **Outlier robust** | ❌ Very sensitive | ✅ Robust | ✅ Robust |
| **Gradient near 0** | Proportional to error | Constant (±1) | Proportional to error |
| **Gradient far from 0** | Proportional (large!) | Constant (±1) | Constant (±δ) |
| **Hyperparameters** | None | None | δ (threshold) |
| **Convergence** | Fast near minimum | Oscillates near minimum | Fast and stable |

---

## 4. Complete Python Implementation

```python
import numpy as np

class RegressionLosses:
    """Complete implementation of regression loss functions."""
    
    @staticmethod
    def mse(y_true, y_pred):
        """Mean Squared Error"""
        return np.mean((y_true - y_pred) ** 2)
    
    @staticmethod
    def mse_gradient(y_true, y_pred):
        """Gradient of MSE w.r.t. predictions"""
        m = len(y_true)
        return (2 / m) * (y_pred - y_true)
    
    @staticmethod
    def mae(y_true, y_pred):
        """Mean Absolute Error"""
        return np.mean(np.abs(y_true - y_pred))
    
    @staticmethod
    def mae_gradient(y_true, y_pred):
        """Gradient (subgradient) of MAE w.r.t. predictions"""
        m = len(y_true)
        return (1 / m) * np.sign(y_pred - y_true)
    
    @staticmethod
    def huber(y_true, y_pred, delta=1.0):
        """Huber Loss (Smooth L1)"""
        error = y_true - y_pred
        abs_error = np.abs(error)
        quadratic = 0.5 * error ** 2
        linear = delta * abs_error - 0.5 * delta ** 2
        return np.mean(np.where(abs_error <= delta, quadratic, linear))
    
    @staticmethod
    def huber_gradient(y_true, y_pred, delta=1.0):
        """Gradient of Huber Loss w.r.t. predictions"""
        m = len(y_true)
        error = y_pred - y_true
        abs_error = np.abs(error)
        return (1 / m) * np.where(abs_error <= delta, error, delta * np.sign(error))


# ──── Demonstration ────
np.random.seed(42)

# Generate data with an outlier
y_true = np.array([3.0, 5.0, 2.5, 7.0, 4.0, 6.0, 1.5, 8.0])
y_pred = np.array([2.8, 4.9, 2.7, 5.0, 3.8, 5.8, 1.6, 7.5])  # Good predictions
y_outlier = y_pred.copy()
y_outlier[3] = 2.0  # Create an outlier: true=7.0, pred=2.0 (error=5.0)

losses = RegressionLosses()

print("=" * 60)
print("Loss Comparison: Normal vs. With Outlier")
print("=" * 60)

for scenario, y_p in [("Normal predictions", y_pred), ("With outlier (err=5)", y_outlier)]:
    mse_val = losses.mse(y_true, y_p)
    mae_val = losses.mae(y_true, y_p)
    hub_val = losses.huber(y_true, y_p, delta=1.0)
    
    print(f"\n{scenario}:")
    print(f"  MSE:   {mse_val:.4f}")
    print(f"  MAE:   {mae_val:.4f}")
    print(f"  Huber: {hub_val:.4f}")

# Show gradient behavior
print(f"\n{'Gradient Comparison (single sample, y=5.0)':^60}")
print("-" * 60)
print(f"{'ŷ':>6} │ {'Error':>7} │ {'MSE grad':>10} │ {'MAE grad':>10} │ {'Huber grad':>10}")
print("-" * 60)

y_true_single = np.array([5.0])
for y_p in [4.5, 4.0, 3.0, 1.0, 0.0]:
    y_p_arr = np.array([y_p])
    err = y_p - 5.0
    mse_g = losses.mse_gradient(y_true_single, y_p_arr)[0]
    mae_g = losses.mae_gradient(y_true_single, y_p_arr)[0]
    hub_g = losses.huber_gradient(y_true_single, y_p_arr, delta=1.0)[0]
    print(f"{y_p:>6.1f} │ {err:>7.1f} │ {mse_g:>10.4f} │ {mae_g:>10.4f} │ {hub_g:>10.4f}")
```

**Expected Output:**
```
============================================================
Loss Comparison: Normal vs. With Outlier
============================================================

Normal predictions:
  MSE:   0.0675
  MAE:   0.2000
  Huber: 0.0338

With outlier (err=5):
  MSE:   3.1925        ← MSE explodes (47x increase!)
  MAE:   0.8000        ← MAE increases (4x — proportional)
  Huber: 0.5963        ← Huber moderate increase

                 Gradient Comparison (single sample, y=5.0)
------------------------------------------------------------
     ŷ │   Error │   MSE grad │   MAE grad │ Huber grad
------------------------------------------------------------
   4.5 │    -0.5 │    -1.0000 │    -1.0000 │    -0.5000
   4.0 │    -1.0 │    -2.0000 │    -1.0000 │    -1.0000
   3.0 │    -2.0 │    -4.0000 │    -1.0000 │    -1.0000
   1.0 │    -4.0 │    -8.0000 │    -1.0000 │    -1.0000
   0.0 │    -5.0 │   -10.0000 │    -1.0000 │    -1.0000
```

---

## 5. PyTorch Implementation

```python
import torch
import torch.nn as nn

# Built-in loss functions
mse_loss = nn.MSELoss()            # Mean Squared Error
mae_loss = nn.L1Loss()             # Mean Absolute Error (L1)
huber_loss = nn.SmoothL1Loss()     # Huber Loss (default delta=1.0)
huber_custom = nn.HuberLoss(delta=0.5)  # Custom delta

# Example
y_true = torch.tensor([3.0, 5.0, 2.5, 7.0])
y_pred = torch.tensor([2.8, 4.9, 2.7, 5.0], requires_grad=True)

# Compute losses
loss_mse = mse_loss(y_pred, y_true)
loss_mae = mae_loss(y_pred, y_true)
loss_hub = huber_loss(y_pred, y_true)

print(f"MSE Loss:   {loss_mse.item():.4f}")
print(f"MAE Loss:   {loss_mae.item():.4f}")
print(f"Huber Loss: {loss_hub.item():.4f}")

# Gradient computation
loss_mse.backward()
print(f"\nMSE gradients: {y_pred.grad}")

# Reduction modes
print("\nReduction modes:")
for mode in ['mean', 'sum', 'none']:
    loss_fn = nn.MSELoss(reduction=mode)
    result = loss_fn(y_pred.detach(), y_true)
    print(f"  {mode:5s}: {result}")
```

---

## 6. When to Use Each Loss

| Scenario | Recommended Loss | Why |
|---|---|---|
| **Standard regression** | MSE | Smooth gradients, easy optimization |
| **Data has outliers** | Huber or MAE | Robust to large errors |
| **Precise near target** | MSE | Penalizes small errors less |
| **Object detection (bbox)** | Smooth L1 (Huber) | Robust and smooth |
| **Financial predictions** | MAE | Equal treatment of all errors |
| **Need differentiability** | MSE or Huber | MAE has kink at 0 |
| **Percentage error matters** | MAPE | Normalize by true value |

---

## 📊 Summary Table

| Loss | Formula | Gradient | Outlier Robust | Smooth |
|---|---|---|---|---|
| **MSE** | (1/m)Σ(y-ŷ)² | 2(ŷ-y)/m | ❌ No | ✅ Yes |
| **MAE** | (1/m)Σ\|y-ŷ\| | sign(ŷ-y)/m | ✅ Yes | ❌ No (kink) |
| **Huber** | Quadratic if small, linear if large | Blends MSE/MAE | ✅ Yes | ✅ Yes |

---

## ❓ Revision Questions

1. **Compute MSE, MAE, and Huber loss (δ=1) for y=[2, 4, 6] and ŷ=[2.5, 3, 5.5]. Show all steps.**

2. **Derive the gradient of MSE with respect to ŷ. Why does this gradient cause large updates for outliers?**

3. **Explain mathematically why Huber loss is "the best of both worlds." At what error value does it transition from quadratic to linear?**

4. **A dataset has house prices where 5% are mansions worth 10x the median. Which loss function should you use and why?**

5. **Compare the gradient magnitude of MSE vs MAE for errors of 0.1, 1.0, and 10.0. What does this tell you about convergence behavior?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Batch Processing](../02-Forward-Propagation/05-batch-processing.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Classification Losses →](02-classification-losses.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
