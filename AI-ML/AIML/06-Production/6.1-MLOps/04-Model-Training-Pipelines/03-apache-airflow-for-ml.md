# Apache Airflow for ML

## Overview

**Apache Airflow** is the most widely adopted workflow orchestration platform. Originally designed for data engineering, it's extensively used for ML pipelines thanks to its powerful scheduling, dependency management, rich operator ecosystem, and battle-tested reliability. This chapter covers using Airflow specifically for ML workflows.

---

## Airflow Core Concepts

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Concept          │ Description                          │
  ├──────────────────┼──────────────────────────────────────┤
  │ DAG              │ Directed Acyclic Graph — the pipeline│
  │ Task             │ Single unit of work in the DAG       │
  │ Operator         │ Template for a task (Python, Bash,   │
  │                  │ Docker, K8s, etc.)                   │
  │ Sensor           │ Wait for a condition (file exists,   │
  │                  │ API returns 200, etc.)               │
  │ XCom             │ Cross-task communication (pass data) │
  │ Connection       │ External system credentials          │
  │ Variable         │ Configurable key-value settings      │
  │ Executor         │ How tasks run (local, Celery, K8s)   │
  └──────────────────┴──────────────────────────────────────┘
```

---

## ML Pipeline DAG Example

```python
from airflow import DAG
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.empty import EmptyOperator
from airflow.sensors.filesystem import FileSensor
from datetime import datetime, timedelta

default_args = {
    "owner": "ml-team",
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
    "email_on_failure": True,
    "email": ["ml-alerts@company.com"],
}


def _validate_data(**context):
    import pandas as pd
    df = pd.read_parquet("/data/daily/latest.parquet")
    assert len(df) > 1000, f"Too few rows: {len(df)}"
    assert df.isnull().mean().max() < 0.1, "Too many nulls"
    context["ti"].xcom_push(key="row_count", value=len(df))
    return "Data valid"


def _train_model(**context):
    import mlflow
    from sklearn.ensemble import GradientBoostingClassifier
    import pandas as pd
    import pickle
    
    df = pd.read_parquet("/data/daily/latest.parquet")
    X, y = df.drop("target", axis=1), df["target"]
    
    with mlflow.start_run():
        model = GradientBoostingClassifier(n_estimators=200)
        model.fit(X, y)
        
        score = model.score(X, y)
        mlflow.log_metric("accuracy", score)
        mlflow.sklearn.log_model(model, "model")
        
        context["ti"].xcom_push(key="model_accuracy", value=score)


def _check_model_quality(**context):
    accuracy = context["ti"].xcom_pull(key="model_accuracy")
    if accuracy >= 0.85:
        return "deploy_model"   # branch to deploy
    else:
        return "alert_team"     # branch to alert


def _deploy_model(**context):
    print("Deploying model to production...")
    # Deployment logic here


def _alert_team(**context):
    accuracy = context["ti"].xcom_pull(key="model_accuracy")
    print(f"⚠️ Model accuracy {accuracy} below threshold. Alerting team.")


with DAG(
    "ml_training_pipeline",
    default_args=default_args,
    schedule_interval="0 6 * * *",   # daily at 6 AM
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=["ml", "production"],
) as dag:

    # Wait for new data file
    wait_for_data = FileSensor(
        task_id="wait_for_data",
        filepath="/data/daily/latest.parquet",
        poke_interval=300,     # check every 5 min
        timeout=3600,          # timeout after 1 hour
    )

    validate = PythonOperator(
        task_id="validate_data",
        python_callable=_validate_data,
    )

    train = PythonOperator(
        task_id="train_model",
        python_callable=_train_model,
    )

    check_quality = BranchPythonOperator(
        task_id="check_model_quality",
        python_callable=_check_model_quality,
    )

    deploy = PythonOperator(
        task_id="deploy_model",
        python_callable=_deploy_model,
    )

    alert = PythonOperator(
        task_id="alert_team",
        python_callable=_alert_team,
    )

    done = EmptyOperator(task_id="done", trigger_rule="none_failed_min_one_success")

    # Define DAG
    wait_for_data >> validate >> train >> check_quality
    check_quality >> [deploy, alert] >> done
```

---

## Airflow for ML: Key Operators

```
  ┌─────────────────────┬──────────────────────────────────────┐
  │ Operator            │ ML Use Case                          │
  ├─────────────────────┼──────────────────────────────────────┤
  │ PythonOperator      │ Run Python training / eval scripts   │
  │ BashOperator        │ Run CLI tools (dvc pull, pytest)     │
  │ DockerOperator      │ Run training in Docker container     │
  │ KubernetesPodOp.    │ Run on K8s with GPU resources        │
  │ BranchPythonOp.     │ Conditional logic (deploy or alert)  │
  │ FileSensor          │ Wait for new data arrival            │
  │ S3KeySensor         │ Wait for file in S3                  │
  │ HttpSensor          │ Wait for API endpoint ready          │
  │ TriggerDagRunOp.    │ Trigger another pipeline             │
  └─────────────────────┴──────────────────────────────────────┘
```

---

## Airflow Limitations for ML

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Limitations:                                                │
  │  • XCom not designed for large data (use file paths instead) │
  │  • No native experiment tracking (integrate MLflow)          │
  │  • DAGs are static (limited dynamic pipeline support)        │
  │  • Steep learning curve for advanced features                │
  │  • Not ML-specific (general-purpose orchestrator)            │
  │                                                                │
  │  Best practice: Use Airflow for orchestration,              │
  │  MLflow for tracking, DVC for data, Docker for isolation    │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What are the core Airflow concepts (DAG, Operator, Sensor, XCom)?**
2. **How can BranchPythonOperator implement a deployment gate?**
3. **What Airflow operators are most useful for ML pipelines?**
4. **What are Airflow's limitations for ML workloads?**

---

[Previous: 02-kubeflow-pipelines.md](02-kubeflow-pipelines.md) | [Next: 04-automated-training.md](04-automated-training.md)
