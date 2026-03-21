# Challenges in ML Production

## Overview

Getting an ML model from a notebook to production is fundamentally different from deploying traditional software. The challenges span **data management**, **model behavior**, **infrastructure**, **organizational processes**, and **regulatory compliance**. This chapter catalogs the most common production ML challenges and how MLOps practices address them.

---

## The "Hidden Technical Debt" of ML Systems

```
  From Google's famous paper (Sculley et al., 2015):
  "Only a small fraction of real-world ML systems is the ML code"

  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  ┌─────────────────────────────────────────────────────┐     │
  │  │              Configuration                          │     │
  │  ├──────────┬─────────────────────────────┬────────────┤     │
  │  │ Data     │                             │ Serving    │     │
  │  │ Collect. │                             │ Infra      │     │
  │  ├──────────┤      ┌───────────┐         ├────────────┤     │
  │  │ Data     │      │  ML Code  │         │ Monitoring │     │
  │  │ Verif.   │      │  (tiny!)  │         │            │     │
  │  ├──────────┤      └───────────┘         ├────────────┤     │
  │  │ Feature  │                             │ Process    │     │
  │  │ Extract. │                             │ Mgmt.      │     │
  │  ├──────────┴─────────────────────────────┴────────────┤     │
  │  │              Machine Resource Management            │     │
  │  └─────────────────────────────────────────────────────┘     │
  │                                                                │
  │  The ML model code is a tiny box surrounded by a massive     │
  │  ecosystem of supporting infrastructure!                     │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Data Challenges

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. DATA QUALITY                                             │
  │     • Missing values, outliers, duplicates                   │
  │     • Label noise (incorrect ground truth)                   │
  │     • Inconsistent formats across sources                    │
  │     → Solution: data validation pipelines (Great Expectations)│
  │                                                                │
  │  2. DATA DRIFT                                               │
  │     • Input data distribution changes over time              │
  │     • e.g., new user demographics, seasonal changes          │
  │     → Solution: statistical drift monitoring (Evidently)     │
  │                                                                │
  │  3. DATA VERSIONING                                          │
  │     • Can't reproduce results without exact data snapshot    │
  │     • Datasets are too large for Git                         │
  │     → Solution: DVC, LakeFS, Delta Lake                     │
  │                                                                │
  │  4. TRAINING-SERVING SKEW                                    │
  │     • Features computed differently at train vs. serve time  │
  │     → Solution: Feature Stores (Feast, Tecton)              │
  │                                                                │
  │  5. DATA PRIVACY                                             │
  │     • PII in training data                                   │
  │     • GDPR right-to-deletion affects model                   │
  │     → Solution: data anonymization, differential privacy    │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Model Challenges

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. REPRODUCIBILITY                                          │
  │     • Same code + same data ≠ same model (random seeds,      │
  │       GPU nondeterminism, library versions)                  │
  │     → Solution: pin versions, set seeds, containerize       │
  │                                                                │
  │  2. CONCEPT DRIFT                                            │
  │     • The relationship between features and target changes   │
  │     • e.g., "what makes a good recommendation" evolves      │
  │     → Solution: continuous monitoring + retraining triggers  │
  │                                                                │
  │  3. FEEDBACK LOOPS                                           │
  │     • Model predictions influence future training data       │
  │     • e.g., recommendation system creates filter bubble     │
  │     → Solution: awareness, counterfactual evaluation,       │
  │       exploration strategies                                 │
  │                                                                │
  │  4. MODEL STALENESS                                          │
  │     • Model trained on old data loses relevance              │
  │     → Solution: scheduled or triggered retraining           │
  │                                                                │
  │  5. UNDECLARED CONSUMERS                                     │
  │     • Other teams start using your model's output            │
  │     • Breaking change in model → breaks their system        │
  │     → Solution: model contracts, versioned APIs             │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Infrastructure Challenges

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. COMPUTE COSTS                                            │
  │     • GPU training is expensive ($2-10/hr per GPU)           │
  │     • Large models need multi-GPU / multi-node               │
  │     → Solution: spot instances, efficient architectures,    │
  │       training optimization                                  │
  │                                                                │
  │  2. LATENCY REQUIREMENTS                                     │
  │     • Real-time serving needs < 50-100ms                     │
  │     • Large models are slow (BERT: ~50ms on GPU)             │
  │     → Solution: model optimization (distillation, quant.),  │
  │       batching, caching, edge deployment                     │
  │                                                                │
  │  3. SCALING                                                  │
  │     • Traffic spikes (Black Friday, viral events)            │
  │     • Need auto-scaling for model serving                    │
  │     → Solution: Kubernetes, serverless, load balancers      │
  │                                                                │
  │  4. GPU MANAGEMENT                                           │
  │     • Sharing GPUs across teams                              │
  │     • Scheduling training jobs efficiently                   │
  │     → Solution: Kubernetes + GPU scheduling, NVIDIA MIG     │
  │                                                                │
  │  5. MULTI-ENVIRONMENT PARITY                                 │
  │     • "Works on my machine" but fails in production          │
  │     → Solution: Docker containers, identical environments   │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Organizational Challenges

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. TEAM SILOS                                               │
  │     Data scientists → throw model over the wall → engineers  │
  │     → Solution: cross-functional teams, shared tooling      │
  │                                                                │
  │  2. SKILL GAPS                                               │
  │     • Data scientists lack engineering skills                │
  │     • Engineers lack ML understanding                        │
  │     → Solution: MLOps platforms that abstract complexity,   │
  │       training, pair programming                             │
  │                                                                │
  │  3. EXPERIMENT CHAOS                                         │
  │     • "Which model was best? What data did we use?"          │
  │     • Lost experiments, unreproducible results               │
  │     → Solution: experiment tracking (MLflow, W&B)           │
  │                                                                │
  │  4. LACK OF STANDARDS                                        │
  │     • Every team uses different tools, naming, processes     │
  │     → Solution: ML platforms, standardized templates        │
  │                                                                │
  │  5. DEPLOYMENT VELOCITY                                      │
  │     • "It takes 3 months to deploy a model"                  │
  │     → Solution: automated pipelines, CI/CD for ML           │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Testing Challenges

```
  ML systems are harder to test than traditional software:

  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Traditional: f(x) = deterministic result                    │
  │  ML:          f(x) = probabilistic result (changes!)        │
  │                                                                │
  │  What to test:                                               │
  │  • Data schema and quality (unit tests for data)             │
  │  • Feature pipeline correctness                              │
  │  • Model performance thresholds (not equality checks!)       │
  │  • Fairness across demographic groups                        │
  │  • Inference latency and throughput                          │
  │  • Training pipeline end-to-end                              │
  │  • Model behavior on edge cases                              │
  │                                                                │
  │  Key insight: ML tests use THRESHOLDS, not exact values!    │
  │  assert accuracy > 0.85  (not assert accuracy == 0.8732)    │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Summary: Challenges and MLOps Solutions

```
  ┌──────────────────────┬──────────────────────────────────────┐
  │ Challenge            │ MLOps Solution                       │
  ├──────────────────────┼──────────────────────────────────────┤
  │ Data quality         │ Data validation pipelines            │
  │ Data drift           │ Statistical monitoring (Evidently)   │
  │ Data versioning      │ DVC, Delta Lake                      │
  │ Training-serving skew│ Feature stores (Feast)               │
  │ Reproducibility      │ Containers, version pinning, seeds   │
  │ Concept drift        │ Performance monitoring + retraining  │
  │ Experiment chaos     │ MLflow, W&B tracking                 │
  │ Slow deployment      │ CI/CD pipelines for ML               │
  │ Model staleness      │ Scheduled / triggered retraining     │
  │ Scaling              │ Kubernetes, auto-scaling             │
  │ Cost management      │ Spot instances, model optimization   │
  │ Team silos           │ Shared platforms, cross-functional   │
  │ Testing ML systems   │ Data tests, model tests, thresholds  │
  │ Governance           │ Model registry, audit trails         │
  └──────────────────────┴──────────────────────────────────────┘
```

---

## Revision Questions

1. **What does Google's "Hidden Technical Debt" paper reveal about ML systems?**
2. **What are the five major data challenges in production ML?**
3. **How do feedback loops create problems in ML systems?**
4. **Why is testing ML systems fundamentally different from testing traditional software?**
5. **Name three organizational challenges that slow down ML deployment.**

---

[Previous: 04-mlops-maturity-levels.md](04-mlops-maturity-levels.md) | [Next: ../02-Data-Management/01-data-versioning-dvc.md](../02-Data-Management/01-data-versioning-dvc.md)
