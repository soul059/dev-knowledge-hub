# Transfer Learning in NLP

## Overview

Transfer learning leverages knowledge from a model trained on a large general corpus and applies it to a specific downstream task. Instead of training from scratch, you start with a pre-trained model (BERT, GPT, etc.) and adapt it — dramatically reducing the data, compute, and time needed. Transfer learning is the single biggest paradigm shift in modern NLP.

---

## The Transfer Learning Pipeline

```
┌──────────────────────────────────────────────────────────┐
│                TRANSFER LEARNING IN NLP                   │
│                                                          │
│  Phase 1: Pre-training (expensive, done once)            │
│  ┌────────────────────────────────────────────┐          │
│  │ Massive corpus (Wikipedia, Books, Web)     │          │
│  │ Self-supervised objective (MLM, CLM)       │          │
│  │ Billions of tokens, days on GPU clusters   │          │
│  └────────────────────┬───────────────────────┘          │
│                       ▼                                   │
│  Phase 2: Fine-tuning (cheap, per task)                  │
│  ┌────────────────────────────────────────────┐          │
│  │ Your small labeled dataset (1K–100K)       │          │
│  │ Task-specific head (classifier, tagger)    │          │
│  │ Minutes to hours on single GPU             │          │
│  └────────────────────────────────────────────┘          │
└──────────────────────────────────────────────────────────┘
```

---

## Two Approaches

### Feature Extraction (Frozen)

```python
from transformers import AutoModel, AutoTokenizer
import torch

# Use BERT as frozen feature extractor
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModel.from_pretrained("bert-base-uncased")

# Freeze all BERT weights
for param in model.parameters():
    param.requires_grad = False

# Extract features
text = "Transfer learning is powerful"
inputs = tokenizer(text, return_tensors="pt")
with torch.no_grad():
    outputs = model(**inputs)
    features = outputs.last_hidden_state[:, 0, :]  # [CLS] vector (768-dim)

# Train a simple classifier on extracted features
# Only the classifier head is trained
```

### Fine-tuning (Unfrozen)

```python
from transformers import BertForSequenceClassification, Trainer, TrainingArguments

model = BertForSequenceClassification.from_pretrained(
    "bert-base-uncased", num_labels=3
)

# ALL parameters are trainable (updated with small learning rate)
training_args = TrainingArguments(
    output_dir="./results",
    learning_rate=2e-5,      # Much smaller than training from scratch
    num_train_epochs=3,
    per_device_train_batch_size=16,
)

trainer = Trainer(model=model, args=training_args, 
                  train_dataset=train_data, eval_dataset=eval_data)
trainer.train()
```

---

## Feature Extraction vs Fine-tuning

| Aspect | Feature Extraction | Fine-tuning |
|--------|:-:|:-:|
| Pre-trained weights | Frozen | Updated |
| Training speed | ⚡ Fast | 🐢 Slower |
| Data needed | Very little | More (but still small) |
| Performance | Good | Best |
| Risk of overfitting | Low | Higher (small data) |
| When to use | <1K samples, quick prototype | 1K+ samples, best accuracy |

---

## Domain Adaptation

```
General pre-trained model (Wikipedia/Books)
         │
         ├─ Direct fine-tuning → works OK for general tasks
         │
         └─ Domain-adaptive pre-training (DAPT) → then fine-tune
              │
              ├─ Continue pre-training on domain text
              │   (medical papers, legal docs, code)
              │
              └─ Then fine-tune on labeled task data
                  → Much better for specialized domains!

Examples:
  BERT → BioBERT (biomedical)
  BERT → SciBERT (scientific)
  BERT → LegalBERT (legal)
  BERT → FinBERT (financial)
  BERT → CodeBERT (programming)
```

---

## Practical Guidelines

```
Dataset size          Recommended approach
──────────────────────────────────────────────
< 100 samples         Zero/few-shot with LLM (GPT-4, Claude)
100–1K samples         Feature extraction (freeze BERT)
1K–10K samples         Fine-tune with low LR (2e-5)
10K–100K samples       Fine-tune, maybe domain-adapt first
> 100K samples         Fine-tune or train from scratch
```

---

## Layer Freezing Strategy

```python
# Gradual unfreezing: unfreeze top layers first
model = BertForSequenceClassification.from_pretrained("bert-base-uncased", num_labels=2)

# Freeze all layers
for param in model.bert.parameters():
    param.requires_grad = False

# Unfreeze only last 2 encoder layers
for param in model.bert.encoder.layer[-2:].parameters():
    param.requires_grad = True

# Classifier head is always trainable
for param in model.classifier.parameters():
    param.requires_grad = True
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Pre-training | Train on large unlabeled corpus (self-supervised) |
| Fine-tuning | Adapt to downstream task with labeled data |
| Feature extraction | Use frozen model as feature generator |
| Domain adaptation | Continue pre-training on domain-specific text |
| Gradual unfreezing | Progressively unfreeze layers during training |
| Learning rate | Use 2e-5 to 5e-5 for fine-tuning (much lower than scratch) |

---

## Revision Questions

1. **What is the difference between feature extraction and fine-tuning?**
2. **Why does fine-tuning use a much smaller learning rate than training from scratch?**
3. **When would you use domain-adaptive pre-training before fine-tuning?**
4. **How does gradual unfreezing work and when is it beneficial?**
5. **With only 200 labeled samples, which transfer learning approach would you choose?**

---

[Previous: 04-pretraining-finetuning.md](04-pretraining-finetuning.md) | [Next: Unit 8 - 01-statistical-mt.md](../08-Machine-Translation/01-statistical-mt.md)
