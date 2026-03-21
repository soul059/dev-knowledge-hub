# Deep Learning Approaches for Text Classification

## Overview

Deep learning models learn features automatically from text, eliminating manual feature engineering. From CNNs that capture local patterns to LSTMs that model sequences to transformers that achieve state-of-the-art results, neural approaches dominate modern text classification.

---

## Architecture Comparison

```
┌──────────────────────────────────────────────────────────┐
│              TEXT CLASSIFICATION ARCHITECTURES            │
│                                                          │
│  CNN:        [Embedding] → [Conv1D + Pool] → [FC] → y  │
│  LSTM:       [Embedding] → [LSTM] → [last hidden] → y  │
│  BiLSTM:     [Embedding] → [BiLSTM] → [concat] → y    │
│  BERT:       [Tokenize] → [BERT] → [CLS token] → y    │
└──────────────────────────────────────────────────────────┘
```

---

## 1. TextCNN (Convolutional)

```python
import torch
import torch.nn as nn

class TextCNN(nn.Module):
    def __init__(self, vocab_size, embed_dim, num_classes, num_filters=100):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        
        # Multiple filter sizes capture different n-gram patterns
        self.convs = nn.ModuleList([
            nn.Conv1d(embed_dim, num_filters, kernel_size=k)
            for k in [3, 4, 5]  # trigram, 4-gram, 5-gram
        ])
        
        self.dropout = nn.Dropout(0.5)
        self.fc = nn.Linear(num_filters * 3, num_classes)
    
    def forward(self, x):
        x = self.embedding(x).permute(0, 2, 1)  # (batch, embed, seq_len)
        
        # Apply each conv + max-pool
        pooled = []
        for conv in self.convs:
            c = torch.relu(conv(x))                # (batch, filters, seq-k+1)
            p = torch.max(c, dim=2).values         # (batch, filters)
            pooled.append(p)
        
        cat = torch.cat(pooled, dim=1)             # (batch, filters*3)
        return self.fc(self.dropout(cat))
```

**Strengths:** Fast, captures local n-gram patterns, good for short texts.

---

## 2. LSTM Classifier

```python
class LSTMClassifier(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_classes):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(embed_dim, hidden_dim, batch_first=True,
                           bidirectional=True, num_layers=2, dropout=0.3)
        self.fc = nn.Linear(hidden_dim * 2, num_classes)  # *2 for bidirectional
        self.dropout = nn.Dropout(0.5)
    
    def forward(self, x):
        embedded = self.embedding(x)               # (batch, seq, embed)
        output, (hidden, cell) = self.lstm(embedded)
        
        # Concatenate last hidden states from both directions
        hidden_cat = torch.cat((hidden[-2], hidden[-1]), dim=1)
        return self.fc(self.dropout(hidden_cat))
```

**Strengths:** Captures long-range dependencies, handles variable-length sequences.

---

## 3. BERT Fine-tuning (State-of-the-Art)

```python
from transformers import BertTokenizer, BertForSequenceClassification
from transformers import Trainer, TrainingArguments

# Load pre-trained BERT
tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertForSequenceClassification.from_pretrained(
    'bert-base-uncased', num_labels=3
)

# Tokenize dataset
def tokenize_function(examples):
    return tokenizer(examples['text'], padding='max_length', 
                     truncation=True, max_length=128)

# Training
training_args = TrainingArguments(
    output_dir='./results',
    num_train_epochs=3,
    per_device_train_batch_size=16,
    learning_rate=2e-5,
    weight_decay=0.01,
    evaluation_strategy='epoch'
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_train,
    eval_dataset=tokenized_test
)
trainer.train()
```

**Strengths:** Best accuracy, understands context deeply, pre-trained knowledge.

---

## Architecture Comparison

| Model | Parameters | Training Speed | Accuracy | Context Window | Best For |
|-------|:-:|:-:|:-:|:-:|---|
| TextCNN | ~1M | ⚡ Fast | Good | Local (3-5 words) | Short texts, fast inference |
| LSTM | ~5M | 🐢 Medium | Good | Full sequence | Sequential patterns |
| BiLSTM | ~10M | 🐢 Medium | Better | Bidirectional | When context from both sides matters |
| BERT-base | 110M | 🐌 Slow | Best | 512 tokens | When accuracy is priority |
| DistilBERT | 66M | 🐢 Medium | Near-best | 512 tokens | Best accuracy/speed trade-off |

---

## Choosing the Right Architecture

```
Dataset size < 1K?     → TF-IDF + Logistic Regression
Dataset size 1K-10K?   → TextCNN or fine-tune DistilBERT
Dataset size 10K-100K? → Fine-tune BERT
Dataset size > 100K?   → Fine-tune BERT or train LSTM from scratch
Need fast inference?   → TextCNN or DistilBERT
Need best accuracy?    → BERT / RoBERTa fine-tuned
```

---

## Revision Questions

1. **How does TextCNN use different filter sizes to capture n-gram patterns?**
2. **Why is bidirectional LSTM better than unidirectional for classification?**
3. **What makes BERT fine-tuning so effective for text classification?**
4. **When would you choose TextCNN over BERT?**
5. **What learning rate should you use when fine-tuning BERT, and why?**

---

[Previous: 03-traditional-ml.md](03-traditional-ml.md) | [Next: 05-multi-label.md](05-multi-label.md)
