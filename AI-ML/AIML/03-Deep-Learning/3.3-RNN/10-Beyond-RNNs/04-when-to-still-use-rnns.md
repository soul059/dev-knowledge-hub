# When to Still Use RNNs

> **Unit 10, Chapter 4** — Practical scenarios where RNNs remain the best choice: streaming, low-latency, edge devices, small data, and sequential decision-making.

---

## 📋 Overview

Despite the Transformer revolution, RNNs are **not obsolete**. There are specific — and important — scenarios where RNNs outperform or are more practical than Transformers. Understanding when to use each architecture is a crucial engineering skill. This chapter covers the niches where RNNs shine and provides decision frameworks for architecture selection.

---

## 🌊 Scenario 1: Online / Streaming Processing

### The Problem

```
Streaming data arrives continuously — you must process each element
as it arrives, WITHOUT seeing future data:

  Sensor: ──→ x₁ ──→ x₂ ──→ x₃ ──→ x₄ ──→ ...
               │       │       │       │
               ▼       ▼       ▼       ▼
            Process  Process  Process  Process
            NOW      NOW      NOW      NOW

Examples:
  • Real-time audio processing
  • Live video analysis
  • Network intrusion detection
  • Online financial trading
  • IoT sensor monitoring
```

### Why RNNs Win

```
RNN (streaming-friendly):
  Time t=1: x₁ → [RNN(h₀)] → h₁ → output₁        Cost: O(d²)
  Time t=2: x₂ → [RNN(h₁)] → h₂ → output₂        Cost: O(d²)
  Time t=3: x₃ → [RNN(h₂)] → h₃ → output₃        Cost: O(d²)
  
  Total state: just h_t (fixed size)
  Latency: CONSTANT per time step ← ideal for streaming

Transformer (problematic for streaming):
  Time t=1: [x₁]         → attend → output₁        Cost: O(d)
  Time t=2: [x₁, x₂]    → attend → output₂        Cost: O(2d)
  Time t=3: [x₁, x₂, x₃]→ attend → output₃        Cost: O(3d)
  Time t=n: [x₁...x_n]   → attend → output_n       Cost: O(nd) 
  
  Must store ALL previous tokens (KV cache grows)
  Latency: INCREASES with each step ← bad for streaming
```

### Practical Example: Real-Time Audio

```python
import torch
import torch.nn as nn

class StreamingRNN(nn.Module):
    """Process audio samples one at a time with constant latency."""
    
    def __init__(self, input_size, hidden_size, output_size):
        super().__init__()
        self.gru = nn.GRUCell(input_size, hidden_size)
        self.fc = nn.Linear(hidden_size, output_size)
        self.hidden_size = hidden_size
    
    def process_sample(self, x_t, h_prev):
        """
        Process single time step — called for each incoming sample.
        Constant time and memory regardless of history length.
        """
        h = self.gru(x_t, h_prev)
        output = self.fc(h)
        return output, h
    
    def stream(self, data_generator):
        """Process infinite stream of data."""
        h = torch.zeros(1, self.hidden_size)
        
        for x_t in data_generator:
            x_t = x_t.unsqueeze(0)  # [1, input_size]
            output, h = self.process_sample(x_t, h)
            yield output  # Emit result immediately

# Usage: constant latency regardless of how long we've been streaming
model = StreamingRNN(input_size=80, hidden_size=256, output_size=40)

import time
h = torch.zeros(1, 256)

for step in range(10000):
    x = torch.randn(1, 80)
    start = time.time()
    out, h = model.process_sample(x, h)
    elapsed = time.time() - start
    
    if step % 2000 == 0:
        print(f"Step {step}: latency = {elapsed*1000:.2f} ms")
        # Latency stays constant: ~0.1 ms for all steps
```

---

## ⚡ Scenario 2: Low-Latency Requirements

### Latency Comparison

```
Processing a 100-token sequence:

  RNN (already processed 99 tokens, need token 100):
    Latency = 1 × O(d²)  ≈ 0.1 ms
    
  Transformer (need to attend to all 100 tokens):
    Latency = O(100 × d)  ≈ 1-5 ms (with KV cache)
    Without cache: O(100² × d) ≈ 50 ms

Applications with strict latency budgets:
  ┌──────────────────────┬────────────────┬──────────────┐
  │ Application          │ Latency Budget │ Best Choice  │
  ├──────────────────────┼────────────────┼──────────────┤
  │ Voice assistant      │ < 50 ms        │ Depends      │
  │ Game AI              │ < 16 ms        │ RNN          │
  │ Trading system       │ < 1 ms         │ RNN          │
  │ Robotic control      │ < 10 ms        │ RNN          │
  │ Real-time captioning │ < 100 ms       │ RNN or Trans │
  └──────────────────────┴────────────────┴──────────────┘
```

---

## 📱 Scenario 3: Edge Devices and Resource Constraints

### Memory Footprint

```
On-device deployment (e.g., smartphone, microcontroller):

  RNN inference state:
    Hidden state: d floats = 256 × 4 bytes = 1 KB
    Parameters: ~500 KB for small model
    Total RAM: ~501 KB ← fits on microcontroller!

  Transformer inference:
    KV cache (100 tokens): L × 2 × 100 × d floats
    = 6 × 2 × 100 × 256 × 4 bytes = 1.2 MB (and growing!)
    Parameters: ~2 MB for comparable model
    Total RAM: ~3.2 MB ← needs smartphone

  For 1000 tokens:
    RNN: still 501 KB
    Transformer KV cache: 12 MB ← memory grows linearly!
```

### Deployment Decision Matrix

```
┌───────────────────────────────────────────────────────────┐
│  Device Constraints → Architecture Choice                 │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  Microcontroller (< 1 MB RAM):                            │
│    → Small RNN (GRU, hidden=64-128)                       │
│                                                           │
│  Mobile phone (1-4 GB RAM):                               │
│    → Medium RNN or small Transformer                      │
│    → LSTM with 256-512 hidden                             │
│                                                           │
│  Edge GPU (4-8 GB):                                       │
│    → Transformer (small/medium)                           │
│    → Distilled models                                     │
│                                                           │
│  Cloud GPU (16-80 GB):                                    │
│    → Large Transformer (preferred)                        │
│    → Full-size models                                     │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

---

## 📉 Scenario 4: Small Datasets

### Inductive Bias Advantage

```
RNNs have strong inductive bias for sequences:
  • Recurrence forces sequential processing
  • Parameter sharing across time steps
  • Hidden state naturally accumulates information
  
  With 1,000 training examples: RNN can learn reasonable patterns
  
Transformers have weak inductive bias:
  • Must learn positional relationships from data
  • Must learn sequential processing from data
  • More parameters to fit → need more data
  
  With 1,000 training examples: Transformer likely overfits
```

### Empirical Guidelines

```
Dataset size vs. architecture performance:

  Performance
     │
     │         ╱─── Transformer
     │        ╱
     │       ╱
     │  ────╱─────────── RNN
     │ ╱  ╱
     │╱  ╱
     │  ╱
     │ ╱
     │╱
     └────────────────────────── Dataset size
      100  1K   10K  100K  1M  10M

  Crossover point: typically ~10K-50K examples
  Below crossover: RNN advantage (less overfitting)
  Above crossover: Transformer advantage (more capacity)
  
  Exception: If pre-trained Transformer available (e.g., fine-tune BERT),
  Transformer wins even with small data due to transfer learning.
```

---

## 🤖 Scenario 5: Sequential Decision-Making

### Reinforcement Learning

```
In RL, the agent must:
1. Observe state at time t
2. Choose action immediately  
3. Move to next state
4. Repeat

RNN as policy network:
  state_t → [RNN(h_{t-1})] → h_t → action_t
  
  ✅ Constant-time decision
  ✅ Naturally maintains belief state (POMDP)
  ✅ Handles partial observability
  ✅ Small memory footprint

Transformer as policy:
  [s₁, s₂, ..., s_t] → [Transformer] → action_t
  
  ✅ Can attend to any past state
  ❌ Growing context window
  ❌ Expensive for long episodes
  ❌ Must re-process entire history
```

### Partially Observable Environments (POMDPs)

```
Agent cannot see full state — must infer from history:

Example: Robot in a dark room
  Observations: [bump, silence, bump, echo, silence, ...]
  True state: position in room (unknown)

RNN naturally maintains a BELIEF STATE:
  h_t ≈ P(true_state | o₁, o₂, ..., o_t)
  
  The hidden state compresses history into a fixed-size
  sufficient statistic. This is exactly what POMDPs need!

Transformer approach: 
  Attend to all past observations → more powerful
  But: episode length can be 10,000+ steps
       → Quadratic attention cost is prohibitive
```

---

## 🔧 Scenario 6: Specific Technical Advantages

### Variable-Length Processing without Padding

```
RNNs process sequences naturally without padding:

  Sequence 1 (length 5):  x₁→x₂→x₃→x₄→x₅ → output
  Sequence 2 (length 3):  x₁→x₂→x₃ → output
  Sequence 3 (length 8):  x₁→x₂→x₃→x₄→x₅→x₆→x₇→x₈ → output
  
  No wasted computation on padding tokens.

Transformers require padding/masking:
  Sequence 1:  x₁ x₂ x₃ x₄ x₅ [PAD] [PAD] [PAD]
  Sequence 2:  x₁ x₂ x₃ [PAD] [PAD] [PAD] [PAD] [PAD]
  Sequence 3:  x₁ x₂ x₃ x₄ x₅ x₆ x₇ x₈
  
  Padded to max length → wasted computation on padding.
  Attention mask needed to ignore padding.
```

### Infinite/Unknown Length Sequences

```
When sequence length is unknown or potentially infinite:

  Data stream:  ... → x_{t-2} → x_{t-1} → x_t → x_{t+1} → ...
                                                    ???
  
  RNN: Process indefinitely with fixed memory
  Transformer: Cannot handle — needs fixed context window
               (Sliding window or Transformer-XL needed)
```

---

## 🧭 Decision Framework

### Architecture Selection Flowchart

```
                    START
                      │
                      ▼
              Is data sequential?
              ╱              ╲
            No                Yes
            │                  │
            ▼                  ▼
        Use CNN/MLP     Is streaming/online
                        processing needed?
                        ╱              ╲
                      Yes               No
                       │                 │
                       ▼                 ▼
                   Use RNN         Dataset size?
                                   ╱          ╲
                              < 10K           > 10K
                                │               │
                                ▼               ▼
                          Pre-trained      Is sequence
                          model avail?     very long (>10K)?
                          ╱        ╲       ╱            ╲
                        Yes        No    Yes             No
                         │          │      │              │
                         ▼          ▼      ▼              ▼
                     Fine-tune   Use    RNN or         Use
                    Transformer  RNN    SSM (Mamba)  Transformer
```

### Quick Decision Table

| Constraint | Use RNN | Use Transformer |
|-----------|---------|-----------------|
| Streaming data | ✅ | ❌ |
| Latency < 5ms | ✅ | ❌ |
| RAM < 1 MB | ✅ | ❌ |
| Dataset < 5K samples | ✅ (unless pre-trained available) | ❌ |
| Sequence > 100K | ✅ | ❌ (or use linear attention) |
| Offline batch processing | ❌ | ✅ |
| Pre-trained model available | ❌ | ✅ |
| Need best accuracy | ❌ | ✅ (with enough data) |
| GPU cluster available | ❌ | ✅ |

---

## 💻 Practical Example: Choosing for a Real Project

### Project: Predictive Text on Mobile Keyboard

```python
# Why RNN wins here:
# 1. Real-time: must predict as user types (< 50ms latency)
# 2. On-device: runs on phone (limited memory)
# 3. Streaming: processes character by character
# 4. Privacy: no cloud, all local processing

class MobileKeyboardPredictor(nn.Module):
    """Tiny GRU for on-device next-character prediction."""
    
    def __init__(self, vocab_size=128, embed_size=32, hidden_size=128):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_size)
        self.gru = nn.GRUCell(embed_size, hidden_size)
        self.fc = nn.Linear(hidden_size, vocab_size)
        self.hidden_size = hidden_size
    
    def predict_next(self, char_idx, hidden):
        """Predict next character given current character and state."""
        emb = self.embedding(char_idx)  # [1, 32]
        hidden = self.gru(emb, hidden)  # [1, 128]
        logits = self.fc(hidden)        # [1, 128]
        return logits, hidden

# Model size analysis
model = MobileKeyboardPredictor()
param_count = sum(p.numel() for p in model.parameters())
model_size_kb = param_count * 4 / 1024

print(f"Parameters: {param_count:,}")    # ~70K parameters
print(f"Model size: {model_size_kb:.0f} KB")  # ~273 KB
print(f"Hidden state: {128 * 4} bytes = 512 bytes")
# Total runtime memory: < 300 KB — fits on ANY device!
```

### Project: Document Classification (Batch)

```python
# Why Transformer wins here:
# 1. Offline: documents processed in batch
# 2. Pre-trained: BERT/RoBERTa available
# 3. Accuracy critical: legal/medical documents
# 4. Cloud GPU available

from transformers import AutoModelForSequenceClassification, AutoTokenizer

model_name = "bert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(
    model_name, num_labels=5
)
# Fine-tune on your data → much better accuracy than RNN
```

---

## 🔮 The Future: Convergence

```
┌──────────────────────────────────────────────────────────────┐
│  The lines are blurring:                                     │
│                                                              │
│  • State Space Models (Mamba): RNN-like but parallelizable   │
│  • RWKV: "RNN" that trains like a Transformer                │
│  • Linear Transformers: O(n) attention ≈ RNN                 │
│  • RetNet: Retention mechanism — hybrid of both              │
│  • xLSTM (2024): Modern LSTM with exponential gating         │
│                                                              │
│  Future architectures will likely combine:                   │
│    ✅ Transformer's training parallelism                      │
│    ✅ RNN's efficient inference                               │
│    ✅ Long-range dependency handling                          │
│    ✅ Linear complexity                                       │
│                                                              │
│  Understanding RNNs is NOT obsolete —                        │
│  the core concepts (hidden state, gating, sequential         │
│  processing) are fundamental to these new architectures.    │
└──────────────────────────────────────────────────────────────┘
```

---

## 📋 Summary Table

| Scenario | Why RNN Wins |
|----------|-------------|
| Streaming/online data | Constant-time per step, no growing context |
| Ultra-low latency | O(d²) per step vs O(nd) for Transformer |
| Edge/IoT devices | Fixed memory footprint, tiny models |
| Small datasets | Strong inductive bias, less overfitting |
| Sequential RL/POMDPs | Natural belief state, constant decision time |
| Very long sequences | Linear vs quadratic scaling |
| Unknown-length sequences | No context window limitation |

---

## ❓ Revision Questions

1. **A factory has 1000 sensors streaming data at 100 Hz, and needs anomaly detection within 5ms.** Justify whether you'd use an RNN or Transformer.

2. **Explain why RNNs have stronger inductive bias for sequences than Transformers.** How does this help with small datasets?

3. **For a POMDP in reinforcement learning, why is an RNN's hidden state a natural choice** for maintaining a belief state? Could a Transformer do this too?

4. **You need to deploy a language model on a smartwatch with 256 KB RAM.** Design an RNN-based solution and explain why a Transformer wouldn't fit.

5. **Describe how State Space Models (like Mamba) combine advantages of both RNNs and Transformers.** What specific property from each do they inherit?

6. **Despite Transformers dominating benchmarks, RNNs remain widely deployed in production.** Give three concrete real-world systems that still use RNNs and explain why.

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [RNN vs Transformer](03-rnn-vs-transformer.md) |
| ⬆️ Unit | [10 Beyond RNNs](../README.md) |
| 🏠 Home | [RNN Home](../README.md) |
| 🔙 Subject Start | [Unit 1: Sequence Modeling](../01-Sequence-Modeling/01-why-sequence-models.md) |

---

> **Congratulations!** 🎉 You've completed the full RNN subject. You now understand the evolution from basic recurrence to modern architectures, and can make informed decisions about when to use each approach.

---

*Last updated: 2024*
