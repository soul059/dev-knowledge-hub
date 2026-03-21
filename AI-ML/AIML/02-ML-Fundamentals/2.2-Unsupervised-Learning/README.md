# 🔍 Unsupervised Learning

## Course Overview

This module covers unsupervised learning techniques — algorithms that discover hidden patterns in unlabeled data. From clustering algorithms (K-Means, DBSCAN, Hierarchical) to dimensionality reduction (PCA, t-SNE) and beyond (anomaly detection, association rules), these notes provide comprehensive coverage of the unsupervised learning landscape.

---

## 📋 Table of Contents

### Unit 1: Introduction to Unsupervised Learning
| # | Topic | File |
|---|-------|------|
| 1 | Supervised vs Unsupervised | [01-supervised-vs-unsupervised.md](01-Introduction-to-Unsupervised-Learning/01-supervised-vs-unsupervised.md) |
| 2 | Types of Unsupervised Learning | [02-types-of-unsupervised-learning.md](01-Introduction-to-Unsupervised-Learning/02-types-of-unsupervised-learning.md) |
| 3 | Applications and Use Cases | [03-applications-and-use-cases.md](01-Introduction-to-Unsupervised-Learning/03-applications-and-use-cases.md) |
| 4 | Challenges | [04-challenges.md](01-Introduction-to-Unsupervised-Learning/04-challenges.md) |

### Unit 2: K-Means Clustering
| # | Topic | File |
|---|-------|------|
| 1 | Algorithm Intuition | [01-algorithm-intuition.md](02-K-Means-Clustering/01-algorithm-intuition.md) |
| 2 | K-Means Algorithm Steps | [02-kmeans-algorithm-steps.md](02-K-Means-Clustering/02-kmeans-algorithm-steps.md) |
| 3 | Choosing K (Elbow, Silhouette) | [03-choosing-k.md](02-K-Means-Clustering/03-choosing-k.md) |
| 4 | Initialization (K-Means++) | [04-initialization-kmeans-plus.md](02-K-Means-Clustering/04-initialization-kmeans-plus.md) |
| 5 | Convergence and Limitations | [05-convergence-and-limitations.md](02-K-Means-Clustering/05-convergence-and-limitations.md) |
| 6 | Mini-batch K-Means | [06-mini-batch-kmeans.md](02-K-Means-Clustering/06-mini-batch-kmeans.md) |

### Unit 3: Hierarchical Clustering
| # | Topic | File |
|---|-------|------|
| 1 | Agglomerative vs Divisive | [01-agglomerative-vs-divisive.md](03-Hierarchical-Clustering/01-agglomerative-vs-divisive.md) |
| 2 | Linkage Methods | [02-linkage-methods.md](03-Hierarchical-Clustering/02-linkage-methods.md) |
| 3 | Dendrograms | [03-dendrograms.md](03-Hierarchical-Clustering/03-dendrograms.md) |
| 4 | Cutting the Dendrogram | [04-cutting-the-dendrogram.md](03-Hierarchical-Clustering/04-cutting-the-dendrogram.md) |
| 5 | Advantages and Limitations | [05-advantages-and-limitations.md](03-Hierarchical-Clustering/05-advantages-and-limitations.md) |

### Unit 4: DBSCAN
| # | Topic | File |
|---|-------|------|
| 1 | Density-based Clustering Concept | [01-density-based-concept.md](04-DBSCAN/01-density-based-concept.md) |
| 2 | Core, Border, and Noise Points | [02-core-border-noise-points.md](04-DBSCAN/02-core-border-noise-points.md) |
| 3 | Parameters (eps, min_samples) | [03-parameters-eps-minsamples.md](04-DBSCAN/03-parameters-eps-minsamples.md) |
| 4 | Algorithm Steps | [04-algorithm-steps.md](04-DBSCAN/04-algorithm-steps.md) |
| 5 | Advantages over K-Means | [05-advantages-over-kmeans.md](04-DBSCAN/05-advantages-over-kmeans.md) |
| 6 | HDBSCAN | [06-hdbscan.md](04-DBSCAN/06-hdbscan.md) |

### Unit 5: Gaussian Mixture Models (GMM)
| # | Topic | File |
|---|-------|------|
| 1 | Mixture Models Concept | [01-mixture-models-concept.md](05-Gaussian-Mixture-Models/01-mixture-models-concept.md) |
| 2 | GMM Intuition | [02-gmm-intuition.md](05-Gaussian-Mixture-Models/02-gmm-intuition.md) |
| 3 | Expectation-Maximization (EM) | [03-expectation-maximization.md](05-Gaussian-Mixture-Models/03-expectation-maximization.md) |
| 4 | Soft Clustering | [04-soft-clustering.md](05-Gaussian-Mixture-Models/04-soft-clustering.md) |
| 5 | Model Selection (BIC, AIC) | [05-model-selection-bic-aic.md](05-Gaussian-Mixture-Models/05-model-selection-bic-aic.md) |
| 6 | Comparison with K-Means | [06-comparison-with-kmeans.md](05-Gaussian-Mixture-Models/06-comparison-with-kmeans.md) |

### Unit 6: Principal Component Analysis (PCA)
| # | Topic | File |
|---|-------|------|
| 1 | Dimensionality Reduction Motivation | [01-dimensionality-reduction-motivation.md](06-Principal-Component-Analysis/01-dimensionality-reduction-motivation.md) |
| 2 | PCA Intuition (Maximize Variance) | [02-pca-intuition.md](06-Principal-Component-Analysis/02-pca-intuition.md) |
| 3 | Mathematical Formulation | [03-mathematical-formulation.md](06-Principal-Component-Analysis/03-mathematical-formulation.md) |
| 4 | Eigenvalue Decomposition Approach | [04-eigenvalue-decomposition.md](06-Principal-Component-Analysis/04-eigenvalue-decomposition.md) |
| 5 | SVD Approach | [05-svd-approach.md](06-Principal-Component-Analysis/05-svd-approach.md) |
| 6 | Choosing Number of Components | [06-choosing-components.md](06-Principal-Component-Analysis/06-choosing-components.md) |
| 7 | Explained Variance Ratio | [07-explained-variance-ratio.md](06-Principal-Component-Analysis/07-explained-variance-ratio.md) |
| 8 | PCA for Visualization | [08-pca-for-visualization.md](06-Principal-Component-Analysis/08-pca-for-visualization.md) |
| 9 | Limitations | [09-limitations.md](06-Principal-Component-Analysis/09-limitations.md) |

### Unit 7: Other Dimensionality Reduction
| # | Topic | File |
|---|-------|------|
| 1 | Linear Discriminant Analysis (LDA) | [01-lda.md](07-Other-Dimensionality-Reduction/01-lda.md) |
| 2 | t-SNE | [02-tsne.md](07-Other-Dimensionality-Reduction/02-tsne.md) |
| 3 | UMAP | [03-umap.md](07-Other-Dimensionality-Reduction/03-umap.md) |
| 4 | Autoencoders (Preview) | [04-autoencoders-preview.md](07-Other-Dimensionality-Reduction/04-autoencoders-preview.md) |
| 5 | Comparison of Methods | [05-comparison-of-methods.md](07-Other-Dimensionality-Reduction/05-comparison-of-methods.md) |

### Unit 8: Association Rules
| # | Topic | File |
|---|-------|------|
| 1 | Market Basket Analysis | [01-market-basket-analysis.md](08-Association-Rules/01-market-basket-analysis.md) |
| 2 | Support, Confidence, Lift | [02-support-confidence-lift.md](08-Association-Rules/02-support-confidence-lift.md) |
| 3 | Apriori Algorithm | [03-apriori-algorithm.md](08-Association-Rules/03-apriori-algorithm.md) |
| 4 | FP-Growth | [04-fp-growth.md](08-Association-Rules/04-fp-growth.md) |
| 5 | Applications | [05-applications.md](08-Association-Rules/05-applications.md) |

### Unit 9: Anomaly Detection
| # | Topic | File |
|---|-------|------|
| 1 | Types of Anomalies | [01-types-of-anomalies.md](09-Anomaly-Detection/01-types-of-anomalies.md) |
| 2 | Statistical Methods | [02-statistical-methods.md](09-Anomaly-Detection/02-statistical-methods.md) |
| 3 | Isolation Forest | [03-isolation-forest.md](09-Anomaly-Detection/03-isolation-forest.md) |
| 4 | One-Class SVM | [04-one-class-svm.md](09-Anomaly-Detection/04-one-class-svm.md) |
| 5 | Local Outlier Factor (LOF) | [05-local-outlier-factor.md](09-Anomaly-Detection/05-local-outlier-factor.md) |
| 6 | Autoencoders for Anomaly Detection | [06-autoencoders-anomaly-detection.md](09-Anomaly-Detection/06-autoencoders-anomaly-detection.md) |

### Unit 10: Clustering Evaluation
| # | Topic | File |
|---|-------|------|
| 1 | Internal Metrics | [01-internal-metrics.md](10-Clustering-Evaluation/01-internal-metrics.md) |
| 2 | External Metrics | [02-external-metrics.md](10-Clustering-Evaluation/02-external-metrics.md) |
| 3 | Visual Inspection | [03-visual-inspection.md](10-Clustering-Evaluation/03-visual-inspection.md) |
| 4 | Domain Knowledge | [04-domain-knowledge.md](10-Clustering-Evaluation/04-domain-knowledge.md) |

---

## 🎯 Learning Objectives

After completing this module, you will be able to:
- Apply clustering algorithms to group unlabeled data
- Reduce dimensionality for visualization and preprocessing
- Detect anomalies and outliers in datasets
- Mine association rules from transactional data
- Evaluate clustering quality with appropriate metrics
- Choose the right unsupervised algorithm for the task

---

## 📊 Algorithm Comparison Table

| Algorithm | Type | Shape | Scalability | Parameters | Noise Handling |
|-----------|------|-------|:-----------:|:----------:|:--------------:|
| K-Means | Clustering | Spherical | ✅ Fast | K | ❌ |
| Hierarchical | Clustering | Any | ⚠️ O(n³) | Linkage | ❌ |
| DBSCAN | Clustering | Any | ✅ Good | eps, min_samples | ✅ |
| GMM | Clustering | Ellipsoidal | ⚠️ Medium | K, covariance | ❌ |
| PCA | Dim. Reduction | Linear | ✅ Fast | n_components | ❌ |
| t-SNE | Dim. Reduction | Nonlinear | ⚠️ Slow | perplexity | ❌ |
| UMAP | Dim. Reduction | Nonlinear | ✅ Fast | n_neighbors | ❌ |
| Isolation Forest | Anomaly Det. | Any | ✅ Fast | contamination | ✅ |

---

[← Previous: Introduction to ML](../2.1-Introduction-to-ML/README.md) | [🏠 Back to Main Course](../../README.md) | [Next: Neural Networks →](../../03-Deep-Learning/3.1-Neural-Networks-Fundamentals/README.md)
