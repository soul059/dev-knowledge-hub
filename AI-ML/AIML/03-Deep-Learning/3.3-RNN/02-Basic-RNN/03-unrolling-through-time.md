# Unrolling Through Time

> **Unit 2, Chapter 3** — Visualizing RNNs as unrolled computational graphs across time steps for forward and backward computation.

---

## 📋 Overview

"Unrolling" an RNN means converting the compact, looped representation into an explicit computational graph where each time step has its own copy of the RNN cell. This view is essential for understanding **forward propagation** and **Backpropagation Through Time (BPTT)**.

---

## 🔄 Folded → Unrolled Transformation

### Folded View (Compact)

```
      ┌──────────────┐
      │              │
      │    ┌─────┐   │
x_t ──┼──▶│ RNN │───┼──▶ y_t
      │    │Cell │   │
      │    └──┬──┘   │
      │       │      │
      └───────┘      │
       h_{t-1}→h_t   │
```

### Unrolled View (Explicit)

```
Time:    t=1         t=2         t=3         t=4         t=5
         │           │           │           │           │
Input:   x₁          x₂          x₃          x₄          x₅
         │           │           │           │           │
         ▼           ▼           ▼           ▼           ▼
       ┌─────┐     ┌─────┐     ┌─────┐     ┌─────┐     ┌─────┐
h₀ ──▶│ RNN │──▶ │ RNN │──▶ │ RNN │──▶ │ RNN │──▶ │ RNN │──▶ h₅
       │Cell │    │Cell │    │Cell │    │Cell │    │Cell │
       └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘
          │          │          │          │          │
          ▼          ▼          ▼          ▼          ▼
         y₁         y₂         y₃         y₄         y₅
         │          │          │          │          │
         ▼          ▼          ▼          ▼          ▼
         L₁         L₂         L₃         L₄         L₅

Total Loss:  L = L₁ + L₂ + L₃ + L₄ + L₅

IMPORTANT: All cells share the SAME parameters {W_xh, W_hh, W_hy, b_h, b_y}
```

---

## 📐 Detailed Computational Graph

```
x₁                  x₂                  x₃
│                   │                   │
▼                   ▼                   ▼
┌────────┐          ┌────────┐          ┌────────┐
│W_xh·x₁│          │W_xh·x₂│          │W_xh·x₃│
└───┬────┘          └───┬────┘          └───┬────┘
    │                   │                   │
    ▼                   ▼                   ▼
  ┌───┐    h₁        ┌───┐    h₂        ┌───┐    h₃
h₀│ + │──────────────▶│ + │──────────────▶│ + │──────▶
  └─┬─┘    ▲          └─┬─┘    ▲          └─┬─┘
    │      │            │      │            │
    ▼      │            ▼      │            ▼
  ┌────┐ ┌────────┐   ┌────┐ ┌────────┐   ┌────┐
  │tanh│ │W_hh·h₀│   │tanh│ │W_hh·h₁│   │tanh│
  └─┬──┘ └────────┘   └─┬──┘ └────────┘   └─┬──┘
    │                    │                    │
    ▼                    ▼                    ▼
  ┌────────┐           ┌────────┐           ┌────────┐
  │W_hy·h₁│           │W_hy·h₂│           │W_hy·h₃│
  └───┬────┘           └───┬────┘           └───┬────┘
      │                    │                    │
      ▼                    ▼                    ▼
     y₁                   y₂                   y₃
      │                    │                    │
      ▼                    ▼                    ▼
   L(y₁,ŷ₁)           L(y₂,ŷ₂)           L(y₃,ŷ₃)
```

---

## 🧮 Why Unrolling Matters

### For Forward Pass

```
The unrolled graph makes the computation explicit:

Step 1:  h₁ = tanh(W_xh · x₁ + W_hh · h₀ + b_h)
         y₁ = W_hy · h₁ + b_y

Step 2:  h₂ = tanh(W_xh · x₂ + W_hh · h₁ + b_h)
         y₂ = W_hy · h₂ + b_y

Step 3:  h₃ = tanh(W_xh · x₃ + W_hh · h₂ + b_h)
         y₃ = W_hy · h₃ + b_y

Each step depends on the previous → SEQUENTIAL computation
(Cannot be parallelized like feedforward or CNN!)
```

### For Backward Pass (BPTT)

```
Gradients flow BACKWARD through the unrolled graph:

L₃ → ∂L₃/∂y₃ → ∂y₃/∂h₃ → ∂h₃/∂W_xh (direct)
                          → ∂h₃/∂h₂ → ∂h₂/∂W_xh (through time)
                                     → ∂h₂/∂h₁ → ∂h₁/∂W_xh (further back)

The unrolled view shows ALL paths gradients can take,
including paths that go back many time steps.
```

---

## 📊 Unrolled Views for Different Tasks

### Many-to-One (e.g., Sentiment Classification)

```
   x₁      x₂      x₃      x₄      x₅
    │       │       │       │       │
    ▼       ▼       ▼       ▼       ▼
  ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
  │ h₁│──▶│ h₂│──▶│ h₃│──▶│ h₄│──▶│ h₅│
  └───┘   └───┘   └───┘   └───┘   └─┬─┘
                                      │     Only LAST
                                      ▼     hidden state
                                     [y]    produces output
```

### One-to-Many (e.g., Text Generation)

```
   x₁
    │
    ▼
  ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
  │ h₁│──▶│ h₂│──▶│ h₃│──▶│ h₄│──▶│ h₅│
  └─┬─┘   └─┬─┘   └─┬─┘   └─┬─┘   └─┬─┘
    │       │       │       │       │
    ▼       ▼       ▼       ▼       ▼
   y₁      y₂      y₃      y₄      y₅
    │
    └──▶ Fed as input to next step (y₁ becomes x₂, etc.)
```

### Many-to-Many (Encoder-Decoder)

```
ENCODE (no output)              DECODE (outputs at each step)
   x₁    x₂    x₃              <s>    y₁    y₂
    │     │     │                │     │     │
    ▼     ▼     ▼                ▼     ▼     ▼
  ┌───┐ ┌───┐ ┌───┐  context  ┌───┐ ┌───┐ ┌───┐
  │ h₁│▶│ h₂│▶│ h₃│──────────▶│ s₁│▶│ s₂│▶│ s₃│
  └───┘ └───┘ └───┘           └─┬─┘ └─┬─┘ └─┬─┘
                                 │     │     │
                                 ▼     ▼     ▼
                                y₁    y₂   <eos>
```

---

## 💻 Implementation: Unrolled Forward Pass

```python
import torch
import torch.nn as nn
import numpy as np

class UnrolledRNN:
    """Explicitly unrolled RNN for educational purposes"""
    
    def __init__(self, input_size, hidden_size, output_size):
        scale = 0.1
        self.W_xh = np.random.randn(hidden_size, input_size) * scale
        self.W_hh = np.random.randn(hidden_size, hidden_size) * scale
        self.W_hy = np.random.randn(output_size, hidden_size) * scale
        self.b_h  = np.zeros(hidden_size)
        self.b_y  = np.zeros(output_size)
    
    def forward_unrolled(self, X):
        """
        X: (T, input_size) — input sequence
        Returns: outputs, hidden_states (the full unrolled computation)
        """
        T = X.shape[0]
        n = self.W_hh.shape[0]
        
        # Store ALL intermediate values (needed for BPTT)
        h_states = [np.zeros(n)]  # h₀
        outputs = []
        pre_activations = []      # Before tanh (needed for gradient)
        
        for t in range(T):
            # Pre-activation
            a_t = self.W_xh @ X[t] + self.W_hh @ h_states[-1] + self.b_h
            pre_activations.append(a_t)
            
            # Hidden state
            h_t = np.tanh(a_t)
            h_states.append(h_t)
            
            # Output
            y_t = self.W_hy @ h_t + self.b_y
            outputs.append(y_t)
        
        return {
            'outputs': np.array(outputs),
            'hidden_states': np.array(h_states),
            'pre_activations': np.array(pre_activations),
            'inputs': X
        }

# Demo
rnn = UnrolledRNN(input_size=3, hidden_size=4, output_size=2)
X = np.random.randn(5, 3)  # 5 time steps, 3 features

result = rnn.forward_unrolled(X)

print("=== Unrolled Forward Pass ===")
print(f"Input shape:           ({X.shape[0]}, {X.shape[1]})")
print(f"Hidden states stored:  {result['hidden_states'].shape}")  # (6, 4) incl h₀
print(f"Pre-activations:       {result['pre_activations'].shape}")  # (5, 4)
print(f"Outputs:               {result['outputs'].shape}")  # (5, 2)
print(f"\nAll {len(result['hidden_states'])} hidden states preserved for BPTT")

for t in range(5):
    print(f"  t={t+1}: h = {result['hidden_states'][t+1].round(4)}")
```

---

## 🔑 Key Insights

```
1. SAME PARAMETERS everywhere:
   ┌─────┐  ┌─────┐  ┌─────┐
   │ W,b │  │ W,b │  │ W,b │   ← All identical
   └─────┘  └─────┘  └─────┘

2. SEQUENTIAL dependency:
   h₁ must be computed before h₂, h₂ before h₃, etc.
   → No parallelism within a sequence (major limitation!)

3. GRADIENT PATHS get longer:
   ∂L₅/∂W passes through h₅ ← h₄ ← h₃ ← h₂ ← h₁
   → Potential vanishing/exploding gradients

4. MEMORY grows linearly:
   Must store all h₁, h₂, ..., h_T for backprop
   → Memory usage ∝ sequence length T
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Folded View | Compact single-cell with self-loop |
| Unrolled View | Explicit copy at each time step |
| Same Parameters | All copies share W_xh, W_hh, W_hy |
| Sequential Computation | h_t depends on h_{t-1}, no parallelism |
| Computational Graph | Unrolled view is the graph for autograd |
| Memory Cost | Must store all hidden states for BPTT |
| Gradient Paths | Gradients flow back through all time steps |

---

## ❓ Revision Questions

1. **Draw the unrolled RNN for a 4-step sequence. Label all weight matrices and show which ones are shared.**

2. **Why can't the forward pass of an RNN be parallelized across time steps like a CNN can be parallelized across spatial locations?**

3. **In the unrolled view, how many times is W_hh "used"? How does this affect gradient computation?**

4. **For a many-to-one task (e.g., sentiment analysis), which parts of the unrolled RNN produce outputs?**

5. **If the sequence length is T=1000, how many hidden states must be stored for BPTT? What is the memory implication?**

6. **Compare the unrolled RNN to a very deep feedforward network. What are the similarities and key differences?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Hidden State](02-hidden-state.md) |
| ➡️ Next | [Forward Propagation in RNN](04-forward-propagation-rnn.md) |
| ⬆️ Unit Overview | [README](../README.md) |
