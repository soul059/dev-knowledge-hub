# Reading Comprehension

## Overview

Reading comprehension (RC) is the task of answering questions about a given passage of text. The model must "read" and "understand" the passage, then locate or generate the answer. This is the most studied form of QA, with SQuAD being the benchmark that popularized the task.

---

## Task Formulation

```
Input:
  Context (C): A passage of text (paragraph or document)
  Question (Q): A natural language question about C

Output:
  Answer (A): Typically a span from C (extractive)
               or a generated answer (abstractive)

Example:
  C: "The Transformer architecture was introduced by Vaswani et al.
      in the 2017 paper 'Attention Is All You Need'. It replaced
      recurrent layers with self-attention mechanisms."
  
  Q: "When was the Transformer introduced?"
  A: "2017"   ← extracted span from C
```

---

## How BERT Does Reading Comprehension

```
Input format:
  [CLS] Question tokens [SEP] Context tokens [SEP]

  [CLS] When was the Transformer introduced [SEP] The Transformer 
  architecture was introduced by Vaswani et al in the 2017 paper ...  [SEP]

BERT outputs:
  For each token in the context, predict:
    - P(start) = probability this token is the answer START
    - P(end)   = probability this token is the answer END

  Two linear layers on top of BERT:
    start_logits = W_s · h_i + b_s    (for each token i)
    end_logits   = W_e · h_i + b_e

  Answer span = argmax(start) to argmax(end)

         [CLS] When was ... [SEP] The Transformer ... in the 2017 paper ... [SEP]
  Start:  0.0   0.0  0.0          0.0    0.0           0.0  0.95  0.0
  End:    0.0   0.0  0.0          0.0    0.0           0.0  0.98  0.0
                                                            ^^^^
                                                           "2017"
```

---

## Fine-Tuning BERT for SQuAD

```python
from transformers import (
    AutoModelForQuestionAnswering, 
    AutoTokenizer, 
    TrainingArguments, 
    Trainer
)
from datasets import load_dataset

# Load data and model
dataset = load_dataset("squad")
model_name = "bert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForQuestionAnswering.from_pretrained(model_name)

def preprocess(examples):
    tokenized = tokenizer(
        examples["question"],
        examples["context"],
        truncation="only_second",
        max_length=384,
        stride=128,
        return_overflowing_tokens=True,
        return_offsets_mapping=True,
        padding="max_length",
    )
    # Map character positions to token positions for start/end
    # (simplified — full implementation handles edge cases)
    return tokenized

train_data = dataset["train"].map(preprocess, batched=True)

training_args = TrainingArguments(
    output_dir="./squad-bert",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    learning_rate=3e-5,
    weight_decay=0.01,
)

trainer = Trainer(model=model, args=training_args, train_dataset=train_data)
trainer.train()
```

---

## Handling Unanswerable Questions (SQuAD 2.0)

```
SQuAD 1.1: Every question has an answer in the passage
SQuAD 2.0: ~33% of questions are UNANSWERABLE

Strategy:
  If the best answer span score < [CLS] score → predict "no answer"

  Score(span) = start_logit[i] + end_logit[j]
  Score(null) = start_logit[CLS] + end_logit[CLS]

  If Score(null) - Score(span) > threshold τ:
    → "This question cannot be answered from the passage"
```

---

## Evaluation Metrics

```
Exact Match (EM):
  1 if predicted answer == ground truth (after normalization)
  0 otherwise
  Normalization: lowercase, remove articles/punctuation, strip whitespace

F1 Score:
  Token-level overlap between prediction and ground truth
  
  Example:
    Prediction: "the 2017 paper"
    Ground truth: "2017"
    
    Precision = |overlap| / |prediction| = 1/3
    Recall    = |overlap| / |ground truth| = 1/1
    F1        = 2 × (1/3 × 1) / (1/3 + 1) = 0.5

Leaderboard scores (SQuAD 2.0):
  Human:                  86.8 EM / 89.5 F1
  BERT-large:             80.0 EM / 83.1 F1
  ALBERT (ensemble):      90.2 EM / 92.2 F1  (superhuman!)
```

---

## Advanced RC Techniques

| Technique | Description |
|-----------|-------------|
| Sliding window | Handle long contexts by chunking with overlap (stride) |
| Document retrieval + RC | First retrieve relevant passage, then RC |
| Multi-passage RC | Read multiple passages to find answer |
| Conversational RC | Track dialogue history for follow-up questions |
| Cross-lingual RC | Train on English, test on other languages |

---

## Revision Questions

1. **How does BERT formulate reading comprehension as a span extraction task?**
2. **What are the start and end logits, and how is the answer span determined?**
3. **How does SQuAD 2.0 handle unanswerable questions?**
4. **What is the difference between Exact Match and F1 score for QA?**
5. **Why is a sliding window approach needed for long documents?**

---

[Previous: 01-types-of-qa.md](01-types-of-qa.md) | [Next: 03-extractive-qa.md](03-extractive-qa.md)
