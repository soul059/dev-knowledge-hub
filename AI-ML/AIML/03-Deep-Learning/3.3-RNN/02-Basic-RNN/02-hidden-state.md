# Hidden State Dynamics

> **Unit 2, Chapter 2** — Understanding h_t = tanh(W_xh·x_t + W_hh·h_{t-1} + b), the hidden state as memory, and how information accumulates over time.

---

## 📋 Overview

The **hidden state** h_t is the heart of an RNN. It serves as the network's memory, encoding a compressed representation of all inputs seen from time step 1 through time step t. Understanding how it updates and what it stores is key to understanding RNNs.

---

## 🧮 The Hidden State Equation

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   h_t = tanh( W_xh · x_t  +  W_hh · h_{t-1}  +  b_h )   │
│          ───   ──────────     ──────────────      ───      │
│           │        │                │               │       │
│        squash   new input      past memory       bias      │
│        to [-1,1]  contribution   contribution              │
│                                                             │
└─────────────────────────────────────────────────────────────┘

Decomposing the equation:

   Term 1: W_xh · x_t      → "What does the current input tell me?"
   Term 2: W_hh · h_{t-1}  → "What do I remember from the past?"
   Term 3: b_h              → "Default activation bias"
   tanh(·)                  → "Squash everything to [-1, 1]"
```

---

## 🧠 Hidden State as Memory

```
┌────────────────────────────────────────────────────────────┐
│                 MEMORY ACCUMULATION                         │
│                                                            │
│  h₀ = [0, 0, 0, 0]              Empty memory              │
│        │                                                    │
│        ▼                                                    │
│  h₁ = tanh(W·x₁ + W·h₀)        Remembers: x₁             │
│        │                                                    │
│        ▼                                                    │
│  h₂ = tanh(W·x₂ + W·h₁)        Remembers: x₁, x₂        │
│        │                                                    │
│        ▼                                                    │
│  h₃ = tanh(W·x₃ + W·h₂)        Remembers: x₁, x₂, x₃   │
│        │                                                    │
│        ▼                                                    │
│  h₄ = tanh(W·x₄ + W·h₃)        Remembers: x₁...x₄       │
│                                                            │
│  Each h_t is a LOSSY COMPRESSION of inputs x₁ through x_t │
│  Recent inputs have stronger influence (more on this       │
│  when we discuss vanishing gradients).                     │
└────────────────────────────────────────────────────────────┘
```

---

## 📐 Why tanh?

```
tanh(x) = (eˣ - e⁻ˣ) / (eˣ + e⁻ˣ)

Properties:
  • Range: (-1, 1)     → bounds hidden state values
  • Zero-centered      → helps gradient flow
  • Saturates at ±1    → can cause vanishing gradients

        1 |        ___________
          |       /
          |      /
        0 |─────/──────────────
          |    /
          |   /
       -1 |__/
          └──────────────────────
           -4  -2   0   2   4

  tanh derivative: 1 - tanh²(x)
  Maximum derivative = 1 (at x=0)
  Derivative → 0 as |x| → ∞ (saturation)
```

### Alternative Activations

```
┌──────────┬──────────┬─────────────────────────┐
│ Function │ Range    │ Notes                   │
├──────────┼──────────┼─────────────────────────┤
│ tanh     │ (-1, 1)  │ Default, zero-centered  │
│ ReLU     │ [0, ∞)   │ Can explode in RNNs     │
│ sigmoid  │ (0, 1)   │ Not zero-centered       │
└──────────┴──────────┴─────────────────────────┘

tanh is preferred for vanilla RNNs because:
1. Zero-centered output → balanced gradient updates
2. Bounded output → prevents hidden state explosion
3. Stronger gradients than sigmoid near zero
```

---

## 🔢 Worked Example: Step-by-Step Hidden State

```
Setup: d=2 (input dim), n=2 (hidden dim)

W_xh = [[0.5,  0.3],    W_hh = [[0.8, -0.2],    b_h = [0.0, 0.0]
         [-0.4, 0.6]]            [0.1,  0.7]]

Input sequence:
   x₁ = [1.0, 0.0]
   x₂ = [0.0, 1.0]
   x₃ = [1.0, 1.0]

h₀ = [0.0, 0.0]

───── Step 1: t=1 ─────
W_xh · x₁ = [0.5·1.0 + 0.3·0.0,  -0.4·1.0 + 0.6·0.0] = [0.50, -0.40]
W_hh · h₀ = [0.8·0.0 + (-0.2)·0.0, 0.1·0.0 + 0.7·0.0] = [0.00,  0.00]
Sum        = [0.50, -0.40]
h₁ = tanh([0.50, -0.40]) = [0.4621, -0.3799]

───── Step 2: t=2 ─────
W_xh · x₂ = [0.5·0.0 + 0.3·1.0,  -0.4·0.0 + 0.6·1.0] = [0.30,  0.60]
W_hh · h₁ = [0.8·0.4621 + (-0.2)·(-0.3799), 
              0.1·0.4621 + 0.7·(-0.3799)]
           = [0.3697 + 0.0760,  0.0462 + (-0.2659)]
           = [0.4457, -0.2197]
Sum        = [0.30+0.4457, 0.60+(-0.2197)] = [0.7457, 0.3803]
h₂ = tanh([0.7457, 0.3803]) = [0.6333, 0.3635]

───── Step 3: t=3 ─────
W_xh · x₃ = [0.5·1.0 + 0.3·1.0,  -0.4·1.0 + 0.6·1.0] = [0.80,  0.20]
W_hh · h₂ = [0.8·0.6333 + (-0.2)·0.3635, 
              0.1·0.6333 + 0.7·0.3635]
           = [0.5066 + (-0.0727),  0.0633 + 0.2545]
           = [0.4339, 0.3178]
Sum        = [0.80+0.4339, 0.20+0.3178] = [1.2339, 0.5178]
h₃ = tanh([1.2339, 0.5178]) = [0.8438, 0.4742]

Summary:
   h₀ = [0.0000,  0.0000]   (empty memory)
   h₁ = [0.4621, -0.3799]   (remembers x₁)
   h₂ = [0.6333,  0.3635]   (remembers x₁, x₂)
   h₃ = [0.8438,  0.4742]   (remembers x₁, x₂, x₃)
```

---

## 🔍 How Information Persists (and Decays)

```
Information from x₁ in h₃:

h₁ = tanh(W_xh · x₁ + W_hh · h₀)                    ← x₁ directly
h₂ = tanh(W_xh · x₂ + W_hh · tanh(W_xh·x₁ + ...))   ← x₁ through h₁
h₃ = tanh(W_xh · x₃ + W_hh · tanh(W_xh·x₂ + W_hh·tanh(W_xh·x₁ + ...)))
                                                        ← x₁ deeply nested

As time progresses:
   • x₁'s influence is repeatedly transformed by W_hh and squashed by tanh
   • Each transformation can reduce x₁'s contribution
   • This is why vanilla RNNs struggle with LONG-TERM dependencies

Signal strength of x₁ at time t (approximately):
   ||∂h_t/∂x₁|| ∝ ||W_hh||^(t-1) · ∏ tanh'(·)
   
   If ||W_hh|| < 1 → signal VANISHES exponentially
   If ||W_hh|| > 1 → signal EXPLODES exponentially
```

---

## 💻 Python Implementation

```python
import numpy as np
import torch
import torch.nn as nn
import matplotlib.pyplot as plt

# ─── Manual hidden state computation ───
def rnn_forward_manual(X, W_xh, W_hh, b_h, h0=None):
    """
    X:    (T, d) input sequence
    W_xh: (n, d) input weights
    W_hh: (n, n) recurrent weights
    b_h:  (n,)   bias
    h0:   (n,)   initial hidden state
    """
    T, d = X.shape
    n = W_hh.shape[0]
    
    h = np.zeros(n) if h0 is None else h0.copy()
    hidden_states = [h.copy()]
    
    for t in range(T):
        h = np.tanh(W_xh @ X[t] + W_hh @ h + b_h)
        hidden_states.append(h.copy())
    
    return np.array(hidden_states)  # (T+1, n)

# Example
np.random.seed(42)
T, d, n = 20, 3, 4
W_xh = np.random.randn(n, d) * 0.5
W_hh = np.random.randn(n, n) * 0.5
b_h  = np.zeros(n)
X    = np.random.randn(T, d)

states = rnn_forward_manual(X, W_xh, W_hh, b_h)

print("Hidden state evolution:")
print(f"  h₀  = {states[0].round(4)}")
print(f"  h₁  = {states[1].round(4)}")
print(f"  h₁₀ = {states[10].round(4)}")
print(f"  h₂₀ = {states[20].round(4)}")
print(f"\nAll values bounded in (-1, 1) due to tanh")

# Visualize hidden state norms over time
norms = [np.linalg.norm(s) for s in states]
print(f"\nHidden state norms: {[f'{n:.3f}' for n in norms[:6]]}...")
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| h_t | Hidden state at time t — the network's "memory" |
| W_xh · x_t | Current input contribution to memory |
| W_hh · h_{t-1} | Past memory contribution (recurrence) |
| tanh | Activation function bounding h_t to (-1, 1) |
| Information Accumulation | h_t encodes all of x₁ through x_t |
| Lossy Compression | h_t has fixed size n, cannot perfectly store all past |
| Signal Decay | Older information weakens through repeated transformations |
| Initial State h₀ | Usually zero vector; starts with empty memory |

---

## ❓ Revision Questions

1. **In the equation h_t = tanh(W_xh·x_t + W_hh·h_{t-1} + b_h), what does each of the three terms inside tanh contribute?**

2. **Work through one time step: Given h_{t-1} = [0.5, -0.3], x_t = [1.0], W_xh = [[0.4], [0.6]], W_hh = [[0.7, 0.1], [-0.2, 0.8]], b_h = [0, 0], compute h_t.**

3. **Why is tanh preferred over ReLU for vanilla RNN hidden states? What would happen with ReLU?**

4. **Explain why information from x₁ becomes weaker in h₁₀₀ compared to h₂. What mathematical property causes this?**

5. **If the hidden dimension is n=50, how many numbers must the network use to summarize an entire 1000-step input sequence?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [RNN Architecture](01-rnn-architecture.md) |
| ➡️ Next | [Unrolling Through Time](03-unrolling-through-time.md) |
| ⬆️ Unit Overview | [README](../README.md) |
