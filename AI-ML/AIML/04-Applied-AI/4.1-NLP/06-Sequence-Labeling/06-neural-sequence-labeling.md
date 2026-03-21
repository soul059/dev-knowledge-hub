# Neural Sequence Labeling

## Overview

Modern sequence labeling uses neural networks to assign labels to each token in a sequence. From BiLSTMs to transformer-based models, neural approaches have pushed NER, POS tagging, and chunking to near-human performance by learning features automatically from data.

---

## Architecture Evolution

```
Rule-based → HMM → CRF → BiLSTM → BiLSTM-CRF → BERT-based
  ~77%       ~95%   ~93%   ~90%      ~93%          ~95%+
                                                  (NER F1)
```

---

## BERT for Sequence Labeling

The current state-of-the-art approach fine-tunes a pre-trained transformer for token classification:

```
Input:     [CLS]  Barack  Obama  visited  Paris  [SEP]
             ↓      ↓      ↓       ↓       ↓      ↓
           ┌─────────────────────────────────────────┐
           │            BERT (12 layers)              │
           └─────────────────────────────────────────┘
             ↓      ↓      ↓       ↓       ↓      ↓
  Hidden:  [h_cls] [h_1]  [h_2]  [h_3]   [h_4]  [h_sep]
                    ↓      ↓       ↓       ↓
  Linear:        [FC]   [FC]    [FC]    [FC]
                    ↓      ↓       ↓       ↓
  Output:        B-PER  I-PER    O     B-LOC
```

```python
from transformers import (AutoTokenizer, AutoModelForTokenClassification,
                          TrainingArguments, Trainer)

# Load pre-trained model for NER
model_name = "bert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(model_name)

# Label mapping
label_list = ['O', 'B-PER', 'I-PER', 'B-ORG', 'I-ORG', 'B-LOC', 'I-LOC']

model = AutoModelForTokenClassification.from_pretrained(
    model_name, num_labels=len(label_list)
)

# Fine-tune on your NER dataset
training_args = TrainingArguments(
    output_dir="./ner_model",
    num_train_epochs=5,
    per_device_train_batch_size=16,
    learning_rate=2e-5,
    weight_decay=0.01,
)
```

---

## Handling Subword Tokenization

```
Problem: BERT uses WordPiece, splitting words into subwords

Word:     "Washington"
Subwords: ["Wash", "##ing", "##ton"]
Label:    B-LOC   ???     ???

Solution: Only label the first subtoken, ignore the rest
  "Wash"    → B-LOC
  "##ing"   → -100  (ignored in loss)
  "##ton"   → -100  (ignored in loss)
```

```python
def align_labels_with_tokens(labels, word_ids):
    """Align word-level labels to subword tokens."""
    aligned = []
    previous_word_id = None
    
    for word_id in word_ids:
        if word_id is None:
            aligned.append(-100)       # Special tokens
        elif word_id != previous_word_id:
            aligned.append(labels[word_id])  # First subtoken
        else:
            aligned.append(-100)       # Subsequent subtokens
        previous_word_id = word_id
    
    return aligned
```

---

## Using Pre-trained NER Directly

```python
from transformers import pipeline

# No training needed — use pre-trained NER model
ner = pipeline("ner", model="dslim/bert-base-NER", grouped_entities=True)

text = "Elon Musk founded SpaceX in Hawthorne, California"
entities = ner(text)

for ent in entities:
    print(f"  {ent['word']:20s} {ent['entity_group']:5s} ({ent['score']:.3f})")
# Elon Musk            PER   (0.999)
# SpaceX               ORG   (0.998)
# Hawthorne            LOC   (0.997)
# California           LOC   (0.999)
```

---

## Model Comparison for Sequence Labeling

| Model | NER F1 (CoNLL) | Speed | Params | Notes |
|-------|:-:|:-:|:-:|---|
| CRF (feature-based) | ~88% | ⚡⚡⚡ | ~1M | Handcrafted features |
| BiLSTM | ~90% | ⚡⚡ | ~5M | Learned features |
| BiLSTM-CRF | ~92% | ⚡⚡ | ~6M | + label dependencies |
| BERT-base | ~92% | ⚡ | 110M | Pre-trained knowledge |
| BERT-large | ~93% | 🐢 | 340M | More capacity |
| RoBERTa | ~93.5% | 🐢 | 355M | Better pre-training |

---

## Revision Questions

1. **How is BERT adapted for token-level classification (NER)?**
2. **Why is subword tokenization alignment important for sequence labeling?**
3. **How does a pre-trained NER pipeline work without any training?**
4. **What advantage does BERT-based NER have over BiLSTM-CRF?**
5. **How do you handle the -100 label for subword tokens during training?**

---

[Previous: 05-crf.md](05-crf.md) | [Next: Unit 7 - 01-ngram-language-models.md](../07-Language-Models/01-ngram-language-models.md)
