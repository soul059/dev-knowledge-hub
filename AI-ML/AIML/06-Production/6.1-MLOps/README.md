# 6.1 MLOps — Complete Study Notes

## Course Overview

MLOps (Machine Learning Operations) bridges the gap between ML model development and production deployment. This course covers the full ML lifecycle — from data management and experiment tracking to model serving, monitoring, and responsible AI governance. You'll learn the tools, patterns, and best practices used by industry teams to reliably deploy and maintain ML systems at scale.

---

## Table of Contents

### Unit 1: Introduction to MLOps
- [01 - What is MLOps?](01-Introduction-to-MLOps/01-what-is-mlops.md)
- [02 - ML Lifecycle](01-Introduction-to-MLOps/02-ml-lifecycle.md)
- [03 - DevOps vs MLOps](01-Introduction-to-MLOps/03-devops-vs-mlops.md)
- [04 - MLOps Maturity Levels](01-Introduction-to-MLOps/04-mlops-maturity-levels.md)
- [05 - Challenges in ML Production](01-Introduction-to-MLOps/05-challenges-in-ml-production.md)

### Unit 2: Data Management
- [01 - Data Versioning (DVC)](02-Data-Management/01-data-versioning-dvc.md)
- [02 - Data Validation](02-Data-Management/02-data-validation.md)
- [03 - Data Pipelines](02-Data-Management/03-data-pipelines.md)
- [04 - Feature Stores](02-Data-Management/04-feature-stores.md)
- [05 - Data Quality Monitoring](02-Data-Management/05-data-quality-monitoring.md)

### Unit 3: Experiment Tracking
- [01 - Why Track Experiments?](03-Experiment-Tracking/01-why-track-experiments.md)
- [02 - MLflow Tracking](03-Experiment-Tracking/02-mlflow-tracking.md)
- [03 - Weights & Biases](03-Experiment-Tracking/03-weights-and-biases.md)
- [04 - Experiment Comparison](03-Experiment-Tracking/04-experiment-comparison.md)
- [05 - Hyperparameter Logging](03-Experiment-Tracking/05-hyperparameter-logging.md)
- [06 - Artifact Management](03-Experiment-Tracking/06-artifact-management.md)

### Unit 4: Model Training Pipelines
- [01 - Pipeline Orchestration](04-Model-Training-Pipelines/01-pipeline-orchestration.md)
- [02 - Kubeflow Pipelines](04-Model-Training-Pipelines/02-kubeflow-pipelines.md)
- [03 - Apache Airflow for ML](04-Model-Training-Pipelines/03-apache-airflow-for-ml.md)
- [04 - Automated Training](04-Model-Training-Pipelines/04-automated-training.md)
- [05 - Distributed Training](04-Model-Training-Pipelines/05-distributed-training.md)

### Unit 5: Model Versioning and Registry
- [01 - Model Versioning](05-Model-Versioning-and-Registry/01-model-versioning.md)
- [02 - MLflow Model Registry](05-Model-Versioning-and-Registry/02-mlflow-model-registry.md)
- [03 - Model Metadata](05-Model-Versioning-and-Registry/03-model-metadata.md)
- [04 - Model Lineage](05-Model-Versioning-and-Registry/04-model-lineage.md)
- [05 - Model Promotion](05-Model-Versioning-and-Registry/05-model-promotion.md)

### Unit 6: Model Serving
- [01 - Serving Patterns (Batch, Real-Time)](06-Model-Serving/01-serving-patterns.md)
- [02 - REST APIs for ML](06-Model-Serving/02-rest-apis-for-ml.md)
- [03 - Model Servers (TF Serving, TorchServe, Triton)](06-Model-Serving/03-model-servers.md)
- [04 - Containerizing Models](06-Model-Serving/04-containerizing-models.md)
- [05 - Serverless Inference](06-Model-Serving/05-serverless-inference.md)

### Unit 7: Model Deployment
- [01 - Deployment Strategies](07-Model-Deployment/01-deployment-strategies.md)
- [02 - A/B Testing](07-Model-Deployment/02-ab-testing.md)
- [03 - Canary Deployments](07-Model-Deployment/03-canary-deployments.md)
- [04 - Blue-Green Deployments](07-Model-Deployment/04-blue-green-deployments.md)
- [05 - Feature Flags](07-Model-Deployment/05-feature-flags.md)
- [06 - Rollback Strategies](07-Model-Deployment/06-rollback-strategies.md)

### Unit 8: Model Monitoring
- [01 - Data Drift Detection](08-Model-Monitoring/01-data-drift-detection.md)
- [02 - Concept Drift](08-Model-Monitoring/02-concept-drift.md)
- [03 - Performance Monitoring](08-Model-Monitoring/03-performance-monitoring.md)
- [04 - Alerting](08-Model-Monitoring/04-alerting.md)
- [05 - Logging and Debugging](08-Model-Monitoring/05-logging-and-debugging.md)
- [06 - Feedback Loops](08-Model-Monitoring/06-feedback-loops.md)

### Unit 9: Infrastructure
- [01 - Cloud Platforms for ML (AWS, GCP, Azure)](09-Infrastructure/01-cloud-platforms-for-ml.md)
- [02 - Kubernetes for ML](09-Infrastructure/02-kubernetes-for-ml.md)
- [03 - GPU Management](09-Infrastructure/03-gpu-management.md)
- [04 - Cost Optimization](09-Infrastructure/04-cost-optimization.md)
- [05 - Scaling Strategies](09-Infrastructure/05-scaling-strategies.md)

### Unit 10: ML Testing
- [01 - Unit Testing ML Code](10-ML-Testing/01-unit-testing-ml-code.md)
- [02 - Data Testing](10-ML-Testing/02-data-testing.md)
- [03 - Model Testing](10-ML-Testing/03-model-testing.md)
- [04 - Integration Testing](10-ML-Testing/04-integration-testing.md)
- [05 - Shadow Mode Testing](10-ML-Testing/05-shadow-mode-testing.md)
- [06 - CI/CD for ML](10-ML-Testing/06-cicd-for-ml.md)

### Unit 11: Responsible AI in Production
- [01 - Bias Detection](11-Responsible-AI-in-Production/01-bias-detection.md)
- [02 - Fairness Monitoring](11-Responsible-AI-in-Production/02-fairness-monitoring.md)
- [03 - Explainability in Production](11-Responsible-AI-in-Production/03-explainability-in-production.md)
- [04 - Model Governance](11-Responsible-AI-in-Production/04-model-governance.md)
- [05 - Regulatory Compliance](11-Responsible-AI-in-Production/05-regulatory-compliance.md)

---

## Key Tools Covered

| Category | Tools |
|----------|-------|
| Experiment Tracking | MLflow, Weights & Biases, Neptune |
| Data Versioning | DVC, LakeFS, Delta Lake |
| Pipeline Orchestration | Kubeflow, Airflow, Prefect, ZenML |
| Model Serving | TF Serving, TorchServe, Triton, BentoML |
| Monitoring | Evidently AI, WhyLabs, Prometheus, Grafana |
| Infrastructure | Docker, Kubernetes, AWS SageMaker, GCP Vertex AI |
| CI/CD | GitHub Actions, Jenkins, GitLab CI |
| Feature Stores | Feast, Tecton, Hopsworks |

---

## Prerequisites

- Python programming proficiency
- Basic understanding of ML model training
- Familiarity with Git and version control
- Basic command line / terminal skills

---

[Back to Main Repository](../../README.md)
