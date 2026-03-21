# Named Entity Recognition

> **Unit 8, Chapter 4** — Sequence labeling with BIO tagging, BiLSTM-CRF architecture, and practical NER pipeline.

---

## 📋 Overview

**Named Entity Recognition (NER)** identifies and classifies named entities (persons, organizations, locations, etc.) in text. It's a many-to-many (equal length) sequence labeling task where each token receives a label.

---

## 📊 BIO Tagging Scheme

```
Text:    "John Smith works at Apple in New York City"
Tokens:  John  Smith works at  Apple in  New  York City
Tags:    B-PER I-PER O     O   B-ORG O   B-LOC I-LOC I-LOC

B = Beginning of entity
I = Inside (continuation) of entity
O = Outside (not an entity)

Entity types: PER (person), ORG (organization), LOC (location),
              DATE, MISC, etc.
```

---

## 🏗️ BiLSTM-CRF Architecture

```
           John   Smith  works   at    Apple   in    New    York   City
            │      │      │      │      │      │      │      │      │
            ▼      ▼      ▼      ▼      ▼      ▼      ▼      ▼      ▼
         ┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐
BiLSTM:  │h→ h←││h→ h←││h→ h←││h→ h←││h→ h←││h→ h←││h→ h←││h→ h←││h→ h←│
         └──┬──┘└──┬──┘└──┬──┘└──┬──┘└──┬──┘└──┬──┘└──┬──┘└──┬──┘└──┬──┘
            │      │      │      │      │      │      │      │      │
            ▼      ▼      ▼      ▼      ▼      ▼      ▼      ▼      ▼
FC:       ┌──┐   ┌──┐   ┌──┐   ┌──┐   ┌──┐   ┌──┐   ┌──┐   ┌──┐   ┌──┐
(emissions)│  │   │  │   │  │   │  │   │  │   │  │   │  │   │  │   │  │
          └──┘   └──┘   └──┘   └──┘   └──┘   └──┘   └──┘   └──┘   └──┘
            │      │      │      │      │      │      │      │      │
            ▼      ▼      ▼      ▼      ▼      ▼      ▼      ▼      ▼
CRF:      B-PER  I-PER   O      O    B-ORG    O    B-LOC  I-LOC  I-LOC

CRF adds transition constraints:
  • B-PER → I-PER: allowed ✓
  • B-PER → I-LOC: not allowed ✗
  • O → I-PER: not allowed ✗ (must start with B)
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn

class BiLSTMTagger(nn.Module):
    """Simple BiLSTM sequence tagger (without CRF)"""
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_tags, pad_idx=0):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=pad_idx)
        self.lstm = nn.LSTM(embed_dim, hidden_dim, num_layers=2,
                            bidirectional=True, batch_first=True, dropout=0.3)
        self.fc = nn.Linear(hidden_dim * 2, num_tags)
        self.dropout = nn.Dropout(0.3)
    
    def forward(self, x):
        """
        x: (batch, seq_len)
        Returns: (batch, seq_len, num_tags) — tag logits per token
        """
        embedded = self.dropout(self.embedding(x))
        output, _ = self.lstm(embedded)
        logits = self.fc(self.dropout(output))
        return logits

# BIO tags for NER
tag_to_idx = {'O': 0, 'B-PER': 1, 'I-PER': 2, 'B-ORG': 3, 
              'I-ORG': 4, 'B-LOC': 5, 'I-LOC': 6, '<PAD>': 7}
num_tags = len(tag_to_idx)

model = BiLSTMTagger(vocab_size=30000, embed_dim=100, hidden_dim=128, num_tags=num_tags)

# Training
text = torch.randint(1, 30000, (16, 40))
tags = torch.randint(0, num_tags, (16, 40))

logits = model(text)
loss = nn.CrossEntropyLoss(ignore_index=7)(logits.view(-1, num_tags), tags.view(-1))

predictions = logits.argmax(dim=-1)
print(f"Input: {text.shape} → Tags: {predictions.shape}")
print(f"Loss: {loss.item():.4f}")
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| NER | Identify and classify named entities in text |
| BIO Scheme | B=beginning, I=inside, O=outside tagging |
| Architecture | BiLSTM (+ optional CRF layer) |
| CRF Layer | Enforces valid tag transitions |
| Task Type | Many-to-many (equal length) sequence labeling |
| Evaluation | Entity-level F1 score (not token-level) |

---

## ❓ Revision Questions

1. **What is the BIO tagging scheme? Why do we need both B and I tags?**
2. **Why is BiLSTM preferred over unidirectional LSTM for NER?**
3. **What does the CRF layer add that a simple softmax per token doesn't?**
4. **Tag this sentence: "Barack Obama visited Google headquarters in Mountain View"**
5. **Why is entity-level F1 a better metric than token-level accuracy for NER?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Sentiment Analysis](03-sentiment-analysis.md) |
| ➡️ Next | [Machine Translation](05-machine-translation.md) |
| ⬆️ Unit Overview | [README](../README.md) |
