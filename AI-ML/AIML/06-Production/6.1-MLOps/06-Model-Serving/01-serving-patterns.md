# Serving Patterns (Batch, Real-Time, Streaming)

## Overview

Model serving is how trained models deliver predictions to consumers. The three primary patterns — batch, real-time, and streaming — each solve different latency, throughput, and infrastructure requirements. Choosing the right pattern depends on how fresh predictions need to be, the volume of requests, and the cost constraints of your system.

---

## Serving Pattern Landscape

```
  ┌───────────────────────────────────────────────────────────────────┐
  │                     Model Serving Patterns                       │
  │                                                                   │
  │  ┌─────────────┐   ┌─────────────────┐   ┌───────────────────┐  │
  │  │   Batch     │   │   Real-Time     │   │    Streaming      │  │
  │  │  Inference  │   │   Inference     │   │    Inference      │  │
  │  └──────┬──────┘   └───────┬─────────┘   └────────┬──────────┘  │
  │         │                   │                       │             │
  │   ┌─────▼─────┐     ┌──────▼──────┐       ┌───────▼───────┐    │
  │   │ Scheduled │     │ REST / gRPC │       │ Kafka / Flink │    │
  │   │ Jobs      │     │ Endpoint    │       │ Pipeline      │    │
  │   │ (Spark,   │     │ (FastAPI,   │       │ (Continuous   │    │
  │   │  Airflow) │     │  TF Serving)│       │  processing)  │    │
  │   └─────┬─────┘     └──────┬──────┘       └───────┬───────┘    │
  │         │                   │                       │             │
  │   ┌─────▼─────┐     ┌──────▼──────┐       ┌───────▼───────┐    │
  │   │ Database  │     │  Response   │       │  Event Sink   │    │
  │   │ / S3      │     │  (JSON)    │       │  (DB, Queue)  │    │
  │   └───────────┘     └─────────────┘       └───────────────┘    │
  │                                                                   │
  └───────────────────────────────────────────────────────────────────┘
```

---

## Batch Inference

Predictions are computed on a schedule over large datasets and stored for later retrieval.

```
  ┌──────────┐     ┌───────────┐     ┌──────────┐     ┌──────────┐
  │ Feature  │────►│  Model    │────►│  Write   │────►│  Serve   │
  │ Store /  │     │  Predict  │     │  to DB / │     │  from    │
  │ Data Lake│     │  (Spark)  │     │  S3      │     │  Cache   │
  └──────────┘     └───────────┘     └──────────┘     └──────────┘
       ▲                                                    │
       │            Scheduled: hourly / daily                │
       └──────────── (Airflow, Cron) ───────────────────────┘
```

```python
import pandas as pd
import joblib
from datetime import datetime

# Batch inference pipeline
def run_batch_inference(input_path: str, output_path: str, model_path: str):
    """Score an entire dataset in one run."""
    model = joblib.load(model_path)
    df = pd.read_parquet(input_path)
    
    features = df[["feature_1", "feature_2", "feature_3"]]
    df["prediction"] = model.predict(features)
    df["prediction_proba"] = model.predict_proba(features)[:, 1]
    df["scored_at"] = datetime.utcnow().isoformat()
    
    df.to_parquet(output_path, index=False)
    print(f"Scored {len(df)} rows → {output_path}")


# Airflow DAG snippet
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import timedelta

dag = DAG(
    "batch_scoring",
    schedule_interval="0 6 * * *",  # daily at 6 AM
    default_args={"retries": 2, "retry_delay": timedelta(minutes=5)},
)

score_task = PythonOperator(
    task_id="score_users",
    python_callable=run_batch_inference,
    op_kwargs={
        "input_path": "s3://data/users/latest.parquet",
        "output_path": "s3://predictions/users/{{ ds }}.parquet",
        "model_path": "s3://models/churn/v3/model.joblib",
    },
    dag=dag,
)
```

---

## Real-Time Inference

Predictions are computed on-demand per request via an API endpoint.

```
  ┌──────────┐     ┌──────────────┐     ┌───────────┐
  │  Client  │────►│  API Server  │────►│  Response  │
  │  Request │     │  (FastAPI /  │     │  (JSON)    │
  │  (JSON)  │◄────│  TF Serving) │◄────│            │
  └──────────┘     └──────┬───────┘     └───────────┘
                          │
                   ┌──────▼───────┐
                   │  Model in    │
                   │  Memory      │
                   │  (loaded     │
                   │   at startup)│
                   └──────────────┘
```

```python
from fastapi import FastAPI
import joblib
import numpy as np

app = FastAPI()
model = joblib.load("model.joblib")  # loaded once at startup


@app.post("/predict")
def predict(features: dict):
    """Real-time single prediction."""
    X = np.array([[features["age"], features["income"], features["tenure"]]])
    prediction = model.predict(X)[0]
    probability = model.predict_proba(X)[0].tolist()
    return {
        "prediction": int(prediction),
        "probabilities": probability,
    }


# Start: uvicorn main:app --host 0.0.0.0 --port 8000
```

---

## Streaming Inference

Predictions are computed continuously as events arrive, with low latency and high throughput.

```
  ┌──────────┐     ┌──────────────┐     ┌───────────┐     ┌──────────┐
  │  Event   │────►│  Stream      │────►│  Model    │────►│  Output  │
  │  Source  │     │  Processor   │     │  Predict  │     │  Topic / │
  │  (Kafka) │     │  (Flink /   │     │           │     │  Sink    │
  │          │     │   Spark SS) │     │           │     │          │
  └──────────┘     └──────────────┘     └───────────┘     └──────────┘
```

```python
from kafka import KafkaConsumer, KafkaProducer
import json
import joblib
import numpy as np

model = joblib.load("model.joblib")

consumer = KafkaConsumer(
    "user-events",
    bootstrap_servers="localhost:9092",
    value_deserializer=lambda m: json.loads(m.decode("utf-8")),
)
producer = KafkaProducer(
    bootstrap_servers="localhost:9092",
    value_serializer=lambda v: json.dumps(v).encode("utf-8"),
)


def stream_predict():
    """Continuously consume events, predict, and publish results."""
    for message in consumer:
        event = message.value
        X = np.array([[event["feature_1"], event["feature_2"], event["feature_3"]]])
        pred = model.predict(X)[0]
        
        result = {
            "user_id": event["user_id"],
            "prediction": int(pred),
            "event_time": event["timestamp"],
        }
        producer.send("predictions", value=result)
        producer.flush()


stream_predict()
```

---

## Comparison Table

```
  ┌──────────────┬─────────────────┬──────────────────┬──────────────────┐
  │ Dimension    │ Batch           │ Real-Time        │ Streaming        │
  ├──────────────┼─────────────────┼──────────────────┼──────────────────┤
  │ Latency      │ Minutes–hours   │ Milliseconds     │ Seconds          │
  │ Throughput   │ Very high       │ Medium           │ High             │
  │ Freshness    │ Stale (last run)│ Live             │ Near-real-time   │
  │ Complexity   │ Low             │ Medium           │ High             │
  │ Cost         │ Low (scheduled) │ Medium (always   │ High (always     │
  │              │                 │  running)        │  running + infra)│
  │ Use cases    │ Reports, email  │ Fraud detection, │ Anomaly detect,  │
  │              │ campaigns,      │ recommendations, │ IoT scoring,     │
  │              │ pre-compute     │ search ranking   │ clickstream      │
  │ Infra        │ Spark, Airflow  │ FastAPI, TF      │ Kafka, Flink,    │
  │              │                 │ Serving, Triton  │ Spark Streaming  │
  │ Scaling      │ Horizontal      │ Replicas +       │ Partitions +     │
  │              │ (more workers)  │ load balancer    │ consumers        │
  └──────────────┴─────────────────┴──────────────────┴──────────────────┘
```

---

## When to Use Each Pattern

```
  ┌──────────────────────────────────────────────────────────────────┐
  │  Decision Guide                                                  │
  │                                                                  │
  │  Need results within milliseconds?                               │
  │    YES ──► Real-Time Inference (REST / gRPC endpoint)            │
  │    NO  ──► Can tolerate minutes of delay?                        │
  │              YES ──► Batch Inference (scheduled jobs)             │
  │              NO  ──► Streaming Inference (event-driven pipeline)  │
  │                                                                  │
  │  Additional considerations:                                      │
  │  • Batch is cheapest — prefer it when freshness doesn't matter   │
  │  • Streaming adds Kafka/Flink complexity — justify the overhead  │
  │  • Hybrid: pre-compute batch + real-time for cold-start users    │
  └──────────────────────────────────────────────────────────────────┘
```

---

## Hybrid Pattern: Batch + Real-Time

Many production systems combine patterns. Pre-compute predictions in batch for known entities, fall back to real-time for unknown or new inputs.

```python
import redis
import joblib
import numpy as np

r = redis.Redis()
model = joblib.load("model.joblib")


def get_prediction(user_id: str, features: dict) -> dict:
    """Check cache (batch results) first, fall back to real-time."""
    cached = r.get(f"pred:{user_id}")
    if cached:
        return {"prediction": float(cached), "source": "batch_cache"}
    
    X = np.array([[features["age"], features["income"], features["tenure"]]])
    pred = model.predict(X)[0]
    
    # Cache for future requests
    r.setex(f"pred:{user_id}", 3600, str(pred))
    return {"prediction": int(pred), "source": "real_time"}
```

---

## Revision Questions

1. **What are the three primary model serving patterns and how do they differ in latency?**
2. **When would you choose batch inference over real-time inference?**
3. **What infrastructure components are needed for streaming inference?**
4. **Describe a hybrid serving pattern and when it is useful.**
5. **What are the cost trade-offs between batch and always-on serving?**

---

[Previous: ../05-Model-Versioning-and-Registry/05-model-promotion.md](../05-Model-Versioning-and-Registry/05-model-promotion.md) | [Next: 02-rest-apis-for-ml.md](02-rest-apis-for-ml.md)
