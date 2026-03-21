# 📐 Vectorized Gradient Computation

> **Chapter 4.4 — Efficient Backpropagation with Matrix Operations**

| Topic | Details |
|---|---|
| **Subject** | Backpropagation — Vectorized Gradient Formulas |
| **Prerequisites** | Chapter 4.3 (Backward Pass), linear algebra |
| **Key Insight** | Vectorized gradients allow processing entire mini-batches in a single backward pass, using the same matrix operations as the forward pass |

---

## 📌 Overview

Chapter 4.3 derived backpropagation for a single sample. In practice, we process mini-batches of m samples simultaneously. This chapter provides the **complete vectorized gradient formulas** that operate on matrices rather than vectors, along with a full NumPy implementation. These are the exact equations used inside PyTorch and TensorFlow.

---

## 1. Vectorized Backpropagation Equations

### 1.1 The Four Key Equations

For layer l, processing a mini-batch of m samples:

```
    ╔═══════════════════════════════════════════════════════════════╗
    ║           THE FOUR EQUATIONS OF BACKPROPAGATION               ║
    ╠═══════════════════════════════════════════════════════════════╣
    ║                                                               ║
    ║  (1) dZ^[l] = dA^[l] ⊙ g'^[l](Z^[l])                       ║
    ║       ↑ Element-wise product of upstream gradient             ║
    ║         with activation derivative                            ║
    ║                                                               ║
    ║  (2) dW^[l] = (1/m) · dZ^[l] · (A^[l-1])ᵀ                  ║
    ║       ↑ Average gradient over the mini-batch                  ║
    ║                                                               ║
    ║  (3) db^[l] = (1/m) · Σⱼ dZ^[l]  (sum over columns)         ║
    ║       ↑ Average bias gradient over the mini-batch             ║
    ║                                                               ║
    ║  (4) dA^[l-1] = (W^[l])ᵀ · dZ^[l]                           ║
    ║       ↑ Propagate gradient to previous layer                  ║
    ║                                                               ║
    ╚═══════════════════════════════════════════════════════════════╝
```

### 1.2 Shape Verification

```
    For layer l with n^[l] neurons, n^[l-1] neurons in previous layer,
    and mini-batch size m:

    Eq (1): dZ^[l]    = dA^[l]     ⊙  g'(Z^[l])
            (n^[l],m) = (n^[l],m)  ⊙  (n^[l],m)     ✓

    Eq (2): dW^[l]     = (1/m) dZ^[l]   · (A^[l-1])ᵀ
            (n^[l],n^[l-1]) = (n^[l],m)  · (m,n^[l-1])   ✓

    Eq (3): db^[l]     = (1/m) sum(dZ^[l], axis=1, keepdims=True)
            (n^[l],1)  = sum over m columns                        ✓

    Eq (4): dA^[l-1]   = (W^[l])ᵀ     · dZ^[l]
            (n^[l-1],m) = (n^[l-1],n^[l]) · (n^[l],m)             ✓
```

### 1.3 Diagram of Information Flow

```
    FORWARD PASS (→)

    A^[l-1] ────────► Z^[l] = W^[l]A^[l-1]+b^[l] ────────► A^[l] = g(Z^[l])
    (n^[l-1],m)         (n^[l],m)                            (n^[l],m)


    BACKWARD PASS (←)

    dA^[l-1] ◄──── dZ^[l] ◄──── dA^[l]
    
    dA^[l-1] = (W^[l])ᵀ · dZ^[l]     │     dZ^[l] = dA^[l] ⊙ g'(Z^[l])
                                       │
                                       ▼
                              dW^[l] = (1/m) dZ^[l] · (A^[l-1])ᵀ
                              db^[l] = (1/m) Σ dZ^[l]
```

---

## 2. Activation Derivatives

### 2.1 Common Activation Gradients

```
    For equation (1): dZ^[l] = dA^[l] ⊙ g'(Z^[l])

    The g'(Z^[l]) term depends on the activation function:

    ┌────────────────┬────────────────────────────────────────┐
    │ Activation     │ g'(Z^[l]) — element-wise               │
    ├────────────────┼────────────────────────────────────────┤
    │ ReLU           │ (Z^[l] > 0).astype(float)              │
    │                │ = 1 where Z > 0, 0 where Z ≤ 0         │
    ├────────────────┼────────────────────────────────────────┤
    │ Sigmoid        │ A^[l] ⊙ (1 - A^[l])                   │
    │                │ = σ(Z) · (1 - σ(Z))                    │
    ├────────────────┼────────────────────────────────────────┤
    │ Tanh           │ 1 - (A^[l])²                           │
    │                │ = 1 - tanh²(Z)                         │
    ├────────────────┼────────────────────────────────────────┤
    │ Leaky ReLU     │ np.where(Z > 0, 1, α)                 │
    │ (slope α)      │ = 1 where Z > 0, α where Z ≤ 0        │
    ├────────────────┼────────────────────────────────────────┤
    │ Softmax + CE   │ dZ^[L] = A^[L] - Y                    │
    │ (combined)     │ Simplified! No explicit g'(Z) needed   │
    └────────────────┴────────────────────────────────────────┘
```

### 2.2 Special Case: Softmax + Cross-Entropy Output Layer

```
    For the output layer with softmax activation and cross-entropy loss:

    dZ^[L] = A^[L] - Y

    Where:
        A^[L] = ŷ = softmax output,  shape (K, m)
        Y = one-hot true labels,      shape (K, m)

    This elegant simplification means:
    • No need to compute softmax derivative separately
    • No need to compute CE gradient separately
    • Just: (prediction - truth) for each class

    Example (single sample, 3 classes):
    A^[L] = [0.7, 0.2, 0.1]    (predicted)
    Y     = [1.0, 0.0, 0.0]    (true: class 0)
    dZ^[L] = [-0.3, 0.2, 0.1]  (errors → gradient)
```

---

## 3. Complete NumPy Implementation

```python
import numpy as np

class NeuralNetwork:
    """
    Complete neural network with vectorized forward and backward passes.
    Supports arbitrary depth, ReLU hidden layers, and softmax output.
    """
    
    def __init__(self, layer_sizes, learning_rate=0.01):
        self.layer_sizes = layer_sizes
        self.L = len(layer_sizes) - 1
        self.lr = learning_rate
        self.params = {}
        self.cache = {}
        self.grads = {}
        
        # He initialization
        np.random.seed(42)
        for l in range(1, self.L + 1):
            self.params[f'W{l}'] = np.random.randn(
                layer_sizes[l], layer_sizes[l-1]
            ) * np.sqrt(2.0 / layer_sizes[l-1])
            self.params[f'b{l}'] = np.zeros((layer_sizes[l], 1))
    
    def relu(self, Z):
        return np.maximum(0, Z)
    
    def relu_derivative(self, Z):
        return (Z > 0).astype(float)
    
    def softmax(self, Z):
        Z_stable = Z - np.max(Z, axis=0, keepdims=True)
        exp_Z = np.exp(Z_stable)
        return exp_Z / np.sum(exp_Z, axis=0, keepdims=True)
    
    def forward(self, X):
        """
        Vectorized forward pass.
        X: shape (n_features, m_samples)
        """
        self.cache['A0'] = X
        A = X
        
        for l in range(1, self.L + 1):
            W = self.params[f'W{l}']
            b = self.params[f'b{l}']
            
            # (Eq Forward): Z = WA + b
            Z = W @ A + b
            self.cache[f'Z{l}'] = Z
            
            if l < self.L:
                A = self.relu(Z)          # Hidden: ReLU
            else:
                A = self.softmax(Z)       # Output: Softmax
            
            self.cache[f'A{l}'] = A
        
        return A
    
    def compute_loss(self, Y_hat, Y):
        """Cross-entropy loss (averaged over mini-batch)."""
        m = Y.shape[1]
        # Clip for numerical stability
        Y_hat_clipped = np.clip(Y_hat, 1e-8, 1 - 1e-8)
        loss = -(1/m) * np.sum(Y * np.log(Y_hat_clipped))
        return loss
    
    def backward(self, Y):
        """
        Vectorized backward pass — The Four Equations.
        Y: one-hot encoded labels, shape (n_classes, m)
        """
        m = Y.shape[1]
        
        # ──── Output layer (Softmax + CE combined) ────
        # dZ^[L] = A^[L] - Y  (special case!)
        dZ = self.cache[f'A{self.L}'] - Y
        
        for l in range(self.L, 0, -1):
            A_prev = self.cache[f'A{l-1}']
            
            # Eq (2): dW = (1/m) dZ · A_prev^T
            self.grads[f'dW{l}'] = (1/m) * dZ @ A_prev.T
            
            # Eq (3): db = (1/m) sum(dZ, axis=1)
            self.grads[f'db{l}'] = (1/m) * np.sum(dZ, axis=1, keepdims=True)
            
            if l > 1:
                # Eq (4): dA_prev = W^T · dZ
                dA_prev = self.params[f'W{l}'].T @ dZ
                
                # Eq (1): dZ_prev = dA_prev ⊙ g'(Z_prev)
                dZ = dA_prev * self.relu_derivative(self.cache[f'Z{l-1}'])
    
    def update_parameters(self):
        """Gradient descent update."""
        for l in range(1, self.L + 1):
            self.params[f'W{l}'] -= self.lr * self.grads[f'dW{l}']
            self.params[f'b{l}'] -= self.lr * self.grads[f'db{l}']
    
    def train_step(self, X, Y):
        """One complete training step: forward → loss → backward → update."""
        Y_hat = self.forward(X)
        loss = self.compute_loss(Y_hat, Y)
        self.backward(Y)
        self.update_parameters()
        return loss
    
    def predict(self, X):
        Y_hat = self.forward(X)
        return np.argmax(Y_hat, axis=0)


# ──── Training Example ────
from sklearn.datasets import make_moons

# Generate 2D classification data
X_train, y_train = make_moons(n_samples=500, noise=0.2, random_state=42)
X_train = X_train.T  # Shape: (2, 500)

# One-hot encode labels
Y_train = np.zeros((2, 500))
Y_train[0, y_train == 0] = 1
Y_train[1, y_train == 1] = 1

# Create and train network
nn = NeuralNetwork(layer_sizes=[2, 16, 16, 2], learning_rate=0.1)

print("Training Neural Network (from scratch with NumPy)")
print("=" * 50)

for epoch in range(1001):
    loss = nn.train_step(X_train, Y_train)
    
    if epoch % 200 == 0:
        predictions = nn.predict(X_train)
        accuracy = np.mean(predictions == y_train)
        print(f"  Epoch {epoch:4d} │ Loss: {loss:.4f} │ Accuracy: {accuracy:.2%}")

# Shape verification
print(f"\nShape Verification:")
for l in range(1, nn.L + 1):
    print(f"  Layer {l}: W{l} {nn.params[f'W{l}'].shape}, "
          f"b{l} {nn.params[f'b{l}'].shape}, "
          f"dW{l} {nn.grads[f'dW{l}'].shape}, "
          f"db{l} {nn.grads[f'db{l}'].shape}")
```

**Expected Output:**
```
Training Neural Network (from scratch with NumPy)
==================================================
  Epoch    0 │ Loss: 0.6931 │ Accuracy: 50.00%
  Epoch  200 │ Loss: 0.1542 │ Accuracy: 95.40%
  Epoch  400 │ Loss: 0.0823 │ Accuracy: 97.80%
  Epoch  600 │ Loss: 0.0571 │ Accuracy: 98.40%
  Epoch  800 │ Loss: 0.0434 │ Accuracy: 99.00%
  Epoch 1000 │ Loss: 0.0349 │ Accuracy: 99.40%
```

---

## 4. Gradient Dimension Quick Reference

```
    Architecture: n⁰ → n¹ → n² → n³,  batch size m

    ┌────────────────────────────────────────────────────────────┐
    │  Forward cache:                                            │
    │  A⁰: (n⁰, m)  Z¹: (n¹, m)  A¹: (n¹, m)                 │
    │                 Z²: (n², m)  A²: (n², m)                  │
    │                 Z³: (n³, m)  A³: (n³, m) = Ŷ             │
    │                                                            │
    │  Gradients (same shapes as parameters):                    │
    │  dW¹: (n¹, n⁰)   db¹: (n¹, 1)                           │
    │  dW²: (n², n¹)    db²: (n², 1)                           │
    │  dW³: (n³, n²)    db³: (n³, 1)                           │
    │                                                            │
    │  Intermediate gradients:                                   │
    │  dZ³: (n³, m)     dA²: (n², m)                            │
    │  dZ²: (n², m)     dA¹: (n¹, m)                            │
    │  dZ¹: (n¹, m)                                             │
    └────────────────────────────────────────────────────────────┘
    
    KEY RULE: dW^[l] has the SAME shape as W^[l]
              db^[l] has the SAME shape as b^[l]
              (gradients match parameter shapes!)
```

---

## 5. Computational Complexity

```
    Forward pass complexity per layer:  O(n^[l] × n^[l-1] × m)
    Backward pass complexity per layer: O(n^[l] × n^[l-1] × m)  (same!)

    Total backprop ≈ 2× forward pass computation

    Memory for activations: O(Σ n^[l] × m)  (must cache for backward)

    ┌────────────────────────────────────────────────────┐
    │  IMPORTANT: Backprop is NOT expensive!              │
    │  It costs only ~2× the forward pass.               │
    │  The real cost is repeated forward+backward         │
    │  over many epochs with large datasets.              │
    └────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Equation | Formula | Purpose | Shape |
|---|---|---|---|
| **(1) Activation gradient** | dZ^[l] = dA^[l] ⊙ g'(Z^[l]) | Through activation | (n^[l], m) |
| **(2) Weight gradient** | dW^[l] = (1/m) dZ^[l] · (A^[l-1])ᵀ | Update weights | (n^[l], n^[l-1]) |
| **(3) Bias gradient** | db^[l] = (1/m) Σⱼ dZ^[l] | Update biases | (n^[l], 1) |
| **(4) Propagate back** | dA^[l-1] = (W^[l])ᵀ · dZ^[l] | Send error to prev layer | (n^[l-1], m) |
| **Softmax+CE (special)** | dZ^[L] = A^[L] - Y | Output layer shortcut | (K, m) |

---

## ❓ Revision Questions

1. **Write out the four vectorized backpropagation equations. For each, explain what it computes and verify the matrix dimensions for a network with architecture 784→128→64→10 and batch size 32.**

2. **Why do we divide by m (batch size) in equations (2) and (3) but not in equations (1) and (4)?**

3. **Derive why dZ^[L] = A^[L] - Y for softmax output with cross-entropy loss. Show the simplification step by step.**

4. **Implement the backward pass for a network with Leaky ReLU (α=0.01) hidden activations instead of ReLU. What changes in the equations?**

5. **The forward pass has complexity O(n²m) per layer. Show that the backward pass has the same complexity. Why does this matter for training efficiency?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Backward Pass Step-by-Step](03-backward-pass-step-by-step.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Vanishing & Exploding Gradients →](05-vanishing-exploding-gradients.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
