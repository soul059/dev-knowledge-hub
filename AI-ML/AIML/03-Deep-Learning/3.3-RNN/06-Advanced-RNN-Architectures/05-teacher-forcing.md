# Teacher Forcing

> **Unit 6, Chapter 5** — Using ground truth vs model predictions as next input during training, exposure bias, and scheduled sampling.

---

## 📋 Overview

**Teacher forcing** is a training strategy for sequence generation models where the ground truth output at time t-1 is fed as input at time t, rather than the model's own prediction. It dramatically speeds up training but introduces **exposure bias**.

---

## 📐 Teacher Forcing vs Free Running

```
TEACHER FORCING (training):
   Ground truth: "J'" "aime" "les" "chats"
                  │     │      │     │
                  ▼     ▼      ▼     ▼
   Decoder:     [s₁]→[s₂]→[s₃]→[s₄]
                  │     │      │     │
                  ▼     ▼      ▼     ▼
   Output:      "J'"  "aime" "les" "chats"

   Even if decoder predicts wrong at t=1, correct input
   is fed at t=2 → errors don't compound during training

FREE RUNNING (inference):
   Predictions:  "J'"  → used as input
                  │
                  ▼
   Decoder:     [s₁]→[s₂]→[s₃]→[s₄]
                         │
                   "aime" → used as input
                              │
                        "les" → used as input
                                   │
                             "chats" → output

   Each prediction is fed back → errors COMPOUND!
```

---

## ⚠️ Exposure Bias

```
┌──────────────────────────────────────────────────────────────┐
│                    EXPOSURE BIAS                              │
│                                                              │
│  Problem: During training, the decoder ALWAYS sees correct   │
│  previous tokens. During inference, it sees its OWN          │
│  (potentially wrong) predictions.                            │
│                                                              │
│  Training:  "The" "cat" "sat" → always correct context       │
│  Inference: "The" "dog" (error!) "ran" → error propagates    │
│                                                              │
│  The model never learns to RECOVER from its own mistakes!   │
│                                                              │
│  Training distribution ≠ Inference distribution              │
│  This mismatch is called EXPOSURE BIAS.                     │
│                                                              │
│  Effect: Small errors at early steps → catastrophic errors  │
│  at later steps (error accumulation / snowball effect)       │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 Scheduled Sampling

```
Solution: GRADUALLY transition from teacher forcing to free running

Epoch 1:   100% teacher forcing  (always ground truth)
Epoch 10:   80% teacher forcing  (mostly ground truth)
Epoch 30:   50% teacher forcing  (mixed)
Epoch 50:   20% teacher forcing  (mostly model predictions)
Epoch 100:   0% teacher forcing  (pure free running)

   ε (probability of using ground truth)
   1.0 │████████
       │        ████
       │            ████
   0.5 │                ████
       │                    ████
       │                        ████
   0.0 │                            ████████
       └──────────────────────────────────────
       0        25       50       75      100
                    Epoch

Common decay schedules:
  • Linear:      ε(k) = max(ε_min, ε₀ - k × decay_rate)
  • Exponential: ε(k) = ε₀ × 0.99^k
  • Inverse sigmoid: ε(k) = k / (k + exp(k/k₀))
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn
import random

class DecoderWithTeacherForcing(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.LSTM(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, vocab_size)
    
    def forward(self, target, hidden, cell, teacher_forcing_ratio=0.5):
        """
        target: (batch, trg_len) — ground truth sequence
        teacher_forcing_ratio: probability of using ground truth
        """
        batch_size = target.shape[0]
        trg_len = target.shape[1]
        vocab_size = self.fc.out_features
        
        outputs = torch.zeros(batch_size, trg_len, vocab_size)
        input_token = target[:, 0]  # <sos> token
        
        for t in range(1, trg_len):
            embedded = self.embedding(input_token).unsqueeze(1)
            output, (hidden, cell) = self.rnn(embedded, (hidden, cell))
            prediction = self.fc(output.squeeze(1))
            outputs[:, t] = prediction
            
            # Teacher forcing decision
            if random.random() < teacher_forcing_ratio:
                input_token = target[:, t]          # Ground truth
            else:
                input_token = prediction.argmax(1)  # Model prediction
        
        return outputs

class ScheduledSampling:
    """Decay teacher forcing ratio over training"""
    def __init__(self, initial_ratio=1.0, decay_rate=0.01, min_ratio=0.0):
        self.ratio = initial_ratio
        self.decay_rate = decay_rate
        self.min_ratio = min_ratio
    
    def step(self):
        self.ratio = max(self.min_ratio, self.ratio - self.decay_rate)
        return self.ratio
    
    def get_ratio(self):
        return self.ratio

# Training loop with scheduled sampling
scheduler = ScheduledSampling(initial_ratio=1.0, decay_rate=0.005)

for epoch in range(100):
    tf_ratio = scheduler.get_ratio()
    # ... training with teacher_forcing_ratio=tf_ratio ...
    scheduler.step()
    
    if epoch % 20 == 0:
        print(f"Epoch {epoch}: teacher_forcing_ratio = {tf_ratio:.3f}")
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Teacher Forcing | Use ground truth as decoder input during training |
| Free Running | Use model's own predictions (inference mode) |
| Exposure Bias | Train/test distribution mismatch from teacher forcing |
| Error Accumulation | Wrong predictions compound without teacher forcing |
| Scheduled Sampling | Gradually reduce teacher forcing ratio over training |
| Benefit of TF | Faster convergence, more stable early training |
| Drawback of TF | Model doesn't learn to recover from errors |

---

## ❓ Revision Questions

1. **What is teacher forcing? Why does it speed up training?**

2. **Explain exposure bias. Give a concrete example of how it causes problems at inference time.**

3. **How does scheduled sampling address exposure bias? Describe a typical schedule.**

4. **If teacher_forcing_ratio=0.7, what fraction of decoder inputs come from ground truth?**

5. **Why can't we just use teacher_forcing_ratio=0 from the start?**

6. **What alternative approaches to teacher forcing exist for training sequence generators?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Attention Mechanism Basics](04-attention-mechanism-basics.md) |
| ➡️ Next | [Encoder-Decoder Framework](../07-Sequence-to-Sequence/01-encoder-decoder-framework.md) |
| ⬆️ Unit Overview | [README](../README.md) |
