# Semantic Similarity

## Overview

Semantic similarity measures how similar two pieces of text are in **meaning**, not just surface form. "Car" and "automobile" are semantically similar despite sharing no characters. This capability underpins search engines, recommendation systems, duplicate detection, and retrieval-augmented generation.

---

## Levels of Similarity

```
Surface similarity (string matching):
  "machine learning" vs "machine learning"  → identical
  "machine learning" vs "deep learning"     → 1 word overlap
  "machine learning" vs "ML"                → 0 overlap

Semantic similarity (meaning):
  "machine learning" vs "ML"                → very similar
  "car" vs "automobile"                     → very similar
  "happy" vs "joyful"                       → very similar
  "bank" (river) vs "bank" (financial)      → different!
```

---

## Word-Level Similarity

```
Using word embeddings:

  cos_sim(w1, w2) = (v_w1 · v_w2) / (||v_w1|| × ||v_w2||)

  Examples (Word2Vec):
    sim("king", "queen")     = 0.85
    sim("cat", "dog")        = 0.76
    sim("car", "automobile") = 0.83
    sim("car", "banana")     = 0.12
    sim("happy", "sad")      = 0.55  ← related but opposite
```

---

## Sentence-Level Similarity

```
Methods for computing sentence similarity:

1. Average word embeddings:
   sent_vec = mean(word_vectors)
   Simple but loses word order

2. TF-IDF weighted average:
   sent_vec = Σ tfidf(w) × vec(w) / Σ tfidf(w)
   Better, emphasizes important words

3. Sentence transformers (SBERT):
   Use a transformer to encode entire sentence into one vector
   Best quality, captures context and meaning

Architecture:
  ┌────────────────┐     ┌────────────────┐
  │   Sentence A    │     │   Sentence B    │
  └────────┬───────┘     └────────┬───────┘
           ↓                      ↓
  ┌────────────────┐     ┌────────────────┐
  │  BERT Encoder   │     │  BERT Encoder   │
  │  (shared weights)│     │  (shared weights)│
  └────────┬───────┘     └────────┬───────┘
           ↓                      ↓
  ┌────────────────┐     ┌────────────────┐
  │  Mean Pooling   │     │  Mean Pooling   │
  └────────┬───────┘     └────────┬───────┘
           ↓                      ↓
        vec_A                  vec_B
           └──────┬───────────┘
                  ↓
         cosine_similarity(vec_A, vec_B)
```

---

## Sentence Transformers (SBERT)

```python
from sentence_transformers import SentenceTransformer, util

model = SentenceTransformer("all-MiniLM-L6-v2")

sentences = [
    "Machine learning is a branch of AI",
    "ML is a subset of artificial intelligence",
    "I love eating pizza on Fridays",
    "Deep learning uses neural networks",
    "The weather is sunny today",
]

embeddings = model.encode(sentences)

# Pairwise similarity matrix
for i in range(len(sentences)):
    for j in range(i+1, len(sentences)):
        sim = util.cos_sim(embeddings[i], embeddings[j]).item()
        print(f"  sim({i},{j}) = {sim:.3f}  | '{sentences[i][:35]}' vs '{sentences[j][:35]}'")

# Expected output:
#   sim(0,1) = 0.89  | 'ML is subset of AI' vs 'Machine learning is branch of AI'
#   sim(0,2) = 0.05  | 'ML...' vs 'pizza...'
#   sim(0,3) = 0.72  | 'ML...' vs 'Deep learning...'
```

---

## Semantic Search

```python
from sentence_transformers import SentenceTransformer, util
import torch

model = SentenceTransformer("all-MiniLM-L6-v2")

# Document corpus
corpus = [
    "Python is a popular programming language",
    "Machine learning enables computers to learn from data",
    "The Eiffel Tower is located in Paris, France",
    "Neural networks are inspired by biological brains",
    "Climate change affects global weather patterns",
    "Deep learning is a subset of machine learning",
]
corpus_embeddings = model.encode(corpus, convert_to_tensor=True)

# Semantic search
query = "How do computers learn?"
query_embedding = model.encode(query, convert_to_tensor=True)

scores = util.cos_sim(query_embedding, corpus_embeddings)[0]
top_results = torch.topk(scores, k=3)

print(f"Query: '{query}'")
for score, idx in zip(top_results.values, top_results.indices):
    print(f"  {score:.3f}: {corpus[idx]}")
# 0.72: Machine learning enables computers to learn from data
# 0.58: Deep learning is a subset of machine learning
# 0.45: Neural networks are inspired by biological brains
```

---

## Similarity Tasks & Datasets

| Task | Description | Dataset |
|------|-------------|---------|
| STS (Semantic Textual Similarity) | Rate similarity 0-5 | STS Benchmark |
| Paraphrase detection | Are sentences paraphrases? | MRPC, QQP |
| Natural Language Inference | Entailment/contradiction/neutral | SNLI, MultiNLI |
| Duplicate question detection | Same question asked differently? | Quora Question Pairs |
| Semantic search | Find relevant documents | MS MARCO |

---

## Model Comparison

| Model | Dimensions | Speed | Quality | Use Case |
|-------|:-:|:-:|:-:|---------|
| TF-IDF + cosine | Sparse | Very fast | Low | Quick baseline |
| Word2Vec average | 300 | Fast | Medium | Simple tasks |
| SBERT (MiniLM) | 384 | Fast | Good | Production search |
| SBERT (mpnet-base) | 768 | Medium | Best | High accuracy needed |
| OpenAI embeddings | 1536 | API call | Very good | Cloud deployment |
| E5 / BGE | 768-1024 | Medium | State-of-art | Best open-source |

---

## Revision Questions

1. **What is the difference between surface similarity and semantic similarity?**
2. **How does SBERT encode sentences into vectors?**
3. **Why is average word embedding a poor method for sentence similarity?**
4. **How does semantic search work using sentence embeddings?**
5. **What is the STS Benchmark and how is it used?**

---

[Previous: 01-coreference-resolution.md](01-coreference-resolution.md) | [Next: 03-sentiment-analysis.md](03-sentiment-analysis.md)
