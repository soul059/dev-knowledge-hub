# Retrieval-Augmented Generation (RAG)

## Overview

Retrieval-Augmented Generation (RAG) combines the power of large language models with external knowledge retrieval, allowing models to access up-to-date, domain-specific information without retraining. Instead of relying solely on knowledge encoded in model parameters during pre-training, RAG systems **retrieve** relevant documents at inference time and **augment** the prompt with that context before **generating** a response. This paradigm addresses key LLM limitations: hallucination, knowledge cutoffs, and lack of domain-specific expertise.

```
┌─────────────────────────────────────────────────────────────────┐
│                    RAG PIPELINE OVERVIEW                        │
│                                                                 │
│   User Query                                                    │
│       │                                                         │
│       ▼                                                         │
│   ┌─────────┐    ┌──────────────┐    ┌─────────────────┐       │
│   │ Embed   │───▶│ Vector Store │───▶│ Top-K Retrieved │       │
│   │ Query   │    │  (Search)    │    │   Documents     │       │
│   └─────────┘    └──────────────┘    └────────┬────────┘       │
│                                               │                 │
│                                               ▼                 │
│   ┌──────────────────────────────────────────────────┐         │
│   │        Augmented Prompt                          │         │
│   │  ┌─────────────────────────────────────────┐     │         │
│   │  │ System: You are a helpful assistant.    │     │         │
│   │  │ Context: [Retrieved Doc 1]              │     │         │
│   │  │          [Retrieved Doc 2]              │     │         │
│   │  │          [Retrieved Doc 3]              │     │         │
│   │  │ Question: {user_query}                  │     │         │
│   │  └─────────────────────────────────────────┘     │         │
│   └──────────────────────┬───────────────────────────┘         │
│                          │                                      │
│                          ▼                                      │
│                   ┌────────────┐                                │
│                   │    LLM     │                                │
│                   │ Generation │                                │
│                   └─────┬──────┘                                │
│                         │                                       │
│                         ▼                                       │
│                  Grounded Answer                                │
│             (with source citations)                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Why RAG?

### Problems RAG Solves

| Problem | Without RAG | With RAG |
|---------|-------------|----------|
| **Knowledge cutoff** | Model only knows training data | Retrieves current information |
| **Hallucination** | May fabricate plausible-sounding facts | Grounds answers in real documents |
| **Domain expertise** | General knowledge only | Access to specialized corpora |
| **Verifiability** | No sources provided | Can cite retrieved documents |
| **Cost of updates** | Retrain entire model ($$$) | Update document store (cheap) |
| **Privacy** | Data baked into model weights | Documents stay in your control |

---

## RAG Architecture Components

### 1. Document Ingestion Pipeline

Before retrieval can happen, documents must be processed and indexed.

```
┌──────────┐    ┌───────────┐    ┌──────────┐    ┌──────────────┐
│ Raw Docs │───▶│  Chunking │───▶│ Embedding│───▶│ Vector Store │
│ (PDF,    │    │ (Split    │    │ (Convert │    │ (Index &     │
│  HTML,   │    │  into     │    │  to dense │    │  Store)      │
│  TXT...) │    │  pieces)  │    │  vectors) │    │              │
└──────────┘    └───────────┘    └──────────┘    └──────────────┘
```

### 2. Chunking Strategies

Chunking determines how documents are split into retrievable units:

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Strategy 1: Fixed-size chunks with overlap
splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,        # characters per chunk
    chunk_overlap=50,      # overlap between chunks
    separators=["\n\n", "\n", ". ", " ", ""]
)

text = """Machine learning is a subset of artificial intelligence.
It focuses on building systems that learn from data.

Deep learning uses neural networks with many layers.
These networks can learn complex patterns automatically."""

chunks = splitter.split_text(text)
for i, chunk in enumerate(chunks):
    print(f"Chunk {i}: {chunk[:80]}...")
```

**Chunking Strategy Comparison:**

| Strategy | Chunk Size | Overlap | Best For |
|----------|-----------|---------|----------|
| Fixed-size | 256-1024 tokens | 10-20% | General purpose |
| Sentence-based | 3-5 sentences | 1 sentence | QA systems |
| Paragraph-based | Natural breaks | None | Structured docs |
| Semantic | Variable | Meaning-based | Research papers |
| Recursive | Hierarchical | Configurable | Mixed content |

### 3. Embedding Models

Embedding models convert text chunks into dense vector representations for similarity search.

```python
from sentence_transformers import SentenceTransformer
import numpy as np

# Load embedding model
model = SentenceTransformer('all-MiniLM-L6-v2')  # 384-dim, fast
# Alternative: 'all-mpnet-base-v2' (768-dim, more accurate)

# Embed documents
documents = [
    "Transformers use self-attention mechanisms",
    "LSTM networks process sequences step by step",
    "Convolutional networks excel at image recognition",
    "RAG combines retrieval with generation"
]

doc_embeddings = model.encode(documents)
print(f"Embedding shape: {doc_embeddings.shape}")  # (4, 384)

# Embed query
query = "How do attention mechanisms work?"
query_embedding = model.encode([query])

# Compute cosine similarity
from numpy.linalg import norm

similarities = np.dot(doc_embeddings, query_embedding.T).flatten()
similarities /= (norm(doc_embeddings, axis=1) * norm(query_embedding))

# Rank results
ranked = sorted(enumerate(similarities), key=lambda x: x[1], reverse=True)
for idx, score in ranked:
    print(f"Score: {score:.4f} | {documents[idx]}")
```

**Popular Embedding Models:**

| Model | Dimensions | Speed | Quality | Use Case |
|-------|-----------|-------|---------|----------|
| all-MiniLM-L6-v2 | 384 | ⚡ Fast | Good | Prototyping |
| all-mpnet-base-v2 | 768 | Medium | Better | Production |
| text-embedding-3-small (OpenAI) | 1536 | API | Very Good | Cloud apps |
| text-embedding-3-large (OpenAI) | 3072 | API | Best | High accuracy |
| BGE-large-en | 1024 | Medium | Excellent | Open-source prod |
| E5-mistral-7b | 4096 | Slow | State-of-art | Research |

### 4. Vector Databases

Vector databases store embeddings and enable fast similarity search.

```python
import chromadb

# Initialize ChromaDB (lightweight, local)
client = chromadb.Client()
collection = client.create_collection(
    name="ml_docs",
    metadata={"hnsw:space": "cosine"}  # distance metric
)

# Add documents
collection.add(
    documents=[
        "Backpropagation computes gradients using the chain rule",
        "Batch normalization stabilizes training of deep networks",
        "Dropout randomly deactivates neurons during training",
        "Learning rate scheduling adjusts LR during training"
    ],
    ids=["doc1", "doc2", "doc3", "doc4"],
    metadatas=[
        {"topic": "optimization"},
        {"topic": "normalization"},
        {"topic": "regularization"},
        {"topic": "optimization"}
    ]
)

# Query
results = collection.query(
    query_texts=["How to prevent overfitting?"],
    n_results=2
)

print("Retrieved documents:")
for doc, dist in zip(results['documents'][0], results['distances'][0]):
    print(f"  Distance: {dist:.4f} | {doc}")
```

**Vector Database Comparison:**

| Database | Type | Scale | Features | Best For |
|----------|------|-------|----------|----------|
| ChromaDB | Embedded | Small-Med | Simple API, local | Prototyping |
| FAISS | Library | Large | GPU support, fast | Research |
| Pinecone | Cloud | Any | Managed, scalable | Production SaaS |
| Weaviate | Self-hosted/Cloud | Large | Hybrid search | Enterprise |
| Qdrant | Self-hosted/Cloud | Large | Filtering, fast | Production |
| pgvector | PostgreSQL ext. | Medium | SQL integration | Existing Postgres |

---

## Complete RAG Pipeline

### End-to-End Implementation

```python
from sentence_transformers import SentenceTransformer
import chromadb
import openai  # or any LLM API

# ============================================================
# STEP 1: INGESTION (run once)
# ============================================================

# Load and chunk documents
def chunk_documents(texts, chunk_size=500, overlap=50):
    chunks = []
    for text in texts:
        for i in range(0, len(text), chunk_size - overlap):
            chunk = text[i:i + chunk_size]
            if len(chunk) > 50:  # skip tiny fragments
                chunks.append(chunk)
    return chunks

# Sample knowledge base
knowledge_base = [
    """Gradient descent is an optimization algorithm used to minimize 
    the loss function. It updates parameters in the direction of the 
    negative gradient. The learning rate controls the step size. 
    Variants include SGD, Adam, and RMSprop.""",
    
    """Regularization techniques prevent overfitting. L1 regularization 
    adds absolute weight penalties and encourages sparsity. L2 
    regularization adds squared weight penalties and encourages small 
    weights. Dropout randomly deactivates neurons during training.""",
    
    """Batch normalization normalizes layer inputs to have zero mean 
    and unit variance. It reduces internal covariate shift and allows 
    higher learning rates. Applied before or after activation functions."""
]

chunks = chunk_documents(knowledge_base)

# Embed and store
embedder = SentenceTransformer('all-MiniLM-L6-v2')
client = chromadb.Client()
collection = client.create_collection("ml_knowledge")

collection.add(
    documents=chunks,
    ids=[f"chunk_{i}" for i in range(len(chunks))],
    embeddings=embedder.encode(chunks).tolist()
)

# ============================================================
# STEP 2: RETRIEVAL + GENERATION (run per query)
# ============================================================

def rag_query(question, n_results=3):
    # Retrieve relevant chunks
    results = collection.query(
        query_embeddings=embedder.encode([question]).tolist(),
        n_results=n_results
    )
    
    # Build augmented prompt
    context = "\n\n".join(results['documents'][0])
    
    prompt = f"""Answer the question based on the provided context. 
If the context doesn't contain enough information, say so.
Cite which parts of the context you used.

Context:
{context}

Question: {question}

Answer:"""
    
    # Generate response (using any LLM)
    # Here using OpenAI as example; replace with any LLM
    response = openai.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "You answer questions using only the provided context."},
            {"role": "user", "content": prompt}
        ],
        temperature=0.1  # low temperature for factual accuracy
    )
    
    return {
        "answer": response.choices[0].message.content,
        "sources": results['documents'][0],
        "distances": results['distances'][0]
    }

# Example usage
result = rag_query("How can I prevent my model from overfitting?")
print("Answer:", result['answer'])
print("\nSources used:")
for i, (source, dist) in enumerate(zip(result['sources'], result['distances'])):
    print(f"  [{i+1}] (similarity: {1-dist:.3f}) {source[:100]}...")
```

---

## Advanced RAG Techniques

### 1. Hybrid Search (Dense + Sparse)

Combining vector similarity with keyword matching (BM25) for better retrieval:

```
Query: "Adam optimizer learning rate"

Dense Search (Semantic):         Sparse Search (BM25/Keywords):
├─ "SGD variants and momentum"   ├─ "Adam optimizer uses adaptive
│   similarity: 0.82             │   learning rates" → BM25: 8.5
├─ "Optimization landscape"      ├─ "Learning rate scheduling"
│   similarity: 0.78             │   → BM25: 6.2
                                          
         ┌──────────────┐
         │  Reciprocal  │
         │ Rank Fusion  │  ← Combine both result lists
         │   (RRF)      │
         └──────┬───────┘
                │
    Final Ranked Results
    (Best of both worlds)
```

### 2. Query Transformation

```python
# Technique: HyDE (Hypothetical Document Embeddings)
# Generate a hypothetical answer, then use IT for retrieval

def hyde_query(question, llm):
    # Ask LLM to generate a hypothetical answer
    hypothetical = llm.generate(
        f"Write a short paragraph answering: {question}"
    )
    # Use the hypothetical answer as the search query
    # (often retrieves better documents than the question alone)
    return retriever.search(hypothetical)
```

### 3. Re-ranking

```
Initial Retrieval (fast, top-20)     Re-ranking (accurate, top-3)
┌────────────────────────────┐      ┌────────────────────────────┐
│ Vector search returns 20   │─────▶│ Cross-encoder scores each  │
│ approximate matches        │      │ (query, document) pair     │
│ using bi-encoder           │      │ More accurate but slower   │
└────────────────────────────┘      └────────────────────────────┘
```

```python
from sentence_transformers import CrossEncoder

# Re-ranker model
reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')

# Score each (query, document) pair
pairs = [(query, doc) for doc in retrieved_docs]
scores = reranker.predict(pairs)

# Re-rank by cross-encoder score
reranked = sorted(zip(retrieved_docs, scores), key=lambda x: x[1], reverse=True)
top_docs = [doc for doc, score in reranked[:3]]
```

### 4. Contextual Compression

Only pass the **relevant parts** of retrieved documents to the LLM:

```
Retrieved Document (500 tokens):
"Machine learning has many applications. [irrelevant content...]
The Adam optimizer combines momentum and RMSprop. It maintains 
per-parameter learning rates. [more irrelevant content...]"

     │ Contextual Compression
     ▼

Compressed (50 tokens):
"The Adam optimizer combines momentum and RMSprop. 
It maintains per-parameter learning rates."
```

---

## RAG Evaluation

### Key Metrics

```
┌─────────────────────────────────────────────────────┐
│              RAG EVALUATION FRAMEWORK               │
│                                                     │
│  Retrieval Quality:          Generation Quality:    │
│  ├─ Precision@K              ├─ Faithfulness        │
│  ├─ Recall@K                 ├─ Answer Relevance    │
│  ├─ MRR (Mean Reciprocal     ├─ Groundedness        │
│  │   Rank)                   ├─ Completeness        │
│  └─ nDCG                     └─ Citation Accuracy   │
│                                                     │
│  End-to-End:                                        │
│  ├─ Answer Correctness                              │
│  ├─ Hallucination Rate                              │
│  └─ User Satisfaction                               │
└─────────────────────────────────────────────────────┘
```

```python
# Using RAGAS framework for evaluation
# pip install ragas

from ragas.metrics import (
    faithfulness,       # Is answer supported by context?
    answer_relevancy,   # Is answer relevant to question?
    context_precision,  # Are retrieved docs relevant?
    context_recall      # Were all needed docs retrieved?
)

# Example evaluation
evaluation_data = {
    "question": "What is dropout?",
    "answer": "Dropout randomly deactivates neurons during training to prevent overfitting.",
    "contexts": ["Dropout randomly deactivates neurons during training..."],
    "ground_truth": "Dropout is a regularization technique that randomly sets neuron activations to zero."
}
```

---

## Common RAG Failure Modes & Solutions

| Failure Mode | Symptom | Solution |
|-------------|---------|----------|
| **Bad chunking** | Relevant info split across chunks | Increase overlap, semantic chunking |
| **Embedding mismatch** | Semantically similar docs not retrieved | Better embedding model, fine-tune |
| **Too few results** | Missing relevant context | Increase top-K, add hybrid search |
| **Too many results** | Context window overflow, noise | Re-ranking, contextual compression |
| **Wrong granularity** | Chunks too big or too small | Experiment with chunk sizes |
| **Outdated index** | Stale information retrieved | Incremental index updates |
| **Query ambiguity** | Retrieves wrong topic | Query expansion, HyDE |
| **Lost in the middle** | LLM ignores middle context | Reorder chunks by relevance |

---

## Production RAG Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  PRODUCTION RAG SYSTEM                       │
│                                                             │
│  ┌─────────┐   ┌──────────┐   ┌────────────┐              │
│  │ Document │──▶│ Chunking │──▶│  Embedding │──┐           │
│  │ Loader   │   │ + Clean  │   │   Model    │  │           │
│  └─────────┘   └──────────┘   └────────────┘  │           │
│                                                │           │
│                              ┌─────────────────▼────┐      │
│                              │   Vector Database    │      │
│                              │  (Pinecone/Qdrant/   │      │
│                              │   Weaviate)          │      │
│                              └─────────────────▲────┘      │
│                                                │           │
│  ┌─────────┐   ┌──────────┐   ┌────────────┐  │           │
│  │  User   │──▶│  Query   │──▶│  Retriever │──┘           │
│  │  Query  │   │ Transform│   │  (top-K)   │              │
│  └─────────┘   └──────────┘   └─────┬──────┘              │
│                                     │                      │
│                              ┌──────▼──────┐               │
│                              │  Re-ranker  │               │
│                              └──────┬──────┘               │
│                                     │                      │
│                              ┌──────▼──────┐               │
│                              │   Prompt    │               │
│                              │  Assembly   │               │
│                              └──────┬──────┘               │
│                                     │                      │
│                              ┌──────▼──────┐               │
│                              │     LLM     │               │
│                              │  Generation │               │
│                              └──────┬──────┘               │
│                                     │                      │
│                              ┌──────▼──────┐               │
│                              │   Output    │               │
│                              │  + Sources  │               │
│                              └─────────────┘               │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │ Monitoring: Latency, Relevance, User Feedback│          │
│  └─────────────────────────────────────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Description | Key Tool/Library |
|---------|-------------|-----------------|
| **RAG Pipeline** | Retrieve → Augment → Generate | LangChain, LlamaIndex |
| **Chunking** | Split docs into retrievable units | RecursiveCharacterTextSplitter |
| **Embedding** | Convert text to dense vectors | SentenceTransformers, OpenAI |
| **Vector Store** | Index and search embeddings | ChromaDB, FAISS, Pinecone |
| **Retrieval** | Find top-K relevant chunks | Cosine similarity, HNSW |
| **Re-ranking** | Improve retrieval precision | Cross-encoders |
| **Hybrid Search** | Dense + sparse retrieval | BM25 + vector search |
| **HyDE** | Hypothetical document for better retrieval | LLM-generated pseudo-docs |
| **Evaluation** | Measure RAG quality | RAGAS framework |
| **Production** | Scalable, monitored pipeline | Vector DB + LLM API + monitoring |

---

## Revision Questions

1. **What are the three stages of RAG, and what happens at each stage?**
2. **Why is chunking strategy important? What happens if chunks are too large or too small?**
3. **Explain the difference between bi-encoder (embedding) search and cross-encoder re-ranking. When would you use each?**
4. **What is HyDE and how does it improve retrieval quality?**
5. **Name three common RAG failure modes and their solutions.**
6. **How does hybrid search (dense + sparse) improve over pure vector search?**

---

[Previous: 05-prompt-engineering-advanced.md](05-prompt-engineering-advanced.md) | [Next: ← Back to README](../README.md)
