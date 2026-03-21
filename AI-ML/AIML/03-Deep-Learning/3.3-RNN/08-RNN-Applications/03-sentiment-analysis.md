# Sentiment Analysis

> **Unit 8, Chapter 3** — Many-to-one RNN for sentiment classification, bidirectional LSTM pipeline, and practical PyTorch implementation.

---

## 📋 Overview

**Sentiment analysis** classifies text by its emotional tone — typically positive, negative, or neutral. It's a classic many-to-one RNN task where the entire sequence is read and a single classification is produced from the final hidden state.

---

## 🏗️ Architecture

```
   "This"  "movie" "was"  "really" "great" "!"
     │       │      │       │       │      │
     ▼       ▼      ▼       ▼       ▼      ▼
   ┌────┐  ┌────┐  ┌────┐  ┌────┐  ┌────┐  ┌────┐
   │Embed│  │Embed│  │Embed│  │Embed│  │Embed│  │Embed│
   └──┬─┘  └──┬─┘  └──┬─┘  └──┬─┘  └──┬─┘  └──┬─┘
      │       │      │       │       │      │
      ▼       ▼      ▼       ▼       ▼      ▼
   ┌────┐  ┌────┐  ┌────┐  ┌────┐  ┌────┐  ┌────┐
   │h→₁ │─▶│h→₂ │─▶│h→₃ │─▶│h→₄ │─▶│h→₅ │─▶│h→₆ │  Forward
   │h←₁ │◀─│h←₂ │◀─│h←₃ │◀─│h←₄ │◀─│h←₅ │◀─│h←₆ │  Backward
   └────┘  └────┘  └────┘  └────┘  └────┘  └────┘
                                                │
                                     ┌──────────┘
                                     ▼
                              [h→₆ ; h←₁]   ← Concatenate
                                     │
                                     ▼
                              ┌──────────┐
                              │ FC + σ   │
                              └────┬─────┘
                                   ▼
                              Positive ✓
```

---

## 💻 Complete PyTorch Pipeline

```python
import torch
import torch.nn as nn

class SentimentBiLSTM(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, output_dim,
                 n_layers=2, dropout=0.5, pad_idx=0):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=pad_idx)
        self.lstm = nn.LSTM(embed_dim, hidden_dim, num_layers=n_layers,
                            bidirectional=True, dropout=dropout, batch_first=True)
        self.fc = nn.Linear(hidden_dim * 2, output_dim)  # *2 for bidirectional
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, text, text_lengths=None):
        """
        text: (batch, seq_len) — padded token indices
        text_lengths: (batch,) — actual lengths (for packing)
        """
        embedded = self.dropout(self.embedding(text))
        
        if text_lengths is not None:
            packed = nn.utils.rnn.pack_padded_sequence(
                embedded, text_lengths.cpu(), batch_first=True, enforce_sorted=False
            )
            output, (hidden, _) = self.lstm(packed)
        else:
            output, (hidden, _) = self.lstm(embedded)
        
        # Concatenate final forward and backward hidden states
        hidden_fwd = hidden[-2]  # (batch, hidden)
        hidden_bwd = hidden[-1]  # (batch, hidden)
        combined = torch.cat([hidden_fwd, hidden_bwd], dim=1)
        
        return self.fc(self.dropout(combined))

# Build model
model = SentimentBiLSTM(
    vocab_size=25000,
    embed_dim=100,
    hidden_dim=256,
    output_dim=1,  # Binary sentiment (use 2+ for multi-class)
    n_layers=2,
    dropout=0.5
)

# Training
criterion = nn.BCEWithLogitsLoss()
optimizer = torch.optim.Adam(model.parameters())

# Simulated batch
text = torch.randint(1, 25000, (32, 100))
labels = torch.randint(0, 2, (32,)).float()

logits = model(text).squeeze(1)
loss = criterion(logits, labels)
predictions = (torch.sigmoid(logits) > 0.5).float()
accuracy = (predictions == labels).float().mean()

print(f"Loss: {loss.item():.4f}, Accuracy: {accuracy.item():.4f}")
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Task Type | Many-to-one (sequence → single label) |
| Architecture | BiLSTM + FC layer |
| Output | Binary: BCEWithLogitsLoss; Multi: CrossEntropyLoss |
| Key Feature | Bidirectional captures full context |
| Pooling | Final hidden state, or mean/max pooling over outputs |
| Pre-trained Embeddings | GloVe, Word2Vec improve performance |

---

## ❓ Revision Questions

1. **Why is BiLSTM better than unidirectional LSTM for sentiment analysis?**
2. **What is the advantage of using packed sequences for variable-length inputs?**
3. **How would you modify this model for 5-star rating prediction?**
4. **Why use BCEWithLogitsLoss instead of BCELoss?**
5. **Would mean pooling over all hidden states work better than using only the final state?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Text Generation](02-text-generation.md) |
| ➡️ Next | [Named Entity Recognition](04-named-entity-recognition.md) |
| ⬆️ Unit Overview | [README](../README.md) |
