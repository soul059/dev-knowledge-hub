# ML Lifecycle

## Overview

The Machine Learning lifecycle describes the end-to-end process of building, deploying, and maintaining an ML system. Unlike traditional software development where the lifecycle is primarily about code, the ML lifecycle revolves around **data**, **experiments**, and **continuous improvement**. Understanding each phase is essential for building reliable ML systems.

---

## The ML Lifecycle Phases

```
  ┌─────────────────────────────────────────────────────────────────┐
  │                                                                   │
  │  1. Problem     2. Data        3. Feature     4. Model          │
  │  Definition     Collection     Engineering    Development       │
  │  ┌─────┐       ┌─────┐       ┌─────┐       ┌─────┐            │
  │  │  ?  │──────►│ 📊 │──────►│ ⚙️ │──────►│ 🧠 │            │
  │  └─────┘       └─────┘       └─────┘       └─────┘            │
  │                                                │                 │
  │                                                ▼                 │
  │  8. Retrain    7. Monitor     6. Deploy     5. Evaluate         │
  │  ┌─────┐       ┌─────┐       ┌─────┐       ┌─────┐            │
  │  │ 🔄 │◄──────│ 👁️ │◄──────│ 🚀 │◄──────│ ✅ │            │
  │  └──┬──┘       └─────┘       └─────┘       └─────┘            │
  │     │                                                           │
  │     └──────────── Loop back to phase 2 or 3 ──────────────►    │
  │                                                                   │
  └─────────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Problem Definition

```
  Before writing any code, define:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  1. Business objective                            │
  │     "Reduce customer churn by 15%"               │
  │                                                    │
  │  2. ML task formulation                           │
  │     "Binary classification: will this customer   │
  │      churn in the next 30 days?"                  │
  │                                                    │
  │  3. Success metrics                               │
  │     Business: revenue saved from retained users  │
  │     ML: precision > 0.8, recall > 0.7            │
  │                                                    │
  │  4. Constraints                                   │
  │     Latency: < 100ms inference                   │
  │     Privacy: no PII in features                  │
  │     Fairness: equal performance across groups    │
  │                                                    │
  │  5. Baseline                                     │
  │     Current heuristic / rule-based system        │
  │     ML model must beat baseline to justify cost  │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Phase 2: Data Collection & Preparation

```
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Data Sources:                                   │
  │  ├── Databases (SQL, NoSQL)                      │
  │  ├── APIs (internal, third-party)                │
  │  ├── Event streams (Kafka, Kinesis)              │
  │  ├── Files (CSV, Parquet, JSON)                  │
  │  └── Manual labeling (annotation tools)          │
  │                                                    │
  │  Data Preparation Steps:                         │
  │  1. Ingestion — collect from all sources         │
  │  2. Cleaning — handle missing, duplicates, noise │
  │  3. Validation — schema checks, range checks     │
  │  4. Labeling — ground truth for supervised ML    │
  │  5. Splitting — train / validation / test sets   │
  │  6. Versioning — snapshot with DVC or similar    │
  │                                                    │
  │  Key principle: "Garbage in, garbage out"        │
  │  Data quality is the #1 factor in ML success     │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Phase 3: Feature Engineering

```
  Transform raw data into features the model can learn from:

  Raw data → Feature pipeline → Feature vectors

  Examples:
  ┌─────────────────────────────┬──────────────────────────┐
  │ Raw Data                    │ Engineered Feature       │
  ├─────────────────────────────┼──────────────────────────┤
  │ Timestamp: 2025-03-15 14:30│ hour_of_day: 14          │
  │                             │ is_weekend: 0            │
  │                             │ day_of_week: 6 (Sat)     │
  ├─────────────────────────────┼──────────────────────────┤
  │ Purchase history (list)     │ total_purchases: 47      │
  │                             │ avg_purchase_value: 35.2 │
  │                             │ days_since_last: 12      │
  ├─────────────────────────────┼──────────────────────────┤
  │ Text: "Great product!"     │ sentiment_score: 0.92    │
  │                             │ word_count: 2            │
  └─────────────────────────────┴──────────────────────────┘

  In production: features are served via Feature Stores
  (covered in Unit 2)
```

---

## Phase 4: Model Development

```
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Iterative experimentation:                      │
  │                                                    │
  │  1. Select model architecture                    │
  │     • Start simple (logistic regression, trees)  │
  │     • Increase complexity if needed              │
  │                                                    │
  │  2. Train model                                  │
  │     • Fit on training data                       │
  │     • Track with experiment tracker (MLflow)     │
  │                                                    │
  │  3. Tune hyperparameters                         │
  │     • Grid search, random search, Bayesian opt.  │
  │     • Log all experiments                        │
  │                                                    │
  │  4. Validate                                     │
  │     • Evaluate on validation set                 │
  │     • Cross-validation for robust estimates      │
  │                                                    │
  │  5. Iterate                                      │
  │     • Try different features, architectures      │
  │     • Compare experiments systematically         │
  │                                                    │
  │  Output: best model + training pipeline code     │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Phase 5: Model Evaluation

```
  Go beyond simple accuracy:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Offline Evaluation:                             │
  │  • Metrics on held-out test set                  │
  │  • Slice analysis (performance per subgroup)     │
  │  • Error analysis (what types of errors?)        │
  │  • Comparison against baseline                   │
  │                                                    │
  │  Pre-deployment Checks:                          │
  │  • Fairness across demographic groups            │
  │  • Model size and latency requirements met       │
  │  • No data leakage from training                 │
  │  • Performance on edge cases                     │
  │                                                    │
  │  Decision Gate:                                  │
  │  ┌──────────────────────────────────┐            │
  │  │ Metrics meet thresholds?        │            │
  │  │  YES → promote to staging       │            │
  │  │  NO  → iterate on model/data    │            │
  │  └──────────────────────────────────┘            │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Phase 6: Deployment

```
  Move model from development to production:

  Deployment options:
  ┌─────────────┬────────────────────────────────────┐
  │ Pattern     │ Description                        │
  ├─────────────┼────────────────────────────────────┤
  │ Batch       │ Run predictions on schedule        │
  │             │ (hourly, daily) on stored data     │
  ├─────────────┼────────────────────────────────────┤
  │ Real-time   │ REST/gRPC API serving predictions  │
  │             │ on demand (< 100ms latency)        │
  ├─────────────┼────────────────────────────────────┤
  │ Edge        │ Deploy to mobile/IoT devices       │
  │             │ (TFLite, ONNX, CoreML)             │
  ├─────────────┼────────────────────────────────────┤
  │ Streaming   │ Predictions on event streams       │
  │             │ (Kafka, Flink)                      │
  └─────────────┴────────────────────────────────────┘

  Deployment strategies: canary, blue-green, A/B testing
  (covered in detail in Unit 7)
```

---

## Phase 7: Monitoring

```
  After deployment, continuously watch for:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Data Drift:                                     │
  │    Input distribution changes over time          │
  │    e.g., user demographics shift                 │
  │                                                    │
  │  Concept Drift:                                  │
  │    Relationship between features and target      │
  │    changes over time                             │
  │    e.g., COVID changed buying patterns           │
  │                                                    │
  │  Performance Degradation:                        │
  │    Model accuracy drops below threshold          │
  │                                                    │
  │  System Health:                                  │
  │    Latency, throughput, error rates, resource use│
  │                                                    │
  │  Action: alert → investigate → retrain if needed │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Phase 8: Retraining

```
  When to retrain:

  1. Scheduled:    Retrain every week / month
  2. Triggered:    Retrain when drift is detected
  3. On-demand:    Retrain when new labeled data arrives
  4. Continuous:   Retrain on every new data batch (online learning)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Retraining Pipeline:                            │
  │                                                    │
  │  New data ──→ Validate ──→ Train ──→ Evaluate   │
  │                                          │        │
  │                                     Pass gate?   │
  │                                     YES → Deploy │
  │                                     NO → Alert   │
  │                                                    │
  │  This should be FULLY AUTOMATED                  │
  │  Human review only for major changes             │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## ML Lifecycle vs. Traditional SDLC

```
  ┌──────────────────┬──────────────────┬──────────────────┐
  │ Aspect           │ Traditional SDLC │ ML Lifecycle     │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Primary artifact │ Code             │ Code + Data +    │
  │                  │                  │ Model            │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Testing          │ Unit tests       │ Data tests +     │
  │                  │ Integration tests│ Model tests +    │
  │                  │                  │ Unit tests       │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Deployment       │ Deploy code once │ Deploy model,    │
  │                  │                  │ retrain often    │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Monitoring       │ Uptime, errors   │ Drift, accuracy, │
  │                  │                  │ fairness         │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Iteration        │ Feature requests │ New data, drift, │
  │                  │                  │ model updates    │
  └──────────────────┴──────────────────┴──────────────────┘
```

---

## Revision Questions

1. **What are the 8 phases of the ML lifecycle?**
2. **Why is problem definition considered the most important phase?**
3. **What is the difference between data drift and concept drift?**
4. **When should a model be retrained?**
5. **How does the ML lifecycle differ from the traditional SDLC?**

---

[Previous: 01-what-is-mlops.md](01-what-is-mlops.md) | [Next: 03-devops-vs-mlops.md](03-devops-vs-mlops.md)
