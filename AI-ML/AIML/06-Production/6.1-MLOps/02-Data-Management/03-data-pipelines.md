# Data Pipelines

## Overview

Data pipelines are automated workflows that move, transform, and prepare data for ML training and serving. A well-designed pipeline ensures data flows reliably from raw sources to model-ready features, with validation, logging, and error handling at every step. This chapter covers pipeline design patterns, orchestration tools, and best practices.

---

## Why Data Pipelines Matter

```
  Without pipelines:              With pipelines:
  ┌─────────────────────┐        ┌─────────────────────┐
  │ Manual scripts       │        │ Automated, scheduled │
  │ "Run step1.py then   │        │ Retry on failure     │
  │  step2.py then..."  │        │ Logged and monitored │
  │ Breaks silently      │        │ Reproducible         │
  │ No error handling    │        │ Versioned            │
  │ "It worked on my PC" │        │ Environment-agnostic │
  └─────────────────────┘        └─────────────────────┘
```

---

## ML Data Pipeline Architecture

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐│
  │  │ Ingest │─►│Validate│─►│ Clean  │─►│Feature │─►│ Store  ││
  │  │        │  │        │  │& Trans.│  │  Eng.  │  │        ││
  │  └────────┘  └────────┘  └────────┘  └────────┘  └────────┘│
  │                                                                │
  │  Ingest:    Pull from DBs, APIs, files, streams              │
  │  Validate:  Schema, quality, completeness checks             │
  │  Clean:     Handle nulls, duplicates, outliers               │
  │  Feature:   Create features, encode categoricals             │
  │  Store:     Save to feature store, data warehouse            │
  │                                                                │
  │  Each step:                                                  │
  │  • Logs inputs/outputs                                       │
  │  • Has retry logic                                           │
  │  • Validates output before passing downstream                │
  │  • Can be monitored independently                            │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Batch vs. Streaming Pipelines

```
  ┌──────────────────┬──────────────────────┬──────────────────────┐
  │ Aspect           │ Batch Pipeline       │ Streaming Pipeline   │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Processing       │ Scheduled intervals  │ Continuous / event   │
  │                  │ (hourly, daily)      │ driven               │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Latency          │ Minutes to hours     │ Seconds to minutes   │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Use case         │ Daily model retrain  │ Real-time features   │
  │                  │ Reporting, analytics │ Fraud detection      │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Tools            │ Airflow, Prefect,    │ Kafka, Flink,        │
  │                  │ Spark, dbt           │ Spark Streaming      │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Complexity       │ Lower               │ Higher               │
  ├──────────────────┼──────────────────────┼──────────────────────┤
  │ Error handling   │ Retry full batch     │ Handle per-event     │
  └──────────────────┴──────────────────────┴──────────────────────┘

  Many ML systems use BOTH:
  • Batch pipeline for training data preparation
  • Streaming pipeline for real-time feature computation
```

---

## Pipeline Orchestration Tools

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Apache Airflow:                                             │
  │    • Most popular orchestrator                               │
  │    • DAGs (Directed Acyclic Graphs) define workflows         │
  │    • Rich UI for monitoring                                  │
  │    • Scheduler, retry, alerting built-in                     │
  │                                                                │
  │  Prefect:                                                    │
  │    • Modern alternative to Airflow                           │
  │    • Pythonic API, easier to learn                           │
  │    • Better error handling and dynamic workflows             │
  │                                                                │
  │  Dagster:                                                    │
  │    • Asset-centric (focus on data products)                  │
  │    • Strong typing and testing                               │
  │    • Good for data engineering + ML                          │
  │                                                                │
  │  Kubeflow Pipelines:                                         │
  │    • ML-specific orchestration on Kubernetes                 │
  │    • Each step is a container                                │
  │    • Integrates with ML experiment tracking                  │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Python: Building a Data Pipeline with Prefect

```python
from prefect import flow, task
import pandas as pd
from datetime import datetime


@task(retries=3, retry_delay_seconds=60)
def ingest_data(source_path: str) -> pd.DataFrame:
    """Load raw data from source."""
    df = pd.read_csv(source_path)
    print(f"Ingested {len(df)} rows")
    return df


@task
def validate_data(df: pd.DataFrame) -> pd.DataFrame:
    """Validate data quality."""
    assert len(df) > 1000, f"Too few rows: {len(df)}"
    assert df["user_id"].nunique() == len(df), "Duplicate user IDs!"
    assert df["age"].between(0, 120).all(), "Invalid age values!"
    
    null_pct = df.isnull().mean()
    for col, pct in null_pct.items():
        assert pct < 0.1, f"Column {col} has {pct:.1%} nulls"
    
    print("✅ Data validation passed")
    return df


@task
def clean_data(df: pd.DataFrame) -> pd.DataFrame:
    """Clean and preprocess data."""
    df = df.drop_duplicates()
    df["age"] = df["age"].fillna(df["age"].median())
    df["income"] = df["income"].clip(lower=0)
    df["signup_date"] = pd.to_datetime(df["signup_date"])
    print(f"Cleaned data: {len(df)} rows")
    return df


@task
def engineer_features(df: pd.DataFrame) -> pd.DataFrame:
    """Create ML features."""
    now = pd.Timestamp.now()
    df["account_age_days"] = (now - df["signup_date"]).dt.days
    df["income_bracket"] = pd.cut(
        df["income"], bins=[0, 30000, 70000, 150000, float("inf")],
        labels=["low", "mid", "high", "very_high"]
    )
    df["purchase_frequency"] = df["purchase_count"] / (df["account_age_days"] + 1)
    print(f"Engineered {len(df.columns)} features")
    return df


@task
def save_features(df: pd.DataFrame, output_path: str):
    """Save processed features."""
    df.to_parquet(output_path, index=False)
    print(f"Saved features to {output_path}")


@flow(name="ml-data-pipeline")
def data_pipeline(source: str, output: str):
    """End-to-end data pipeline."""
    raw = ingest_data(source)
    validated = validate_data(raw)
    cleaned = clean_data(validated)
    features = engineer_features(cleaned)
    save_features(features, output)
    return features


# Run the pipeline
if __name__ == "__main__":
    data_pipeline(
        source="data/raw/users.csv",
        output="data/features/users_features.parquet"
    )
```

---

## Python: Apache Airflow DAG

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

default_args = {
    "owner": "ml-team",
    "retries": 3,
    "retry_delay": timedelta(minutes=5),
    "email_on_failure": True,
    "email": ["ml-team@company.com"],
}

with DAG(
    "ml_data_pipeline",
    default_args=default_args,
    schedule_interval="0 2 * * *",  # daily at 2 AM
    start_date=datetime(2025, 1, 1),
    catchup=False,
) as dag:

    ingest = PythonOperator(
        task_id="ingest_data",
        python_callable=ingest_from_database,
    )

    validate = PythonOperator(
        task_id="validate_data",
        python_callable=run_validation_checks,
    )

    transform = PythonOperator(
        task_id="transform_data",
        python_callable=clean_and_transform,
    )

    features = PythonOperator(
        task_id="engineer_features",
        python_callable=compute_features,
    )

    store = PythonOperator(
        task_id="store_features",
        python_callable=write_to_feature_store,
    )

    # Define execution order
    ingest >> validate >> transform >> features >> store
```

---

## Best Practices

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. IDEMPOTENCY                                              │
  │     Running pipeline twice with same input = same output     │
  │     → Safe to retry on failure                               │
  │                                                                │
  │  2. ATOMICITY                                                │
  │     Each step either fully succeeds or fully fails           │
  │     → No partial writes that corrupt downstream              │
  │                                                                │
  │  3. MONITORING                                               │
  │     Log row counts, null rates, processing time at each step │
  │     → Detect problems early                                  │
  │                                                                │
  │  4. TESTING                                                  │
  │     Unit test each pipeline step independently               │
  │     → Catch bugs before production                           │
  │                                                                │
  │  5. PARAMETERIZATION                                         │
  │     Pass dates, paths, configs as parameters                 │
  │     → Easy to backfill, test, and debug                     │
  │                                                                │
  │  6. INCREMENTAL PROCESSING                                   │
  │     Process only new/changed data, not everything            │
  │     → Faster, cheaper, more scalable                        │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What are the five stages of a typical ML data pipeline?**
2. **When would you use a streaming pipeline vs. a batch pipeline?**
3. **Compare Airflow, Prefect, and Dagster — key differences?**
4. **Why is idempotency important in data pipelines?**
5. **What does incremental processing mean and why is it valuable?**

---

[Previous: 02-data-validation.md](02-data-validation.md) | [Next: 04-feature-stores.md](04-feature-stores.md)
