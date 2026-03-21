# LSTM vs GRU Comparison

> **Unit 5, Chapter 4** — Head-to-head comparison: parameter count, performance, training speed, and when to use which.

---

## 📋 Overview

LSTM and GRU are the two most widely used gated RNN architectures. Neither is universally better — the choice depends on the dataset, task, and computational constraints. This chapter provides a thorough comparison.

---

## 📊 Structural Comparison

```
┌─────────────────────┬──────────────────────┬──────────────────────┐
│ Feature             │ LSTM                 │ GRU                  │
├─────────────────────┼──────────────────────┼──────────────────────┤
│ Year introduced     │ 1997                 │ 2014                 │
│ Gates               │ 3 (forget,input,out) │ 2 (update, reset)    │
│ State vectors       │ 2 (C_t, h_t)         │ 1 (h_t)              │
│ Output gating       │ Yes                  │ No                   │
│ Forget/Input        │ Independent          │ Coupled (z, 1-z)     │
│ Cell state          │ Separate C_t         │ Merged into h_t      │
│ Reset mechanism     │ Via forget gate      │ Explicit reset gate  │
└─────────────────────┴──────────────────────┴──────────────────────┘
```

---

## 🧮 Parameter Count Comparison

```
For input_size=d, hidden_size=n:

LSTM parameters:
   4 gates × [n×(n+d) + n] = 4n(n+d) + 4n = 4n² + 4nd + 4n

GRU parameters:
   3 gates × [n×(n+d) + n] = 3n(n+d) + 3n = 3n² + 3nd + 3n

Ratio: GRU/LSTM = 3/4 = 75%

┌──────────────────────────────────────────────────────┐
│  Example: d=100, n=256                               │
│                                                      │
│  LSTM: 4×256×(256+100) + 4×256 = 365,568 + 1,024   │
│      = 366,592 parameters                           │
│                                                      │
│  GRU:  3×256×(256+100) + 3×256 = 274,176 + 768     │
│      = 274,944 parameters                           │
│                                                      │
│  Savings: 91,648 fewer (25% reduction)              │
└──────────────────────────────────────────────────────┘
```

---

## ⚡ Performance Comparison

```
┌──────────────────────┬────────────┬────────────┬─────────────────┐
│ Metric               │ LSTM       │ GRU        │ Winner          │
├──────────────────────┼────────────┼────────────┼─────────────────┤
│ Parameters           │ 4n(n+d+1)  │ 3n(n+d+1)  │ GRU (25% fewer) │
│ Training speed       │ Slower     │ Faster     │ GRU             │
│ Convergence speed    │ Slower     │ Faster     │ GRU             │
│ Memory usage         │ Higher     │ Lower      │ GRU             │
│ Long sequences       │ Better     │ Good       │ LSTM            │
│ Small datasets       │ Overfits   │ Better     │ GRU             │
│ Large datasets       │ Better     │ Good       │ LSTM (slightly) │
│ Very long deps (>100)│ Better     │ Adequate   │ LSTM            │
│ NLP benchmarks       │ ~Equal     │ ~Equal     │ Tie             │
│ Speech recognition   │ Better     │ Good       │ LSTM            │
│ Music modeling       │ Better     │ Good       │ LSTM            │
└──────────────────────┴────────────┴────────────┴─────────────────┘

Key finding (Chung et al., 2014; Jozefowicz et al., 2015):
"No clear winner — performance depends on dataset and task"
```

---

## 📐 Theoretical Differences

```
CAPACITY DIFFERENCE:

LSTM can do things GRU cannot (and vice versa):

1. LSTM: Independent forget and input
   Can simultaneously KEEP all old info AND ADD new info:
   f_t = 1, i_t = 1 → C_t = C_{t-1} + C̃_t  (accumulation)
   
   GRU forces trade-off:
   z_t controls BOTH → (1-z)·old + z·new = 1 (convex combination)

2. LSTM: Output gating
   Can store info in C_t without exposing it in h_t:
   o_t ≈ 0 → h_t ≈ 0, but C_t may have rich information
   
   GRU: h_t IS the state → everything is always "visible"

3. GRU: Explicit reset
   r_t ≈ 0 makes candidate ignore ALL history
   LSTM forget gate can only erase cell state, not h_{t-1}'s
   contribution to gate computations

GRADIENT FLOW:
   LSTM: ∂C_t/∂C_{t-1} = diag(f_t)
   GRU:  ∂h_t/∂h_{t-1} = diag(1-z_t) + z_t terms
   
   Both create paths for gradient preservation when gates ≈ identity
```

---

## 💻 Empirical Comparison

```python
import torch
import torch.nn as nn
import time

def compare_models(input_size=50, hidden_size=128, seq_len=100, batch_size=32):
    """Compare LSTM and GRU on speed and parameter count"""
    
    lstm = nn.LSTM(input_size, hidden_size, batch_first=True)
    gru = nn.GRU(input_size, hidden_size, batch_first=True)
    
    lstm_params = sum(p.numel() for p in lstm.parameters())
    gru_params = sum(p.numel() for p in gru.parameters())
    
    x = torch.randn(batch_size, seq_len, input_size)
    
    # Speed test
    n_iters = 100
    
    start = time.time()
    for _ in range(n_iters):
        _ = lstm(x)
    lstm_time = time.time() - start
    
    start = time.time()
    for _ in range(n_iters):
        _ = gru(x)
    gru_time = time.time() - start
    
    print("=== LSTM vs GRU Comparison ===")
    print(f"{'Metric':<25} {'LSTM':>12} {'GRU':>12} {'Ratio':>10}")
    print("-" * 60)
    print(f"{'Parameters':<25} {lstm_params:>12,} {gru_params:>12,} "
          f"{gru_params/lstm_params:>10.2%}")
    print(f"{'Time (ms)':<25} {lstm_time*1000/n_iters:>12.2f} "
          f"{gru_time*1000/n_iters:>12.2f} "
          f"{gru_time/lstm_time:>10.2%}")
    print(f"{'Hidden state outputs':<25} {'h_n + c_n':>12} {'h_n':>12}")
    
compare_models()
```

---

## 📋 Summary Table

| Aspect | LSTM | GRU |
|--------|------|-----|
| Parameters | 4n(n+d+1) | 3n(n+d+1) — 25% fewer |
| Gates | 3 (forget, input, output) | 2 (update, reset) |
| States | 2 (cell + hidden) | 1 (hidden only) |
| Speed | Slower (~33%) | Faster |
| Memory | More | Less |
| Expressiveness | More (independent gates) | Less (coupled gates) |
| Best for | Large data, long deps | Small data, fast training |
| Default choice | When unsure | When resources limited |

---

## ❓ Revision Questions

1. **Calculate the exact parameter counts for LSTM and GRU with input_size=20, hidden_size=64.**

2. **What can LSTM do that GRU cannot due to independent forget and input gates?**

3. **Why is GRU faster to train than LSTM despite similar architecture?**

4. **Explain how GRU's lack of an output gate affects its behavior compared to LSTM.**

5. **You have a small dataset (1000 samples) with sequences of length 50. Would you choose LSTM or GRU? Why?**

6. **Jozefowicz et al. (2015) tested many RNN variants. What was their conclusion about LSTM vs GRU?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [GRU Equations](03-gru-equations.md) |
| ➡️ Next | [When to Use Which](05-when-to-use-which.md) |
| ⬆️ Unit Overview | [README](../README.md) |
