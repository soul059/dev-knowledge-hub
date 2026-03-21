# Open-Domain Question Answering

## Overview

Open-Domain QA (ODQA) answers questions using a large corpus (e.g., all of Wikipedia) rather than a single given passage. The system must first **retrieve** relevant documents and then **read** them to extract or generate an answer. This is the foundation for modern AI assistants and search engines.

---

## Architecture: Retriever-Reader Pipeline

```
User Question: "What year did the Titanic sink?"
         │
         ▼
┌─────────────────────┐
│     RETRIEVER        │  Find relevant documents from corpus
│  (BM25 / DPR)       │
└─────────┬───────────┘
          │  Top-k passages
          ▼
┌─────────────────────┐
│      READER          │  Extract answer from retrieved passages
│  (BERT / RoBERTa)    │
└─────────┬───────────┘
          │
          ▼
    Answer: "1912"

Corpus: 21M+ Wikipedia passages (100-word chunks)
```

---

## Retriever: Finding Relevant Passages

### Sparse Retrieval (BM25)

```
BM25 scoring:
  score(q, d) = Σ IDF(q_i) × [tf(q_i, d) × (k₁+1)] / 
                               [tf(q_i, d) + k₁ × (1 - b + b × |d|/avgdl)]

  Where:
    q_i    = query term
    tf     = term frequency in document
    IDF    = inverse document frequency
    |d|    = document length
    avgdl  = average document length
    k₁, b  = tuning parameters (typically k₁=1.2, b=0.75)

  Pros: Fast, no training needed, strong baseline
  Cons: Exact match only, misses synonyms
```

### Dense Retrieval (DPR)

```
Dense Passage Retrieval:
  Encode question and passage into dense vectors using BERT

  q_vec = BERT_Q(question)      # 768-dim vector
  p_vec = BERT_P(passage)       # 768-dim vector

  similarity = dot(q_vec, p_vec)

  Training: Contrastive learning
    - Positive pairs: (question, gold passage)
    - Negative pairs: (question, random/hard passages)
    - Loss: maximize similarity for positives, minimize for negatives

  At inference:
    1. Pre-encode all 21M passages → FAISS index
    2. Encode question → single vector
    3. Nearest neighbor search → top-k passages
    4. ~50ms retrieval from 21M passages!
```

---

## Reader: Extracting the Answer

```python
from transformers import pipeline

# The reader takes retrieved passages and extracts answers
reader = pipeline("question-answering", model="deepset/roberta-base-squad2")

retrieved_passages = [
    "The RMS Titanic sank on 15 April 1912 after striking an iceberg.",
    "The Titanic was a British passenger liner operated by the White Star Line.",
    "The sinking resulted in the deaths of more than 1,500 people.",
]

question = "What year did the Titanic sink?"

best_answer = None
best_score = -1

for passage in retrieved_passages:
    result = reader(question=question, context=passage)
    if result["score"] > best_score:
        best_score = result["score"]
        best_answer = result["answer"]

print(f"Answer: {best_answer} (score: {best_score:.4f})")
# Answer: 1912 (score: 0.9876)
```

---

## Full ODQA Pipeline with Haystack

```python
from haystack.document_stores import FAISSDocumentStore
from haystack.nodes import DensePassageRetriever, FARMReader
from haystack.pipelines import ExtractiveQAPipeline

# 1. Document Store
document_store = FAISSDocumentStore(
    embedding_dim=768,
    faiss_index_factory_str="Flat"
)

# 2. Index documents
docs = [{"content": text} for text in wikipedia_passages]
document_store.write_documents(docs)

# 3. Retriever
retriever = DensePassageRetriever(
    document_store=document_store,
    query_embedding_model="facebook/dpr-question_encoder-single-nq-base",
    passage_embedding_model="facebook/dpr-ctx_encoder-single-nq-base"
)
document_store.update_embeddings(retriever)

# 4. Reader
reader = FARMReader(model_name_or_path="deepset/roberta-base-squad2")

# 5. Pipeline
pipeline = ExtractiveQAPipeline(reader, retriever)
result = pipeline.run(query="When was Python created?", params={"Retriever": {"top_k": 5}})
print(result["answers"][0].answer)
```

---

## Retriever Comparison

| Method | Speed | Accuracy | Training | Synonym Handling |
|--------|:---:|:---:|:---:|:---:|
| BM25 (sparse) | Very fast | Good baseline | None | ✗ |
| DPR (dense) | Fast (FAISS) | Better | Required | ✓ |
| ColBERT | Medium | Best | Required | ✓ |
| Hybrid (BM25+DPR) | Medium | Best overall | Partial | ✓ |

---

## RAG: Retrieval-Augmented Generation

```
RAG combines retrieval with generation:

  Question → Retriever → Top-k passages → Generator (BART/T5) → Answer

  Unlike extractive QA:
    - Can synthesize information from multiple passages
    - Can generate fluent natural language answers
    - Can say "I don't know" naturally

  RAG = ODQA + Language Generation
  Used by: ChatGPT (with browsing), Bing Chat, Perplexity AI
```

---

## Key ODQA Benchmarks

| Dataset | Corpus | Questions | Answer Type |
|---------|--------|:-:|:-:|
| Natural Questions | Wikipedia | 300K | Short/long span |
| TriviaQA | Web + Wikipedia | 95K | Span |
| WebQuestions | Freebase | 5.8K | Entity |
| SQuAD-Open | Wikipedia (full) | 10K | Span |
| EfficientQA | Wikipedia | 1.7K | Short answer |

---

## Revision Questions

1. **What is the difference between open-domain QA and reading comprehension?**
2. **How does BM25 retrieval work and what are its limitations?**
3. **How does Dense Passage Retrieval (DPR) improve over BM25?**
4. **What is the role of FAISS in open-domain QA?**
5. **How does RAG differ from a standard retriever-reader pipeline?**

---

[Previous: 03-extractive-qa.md](03-extractive-qa.md) | [Next: 05-knowledge-based-qa.md](05-knowledge-based-qa.md)
