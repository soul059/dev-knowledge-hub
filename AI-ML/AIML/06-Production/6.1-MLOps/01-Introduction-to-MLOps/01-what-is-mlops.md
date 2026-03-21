# What is MLOps?

## Overview

**MLOps** (Machine Learning Operations) is a set of practices, tools, and cultural philosophies that unify ML system development (Dev) and ML system operations (Ops). It aims to automate and streamline the end-to-end ML lifecycle — from data preparation and model training to deployment, monitoring, and retraining. MLOps applies the lessons learned from DevOps and software engineering to the unique challenges of machine learning.

---

## Why MLOps Exists

```
The problem: most ML models never make it to production

  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Research / Jupyter Notebook World:                           │
  │    • Train model → good accuracy → done! ✅                 │
  │                                                                │
  │  Production Reality:                                          │
  │    • Train model → deploy → monitor → data changes →        │
  │      retrain → redeploy → scale → debug → repeat forever    │
  │                                                                │
  │  Statistics:                                                  │
  │    • ~87% of ML projects never reach production              │
  │    • Average time from prototype to production: 31 days      │
  │    • Only 22% of companies using ML have successfully        │
  │      deployed a model                                        │
  │                                                                │
  │  Root cause: ML has unique operational challenges that       │
  │  traditional software engineering doesn't address            │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## What Makes ML Different from Traditional Software

```
  Traditional Software:              ML Systems:
  ┌─────────────────────┐           ┌─────────────────────┐
  │                     │           │                     │
  │  Code → Behavior    │           │  Code + Data +      │
  │                     │           │  Model → Behavior   │
  │  Deterministic      │           │  Probabilistic      │
  │  Logic is explicit  │           │  Logic is learned   │
  │  Test with asserts  │           │  Test with metrics  │
  │  Bug = code defect  │           │  Bug = data issue,  │
  │                     │           │  drift, or both     │
  │  Deploy once,       │           │  Deploy, monitor,   │
  │  maintain code      │           │  retrain, redeploy  │
  │                     │           │                     │
  └─────────────────────┘           └─────────────────────┘

  In ML, the model behavior depends on THREE things:
    1. Code (preprocessing, model architecture)
    2. Data (training data, feature distributions)
    3. Configuration (hyperparameters, thresholds)

  Change ANY of these → behavior changes!
  → Need to version and track ALL of them
```

---

## MLOps = DevOps + Data + Models

```
  ┌──────────────────────────────────────────────────────────────┐
  │                         MLOps                                  │
  │                                                                │
  │  ┌──────────┐    ┌──────────┐    ┌──────────┐               │
  │  │  DevOps  │    │   Data   │    │  Model   │               │
  │  │          │  + │  Eng.    │  + │  Mgmt.   │               │
  │  │ CI/CD    │    │ Pipelines│    │ Training │               │
  │  │ IaC      │    │ Quality  │    │ Serving  │               │
  │  │ Monitor  │    │ Version  │    │ Monitor  │               │
  │  └──────────┘    └──────────┘    └──────────┘               │
  │                                                                │
  │  Key Principles:                                              │
  │  1. Automation — minimize manual steps                       │
  │  2. Reproducibility — same data + code = same model          │
  │  3. Versioning — track code, data, models, configs           │
  │  4. Monitoring — detect issues before users do               │
  │  5. Collaboration — data scientists + ML engineers + ops     │
  │  6. Continuous — continuous training, deployment, monitoring  │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Core Components of MLOps

```
  ┌─────────────────────┬──────────────────────────────────────┐
  │ Component           │ Purpose                              │
  ├─────────────────────┼──────────────────────────────────────┤
  │ Data Management     │ Version, validate, and pipeline data │
  │ Experiment Tracking │ Log metrics, params, and artifacts   │
  │ Model Registry      │ Store and version trained models     │
  │ Pipeline Orchestr.  │ Automate training workflows          │
  │ Model Serving       │ Deploy models for inference          │
  │ Monitoring          │ Track model performance and drift    │
  │ CI/CD for ML        │ Automate testing and deployment      │
  │ Feature Store       │ Centralize feature engineering       │
  │ Infrastructure      │ Manage compute resources (GPU, K8s)  │
  │ Governance          │ Ensure fairness, compliance, audit   │
  └─────────────────────┴──────────────────────────────────────┘
```

---

## MLOps Workflow (High Level)

```
  ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐
  │  Data  │───►│ Model  │───►│ Model  │───►│  Model │
  │  Prep  │    │Training│    │  Eval  │    │  Deploy│
  └────────┘    └────────┘    └────────┘    └────┬───┘
      ▲                                          │
      │                                          ▼
  ┌───┴────┐                              ┌────────┐
  │  Data  │◄─────── Trigger ◄────────────│Monitor │
  │ Update │         Retrain              │& Drift │
  └────────┘                              └────────┘

  This is a CONTINUOUS LOOP, not a one-time pipeline!

  CT (Continuous Training):  Retrain when data/performance changes
  CI (Continuous Integration): Test code, data, and model changes
  CD (Continuous Delivery):    Auto-deploy validated models
```

---

## Key Roles in MLOps

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Role             │ Responsibilities                     │
  ├──────────────────┼──────────────────────────────────────┤
  │ Data Scientist   │ EDA, feature engineering, model dev  │
  │ ML Engineer      │ Production code, pipelines, serving  │
  │ Data Engineer    │ Data pipelines, storage, quality     │
  │ MLOps Engineer   │ Infrastructure, CI/CD, monitoring    │
  │ Platform Eng.    │ ML platform, tooling, self-service   │
  └──────────────────┴──────────────────────────────────────┘

  In smaller teams, one person may wear multiple hats!
```

---

## Popular MLOps Tool Ecosystem

```
  ┌──────────────────────────────────────────────────────────────┐
  │                     MLOps Tool Landscape                      │
  │                                                                │
  │  End-to-End Platforms:                                       │
  │    AWS SageMaker, GCP Vertex AI, Azure ML, Databricks        │
  │                                                                │
  │  Open-Source Ecosystem:                                      │
  │    ┌────────────┐ ┌────────────┐ ┌────────────┐             │
  │    │  MLflow    │ │   DVC      │ │  Kubeflow  │             │
  │    │ (tracking) │ │ (data ver.)│ │ (pipelines)│             │
  │    └────────────┘ └────────────┘ └────────────┘             │
  │    ┌────────────┐ ┌────────────┐ ┌────────────┐             │
  │    │  BentoML   │ │  Feast     │ │ Evidently  │             │
  │    │ (serving)  │ │(feat.store)│ │(monitoring)│             │
  │    └────────────┘ └────────────┘ └────────────┘             │
  │    ┌────────────┐ ┌────────────┐ ┌────────────┐             │
  │    │  ZenML     │ │  Prefect   │ │ Great Exp. │             │
  │    │(framework) │ │(orchestr.) │ │(data valid)│             │
  │    └────────────┘ └────────────┘ └────────────┘             │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What is MLOps and why is it needed?**
2. **How do ML systems differ from traditional software systems?**
3. **What are the three things that determine ML model behavior?**
4. **List the core components of an MLOps platform.**
5. **What does the continuous loop (CT/CI/CD) mean in MLOps?**

---

[Next: 02-ml-lifecycle.md](02-ml-lifecycle.md)
