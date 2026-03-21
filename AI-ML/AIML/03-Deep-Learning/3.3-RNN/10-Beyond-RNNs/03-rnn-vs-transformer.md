# RNN vs Transformer: Comprehensive Comparison

> **Unit 10, Chapter 3** — Detailed comparison of RNNs and Transformers across speed, parallelism, long-range dependencies, memory, and practical considerations.

---

## 📋 Overview

The transition from RNNs to Transformers represents one of the most significant paradigm shifts in deep learning. Yet the comparison is nuanced — each architecture has distinct strengths. This chapter provides a systematic comparison across every dimension that matters for choosing between them: computational complexity, parallelism, memory usage, performance on different tasks, and practical training considerations.

---

## ⚡ Speed and Parallelism

### Training Speed

```
RNN Training (Sequential):
─────────────────────────
  x₁ → h₁ → x₂ → h₂ → x₃ → h₃ → ... → x_n → h_n
  
  Cannot start h₂ until h₁ is computed
  GPU utilization: one operation at a time
  
  Time complexity: O(n) sequential steps
  Each step: O(d²) for matrix multiply
  Total: O(n · d²)

Transformer Training (Parallel):
─────────────────────────────
  [x₁, x₂, x₃, ..., x_n] → Self-Attention → [z₁, z₂, z₃, ..., z_n]
  
  All positions computed simultaneously
  GPU utilization: full parallelism
  
  Time complexity: O(1) sequential steps
  Self-attention: O(n² · d) total computation
  Total: O(n² · d) but done in O(1) parallel steps
```

### Practical Training Times

```
Task: Language modeling on WikiText-103

  Model           Params    Training Time    Perplexity
  ──────────────  ────────  ──────────────   ──────────
  LSTM (3-layer)  66M       5 days           33.0
  AWD-LSTM        33M       3 days           30.2
  Transformer-XL  44M       1.5 days         24.0
  GPT-2 (small)   117M      8 hours*         21.0
  
  * With modern GPU and mixed precision

Even accounting for more parameters, Transformers
train FASTER due to parallelism.
```

### Inference Speed

```
RNN Inference (autoregressive):
  Generating token n+1 requires only h_n and x_n
  Memory: O(d) — just the hidden state
  Time per token: O(d²)

Transformer Inference (autoregressive):
  Generating token n+1 requires attending to ALL previous tokens
  Memory: O(n · d) — KV cache for all previous positions
  Time per token: O(n · d) — grows linearly with generated length!

  ┌────────────────────────────────────────────────────────┐
  │  RNN wins at inference for very long generation!       │
  │  Each new token is O(d²) regardless of position.       │
  │  Transformer: each new token costs more as n grows.    │
  └────────────────────────────────────────────────────────┘
```

---

## 📏 Long-Range Dependencies

### Maximum Path Length

```
How many transformations must information traverse?

RNN:          x₁ → h₁ → h₂ → h₃ → ... → h_n      Path: O(n)
Transformer:  x₁ ─────────────────────────→ x_n      Path: O(1)
CNN (k=3):    x₁ → [3] → [3] → ... → [3] → x_n     Path: O(n/k) = O(log(n))

                Information preservation over path:

  Signal │
strength │
         │ ╲  RNN
    1.0  │   ╲
         │     ╲___
         │          ╲_______              Transformer
         │  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
    0.5  │                  ╲_______
         │                          ╲___
    0.0  │                               ╲_
         └──────────────────────────────────── Distance
          1    5   10   20   50  100  200

  RNN: signal degrades over distance (vanishing gradient)
  Transformer: constant signal strength (direct attention)
```

### Empirical Evidence

```
Task: Copy problem (remember input after N blank steps)

  Delay    LSTM Accuracy    Transformer Accuracy
  ──────   ──────────────   ────────────────────
  10       99%              99%
  50       95%              99%
  100      78%              99%
  500      12%              99%
  1000     ~0%              98%

Transformers maintain near-perfect accuracy
regardless of distance!
```

---

## 💾 Memory Usage

### Training Memory

```
┌─────────────────────────────────────────────────────────────┐
│              Memory Usage During Training                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  RNN:                                                       │
│    Parameters:    O(d²)          — weight matrices           │
│    Activations:   O(n · d)       — hidden states for BPTT   │
│    Total:         O(d² + n·d)                               │
│                                                             │
│  Transformer:                                               │
│    Parameters:    O(L · d²)      — L layers of FFN + attn   │
│    Activations:   O(L · n² + L · n · d)                     │
│                   ↑ attention matrices  ↑ FFN activations    │
│    Total:         O(L·d² + L·n²·d)                          │
│                                                             │
│  Critical difference: Transformer has O(n²) attention maps  │
│                                                             │
│  Example (n=1000, d=512, L=6):                              │
│    RNN:         ~0.5M + 0.5M = ~1M floats                   │
│    Transformer: ~1.5M + 6M + 3M = ~10.5M floats             │
│                                                             │
│  Example (n=10000, d=512, L=6):                             │
│    RNN:         ~0.5M + 5M = ~5.5M floats                   │
│    Transformer: ~1.5M + 600M + 30M = ~631M floats!!        │
│                 ↑ O(n²) dominates for long sequences        │
└─────────────────────────────────────────────────────────────┘
```

### Inference Memory

```
Autoregressive generation (producing one token at a time):

RNN:
  Store: current hidden state h_t ∈ ℝ^d
  Memory: O(d) — CONSTANT regardless of sequence length
  
Transformer (with KV cache):
  Store: all previous keys & values for all layers
  K cache: [L layers × n positions × d dimensions]
  V cache: [L layers × n positions × d dimensions]  
  Memory: O(L · n · d) — GROWS with each generated token

For long generation (n = 10000, L = 32, d = 4096):
  RNN:         4096 floats = 16 KB
  Transformer: 32 × 10000 × 4096 × 2 = 2.6 billion floats = 10 GB!
```

---

## 📊 Comprehensive Comparison Table

| Dimension | RNN (LSTM/GRU) | Transformer |
|-----------|----------------|-------------|
| **Architecture** | Recurrent, sequential | Attention-based, parallel |
| **Training parallelism** | ❌ O(n) sequential | ✅ O(1) sequential |
| **Training computation** | O(n · d²) | O(n² · d) |
| **Inference (per token)** | ✅ O(d²) constant | ❌ O(n · d) grows |
| **Inference memory** | ✅ O(d) constant | ❌ O(n · d) grows |
| **Max path length** | ❌ O(n) | ✅ O(1) |
| **Long-range deps** | ❌ Vanishing gradient | ✅ Direct attention |
| **Very long sequences** | ✅ Linear scaling | ❌ Quadratic scaling |
| **Streaming/online** | ✅ Natural | ❌ Needs full context |
| **Parameter count** | ✅ Fewer | ❌ More (but used better) |
| **Small data** | ✅ Strong inductive bias | ❌ Needs more data |
| **Large data** | ❌ Saturates | ✅ Scales well |
| **Hardware utilization** | ❌ Poor GPU usage | ✅ Excellent GPU usage |
| **Established theory** | ✅ Well understood | ⚠️ Still being studied |

---

## 🔬 Task-Specific Comparison

### Natural Language Processing

```
┌─────────────────────────┬─────────┬──────────────┐
│ Task                    │ Winner  │ Why          │
├─────────────────────────┼─────────┼──────────────┤
│ Machine Translation     │ Trans.  │ Parallelism  │
│ Language Modeling        │ Trans.  │ Long-range   │
│ Named Entity Recog.     │ Trans.  │ BERT context │
│ Sentiment Analysis      │ Trans.  │ Pre-training │
│ Text Generation         │ Trans.  │ GPT quality  │
│ Online chatbot (low lat)│ RNN     │ Streaming    │
│ On-device NLP           │ RNN     │ Small memory │
└─────────────────────────┴─────────┴──────────────┘
```

### Time Series and Signals

```
┌─────────────────────────┬─────────┬──────────────────┐
│ Task                    │ Winner  │ Why              │
├─────────────────────────┼─────────┼──────────────────┤
│ Short forecasting       │ Tie     │ Both work well   │
│ Long forecasting        │ Trans.  │ Long-range deps  │
│ Streaming sensors       │ RNN     │ Online processing│
│ Anomaly detection       │ RNN     │ Sequential nature│
│ Audio classification    │ Trans.  │ Parallelism      │
│ Real-time audio         │ RNN     │ Low latency      │
└─────────────────────────┴─────────┴──────────────────┘
```

### Reinforcement Learning

```
┌─────────────────────────┬─────────┬──────────────────┐
│ Task                    │ Winner  │ Why              │
├─────────────────────────┼─────────┼──────────────────┤
│ Game playing            │ RNN     │ Sequential       │
│ Robot control           │ RNN     │ Real-time        │
│ Offline RL              │ Trans.  │ Decision Trans.  │
│ POMDP (partial obs.)    │ RNN     │ Hidden state     │
└─────────────────────────┴─────────┴──────────────────┘
```

---

## 📐 Scaling Laws

### How Performance Scales with Compute

```
Performance
  │
  │                               ╱─── Transformer
  │                           ╱───
  │                       ╱───
  │                   ╱───
  │               ╱───
  │           ╱───      ╱─────────── RNN
  │       ╱───      ╱───
  │   ╱───      ╱───
  │╱───     ╱───
  │     ╱───
  │ ╱───
  │╱
  └──────────────────────────────────── Compute (FLOPs)
    10²⁰  10²¹  10²²  10²³  10²⁴

Key insight: Transformers scale BETTER with compute.
More compute → proportionally better performance.
RNNs plateau earlier due to sequential bottleneck.
```

---

## 💻 Code Comparison

### Same Task: Next-Token Prediction

```python
import torch
import torch.nn as nn

# ═══ RNN Approach ═══
class RNNModel(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_layers):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(embed_dim, hidden_dim, num_layers,
                           batch_first=True, dropout=0.3)
        self.fc = nn.Linear(hidden_dim, vocab_size)
    
    def forward(self, x, hidden=None):
        emb = self.embedding(x)
        out, hidden = self.lstm(emb, hidden)
        return self.fc(out), hidden

# ═══ Transformer Approach ═══
class TransformerModel(nn.Module):
    def __init__(self, vocab_size, embed_dim, num_heads, num_layers, dim_ff):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.pos_enc = nn.Embedding(512, embed_dim)  # learned positions
        encoder_layer = nn.TransformerEncoderLayer(
            d_model=embed_dim, nhead=num_heads,
            dim_feedforward=dim_ff, batch_first=True
        )
        self.transformer = nn.TransformerEncoder(encoder_layer, num_layers)
        self.fc = nn.Linear(embed_dim, vocab_size)
    
    def forward(self, x):
        seq_len = x.size(1)
        positions = torch.arange(seq_len, device=x.device).unsqueeze(0)
        emb = self.embedding(x) + self.pos_enc(positions)
        
        # Causal mask: prevent attending to future
        mask = nn.Transformer.generate_square_subsequent_mask(seq_len).to(x.device)
        out = self.transformer(emb, mask=mask)
        return self.fc(out)

# Compare parameter counts
rnn_model = RNNModel(10000, 256, 512, 2)
trans_model = TransformerModel(10000, 256, 8, 2, 1024)

rnn_params = sum(p.numel() for p in rnn_model.parameters())
trans_params = sum(p.numel() for p in trans_model.parameters())

print(f"RNN parameters:         {rnn_params:,}")
print(f"Transformer parameters: {trans_params:,}")
```

---

## 🌊 Hybrid Approaches

```
The best of both worlds:

1. Transformer-XL (2019)
   Transformer with recurrence for longer context
   Caches previous segments' hidden states

2. Compressive Transformer (2020)
   Compresses old memories to extend context window

3. Linear Attention (Katharopoulos 2020)
   O(n) attention approximation — like an RNN!
   Attention(Q,K,V) ≈ φ(Q) · (φ(K)^T · V)

4. RWKV (2023)
   "RNN with Transformer-level performance"
   Linear complexity, parallelizable training

5. State Space Models (Mamba, 2023)
   Selective state spaces — recurrent but parallelizable
   Competitive with Transformers on language tasks
   
┌──────────────────────────────────────────────────────┐
│  The future may not be "RNN vs Transformer"          │
│  but rather hybrid architectures that combine         │
│  the best properties of both.                        │
└──────────────────────────────────────────────────────┘
```

---

## 📋 Summary Table

| Comparison Axis | RNN Advantage | Transformer Advantage |
|----------------|---------------|----------------------|
| Training speed | — | Parallel processing |
| Inference cost | Constant per token | — |
| Long-range deps | — | O(1) path length |
| Memory (inference) | O(d) constant | — |
| Memory (training) | — | O(n²) attention maps |
| Small data | Strong inductive bias | — |
| Large data / scale | — | Better scaling laws |
| Streaming | Online processing | — |
| GPU utilization | — | Full parallelism |
| Research momentum | — | Active development |

---

## ❓ Revision Questions

1. **Explain why Transformers train faster than RNNs** despite often having more parameters and O(n²) attention complexity.

2. **For autoregressive generation of 10,000 tokens**, compare the total inference cost of an RNN vs a Transformer. Which is more efficient and why?

3. **A stream of sensor data arrives at 1000 Hz and needs real-time processing.** Would you choose an RNN or Transformer? Justify your answer.

4. **Transformers scale better with compute than RNNs.** What is the fundamental reason? How does this relate to the sequential bottleneck?

5. **State Space Models (like Mamba) combine properties of both architectures.** What specific properties do they take from each? Why is this promising?

6. **Given unlimited compute and data, should you ever use an RNN?** Describe at least two scenarios where RNNs remain the better choice.

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Transformer Preview](02-transformer-preview.md) |
| ⬆️ Unit | [10 Beyond RNNs](../README.md) |
| ➡️ Next | [When to Still Use RNNs](04-when-to-still-use-rnns.md) |
| 🏠 Home | [RNN Home](../README.md) |

---

*Last updated: 2024*
