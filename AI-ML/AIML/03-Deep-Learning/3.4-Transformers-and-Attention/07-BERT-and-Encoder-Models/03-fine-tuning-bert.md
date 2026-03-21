[← Pretraining Objectives](02-pretraining-objectives.md) | [BERT Variants →](04-bert-variants.md)

---

# Fine-tuning BERT for Downstream Tasks

Fine-tuning is what makes BERT practically powerful: a pretrained BERT model can be adapted to a wide variety of NLP tasks by adding a small task-specific output layer and training end-to-end on labeled data. Unlike feature-based approaches that freeze pretrained representations, fine-tuning updates all of BERT's parameters along with the new output layer. This approach achieves state-of-the-art results on tasks ranging from text classification to question answering, typically requiring only a few epochs of training on modest hardware. The key insight is that BERT's pretrained weights already encode rich linguistic knowledge — fine-tuning simply steers these representations toward the specific task at hand.

---

## 1. Fine-tuning Framework

### General Approach

```
  ┌──────────────────────────────────────────────────────────┐
  │                    FINE-TUNING PIPELINE                   │
  │                                                          │
  │   Step 1: Load pretrained BERT (110M params)             │
  │   Step 2: Add task-specific output head (small layer)    │
  │   Step 3: Train ALL parameters end-to-end on task data   │
  │   Step 4: Evaluate on held-out test set                  │
  │                                                          │
  │   Pretrained BERT ──→ + Output Head ──→ Fine-tuned BERT  │
  │   (general knowledge)   (task-specific)   (specialized)  │
  └──────────────────────────────────────────────────────────┘
```

### Recommended Hyperparameters

| Hyperparameter     | Recommended Range    | Notes                                |
|--------------------|----------------------|--------------------------------------|
| Learning rate      | 2e-5 to 5e-5        | Much lower than pretraining (1e-4)   |
| Batch size         | 16 or 32            | Larger if GPU memory allows          |
| Epochs             | 3 – 4               | More epochs risk overfitting         |
| Warmup proportion  | 10% of total steps   | Linear warmup then linear decay      |
| Weight decay       | 0.01                 | Applied to all layers except biases  |
| Max sequence length| 128 or 512           | Task-dependent                       |
| Dropout            | 0.1                  | Keep pretrained dropout rate         |

### Learning Rate Schedule

```
  Learning Rate During Fine-tuning
  
  Rate ↑
  5e-5 │      ╱╲
       │     ╱  ╲
       │    ╱    ╲
       │   ╱      ╲
       │  ╱        ╲
  2e-5 │ ╱          ╲
       │╱            ╲
     0 │──────────────╲──────→ Steps
       0   warmup      total
       │←──10%──→│
```

---

## 2. Task Type 1: Sequence Classification

Used for: sentiment analysis, spam detection, topic classification, natural language inference.

### Architecture

```
  Input: [CLS] This movie is absolutely wonderful ! [SEP]

  ┌────────────────────────────────────────────┐
  │           BERT Encoder (12 layers)         │
  └────────────────────────────────────────────┘
         ↓           ↓     ↓    ↓    ↓    ↓
    h_[CLS]       h_This  ...              h_[SEP]
      (768)
         ↓
  ┌──────────────┐
  │   Dropout    │
  │   (p=0.1)   │
  └──────────────┘
         ↓
  ┌──────────────┐
  │ Linear Layer │
  │ (768 → C)   │   C = number of classes
  └──────────────┘
         ↓
  ┌──────────────┐
  │   Softmax    │
  └──────────────┘
         ↓
  P(positive), P(negative)
```

### Math

```
h_CLS = BERT(input)[0]            # [CLS] token output, shape (768,)
logits = W · Dropout(h_CLS) + b   # W ∈ ℝ^(C × 768), b ∈ ℝ^C
P(class_j) = softmax(logits)_j = exp(logits_j) / Σₖ exp(logits_k)

Loss = CrossEntropy = -Σⱼ y_j · log P(class_j)

For binary classification (C=2):
  New parameters: 768 × 2 + 2 = 1,538 (vs 110M pretrained)
```

---

## 3. Task Type 2: Token Classification (NER)

Used for: named entity recognition, part-of-speech tagging, chunking.

### Architecture

```
  Input: [CLS] John works at Google in London [SEP]

  ┌────────────────────────────────────────────────────┐
  │              BERT Encoder (12 layers)               │
  └────────────────────────────────────────────────────┘
    ↓      ↓      ↓     ↓     ↓      ↓     ↓       ↓
  h_CLS  h_John h_work h_at h_Goog h_in  h_Lond  h_SEP
           ↓      ↓     ↓     ↓      ↓     ↓
  ┌─────────────────────────────────────────────────┐
  │  Linear Layer (768 → num_labels) for EACH token │
  └─────────────────────────────────────────────────┘
           ↓      ↓     ↓     ↓      ↓     ↓
         B-PER    O     O   B-ORG    O   B-LOC

  Labels follow BIO/IOB2 scheme:
    B-XXX = Beginning of entity XXX
    I-XXX = Inside (continuation) of entity XXX
    O     = Outside any entity
```

### Math

```
For each token position i (excluding [CLS] and [SEP]):
  h_i = BERT(input)[i]                         # (768,)
  logits_i = W_ner · Dropout(h_i) + b_ner      # (num_labels,)
  P(label_j | token_i) = softmax(logits_i)_j

Loss = -(1/N) Σᵢ Σⱼ y_{i,j} · log P(label_j | token_i)

where N = number of non-special tokens
```

### Handling WordPiece Subwords

```
  Original:    "Washington"
  WordPiece:   ["Wash", "##ington"]

  Strategy 1 (First subword only):
    Assign label only to "Wash", ignore "##ington" in loss
    "Wash"     → B-LOC
    "##ington" → ignored (label = -100)

  Strategy 2 (All subwords):
    "Wash"     → B-LOC
    "##ington" → I-LOC
```

---

## 4. Task Type 3: Question Answering (Extractive)

Used for: SQuAD-style reading comprehension where the answer is a span in the passage.

### Architecture

```
  Input: [CLS] Where is Paris ? [SEP] Paris is the capital of France . [SEP]
               ├── Question ──┤       ├──────── Context ────────────┤

  ┌──────────────────────────────────────────────────────────────────┐
  │                   BERT Encoder (12 layers)                       │
  └──────────────────────────────────────────────────────────────────┘
    ↓     ↓    ↓   ↓   ↓    ↓    ↓     ↓    ↓     ↓    ↓     ↓
  h_CLS h_Whe h_is h_Pa h_? h_SEP h_Par h_is h_the h_cap h_of h_Fra ...

  For context tokens only:
    ↓     ↓     ↓     ↓     ↓     ↓
  ┌────────────────────────────────────┐
  │  Start score: S_i = w_s · h_i     │   w_s ∈ ℝ^768
  │  End score:   E_i = w_e · h_i     │   w_e ∈ ℝ^768
  └────────────────────────────────────┘
    ↓     ↓     ↓     ↓     ↓     ↓
  P(start = i) = softmax(S)_i
  P(end   = j) = softmax(E)_j

  Answer span = argmax_{i,j where j≥i} [S_i + E_j]
```

### Math

```
Start probability: P(start = i) = exp(w_s · h_i) / Σₖ exp(w_s · h_k)
End probability:   P(end = j)   = exp(w_e · h_j) / Σₖ exp(w_e · h_k)

Loss = -(1/2) [log P(start = s*) + log P(end = e*)]

where s* and e* are the ground-truth start and end positions.

Answer score for span (i, j):  score(i,j) = S_i + E_j,   subject to j ≥ i

New parameters: only 2 × 768 = 1,536
```

### Worked Example

```
Context: "Paris is the capital of France"
Question: "What is the capital of France?"

Token positions (context only):
  6: "Paris"   7: "is"   8: "the"   9: "capital"   10: "of"   11: "France"

Start scores (after softmax):
  Paris: 0.72   is: 0.02   the: 0.05   capital: 0.15   of: 0.01   France: 0.05

End scores (after softmax):
  Paris: 0.85   is: 0.02   the: 0.01   capital: 0.03   of: 0.01   France: 0.08

Best span: start=6 (Paris), end=6 (Paris)
Answer: "Paris"

Or for "What country is Paris the capital of?":
Best span: start=11 (France), end=11 (France)
Answer: "France"
```

---

## 5. Task Type 4: Sentence Pair Classification

Used for: natural language inference (NLI), paraphrase detection, semantic similarity.

### Architecture

```
  Input: [CLS] A man is eating food [SEP] A man is consuming a meal [SEP]
               ├─── Sentence A ────┤     ├─── Sentence B ──────────┤
               segment_id = 0            segment_id = 1

  ┌────────────────────────────────────────────────────────────┐
  │              BERT Encoder (12 layers)                       │
  │  Segment embeddings distinguish Sentence A from B           │
  └────────────────────────────────────────────────────────────┘
                        ↓
                   h_[CLS] (768)
                        ↓
  ┌──────────────────────────────┐
  │  Linear (768 → 3) + Softmax │
  └──────────────────────────────┘
                        ↓
  P(entailment), P(contradiction), P(neutral)
```

### NLI Labels

| Label          | Meaning                                   | Example                                    |
|----------------|-------------------------------------------|--------------------------------------------|
| Entailment     | Sentence A implies Sentence B             | A: "A dog is running" → B: "An animal moves"|
| Contradiction  | Sentences are logically opposite          | A: "It is sunny" → B: "It is raining"       |
| Neutral        | Neither entailment nor contradiction      | A: "A man walks" → B: "A man walks his dog"  |

---

## 6. Comparison of Task Architectures

```
  ┌─────────────────────────────────────────────────────────────────┐
  │              BERT Fine-tuning Task Comparison                   │
  │                                                                 │
  │  Classification    Token-level       QA (Span)      Pair Tasks  │
  │                                                                 │
  │  [CLS]→ label     Each tok→ label   Start/End pos  [CLS]→ label│
  │                                                                 │
  │  ┌─────────┐      ┌─────────┐      ┌─────────┐    ┌─────────┐ │
  │  │ Softmax │      │ Softmax │      │ Softmax │    │ Softmax │ │
  │  └────┬────┘      └────┬────┘      └────┬────┘    └────┬────┘ │
  │  ┌────┴────┐      ┌────┴────┐      ┌────┴────┐    ┌────┴────┐ │
  │  │ Linear  │      │ Linear  │      │2 vectors│    │ Linear  │ │
  │  │(768→C)  │      │(768→L)  │      │ w_s,w_e │    │(768→C)  │ │
  │  └────┬────┘      └────┬────┘      └────┬────┘    └────┬────┘ │
  │       │           ┌────┴────┐      ┌────┴────┐         │      │
  │  h_[CLS]          │h₁ h₂..hₙ│     │h₁ h₂..hₙ│    h_[CLS]    │
  │       │           └────┬────┘      └────┬────┘         │      │
  │  ┌────┴─────────────────┴──────────────┴────────────────┴───┐  │
  │  │                    BERT ENCODER                          │  │
  │  └──────────────────────────────────────────────────────────┘  │
  └─────────────────────────────────────────────────────────────────┘
```

---

## 7. Fine-tuning Best Practices

### Learning Rate Selection

```
  Why use a small learning rate (2e-5 to 5e-5)?

  - Pretrained weights encode valuable knowledge
  - Large learning rates would destroy pretrained representations
  - The "catastrophic forgetting" problem:
    Too-fast fine-tuning → model forgets general language knowledge

  Layer-wise Learning Rate Decay (advanced):
  ┌──────────────────────────────────────┐
  │  Layer 12 (top):    lr = 5e-5       │  ← learns task-specific features
  │  Layer 11:          lr = 4.5e-5     │
  │  Layer 10:          lr = 4.05e-5    │
  │  ...                                │
  │  Layer 1 (bottom):  lr = 1.4e-5     │  ← preserves general features
  │  Embeddings:        lr = 1.3e-5     │
  │                                     │
  │  Decay factor: ξ = 0.9              │
  │  lr_layer_n = lr_top × ξ^(12-n)    │
  └──────────────────────────────────────┘
```

### Common Pitfalls and Solutions

| Pitfall                    | Solution                                          |
|----------------------------|---------------------------------------------------|
| Overfitting on small data  | Use fewer epochs (2-3), increase dropout           |
| Training instability       | Use warmup, lower learning rate, gradient clipping |
| Slow training              | Freeze lower layers, use mixed precision (fp16)    |
| Out of memory              | Reduce batch size, use gradient accumulation        |
| Poor performance           | Try different learning rates, more data, longer seq|

### Gradient Accumulation

```
Effective batch size = micro_batch_size × gradient_accumulation_steps

Example: GPU memory fits batch_size=8
  gradient_accumulation_steps = 4
  effective_batch_size = 8 × 4 = 32 (as if we used batch_size=32)
```

---

## 8. Code: Fine-tuning for Sentiment Classification

### Complete Training Pipeline

```python
import torch
from torch.utils.data import DataLoader
from transformers import (
    BertTokenizer,
    BertForSequenceClassification,
    AdamW,
    get_linear_schedule_with_warmup,
)
from datasets import load_dataset

# ── 1. Load dataset ─────────────────────────────────────────
dataset = load_dataset("imdb")
tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")

def tokenize_function(examples):
    return tokenizer(
        examples["text"],
        padding="max_length",
        truncation=True,
        max_length=256,
    )

tokenized = dataset.map(tokenize_function, batched=True)
tokenized.set_format("torch", columns=["input_ids", "attention_mask", "label"])

train_loader = DataLoader(tokenized["train"], batch_size=16, shuffle=True)
test_loader = DataLoader(tokenized["test"], batch_size=16)

# ── 2. Load pretrained BERT with classification head ────────
model = BertForSequenceClassification.from_pretrained(
    "bert-base-uncased",
    num_labels=2,  # positive / negative
)
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model.to(device)

# ── 3. Set up optimizer and scheduler ───────────────────────
EPOCHS = 3
LEARNING_RATE = 2e-5

optimizer = AdamW(model.parameters(), lr=LEARNING_RATE, weight_decay=0.01)

total_steps = len(train_loader) * EPOCHS
warmup_steps = int(0.1 * total_steps)

scheduler = get_linear_schedule_with_warmup(
    optimizer,
    num_warmup_steps=warmup_steps,
    num_training_steps=total_steps,
)

# ── 4. Training loop ────────────────────────────────────────
model.train()
for epoch in range(EPOCHS):
    total_loss = 0
    correct = 0
    total = 0

    for batch in train_loader:
        input_ids = batch["input_ids"].to(device)
        attention_mask = batch["attention_mask"].to(device)
        labels = batch["label"].to(device)

        outputs = model(
            input_ids=input_ids,
            attention_mask=attention_mask,
            labels=labels,
        )

        loss = outputs.loss
        logits = outputs.logits

        total_loss += loss.item()
        preds = torch.argmax(logits, dim=-1)
        correct += (preds == labels).sum().item()
        total += labels.size(0)

        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
        optimizer.step()
        scheduler.step()
        optimizer.zero_grad()

    avg_loss = total_loss / len(train_loader)
    accuracy = correct / total
    print(f"Epoch {epoch+1}/{EPOCHS} — Loss: {avg_loss:.4f}, Acc: {accuracy:.4f}")

# ── 5. Evaluation ───────────────────────────────────────────
model.eval()
correct = 0
total = 0

with torch.no_grad():
    for batch in test_loader:
        input_ids = batch["input_ids"].to(device)
        attention_mask = batch["attention_mask"].to(device)
        labels = batch["label"].to(device)

        outputs = model(input_ids=input_ids, attention_mask=attention_mask)
        preds = torch.argmax(outputs.logits, dim=-1)
        correct += (preds == labels).sum().item()
        total += labels.size(0)

print(f"Test Accuracy: {correct / total:.4f}")

# ── 6. Save the fine-tuned model ────────────────────────────
model.save_pretrained("./bert-imdb-sentiment")
tokenizer.save_pretrained("./bert-imdb-sentiment")
```

### Using HuggingFace Trainer (Simplified)

```python
from transformers import (
    BertTokenizer,
    BertForSequenceClassification,
    Trainer,
    TrainingArguments,
)
from datasets import load_dataset
import numpy as np
from sklearn.metrics import accuracy_score, f1_score

# Load and tokenize
dataset = load_dataset("imdb")
tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")

def tokenize(examples):
    return tokenizer(examples["text"], padding="max_length",
                     truncation=True, max_length=256)

tokenized = dataset.map(tokenize, batched=True)

# Model
model = BertForSequenceClassification.from_pretrained(
    "bert-base-uncased", num_labels=2
)

# Metrics
def compute_metrics(eval_pred):
    logits, labels = eval_pred
    predictions = np.argmax(logits, axis=-1)
    return {
        "accuracy": accuracy_score(labels, predictions),
        "f1": f1_score(labels, predictions, average="weighted"),
    }

# Training arguments
training_args = TrainingArguments(
    output_dir="./results",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    per_device_eval_batch_size=32,
    learning_rate=2e-5,
    weight_decay=0.01,
    warmup_ratio=0.1,
    evaluation_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
    metric_for_best_model="accuracy",
    fp16=True,  # mixed precision for speed
    logging_steps=100,
)

# Trainer
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized["train"],
    eval_dataset=tokenized["test"],
    compute_metrics=compute_metrics,
)

# Train and evaluate
trainer.train()
results = trainer.evaluate()
print(f"Test Accuracy: {results['eval_accuracy']:.4f}")
print(f"Test F1:       {results['eval_f1']:.4f}")
```

### Quick Inference After Fine-tuning

```python
from transformers import pipeline

# Load fine-tuned model
classifier = pipeline("sentiment-analysis", model="./bert-imdb-sentiment")

reviews = [
    "This movie was absolutely fantastic! Best film of the year.",
    "Terrible acting and a boring plot. Total waste of time.",
    "It was okay, nothing special but not bad either.",
]

for review in reviews:
    result = classifier(review)[0]
    print(f"Text: {review[:50]}...")
    print(f"  Label: {result['label']}, Score: {result['score']:.4f}\n")
```

---

## 9. Real-World Applications

| Task                        | Output Strategy         | Example Use Case                    |
|-----------------------------|-------------------------|-------------------------------------|
| Sentiment Analysis          | [CLS] → 2 classes      | Product review classification       |
| Spam Detection              | [CLS] → 2 classes      | Email filtering                     |
| Named Entity Recognition    | Per-token → BIO labels  | Information extraction from docs    |
| Question Answering          | Start/end span          | Customer support automation         |
| Natural Language Inference  | [CLS] → 3 classes      | Fact verification systems           |
| Paraphrase Detection        | [CLS] → 2 classes      | Duplicate question detection        |
| Semantic Textual Similarity | [CLS] → regression      | Search query matching               |

---

## 10. Summary Table

| Concept                | Key Details                                                 |
|------------------------|-------------------------------------------------------------|
| Fine-tuning approach   | Add output head, train all parameters end-to-end            |
| Learning rate          | 2e-5 to 5e-5 (much lower than pretraining)                  |
| Training epochs        | 3-4 epochs typical                                          |
| Batch size             | 16 or 32 (with gradient accumulation if needed)             |
| Classification         | [CLS] output → linear layer → softmax over classes          |
| Token classification   | Each token output → linear layer → label (BIO scheme)       |
| Question answering     | Two vectors (w_s, w_e) dot-product with token outputs       |
| Sentence pairs         | Segment embeddings (0/1) distinguish sentences A and B      |
| Key challenge          | Avoiding catastrophic forgetting of pretrained knowledge     |
| New parameters added   | Typically < 2,000 (vs. 110M pretrained)                     |

---

## 11. Revision Questions

1. **Why does BERT fine-tuning use a much smaller learning rate (2e-5) than pretraining (1e-4)? What happens if you use too large a learning rate?**

2. **Explain how BERT handles extractive question answering. What are the start and end vectors, and how is the answer span determined?**

3. **For token classification (NER), how does BERT handle WordPiece subword tokens that split a single word into multiple pieces?**

4. **Compare the number of new parameters added for classification vs. question answering tasks. Why is fine-tuning so parameter-efficient?**

5. **What is gradient accumulation and why is it useful when fine-tuning BERT on limited GPU memory?**

6. **Describe the role of segment embeddings in sentence pair tasks. How do they help BERT distinguish between the two input sentences?**

---

[← Pretraining Objectives](02-pretraining-objectives.md) | [BERT Variants →](04-bert-variants.md)
