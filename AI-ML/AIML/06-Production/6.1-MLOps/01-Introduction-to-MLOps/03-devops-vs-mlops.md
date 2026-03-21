# DevOps vs MLOps

## Overview

MLOps builds on the principles of DevOps but extends them to handle the unique challenges of machine learning systems. While DevOps focuses on automating software delivery, MLOps must also manage **data**, **models**, **experiments**, and **continuous training**. This chapter compares the two disciplines and highlights where MLOps diverges.

---

## DevOps Principles (Quick Recap)

```
  DevOps core practices:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  CI (Continuous Integration):                    │
  │    Code commit → build → test → merge            │
  │                                                    │
  │  CD (Continuous Delivery/Deployment):            │
  │    Merge → package → deploy → release            │
  │                                                    │
  │  IaC (Infrastructure as Code):                   │
  │    Define infra in config files (Terraform, etc.)│
  │                                                    │
  │  Monitoring & Observability:                     │
  │    Logs, metrics, traces, alerts                 │
  │                                                    │
  │  Automation:                                     │
  │    Minimize manual steps in the pipeline         │
  │                                                    │
  │  Collaboration:                                  │
  │    Break down silos between dev and ops teams    │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Side-by-Side Comparison

```
  ┌────────────────────┬────────────────────┬────────────────────────┐
  │ Aspect             │ DevOps             │ MLOps                  │
  ├────────────────────┼────────────────────┼────────────────────────┤
  │ What you build     │ Software           │ ML models + software   │
  ├────────────────────┼────────────────────┼────────────────────────┤
  │ Primary artifact   │ Application code   │ Code + data + model    │
  ├────────────────────┼────────────────────┼────────────────────────┤
  │ Version control    │ Code (Git)         │ Code + data + model    │
  │                    │                    │ (Git + DVC + registry) │
  ├────────────────────┼────────────────────┼────────────────────────┤
  │ CI                 │ Build + test code  │ Build + test code +    │
  │                    │                    │ validate data + train  │
  │                    │                    │ + evaluate model       │
  ├────────────────────┼────────────────────┼────────────────────────┤
  │ CD                 │ Deploy application │ Deploy model + serving │
  │                    │                    │ infrastructure         │
  ├────────────────────┼────────────────────┼────────────────────────┤
  │ CT                 │ N/A                │ Continuous Training    │
  │                    │                    │ (retrain on new data)  │
  ├────────────────────┼────────────────────┼────────────────────────┤
  │ Testing            │ Unit, integration, │ + data tests, model    │
  │                    │ E2E tests          │ tests, fairness tests  │
  ├────────────────────┼────────────────────┼────────────────────────┤
  │ Monitoring         │ Uptime, latency,   │ + data drift, concept  │
  │                    │ error rates        │ drift, model accuracy  │
  ├────────────────────┼────────────────────┼────────────────────────┤
  │ Rollback           │ Revert code deploy │ Revert model version + │
  │                    │                    │ possibly data version  │
  ├────────────────────┼────────────────────┼────────────────────────┤
  │ Team               │ Devs + Ops         │ Data scientists + ML   │
  │                    │                    │ eng + data eng + ops   │
  └────────────────────┴────────────────────┴────────────────────────┘
```

---

## What MLOps Adds Beyond DevOps

```
  MLOps introduces concepts that don't exist in traditional DevOps:

  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. DATA VERSIONING                                          │
  │     DevOps versions code.                                    │
  │     MLOps versions code + data + models.                     │
  │     Tools: DVC, LakeFS, Delta Lake                           │
  │                                                                │
  │  2. EXPERIMENT TRACKING                                      │
  │     DevOps has no concept of experiments.                     │
  │     MLOps tracks every training run: params, metrics,        │
  │     artifacts, code version, data version.                   │
  │     Tools: MLflow, W&B, Neptune                              │
  │                                                                │
  │  3. CONTINUOUS TRAINING (CT)                                 │
  │     DevOps: deploy new code when written.                    │
  │     MLOps: retrain model when data changes or drifts.        │
  │     Trigger: scheduled, data-driven, performance-driven.     │
  │                                                                │
  │  4. MODEL REGISTRY                                           │
  │     Like a container registry, but for models.               │
  │     Stage models: None → Staging → Production → Archived    │
  │     Tools: MLflow Registry, SageMaker Model Registry         │
  │                                                                │
  │  5. FEATURE STORES                                           │
  │     Centralized place to compute, store, and serve features. │
  │     Ensures consistency between training and serving.        │
  │     Tools: Feast, Tecton, Hopsworks                          │
  │                                                                │
  │  6. DATA/MODEL MONITORING                                    │
  │     Beyond system health: monitor data distributions,        │
  │     model predictions, drift, and fairness.                  │
  │     Tools: Evidently, WhyLabs, NannyML                       │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## CI/CD/CT Pipeline Comparison

```
  DevOps CI/CD:
  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
  │Commit│──►│Build │──►│ Test │──►│Package│──►│Deploy│
  └──────┘   └──────┘   └──────┘   └──────┘   └──────┘

  MLOps CI/CD/CT:
  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
  │ Data │──►│Valid.│──►│Train │──►│ Eval │──►│Deploy│
  │Change│   │ Data │   │Model │   │Model │   │Model │
  └──────┘   └──────┘   └──────┘   └──┬───┘   └──────┘
                                       │
                                  Pass gate?
                                  ├── YES → Deploy
                                  └── NO  → Alert team

  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
  │Code  │──►│Build │──►│ Test │──►│Merge │
  │Change│   │ Pipe │   │ Pipe │   │      │
  └──────┘   └──────┘   └──────┘   └──────┘

  TWO pipelines running in parallel:
  1. Code pipeline (like DevOps)
  2. Data/model pipeline (unique to MLOps)
```

---

## The Training-Serving Skew Problem

```
  A uniquely ML problem that DevOps never faces:

  Training:
    Features computed in batch on historical data
    Using Python/pandas in a notebook
    
  Serving:
    Features computed in real-time on live data
    Using Java/Go in a microservice

  If the feature computation differs even slightly:
    → Model behaves differently in production!
    → Called "training-serving skew"

  ┌──────────────────────────────────────────────────┐
  │  Example:                                         │
  │                                                    │
  │  Training: age = 2025 - birth_year   (integer)   │
  │  Serving:  age = now() - birthday    (float!)    │
  │                                                    │
  │  Tiny difference → model predictions change!     │
  │                                                    │
  │  Solution: Feature Stores ensure same code       │
  │  computes features for both training and serving │
  └──────────────────────────────────────────────────┘
```

---

## When to Use DevOps vs MLOps

```
  Use DevOps when:
    • Building traditional web apps, APIs, microservices
    • No machine learning involved
    • Business logic is explicit (if/else rules)

  Use MLOps when:
    • Model predictions are part of the product
    • Data changes over time
    • Model needs retraining
    • Need to track experiments and model versions
    • Need to monitor for data/concept drift

  Many teams need BOTH:
    MLOps for the model pipeline
    DevOps for the application that consumes predictions
```

---

## Revision Questions

1. **What does MLOps add beyond traditional DevOps?**
2. **What is Continuous Training (CT) and why doesn't DevOps need it?**
3. **What is training-serving skew and how do Feature Stores solve it?**
4. **Compare CI pipelines in DevOps vs MLOps.**
5. **Why does MLOps require versioning data and models, not just code?**

---

[Previous: 02-ml-lifecycle.md](02-ml-lifecycle.md) | [Next: 04-mlops-maturity-levels.md](04-mlops-maturity-levels.md)
