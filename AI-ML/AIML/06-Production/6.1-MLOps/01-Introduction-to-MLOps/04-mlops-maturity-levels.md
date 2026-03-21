# MLOps Maturity Levels

## Overview

MLOps maturity levels describe how automated, reliable, and scalable an organization's ML operations are. Most teams start at Level 0 (manual everything) and gradually mature toward Level 3+ (fully automated, self-healing). Understanding these levels helps teams identify where they are and plan a realistic roadmap.

---

## The MLOps Maturity Model

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Level 0: Manual / No MLOps                                  │
  │  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░            │
  │                                                                │
  │  Level 1: DevOps but no MLOps                                │
  │  ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░            │
  │                                                                │
  │  Level 2: Automated Training                                 │
  │  ██████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░            │
  │                                                                │
  │  Level 3: Automated Deployment                               │
  │  ████████████████████████░░░░░░░░░░░░░░░░░░░░░░            │
  │                                                                │
  │  Level 4: Full MLOps (Automated Retraining)                  │
  │  ████████████████████████████████████████████████            │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Level 0: Manual Process

```
  ┌──────────────────────────────────────────────────┐
  │  LEVEL 0: Manual / No MLOps                      │
  │                                                    │
  │  How it works:                                   │
  │  • Data scientist works in Jupyter notebooks     │
  │  • Manual data download and preprocessing        │
  │  • Train model locally, pick best by gut feel    │
  │  • Hand off model file to engineering team       │
  │  • Engineer manually deploys to server           │
  │  • No monitoring, no retraining                  │
  │                                                    │
  │  Characteristics:                                │
  │  ✗ No version control for data or models         │
  │  ✗ No experiment tracking                        │
  │  ✗ No automated testing                          │
  │  ✗ No monitoring                                 │
  │  ✗ Deployment takes days/weeks                   │
  │  ✗ Not reproducible                              │
  │                                                    │
  │  Typical team: 1-2 data scientists               │
  │  Most ML teams start here!                       │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Level 1: DevOps but No MLOps

```
  ┌──────────────────────────────────────────────────┐
  │  LEVEL 1: DevOps without MLOps                   │
  │                                                    │
  │  How it works:                                   │
  │  • Code is in Git with CI/CD                     │
  │  • Training code runs in scripts (not notebooks) │
  │  • Model is deployed via standard CI/CD pipeline │
  │  • Basic application monitoring (latency, errors)│
  │                                                    │
  │  What's added vs. Level 0:                       │
  │  ✓ Version control for code                      │
  │  ✓ CI/CD for deployment                          │
  │  ✓ Basic infrastructure automation               │
  │                                                    │
  │  What's still missing:                           │
  │  ✗ No data versioning                            │
  │  ✗ No experiment tracking                        │
  │  ✗ No model registry                             │
  │  ✗ No data/model monitoring                      │
  │  ✗ No automated retraining                       │
  │  ✗ Retraining is manual and ad-hoc               │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Level 2: Automated Training Pipeline

```
  ┌──────────────────────────────────────────────────┐
  │  LEVEL 2: Automated Training                     │
  │                                                    │
  │  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐     │
  │  │ Data │──►│Train │──►│ Eval │──►│Regis.│     │
  │  │Valid.│   │Model │   │Model │   │Model │     │
  │  └──────┘   └──────┘   └──────┘   └──────┘     │
  │       ▲         automated pipeline                │
  │       │                                            │
  │  Trigger: schedule, new data, manual              │
  │                                                    │
  │  What's added vs. Level 1:                       │
  │  ✓ Experiment tracking (MLflow, W&B)             │
  │  ✓ Data versioning (DVC)                         │
  │  ✓ Automated training pipeline                   │
  │  ✓ Model registry                                │
  │  ✓ Model evaluation gates                        │
  │                                                    │
  │  What's still missing:                           │
  │  ✗ Deployment is still manual/semi-manual        │
  │  ✗ No automated monitoring/alerting              │
  │  ✗ No automatic retraining triggers              │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Level 3: Automated Deployment

```
  ┌──────────────────────────────────────────────────┐
  │  LEVEL 3: Automated Training + Deployment        │
  │                                                    │
  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐  │
  │  │ Data │►│Train │►│ Eval │►│Regis.│►│Deploy│  │
  │  │Valid.│ │Model │ │Model │ │Model │ │Model │  │
  │  └──────┘ └──────┘ └──────┘ └──────┘ └──┬───┘  │
  │                                          │       │
  │       ┌──────────────────────────────────┘       │
  │       ▼                                           │
  │  ┌──────┐   ┌──────┐                             │
  │  │Canary│──►│  Full │                             │
  │  │Deploy│   │Rollout│                             │
  │  └──────┘   └──────┘                             │
  │                                                    │
  │  What's added vs. Level 2:                       │
  │  ✓ Automated model deployment (CD for models)    │
  │  ✓ Canary / A-B testing deployments              │
  │  ✓ Automated rollback on failure                 │
  │  ✓ Model serving infrastructure                  │
  │  ✓ Basic model monitoring                        │
  │                                                    │
  │  What's still missing:                           │
  │  ✗ No automatic retraining loop                  │
  │  ✗ Monitoring doesn't trigger retraining         │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Level 4: Full MLOps (Closed Loop)

```
  ┌──────────────────────────────────────────────────────────────┐
  │  LEVEL 4: Full MLOps — Continuous Everything                 │
  │                                                                │
  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐    │
  │  │ Data │►│Train │►│ Eval │►│Regis.│►│Deploy│►│Monit.│    │
  │  │Valid.│ │Model │ │Model │ │Model │ │Model │ │& Log │    │
  │  └──┬───┘ └──────┘ └──────┘ └──────┘ └──────┘ └──┬───┘    │
  │     ▲                                              │         │
  │     │           CLOSED LOOP                        │         │
  │     │                                              ▼         │
  │     │  ┌────────────────────────────────────────────────┐    │
  │     │  │ Drift detected? Performance dropped?          │    │
  │     │  │ YES → trigger retraining automatically        │    │
  │     └──│ Fetch new data → re-enter pipeline            │    │
  │        └────────────────────────────────────────────────┘    │
  │                                                                │
  │  Everything automated:                                       │
  │  ✓ CT: Continuous Training (auto-retrain on drift)           │
  │  ✓ CI: Continuous Integration (test code + data + model)     │
  │  ✓ CD: Continuous Deployment (auto-deploy validated models)  │
  │  ✓ CM: Continuous Monitoring (drift, fairness, performance)  │
  │  ✓ Feature stores for consistency                            │
  │  ✓ A/B testing with automatic winner selection               │
  │  ✓ Model governance and audit trails                         │
  │  ✓ Self-healing: auto-rollback on degradation                │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Google's MLOps Maturity Model

```
  Google defines 3 levels (widely referenced):

  ┌─────────┬───────────────────────────────────────────────┐
  │ Level   │ Description                                   │
  ├─────────┼───────────────────────────────────────────────┤
  │ Level 0 │ Manual process                                │
  │         │ • No CI/CD, no automation                     │
  │         │ • Notebook-driven development                 │
  │         │ • Disconnect between ML and ops               │
  ├─────────┼───────────────────────────────────────────────┤
  │ Level 1 │ ML pipeline automation                        │
  │         │ • Automated training pipeline                 │
  │         │ • CT: retrain in production                   │
  │         │ • Feature store for consistency               │
  │         │ • Metadata and artifact management            │
  ├─────────┼───────────────────────────────────────────────┤
  │ Level 2 │ CI/CD pipeline automation                     │
  │         │ • CI: test pipeline components                │
  │         │ • CD: deploy pipeline to production           │
  │         │ • Automated model validation and deployment   │
  │         │ • Monitor, retrain, redeploy automatically    │
  └─────────┴───────────────────────────────────────────────┘
```

---

## Maturity Assessment Checklist

```
  Rate your team (✓ = yes, ✗ = no):

  ┌────────────────────────────────────┬──┬──┬──┬──┬──┐
  │ Practice                           │L0│L1│L2│L3│L4│
  ├────────────────────────────────────┼──┼──┼──┼──┼──┤
  │ Code in version control            │  │✓ │✓ │✓ │✓ │
  │ Training code as scripts           │  │✓ │✓ │✓ │✓ │
  │ CI/CD for application code         │  │✓ │✓ │✓ │✓ │
  │ Data versioning                    │  │  │✓ │✓ │✓ │
  │ Experiment tracking                │  │  │✓ │✓ │✓ │
  │ Automated training pipeline        │  │  │✓ │✓ │✓ │
  │ Model registry                     │  │  │✓ │✓ │✓ │
  │ Automated model deployment         │  │  │  │✓ │✓ │
  │ Canary/A-B deployments             │  │  │  │✓ │✓ │
  │ Model monitoring                   │  │  │  │✓ │✓ │
  │ Automated rollback                 │  │  │  │✓ │✓ │
  │ Drift-triggered retraining         │  │  │  │  │✓ │
  │ Feature store                      │  │  │  │  │✓ │
  │ Model governance & audit           │  │  │  │  │✓ │
  │ Self-healing pipeline              │  │  │  │  │✓ │
  └────────────────────────────────────┴──┴──┴──┴──┴──┘
```

---

## Revision Questions

1. **What are the key characteristics of MLOps Level 0?**
2. **What is the main difference between Level 2 and Level 3?**
3. **What makes Level 4 a "closed loop" system?**
4. **How does Google's MLOps maturity model differ from the 5-level model?**
5. **Name five practices that distinguish Level 4 from Level 2.**

---

[Previous: 03-devops-vs-mlops.md](03-devops-vs-mlops.md) | [Next: 05-challenges-in-ml-production.md](05-challenges-in-ml-production.md)
