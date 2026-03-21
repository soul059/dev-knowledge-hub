# Encoder-Decoder Framework

> **Unit 7, Chapter 1** — Complete seq2seq pipeline: encoding variable-length input, decoding step-by-step, start/end tokens, and training procedure.

---

## 📋 Overview

The **sequence-to-sequence (seq2seq)** framework is a complete pipeline for transforming one sequence into another. This chapter covers the end-to-end pipeline including data preparation, special tokens, training procedure, and inference.

---

## 🏗️ Complete Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                  SEQ2SEQ PIPELINE                                │
│                                                                  │
│  1. PREPROCESSING                                               │
│     Source: "I love cats"                                        │
│     Target: "J'aime les chats"                                  │
│                                                                  │
│  2. TOKENIZE + ADD SPECIAL TOKENS                               │
│     Source: [<sos>, "I", "love", "cats", <eos>]                 │
│     Target: [<sos>, "J'", "aime", "les", "chats", <eos>]       │
│                                                                  │
│  3. NUMERICALIZE                                                │
│     Source: [1, 45, 892, 234, 2]                                │
│     Target: [1, 67, 423, 156, 789, 2]                          │
│                                                                  │
│  4. ENCODE                                                      │
│     [1, 45, 892, 234, 2] → Encoder → context (h, c)           │
│                                                                  │
│  5. DECODE (training with teacher forcing)                      │
│     Input:  [<sos>, "J'", "aime", "les", "chats"]              │
│     Target: ["J'", "aime", "les", "chats", <eos>]              │
│     (Input shifted right by 1 from target)                      │
│                                                                  │
│  6. COMPUTE LOSS                                                │
│     Cross-entropy between predictions and target                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔑 Special Tokens

```
┌──────────┬────────┬──────────────────────────────────────┐
│ Token    │ ID     │ Purpose                              │
├──────────┼────────┼──────────────────────────────────────┤
│ <pad>    │ 0      │ Padding for batching                 │
│ <sos>    │ 1      │ Start of sequence (decoder input)    │
│ <eos>    │ 2      │ End of sequence (stop signal)        │
│ <unk>    │ 3      │ Unknown/out-of-vocabulary word       │
└──────────┴────────┴──────────────────────────────────────┘

Training alignment:
   Decoder input:  <sos>  J'   aime  les   chats
   Decoder target: J'     aime les   chats <eos>
                   ─────  ──── ────  ───── ─────
                   Predict each next token from previous
```

---

## 🧮 Training vs Inference

```
TRAINING (teacher forcing):
   ┌─────────────────────────────────────────────┐
   │ Decoder input at t:  ground truth y_{t-1}   │
   │ Loss at t:           CE(prediction_t, y_t)  │
   │ All steps computed in parallel (packed)      │
   └─────────────────────────────────────────────┘

INFERENCE (autoregressive):
   ┌─────────────────────────────────────────────┐
   │ Decoder input at t:  model prediction ŷ_{t-1}│
   │ Stop when:           <eos> predicted or      │
   │                      max_length reached       │
   │ MUST be sequential (each step needs previous)│
   └─────────────────────────────────────────────┘

Inference loop:
   t=0: input = <sos>
   t=1: predict ŷ₁ = "J'"  → input = "J'"
   t=2: predict ŷ₂ = "aime" → input = "aime"
   ...
   t=k: predict ŷ_k = <eos> → STOP
```

---

## 💻 Complete PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.optim as optim
import random

class Encoder(nn.Module):
    def __init__(self, src_vocab_size, embed_dim, hidden_dim, n_layers, dropout):
        super().__init__()
        self.embedding = nn.Embedding(src_vocab_size, embed_dim, padding_idx=0)
        self.rnn = nn.LSTM(embed_dim, hidden_dim, n_layers,
                           dropout=dropout, batch_first=True)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, src):
        embedded = self.dropout(self.embedding(src))
        outputs, (hidden, cell) = self.rnn(embedded)
        return hidden, cell

class Decoder(nn.Module):
    def __init__(self, trg_vocab_size, embed_dim, hidden_dim, n_layers, dropout):
        super().__init__()
        self.embedding = nn.Embedding(trg_vocab_size, embed_dim, padding_idx=0)
        self.rnn = nn.LSTM(embed_dim, hidden_dim, n_layers,
                           dropout=dropout, batch_first=True)
        self.fc_out = nn.Linear(hidden_dim, trg_vocab_size)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, trg_token, hidden, cell):
        embedded = self.dropout(self.embedding(trg_token.unsqueeze(1)))
        output, (hidden, cell) = self.rnn(embedded, (hidden, cell))
        prediction = self.fc_out(output.squeeze(1))
        return prediction, hidden, cell

class Seq2Seq(nn.Module):
    def __init__(self, encoder, decoder, sos_idx=1, eos_idx=2):
        super().__init__()
        self.encoder = encoder
        self.decoder = decoder
        self.sos_idx = sos_idx
        self.eos_idx = eos_idx
    
    def forward(self, src, trg, teacher_forcing_ratio=0.5):
        batch_size, trg_len = trg.shape
        trg_vocab_size = self.decoder.fc_out.out_features
        
        outputs = torch.zeros(batch_size, trg_len - 1, trg_vocab_size)
        hidden, cell = self.encoder(src)
        
        input_token = trg[:, 0]  # <sos>
        
        for t in range(1, trg_len):
            prediction, hidden, cell = self.decoder(input_token, hidden, cell)
            outputs[:, t-1] = prediction
            
            if random.random() < teacher_forcing_ratio:
                input_token = trg[:, t]
            else:
                input_token = prediction.argmax(1)
        
        return outputs
    
    @torch.no_grad()
    def translate(self, src, max_len=50):
        self.eval()
        hidden, cell = self.encoder(src)
        
        input_token = torch.full((src.size(0),), self.sos_idx, dtype=torch.long)
        translations = []
        
        for _ in range(max_len):
            prediction, hidden, cell = self.decoder(input_token, hidden, cell)
            top_token = prediction.argmax(1)
            translations.append(top_token)
            
            if (top_token == self.eos_idx).all():
                break
            input_token = top_token
        
        return torch.stack(translations, dim=1)

# Build and train
enc = Encoder(8000, 256, 512, 2, 0.3)
dec = Decoder(10000, 256, 512, 2, 0.3)
model = Seq2Seq(enc, dec)

optimizer = optim.Adam(model.parameters(), lr=0.001)
criterion = nn.CrossEntropyLoss(ignore_index=0)  # Ignore padding

# Training step
src = torch.randint(1, 8000, (32, 20))
trg = torch.randint(1, 10000, (32, 25))

output = model(src, trg, teacher_forcing_ratio=0.5)
loss = criterion(output.view(-1, 10000), trg[:, 1:].reshape(-1))
print(f"Loss: {loss.item():.4f}")
```

---

## 📋 Summary Table

| Component | Description |
|-----------|-------------|
| Encoder | Reads source → context vector |
| Decoder | Generates target autoregressively |
| \<sos\> | Decoder start signal |
| \<eos\> | Decoder stop signal |
| Teacher Forcing | Ground truth as decoder input (training) |
| Autoregressive | Own predictions as input (inference) |
| Loss | Cross-entropy, ignore \<pad\> tokens |

---

## ❓ Revision Questions

1. **Describe the complete seq2seq pipeline from raw text to model prediction.**

2. **Why is the target sequence shifted by one position for decoder input vs decoder target?**

3. **How does the model know when to stop generating during inference?**

4. **Why does the loss function use `ignore_index=0` for the padding token?**

5. **What would happen if you used teacher forcing ratio = 1.0 during inference?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Teacher Forcing](../06-Advanced-RNN-Architectures/05-teacher-forcing.md) |
| ➡️ Next | [Applications](02-applications.md) |
| ⬆️ Unit Overview | [README](../README.md) |
