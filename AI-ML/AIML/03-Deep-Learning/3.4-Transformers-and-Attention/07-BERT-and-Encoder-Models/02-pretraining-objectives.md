[← BERT Architecture](01-bert-architecture.md) | [Fine-tuning BERT →](03-fine-tuning-bert.md)

---

# BERT Pretraining: MLM and NSP

BERT's groundbreaking performance stems from its novel pretraining strategy. Rather than predicting the next word in a sequence (as in traditional language models), BERT introduces two self-supervised objectives: **Masked Language Modeling (MLM)** and **Next Sentence Prediction (NSP)**. These objectives allow BERT to learn deep bidirectional representations from massive amounts of unlabeled text. The model is pretrained on BooksCorpus (800M words) and English Wikipedia (2,500M words), training for 1M steps with a batch size of 256 sequences of 512 tokens each. After pretraining, BERT can be fine-tuned on specific downstream tasks with minimal additional architecture.

---

## 1. Masked Language Modeling (MLM)

### Core Idea

MLM randomly selects 15% of the input tokens and replaces them according to a specific strategy, then trains the model to predict the original tokens.

### Masking Strategy

Of the 15% selected tokens:

```
  ┌──────────────────────────────────────────────────────────┐
  │          MASKING STRATEGY (15% of tokens)                │
  │                                                          │
  │   80%  →  Replace with [MASK] token                      │
  │   10%  →  Replace with a random vocabulary token         │
  │   10%  →  Keep the original token unchanged              │
  │                                                          │
  │   Example: "The cat sat on the mat"                      │
  │                                                          │
  │   Selected token: "sat" (position 3)                     │
  │                                                          │
  │   80% chance: "The cat [MASK] on the mat"                │
  │   10% chance: "The cat banana on the mat"                │
  │   10% chance: "The cat sat on the mat"                   │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

### Why This Strategy?

| Strategy Component | Reason                                                          |
|--------------------|-----------------------------------------------------------------|
| 80% → [MASK]      | Trains the model to predict masked words from context           |
| 10% → random      | Prevents model from assuming [MASK] always indicates prediction |
| 10% → unchanged   | Biases model toward actual observed tokens                      |

If we always used [MASK], the model would never see real tokens during pretraining but would see only real tokens during fine-tuning — creating a **pretrain–fine-tune mismatch**. The 10%/10% split mitigates this.

### MLM Loss Function

For each masked position `i`, BERT predicts the original token:

```
P(token_i | context) = softmax(W · h_i + b)

where:
  h_i  = final hidden state at position i       (dimension d = 768)
  W    = output weight matrix                    (V × d)
  b    = output bias                             (V)
  V    = vocabulary size = 30,522

MLM Loss = - (1/M) Σᵢ log P(token_i | context)

where M = number of masked tokens in the sequence
```

---

## 2. MLM: Step-by-Step Worked Example

```
Step 1: Original Input
  "The quick brown fox jumps over the lazy dog"

Step 2: Tokenize (WordPiece)
  ["[CLS]", "the", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "dog", "[SEP]"]
  IDs: [101, 1996, 4248, 2829, 4419, 14523, 2058, 1996, 13971, 3899, 102]

Step 3: Select 15% of tokens for masking (exclude [CLS] and [SEP])
  9 content tokens × 0.15 ≈ 1-2 tokens
  Selected positions: 2 ("quick") and 8 ("lazy")

Step 4: Apply masking strategy
  Position 2 ("quick"):
    - Random draw → 80% bucket → replace with [MASK]
  Position 8 ("lazy"):
    - Random draw → 10% bucket → replace with random token "blue"

  Masked input:
  ["[CLS]", "the", "[MASK]", "brown", "fox", "jumps", "over", "the", "blue", "dog", "[SEP]"]

Step 5: Forward pass through BERT
  h₂ = BERT output at position 2   → (768,)
  h₈ = BERT output at position 8   → (768,)

Step 6: Predict original tokens
  P("quick" | context) = softmax(W · h₂ + b)["quick"]
  P("lazy"  | context) = softmax(W · h₈ + b)["lazy"]

Step 7: Compute loss
  L_MLM = -½ [log P("quick" | context) + log P("lazy" | context)]

Step 8: Backpropagate and update all BERT parameters
```

---

## 3. Next Sentence Prediction (NSP)

### Core Idea

NSP trains BERT to understand relationships between sentence pairs. Given sentences A and B, predict whether B actually follows A in the original corpus.

```
  ┌─────────────────────────────────────────────────────────────┐
  │              NEXT SENTENCE PREDICTION (NSP)                 │
  │                                                             │
  │  Input:   [CLS] Sentence A [SEP] Sentence B [SEP]          │
  │                                                             │
  │  Training data construction:                                │
  │                                                             │
  │  50% of the time (IsNext):                                  │
  │    Sentence A: "The cat sat on the mat"                     │
  │    Sentence B: "It was purring softly"         ← actual     │
  │    Label: IsNext (1)                                        │
  │                                                             │
  │  50% of the time (NotNext):                                 │
  │    Sentence A: "The cat sat on the mat"                     │
  │    Sentence B: "Stock prices rose sharply"     ← random     │
  │    Label: NotNext (0)                                       │
  │                                                             │
  └─────────────────────────────────────────────────────────────┘
```

### NSP Architecture

```
  Input: [CLS] sentence_A [SEP] sentence_B [SEP]
          ↓
  ┌─────────────────────────┐
  │  BERT Encoder (12 layers)│
  └─────────────────────────┘
          ↓
  [CLS] hidden state h_CLS → (768,)
          ↓
  ┌─────────────────────────┐
  │  Pooler: tanh(W_p · h)  │
  └─────────────────────────┘
          ↓
  pooled_output → (768,)
          ↓
  ┌─────────────────────────┐
  │  Linear: W_nsp (2 × 768)│
  │  + softmax               │
  └─────────────────────────┘
          ↓
  P(IsNext), P(NotNext)
```

### NSP Loss Function

```
L_NSP = - [y · log P(IsNext) + (1-y) · log P(NotNext)]

where y = 1 if sentence B actually follows A, else y = 0

P(IsNext | h_CLS) = softmax(W_nsp · tanh(W_p · h_CLS + b_p) + b_nsp)
```

---

## 4. Combined Pretraining Loss

BERT minimizes the sum of both objectives simultaneously:

```
L_total = L_MLM + L_NSP

         ┌─────────────────────────────────────────────────────┐
         │                                                     │
         │   L_total = L_MLM + L_NSP                           │
         │                                                     │
         │          = -1/M Σᵢ log P(tokenᵢ | context)         │
         │            - [y·log P(IsNext) + (1-y)·log(NotNext)] │
         │                                                     │
         │   Both losses are computed on every training sample. │
         │   No weighting factor — they contribute equally.     │
         │                                                     │
         └─────────────────────────────────────────────────────┘
```

---

## 5. Pretraining Data and Setup

### Training Corpus

| Corpus            | Size             | Description                         |
|-------------------|------------------|-------------------------------------|
| BooksCorpus       | ~800M words      | 11,038 unpublished books            |
| English Wikipedia | ~2,500M words    | Text only (lists, tables removed)   |
| **Total**         | **~3,300M words**| Combined corpus for pretraining     |

### Training Hyperparameters

```
Optimizer:         Adam (β₁=0.9, β₂=0.999, ε=1e-6)
Learning rate:     1e-4 (with linear warmup over first 10,000 steps)
Weight decay:      0.01 (L2 regularization)
Dropout:           0.1 (on all layers)
Batch size:        256 sequences
Sequence length:   128 tokens (90% of training), 512 tokens (10% of training)
Training steps:    1,000,000
Training time:     4 days on 16 TPU v3 chips (BERT-base)
                   4 days on 64 TPU v3 chips (BERT-large)
```

### Training Schedule Detail

```
  Steps 1 — 900,000:
    Sequence length: 128 tokens
    Purpose: Learn most of the model's knowledge efficiently

  Steps 900,001 — 1,000,000:
    Sequence length: 512 tokens
    Purpose: Learn long-range positional embeddings

  This two-phase approach reduces training cost by ~3× compared to
  training with 512 tokens throughout.
```

---

## 6. MLM Implementation Details

### Token Selection Process

```python
# Pseudocode for MLM masking
def create_mlm_masks(token_ids, vocab_size, mask_token_id):
    """
    Apply BERT's 80-10-10 masking strategy.
    """
    labels = token_ids.clone()
    
    # Probability matrix for selecting tokens
    probability_matrix = torch.full(labels.shape, 0.15)
    
    # Don't mask special tokens ([CLS], [SEP], [PAD])
    special_tokens_mask = get_special_tokens_mask(token_ids)
    probability_matrix.masked_fill_(special_tokens_mask, value=0.0)
    
    # Decide which tokens to mask
    masked_indices = torch.bernoulli(probability_matrix).bool()
    
    # Only compute loss on masked tokens
    labels[~masked_indices] = -100  # CrossEntropyLoss ignores -100
    
    # 80% → [MASK]
    indices_replaced = (torch.bernoulli(torch.full(labels.shape, 0.8)).bool()
                        & masked_indices)
    token_ids[indices_replaced] = mask_token_id
    
    # 10% → random token
    indices_random = (torch.bernoulli(torch.full(labels.shape, 0.5)).bool()
                      & masked_indices & ~indices_replaced)
    random_words = torch.randint(vocab_size, labels.shape)
    token_ids[indices_random] = random_words[indices_random]
    
    # 10% → unchanged (no action needed)
    
    return token_ids, labels
```

---

## 7. Code: MLM with HuggingFace

### Basic MLM Prediction

```python
from transformers import BertTokenizer, BertForMaskedLM
import torch

tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")
model = BertForMaskedLM.from_pretrained("bert-base-uncased")
model.eval()

# Create a sentence with [MASK]
text = "The capital of France is [MASK]."
inputs = tokenizer(text, return_tensors="pt")

# Find the position of [MASK]
mask_index = (inputs["input_ids"] == tokenizer.mask_token_id).nonzero(as_tuple=True)[1]

with torch.no_grad():
    outputs = model(**inputs)
    logits = outputs.logits  # (1, seq_len, vocab_size)

# Get top-5 predictions for the masked position
mask_logits = logits[0, mask_index, :]
top5 = torch.topk(mask_logits, 5, dim=-1)

print(f"Input: {text}")
print("Top 5 predictions:")
for i, (score, idx) in enumerate(zip(top5.values[0], top5.indices[0])):
    token = tokenizer.decode(idx)
    print(f"  {i+1}. {token:12s}  (score: {score:.3f})")
```

**Expected Output:**
```
Input: The capital of France is [MASK].
Top 5 predictions:
  1. paris         (score: 18.234)
  2. lyon          (score: 12.876)
  3. lille         (score: 11.543)
  4. marseille     (score: 10.987)
  5. toulouse      (score: 10.234)
```

### Multiple Masks

```python
# BERT can predict multiple [MASK] tokens simultaneously
text = "The [MASK] cat [MASK] on the mat."
inputs = tokenizer(text, return_tensors="pt")
mask_positions = (inputs["input_ids"] == tokenizer.mask_token_id).nonzero(as_tuple=True)[1]

with torch.no_grad():
    logits = model(**inputs).logits

for pos in mask_positions:
    top_token = tokenizer.decode(logits[0, pos].argmax())
    print(f"Position {pos.item()}: predicted '{top_token}'")
# Position 1: predicted 'black'
# Position 3: predicted 'sat'
```

### NSP Example

```python
from transformers import BertTokenizer, BertForNextSentencePrediction
import torch

tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")
model = BertForNextSentencePrediction.from_pretrained("bert-base-uncased")
model.eval()

# Example 1: Actual consecutive sentences (IsNext)
sentence_a = "The weather is beautiful today."
sentence_b = "Let's go for a walk in the park."
inputs = tokenizer(sentence_a, sentence_b, return_tensors="pt")

with torch.no_grad():
    outputs = model(**inputs)
    logits = outputs.logits  # (1, 2) — [IsNext_score, NotNext_score]

probs = torch.softmax(logits, dim=-1)
print(f"Sentence A: {sentence_a}")
print(f"Sentence B: {sentence_b}")
print(f"P(IsNext):  {probs[0, 0]:.4f}")
print(f"P(NotNext): {probs[0, 1]:.4f}")

# Example 2: Unrelated sentences (NotNext)
sentence_a = "The weather is beautiful today."
sentence_b = "Quantum computing uses qubits for processing."
inputs = tokenizer(sentence_a, sentence_b, return_tensors="pt")

with torch.no_grad():
    outputs = model(**inputs)
    probs = torch.softmax(outputs.logits, dim=-1)

print(f"\nSentence A: {sentence_a}")
print(f"Sentence B: {sentence_b}")
print(f"P(IsNext):  {probs[0, 0]:.4f}")
print(f"P(NotNext): {probs[0, 1]:.4f}")
```

---

## 8. Limitations of BERT's Pretraining

### MLM Limitations

```
Problem 1: Independence Assumption
  Masked tokens are predicted independently of each other.
  If "New" and "York" are both masked, the model predicts each
  without knowing the other is masked too.

  "I went to [MASK] [MASK] for vacation"
  Predict "New" without knowing "York" is also masked, and vice versa.

Problem 2: 15% Masking Rate
  Only 15% of tokens provide training signal per example.
  Autoregressive models (GPT) use 100% of tokens for training.
  This makes BERT's pretraining less sample-efficient.

Problem 3: Pretrain–Fine-tune Mismatch
  [MASK] token never appears during fine-tuning.
  The 80-10-10 strategy mitigates but doesn't fully solve this.
```

### NSP Limitations

```
Later work (RoBERTa, ALBERT) showed that NSP is NOT essential:
  - NSP may conflate topic prediction with coherence prediction
  - When sentence pairs come from different documents, the model
    can distinguish them just by topic, without learning coherence
  - RoBERTa removes NSP entirely and achieves better results
```

---

## 9. Real-World Applications

| Application                   | How Pretraining Helps                                     |
|-------------------------------|-----------------------------------------------------------|
| **Transfer Learning**         | MLM learns universal language representations              |
| **Low-Resource Languages**    | Multilingual BERT pretrains on 104 languages               |
| **Domain Adaptation**         | Continue pretraining on domain text (BioBERT, SciBERT)     |
| **Data Augmentation**         | Use MLM to generate plausible token replacements           |
| **Feature Extraction**        | Use frozen BERT embeddings as input features               |
| **Probing Tasks**             | Study what linguistic knowledge each layer captures        |

---

## 10. Summary Table

| Concept              | Details                                                       |
|----------------------|---------------------------------------------------------------|
| MLM objective        | Predict 15% randomly masked tokens from bidirectional context |
| Masking strategy     | 80% [MASK], 10% random token, 10% unchanged                  |
| NSP objective        | Binary classification: does sentence B follow sentence A?     |
| Training data        | BooksCorpus (800M words) + Wikipedia (2,500M words)           |
| Combined loss        | L_total = L_MLM + L_NSP (equal weighting)                     |
| Optimizer            | Adam with warmup and linear decay                             |
| Training duration    | 1M steps, 4 days on 16-64 TPU v3 chips                       |
| MLM limitation       | Independent predictions, only 15% signal per sample           |
| NSP limitation       | Later shown to be unnecessary (RoBERTa removes it)            |
| Key innovation       | Bidirectional pretraining enables powerful transfer learning   |

---

## 11. Revision Questions

1. **Explain the 80-10-10 masking strategy in MLM. Why doesn't BERT simply replace all selected tokens with [MASK]?**

2. **Walk through a complete MLM example: given a sentence, show which tokens are selected, how they are modified, and how the loss is computed.**

3. **What is the Next Sentence Prediction task? How are positive and negative training examples constructed?**

4. **Why did later models like RoBERTa remove the NSP objective? What evidence suggests NSP is not necessary?**

5. **BERT's MLM predicts masked tokens independently. Why is this a limitation, and how might you address it? (Hint: consider XLNet's permutation language modeling.)**

6. **Compare BERT's pretraining approach with GPT's autoregressive pretraining. What are the trade-offs in terms of bidirectionality, training efficiency, and downstream task suitability?**

---

[← BERT Architecture](01-bert-architecture.md) | [Fine-tuning BERT →](03-fine-tuning-bert.md)
