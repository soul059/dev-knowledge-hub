# Vanishing Gradient Problem

> **Unit 3, Chapter 3** — Why gradients vanish in vanilla RNNs: tanh derivative < 1, W_hh eigenvalues < 1, and the failure to learn long-term dependencies.

---

## 📋 Overview

The **vanishing gradient problem** is the most critical limitation of vanilla RNNs. When training on long sequences, gradients flowing backward through time become exponentially small, making it impossible to learn dependencies between distant time steps. This was identified by Hochreiter (1991) and Bengio et al. (1994).

---

## ❌ The Problem Visualized

```
Input:  "I grew up in France ... (50 words) ... I speak fluent ___"
                  ↑                                           ↑
              Relevant info                          Prediction point
              (50 steps back)

Gradient signal from "___" trying to reach "France":

Loss at T ──▶ δ_T ──▶ δ_{T-1} ──▶ ... ──▶ δ_{T-50}
                                              ↑
                    Each step multiplies by ≈ 0.7-0.9

If factor = 0.8:  0.8^50 = 0.000014  ← gradient is essentially ZERO

The network CANNOT learn that "France" is relevant to predicting "French"
because the gradient signal dies before reaching that far back!
```

---

## 🧮 Why Gradients Vanish

### Cause 1: tanh Derivative is ≤ 1

```
tanh'(x) = 1 - tanh²(x)

Maximum value: tanh'(0) = 1
For |x| > 0: tanh'(x) < 1

tanh'(x)
   1.0 │      ∧
       │     / \
       │    /   \
   0.5 │   /     \
       │  /       \
       │ /         \
   0.0 │/           \________
       └──────────────────────
       -4  -2   0   2   4

At each time step, the gradient is multiplied by tanh'(a_t) ≤ 1
Over T steps: ∏ tanh'(a_t) ≤ 1^T = 1, but typically << 1
```

### Cause 2: W_hh Eigenvalues < 1

```
Jacobian at each step: J_t = diag(1-h_t²) · W_hh

Gradient from time T to time k:
   ∏(t=k+1 to T) J_t

If spectral radius ρ(W_hh) < 1:
   ||J_t|| < 1 for each t
   ||∏ J_t|| ≤ ∏ ||J_t|| → 0 exponentially

Even if ρ(W_hh) = 1, the tanh derivative < 1 
causes overall shrinkage!
```

### Combined Effect

```
Effective shrinkage per step ≈ max_singular_value(W_hh) × avg(tanh')

Typical values:
   σ_max(W_hh) ≈ 0.9     (initialized near 1 but < 1)
   avg(tanh')  ≈ 0.65     (typical for non-saturated units)

Combined factor ≈ 0.9 × 0.65 = 0.585 per step

After n steps:
   n=5:   0.585^5  = 0.0716    (93% gradient lost)
   n=10:  0.585^10 = 0.00513   (99.5% lost)
   n=20:  0.585^20 = 0.0000263 (essentially zero)
   n=50:  0.585^50 ≈ 10⁻¹²    (numerical zero!)
```

---

## 📊 Long-Term Dependency Failure

### Experiment: Remembering a Signal

```
Task: Remember the first input and reproduce it at the end

Input:   [signal, noise, noise, noise, ..., noise, ???]
         t=1      t=2    t=3    t=4          t=T-1  t=T

Target:  reproduce signal at t=T

         Signal strength in hidden state:
         
   1.0 │█
       │█
       │█░
       │█░░
   0.5 │█░░░
       │█░░░░
       │█░░░░░░
       │█░░░░░░░░░
   0.0 │█░░░░░░░░░░░░░░░░░░░
       └──────────────────────
       t=1   t=5   t=10  t=20

Signal FADES as information passes through repeated
tanh squashing and W_hh multiplication.
```

---

## 💻 Demonstration

```python
import torch
import torch.nn as nn
import numpy as np

# ─── Experiment: Vanilla RNN fails on long-range dependencies ───
class VanillaRNN(nn.Module):
    def __init__(self, input_size, hidden_size):
        super().__init__()
        self.rnn = nn.RNN(input_size, hidden_size, batch_first=True)
        self.fc = nn.Linear(hidden_size, 1)
    
    def forward(self, x):
        output, _ = self.rnn(x)
        return self.fc(output[:, -1, :])  # Use last hidden state

def copy_first_task(seq_len, n_samples=1000):
    """Task: output the first element of the sequence"""
    X = torch.randn(n_samples, seq_len, 1)
    # Target = first element
    y = X[:, 0, 0:1]
    return X, y

# Test with different sequence lengths
print("=== Vanilla RNN: Copy First Element Task ===")
print(f"{'Seq Len':<10} {'Final Loss':>12} {'Learned?':>10}")
print("-" * 35)

for seq_len in [5, 10, 20, 50, 100, 200]:
    model = VanillaRNN(1, 32)
    optimizer = torch.optim.Adam(model.parameters(), lr=0.01)
    
    X, y = copy_first_task(seq_len)
    
    # Train for 200 steps
    for epoch in range(200):
        pred = model(X)
        loss = ((pred - y) ** 2).mean()
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
    
    final_loss = loss.item()
    learned = "✓" if final_loss < 0.1 else "✗ FAILED"
    print(f"{seq_len:<10} {final_loss:>12.6f} {learned:>10}")

# ─── Visualize gradient norms ───
print("\n=== Gradient Norms at Each Time Step ===")
model = VanillaRNN(1, 32)
x = torch.randn(1, 50, 1, requires_grad=True)
output, _ = model.rnn(x)
loss = output[0, -1, :].sum()
loss.backward()

grad = x.grad[0]  # (50, 1)
for t in [0, 5, 10, 20, 30, 40, 49]:
    print(f"  t={t:>2}: gradient norm = {grad[t].norm().item():.8f}")
print("\nGradient at early time steps is MUCH smaller → vanishing!")
```

---

## 📐 Why This Matters Practically

```
Tasks that FAIL with vanilla RNNs:

1. Language Modeling (long context)
   "The trophy didn't fit in the suitcase because it was too [big/small]"
   Need to remember "trophy" or "suitcase" to resolve "it"

2. Machine Translation (long sentences)
   Word order changes between languages require long-range alignment

3. Music Generation
   Musical patterns repeat with long periods (verse-chorus structure)

4. Time Series (seasonal patterns)
   Monthly patterns require remembering 30+ steps back

Tasks that WORK with vanilla RNNs:
   Short sequences (< 10 steps)
   Dependencies only between adjacent steps
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Vanishing Gradients | Gradients → 0 exponentially with distance |
| Cause: tanh' ≤ 1 | Each step multiplies gradient by factor ≤ 1 |
| Cause: \|λ(W_hh)\| < 1 | Repeated multiplication by small matrix shrinks signal |
| Effect | Cannot learn dependencies beyond ~10-20 time steps |
| Detection | Training loss plateaus, model ignores early inputs |
| NOT Fixed By | Longer training, larger learning rate, bigger hidden size |
| Fixed By | LSTM/GRU (additive paths), skip connections |
| Historical | Identified by Hochreiter (1991), formalized by Bengio (1994) |

---

## ❓ Revision Questions

1. **If the effective shrinkage factor per time step is 0.7, how many time steps before the gradient is less than 1% of its original magnitude?**

2. **Draw the tanh derivative curve. At what value of the pre-activation does the derivative equal 0.5?**

3. **Explain the "copy first element" task. Why does a vanilla RNN fail at this for sequences longer than ~20 steps?**

4. **Why can't we solve vanishing gradients by simply using a larger learning rate?**

5. **What is the relationship between the eigenvalues of W_hh and gradient flow? How would you initialize W_hh to slow down vanishing?**

6. **Give two real-world NLP examples where vanishing gradients would prevent a vanilla RNN from learning the correct output.**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Gradient Flow](02-gradient-flow.md) |
| ➡️ Next | [Exploding Gradient Problem](04-exploding-gradient-problem.md) |
| ⬆️ Unit Overview | [README](../README.md) |
