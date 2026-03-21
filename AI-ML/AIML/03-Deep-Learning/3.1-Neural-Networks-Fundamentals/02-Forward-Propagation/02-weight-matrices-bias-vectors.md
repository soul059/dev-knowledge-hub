# ⚖️ Weight Matrices & Bias Vectors

> **Chapter 2.2 — The Learnable Parameters of Neural Networks**

| Topic | Details |
|---|---|
| **Subject** | Forward Propagation — Weights and Biases |
| **Prerequisites** | Linear algebra (matrix multiplication), Chapter 2.1 |
| **Key Insight** | Weights determine how information flows between layers; biases shift the activation threshold — together they are ALL the learnable parameters in a dense network |

---

## 📌 Overview

Every neural network learns by adjusting two types of parameters: **weights** and **biases**. Weights control the strength of connections between neurons in adjacent layers, while biases allow neurons to activate even when inputs are zero. Understanding their dimensions, roles, and how to count them is fundamental to designing and debugging neural networks.

---

## 1. Weight Matrices

### 1.1 Definition and Dimensions

For layer l connecting n^[l-1] neurons to n^[l] neurons:

```
    W^[l] has shape: (n^[l], n^[l-1])
    
    ┌───────────────────────────────────────┐
    │  W^[l] ∈ ℝ^(n^[l] × n^[l-1])        │
    │                                       │
    │  Rows    = neurons in current layer l  │
    │  Columns = neurons in previous layer   │
    │                                       │
    │  W^[l]ᵢⱼ = weight from neuron j in    │
    │            layer (l-1) to neuron i     │
    │            in layer l                  │
    └───────────────────────────────────────┘
```

### 1.2 Dimensional Diagram

```
    Layer (l-1)              W^[l]                Layer l
    n^[l-1] = 3              (4 × 3)              n^[l] = 4

    ┌───┐                ┌──────────────┐         ┌───┐
    │ a₁│────────────────│w₁₁ w₁₂ w₁₃ │────────►│ z₁│
    ├───┤                │w₂₁ w₂₂ w₂₃ │         ├───┤
    │ a₂│────────────────│w₃₁ w₃₂ w₃₃ │────────►│ z₂│
    ├───┤                │w₄₁ w₄₂ w₄₃ │         ├───┤
    │ a₃│────────────────│              │────────►│ z₃│
    └───┘                └──────────────┘         ├───┤
                                                  │ z₄│
                                                  └───┘
    
    W^[l] · a^[l-1] = z^[l]  (before adding bias)
    
    ┌──────────────┐   ┌───┐     ┌───┐
    │w₁₁ w₁₂ w₁₃ │   │ a₁│     │ z₁│
    │w₂₁ w₂₂ w₂₃ │ · │ a₂│  =  │ z₂│
    │w₃₁ w₃₂ w₃₃ │   │ a₃│     │ z₃│
    │w₄₁ w₄₂ w₄₃ │   └───┘     │ z₄│
    └──────────────┘             └───┘
      (4 × 3)        (3 × 1)    (4 × 1)
```

### 1.3 What Weights Mean

Each weight w^[l]ᵢⱼ represents:

| Aspect | Interpretation |
|---|---|
| **Magnitude** | Strength of the connection — large |w| means strong influence |
| **Sign (positive)** | Excitatory connection — input increases output |
| **Sign (negative)** | Inhibitory connection — input decreases output |
| **Near zero** | Weak/irrelevant connection — input barely affects output |
| **Row i** | All weights feeding INTO neuron i of layer l |
| **Column j** | All weights FROM neuron j of layer (l-1) |

```
    Example: W^[1] row 3 = [0.8, -0.3, 0.1]
    
    Interpretation: Neuron 3 in layer 1...
    • Is strongly excited by a₁ (weight 0.8)
    • Is moderately inhibited by a₂ (weight -0.3)
    • Is weakly excited by a₃ (weight 0.1)
```

---

## 2. Bias Vectors

### 2.1 Definition and Dimensions

For layer l with n^[l] neurons:

```
    b^[l] has shape: (n^[l], 1)
    
    ┌──────────────────────────────────────┐
    │  b^[l] ∈ ℝ^(n^[l] × 1)             │
    │                                      │
    │  One bias per neuron in layer l       │
    │  Added AFTER the weighted sum         │
    │                                      │
    │  z^[l] = W^[l] · a^[l-1] + b^[l]    │
    └──────────────────────────────────────┘
```

### 2.2 The Role of Bias

```
    WITHOUT BIAS:                         WITH BIAS:
    z = w·x                               z = w·x + b
    
    Decision boundary must                Decision boundary can be
    pass through the origin:              shifted anywhere:
    
    y                                     y
    │    ╱                                │         ╱
    │   ╱                                 │        ╱
    │  ╱                                  │       ╱
    │ ╱                                   │      ╱
    ●─────── x                            │─────●──── x
    ╱                                     │   ╱
    Must pass through (0,0)               Shifted by b
```

**Bias shifts the activation function horizontally:**

```
    Without bias: σ(wx)              With bias: σ(wx + b)
    
    Activation                       Activation
    │       ╱ ─                      │          ╱ ─
    │     ╱                          │        ╱
    │   ╱                            │      ╱
    │ ╱                              │    ╱
    ●───────── z=wx                  │──●──────── z=wx+b
    │                                │
    Always activates at x=0          Activates at x = -b/w
                                     (shifted by bias!)
```

### 2.3 Why Bias is Necessary

Without bias, a neuron with all-zero inputs produces zero output regardless of weights:

```
    z = w₁(0) + w₂(0) + w₃(0) = 0      (always zero without bias)
    z = w₁(0) + w₂(0) + w₃(0) + b = b  (non-zero with bias!)
```

**The bias allows the neuron to have a non-zero default output**, which is critical for proper representation learning.

---

## 3. Complete Notation Convention

### 3.1 Standard Notation

```
    ┌──────────────────────────────────────────────────────────┐
    │              NEURAL NETWORK NOTATION                      │
    ├──────────────────────────────────────────────────────────┤
    │                                                          │
    │  Layer index:      l = 0, 1, 2, ..., L                   │
    │  Layer 0:          Input layer (no parameters)           │
    │  Layer L:          Output layer                          │
    │                                                          │
    │  n^[l]:            Number of neurons in layer l          │
    │  W^[l]:            Weight matrix, shape (n^[l], n^[l-1]) │
    │  b^[l]:            Bias vector,  shape (n^[l], 1)        │
    │  z^[l]:            Pre-activation, shape (n^[l], 1)      │
    │  a^[l]:            Activation, shape (n^[l], 1)          │
    │  g^[l]:            Activation function for layer l       │
    │                                                          │
    │  a^[0] = x         Input vector                          │
    │  a^[L] = ŷ         Output (prediction)                   │
    │                                                          │
    │  Forward pass:                                           │
    │  z^[l] = W^[l] · a^[l-1] + b^[l]                       │
    │  a^[l] = g^[l](z^[l])                                   │
    │                                                          │
    └──────────────────────────────────────────────────────────┘
```

### 3.2 Superscript vs Subscript

```
    W^[l]     — layer l (superscript in brackets)
    wᵢⱼ^[l]  — row i, column j of weight matrix in layer l
    aᵢ^[l]   — activation of neuron i in layer l
    
    Do NOT confuse:
    W^[2]     — weights of layer 2
    W²        — W squared (matrix power)
    w₂        — second element of a weight vector
```

---

## 4. Parameter Counting

### 4.1 Formula

```
    Parameters in layer l:
        Weights:  n^[l] × n^[l-1]
        Biases:   n^[l]
        Total:    n^[l] × (n^[l-1] + 1)

    Total network parameters:
                L
        P = Σ   n^[l] × (n^[l-1] + 1)
              l=1
```

### 4.2 Detailed Worked Example

**Architecture**: 784 → 256 → 128 → 64 → 10

```
    ┌───────┬──────────┬───────────────────┬────────┬─────────┬──────────┐
    │ Layer │ From→To  │ Weight Matrix     │ Bias   │ Params  │ Cumul.   │
    ├───────┼──────────┼───────────────────┼────────┼─────────┼──────────┤
    │   1   │ 784→256  │ 256×784 = 200,704│  256   │ 200,960 │ 200,960  │
    │   2   │ 256→128  │ 128×256 = 32,768 │  128   │  32,896 │ 233,856  │
    │   3   │ 128→64   │  64×128 =  8,192 │   64   │   8,256 │ 242,112  │
    │   4   │  64→10   │  10×64  =    640 │   10   │     650 │ 242,762  │
    ├───────┼──────────┼───────────────────┼────────┼─────────┼──────────┤
    │ Total │          │          242,304  │   458  │ 242,762 │          │
    └───────┴──────────┴───────────────────┴────────┴─────────┴──────────┘

    Note: First layer dominates! (200,960 / 242,762 = 82.8% of all parameters)
    This is typical — the first layer connecting to high-dimensional input is largest.
```

### 4.3 Memory Requirements

```
    Each parameter is typically stored as:
    • float32: 4 bytes  (standard training)
    • float16: 2 bytes  (mixed precision training)
    • int8:    1 byte   (quantized inference)

    For our 242,762 parameter network:
    • float32: 242,762 × 4 = 971,048 bytes ≈ 0.93 MB
    • float16: 242,762 × 2 = 485,524 bytes ≈ 0.46 MB

    For GPT-3 (175 billion parameters):
    • float32: 175B × 4 = 700 GB  ← needs multiple GPUs!
    • float16: 175B × 2 = 350 GB
    • int8:    175B × 1 = 175 GB  (quantized)
```

---

## 5. Python Implementation — Parameter Inspection

```python
import numpy as np

class ParameterInspector:
    """Analyze weight matrices and biases in a neural network."""
    
    def __init__(self, layer_sizes):
        """
        Parameters:
            layer_sizes: list of ints, e.g., [784, 256, 128, 10]
        """
        self.layer_sizes = layer_sizes
        self.L = len(layer_sizes) - 1
        
        # Initialize parameters
        self.W = {}
        self.b = {}
        for l in range(1, self.L + 1):
            n_l = layer_sizes[l]
            n_l_prev = layer_sizes[l-1]
            # Xavier/Glorot initialization
            scale = np.sqrt(2.0 / (n_l + n_l_prev))
            self.W[l] = np.random.randn(n_l, n_l_prev) * scale
            self.b[l] = np.zeros((n_l, 1))
    
    def summary(self):
        """Print detailed parameter summary."""
        print(f"\nNetwork Architecture: {' → '.join(map(str, self.layer_sizes))}")
        print("=" * 70)
        print(f"{'Layer':>6} │ {'Shape':>12} │ {'Weights':>10} │ {'Biases':>7} │ {'Total':>10} │ {'%':>6}")
        print("─" * 70)
        
        total_w, total_b = 0, 0
        for l in range(1, self.L + 1):
            n_w = self.W[l].size
            n_b = self.b[l].size
            total = n_w + n_b
            total_w += n_w
            total_b += n_b
            shape = f"{self.layer_sizes[l]}×{self.layer_sizes[l-1]}"
            pct = 0  # calculated after
            print(f"{l:>6} │ {shape:>12} │ {n_w:>10,} │ {n_b:>7,} │ {total:>10,} │")
        
        grand_total = total_w + total_b
        print("─" * 70)
        print(f"{'Total':>6} │ {'':>12} │ {total_w:>10,} │ {total_b:>7,} │ {grand_total:>10,} │ 100.0%")
        print(f"\nMemory: {grand_total * 4 / 1024 / 1024:.2f} MB (float32)")
        print(f"        {grand_total * 2 / 1024 / 1024:.2f} MB (float16)")
    
    def inspect_layer(self, l):
        """Inspect a specific layer's parameters."""
        print(f"\n--- Layer {l} ---")
        print(f"W^[{l}] shape: {self.W[l].shape}")
        print(f"b^[{l}] shape: {self.b[l].shape}")
        print(f"W^[{l}] stats: mean={self.W[l].mean():.4f}, "
              f"std={self.W[l].std():.4f}, "
              f"min={self.W[l].min():.4f}, "
              f"max={self.W[l].max():.4f}")
        print(f"b^[{l}] stats: mean={self.b[l].mean():.4f}, "
              f"std={self.b[l].std():.4f}")
        
        # Show a small slice of the weight matrix
        rows, cols = min(4, self.W[l].shape[0]), min(4, self.W[l].shape[1])
        print(f"\nW^[{l}] (top-left {rows}×{cols} slice):")
        for i in range(rows):
            row_str = " ".join(f"{self.W[l][i, j]:>7.4f}" for j in range(cols))
            print(f"  [{row_str} ...]")
        if self.W[l].shape[0] > rows:
            print(f"  [...{' ':>{cols*8}}]")

# Demo
np.random.seed(42)

# Common architectures
print("=" * 70)
print("COMMON NEURAL NETWORK ARCHITECTURES — Parameter Analysis")
print("=" * 70)

architectures = {
    "MNIST Simple":    [784, 128, 10],
    "MNIST Deep":      [784, 256, 128, 64, 10],
    "CIFAR-10 MLP":    [3072, 1024, 512, 256, 10],
    "Tiny Network":    [4, 8, 6, 3],
    "Wide Network":    [100, 1000, 1000, 10],
}

for name, sizes in architectures.items():
    inspector = ParameterInspector(sizes)
    print(f"\n{'─'*70}")
    print(f"  {name}")
    inspector.summary()

# Detailed inspection of a small network
print("\n" + "=" * 70)
print("DETAILED INSPECTION — Tiny Network [4 → 8 → 6 → 3]")
print("=" * 70)
inspector = ParameterInspector([4, 8, 6, 3])
inspector.inspect_layer(1)
inspector.inspect_layer(2)
inspector.inspect_layer(3)
```

---

## 6. PyTorch — Accessing Weights and Biases

```python
import torch
import torch.nn as nn

model = nn.Sequential(
    nn.Linear(784, 256),
    nn.ReLU(),
    nn.Linear(256, 128),
    nn.ReLU(),
    nn.Linear(128, 10)
)

# Access all parameters
print("All named parameters:")
for name, param in model.named_parameters():
    print(f"  {name:20s} shape: {str(list(param.shape)):15s} "
          f"numel: {param.numel():>8,}")

# Access specific layer weights
print(f"\nLayer 0 (first linear) weight shape: {model[0].weight.shape}")
print(f"Layer 0 (first linear) bias shape:   {model[0].bias.shape}")
print(f"Layer 0 weight[0,:5]: {model[0].weight.data[0, :5]}")

# Total parameters
total = sum(p.numel() for p in model.parameters())
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(f"\nTotal parameters:     {total:,}")
print(f"Trainable parameters: {trainable:,}")
```

---

## 📊 Summary Table

| Concept | Key Point |
|---|---|
| **Weight matrix W^[l]** | Shape: (n^[l], n^[l-1]) — connects layer l-1 to layer l |
| **Bias vector b^[l]** | Shape: (n^[l], 1) — one bias per neuron in layer l |
| **Forward equation** | z^[l] = W^[l] · a^[l-1] + b^[l] |
| **Weight meaning** | wᵢⱼ > 0: excitatory; wᵢⱼ < 0: inhibitory; wᵢⱼ ≈ 0: no connection |
| **Bias role** | Shifts activation threshold; allows non-zero output with zero input |
| **Parameter count** | Layer l: n^[l] × n^[l-1] + n^[l] = n^[l] × (n^[l-1] + 1) |
| **Total parameters** | P = Σ n^[l] × (n^[l-1] + 1) for l=1..L |
| **Dominance** | First layer (connecting to input) typically has the most parameters |
| **Memory** | float32: 4 bytes/param; float16: 2 bytes/param |

---

## ❓ Revision Questions

1. **For a network with architecture 100 → 64 → 32 → 16 → 5, write the exact shape of every weight matrix and bias vector. Then compute the total parameter count.**

2. **Explain why the weight matrix W^[l] has shape (n^[l], n^[l-1]) and not (n^[l-1], n^[l]). How does this relate to the matrix multiplication z = Wa?**

3. **What would happen if we removed all biases from a network? Give a specific example where the network would fail without biases.**

4. **A colleague tells you their model has 1 million parameters. If the architecture is 784 → H → 10, what is H? (Set up and solve the equation.)**

5. **Explain the relationship between weight magnitude and connection strength. What does it mean if a trained weight is very close to zero?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Input, Hidden & Output Layers](01-input-hidden-output-layers.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Forward Pass Computation →](03-forward-pass-computation.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
