# 1.3 Applications and Use Cases of Unsupervised Learning

> **Chapter Overview:** Unsupervised learning powers some of the most impactful real-world applications across industries. This chapter provides a deep dive into seven major application domains — from customer segmentation to gene expression analysis — with concrete examples, techniques used, and business/scientific value delivered.

---

## Table of Contents

1. [Customer Segmentation](#1-customer-segmentation)
2. [Anomaly Detection in Fraud & Cybersecurity](#2-anomaly-detection-in-fraud--cybersecurity)
3. [Recommendation Systems](#3-recommendation-systems)
4. [Data Compression & Dimensionality Reduction](#4-data-compression--dimensionality-reduction)
5. [Topic Modeling in NLP](#5-topic-modeling-in-nlp)
6. [Image Segmentation](#6-image-segmentation)
7. [Gene Expression Analysis](#7-gene-expression-analysis)
8. [Applications Summary Table](#8-applications-summary-table)
9. [Python Worked Examples](#9-python-worked-examples)
10. [Key Takeaways](#10-key-takeaways)
11. [Revision Questions](#11-revision-questions)

---

## 1. Customer Segmentation

### The Problem

A business has thousands or millions of customers with varying behaviors. Rather than treating all customers the same, we want to **identify natural groups** and tailor strategies to each.

### How Unsupervised Learning Helps

```
    RAW CUSTOMER DATA                    SEGMENTED CUSTOMERS
    ┌──────────────────┐                 ┌──────────────────┐
    │ ○ ○ ○ ○ ○ ○ ○ ○  │                 │ ● ● ●   ▲ ▲ ▲   │
    │ ○ ○ ○ ○ ○ ○ ○ ○  │   K-Means /    │ ● ● ●   ▲ ▲ ▲   │
    │ ○ ○ ○ ○ ○ ○ ○ ○  │ ─────────── →  │                   │
    │ ○ ○ ○ ○ ○ ○ ○ ○  │   GMM / etc.   │ ■ ■ ■ ■   ◆ ◆   │
    │ ○ ○ ○ ○ ○ ○ ○ ○  │                 │ ■ ■ ■ ■   ◆ ◆   │
    └──────────────────┘                 └──────────────────┘
    All look the same!                   4 distinct segments!
```

### Features Used

| Feature Category    | Examples                                            |
|--------------------|-----------------------------------------------------|
| **Demographic**    | Age, gender, income, location                       |
| **Behavioral**     | Purchase frequency, avg. order value, last purchase |
| **RFM Analysis**   | Recency, Frequency, Monetary value                  |
| **Engagement**     | Website visits, email opens, app usage              |

### RFM Analysis — A Classic Segmentation Framework

```
    RFM Scoring:
    ┌──────────┬───────────────────────────────────┬──────────┐
    │ Metric   │ Definition                        │ Score    │
    ├──────────┼───────────────────────────────────┼──────────┤
    │ Recency  │ Days since last purchase           │ 1-5      │
    │ Frequency│ Number of purchases in period      │ 1-5      │
    │ Monetary │ Total spend in period              │ 1-5      │
    └──────────┴───────────────────────────────────┴──────────┘

    Customer A: R=5, F=5, M=5 → "Champions" (best customers)
    Customer B: R=1, F=1, M=1 → "Lost" (need re-engagement)
    Customer C: R=5, F=1, M=3 → "Promising" (new, moderate spend)
```

### Common Algorithms

- **K-Means:** Fast, works well for spherical clusters in RFM space
- **GMM:** Soft assignments — customer can partially belong to multiple segments
- **DBSCAN:** Finds oddly-shaped segments and identifies outlier customers
- **Hierarchical:** Provides dendrogram showing how segments relate

### Business Value

```
    Segment        │ Strategy                    │ Expected ROI Lift
    ───────────────┼─────────────────────────────┼──────────────────
    Champions      │ Loyalty rewards, early access│ 15-25%
    At-Risk        │ Win-back campaigns           │ 10-20%
    New Customers  │ Onboarding sequences         │ 20-30%
    Price-Sensitive│ Discount offers              │ 5-15%
    High-Value     │ Premium support, upselling   │ 25-40%
```

---

## 2. Anomaly Detection in Fraud & Cybersecurity

### The Problem

Fraudulent transactions and cyber attacks are **rare events** (often < 0.1% of data) hidden among millions of legitimate activities. Supervised approaches struggle because:
- Very few labeled fraud cases
- Fraudsters constantly change tactics (concept drift)
- False negatives are extremely costly

### Unsupervised Anomaly Detection Pipeline

```
    ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ Raw     │    │ Feature  │    │ Anomaly  │    │ Scoring  │    │ Alert    │
    │ Trans-  │ →  │ Engineer-│ →  │ Detection│ →  │ & Rank-  │ →  │ & Human  │
    │ actions │    │ ing      │    │ Model    │    │ ing      │    │ Review   │
    └─────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
```

### Techniques by Domain

#### Financial Fraud

```
    Features:
    • Transaction amount (deviation from user's average)
    • Time of day / day of week
    • Geolocation (impossible travel)
    • Merchant category change
    • Velocity (transactions per hour)

    Models:
    • Isolation Forest → anomaly score per transaction
    • Autoencoder → high reconstruction error = suspicious
    • One-Class SVM → learns boundary of "normal" behavior
```

#### Cybersecurity (Network Intrusion Detection)

```
    Network Traffic Flow:
    ┌──────────────────────────────────────────────┐
    │  Normal traffic patterns:                     │
    │  ═══════════════════════════════              │
    │  ═══════════════════════════════              │
    │  ═══════════════════════════════              │
    │                                               │
    │  Anomalous patterns:                          │
    │  ═══════════════════════▓▓▓▓▓▓▓▓▓▓ ← DDoS   │
    │  ═══════╳═══════════════════════ ← port scan │
    │  ═══╳╳╳╳╳╳╳═════════════════ ← data exfil.  │
    └──────────────────────────────────────────────┘

    Algorithms: LOF, DBSCAN, Clustering-based, Deep autoencoders
```

### Mathematical Foundation — Local Outlier Factor (LOF)

```
    LOF(x) measures how much denser x's neighbors are compared to x:

    1. k-distance(x) = distance to x's k-th nearest neighbor

    2. Reachability distance:
       reach_distₖ(x, o) = max{k-distance(o), d(x, o)}

    3. Local reachability density:
       lrd(x) = 1 / (avg reachability dist to k-NNs)

    4. LOF(x) = avg[ lrd(neighbor) / lrd(x) ] for all k-NNs

    LOF ≈ 1  → similar density to neighbors (normal)
    LOF >> 1 → much less dense than neighbors (outlier!)
```

---

## 3. Recommendation Systems

### The Problem

Given a user-item interaction matrix (ratings, clicks, purchases), **predict which items a user will like** — often framed as a matrix completion problem.

### Unsupervised Approach: Collaborative Filtering via Matrix Factorization

```
    User-Item Rating Matrix R (sparse):
    
              Movie1  Movie2  Movie3  Movie4  Movie5
    User1  [  5       3       ?       1       ?    ]
    User2  [  4       ?       ?       1       ?    ]
    User3  [  1       1       ?       5       4    ]
    User4  [  ?       1       5       4       ?    ]

    Factorize:  R ≈ U × Vᵀ

    U (users × k latent factors)    V (items × k latent factors)

    ┌───────┐       ┌───────────┐
    │ Users │       │   Items   │
    │ n × k │   ×   │   k × m   │   =   R̂ (predicted ratings)
    └───────┘       └───────────┘
```

### Mathematics

```
    Minimize:  Σ      (rᵢⱼ - uᵢᵀvⱼ)² + λ(‖U‖² + ‖V‖²)
             (i,j)∈Ω

    Where:
    Ω     = set of observed (non-missing) entries
    uᵢ    = latent factor vector for user i (k dimensions)
    vⱼ    = latent factor vector for item j (k dimensions)
    λ     = regularization strength
    k     = number of latent factors (typically 20-200)
```

### Other Unsupervised Recommendation Techniques

- **Clustering users:** Group similar users → recommend what the group likes
- **Item-item similarity:** Find items similar to what user already liked (cosine similarity)
- **Autoencoders:** Learn compressed user preference representations

---

## 4. Data Compression & Dimensionality Reduction

### The Problem

High-dimensional data is expensive to store, slow to process, and hard to visualize. We need to **compress** it while preserving essential information.

### Applications Across Domains

```
    ┌─────────────────────────────────────────────────────────┐
    │  Domain         │ Original Dim │ Reduced Dim │ Method   │
    ├─────────────────┼──────────────┼─────────────┼──────────┤
    │ Face images     │ 10,000 px    │ 100-200     │ PCA      │
    │ Text documents  │ 50,000 words │ 100-300     │ LSA/SVD  │
    │ Gene expression │ 20,000 genes │ 10-50       │ PCA/UMAP │
    │ Sensor data     │ 1,000 ch.    │ 10-20       │ PCA/AE   │
    │ Word embeddings │ Vocab size   │ 100-300     │ Word2Vec │
    └─────────────────────────────────────────────────────────┘
```

### Compression Ratio

```
    Compression Ratio = Original Size / Compressed Size

    Example: 1000-dim data → PCA with 50 components
    Ratio = 1000 / 50 = 20:1 compression

    But we keep 95%+ of the variance!

    Variance explained vs. components:
    
    Var%
    100│ ████████████████████████████████████
     95│ ████████████████████████████
     90│ ████████████████████████
     80│ ████████████████████
     60│ ████████████████
     40│ ████████████
     20│ ████████
      0│
       └──────────────────────────────────── k
        5  10  15  20  25  30  35  40  45  50

    Often 95% variance captured with 5-10% of original dims!
```

### Image Compression Example (PCA)

```
    Original Image (64×64 = 4096 pixels per image)
    ┌────────────────┐
    │                │       PCA with k components:
    │   ☺            │ →     k=10:  Blurry but recognizable
    │                │       k=50:  Good quality
    │                │       k=100: Near-original
    └────────────────┘       k=200: Indistinguishable

    Storage: 4096 values → k values + k×4096 shared basis vectors
    For 10,000 images with k=50: 10000×4096 → 10000×50 + 50×4096
                                 = 40.96M   → 0.70M  (58× compression!)
```

---

## 5. Topic Modeling in NLP

### The Problem

Given a collection of documents, automatically discover the **latent topics** being discussed without any topic labels.

### Latent Dirichlet Allocation (LDA)

```
    Documents are mixtures of topics; topics are distributions over words.

    ┌──────────────────────────────────────────────────────────┐
    │                                                          │
    │  Document 1: "The player scored a goal in the match"     │
    │                                                          │
    │    Topic Distribution:  Sports: 80%  |  Politics: 5%     │
    │                         Health: 10%  |  Tech: 5%         │
    │                                                          │
    │  Topic "Sports" word distribution:                       │
    │    player: 0.08, goal: 0.07, match: 0.06, team: 0.05,   │
    │    score: 0.05, game: 0.04, ...                          │
    │                                                          │
    │  Topic "Health" word distribution:                       │
    │    patient: 0.07, treatment: 0.06, disease: 0.05,        │
    │    doctor: 0.04, health: 0.04, ...                       │
    │                                                          │
    └──────────────────────────────────────────────────────────┘
```

### Generative Process

```
    For each document d:
        1. Draw topic proportions θ_d ~ Dirichlet(α)
        2. For each word position n in document d:
            a. Draw topic assignment z_dn ~ Multinomial(θ_d)
            b. Draw word w_dn ~ Multinomial(φ_{z_dn})

    Where:
        α   = Dirichlet prior on topic proportions (hyperparameter)
        θ_d = topic mixture for document d
        φ_k = word distribution for topic k
        K   = number of topics (set by user)
```

### Applications of Topic Modeling

| Application                  | Description                                     |
|-----------------------------|-------------------------------------------------|
| **News categorization**     | Discover topics across thousands of articles    |
| **Scientific literature**   | Find research themes in paper abstracts         |
| **Social media analysis**   | Track trending topics on Twitter/Reddit         |
| **Customer feedback**       | Identify common complaint themes                |
| **Legal document analysis** | Categorize case files by legal topics           |

---

## 6. Image Segmentation

### The Problem

Partition an image into **meaningful regions** (segments) based on pixel similarity — without labeled training masks.

### Unsupervised Image Segmentation Approaches

```
    Original Image          Segmented Image
    ┌──────────────┐        ┌──────────────┐
    │ ░░░░████████ │        │ ▒▒▒▒████████ │  ← Sky (segment 1)
    │ ░░░░████████ │        │ ▒▒▒▒████████ │
    │ ░░████░░░░░░ │   →    │ ▓▓████░░░░░░ │  ← Tree (segment 2)
    │ ████████████ │        │ ████████████ │  ← Ground (segment 3)
    │ ████████████ │        │ ████████████ │
    └──────────────┘        └──────────────┘
```

### Feature Representation for Pixels

```
    Each pixel → feature vector:

    [R, G, B, x_position, y_position]    ← color + spatial

    Or more advanced:
    [L*, a*, b*, x, y, texture_features, edge_features]

    Then apply clustering (K-Means, GMM, Mean Shift) to group pixels.
```

### Methods

| Method              | Approach                               | Best For                    |
|--------------------|----------------------------------------|-----------------------------|
| **K-Means on pixels** | Cluster by color/position            | Simple color-based segments |
| **Mean Shift**     | Mode-finding in color-spatial space    | Adaptive # segments         |
| **Normalized Cuts**| Graph-based spectral clustering        | Complex boundaries          |
| **SLIC Superpixels** | K-Means in CIELAB + xy space        | Over-segmentation (preprocessing)|
| **Felzenszwalb**   | Graph-based, efficient                 | Fast initial segmentation    |

### Mathematical Basis — K-Means for Image Segmentation

```
    Given image I with N pixels, each pixel pᵢ = [rᵢ, gᵢ, bᵢ, xᵢ, yᵢ]

    Minimize:  Σ    Σ     w_spatial · ‖pos(pᵢ) - pos(μₖ)‖²
              k=1  pᵢ∈Cₖ
                        + w_color · ‖color(pᵢ) - color(μₖ)‖²

    The weights w_spatial and w_color control compactness vs. color purity.
```

---

## 7. Gene Expression Analysis

### The Problem

In genomics, **gene expression data** measures the activity level of thousands of genes across multiple samples (patients, cell types, time points). The goal: discover which genes behave similarly and which samples are related.

### Data Structure

```
    Gene Expression Matrix:
                    Sample1  Sample2  Sample3  ...  Sample_m
    Gene_1    [     2.5      3.1      0.2     ...    1.8    ]
    Gene_2    [     0.1      0.3      4.5     ...    3.2    ]
    Gene_3    [     5.2      4.8      5.1     ...    4.9    ]
    ...
    Gene_n    [     1.1      2.3      1.0     ...    0.8    ]

    Typically: n = 20,000+ genes, m = 50-500 samples
```

### Bi-Clustering: Clustering Genes AND Samples

```
    ┌──────────────────────────────────────┐
    │     Original Expression Matrix       │
    │ ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐     │
    │ │░░│▓▓│░░│██│░░│▓▓│░░│██│░░│▓▓│     │
    │ │▓▓│░░│▓▓│░░│▓▓│░░│▓▓│░░│▓▓│░░│     │
    │ │██│██│██│░░│██│██│██│░░│██│██│     │
    │ │░░│▓▓│░░│██│░░│▓▓│░░│██│░░│▓▓│     │
    │ └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘     │
    │              ↓ Cluster rows & cols    │
    │     Reordered Expression Matrix      │
    │ ┌──┬──┬──┬──┬──║──┬──┬──┬──┬──┐     │
    │ │██│██│██│██│██║░░│░░│░░│░░│░░│     │
    │ │██│██│██│██│██║░░│░░│░░│░░│░░│     │
    │ ╠══╬══╬══╬══╬══╬══╬══╬══╬══╬══╣     │
    │ │░░│░░│░░│░░│░░║▓▓│▓▓│▓▓│▓▓│▓▓│     │
    │ │░░│░░│░░│░░│░░║▓▓│▓▓│▓▓│▓▓│▓▓│     │
    │ └──┴──┴──┴──┴──╨──┴──┴──┴──┴──┘     │
    │     Clear co-expression blocks!      │
    └──────────────────────────────────────┘
```

### Key Applications in Genomics

| Application                    | Unsupervised Method           | Impact                          |
|-------------------------------|-------------------------------|---------------------------------|
| Cancer subtype discovery       | Hierarchical clustering       | Identified breast cancer subtypes|
| Gene co-expression networks   | WGCNA (correlation + clustering)| Pathway discovery              |
| Cell type identification       | t-SNE/UMAP + clustering      | Single-cell RNA-seq revolution  |
| Drug response profiling       | PCA + clustering              | Pharmacogenomics                |
| Evolutionary relationships    | Hierarchical clustering       | Phylogenetic analysis           |

### Single-Cell RNA-Seq Pipeline

```
    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ Raw      │    │ Quality  │    │ Dim.     │    │ Cluster  │
    │ Counts   │ →  │ Control  │ →  │ Reduction│ →  │ Analysis │
    │ (20K     │    │ & Norm.  │    │ PCA →    │    │ Leiden / │
    │  genes × │    │          │    │ UMAP     │    │ Louvain  │
    │  10K     │    │          │    │          │    │          │
    │  cells)  │    │          │    │          │    │          │
    └──────────┘    └──────────┘    └──────────┘    └──────────┘
                                                          │
                                                    ┌─────┴─────┐
                                                    │ Cell Type  │
                                                    │ Annotation │
                                                    └───────────┘
```

---

## 8. Applications Summary Table

```
┌─────────────────────┬───────────────────┬──────────────────┬────────────────────────┐
│ Application         │ UL Method(s)      │ Input Data       │ Business/Science Value │
├─────────────────────┼───────────────────┼──────────────────┼────────────────────────┤
│ Customer Segment.   │ K-Means, GMM      │ Purchase history │ Targeted marketing     │
│ Fraud Detection     │ Isolation Forest, │ Transaction logs │ Financial loss prevent.│
│                     │ LOF, Autoencoder  │                  │                        │
│ Recommendations     │ Matrix Factor.,   │ User-item ratings│ Revenue increase       │
│                     │ Clustering        │                  │                        │
│ Data Compression    │ PCA, Autoencoders │ Images, text     │ Storage savings        │
│ Topic Modeling      │ LDA, NMF          │ Text documents   │ Content organization   │
│ Image Segmentation  │ K-Means, Mean     │ Pixel features   │ Object recognition     │
│                     │ Shift, Spectral   │                  │ preprocessing          │
│ Gene Expression     │ Hierarchical,     │ Expression matrix│ Disease understanding  │
│                     │ PCA, UMAP         │                  │ Drug discovery         │
└─────────────────────┴───────────────────┴──────────────────┴────────────────────────┘
```

---

## 9. Python Worked Examples

### Customer Segmentation with K-Means

```python
import numpy as np
import pandas as pd
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

# Simulate RFM data
np.random.seed(42)
n_customers = 500
data = pd.DataFrame({
    'Recency': np.random.exponential(30, n_customers),
    'Frequency': np.random.poisson(5, n_customers),
    'Monetary': np.random.lognormal(4, 1, n_customers)
})

# Standardize features
scaler = StandardScaler()
X_scaled = scaler.fit_transform(data)

# Segment into 4 groups
kmeans = KMeans(n_clusters=4, random_state=42, n_init=10)
data['Segment'] = kmeans.fit_predict(X_scaled)

# Analyze segments
segment_summary = data.groupby('Segment').agg({
    'Recency': 'mean',
    'Frequency': 'mean',
    'Monetary': 'mean'
}).round(2)
print(segment_summary)
```

### Anomaly Detection with Isolation Forest

```python
from sklearn.ensemble import IsolationForest
import numpy as np

# Simulate transaction data
np.random.seed(42)
normal = np.random.randn(1000, 2) * [100, 50] + [500, 200]  # amount, time
fraud = np.array([[2500, 3], [3000, 2], [50, 450], [2800, 4], [100, 480]])
X = np.vstack([normal, fraud])

# Detect anomalies
iso_forest = IsolationForest(contamination=0.01, random_state=42)
predictions = iso_forest.fit_predict(X)

n_anomalies = (predictions == -1).sum()
print(f"Detected {n_anomalies} anomalies out of {len(X)} transactions")
print(f"Anomaly indices: {np.where(predictions == -1)[0]}")
```

### Topic Modeling with LDA

```python
from sklearn.decomposition import LatentDirichletAllocation
from sklearn.feature_extraction.text import CountVectorizer

documents = [
    "machine learning algorithms data science artificial intelligence",
    "deep neural networks training backpropagation gradient descent",
    "patient treatment hospital medical diagnosis healthcare",
    "drug therapy clinical trial pharmaceutical medication",
    "stock market trading financial investment portfolio returns",
    "economic growth inflation monetary policy interest rates",
    "neural network deep learning model training optimization",
    "medical imaging diagnosis radiology patient scan MRI",
]

vectorizer = CountVectorizer(stop_words='english')
X_counts = vectorizer.fit_transform(documents)

lda = LatentDirichletAllocation(n_components=3, random_state=42)
lda.fit(X_counts)

feature_names = vectorizer.get_feature_names_out()
for topic_idx, topic in enumerate(lda.components_):
    top_words = [feature_names[i] for i in topic.argsort()[-5:]]
    print(f"Topic {topic_idx}: {', '.join(top_words)}")
```

---

## 10. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **Customer segmentation** with clustering drives personalized marketing and can increase ROI by 15-40% |
| 2 | **Anomaly detection** is critical in fraud and cybersecurity where labeled fraud data is scarce |
| 3 | **Recommendation systems** use matrix factorization and clustering to predict user preferences |
| 4 | **PCA and autoencoders** can compress data 10-100× while preserving >95% of information |
| 5 | **Topic modeling** (LDA) discovers themes in text without requiring category labels |
| 6 | **Image segmentation** clusters pixels by color/texture to identify regions |
| 7 | **Gene expression analysis** uses hierarchical clustering and UMAP to discover cell types and disease subtypes |

---

## 11. Revision Questions

1. **Customer Segmentation:** Explain the RFM framework. How would you use K-Means on RFM features to segment customers, and what preprocessing step is crucial?

2. **Fraud Detection:** Why is unsupervised learning preferred over supervised learning for fraud detection in many cases? What are the advantages of Isolation Forest over LOF for this task?

3. **Recommendations:** Describe matrix factorization for collaborative filtering. What do the latent factors represent, and how do we handle the sparsity of the rating matrix?

4. **Topic Modeling:** In LDA, each document is modeled as a mixture of topics. Explain the generative process. How do you choose the number of topics K?

5. **Genomics:** Describe the single-cell RNA-seq analysis pipeline using unsupervised learning. Why is dimensionality reduction a critical step before clustering?

6. **Cross-Domain:** Choose any two applications from this chapter and design a combined pipeline. For example, how might customer segmentation feed into a recommendation system?

---

<div align="center">

| [← Types of Unsupervised Learning](./02-types-of-unsupervised-learning.md) | [Up: Introduction](./README.md) | [Next: Challenges →](./04-challenges.md) |
|:--------------------------------------------------------------------------:|:-------------------------------:|:----------------------------------------:|

</div>
