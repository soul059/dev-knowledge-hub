[← BERT Variants](04-bert-variants.md) | [GPT Architecture →](../08-GPT-and-Decoder-Models/01-gpt-architecture.md)

---

# BERT in Practice: Real-World Applications

BERT has become the backbone of modern NLP systems across industry and research. Its ability to produce rich, contextualized representations makes it exceptionally versatile: the same pretrained model can power sentiment analysis, named entity recognition, question answering, semantic search, and more — each requiring only a thin task-specific layer on top. This chapter provides end-to-end practical implementations of BERT's most important applications, complete with code, performance benchmarks, and production considerations. By the end, you will be able to build, evaluate, and deploy BERT-based NLP pipelines for real-world use cases.

---

## 1. Application Overview

```
  ┌─────────────────────────────────────────────────────────────┐
  │              BERT APPLICATION LANDSCAPE                      │
  │                                                             │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
  │  │  Sentiment   │  │     NER      │  │  Question    │      │
  │  │  Analysis    │  │ (Token-level)│  │  Answering   │      │
  │  │  [CLS]→label │  │  tok→BIO tag │  │  span predict│      │
  │  └──────────────┘  └──────────────┘  └──────────────┘      │
  │                                                             │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
  │  │  Semantic    │  │   Search &   │  │  Text        │      │
  │  │  Similarity  │  │   Ranking    │  │  Generation  │      │
  │  │  embed→cosine│  │  bi/cross-enc│  │  (limited)   │      │
  │  └──────────────┘  └──────────────┘  └──────────────┘      │
  │                                                             │
  └─────────────────────────────────────────────────────────────┘
```

---

## 2. Sentiment Analysis

### Pipeline Architecture

```
  Input: "This product exceeded all my expectations!"
           ↓
  ┌──────────────────┐
  │ BERT Tokenizer   │  →  [CLS] this product exceeded all my expectations ! [SEP]
  └──────────────────┘
           ↓
  ┌──────────────────┐
  │ BERT Encoder     │  →  Contextual embeddings (seq_len, 768)
  │ (12 layers)      │
  └──────────────────┘
           ↓
  h_[CLS] = (768,)
           ↓
  ┌──────────────────┐
  │ Linear (768→2)   │  →  logits: [2.8, -1.3]
  │ + Softmax        │  →  probs:  [0.98, 0.02]
  └──────────────────┘
           ↓
  Output: POSITIVE (confidence: 98.3%)
```

### Code: Sentiment Analysis Pipeline

```python
from transformers import pipeline

# Method 1: Using HuggingFace pipeline (simplest)
sentiment_pipeline = pipeline(
    "sentiment-analysis",
    model="nlptown/bert-base-multilingual-uncased-sentiment",
)

reviews = [
    "Absolutely love this product! Best purchase ever.",
    "Terrible quality, broke after one week. Very disappointed.",
    "It's okay, nothing special. Does the job.",
    "The customer service was outstanding and very helpful!",
    "Worst experience of my life. Never buying again.",
]

print("=" * 65)
print(f"{'Review':<45} {'Label':<12} {'Score'}")
print("=" * 65)
for review in reviews:
    result = sentiment_pipeline(review)[0]
    print(f"{review[:44]:<45} {result['label']:<12} {result['score']:.4f}")
```

### Code: Custom Sentiment Classifier

```python
import torch
import torch.nn as nn
from transformers import BertModel, BertTokenizer

class SentimentClassifier(nn.Module):
    def __init__(self, num_classes=3, dropout=0.3):
        super().__init__()
        self.bert = BertModel.from_pretrained("bert-base-uncased")
        self.dropout = nn.Dropout(dropout)
        self.classifier = nn.Linear(768, num_classes)

    def forward(self, input_ids, attention_mask):
        outputs = self.bert(input_ids=input_ids, attention_mask=attention_mask)
        pooled = outputs.pooler_output           # [CLS] token, (batch, 768)
        dropped = self.dropout(pooled)
        logits = self.classifier(dropped)         # (batch, num_classes)
        return logits

# Usage
model = SentimentClassifier(num_classes=3)
tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")

text = "The movie was surprisingly good!"
inputs = tokenizer(text, return_tensors="pt", padding=True, truncation=True)

with torch.no_grad():
    logits = model(inputs["input_ids"], inputs["attention_mask"])
    probs = torch.softmax(logits, dim=-1)
    labels = ["Negative", "Neutral", "Positive"]
    pred = labels[probs.argmax().item()]
    print(f"Prediction: {pred} ({probs.max():.2%})")
```

---

## 3. Named Entity Recognition (NER)

### NER Tag Scheme

```
  BIO Tagging (IOB2 format):
  ═════════════════════════

  Text:   "Barack Obama visited the White House in Washington"

  Tokens:  Barack   Obama  visited  the  White  House  in  Washington
  Tags:    B-PER    I-PER  O        O    B-LOC  I-LOC  O   B-LOC

  Tag meanings:
    B-PER  = Beginning of a person entity
    I-PER  = Inside (continuation) of a person entity
    B-ORG  = Beginning of an organization entity
    I-ORG  = Inside an organization
    B-LOC  = Beginning of a location entity
    I-LOC  = Inside a location
    B-MISC = Beginning of a miscellaneous entity
    I-MISC = Inside a miscellaneous entity
    O      = Outside any entity
```

### Code: NER Pipeline

```python
from transformers import pipeline

ner_pipeline = pipeline(
    "ner",
    model="dslim/bert-base-NER",
    aggregation_strategy="simple",  # merge subword tokens
)

texts = [
    "Elon Musk founded SpaceX in Hawthorne, California.",
    "The United Nations headquarters is in New York City.",
    "Apple Inc. released the iPhone 15 in September 2023.",
]

for text in texts:
    print(f"\nText: {text}")
    entities = ner_pipeline(text)
    for ent in entities:
        print(f"  {ent['entity_group']:6s} | {ent['word']:20s} | "
              f"Score: {ent['score']:.4f} | Pos: {ent['start']}-{ent['end']}")
```

**Expected Output:**
```
Text: Elon Musk founded SpaceX in Hawthorne, California.
  PER    | Elon Musk            | Score: 0.9987 | Pos: 0-9
  ORG    | SpaceX               | Score: 0.9964 | Pos: 18-24
  LOC    | Hawthorne             | Score: 0.9951 | Pos: 28-37
  LOC    | California            | Score: 0.9978 | Pos: 39-49
```

### Code: Custom NER Model

```python
import torch
from transformers import BertTokenizerFast, BertForTokenClassification

# Labels for CoNLL-2003
label_list = ["O", "B-PER", "I-PER", "B-ORG", "I-ORG",
              "B-LOC", "I-LOC", "B-MISC", "I-MISC"]

tokenizer = BertTokenizerFast.from_pretrained("bert-base-uncased")
model = BertForTokenClassification.from_pretrained(
    "bert-base-uncased", num_labels=len(label_list)
)

text = "Marie Curie worked at the University of Paris"
inputs = tokenizer(text, return_tensors="pt", is_split_into_words=False)

with torch.no_grad():
    outputs = model(**inputs)
    predictions = torch.argmax(outputs.logits, dim=-1)[0]

tokens = tokenizer.convert_ids_to_tokens(inputs["input_ids"][0])
for token, pred_id in zip(tokens, predictions):
    if token not in ["[CLS]", "[SEP]", "[PAD]"]:
        print(f"  {token:15s} → {label_list[pred_id]}")
```

---

## 4. Extractive Question Answering

### How It Works

```
  ┌─────────────────────────────────────────────────────────────┐
  │            EXTRACTIVE QUESTION ANSWERING                     │
  │                                                             │
  │  Question: "When was Python created?"                       │
  │  Context:  "Python was created by Guido van Rossum and      │
  │             first released in 1991. It emphasizes code      │
  │             readability and simplicity."                     │
  │                                                             │
  │  Input to BERT:                                             │
  │  [CLS] when was python created ? [SEP] python was created  │
  │  by guido van rossum and first released in 1991 . it       │
  │  emphasizes code readability and simplicity . [SEP]         │
  │                                                             │
  │  BERT predicts:                                             │
  │    Start position → "1991"                                  │
  │    End position   → "1991"                                  │
  │                                                             │
  │  Answer: "1991"                                             │
  └─────────────────────────────────────────────────────────────┘
```

### Code: Question Answering

```python
from transformers import pipeline

qa_pipeline = pipeline(
    "question-answering",
    model="deepset/bert-base-cased-squad2",
)

context = """
The Transformer architecture was introduced in the paper "Attention Is All You Need"
by Vaswani et al. in 2017. It revolutionized natural language processing by replacing
recurrent neural networks with self-attention mechanisms. The original Transformer
consists of an encoder and a decoder, each with 6 layers. BERT, introduced in 2018,
uses only the encoder part with 12 layers and 110 million parameters.
"""

questions = [
    "When was the Transformer introduced?",
    "Who wrote the Attention Is All You Need paper?",
    "How many layers does BERT have?",
    "What did the Transformer replace?",
    "How many parameters does BERT have?",
]

for question in questions:
    result = qa_pipeline(question=question, context=context)
    print(f"Q: {question}")
    print(f"A: {result['answer']}  (confidence: {result['score']:.4f})")
    print()
```

### Code: Custom QA with Score Analysis

```python
import torch
from transformers import BertTokenizer, BertForQuestionAnswering

tokenizer = BertTokenizer.from_pretrained("deepset/bert-base-cased-squad2")
model = BertForQuestionAnswering.from_pretrained("deepset/bert-base-cased-squad2")
model.eval()

question = "What is the capital of France?"
context = "France is a country in Western Europe. Its capital is Paris, which is known for the Eiffel Tower."

inputs = tokenizer(question, context, return_tensors="pt")

with torch.no_grad():
    outputs = model(**inputs)
    start_logits = outputs.start_logits[0]
    end_logits = outputs.end_logits[0]

# Get top-3 start and end positions
start_top3 = torch.topk(start_logits, 3)
end_top3 = torch.topk(end_logits, 3)

tokens = tokenizer.convert_ids_to_tokens(inputs["input_ids"][0])

print("Top-3 start positions:")
for score, idx in zip(start_top3.values, start_top3.indices):
    print(f"  Position {idx.item():3d} ({tokens[idx]:10s}): score = {score:.3f}")

print("\nTop-3 end positions:")
for score, idx in zip(end_top3.values, end_top3.indices):
    print(f"  Position {idx.item():3d} ({tokens[idx]:10s}): score = {score:.3f}")

# Extract best answer
start_idx = start_logits.argmax().item()
end_idx = end_logits.argmax().item()
answer_tokens = inputs["input_ids"][0][start_idx:end_idx + 1]
answer = tokenizer.decode(answer_tokens)
print(f"\nAnswer: {answer}")
```

---

## 5. Semantic Similarity and Sentence Embeddings

### Approaches to Sentence Similarity

```
  Method 1: [CLS] Token (Naive)
  ─────────────────────────────
  Sentence A → BERT → h_[CLS]_A
  Sentence B → BERT → h_[CLS]_B
  Similarity = cosine(h_[CLS]_A, h_[CLS]_B)
  ⚠ Problem: [CLS] not optimized for similarity without fine-tuning

  Method 2: Mean Pooling (Better)
  ───────────────────────────────
  Sentence A → BERT → mean(all token outputs) → emb_A
  Sentence B → BERT → mean(all token outputs) → emb_B
  Similarity = cosine(emb_A, emb_B)

  Method 3: Sentence-BERT (Best)
  ──────────────────────────────
  Uses siamese network architecture with fine-tuned BERT:

  Sentence A ──→ [BERT] ──→ Mean Pool ──→ emb_A ──┐
                                                    ├──→ cosine similarity
  Sentence B ──→ [BERT] ──→ Mean Pool ──→ emb_B ──┘
  (shared weights)

  Trained with contrastive or triplet loss to produce meaningful distances.
```

### Cosine Similarity Formula

```
cosine(A, B) = (A · B) / (‖A‖ × ‖B‖)

            = Σᵢ(Aᵢ × Bᵢ) / (√Σᵢ Aᵢ² × √Σᵢ Bᵢ²)

Range: [-1, 1]
  1  = identical direction (most similar)
  0  = orthogonal (unrelated)
 -1  = opposite direction (most dissimilar)
```

### Code: Sentence Similarity with Sentence-Transformers

```python
from sentence_transformers import SentenceTransformer, util

model = SentenceTransformer("all-MiniLM-L6-v2")  # BERT-based, optimized

sentences = [
    "The cat sits on the mat.",
    "A feline is resting on a rug.",
    "Dogs are loyal pets.",
    "Machine learning is a subset of artificial intelligence.",
    "AI includes techniques like deep learning.",
]

# Encode all sentences to dense vectors
embeddings = model.encode(sentences, convert_to_tensor=True)

print("Pairwise Cosine Similarities:")
print("-" * 55)
for i in range(len(sentences)):
    for j in range(i + 1, len(sentences)):
        sim = util.cos_sim(embeddings[i], embeddings[j]).item()
        s_i = sentences[i][:25]
        s_j = sentences[j][:25]
        bar = "█" * int(sim * 20) if sim > 0 else ""
        print(f"  ({i},{j}) {s_i:26s} ↔ {s_j:26s}: {sim:.4f} {bar}")
```

### Code: Mean Pooling from Scratch

```python
import torch
from transformers import BertTokenizer, BertModel

tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")
model = BertModel.from_pretrained("bert-base-uncased")
model.eval()

def mean_pooling(model_output, attention_mask):
    """Average token embeddings, weighted by attention mask."""
    token_embeddings = model_output.last_hidden_state    # (batch, seq, 768)
    mask_expanded = attention_mask.unsqueeze(-1).float()  # (batch, seq, 1)
    sum_embeddings = (token_embeddings * mask_expanded).sum(dim=1)
    sum_mask = mask_expanded.sum(dim=1).clamp(min=1e-9)
    return sum_embeddings / sum_mask                      # (batch, 768)

def get_embedding(text):
    inputs = tokenizer(text, return_tensors="pt", padding=True, truncation=True)
    with torch.no_grad():
        outputs = model(**inputs)
    return mean_pooling(outputs, inputs["attention_mask"])

# Compare two sentences
emb_a = get_embedding("The weather is sunny today.")
emb_b = get_embedding("It is a bright and clear day.")
emb_c = get_embedding("Stock prices dropped sharply.")

cos = torch.nn.CosineSimilarity(dim=-1)
print(f"Sunny ↔ Bright: {cos(emb_a, emb_b).item():.4f}")  # High similarity
print(f"Sunny ↔ Stocks: {cos(emb_a, emb_c).item():.4f}")  # Low similarity
```

---

## 6. Search and Ranking

### Bi-Encoder vs. Cross-Encoder

```
  Bi-Encoder (Two-Tower):
  ════════════════════════
  ┌────────┐     ┌────────┐
  │ Query  │     │  Doc   │     Encode independently
  │  BERT  │     │  BERT  │     (shared or separate weights)
  └───┬────┘     └───┬────┘
      ↓              ↓
   q_embed        d_embed       768-dim vectors
      ↓              ↓
      └──── cosine ──┘           Fast dot-product similarity
            score

  ✓ Pre-compute document embeddings offline
  ✓ Fast retrieval (milliseconds for millions of docs)
  ✗ Less accurate (no cross-attention between query and doc)

  Cross-Encoder:
  ══════════════
  ┌────────────────────────┐
  │ [CLS] Query [SEP] Doc │     Encode jointly
  │        BERT            │     Full cross-attention
  └──────────┬─────────────┘
             ↓
        h_[CLS] → Linear → score

  ✓ More accurate (full interaction between query and doc)
  ✗ Slow (must run BERT for every query-doc pair)
  ✗ Cannot pre-compute; O(n) inference per query

  Production Strategy: Two-Stage Pipeline
  ═══════════════════════════════════════
  Stage 1 (Retrieval):  Bi-encoder retrieves top-100 candidates
  Stage 2 (Re-ranking): Cross-encoder re-ranks top-100 for precision

  Query → [Bi-Encoder] → Top 100 docs → [Cross-Encoder] → Top 10 docs
          (fast, broad)                   (slow, precise)
```

### Code: Semantic Search with Bi-Encoder

```python
from sentence_transformers import SentenceTransformer, util
import torch

model = SentenceTransformer("all-MiniLM-L6-v2")

# Document corpus
documents = [
    "Python is a high-level programming language.",
    "Machine learning uses statistical methods to learn from data.",
    "The Eiffel Tower is located in Paris, France.",
    "Deep learning is a subset of machine learning using neural networks.",
    "JavaScript is used for web development.",
    "Natural language processing deals with text and speech.",
    "The Great Wall of China is visible from space.",
    "BERT is a transformer model for NLP tasks.",
]

# Pre-compute document embeddings (do this ONCE)
doc_embeddings = model.encode(documents, convert_to_tensor=True)

def search(query, top_k=3):
    query_embedding = model.encode(query, convert_to_tensor=True)
    scores = util.cos_sim(query_embedding, doc_embeddings)[0]
    top_results = torch.topk(scores, k=top_k)

    print(f"\nQuery: '{query}'")
    print("-" * 60)
    for score, idx in zip(top_results.values, top_results.indices):
        print(f"  Score: {score:.4f} | {documents[idx]}")

search("What is deep learning?")
search("Tell me about famous landmarks")
search("How to build websites")
```

### Code: Cross-Encoder Re-Ranking

```python
from sentence_transformers import CrossEncoder

cross_encoder = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")

query = "What is machine learning?"
candidates = [
    "Machine learning uses statistical methods to learn from data.",
    "The Eiffel Tower is located in Paris, France.",
    "Deep learning is a subset of machine learning using neural networks.",
    "JavaScript is used for web development.",
    "BERT is a transformer model for NLP tasks.",
]

# Score each query-document pair
pairs = [[query, doc] for doc in candidates]
scores = cross_encoder.predict(pairs)

# Rank by score
ranked = sorted(zip(scores, candidates), reverse=True)

print(f"Query: '{query}'\n")
for rank, (score, doc) in enumerate(ranked, 1):
    print(f"  Rank {rank}: Score={score:.4f} | {doc}")
```

---

## 7. Performance Benchmarks

### GLUE Benchmark Results

```
  GLUE (General Language Understanding Evaluation)
  ════════════════════════════════════════════════

  Task   | Metric   | BERT-base | BERT-large | RoBERTa | Human
  ─────────────────────────────────────────────────────────────
  MNLI   | Accuracy | 84.6      | 86.7       | 90.2    | 92.0
  QQP    | F1       | 71.2      | 72.1       | 74.3    | 86.0
  QNLI   | Accuracy | 90.5      | 92.7       | 94.7    | 91.2
  SST-2  | Accuracy | 93.5      | 94.9       | 96.4    | 97.8
  CoLA   | Matthews | 52.1      | 60.5       | 68.0    | 66.4
  STS-B  | Pearson  | 85.8      | 86.5       | 92.4    | 92.7
  MRPC   | F1       | 88.9      | 89.3       | 90.9    | 86.3
  RTE    | Accuracy | 66.4      | 70.1       | 86.6    | 93.6
  ─────────────────────────────────────────────────────────────
  Avg    |          | 79.6      | 82.1       | 88.5    |
```

### SQuAD Benchmark Results

```
  SQuAD (Stanford Question Answering Dataset)
  ════════════════════════════════════════════

  Model        | SQuAD 1.1 (EM/F1) | SQuAD 2.0 (EM/F1)
  ──────────────────────────────────────────────────────
  BERT-base    | 80.8 / 88.5        | 73.7 / 76.3
  BERT-large   | 84.1 / 90.9        | 78.7 / 81.9
  RoBERTa-large| 88.9 / 94.6        | 86.5 / 89.4
  Human        | 82.3 / 91.2        | 86.8 / 89.5

  EM = Exact Match, F1 = token-level F1 score
```

### NER Benchmark (CoNLL-2003)

```
  Model        | Precision | Recall | F1
  ──────────────────────────────────────
  BERT-base    | 91.2      | 91.8   | 91.5
  BERT-large   | 92.1      | 92.5   | 92.3
  RoBERTa-large| 92.8      | 93.1   | 92.9
  Human        | ~97       | ~97    | ~97
```

---

## 8. Production Deployment Considerations

### Optimization Techniques

```
  ┌─────────────────────────────────────────────────────────────┐
  │              PRODUCTION OPTIMIZATION STACK                   │
  │                                                             │
  │  Level 1: Model Selection                                   │
  │    DistilBERT or TinyBERT for latency-sensitive apps        │
  │                                                             │
  │  Level 2: Quantization                                      │
  │    FP32 → INT8: 4× smaller, 2-3× faster, ~1% accuracy drop │
  │                                                             │
  │  Level 3: Pruning                                           │
  │    Remove unnecessary attention heads (30-40% can be pruned)│
  │                                                             │
  │  Level 4: ONNX Runtime / TensorRT                           │
  │    Optimized inference engines: 2-5× speedup                │
  │                                                             │
  │  Level 5: Caching                                           │
  │    Cache embeddings for repeated inputs                     │
  │    Pre-compute document embeddings for search               │
  └─────────────────────────────────────────────────────────────┘
```

### Code: ONNX Export for Production

```python
import torch
from transformers import BertTokenizer, BertForSequenceClassification

model_name = "bert-base-uncased"
tokenizer = BertTokenizer.from_pretrained(model_name)
model = BertForSequenceClassification.from_pretrained(model_name, num_labels=2)
model.eval()

# Create dummy input for tracing
dummy_input = tokenizer("Example text for export", return_tensors="pt")

torch.onnx.export(
    model,
    (dummy_input["input_ids"], dummy_input["attention_mask"]),
    "bert_classifier.onnx",
    input_names=["input_ids", "attention_mask"],
    output_names=["logits"],
    dynamic_axes={
        "input_ids": {0: "batch", 1: "sequence"},
        "attention_mask": {0: "batch", 1: "sequence"},
        "logits": {0: "batch"},
    },
    opset_version=14,
)
print("Model exported to bert_classifier.onnx")
```

### Code: Quantization

```python
import torch
from transformers import BertModel, BertTokenizer

model = BertModel.from_pretrained("bert-base-uncased")
tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")

# Dynamic quantization (easiest, CPU-only)
quantized_model = torch.quantization.quantize_dynamic(
    model,
    {torch.nn.Linear},    # quantize linear layers
    dtype=torch.qint8,
)

# Compare sizes
import os

torch.save(model.state_dict(), "bert_fp32.pt")
torch.save(quantized_model.state_dict(), "bert_int8.pt")

fp32_size = os.path.getsize("bert_fp32.pt") / 1e6
int8_size = os.path.getsize("bert_int8.pt") / 1e6
print(f"FP32 model: {fp32_size:.1f} MB")
print(f"INT8 model: {int8_size:.1f} MB")
print(f"Compression: {fp32_size / int8_size:.1f}×")

# Clean up
os.remove("bert_fp32.pt")
os.remove("bert_int8.pt")
```

---

## 9. End-to-End Multi-Task Pipeline

```python
from transformers import pipeline

# Initialize multiple BERT-powered pipelines
sentiment = pipeline("sentiment-analysis")
ner = pipeline("ner", aggregation_strategy="simple")
qa = pipeline("question-answering")
similarity = pipeline("zero-shot-classification")

def analyze_text(text, context=None, categories=None):
    """Run multiple BERT analyses on a given text."""
    results = {}

    # Sentiment
    sent = sentiment(text[:512])[0]
    results["sentiment"] = f"{sent['label']} ({sent['score']:.2%})"

    # Named entities
    entities = ner(text[:512])
    results["entities"] = [
        f"{e['entity_group']}: {e['word']}" for e in entities
    ]

    # Question answering (if context provided)
    if context:
        answer = qa(question=text, context=context)
        results["answer"] = f"{answer['answer']} ({answer['score']:.2%})"

    # Zero-shot classification (if categories provided)
    if categories:
        cls = similarity(text, categories)
        results["category"] = f"{cls['labels'][0]} ({cls['scores'][0]:.2%})"

    return results

# Demo
text = "Apple CEO Tim Cook announced new AI features at WWDC in Cupertino."
categories = ["Technology", "Sports", "Politics", "Entertainment"]

results = analyze_text(text, categories=categories)

print(f"Text: {text}\n")
for key, value in results.items():
    if isinstance(value, list):
        print(f"  {key}:")
        for item in value:
            print(f"    - {item}")
    else:
        print(f"  {key}: {value}")
```

---

## 10. Real-World Applications Summary

| Industry          | Application                  | BERT Model Used            | Impact                    |
|-------------------|------------------------------|----------------------------|---------------------------|
| **Search**        | Query understanding          | BERT / RoBERTa             | 10% relevance improvement |
| **Healthcare**    | Clinical NER                 | BioBERT / ClinicalBERT     | Automated record parsing  |
| **Finance**       | Sentiment from news          | FinBERT                    | Trading signal generation |
| **Legal**         | Contract analysis            | LegalBERT                  | Clause extraction         |
| **E-commerce**    | Product search               | Sentence-BERT              | Semantic product matching |
| **Customer Support** | Intent classification     | DistilBERT                 | Automated ticket routing  |
| **Social Media**  | Hate speech detection        | BERT + fine-tuning         | Content moderation        |
| **Education**     | Automated grading            | BERT + regression head     | Essay scoring             |

---

## 11. Summary Table

| Concept                    | Key Details                                                |
|----------------------------|------------------------------------------------------------|
| Sentiment analysis         | [CLS] → linear → softmax; simple and effective             |
| Named entity recognition   | Per-token classification with BIO tagging scheme           |
| Question answering         | Predict start/end positions in context passage             |
| Semantic similarity        | Mean pooling or Sentence-BERT for dense embeddings         |
| Bi-encoder search          | Pre-compute doc embeddings; fast cosine retrieval          |
| Cross-encoder ranking      | Joint encoding; accurate but slow; used for re-ranking     |
| Production optimization    | Quantization, pruning, ONNX, distillation                  |
| Domain-specific BERTs      | BioBERT, FinBERT, LegalBERT, SciBERT                      |
| Benchmark (GLUE)           | BERT-base: 79.6 avg, RoBERTa: 88.5 avg                    |
| Two-stage search           | Bi-encoder retrieval → cross-encoder re-ranking            |

---

## 12. Revision Questions

1. **Explain the difference between bi-encoder and cross-encoder for search. When would you use each approach, and why is a two-stage pipeline common in production?**

2. **How does BERT handle extractive question answering? What happens when the answer is not present in the given context?**

3. **Compare three methods for computing sentence similarity with BERT: [CLS] token, mean pooling, and Sentence-BERT. Which is best and why?**

4. **Describe how you would deploy a BERT-based sentiment analysis system for a high-traffic production API. What optimization techniques would you apply?**

5. **Why do domain-specific BERT variants (BioBERT, FinBERT) outperform general BERT on domain tasks? How are they created?**

6. **Given a dataset of 1 million product descriptions, design a semantic search system using BERT. Describe the indexing and query pipeline, including latency considerations.**

---

[← BERT Variants](04-bert-variants.md) | [GPT Architecture →](../08-GPT-and-Decoder-Models/01-gpt-architecture.md)
