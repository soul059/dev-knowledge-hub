# Unit 3: Threat Detection — Topic 4: Anomaly Detection

## Overview

**Anomaly detection** identifies threats by recognizing deviations from established normal patterns without prior knowledge of specific attack signatures. Using statistical methods and machine learning, anomaly detection can uncover previously unknown threats, insider threats, and novel attack techniques that signature-based methods miss.

---

## 1. Anomaly Detection Concepts

```
TYPES OF ANOMALY DETECTION:

  STATISTICAL ANOMALY:
  → Baseline of normal values
  → Alert when value exceeds N standard deviations
  → Example: User normally downloads 50MB/day
    → Alert when download > 500MB

  BEHAVIORAL ANOMALY:
  → Model of normal behavior patterns
  → Alert when behavior deviates
  → Example: User normally logs in 9-5
    → Alert on 3 AM login

  STRUCTURAL ANOMALY:
  → Normal network/data structure
  → Alert on unexpected patterns
  → Example: New internal communication path
    → Two servers that never talked before

  TEMPORAL ANOMALY:
  → Normal time-based patterns
  → Alert on unusual timing
  → Example: Backup job runs at 2 AM daily
    → Alert when it runs at noon

SUPERVISED vs UNSUPERVISED:

  SUPERVISED ML:
  → Trained on labeled data (known good/bad)
  → Classification models
  → Requires labeled training data
  → Good for known attack types
  
  UNSUPERVISED ML:
  → No labeled data required
  → Finds patterns autonomously
  → Clustering, density estimation
  → Better for unknown threats
  → More false positives
  
  SEMI-SUPERVISED:
  → Trained on "normal" data only
  → Anything different = anomaly
  → Most common in security
  → Requires good baseline period
```

---

## 2. Statistical Methods

```
STATISTICAL APPROACHES:

THRESHOLD-BASED:
  → Simple upper/lower bounds
  → Static or dynamic thresholds
  → Example: Alert if CPU > 95%
  → Easy but many false positives

Z-SCORE:
  → Measures standard deviations from mean
  → Z = (x - μ) / σ
  → |Z| > 3 typically considered anomalous
  → Assumes normal distribution
  
  Example:
  User daily login count:
  Mean (μ) = 5 logins
  Std Dev (σ) = 2
  Today: 15 logins
  Z = (15 - 5) / 2 = 5 → ANOMALY

MOVING AVERAGE:
  → Rolling window baseline
  → Adapts to gradual changes
  → Detects sudden spikes/drops
  → Good for time-series data

PERCENTILE-BASED:
  → Alert when value > 99th percentile
  → Handles non-normal distributions
  → More robust than Z-score
  → Example: Data transfer > 99th percentile

ENTROPY ANALYSIS:
  → Measures randomness/disorder
  → High entropy in DNS queries = tunneling
  → High entropy in file names = encryption/packing
  → Shannon entropy calculation

  # Python entropy calculation
  import math
  from collections import Counter
  
  def entropy(data):
      counter = Counter(data)
      length = len(data)
      return -sum(
          (count/length) * math.log2(count/length)
          for count in counter.values()
      )
  
  # Normal domain: entropy ≈ 2.5-3.5
  # Tunneling domain: entropy ≈ 4.0-5.0
```

---

## 3. Machine Learning Methods

```
ML FOR SECURITY ANOMALY DETECTION:

CLUSTERING (K-MEANS):
  → Groups similar data points
  → Points far from clusters = anomalies
  → Good for network traffic analysis
  → Requires choosing K (number of clusters)

ISOLATION FOREST:
  → Specifically designed for anomaly detection
  → Random partitioning of data
  → Anomalies require fewer partitions
  → Efficient and scalable
  → Works well with high-dimensional data

AUTOENCODERS (Neural Networks):
  → Learns to compress and reconstruct data
  → High reconstruction error = anomaly
  → Good for complex patterns
  → Requires significant training data

PRACTICAL IMPLEMENTATIONS:

  ELASTIC ML:
  → Built-in anomaly detection jobs
  → Time series analysis
  → Population analysis
  → Influencer detection
  
  # Create ML job for unusual process
  PUT _ml/anomaly_detectors/unusual_process
  {
    "analysis_config": {
      "detectors": [{
        "function": "rare",
        "by_field_name": "process.name",
        "partition_field_name": "host.name"
      }]
    },
    "data_description": {"time_field": "@timestamp"}
  }

  SPLUNK ML TOOLKIT:
  → DensityFunction for anomaly scoring
  → LogisticRegression for classification
  → KMeans for clustering
  
  | fit DensityFunction bytes_out
  | where isOutlier_bytes_out = 1

  SENTINEL ML:
  → Built-in ML behavior analytics
  → Custom ML rules
  → Fusion alerts (multi-signal correlation)
```

---

## 4. Implementing Anomaly Detection

```
IMPLEMENTATION WORKFLOW:

PHASE 1: DATA PREPARATION
  → Identify relevant data sources
  → Clean and normalize data
  → Handle missing values
  → Feature engineering
  → Establish time window

PHASE 2: BASELINE PERIOD
  → Collect 2-4 weeks of normal data
  → Validate no incidents during baseline
  → Calculate statistical baselines
  → Train ML models
  → Validate model accuracy

PHASE 3: DETECTION
  → Apply model to real-time data
  → Score anomalies
  → Set initial thresholds
  → Monitor in passive mode
  → Review results

PHASE 4: TUNING
  → Adjust thresholds
  → Exclude known patterns
  → Reduce false positives
  → Validate true positives
  → Iterate continuously

USE CASES:
  Use Case                  | Method          | Data Source
  Unusual login times       | Time-based      | Auth logs
  Data exfiltration         | Volume-based    | Proxy/DLP
  DNS tunneling             | Entropy         | DNS logs
  Beaconing                 | Periodicity     | Proxy/FW
  Unusual process           | Rarity          | Sysmon
  Lateral movement          | Graph-based     | Auth logs
  Account compromise        | Multi-signal    | Multiple
  Insider threat            | UEBA            | Multiple

CHALLENGES:
  → Establishing accurate baselines
  → Handling seasonal patterns
  → High false positive rates
  → Concept drift (normal changes over time)
  → Training data quality
  → Model explainability
  → Resource requirements
```

---

## 5. Best Practices

```
ANOMALY DETECTION BEST PRACTICES:

  [ ] Start with well-understood use cases
  [ ] Use sufficient baseline period (2-4 weeks minimum)
  [ ] Combine with rule-based detection (not replace)
  [ ] Implement feedback loop for analyst input
  [ ] Track false positive rate over time
  [ ] Retrain models periodically
  [ ] Account for business cycles (weekends, holidays)
  [ ] Use risk scoring rather than binary alerts
  [ ] Document model parameters and decisions
  [ ] Monitor model performance continuously

DEFENSE-IN-DEPTH DETECTION:
  Layer 1: IOC-based (known threats)
  Layer 2: Rule-based (known behaviors)
  Layer 3: Behavioral (known techniques)
  Layer 4: Anomaly (unknown threats)
  
  Each layer catches what others miss
  Combined = comprehensive detection

MEASURING SUCCESS:
  Metric               | Target
  True positive rate    | > 5% (for anomaly detection)
  Alert volume          | Manageable for team
  Novel threat detection| Finds threats rules missed
  Time to baseline      | < 4 weeks
  Model drift           | Retrain quarterly
```

---

## Summary Table

| Method | Approach | Best For | Complexity |
|--------|----------|----------|------------|
| Statistical | Z-score, percentile | Volume anomalies | Low |
| Entropy | Shannon entropy | DNS tunneling | Low |
| Clustering | K-Means, DBSCAN | Network traffic | Medium |
| Isolation Forest | Tree-based isolation | General anomalies | Medium |
| Autoencoder | Neural network | Complex patterns | High |
| UEBA | Multi-signal ML | User behavior | High |

---

## Revision Questions

1. How does anomaly detection complement signature-based detection?
2. What statistical methods are used for anomaly detection?
3. How does Isolation Forest work for identifying anomalies?
4. What is the importance of the baseline period?
5. What are the main challenges with anomaly detection in security?

---

*Previous: [03-behavior-based-detection.md](03-behavior-based-detection.md) | Next: [05-mitre-attack-for-detection.md](05-mitre-attack-for-detection.md)*

---

*[Back to README](../README.md)*
