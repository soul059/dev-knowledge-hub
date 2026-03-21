# Topic Modeling

## Overview

Topic modeling automatically discovers abstract "topics" in a collection of documents. Each topic is a distribution over words, and each document is a mixture of topics. This unsupervised technique is invaluable for exploring large corpora, organizing content, and understanding themes in text data.

---

## Intuition

```
Given 1000 news articles, topic modeling might discover:

  Topic 1 (Sports):    goal, player, team, match, score, league
  Topic 2 (Politics):  president, election, vote, congress, policy
  Topic 3 (Tech):      AI, data, algorithm, computer, software
  Topic 4 (Health):    patient, treatment, disease, hospital, vaccine

Each document is a mixture:
  Article about "AI in healthcare" → 40% Tech + 50% Health + 10% Other
  Article about "election results" → 90% Politics + 10% Other
```

---

## Latent Dirichlet Allocation (LDA)

```
The most popular topic model (Blei et al., 2003)

Generative process:
  For each document d:
    1. Draw topic proportions θ_d ~ Dirichlet(α)
       θ_d = [0.3, 0.5, 0.1, 0.1]  (how much of each topic)
    
    2. For each word w in document d:
       a. Choose a topic z ~ Multinomial(θ_d)
          z = "Health"
       b. Choose a word w ~ Multinomial(φ_z)
          w = "patient"
  
  φ_z = word distribution for topic z
  θ_d = topic distribution for document d

Inference:
  Given observed words, infer latent topics
  Methods: Variational inference, Gibbs sampling

Plate notation:
  ┌─────────────────────────┐
  │  α → θ_d → z_n → w_n   │  × N words
  │              ↑           │
  │  β → φ_k ───┘           │  × K topics
  └─────────────────────────┘
      × D documents
```

---

## LDA Implementation

```python
from sklearn.decomposition import LatentDirichletAllocation
from sklearn.feature_extraction.text import CountVectorizer

documents = [
    "Machine learning algorithms analyze data patterns",
    "Neural networks are used in deep learning models",
    "The election results surprised many political analysts",
    "Congress debated the new healthcare policy today",
    "AI systems can diagnose diseases from medical images",
    "The president signed a new technology innovation bill",
    "Patients received vaccine doses at hospitals nationwide",
    "Data scientists use Python for machine learning projects",
]

# Create document-term matrix
vectorizer = CountVectorizer(max_df=0.95, min_df=1, stop_words="english")
dtm = vectorizer.fit_transform(documents)

# Fit LDA
lda = LatentDirichletAllocation(n_components=3, random_state=42)
lda.fit(dtm)

# Display topics
feature_names = vectorizer.get_feature_names_out()
for idx, topic in enumerate(lda.components_):
    top_words = [feature_names[i] for i in topic.argsort()[-5:]]
    print(f"Topic {idx}: {', '.join(top_words)}")

# Topic 0: data, learning, machine, algorithms, patterns
# Topic 1: new, policy, president, election, congress
# Topic 2: patients, diseases, medical, vaccine, hospitals
```

---

## BERTopic (Neural Topic Modeling)

```python
from bertopic import BERTopic

# BERTopic uses:
# 1. Sentence embeddings (SBERT)
# 2. UMAP for dimensionality reduction
# 3. HDBSCAN for clustering
# 4. c-TF-IDF for topic representation

model = BERTopic(language="english", min_topic_size=5)

topics, probs = model.fit_transform(documents)

# View topics
print(model.get_topic_info())

# Visualize
fig = model.visualize_topics()
fig.show()

# Topic evolution over time
topics_over_time = model.topics_over_time(documents, timestamps)
model.visualize_topics_over_time(topics_over_time)

# BERTopic pipeline:
#   Documents → SBERT embeddings → UMAP (reduce dims)
#                                      ↓
#                                  HDBSCAN (cluster)
#                                      ↓
#                                  c-TF-IDF (name topics)
```

---

## Comparison of Methods

| Method | Type | Strengths | Weaknesses |
|--------|------|-----------|------------|
| LDA | Probabilistic | Interpretable, well-studied | Bag-of-words, slow |
| NMF | Matrix factorization | Faster, coherent topics | Less flexible |
| LSA/LSI | SVD-based | Fast, captures synonymy | No probabilistic model |
| Top2Vec | Neural | Automatic topic count | Less interpretable |
| BERTopic | Neural + clustering | Best quality, flexible | Requires GPU |
| CTM | Neural (VAE) | Handles short texts | Complex training |

---

## Evaluation

```
Topic Coherence: How interpretable are the topics?

  C_v coherence (most popular):
    For each topic, measure how often top words co-occur
    Higher = more coherent topics

  Example:
    Topic A: ["neural", "network", "deep", "learning", "layer"]
    → High coherence (related words)
    
    Topic B: ["apple", "congress", "neuron", "pizza", "river"]
    → Low coherence (random words)

  Perplexity: How well does the model predict held-out documents?
    Lower = better fit (but not always better interpretability!)
```

---

## Applications

| Application | Description |
|-------------|-------------|
| Content organization | Auto-categorize articles, papers |
| Trend analysis | Track topic evolution over time |
| Customer feedback | Discover complaint themes |
| Research surveys | Map research landscape |
| Recommendation | "Similar topics" suggestions |
| Content moderation | Flag emerging harmful topics |

---

## Revision Questions

1. **What is topic modeling and how does LDA work?**
2. **What are the hyperparameters α and β in LDA?**
3. **How does BERTopic differ from traditional LDA?**
4. **What is topic coherence and why is it used over perplexity?**
5. **When would you choose BERTopic over LDA?**

---

[Previous: 03-sentiment-analysis.md](03-sentiment-analysis.md) | [Next: 05-information-extraction.md](05-information-extraction.md)
