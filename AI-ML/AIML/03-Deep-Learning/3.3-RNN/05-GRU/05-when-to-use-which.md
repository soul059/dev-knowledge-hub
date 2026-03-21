# When to Use Which

> **Unit 5, Chapter 5** — Decision guide for choosing between vanilla RNN, LSTM, and GRU based on dataset size, sequence length, and constraints.

---

## 📋 Overview

Choosing between RNN variants is a practical engineering decision. This chapter provides actionable guidelines based on dataset characteristics, computational budget, and task requirements.

---

## 🗺️ Decision Flowchart

```
                    Start
                      │
                      ▼
              Sequence data?
              │           │
             No          Yes
              │           │
              ▼           ▼
          Use FNN    Sequence length?
                     │         │
                   Short      Long
                   (<20)      (>50)
                     │         │
                     ▼         ▼
               Vanilla    Need long-term
               RNN OK     dependencies?
                          │         │
                         No        Yes
                          │         │
                          ▼         ▼
                        GRU     Dataset size?
                              │         │
                           Small      Large
                           (<10K)     (>100K)
                              │         │
                              ▼         ▼
                            GRU       LSTM
                    (fewer params)  (more capacity)
```

---

## 📊 Decision Matrix

```
┌────────────────────┬──────────┬──────────┬──────────┐
│ Scenario           │ RNN      │ GRU      │ LSTM     │
├────────────────────┼──────────┼──────────┼──────────┤
│ Short sequences    │ ✓✓       │ ✓✓       │ ✓        │
│ Long sequences     │ ✗        │ ✓        │ ✓✓       │
│ Small dataset      │ ✓        │ ✓✓       │ ✓        │
│ Large dataset      │ ✗        │ ✓        │ ✓✓       │
│ Fast prototyping   │ ✓✓       │ ✓✓       │ ✓        │
│ Production NLP     │ ✗        │ ✓        │ ✓✓       │
│ Edge/mobile deploy │ ✓        │ ✓✓       │ ✓        │
│ GPU available      │ ✓        │ ✓        │ ✓✓       │
│ Counting/timing    │ ✗        │ ✓        │ ✓✓       │
│ Simple patterns    │ ✓✓       │ ✓        │ ✓        │
│ Established field  │ ✗        │ ✓        │ ✓✓       │
│ Research/new task  │ ✗        │ ✓✓       │ ✓✓       │
└────────────────────┴──────────┴──────────┴──────────┘

✓✓ = Recommended  ✓ = Acceptable  ✗ = Not recommended
```

---

## 📐 Key Decision Factors

### 1. Dataset Size

```
Small (< 10K samples):
   → GRU preferred (fewer parameters → less overfitting)
   → Vanilla RNN might work for very short sequences

Medium (10K - 100K):
   → Both GRU and LSTM work well
   → GRU trains faster → try GRU first

Large (> 100K):
   → LSTM can leverage its extra capacity
   → Both work, but LSTM may edge out slightly
   → Consider Transformer if dataset is very large
```

### 2. Sequence Length

```
Short (< 20 tokens):
   → Vanilla RNN may suffice
   → GRU/LSTM are safe choices

Medium (20-200 tokens):
   → GRU or LSTM (both handle this well)
   → Vanilla RNN will struggle with longer end

Long (200+ tokens):
   → LSTM preferred (better long-term memory)
   → Consider Transformer for very long sequences (>500)

Very Long (1000+ tokens):
   → Transformer strongly preferred
   → If must use RNN: LSTM + attention
```

### 3. Computational Budget

```
Tight budget / real-time / edge:
   → GRU (25% fewer params, faster inference)
   → Quantized models

Moderate budget:
   → Either works — try GRU first for speed

Unlimited budget:
   → LSTM, or better yet, Transformer
   → Hyperparameter search over both GRU and LSTM
```

---

## 🔬 Empirical Guidelines

```
┌──────────────────────────────────────────────────────────────────┐
│                 PRACTICAL RECOMMENDATIONS                        │
│                                                                  │
│  1. DEFAULT: Start with GRU                                     │
│     • Faster to train, fewer hyperparameters                    │
│     • Often performs as well as LSTM                             │
│     • Easier to debug                                           │
│                                                                  │
│  2. SWITCH TO LSTM IF:                                          │
│     • GRU underperforms after tuning                            │
│     • Task requires very long-range dependencies               │
│     • You have enough data and compute                          │
│     • Task involves counting or precise timing                  │
│                                                                  │
│  3. CONSIDER TRANSFORMER IF:                                    │
│     • Sequences are very long (>500)                            │
│     • You need parallelism during training                      │
│     • State-of-the-art performance is critical                  │
│     • You have large amounts of data                            │
│                                                                  │
│  4. ALWAYS:                                                     │
│     • Try both and compare on validation set                    │
│     • The difference is often < 1% accuracy                     │
│     • Architecture matters less than data quality               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 💻 Quick Comparison Framework

```python
import torch
import torch.nn as nn

class QuickCompare(nn.Module):
    """Easily switch between RNN, GRU, LSTM"""
    def __init__(self, rnn_type, input_size, hidden_size, output_size, num_layers=1):
        super().__init__()
        rnn_types = {'RNN': nn.RNN, 'GRU': nn.GRU, 'LSTM': nn.LSTM}
        self.rnn = rnn_types[rnn_type](
            input_size, hidden_size, num_layers=num_layers, batch_first=True
        )
        self.fc = nn.Linear(hidden_size, output_size)
        self.rnn_type = rnn_type
    
    def forward(self, x):
        output, hidden = self.rnn(x)
        if self.rnn_type == 'LSTM':
            hidden = hidden[0]  # Use h_n, not c_n
        return self.fc(hidden[-1])

# Compare all three
for rnn_type in ['RNN', 'GRU', 'LSTM']:
    model = QuickCompare(rnn_type, input_size=20, hidden_size=64, output_size=5)
    params = sum(p.numel() for p in model.parameters())
    print(f"{rnn_type:>4}: {params:>6,} parameters")

# Output:
#  RNN:  6,085 parameters
#  GRU: 16,965 parameters
# LSTM: 22,405 parameters
```

---

## 📋 Summary Table

| Factor | Choose GRU | Choose LSTM |
|--------|-----------|-------------|
| Dataset size | Small-medium | Large |
| Sequence length | Short-medium | Long |
| Training speed priority | Yes | No |
| Memory/compute constraint | Yes | No |
| Long-range dependencies | Moderate | Critical |
| Research tradition | New task | Established benchmarks |
| Model simplicity | Preferred | Not a concern |

---

## ❓ Revision Questions

1. **You have 5000 training samples with sequences of length 30. Which architecture would you start with and why?**

2. **A speech recognition system processes 1-second audio clips (100 frames). Would you recommend RNN, GRU, or LSTM?**

3. **When would you choose vanilla RNN over both GRU and LSTM?**

4. **Your LSTM model is overfitting. Would switching to GRU likely help? Why?**

5. **At what sequence length do you start considering Transformers over LSTMs?**

6. **"Architecture choice matters less than data quality." Explain this statement in the context of GRU vs LSTM.**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [LSTM vs GRU Comparison](04-lstm-vs-gru-comparison.md) |
| ➡️ Next | [Bidirectional RNN](../06-Advanced-RNN-Architectures/01-bidirectional-rnn.md) |
| ⬆️ Unit Overview | [README](../README.md) |
