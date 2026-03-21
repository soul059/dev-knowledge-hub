# Pipeline Orchestration

## Overview

Pipeline orchestration automates the execution of multi-step ML workflows — from data ingestion to model deployment. An orchestrator manages task dependencies, scheduling, retries, parallelism, and monitoring. This chapter covers orchestration concepts and compares the major tools: **Apache Airflow**, **Prefect**, **Dagster**, **Kubeflow Pipelines**, and **ZenML**.

---

## Why Orchestrate?

```
  Without orchestration:
    "Run step1.py, then step2.py, then step3.py..."
    • What if step2 fails? Re-run everything?
    • What if step1 and step3 can run in parallel?
    • Who monitors the pipeline at 3 AM?
    • How do you schedule it to run daily?

  With orchestration:
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  ┌──────┐     ┌──────┐                                      │
  │  │Ingest│────►│Valid.│──┐                                    │
  │  └──────┘     └──────┘  │   ┌──────┐   ┌──────┐  ┌──────┐ │
  │                          ├──►│Train │──►│ Eval │─►│Deploy│ │
  │  ┌──────┐     ┌──────┐  │   └──────┘   └──────┘  └──────┘ │
  │  │Feat. │────►│Store │──┘                                    │
  │  │Eng.  │     │      │                                       │
  │  └──────┘     └──────┘                                       │
  │                                                                │
  │  The orchestrator handles:                                   │
  │  ✓ Dependency ordering (train after validate)               │
  │  ✓ Parallelism (ingest + feature eng. in parallel)          │
  │  ✓ Retries on failure (retry step2, not entire pipeline)    │
  │  ✓ Scheduling (daily at 2 AM, on new data arrival)         │
  │  ✓ Monitoring (dashboard, alerts, logs)                     │
  │  ✓ Idempotency (safe to re-run)                            │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## DAG (Directed Acyclic Graph)

```
  Pipelines are represented as DAGs:

  DAG = set of tasks + dependencies (no cycles!)

  ┌──────┐     ┌──────┐     ┌──────┐
  │  A   │────►│  C   │────►│  E   │
  └──────┘     └──────┘     └──────┘
       │                        ▲
       │       ┌──────┐     ┌──┴───┐
       └──────►│  B   │────►│  D   │
               └──────┘     └──────┘

  Execution order: A → (B, C in parallel) → D → E
  
  Properties:
  • Directed: tasks have order (A before B)
  • Acyclic: no circular dependencies
  • Each task runs ONCE per pipeline execution
```

---

## Tool Comparison

```
  ┌─────────────┬───────────┬───────────┬───────────┬───────────┬───────────┐
  │ Feature     │ Airflow   │ Prefect   │ Dagster   │ Kubeflow  │ ZenML     │
  ├─────────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │ Focus       │ General   │ General   │ Data +    │ ML on K8s │ ML-native │
  │             │ workflow  │ workflow  │ asset-    │           │           │
  │             │           │           │ centric   │           │           │
  ├─────────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │ DAG def.    │ Python    │ Python    │ Python    │ Python/   │ Python    │
  │             │           │           │           │ YAML      │           │
  ├─────────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │ Scheduling  │ ✓✓✓       │ ✓✓        │ ✓✓        │ ✓         │ ✓         │
  ├─────────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │ Dynamic     │ Limited   │ ✓✓✓       │ ✓✓        │ Limited   │ ✓         │
  │ pipelines   │           │           │           │           │           │
  ├─────────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │ ML-specific │ Basic     │ Basic     │ Medium    │ ✓✓✓       │ ✓✓✓       │
  ├─────────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │ K8s native  │ Via exec. │ Via agent │ Via exec. │ ✓✓✓       │ ✓         │
  ├─────────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │ Learning    │ Medium    │ Low       │ Medium    │ High      │ Low       │
  │ curve       │           │           │           │           │           │
  ├─────────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │ Community   │ Huge      │ Growing   │ Growing   │ Medium    │ Small     │
  └─────────────┴───────────┴───────────┴───────────┴───────────┴───────────┘
```

---

## Python: Simple Pipeline with ZenML

```python
from zenml import pipeline, step
from zenml.client import Client
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import f1_score


@step
def load_data() -> pd.DataFrame:
    return pd.read_csv("data/train.csv")


@step
def preprocess(df: pd.DataFrame) -> tuple:
    X = df.drop("target", axis=1)
    y = df["target"]
    return X, y


@step
def train_model(X: pd.DataFrame, y: pd.Series) -> RandomForestClassifier:
    model = RandomForestClassifier(n_estimators=100)
    model.fit(X, y)
    return model


@step
def evaluate(model: RandomForestClassifier, X: pd.DataFrame, 
             y: pd.Series) -> float:
    predictions = model.predict(X)
    score = f1_score(y, predictions)
    print(f"F1 Score: {score:.4f}")
    return score


@pipeline
def training_pipeline():
    df = load_data()
    X, y = preprocess(df)
    model = train_model(X, y)
    score = evaluate(model, X, y)


# Run the pipeline
training_pipeline()
```

---

## Choosing the Right Orchestrator

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Use Airflow when:                                           │
  │  • Complex scheduling needs (cron, sensors, triggers)        │
  │  • Existing Airflow infrastructure                           │
  │  • Data engineering + ML combined workflows                  │
  │                                                                │
  │  Use Prefect when:                                           │
  │  • Want modern, Pythonic API                                 │
  │  • Need dynamic pipelines (unknown tasks at runtime)         │
  │  • Smaller team, quick setup                                 │
  │                                                                │
  │  Use Kubeflow when:                                          │
  │  • Kubernetes-native environment                             │
  │  • Each step needs isolated container                        │
  │  • Heavy ML workloads with GPU scheduling                    │
  │                                                                │
  │  Use ZenML when:                                             │
  │  • ML-first pipeline framework needed                        │
  │  • Want to swap orchestrators easily                         │
  │  • Smaller team, getting started with MLOps                  │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What problems does pipeline orchestration solve?**
2. **What is a DAG and why is the "acyclic" property important?**
3. **Compare Airflow and Prefect — when would you choose each?**
4. **What makes Kubeflow Pipelines unique compared to general orchestrators?**
5. **What does a pipeline orchestrator handle automatically?**

---

[Next: 02-kubeflow-pipelines.md](02-kubeflow-pipelines.md)
