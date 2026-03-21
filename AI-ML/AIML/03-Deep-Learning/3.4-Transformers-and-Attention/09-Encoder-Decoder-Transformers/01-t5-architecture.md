# T5: Text-to-Text Transfer Transformer

[← Prompt Engineering](../08-GPT-and-Decoder-Models/06-prompt-engineering.md) | [BART Architecture →](02-bart-architecture.md)

---

## Overview

T5 (Text-to-Text Transfer Transformer) was introduced by Raffel et al. (2019) in *"Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer."* The central insight is remarkably elegant: cast **every NLP task** — classification, translation, summarization, question answering — into a single unified format where the model receives a text string as input and produces a text string as output. By prepending a task-specific prefix such as `"translate English to German:"` or `"summarize:"`, the same architecture, loss function, and training procedure can handle vastly different tasks. This simplifies the research landscape and enables systematic comparison of pre-training objectives, architectures, and data scales. T5 comes in multiple sizes from Small (60M parameters) to XXL (11B parameters), and was pre-trained on the Colossal Clean Crawled Corpus (C4), a cleaned version of Common Crawl containing ~750 GB of English text.

---

## 1. The Text-to-Text Framework

### 1.1 Core Idea

Every NLP task is reformulated as a **text-to-text** mapping:

```
Input:  "task prefix: <input text>"
Output: "<output text>"
```

### 1.2 Task Prefix Examples

| Task               | Input Format                                        | Expected Output            |
|---------------------|-----------------------------------------------------|----------------------------|
| Translation         | `"translate English to German: That is good."`      | `"Das ist gut."`           |
| Summarization       | `"summarize: <long article text>"`                  | `"<concise summary>"`      |
| Sentiment           | `"sst2 sentence: This movie is great."`             | `"positive"`               |
| Similarity          | `"stsb sentence1: A man sings. sentence2: ..."`    | `"4.2"`                    |
| Question Answering  | `"question: Who won? context: The Lakers won..."`   | `"The Lakers"`             |

### 1.3 Why Text-to-Text?

- **Unified API**: one model, one loss, one decoding strategy for all tasks.
- **Transfer across tasks**: knowledge from one task can help another.
- **Simplicity**: no task-specific heads — the decoder generates answer tokens.

---

## 2. Architecture

### 2.1 Encoder-Decoder Transformer

T5 uses the original encoder-decoder architecture from Vaswani et al. (2017) with several modifications:

```
                        T5 ARCHITECTURE
  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │   INPUT: "translate English to German: Good day."   │
  │                       │                             │
  │                 ┌─────▼─────┐                       │
  │                 │ Tokenizer │                       │
  │                 └─────┬─────┘                       │
  │                       │                             │
  │          ┌────────────▼────────────┐                │
  │          │      ENCODER            │                │
  │          │  ┌──────────────────┐   │                │
  │          │  │ Self-Attention   │   │                │
  │          │  │ + Relative Bias  │   │                │
  │          │  ├──────────────────┤   │                │
  │          │  │ Feed-Forward     │   │                │
  │          │  │ Network          │   │                │
  │          │  └──────────────────┘   │                │
  │          │    × N layers           │                │
  │          └────────────┬────────────┘                │
  │                       │ encoder output              │
  │          ┌────────────▼────────────┐                │
  │          │      DECODER            │                │
  │          │  ┌──────────────────┐   │                │
  │          │  │ Causal Self-Attn │   │                │
  │          │  │ + Relative Bias  │   │                │
  │          │  ├──────────────────┤   │                │
  │          │  │ Cross-Attention  │   │                │
  │          │  │ (attend encoder) │   │                │
  │          │  ├──────────────────┤   │                │
  │          │  │ Feed-Forward     │   │                │
  │          │  │ Network          │   │                │
  │          │  └──────────────────┘   │                │
  │          │    × N layers           │                │
  │          └────────────┬────────────┘                │
  │                       │                             │
  │                 ┌─────▼─────┐                       │
  │                 │  Softmax  │                       │
  │                 └─────┬─────┘                       │
  │                       │                             │
  │   OUTPUT: "Guten Tag."                              │
  └─────────────────────────────────────────────────────┘
```

### 2.2 Key Modifications from Original Transformer

| Feature                    | Original Transformer       | T5                              |
|----------------------------|----------------------------|---------------------------------|
| Positional encoding        | Sinusoidal (absolute)      | Relative positional bias        |
| Layer normalization        | Post-norm                  | Pre-norm (RMSNorm-style)        |
| Activation function        | ReLU                       | GeLU (Gaussian Error Linear)    |
| Dropout                    | Applied                    | Applied during training         |
| Embedding sharing          | Separate                   | Shared input-output embeddings  |
| Bias terms in dense layers | Yes                        | No bias terms                   |

---

## 3. Relative Positional Bias

### 3.1 Motivation

Absolute positional encodings (sinusoidal or learned) encode the position of each token directly. T5 instead uses **relative positional bias** — a learned scalar added to the attention logits based on the *distance* between query and key positions.

### 3.2 Mathematical Formulation

Standard attention with relative bias:

$$
\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^T}{\sqrt{d_k}} + B\right) V
$$

Where **B** is the relative position bias matrix:

```
B[i][j] = b(i - j)
```

The function `b(·)` maps relative distances to learned scalar biases. Distances are **bucketed** to handle long sequences:

```
For |i - j| ≤ threshold:  linear mapping (one bucket per position)
For |i - j| > threshold:  logarithmic bucketing
```

### 3.3 Bucketing Formula

```
bucket(d) = {
    d,                                     if d < num_linear_buckets
    num_linear_buckets + log(d / num_linear_buckets)
        / log(max_distance / num_linear_buckets)
        × (num_buckets - num_linear_buckets),   otherwise
}
```

With typical values: `num_buckets = 32`, `num_linear_buckets = 16`, `max_distance = 128`.

### 3.4 Key Properties

- Shared across layers (within the first layer, then reused).
- Separate bias tables for encoder self-attention and decoder self/cross-attention.
- No sinusoidal functions needed — purely learned.

---

## 4. Pre-Training: Span Corruption

### 4.1 Objective

T5 is pre-trained using **span corruption** (also called span denoising). Random contiguous spans of tokens are replaced with sentinel tokens, and the model must predict the original spans.

### 4.2 Example

```
Original:  "The quick brown fox jumps over the lazy dog"

Corrupted: "The <extra_id_0> fox jumps <extra_id_1> dog"

Target:    "<extra_id_0> quick brown <extra_id_1> over the lazy"
```

### 4.3 Corruption Parameters

| Parameter             | Value    | Description                          |
|-----------------------|----------|--------------------------------------|
| Corruption rate       | 15%      | Fraction of tokens masked            |
| Mean span length      | 3        | Average length of each masked span   |
| Sentinel tokens       | 100      | `<extra_id_0>` to `<extra_id_99>`    |

### 4.4 Why Span Corruption?

The paper compared multiple pre-training objectives:

```
┌─────────────────────────┬─────────────────────────────────────┐
│ Objective               │ Description                         │
├─────────────────────────┼─────────────────────────────────────┤
│ Language Model (LM)     │ Predict next token (left-to-right)  │
│ BERT-style              │ Mask individual tokens               │
│ Deshuffling             │ Unscramble shuffled sentence         │
│ Span Corruption ✓       │ Mask & predict contiguous spans      │
├─────────────────────────┼─────────────────────────────────────┤
│ Winner                  │ Span corruption (best avg score)     │
└─────────────────────────┴─────────────────────────────────────┘
```

Span corruption is more efficient because the target sequence is shorter than the full input.

---

## 5. T5 Model Sizes

```
┌──────────┬────────────┬────────┬────────┬────────────────┐
│ Variant  │ Parameters │ Layers │ d_model│ d_ff           │
├──────────┼────────────┼────────┼────────┼────────────────┤
│ Small    │ 60M        │ 6      │ 512    │ 2048           │
│ Base     │ 220M       │ 12     │ 768    │ 3072           │
│ Large    │ 770M       │ 24     │ 1024   │ 4096           │
│ 3B       │ 2.8B       │ 24     │ 1024   │ 16384          │
│ 11B (XXL)│ 11B        │ 24     │ 1024   │ 65536          │
└──────────┴────────────┴────────┴────────┴────────────────┘
```

---

## 6. Training Details

### 6.1 Loss Function

Standard cross-entropy over output tokens:

```
L = -∑_{t=1}^{T} log P(y_t | y_{<t}, X)
```

Where `X` is the encoder input and `y_t` is the t-th target token.

### 6.2 Optimizer and Schedule

- **Optimizer**: AdaFactor (memory-efficient variant of Adam)
- **Learning rate**: Inverse square root schedule: `lr = 1 / sqrt(max(step, warmup_steps))`
- **Warmup**: 10,000 steps
- **Pre-training steps**: 1,000,000 (on C4)

---

## 7. Worked Example: Sentiment Classification as Text-to-Text

### Step 1: Format Input

```
Input:  "sst2 sentence: This film captures the essence of human connection."
```

### Step 2: Encoder Processing

The encoder processes all input tokens in parallel with bidirectional self-attention + relative positional bias.

### Step 3: Decoder Generation

The decoder generates tokens autoregressively:

```
Step 0: <BOS>        → P("positive") = 0.92, P("negative") = 0.08
Step 1: "positive"   → P(<EOS>) = 0.99
Output: "positive"
```

### Step 4: Map Back

```
"positive" → Positive sentiment (label 1)
```

---

## 8. Code: Using T5 with HuggingFace

### 8.1 Basic Inference for Multiple Tasks

```python
from transformers import T5ForConditionalGeneration, T5Tokenizer

# Load model and tokenizer
model_name = "t5-small"
tokenizer = T5Tokenizer.from_pretrained(model_name)
model = T5ForConditionalGeneration.from_pretrained(model_name)

# --- Translation ---
input_text = "translate English to German: How are you today?"
input_ids = tokenizer(input_text, return_tensors="pt").input_ids
outputs = model.generate(input_ids, max_length=50)
print("Translation:", tokenizer.decode(outputs[0], skip_special_tokens=True))

# --- Summarization ---
article = (
    "summarize: The tower is 324 metres tall, about the same height as "
    "an 81-storey building, and the tallest structure in Paris. Its base "
    "is square, measuring 125 metres on each side. It was the first "
    "structure to reach a height of 300 metres."
)
input_ids = tokenizer(article, return_tensors="pt").input_ids
outputs = model.generate(input_ids, max_length=60)
print("Summary:", tokenizer.decode(outputs[0], skip_special_tokens=True))

# --- Sentiment / Classification ---
input_text = "sst2 sentence: This movie was absolutely wonderful."
input_ids = tokenizer(input_text, return_tensors="pt").input_ids
outputs = model.generate(input_ids, max_length=10)
print("Sentiment:", tokenizer.decode(outputs[0], skip_special_tokens=True))
```

### 8.2 Fine-Tuning T5

```python
import torch
from torch.utils.data import DataLoader, Dataset
from transformers import T5ForConditionalGeneration, T5Tokenizer, AdamW

class TextToTextDataset(Dataset):
    def __init__(self, inputs, targets, tokenizer, max_len=128):
        self.inputs = inputs
        self.targets = targets
        self.tokenizer = tokenizer
        self.max_len = max_len

    def __len__(self):
        return len(self.inputs)

    def __getitem__(self, idx):
        input_enc = self.tokenizer(
            self.inputs[idx], max_length=self.max_len,
            padding="max_length", truncation=True, return_tensors="pt"
        )
        target_enc = self.tokenizer(
            self.targets[idx], max_length=self.max_len,
            padding="max_length", truncation=True, return_tensors="pt"
        )
        labels = target_enc.input_ids.squeeze()
        labels[labels == self.tokenizer.pad_token_id] = -100
        return {
            "input_ids": input_enc.input_ids.squeeze(),
            "attention_mask": input_enc.attention_mask.squeeze(),
            "labels": labels,
        }

# Example data
train_inputs = [
    "summarize: The cat sat on the mat. It was a sunny day.",
    "summarize: Machine learning uses data to find patterns.",
]
train_targets = ["Cat on mat on sunny day.", "ML finds data patterns."]

tokenizer = T5Tokenizer.from_pretrained("t5-small")
model = T5ForConditionalGeneration.from_pretrained("t5-small")

dataset = TextToTextDataset(train_inputs, train_targets, tokenizer)
loader = DataLoader(dataset, batch_size=2, shuffle=True)

optimizer = AdamW(model.parameters(), lr=3e-4)

model.train()
for epoch in range(3):
    for batch in loader:
        outputs = model(
            input_ids=batch["input_ids"],
            attention_mask=batch["attention_mask"],
            labels=batch["labels"],
        )
        loss = outputs.loss
        loss.backward()
        optimizer.step()
        optimizer.zero_grad()
        print(f"Epoch {epoch}, Loss: {loss.item():.4f}")
```

### 8.3 Span Corruption (Pre-Training Objective Demonstration)

```python
import random

def span_corrupt(tokens, corruption_rate=0.15, mean_span_length=3):
    """Simulate T5's span corruption pre-training objective."""
    n = len(tokens)
    num_corrupt = max(1, int(n * corruption_rate))
    mask = [False] * n

    corrupted = 0
    while corrupted < num_corrupt:
        span_len = max(1, int(random.expovariate(1.0 / mean_span_length)))
        start = random.randint(0, n - 1)
        for i in range(start, min(start + span_len, n)):
            if not mask[i]:
                mask[i] = True
                corrupted += 1
            if corrupted >= num_corrupt:
                break

    # Build corrupted input and target
    corrupted_tokens, target_tokens = [], []
    sentinel_id = 0
    in_span = False
    for i, token in enumerate(tokens):
        if mask[i]:
            if not in_span:
                corrupted_tokens.append(f"<extra_id_{sentinel_id}>")
                target_tokens.append(f"<extra_id_{sentinel_id}>")
                sentinel_id += 1
                in_span = True
            target_tokens.append(token)
        else:
            corrupted_tokens.append(token)
            in_span = False

    return " ".join(corrupted_tokens), " ".join(target_tokens)

sentence = "The quick brown fox jumps over the lazy dog near the river"
tokens = sentence.split()
corrupted, target = span_corrupt(tokens)
print(f"Original:  {sentence}")
print(f"Corrupted: {corrupted}")
print(f"Target:    {target}")
```

---

## 9. Real-World Applications

| Application               | Task Prefix               | Industry Use Case                      |
|---------------------------|---------------------------|----------------------------------------|
| Machine Translation       | `translate X to Y:`      | Localization platforms, travel apps     |
| Document Summarization    | `summarize:`              | News aggregators, legal document review |
| Sentiment Analysis        | `sst2 sentence:`          | Product review analysis, brand monitoring|
| Grammar Correction        | `grammar:`                | Writing assistants, educational tools   |
| Question Answering        | `question: ... context:`  | Customer support bots, search engines   |
| Paraphrase Detection      | `mrpc sentence1: ...`    | Plagiarism detection, deduplication     |
| Data-to-Text Generation   | `describe:`               | Sports reports, financial summaries     |

---

## 10. Summary Table

| Aspect                 | Detail                                              |
|------------------------|-----------------------------------------------------|
| **Paper**              | Raffel et al. (2019)                                |
| **Framework**          | Text-to-text (every task = string → string)         |
| **Architecture**       | Encoder-decoder transformer                         |
| **Positional Encoding**| Relative positional bias (learned, bucketed)        |
| **Pre-training Data**  | C4 (Colossal Clean Crawled Corpus, ~750 GB)         |
| **Pre-training Task**  | Span corruption (15% rate, mean span length 3)      |
| **Sizes**              | Small (60M) → XXL (11B)                             |
| **Optimizer**          | AdaFactor                                           |
| **Key Innovation**     | Unified text-to-text format for all NLP tasks       |
| **Loss**               | Cross-entropy on generated tokens                   |
| **Tokenizer**          | SentencePiece (unigram model, 32k vocab)            |

---

## 11. Revision Questions

1. **Explain the text-to-text framework.** How does T5 convert a classification task into a generation task? What are the advantages and potential limitations of this approach?

2. **Compare relative positional bias with sinusoidal positional encoding.** How does bucketing help T5 handle sequences longer than those seen during training?

3. **Describe the span corruption pre-training objective.** Why did T5 choose span corruption over BERT-style masking or language modeling? What makes the target sequence more efficient?

4. **Walk through a worked example** of how T5 performs machine translation from input formatting to final decoding. Include the role of the encoder and decoder.

5. **What is the role of the task prefix?** How does a single model distinguish between translation, summarization, and classification without separate output heads?

6. **Compare T5 model sizes.** How does scaling from T5-Small (60M) to T5-XXL (11B) affect performance on downstream benchmarks? What are the compute tradeoffs?

---

[← Prompt Engineering](../08-GPT-and-Decoder-Models/06-prompt-engineering.md) | [BART Architecture →](02-bart-architecture.md)
