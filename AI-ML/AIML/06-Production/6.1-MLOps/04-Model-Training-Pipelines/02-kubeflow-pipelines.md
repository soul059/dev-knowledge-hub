# Kubeflow Pipelines

## Overview

**Kubeflow Pipelines** is an ML pipeline orchestration platform built on **Kubernetes**. Each pipeline step runs as an isolated container, enabling reproducibility, scalability, and GPU scheduling. It's ideal for teams already using Kubernetes who need containerized, production-grade ML workflows with experiment tracking and artifact management built in.

---

## Architecture

```
  ┌──────────────────────────────────────────────────────────────┐
  │                    Kubeflow Pipelines                          │
  │                                                                │
  │  ┌──────────────────────────────────────────────────────┐    │
  │  │                   Pipeline UI                        │    │
  │  │  • Visualize DAGs, track runs, compare experiments  │    │
  │  └──────────────────────────────────────────────────────┘    │
  │                          │                                    │
  │  ┌──────────────────────────────────────────────────────┐    │
  │  │                Pipeline Service                      │    │
  │  │  • REST API, scheduling, run management             │    │
  │  └──────────────────────────────────────────────────────┘    │
  │                          │                                    │
  │  ┌──────────────────────────────────────────────────────┐    │
  │  │               Kubernetes Cluster                     │    │
  │  │                                                      │    │
  │  │  ┌────────┐   ┌────────┐   ┌────────┐              │    │
  │  │  │Step 1  │──►│Step 2  │──►│Step 3  │              │    │
  │  │  │(pod)   │   │(pod)   │   │(pod)   │              │    │
  │  │  │CPU     │   │GPU     │   │CPU     │              │    │
  │  │  └────────┘   └────────┘   └────────┘              │    │
  │  │                                                      │    │
  │  │  Each step = separate Docker container (pod)        │    │
  │  │  Data passed via volumes or artifact store          │    │
  │  └──────────────────────────────────────────────────────┘    │
  │                                                                │
  │  ┌──────────────────┐  ┌──────────────────┐                  │
  │  │ Artifact Store   │  │ Metadata Store   │                  │
  │  │ (Minio/S3/GCS)   │  │ (MySQL)          │                  │
  │  └──────────────────┘  └──────────────────┘                  │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Python: Defining a Kubeflow Pipeline

```python
from kfp import dsl, compiler
from kfp.dsl import Input, Output, Dataset, Model, Metrics


@dsl.component(base_image="python:3.11-slim",
               packages_to_install=["pandas", "scikit-learn"])
def load_data(output_data: Output[Dataset]):
    import pandas as pd
    df = pd.read_csv("gs://my-bucket/data/train.csv")
    df.to_csv(output_data.path, index=False)


@dsl.component(base_image="python:3.11-slim",
               packages_to_install=["pandas", "scikit-learn"])
def train_model(
    input_data: Input[Dataset],
    n_estimators: int,
    max_depth: int,
    output_model: Output[Model],
    output_metrics: Output[Metrics],
):
    import pandas as pd
    import pickle
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import accuracy_score, f1_score
    
    df = pd.read_csv(input_data.path)
    X = df.drop("target", axis=1)
    y = df["target"]
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
    
    model = RandomForestClassifier(
        n_estimators=n_estimators, max_depth=max_depth
    )
    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    
    acc = accuracy_score(y_test, y_pred)
    f1 = f1_score(y_test, y_pred)
    output_metrics.log_metric("accuracy", acc)
    output_metrics.log_metric("f1_score", f1)
    
    with open(output_model.path, "wb") as f:
        pickle.dump(model, f)


@dsl.component(base_image="python:3.11-slim")
def deploy_model(
    model: Input[Model],
    f1_threshold: float,
    metrics: Input[Metrics],
):
    print(f"Checking deployment gate...")
    # Deploy logic here if metrics meet threshold
    print(f"Model deployed successfully!")


@dsl.pipeline(name="ml-training-pipeline")
def training_pipeline(
    n_estimators: int = 100,
    max_depth: int = 10,
    f1_threshold: float = 0.8,
):
    load_task = load_data()
    
    train_task = train_model(
        input_data=load_task.outputs["output_data"],
        n_estimators=n_estimators,
        max_depth=max_depth,
    )
    # Request GPU for training
    train_task.set_gpu_limit(1)
    train_task.set_memory_limit("8Gi")
    
    deploy_task = deploy_model(
        model=train_task.outputs["output_model"],
        f1_threshold=f1_threshold,
        metrics=train_task.outputs["output_metrics"],
    )


# Compile to YAML
compiler.Compiler().compile(training_pipeline, "pipeline.yaml")

# Submit to Kubeflow
from kfp.client import Client
client = Client(host="http://kubeflow.example.com")
client.create_run_from_pipeline_package(
    "pipeline.yaml",
    arguments={"n_estimators": 200, "max_depth": 15},
)
```

---

## When to Use Kubeflow

```
  ✓ Already running Kubernetes
  ✓ Need containerized, isolated steps
  ✓ Need GPU scheduling per step
  ✓ Want built-in experiment tracking UI
  ✓ Multi-team environment with shared cluster

  ✗ Don't use if:
  ✗ No Kubernetes expertise on team
  ✗ Simple pipelines (Prefect/ZenML simpler)
  ✗ Single-machine workloads
```

---

## Revision Questions

1. **How does Kubeflow Pipelines use Kubernetes pods for pipeline steps?**
2. **What are the advantages of running each step in a separate container?**
3. **How do you pass data between pipeline steps in Kubeflow?**
4. **When should you choose Kubeflow over simpler orchestrators?**

---

[Previous: 01-pipeline-orchestration.md](01-pipeline-orchestration.md) | [Next: 03-apache-airflow-for-ml.md](03-apache-airflow-for-ml.md)
