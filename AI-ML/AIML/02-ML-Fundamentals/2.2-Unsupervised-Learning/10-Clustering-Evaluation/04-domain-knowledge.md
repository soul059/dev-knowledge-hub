# 🧠 Domain Knowledge in Clustering Evaluation

> **Chapter 10.4 — Beyond Metrics: Practical Evaluation with Business Context**

---

## 📌 Overview

Quantitative metrics and visualizations are essential, but the ultimate test of clustering quality is whether the clusters are **meaningful, actionable, and useful** in the real-world context. **Domain knowledge** bridges the gap between statistical measures and practical value. A clustering with slightly lower Silhouette Score may be far more useful if the clusters align with business needs.

### Why Domain Knowledge Matters

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  SCENARIO: Customer Segmentation                                     │
│                                                                      │
│  Algorithm A: Silhouette = 0.72                                      │
│    Clusters: "High income", "Low income", "Middle income"            │
│    → Not actionable (just re-discovers income brackets)              │
│                                                                      │
│  Algorithm B: Silhouette = 0.58                                      │
│    Clusters: "Price-sensitive loyals", "Brand explorers",            │
│              "Seasonal bulk buyers", "Premium seekers"               │
│    → Highly actionable (each segment gets different strategy!)       │
│                                                                      │
│  CONCLUSION: Algorithm B is BETTER despite lower metric!             │
│  Domain knowledge tells us Algorithm B produces                      │
│  INTERPRETABLE and ACTIONABLE clusters.                              │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🔍 1. Cluster Interpretability

### Cluster Profiling

After clustering, **profile each cluster** by examining its key characteristics:

```python
import pandas as pd
import numpy as np
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

# Example: Customer data
np.random.seed(42)
n = 500
data = pd.DataFrame({
    'age': np.concatenate([np.random.normal(25, 5, 150),
                          np.random.normal(45, 8, 200),
                          np.random.normal(65, 7, 150)]),
    'annual_spend': np.concatenate([np.random.normal(500, 200, 150),
                                    np.random.normal(2000, 500, 200),
                                    np.random.normal(800, 300, 150)]),
    'visits_per_month': np.concatenate([np.random.normal(8, 3, 150),
                                        np.random.normal(4, 2, 200),
                                        np.random.normal(2, 1, 150)]),
    'online_ratio': np.concatenate([np.random.normal(0.8, 0.1, 150),
                                    np.random.normal(0.5, 0.2, 200),
                                    np.random.normal(0.2, 0.1, 150)]),
})

# Cluster
scaler = StandardScaler()
X_scaled = scaler.fit_transform(data)

kmeans = KMeans(n_clusters=3, random_state=42, n_init=10)
data['cluster'] = kmeans.fit_predict(X_scaled)

# ═══════════════════════════════════════
# CLUSTER PROFILING
# ═══════════════════════════════════════

# Summary statistics per cluster
profile = data.groupby('cluster').agg(['mean', 'std', 'count'])
print("Cluster Profiles:")
print(profile.round(2).to_string())

# Normalized profile (compare to overall mean)
overall_mean = data.drop('cluster', axis=1).mean()
for cluster_id in range(3):
    cluster_data = data[data['cluster'] == cluster_id]
    cluster_mean = cluster_data.drop('cluster', axis=1).mean()
    deviation = (cluster_mean - overall_mean) / overall_mean * 100
    
    print(f"\nCluster {cluster_id} (n={len(cluster_data)}):")
    for col, dev in deviation.items():
        direction = "↑" if dev > 10 else "↓" if dev < -10 else "≈"
        print(f"  {col:25s}: {cluster_mean[col]:8.2f}  ({direction} {dev:+.1f}%)")
```

### Naming Clusters

```
┌──────────────────────────────────────────────────────────────┐
│              CLUSTER NAMING FRAMEWORK                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: Examine cluster profiles (means, distributions)    │
│  Step 2: Identify distinguishing features per cluster       │
│  Step 3: Create descriptive names based on key attributes   │
│  Step 4: Validate names with domain experts                 │
│                                                              │
│  Example Results:                                            │
│                                                              │
│  Cluster 0 → "Digital Natives"                               │
│    Young (25), high visits (8/mo), high online ratio (80%)  │
│    Low spend ($500)                                          │
│                                                              │
│  Cluster 1 → "Premium Professionals"                         │
│    Mid-age (45), high spend ($2000), moderate visits (4/mo) │
│    Mixed channels (50% online)                               │
│                                                              │
│  Cluster 2 → "Traditional Seniors"                           │
│    Older (65), moderate spend ($800), low visits (2/mo)      │
│    Low online ratio (20%) → prefer in-store                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Visualization: Radar Chart

```python
import matplotlib.pyplot as plt
import numpy as np

def plot_cluster_radar(data, cluster_col='cluster', figsize=(10, 8)):
    """Radar chart comparing cluster profiles."""
    features = [c for c in data.columns if c != cluster_col]
    n_features = len(features)
    clusters = sorted(data[cluster_col].unique())
    
    # Normalize to [0, 1] for radar chart
    normalized = data.copy()
    for f in features:
        normalized[f] = (data[f] - data[f].min()) / (data[f].max() - data[f].min())
    
    # Create radar
    angles = np.linspace(0, 2 * np.pi, n_features, endpoint=False).tolist()
    angles += angles[:1]  # Close the circle
    
    fig, ax = plt.subplots(figsize=figsize, subplot_kw=dict(polar=True))
    
    colors = ['#e74c3c', '#2ecc71', '#3498db', '#f39c12']
    names = ['Digital Natives', 'Premium Professionals', 'Traditional Seniors']
    
    for i, cluster in enumerate(clusters):
        values = normalized[normalized[cluster_col] == cluster][features].mean().tolist()
        values += values[:1]
        
        ax.plot(angles, values, 'o-', linewidth=2, color=colors[i],
                label=f'C{cluster}: {names[i]}')
        ax.fill(angles, values, alpha=0.15, color=colors[i])
    
    ax.set_xticks(angles[:-1])
    ax.set_xticklabels(features, fontsize=10)
    ax.set_title('Cluster Profiles', fontsize=14)
    ax.legend(loc='upper right', bbox_to_anchor=(1.3, 1.1))
    
    plt.tight_layout()
    plt.show()

plot_cluster_radar(data)
```

---

## 📈 2. Business Metrics Evaluation

### Connecting Clusters to Business Value

```
┌──────────────────────────────────────────────────────────────┐
│           BUSINESS-ORIENTED EVALUATION                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  For each cluster, measure:                                  │
│                                                              │
│  Revenue Metrics:                                            │
│  • Average Revenue Per User (ARPU)                           │
│  • Customer Lifetime Value (CLV)                             │
│  • Conversion rate                                           │
│                                                              │
│  Engagement Metrics:                                         │
│  • Retention rate per cluster                                │
│  • Churn rate per cluster                                    │
│  • Engagement frequency                                      │
│                                                              │
│  Operational Metrics:                                        │
│  • Cost-to-serve per cluster                                 │
│  • Support ticket volume                                     │
│  • Net Promoter Score (NPS)                                  │
│                                                              │
│  KEY QUESTION: "Can we take DIFFERENT ACTIONS for            │
│  different clusters, and will that improve outcomes?"        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```python
# Example: Business metrics by cluster
business_metrics = data.copy()
business_metrics['ltv'] = data['annual_spend'] * (data['visits_per_month'] * 12 / 50)
business_metrics['churn_risk'] = 1 / (data['visits_per_month'] + 1)

print("Business Metrics by Cluster:")
print(business_metrics.groupby('cluster')[['annual_spend', 'ltv', 'churn_risk']].mean().round(2))
```

---

## 🔄 3. Stability Analysis

### How Stable Are the Clusters?

```
┌──────────────────────────────────────────────────────────────┐
│              STABILITY ANALYSIS                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Good clusters should be STABLE:                             │
│  • Same clusters emerge with different random seeds          │
│  • Clusters survive small perturbations to the data          │
│  • Subsampling produces consistent results                   │
│                                                              │
│  Methods:                                                    │
│  1. Bootstrap stability: Cluster random subsamples           │
│  2. Random seed variation: Run K-Means with different seeds  │
│  3. Feature perturbation: Add noise to features              │
│  4. Cross-validation: Cluster on train, assign test          │
│                                                              │
│  Metric: ARI between runs (should be > 0.8 for stable)      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```python
from sklearn.metrics import adjusted_rand_score

def cluster_stability(X, n_clusters=3, n_runs=20, subsample_ratio=0.8):
    """Assess cluster stability via bootstrap resampling."""
    n = X.shape[0]
    subsample_size = int(n * subsample_ratio)
    
    # Reference clustering (on full data)
    ref_labels = KMeans(n_clusters=n_clusters, random_state=42, 
                        n_init=10).fit_predict(X)
    
    ari_scores = []
    for i in range(n_runs):
        # Random subsample
        idx = np.random.choice(n, subsample_size, replace=False)
        X_sub = X[idx]
        
        # Cluster subsample
        sub_labels = KMeans(n_clusters=n_clusters, random_state=i, 
                           n_init=10).fit_predict(X_sub)
        
        # Compare with reference (on shared points)
        ari = adjusted_rand_score(ref_labels[idx], sub_labels)
        ari_scores.append(ari)
    
    print(f"Stability Analysis (K={n_clusters}):")
    print(f"  Mean ARI:  {np.mean(ari_scores):.4f}")
    print(f"  Std ARI:   {np.std(ari_scores):.4f}")
    print(f"  Min ARI:   {np.min(ari_scores):.4f}")
    
    return ari_scores

stability = cluster_stability(X_scaled, n_clusters=3, n_runs=50)
```

---

## 🤝 4. Expert Validation

### The Validation Process

```
╔══════════════════════════════════════════════════════════════╗
║              EXPERT VALIDATION WORKFLOW                     ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  Step 1: PRESENT cluster profiles to domain experts          ║
║          (marketing team, doctors, engineers, etc.)           ║
║                                                              ║
║  Step 2: ASK key questions:                                  ║
║          • "Do these groups make sense?"                      ║
║          • "Do the distinguishing features align with        ║
║            your domain knowledge?"                           ║
║          • "Are there known groups missing?"                 ║
║          • "Would you take different actions for             ║
║            each cluster?"                                    ║
║                                                              ║
║  Step 3: ITERATE based on feedback                           ║
║          • Adjust K                                          ║
║          • Add/remove features                               ║
║          • Try different algorithms                          ║
║          • Refine preprocessing                              ║
║                                                              ║
║  Step 4: VALIDATE with real-world outcomes                   ║
║          • A/B test cluster-specific strategies               ║
║          • Monitor business metrics per cluster               ║
║          • Track cluster membership over time                ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 📊 5. Complete Evaluation Framework

```
┌──────────────────────────────────────────────────────────────┐
│     COMPREHENSIVE CLUSTERING EVALUATION FRAMEWORK           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  LEVEL 1: QUANTITATIVE (Automatic)                           │
│  ┌────────────────────────────────────────────────┐          │
│  │ Internal: Silhouette, CH, DB, WCSS             │          │
│  │ External: ARI, NMI (if labels available)       │          │
│  │ → Provides numerical quality scores            │          │
│  └────────────────────────────────────────────────┘          │
│                     │                                        │
│                     ▼                                        │
│  LEVEL 2: VISUAL (Semi-automatic)                            │
│  ┌────────────────────────────────────────────────┐          │
│  │ Scatter plots, silhouette plots, dendrograms   │          │
│  │ PCA/t-SNE/UMAP projections                     │          │
│  │ → Reveals patterns metrics miss                │          │
│  └────────────────────────────────────────────────┘          │
│                     │                                        │
│                     ▼                                        │
│  LEVEL 3: DOMAIN (Manual)                                    │
│  ┌────────────────────────────────────────────────┐          │
│  │ Cluster profiling and naming                   │          │
│  │ Expert validation                              │          │
│  │ Business metric alignment                      │          │
│  │ Actionability assessment                       │          │
│  │ → Ensures real-world value                     │          │
│  └────────────────────────────────────────────────┘          │
│                     │                                        │
│                     ▼                                        │
│  LEVEL 4: OPERATIONAL (Ongoing)                              │
│  ┌────────────────────────────────────────────────┐          │
│  │ Stability analysis                             │          │
│  │ Temporal consistency (clusters over time)       │          │
│  │ A/B testing of cluster-specific strategies     │          │
│  │ → Validates in production                      │          │
│  └────────────────────────────────────────────────┘          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 📋 Summary Table

| Evaluation Aspect | Methods | Key Question |
|-------------------|---------|-------------|
| **Interpretability** | Cluster profiling, naming | "Do clusters make sense?" |
| **Business Value** | Business metrics per cluster | "Can we act differently?" |
| **Stability** | Bootstrap, seed variation | "Are clusters robust?" |
| **Expert Validation** | Domain expert review | "Do experts recognize these groups?" |
| **Actionability** | Strategy mapping | "Will cluster-specific actions improve outcomes?" |
| **Temporal** | Tracking clusters over time | "Are clusters consistent?" |

---

## ❓ Revision Questions

1. **Why might a clustering with a higher Silhouette Score be less useful than one with a lower score? Give a concrete business example.**
   Discuss the gap between statistical quality and practical utility.

2. **Describe the cluster profiling process. How would you present cluster profiles to a non-technical marketing team?**
   Consider visualization choices, naming conventions, and actionable insights.

3. **What is cluster stability analysis and why is it important? How would you implement it?**
   Discuss bootstrap methods, ARI between runs, and what instability implies.

4. **Design a complete evaluation framework for a customer segmentation project at an e-commerce company.**
   Include quantitative metrics, visualizations, domain validation, and operational testing.

5. **An expert says "I expected 5 customer segments based on our market research, but the algorithm found 3." How would you handle this disagreement?**
   Discuss when to trust the algorithm vs. domain expertise, and potential resolutions.

6. **How would you track the quality and relevance of clusters in production over time? What would trigger a re-clustering?**
   Discuss drift detection, metric monitoring, and re-training schedules.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Visual Inspection](./03-visual-inspection.md) | [📂 Unsupervised Learning](../README.md) | [📚 Back to ML Fundamentals](../../README.md) |

---

> **Key Takeaway:** The best clustering is one that is statistically sound AND practically useful. Domain knowledge, expert validation, and business metrics are just as important as Silhouette Scores and ARI values. Always evaluate clustering through a multi-level framework: quantitative metrics → visualization → domain validation → operational testing. The ultimate measure of clustering quality is whether it drives better decisions.
