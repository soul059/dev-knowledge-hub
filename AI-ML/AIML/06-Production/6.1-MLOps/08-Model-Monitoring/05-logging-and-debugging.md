# 🔍 Logging and Debugging ML Systems

> **Unit 8 · Chapter 5** — Structured logging for ML pipelines, prediction logging, debugging production failures, request tracing, and the ELK stack for ML observability.

---

## 📋 Chapter Overview

When an ML model misbehaves in production, logs are your **crime scene evidence**. Unlike traditional software bugs that produce stack traces, ML failures are often **silent** — the model returns a valid response, but the prediction is wrong. Debugging requires tracing the full journey from raw input through feature engineering to model inference and back.

Effective ML logging goes beyond `print()` statements. It requires **structured logging** (machine-parseable JSON), **prediction logging** (inputs, outputs, and metadata for every inference), and **distributed tracing** (following a request across microservices). Combined with tools like the ELK stack (Elasticsearch, Logstash, Kibana), teams can diagnose production ML failures in minutes instead of days.

This chapter covers:

- Structured logging principles for ML systems
- Prediction logging: what to capture and how
- Debugging common production ML failures
- Distributed tracing for ML pipelines
- ELK stack setup for ML observability

---

## 🏗️ ML Logging Architecture

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                    ML LOGGING PIPELINE                          │
  │                                                                  │
  │  ┌────────────┐                                                 │
  │  │   Client   │──── Request ────┐                               │
  │  └────────────┘                 │                               │
  │                                  ▼                               │
  │  ┌──────────────────────────────────────────────────────────┐   │
  │  │                    ML SERVICE                             │   │
  │  │                                                           │   │
  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐  │   │
  │  │  │ Request  │  │ Feature  │  │  Model   │  │Response │  │   │
  │  │  │ Parsing  │─▶│ Extract  │─▶│ Inference│─▶│ Format  │  │   │
  │  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬────┘  │   │
  │  │       │LOG          │LOG          │LOG          │LOG     │   │
  │  │       ▼             ▼             ▼             ▼        │   │
  │  │  ┌───────────────────────────────────────────────────┐   │   │
  │  │  │              STRUCTURED LOG EMITTER               │   │   │
  │  │  │  { "trace_id", "step", "features", "prediction" } │   │   │
  │  │  └───────────────────────┬───────────────────────────┘   │   │
  │  └──────────────────────────┼───────────────────────────────┘   │
  │                              │                                   │
  │                              ▼                                   │
  │  ┌──────────────────────────────────────────────────────────┐   │
  │  │                    LOG PIPELINE                           │   │
  │  │                                                           │   │
  │  │  ┌──────────┐    ┌──────────────┐    ┌──────────────┐    │   │
  │  │  │ Logstash │    │Elasticsearch │    │   Kibana     │    │   │
  │  │  │ (Ingest) │───▶│  (Store /    │───▶│ (Visualize / │    │   │
  │  │  │          │    │   Search)    │    │   Query)     │    │   │
  │  │  └──────────┘    └──────────────┘    └──────────────┘    │   │
  │  └──────────────────────────────────────────────────────────┘   │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 📝 Structured Logging

### Why Structured Logging?

```
  ❌ UNSTRUCTURED (hard to parse, query, and analyze):
  ─────────────────────────────────────────────────
  2024-01-15 10:23:45 INFO Prediction made: 0.87 for user 12345,
  latency 23ms, model fraud_v3

  ✅ STRUCTURED JSON (machine-parseable, filterable):
  ───────────────────────────────────────────────────
  {
    "timestamp": "2024-01-15T10:23:45.123Z",
    "level": "INFO",
    "service": "fraud-detector",
    "trace_id": "abc-123-def",
    "event": "prediction_complete",
    "model_name": "fraud_detector",
    "model_version": "v3",
    "user_id": "12345",
    "prediction": 0.87,
    "latency_ms": 23,
    "features_hash": "sha256:9f86d..."
  }
```

---

## 🐍 Python Implementation

### Structured Logger for ML

```python
import json
import logging
import uuid
import time
import hashlib
from datetime import datetime, timezone
from typing import Any, Dict, Optional
from contextvars import ContextVar
from functools import wraps

# Context variable for request tracing
current_trace_id: ContextVar[str] = ContextVar(
    "trace_id", default="no-trace"
)


class StructuredMLFormatter(logging.Formatter):
    """JSON formatter that includes ML-specific fields."""

    def format(self, record: logging.LogRecord) -> str:
        log_entry = {
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "trace_id": current_trace_id.get(),
            "service": "ml-service",
        }

        # Include extra ML fields if present
        for field in [
            "model_name", "model_version", "prediction",
            "latency_ms", "features", "error_type",
            "user_id", "request_id", "step",
        ]:
            if hasattr(record, field):
                log_entry[field] = getattr(record, field)

        if record.exc_info:
            log_entry["exception"] = self.formatException(record.exc_info)

        return json.dumps(log_entry)


def setup_ml_logger(name: str, level: int = logging.INFO) -> logging.Logger:
    """Create a structured ML logger."""
    logger = logging.getLogger(name)
    logger.setLevel(level)

    handler = logging.StreamHandler()
    handler.setFormatter(StructuredMLFormatter())
    logger.addHandler(handler)

    return logger


# --- Initialize ---
ml_logger = setup_ml_logger("ml_service")
```

### Prediction Logger

```python
import numpy as np
from dataclasses import dataclass, field, asdict
from typing import List


@dataclass
class PredictionLog:
    """Complete record of a single prediction for debugging."""
    request_id: str
    trace_id: str
    timestamp: str
    model_name: str
    model_version: str
    input_features: Dict[str, Any]
    feature_vector: List[float]
    prediction: Any
    prediction_proba: Optional[List[float]]
    latency_ms: float
    feature_hash: str
    metadata: Dict[str, Any] = field(default_factory=dict)


class PredictionLogger:
    """
    Logs every prediction with full context for debugging
    and offline analysis.
    """

    def __init__(
        self,
        logger: logging.Logger,
        log_features: bool = True,
        sample_rate: float = 1.0,  # Log 100% by default
    ):
        self.logger = logger
        self.log_features = log_features
        self.sample_rate = sample_rate
        self.buffer: List[PredictionLog] = []
        self.buffer_size = 100

    def _compute_feature_hash(self, features: dict) -> str:
        """Hash features for deduplication and lookup."""
        feature_str = json.dumps(features, sort_keys=True, default=str)
        return hashlib.sha256(feature_str.encode()).hexdigest()[:16]

    def log_prediction(
        self,
        model_name: str,
        model_version: str,
        input_features: dict,
        feature_vector: list,
        prediction: Any,
        prediction_proba: Optional[list],
        latency_ms: float,
        metadata: Optional[dict] = None,
    ) -> PredictionLog:
        """Log a prediction with full context."""

        # Apply sampling
        if np.random.random() > self.sample_rate:
            return None

        request_id = str(uuid.uuid4())
        trace_id = current_trace_id.get()

        pred_log = PredictionLog(
            request_id=request_id,
            trace_id=trace_id,
            timestamp=datetime.now(timezone.utc).isoformat(),
            model_name=model_name,
            model_version=model_version,
            input_features=(
                input_features if self.log_features else {}
            ),
            feature_vector=(
                feature_vector if self.log_features else []
            ),
            prediction=prediction,
            prediction_proba=prediction_proba,
            latency_ms=latency_ms,
            feature_hash=self._compute_feature_hash(input_features),
            metadata=metadata or {},
        )

        # Structured log
        self.logger.info(
            "prediction_logged",
            extra={
                "model_name": model_name,
                "model_version": model_version,
                "prediction": prediction,
                "latency_ms": latency_ms,
                "request_id": request_id,
                "step": "inference",
            },
        )

        # Buffer for batch writing
        self.buffer.append(pred_log)
        if len(self.buffer) >= self.buffer_size:
            self._flush_buffer()

        return pred_log

    def _flush_buffer(self):
        """Write buffered predictions to storage."""
        if not self.buffer:
            return
        # In production: write to BigQuery, S3, or Kafka
        self.logger.info(
            f"Flushed {len(self.buffer)} prediction logs to storage"
        )
        self.buffer.clear()


# --- Usage ---
pred_logger = PredictionLogger(ml_logger, sample_rate=1.0)
```

### Request Tracing Middleware

```python
from contextlib import contextmanager


@contextmanager
def trace_request(request_id: Optional[str] = None):
    """Context manager for distributed tracing."""
    trace_id = request_id or str(uuid.uuid4())
    token = current_trace_id.set(trace_id)

    ml_logger.info(
        "request_started",
        extra={"step": "request_start", "request_id": trace_id},
    )

    start_time = time.time()
    try:
        yield trace_id
    except Exception as e:
        ml_logger.error(
            f"request_failed: {str(e)}",
            extra={
                "step": "request_error",
                "error_type": type(e).__name__,
                "request_id": trace_id,
            },
            exc_info=True,
        )
        raise
    finally:
        elapsed = (time.time() - start_time) * 1000
        ml_logger.info(
            "request_completed",
            extra={
                "step": "request_end",
                "latency_ms": round(elapsed, 2),
                "request_id": trace_id,
            },
        )
        current_trace_id.reset(token)


def trace_step(step_name: str):
    """Decorator to trace individual pipeline steps."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            ml_logger.info(
                f"step_started: {step_name}",
                extra={"step": step_name},
            )
            start = time.time()
            try:
                result = func(*args, **kwargs)
                elapsed = (time.time() - start) * 1000
                ml_logger.info(
                    f"step_completed: {step_name}",
                    extra={
                        "step": step_name,
                        "latency_ms": round(elapsed, 2),
                    },
                )
                return result
            except Exception as e:
                ml_logger.error(
                    f"step_failed: {step_name}: {e}",
                    extra={
                        "step": step_name,
                        "error_type": type(e).__name__,
                    },
                    exc_info=True,
                )
                raise
        return wrapper
    return decorator


# --- Full traced ML service example ---

class TracedMLService:
    """ML service with comprehensive logging and tracing."""

    def __init__(self, model, model_name: str, model_version: str):
        self.model = model
        self.model_name = model_name
        self.model_version = model_version
        self.pred_logger = PredictionLogger(ml_logger)

    @trace_step("validate_input")
    def validate_input(self, raw_input: dict) -> dict:
        """Validate and sanitize input."""
        required_fields = ["age", "income", "transaction_amount"]
        missing = [f for f in required_fields if f not in raw_input]
        if missing:
            raise ValueError(f"Missing required fields: {missing}")
        return raw_input

    @trace_step("extract_features")
    def extract_features(self, validated_input: dict) -> list:
        """Transform raw input into feature vector."""
        return [
            validated_input["age"],
            np.log1p(validated_input["income"]),
            validated_input["transaction_amount"],
        ]

    @trace_step("model_inference")
    def run_inference(self, feature_vector: list) -> tuple:
        """Run model prediction."""
        prediction = self.model.predict([feature_vector])[0]
        proba = self.model.predict_proba([feature_vector])[0].tolist()
        return prediction, proba

    def predict(self, raw_input: dict) -> dict:
        """Full prediction pipeline with tracing."""
        with trace_request() as trace_id:
            start = time.time()

            validated = self.validate_input(raw_input)
            features = self.extract_features(validated)
            prediction, proba = self.run_inference(features)
            latency = (time.time() - start) * 1000

            self.pred_logger.log_prediction(
                model_name=self.model_name,
                model_version=self.model_version,
                input_features=raw_input,
                feature_vector=features,
                prediction=int(prediction),
                prediction_proba=proba,
                latency_ms=latency,
            )

            return {
                "prediction": int(prediction),
                "probability": proba,
                "trace_id": trace_id,
                "latency_ms": round(latency, 2),
            }
```

---

## 🔧 Debugging Common Production ML Failures

```
  ┌──────────────────────────────────────────────────────────────────┐
  │              COMMON ML PRODUCTION FAILURES                       │
  │                                                                  │
  │  ┌───────────────────┐  ┌──────────────────────────────────────┐│
  │  │ FAILURE           │  │ DEBUG STEPS                          ││
  │  ├───────────────────┤  ├──────────────────────────────────────┤│
  │  │                   │  │                                      ││
  │  │ NaN/Null          │  │ 1. Check input features for nulls   ││
  │  │ predictions       │  │ 2. Check feature engineering logs   ││
  │  │                   │  │ 3. Verify model loaded correctly    ││
  │  │                   │  │ 4. Check for division by zero in    ││
  │  │                   │  │    feature transforms               ││
  │  ├───────────────────┤  ├──────────────────────────────────────┤│
  │  │                   │  │                                      ││
  │  │ Latency spike     │  │ 1. Check model input size (batching)││
  │  │                   │  │ 2. Check GPU/CPU utilization        ││
  │  │                   │  │ 3. Check feature store latency      ││
  │  │                   │  │ 4. Check for memory pressure / GC   ││
  │  ├───────────────────┤  ├──────────────────────────────────────┤│
  │  │                   │  │                                      ││
  │  │ Wrong predictions │  │ 1. Compare input features to        ││
  │  │ (silent failure)  │  │    training distribution            ││
  │  │                   │  │ 2. Check for feature name mismatch  ││
  │  │                   │  │ 3. Verify feature ordering          ││
  │  │                   │  │ 4. Check for stale feature cache    ││
  │  ├───────────────────┤  ├──────────────────────────────────────┤│
  │  │                   │  │                                      ││
  │  │ Model crash /     │  │ 1. Check OOM in container logs     ││
  │  │ OOM               │  │ 2. Verify input tensor shapes      ││
  │  │                   │  │ 3. Check batch size limits          ││
  │  │                   │  │ 4. Review memory profile of model   ││
  │  └───────────────────┘  └──────────────────────────────────────┘│
  └──────────────────────────────────────────────────────────────────┘
```

### Debugging Utility Functions

```python
class MLDebugger:
    """Utilities for debugging production ML issues."""

    @staticmethod
    def compare_features_to_training(
        production_features: dict,
        training_stats: dict,
    ) -> dict:
        """
        Check if production features are within training distribution.
        training_stats: {"feature_name": {"mean": x, "std": y, "min": a, "max": b}}
        """
        anomalies = {}
        for feature, value in production_features.items():
            if feature not in training_stats:
                anomalies[feature] = {
                    "issue": "unknown_feature",
                    "value": value,
                }
                continue

            stats = training_stats[feature]
            z_score = abs(value - stats["mean"]) / (stats["std"] + 1e-8)

            if z_score > 5:
                anomalies[feature] = {
                    "issue": "extreme_outlier",
                    "value": value,
                    "z_score": round(z_score, 2),
                    "training_range": [stats["min"], stats["max"]],
                }
            elif value < stats["min"] or value > stats["max"]:
                anomalies[feature] = {
                    "issue": "out_of_training_range",
                    "value": value,
                    "training_range": [stats["min"], stats["max"]],
                }

        return {
            "anomalies_found": len(anomalies),
            "details": anomalies,
        }

    @staticmethod
    def check_feature_consistency(
        expected_features: list,
        received_features: list,
    ) -> dict:
        """Check for missing, extra, or reordered features."""
        expected_set = set(expected_features)
        received_set = set(received_features)

        return {
            "missing": list(expected_set - received_set),
            "unexpected": list(received_set - expected_set),
            "order_match": expected_features == received_features,
            "count_match": len(expected_features) == len(received_features),
        }

    @staticmethod
    def replay_prediction(
        model,
        prediction_log: PredictionLog,
    ) -> dict:
        """Replay a logged prediction to verify reproducibility."""
        replayed = model.predict([prediction_log.feature_vector])[0]
        original = prediction_log.prediction

        return {
            "original_prediction": original,
            "replayed_prediction": replayed,
            "match": original == replayed,
            "request_id": prediction_log.request_id,
            "feature_hash": prediction_log.feature_hash,
        }
```

---

## 📦 ELK Stack for ML

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                    ELK STACK FOR ML                              │
  │                                                                  │
  │  ┌────────────┐    ┌────────────┐    ┌────────────┐             │
  │  │ ML Service │    │ ML Service │    │ Training   │             │
  │  │ (logs)     │    │ (logs)     │    │ Pipeline   │             │
  │  └─────┬──────┘    └─────┬──────┘    └─────┬──────┘             │
  │        │                 │                 │                     │
  │        └─────────────────┼─────────────────┘                    │
  │                          │                                       │
  │                          ▼                                       │
  │               ┌──────────────────┐                              │
  │               │    LOGSTASH      │                              │
  │               │  (or Filebeat)   │                              │
  │               │  ──────────────  │                              │
  │               │  • Parse JSON    │                              │
  │               │  • Add fields    │                              │
  │               │  • Filter/route  │                              │
  │               └────────┬─────────┘                              │
  │                        │                                         │
  │                        ▼                                         │
  │               ┌──────────────────┐                              │
  │               │ ELASTICSEARCH    │                              │
  │               │  ──────────────  │                              │
  │               │  Index: ml-preds │                              │
  │               │  Index: ml-errors│                              │
  │               │  Index: ml-drift │                              │
  │               └────────┬─────────┘                              │
  │                        │                                         │
  │                        ▼                                         │
  │               ┌──────────────────┐                              │
  │               │     KIBANA       │                              │
  │               │  ──────────────  │                              │
  │               │  • Pred explorer │                              │
  │               │  • Error dashboard│                             │
  │               │  • Trace viewer  │                              │
  │               └──────────────────┘                              │
  └──────────────────────────────────────────────────────────────────┘

  Useful Kibana Queries:
  ─────────────────────
  • Find all predictions for a user:
    user_id: "12345" AND event: "prediction_complete"

  • Find errors in the last hour:
    level: "ERROR" AND @timestamp > now-1h

  • Trace a request across services:
    trace_id: "abc-123-def"

  • Find slow predictions:
    latency_ms > 500 AND event: "prediction_complete"
```

### Docker Compose for ELK + ML Service

```python
DOCKER_COMPOSE_ELK = """
version: '3.8'

services:
  elasticsearch:
    image: elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - 'ES_JAVA_OPTS=-Xms512m -Xmx512m'
    ports:
      - '9200:9200'
    volumes:
      - es-data:/usr/share/elasticsearch/data

  logstash:
    image: logstash:8.11.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: kibana:8.11.0
    ports:
      - '5601:5601'
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

  ml-service:
    build: .
    environment:
      - LOG_FORMAT=json
      - LOG_LEVEL=INFO
    logging:
      driver: 'json-file'
      options:
        max-size: '10m'
        max-file: '3'

volumes:
  es-data:
"""

LOGSTASH_CONFIG = """
input {
  file {
    path => "/var/log/ml-service/*.log"
    codec => json
    start_position => "beginning"
  }
}

filter {
  # Parse timestamp
  date {
    match => ["timestamp", "ISO8601"]
    target => "@timestamp"
  }

  # Add ML-specific fields for indexing
  if [event] == "prediction_complete" {
    mutate {
      add_field => { "[@metadata][index]" => "ml-predictions" }
    }
  } else if [level] == "ERROR" {
    mutate {
      add_field => { "[@metadata][index]" => "ml-errors" }
    }
  } else {
    mutate {
      add_field => { "[@metadata][index]" => "ml-logs" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "%{[@metadata][index]}-%{+YYYY.MM.dd}"
  }
}
"""
```

---

## 📊 What to Log — Summary Table

| Category | What to Log | Why | Storage Cost |
|----------|-------------|-----|-------------|
| **Request metadata** | trace_id, timestamp, user_id, endpoint | Tracing, debugging | Low |
| **Input features** | Raw + processed feature values | Reproduce predictions, drift analysis | High |
| **Feature hash** | SHA256 of feature vector | Dedup, lookup without storing full features | Low |
| **Prediction** | Output value, probabilities, class | Quality monitoring, audit trail | Low |
| **Latency** | Per-step and total inference time | Performance monitoring | Low |
| **Model info** | Name, version, framework | Multi-model debugging | Low |
| **Errors** | Exception type, stack trace, input context | Root cause analysis | Medium |
| **Feature store** | Cache hit/miss, staleness, lookup time | Feature pipeline debugging | Low |

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **Use structured JSON logging** — unstructured text logs are nearly useless for automated analysis and dashboarding at scale. |
| 2 | **Log every prediction** (or sample at high traffic) — you need input/output pairs to diagnose silent failures and compute accuracy post-hoc. |
| 3 | **Distributed tracing is essential** — a single trace_id linking request ingestion → feature extraction → inference → response enables end-to-end debugging. |
| 4 | **Feature hashing** saves storage while enabling replay — store the full feature vector for sampled predictions, but log the hash for every prediction. |
| 5 | **The ELK stack** (or alternatives like Loki/Grafana) provides the search, visualization, and alerting infrastructure that makes logs actionable. |

---

## ❓ Revision Questions

1. **Why is structured logging critical for production ML systems?**
   > Structured (JSON) logs can be automatically parsed, indexed, queried, and visualized by tools like Elasticsearch and Kibana. With unstructured text logs, finding "all predictions with latency > 500ms for model v3 in the last hour" requires complex regex parsing. With structured logs, it's a simple field-based query. Structured logs also enable automated alerting on specific fields.

2. **What information should a prediction log capture, and why?**
   > A prediction log should capture: (1) input features (to reproduce the prediction and analyze drift), (2) the prediction and probabilities (for quality monitoring), (3) latency (for performance tracking), (4) model name/version (to identify which model made the prediction), (5) trace_id (to correlate with upstream/downstream services), and (6) timestamp (for temporal analysis). This enables debugging, auditing, and offline accuracy computation.

3. **How does request tracing help debug ML failures that span multiple services?**
   > In a microservice architecture, a prediction request may flow through an API gateway, feature store, preprocessing service, and model server. Without a shared trace_id propagated across all services, correlating logs from these different components is impossible. With tracing, an engineer can query all logs for a single trace_id and see the complete request journey, identifying exactly where a failure or delay occurred.

4. **Why might you use feature hashing instead of logging full feature vectors?**
   > Full feature vectors can be large (hundreds of features, high-cardinality embeddings), making it expensive to store for every prediction at high QPS. A feature hash (SHA256 of the sorted feature JSON) is fixed-size and cheap to store. It enables deduplication (detecting identical inputs), prediction replay (lookup the full features by hash from a sampled store), and consistency checks — all without the storage cost of logging every feature for every request.

5. **Describe how you would debug a model that suddenly starts returning wrong predictions without any error logs.**
   > (1) Check the prediction distribution dashboard — has the output distribution shifted? (2) Query recent prediction logs and compare input features to training statistics using z-scores. (3) Verify feature ordering matches what the model expects (a common silent failure). (4) Check if the model version changed recently in the registry. (5) Replay a few logged predictions to verify reproducibility. (6) Compare against a shadow model or rule-based fallback. (7) Check upstream data pipelines for schema changes or data quality issues.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Alerting Systems](./04-alerting.md) | [📂 Model Monitoring](./README.md) | [Feedback Loops →](./06-feedback-loops.md) |

---

*Unit 8 · Chapter 5 · Logging and Debugging ML Systems — MLOps Course Notes*
